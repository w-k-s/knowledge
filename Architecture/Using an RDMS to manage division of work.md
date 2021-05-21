# Using an RDMS to manage division of work between threads or nodes

A transactional database can be used to control the division of work between two threads (assuming that the workload processing state is represented in the database). A transactional database can be used to enforce that either:

1. All work is done by a single thread
2. Work is divided evenly between multiple threads.

## All work is done by a single thread

This is done by locking all the workload rows and updating their status to done once they have been processed. Only one thread will be able to acquire the locked rows so by the time the second thread acquires the lock, the status of the workload rows will be set to DONE so there will be no work for the second thread to do.

The database queries that makes this possible are:

```sql
BEGIN TRANSACTION
SELECT id,user_json FROM source WHERE is_processed = FALSE ORDER BY id FOR UPDATE;
UPDATE source SET is_processed = TRUE WHERE id = ?
COMMIT
```

| Line                                                 | Role                                                                                        |
|------------------------------------------------------|---------------------------------------------------------------------------------------------|
| `BEGIN TRANSACTION`                                  | Begins a transaction                                                                        |
| `SELECT .. WHERE is_processed = FALSE .. FOR UPDATE` | Selects the workload rows and locks them until the end of the transaction                   |
| `UPDATE SET is_processed = TRUE`                     | Updates the status of the workload rows so that the second thread does not repeat the work. |
| `COMMIT`                                             | Commits the transaction and releases the lock                                               |

## Work is divided evenly between multiple threads.

This is achieved using the following queries:

```sql
BEGIN TRANSACTION
SELECT user_json FROM source WHERE is_processed = FALSE ORDER BY id FOR UPDATE SKIP LOCKED LIMIT ?
COMMIT
```

| Line                                                                               | Role                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
|------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `BEGIN TRANSACTION`                                                                | Begins a transaction                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `SELECT .. ORDER BY .. WHERE is_processed = FALSE FOR UPDATE SKIP LOCKED LIMIT ..` | Each part of this query has a role:<br><br>- `ORDER BY` is important because it ensures all threads view the workload rows in the same order.<br>- `FOR UPDATE .. LIMIT ..` locks a certain number of the selected rows. The first thread will work on these rows.<br><br>- `SKIP LOCKED` instructs the database to ignore locked rows. This allows a second thread to pick up rows that aren't being worked on by the first thread.<br><br>-`WHERE is_processed = FALSE` is a second line of defense to make sure the second thread does not pick up work that was already done.<br> |
| `UPDATE SET is_processed = TRUE`                                                   | Updates the status of the workload rows so that the second thread does not repeat the work.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `COMMIT`                                                                           | Commits the transaction and releases the lock                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |

## Practical Applications

In this repository, we've used the simple example of using a transactional database to control the division of work between one or more threads, but the same queries can be used to divide work between one or more nodes in a kubernetes cluster.


