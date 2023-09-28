# Questions that Architects / Backend Engineers should always ask

## Miscellanious

**1. What if there are multiple instances of my service/application running?**

_Example 1_:

- Two instances consuming from same queue [(Competing Consumer)](https://docs.microsoft.com/en-us/azure/architecture/patterns/competing-consumers)
- This might result in messages being processed out of order
- e.g. if a customer creates an order and immediately cancels it, two messages are added to a queue
- instance A picks up create order, instance B picks up cancel order at the same time
- instance B immediately dismisses cancel order because order does not exist yet?
- instance A still creates order after a few seconds - not what the custoemr intended
- **SOLUTION**: queue sharding ensures message grouped by an id (e.g. customer id) is always handled by the same instance

_Example 2_:

- Load Balancer balancing between instance A and B
- Applciation saves data on instance's physical memory
- Data might be saved when load balancer routes to instance A.
- That same data will not be available when load balancer routes to instance B.
- **SOLUTION**: Use sticky sessions or save data in cloud volume.

**2. What if this endpoint is called twice simultaneously?**

**3. What if X is down?**

- Circuit breaker pattern
- Redundancy

## Messaging

**1. What if the message is never received?**

**2. What if duplicated messages are received?**

**3. What if messages are received out of order?**

## Integrations

**1. How do the two systems communicate with each other**

- What format is the data in: JSON, XML, CSV?
- How is the data received: Push, Polling?
- How is the data sent: HTTP, SOAP?
- How often is data sent/received: Batch, Real-Time?
- Which data fields are optional and which fields are mandatory?
- For date fields: Which timezone is being used?
- Complete list of error codes
- For asynchronous communication: Is there an error queue?

**2. If a synchronous request to a external service times-out, can we determine the state on the 3rd party system**

- External service in this context refers to a service that is outside our microservice cluster.

**3. What is the availability of the external system**

- External service in this context refers to a service that is outside our microservice cluster.

**4. List possible error scenarios? Draw sequence diagrams**
