# DECISIONS.md

1. **Indexes.**
On `employees`: unique `{"emp_code": 1}` guarantees employee uniqueness; `{"department": 1, "joined_on": 1}` accelerates department aggregations.
On `attendance_logs`: unique `{"emp_code": 1, "date": 1}` enforces one record per employee/day and resolves the punch-in race; `{"emp_code": 1, "punch_in": -1}` backs open-record lookup on punch-out; `{"date": -1, "emp_code": 1}` supports paginated listing; and `{"date": 1, "late_minutes": -1}` serves the leaderboard. I considered and rejected indexing `history` because indexing large, growing subdocuments adds heavy write penalty to audit trails that are only appended to and never queried by key.

2. **Punch-in race.**
Two identical requests validate the employee, compute the date, and concurrently invoke `insert_one`. At the database engine level, the unique index on `{"emp_code": 1, "date": 1}` ensures that exactly one write acquires the index slot and succeeds (yielding 201). The second request encounters a `DuplicateKeyError`, which our application catches and translates into HTTP 409.

3. **Ties.**
The aggregation pipeline uses MongoDB's `$rank` window function for standard competition ranking (1, 2, 2, 4) and filters `rank <= limit`. If two employees tie at the cutoff, both share the qualifying rank and both are included in the response (which may return more than `limit` rows), ordered deterministically by `total_late_minutes` descending, then `emp_code` ascending.

4. **Headcount.**
The summary aggregation starts from the `employees` collection (filtering `joined_on <= end_of_month`), groups by `department` to compute total `headcount` via `{"$sum": 1}`, and performs a `$lookup` to join records from `attendance_logs`. Employees with zero records still count toward department headcount.

5. **One thing you would change for 100x data.**
At 100x scale (~10 million attendance records), I would shard `attendance_logs` using a compound shard key `{"emp_code": "hashed", "date": 1}` to distribute write load, and maintain incremental daily/monthly pre-aggregated rollups on punch-out rather than executing full month range joins at read time.
