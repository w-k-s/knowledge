# Consider Poor Man's Event Sourcing

## Case Study

### Goal

Recently I worked on a feature where, at the end of a day, a CSV  report needed to be generated every midnight; this report had to be archived on S3 and uploaded to the SFTP server of a regulatory authority.

### Requirements

The responsibility of generating the report fell on a single service within a micro-service architecture. There were multiple pods of this service within the Kubernetes cluster, but only one of them was to  generate the report. Furthermore, in case the reporting process was interrupted (e.g. the reporting pod is deleted), the pod that takes over should resume the process from where the terminated pod left off. 

### Implementation

For the first requirement, **Ensure that only one pod executes the report generation process at a time**, we had an `end_of_day_reporting` table. Each of the pods would try to acquire a lock on the table and the first pod that managed to do so would become the pod that generated the report:

```sql
LOCK TABLE end_of_day_reporting IN ACCESS EXCLUSIVE MODE NOWAIT;
```

Initially, we had designed the schema of the `end_of_day_reporting` to store the latest status of the reporting process:

```sql
CREATE TYPE end_of_day_report_status AS ENUM (
	'STARTED', 
	'ROWS_CREATED', 
	'REPORT_ARCHIVED',
	'REPORT_SUBMITTED'
);

CREATE TABLE end_of_day_reporting(
	process_id BIGSERIAL PRIMARY KEY,
	date DATE NOT NULL,
	status end_of_day_report_status NOT NULL,
	created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
	modified_at TIMESTAMP WITH TIME ZONE
);
```
However, after a review with the team, we decided to take a leaf from event-sourcing and add a row for each event:

```sql
CREATE TYPE end_of_day_report_status AS ENUM (
	'STARTED', 
	'ROWS_CREATED',
	'ROWS_CREATION_FAILURE',
	'REPORT_ARCHIVED',
	'REPORT_ARCHIVING_FAILURE',
	'REPORT_SUBMITTED',
	'REPORT_SUBMISSION_FAILURE'
);

CREATE TABLE end_of_day_reporting(
	process_id BIGSERIAL PRIMARY KEY,
	date DATE NOT NULL
);

CREATE TABLE end_of_day_reporting_event(
	event_id BIGSERIAL PRIMARY KEY
	process_id BIGSERIAL REFERENCES end_of_day_reporting(process_id),
	status end_of_day_report_status NOT NULL,
	failure_details VARCHAR(255),
	created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```
This proved to be very advantageous, for a number of reasons:
1. **We could query if a step had already been completed.** This ensured that no step was repeated twice. For example, before starting the row creation step, we queried to check if the step had already been completed. This prevented duplicate rows from being outputted in the final CSV file.
2. **We could query if a prerequisite steps had completed**. For example, the archiving step had a dependency on the row creation step so we could run a query to make sure that the previous step had been successful.
3. **We could keep track of how long each step took and when exactly it completed**. The more users that used our application, the larger the report would be and the report _had_ to be submitted to the regulatory authority before midnight. Submission after midnight would have resulted in a lot of time-consuming administrative work. 
By storing the time at which each event completed, we could monitor how long each step took (for any single report) and proactively take action if performance was becoming a problem.

The lesson that I learnt from this experience is that while building a full-fledged event-sourced application can be quite complicated (due to the steep learning curve and the infrastructure required to make it work), a simplified version of it can be quite beneficial in building a feature in a robust, idempotent and monitorable way.