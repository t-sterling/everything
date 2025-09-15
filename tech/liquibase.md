In an “aggregator” (master) changelog that uses `<include>` (or `<includeAll>`), **every executed changeSet—no matter which included file it lives in—is recorded in the same `DATABASECHANGELOG` table** for that Liquibase run.

Key details:

* **Where:** In the target database’s `DATABASECHANGELOG` table (and `DATABASECHANGELOGLOCK`) located in Liquibase’s schema.

  * By default it goes into the connection’s default schema.
  * You can override where Liquibase’s own tables live with `--liquibaseSchemaName` (and their names with `--databaseChangeLogTableName` / `--databaseChangeLogLockTableName`).

* **How entries are keyed:** Each row is keyed by **(ID, AUTHOR, FILENAME)**.

  * For included files, the **FILENAME** column is the path of the included changelog file (or its `logicalFilePath` if you set one).
  * The master file itself only gets rows if it contains its own changeSets.

* **What you’ll see:** run a quick peek:

  ```sql
  SELECT id, author, filename, dateexecuted, exectype
  FROM   <liquibase_schema>.databasechangelog
  ORDER  BY orderexecuted;
  ```

  You’ll see entries like:

  ```
  JIRA-123 | tim | schemas/sales/sales-master.sql   | 2025-09-15 ... | EXECUTED
  JIRA-207 | tim | schemas/billing/billing-master.sql| 2025-09-15 ... | EXECUTED
  ```

* **Multiple schemas note:** Even if your included files create objects in different DB schemas, **all their changeSets are still tracked in that single `DATABASECHANGELOG`** for the run. If you want **separate per-schema histories**, run Liquibase separately per schema and/or point each run to its own changelog table/schema via the flags above.
