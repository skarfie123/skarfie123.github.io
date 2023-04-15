When performing transactions on the same tables in Postgres from multiple apps, you might face deadlocks.

You might get an error like this one (example from [here](https://stackoverflow.com/questions/31178641/how-to-avert-deadlock-detected-issue-on-postgres-sqlalchemy)):

```python
DBAPIError: (TransactionRollbackError) deadlock detected
DETAIL:  Process 61086 waits for ExclusiveLock on tuple (7217,55) of relation 626383 of database 380717; blocked by process 61094.
Process 61094 waits for ShareLock on transaction 55622134; blocked by process 61088.
Process 61088 waits for ShareLock on transaction 55622141; blocked by process 61086.
HINT:  See server log for query details.
'UPDATE proxies SET date_deleted_utc=%(date_deleted_utc)s WHERE proxies.service_token = %(service_token_1)s' {'date_deleted_utc': datetime.datetime(2015, 7, 2, 12, 57, 23, 2358), 'service_token_1': u'3733a37e-2094-11e5-90b7-0242ac110080'}
```

The first question you might have is: what apps are conflicting? To find out, you can run this query against the database, using the process ids from the error message:
```sql
select pid,application_name from pg_stat_activity where pid in (61086,61094,61088);
```
This will give the `application_name` of each process involved, so you know which codebase(s) to look into. This uses the `pg_stat_activity` which is built into Postgres by default.

This method isn't perfect, because processes only appear in this table if they are **still active**, but in many cases this is good enough.

To prevent deadlocks makes sure all apps that perform transactions on that table, acquire locks in the same order. For example you can sort by a unique `id` and select for update:
```sql
SELECT ... ORDER BY id FOR UPDATE
```