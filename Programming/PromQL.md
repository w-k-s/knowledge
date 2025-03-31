# PromQL

## 1. Meters

### 1.1 Counters

- A counter is a meter whose value can only increment. 
- Decrecments can occur when the counter is reset. This typically occurs if the node restarts.
- If an application has multiple replicas, a label must be used to distinguish count of each replica, otherwise the counter will display erratic behaviour.

Let's explore the last point further.
- Consider you have multiple replicas of a microservice e.g. the `replicaCount` is set for your service is set to 3 on your Kubernetes Cluster.
- Each replica has the same codebase therefore each replica has a counter with the same name e.g. `endpoint_requests_total` that increments whenever an API endpoint is called.
- A load balancer routes incoming requests to one of the available replicas.
- Let's consider the following sequence of events:
    - At 00:00, Replica 1 receives a request, Replica 1's counter increments to 1
    - At 00:05, Replica 1 receives a request, Replica 1's counter increments to 2
    - At 00:10, Replica 2 receives a request, Replica 2's counter increment to 1
    - At 00:15, Replica 1 receives a request, Replica 1's counter increment to 3
    - At 00:20, Replica 3 receives a request, Replica 3's counter increment to 1
- If all counters use the same name, without any label to distinguish between the node, then we will see our counters value oscillating:

![](./images/promql-counter-without-instanceid.excalidraw.png)

- If the counters value oscilates like this, then any function that we apply on the counter e.g. `increase`, `rate` will result in wild values.
- To distinguish between the counters, we assign a label to the counter when it is created (at the time that the application launched). This label is typically called `instance.id` and it is assigned a value that uniquely identifies the replica. For example, this could be the instance ID of an EC2.
- This ofcourse raises the question, what if we wanted to total the count across all instances; see the `sum by` aggregate function below.

![](./images/promql-counter-with-instanceid.excalidraw.png)

The same concept applies for applications that run on multiple threads or applications that are forked to run on multiple processes. Let's consider the example of a Flask application running on Gunicorn. 

- Gunicorn runs a Flask application on multiple workers. Each worker is capable of handling HTTP requests. We can think of each worker as a replica of the flask application.
- Gunicorn creates workers in one of 2 modes:
    - With `preloading`, a worker is created in a single process. Then, the process is forked and each child process receives copy of a resources from the parent process. This includes global variables, sockets and database connections.
    - Without `preloading`, a worker is created in separate processes. Each process has its own sockets, database connections and global variables.

- Therefore, when `preloading` you have to be careful that the same `instance.id` is not copied to each forked process.
- This can happen, for example, if the `instance.id` is set to the Process ID `getpid()` on the main process. Once the main process if forked, each child process has the same `instance.id` set to the id of the parent process, rather than the id of the forked child process. In this case, you could expect to see oscillating values of the counter.

![](./images/gunicorn-preloading.excalidraw.png)


## 2. Query Functions

### 2.1 `increase`

- Given a counter with value `v`, the `increase` is the difference between the value at the current time and the value at an earlier time:

- For example, let's say there is a counter named `my_counter`:
    - At 12:00, the value of `my_counter` is is 10
    - 10 minutes later, the value of `my_counter` is 12; 
    - The increase over 10m is 2. 

- In PromQL this is expressed as `increase(my_counter{}[10m])`

- Mathematically you might express this as `Δv = v[t] - v[t-Δt]` where:
    - `Δv` denotes increase
    - `v[t]` is the value of the counter at time _t_.
    -  `Δt` is the time interval (10 minutes in the above example).
    - `v[t-Δt]` is the value at an interval before t2.

![Prometheus Increase](./images/promql-increase.excalidraw.png)

### 2.2 `rate`

- Rate is the rate of increase of the value per second
- Mathematically you might express this as `Rate = Δv/Δt` where Δt is **always** measured in seconds.
- For example:
    - At 12:00, the value of `my_counter` is is 10
    - 10 minutes later, the value of `my_counter` is is 12; 
    - The rate over 10m is 0.003. 
        - 10 minutes  = 600 seconds
        - Δv = 2
        - Δt = 600
        - Rate = 2/600 = 0.003
- In PromQL this is expressed as `rate(my_counter{}[10m])`

![Prometheus Rate](./images/promql-rate.excalidraw.png)

## 3. Aggregate Functions

### 3.1 `sum by`

- Suppose we wanted the total number of API calls for a particular endpoint e.g. `get_user_profile`.
- As mentioned earlier, if we have 3 replicas of the web service, then we will have 3 copies of the counter.
- To get the total number of API calls for each endpoint, we can group by the endpoint num and then sum the counter:

```
sum(endpoint_requests_total) by (endpoint)
```

![](./images/promql-sum-by.excalidraw.png)

##  4. Flask OpenTelemetry Setup
       
*app.py*
```python
from instrument_app import setup_logging, create_app

# If the application is launched using the gunicorn app:app command,
# then gunicorn will load the app module (app.py) and look for the app variable.
# (Note: the name of the module is on the left of the colon, and the name of the variable is on the right)
# Since the app module is loaded, __name__ will be set to `app` rather than __main__.
if __name__ == "app":
    setup_logging()
    # Gunicorn is looking for this `app` variable. Do not remove or discard this assignment.
    app = create_app()
```

*instrument_app/app.py*
```python
import os
import time
from flask import Flask, request, request_started, request_finished
from opentelemetry import trace, metrics
from opentelemetry._logs import set_logger_provider
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.metrics import MeterProvider
from opentelemetry.sdk.metrics.export import PeriodicExportingMetricReader
from opentelemetry.exporter.otlp.proto.http.metric_exporter import OTLPMetricExporter
from opentelemetry.exporter.otlp.proto.http._log_exporter import OTLPLogExporter
from opentelemetry.sdk._logs import LoggerProvider, LoggingHandler
from opentelemetry.sdk._logs.export import BatchLogRecordProcessor
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.sdk.resources import Resource

meter = metrics.get_meter("flask_app_metrics")

# Create metrics instruments
endpoint_request_counter = meter.create_counter(
    name="endpoint_requests_total",
    description="Total number of requests per endpoint",
    unit="1",
)

http_status_counter = meter.create_counter(
    name="http_status_codes_total",
    description="Total number of HTTP status codes",
    unit="1",
)

http_duration_histogram = meter.create_histogram(
    name="http_request_duration_seconds",
    description="HTTP request duration in seconds",
    unit="s",
)

def setup_logging(testing=False):
    # recreate_defaults() must be called at the beginning of this function, otherwise structlog will not be sent to opentelemetry.
    # recreate_defaults() sends structlogs through python standard logging and Opentelemetry is configured to read from standard logging.
    default_processors = recreate_defaults() or []
    processors = default_processors + [
        structlog.contextvars.merge_contextvars,
        structlog.processors.add_log_level,
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.dict_tracebacks,
        structlog.processors.JSONRenderer(),
    ]
    structlog.configure(processors=processors)

def create_app():
    app = Flask(
        __name__,
        instance_relative_config=True,
    )

    environment = os.environ.get("FLASK_ENV", "development")

    app.config.from_object("config")
    app.config.from_pyfile(f"config-{environment.lower()}.py")

    _setup_instrumentation(app)
    _register_blueprints(app)

    request_started.connect(bind_request_details, app)
    request_finished.connect(bind_request_finished, app)

    return app

def bind_request_details(sender, **extras):
    if request:
        request.start_time = time.time()


def bind_request_finished(sender, response, **extras):
    endpoint = request.endpoint if request.endpoint else request.path
    method = request.method
    status_code = response.status_code

    # Calculate duration
    duration = time.time() - request.start_time

    # Record metrics with appropriate labels/attributes
    endpoint_request_counter.add(1, {"endpoint": endpoint})
    http_status_counter.add(1, {"status_code": str(status_code), "endpoint": endpoint})
    http_duration_histogram.record(duration, {"method": method, "endpoint": endpoint})


def _setup_instrumentation(app):
    # Each gunicorn worker will have its own counter.
    # For the sake of simplicity, we will append the pid to the instance id to distinguish the counter of each worker.
    # https://github.com/open-telemetry/opentelemetry-python/issues/3885
    #
    # In a multi-process Flask application, you could use the process_id as the instance.id to distinguish between counters with the same name.
    # However, If you preload the app, you must set the instance.id after preloading the app.
    # If you set the instance.id before preloading, the instance.id set on the resource will be the same for all processes.
    # If you set the instance.id after preloading, the instance.id set on the resource will be different for each process.
    resource = Resource.create(
        {
            "service.name": os.getenv("OTEL_SERVICE_NAME", "my-api"),
            "service.instance.id": f"{_get_server_id()}:{os.getpid()}",
        }
    )

    trace_provider = TracerProvider(resource=resource)
    trace.set_tracer_provider(trace_provider)

    otlp_trace_exporter = OTLPSpanExporter()
    trace_provider.add_span_processor(BatchSpanProcessor(otlp_trace_exporter))

    metric_reader = PeriodicExportingMetricReader(OTLPMetricExporter())
    metric_provider = MeterProvider(resource=resource, metric_readers=[metric_reader])
    opentelemetry.metrics.set_meter_provider(metric_provider)

    # https://docs.honeycomb.io/send-data/logs/opentelemetry/sdk/python/
    logger_provider = LoggerProvider(resource=resource)
    set_logger_provider(logger_provider)

    otlp_log_exporter = OTLPLogExporter()
    logger_provider.add_log_record_processor(BatchLogRecordProcessor(otlp_log_exporter))

    handler = LoggingHandler(level=logging.INFO)

    root_logger = logging.getLogger()
    root_logger.addHandler(handler)

    FlaskInstrumentor().instrument_app(app)
```