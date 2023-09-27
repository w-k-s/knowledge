# System Design - End of Day Report

Our team was tasked with building a feature that would generate a CSV report everyday at midnight and upload it to the SFTP Server of a regulatory authority. On this page I've documented some of aspects of the design that I found noteworthy.

---

## Requirements

### Functional Requirements

| **Requirement**         | **Description**                                                        |
|-------------------------|------------------------------------------------------------------------|
| **Generate CSV Report** | Extract rows from a table and use them to generate a headed CSV file.  |
| **Upload CSV Report**   | Upload the CSV file to the SFTP server of regulatory authority         |
| **Archive CSV Report**  | If the CSV file is uploaded to SFTP, archive a copy to our S3 storage. |

### Non-Functional Requirements

I won't cover all the non-functional requirements; just the ones relevant to the article.

| **Requirement**    | **Description**                                                                                                                                                                     |
|--------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Resumable**      | In case the process is interrupted, the system should continue from where it left off once the process resumed.                                                                     |
| **No Duplication** | Ensure that multiple copies of the report are not uploaded/archived.                                                                                                                |
| **Monitoring**     | This was a quick-and-easy solution for Friends and Family. We wanted to make sure we could add monitoring and alerts to detect when we needed to migrate to a more scalable design. |

----

## Components

### Distributed Locks

- There were multiple instances of the reporting microservice. In order to ensure that duplicate reports are not generated, we needed to ensure that only one of the services generated the report and upload/archived it.

- To do this, we implemented Distributed Locks using [ShedLock](https://github.com/lukas-krecan/ShedLock). 

- Each of the instances would try to acquire a lock on a table and the first onstance that managed to do so would be reponsible for generating the report.

- I believe the same can be implemented without ShedLock using this query:

	```sql
	LOCK TABLE end_of_day_reporting IN ACCESS EXCLUSIVE MODE NOWAIT;
	```

---

## Database Schema

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
However, after a review with the team, we decided add a row for each event:

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
---

# Evaluation

| **Requirement**    | **Evaluation**                                                                                                                                                                                                                  |
|--------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Resumable**      | - By storing the status of each step of the process, we could determine if a step had already been completed.<br/> If it hadn't we could continue from where we left off.                                                       |
| **No Duplication** | - By using distributed locks, we were able to ensure that only one instance participated in the EOD process. However, I felt this was an oversimplistic design. Hopefully, I'll find time to describe a more scalable approach. |
| **Monitoring**     | - We stored the start/end time of each step. We could add alerts to detect if generation took too long and start implementing the more scalable solution.                                                                       |