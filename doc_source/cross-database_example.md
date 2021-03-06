# Examples of using a cross\-database query<a name="cross-database_example"></a>

Use the following examples to help learn how to set up a cross\-database query that references an Amazon Redshift database\. 

To start, create databases `db1` and `db2` and users `user1` and `user2` in your Amazon Redshift cluster\. For more information, see [CREATE DATABASE](r_CREATE_DATABASE.md) and [CREATE USER](r_CREATE_USER.md)\.

```
--As user1 on db1
CREATE DATABASE db1;

CREATE DATABASE db2;

CREATE USER user1 PASSWORD 'Redshift01';

CREATE USER user2 PASSWORD 'Redshift01';
```

As `user1` on `db1`, create a table, grant access privileges to `user2`, and insert values into `table1`\. For more information, see [GRANT](r_GRANT.md) and [INSERT](r_INSERT_30.md)\.

```
--As user1 on db1
CREATE TABLE table1 (c1 int, c2 int, c3 int);

GRANT SELECT ON table1 TO user2;

INSERT INTO table1 VALUES (1,2,3),(4,5,6),(7,8,9);
```

As `user2` on `db2`, run a cross\-database query in `db2` using the three\-part notation\. 

```
--As user2 on db2
SELECT * from db1.public.table1 ORDER BY c1;
c1 | c2  | c3
---+-----+----
1  |  2  | 3
4  |  5  | 6
7  |  8  | 9
(3 rows)
```

As `user2` on `db2`, create an external schema and run a cross\-database query in `db2` using the external schema notation\. 

```
--As user2 on db2
CREATE EXTERNAL SCHEMA db1_public_sch
FROM REDSHIFT DATABASE 'db1' SCHEMA 'public';

SELECT * FROM db1_public_sch.table1 ORDER BY c1;

c1  | c2 | c3
----+----+----
1   | 2  | 3
4   | 5  | 6
7   | 8  | 9
(3 rows)
```

To create different views and grant permissions to those views, as `user1` on `db1`, do the following\. 

```
--As user1 on db1
CREATE VIEW regular_view AS SELECT c1 FROM table1;

GRANT SELECT ON regular_view TO user2;


CREATE MATERIALIZED VIEW mat_view AS SELECT c2 FROM table1;

GRANT SELECT ON mat_view TO user2;


CREATE VIEW late_bind_view AS SELECT c3 FROM public.table1 WITH NO SCHEMA BINDING;

GRANT SELECT ON late_bind_view TO user2;
```

As `user2` on `db2`, run the following cross\-database query using the three\-part notation to view the particular view\.

```
--As user2 on db2
SELECT * FROM db1.public.regular_view;
c1
----
1
4
7
(3 rows)

SELECT * FROM db1.public.mat_view;
c2
----
8
5
2
(3 rows)

SELECT * FROM db1.public.late_bind_view;
c3
----
3
6 
9
(3 rows)
```

As `user2` on `db2`, run the following cross\-database query using the external schema notation to query the late\-binding view\.

```
--As user2 on db2
SELECT * FROM db1_public_sch.late_bind_view;
c3
----
3
6
9
(3 rows)
```

As `user2` on `db2`, run the following command using connected tables in a single query\.

```
--As user2 on db2
CREATE TABLE table1 (a int, b int, c int);

INSERT INTO table1 VALUES (1,2,3), (4,5,6), (7,8,9);

SELECT a AS col_1, (db1.public.table1.c2 + b) AS sum_col2, (db1.public.table1.c3 + c) AS sum_col3 FROM db1.public.table1, table1 WHERE db1.public.table1.c1 = a;
col_1 | sum_col2 | sum_col3
------+----------+----------
1     | 4        | 6
4     | 10       | 12
7     | 16       | 18
(3 rows)
```

The following example lists all databases on the cluster\.

```
select database_name, database_owner, database_type 
from svv_redshift_databases 
where database_name in ('db1', 'db2');

 database_name | database_owner | database_type 
---------------+----------------+---------------
 db1           |            100 | local
 db2           |            100 | local
(2 rows)
```

The following example lists all Amazon Redshift schemas of all databases on the cluster\.

```
select database_name, schema_name, schema_owner, schema_type 
from svv_redshift_schemas 
where database_name in ('db1', 'db2');

 database_name |    schema_name     | schema_owner | schema_type 
---------------+--------------------+--------------+-------------
 db1           | pg_catalog         |            1 | local
 db1           | public             |            1 | local
 db1           | information_schema |            1 | local
 db2           | pg_catalog         |            1 | local
 db2           | public             |            1 | local
 db2           | information_schema |            1 | local
(6 rows)
```

The following example lists all Amazon Redshift tables or views of all databases on the cluster\.

```
select database_name, schema_name, table_name, table_type 
from svv_redshift_tables 
where database_name in ('db1', 'db2') and schema_name in ('public');

 database_name | schema_name |     table_name      | table_type 
---------------+-------------+---------------------+------------
 db1           | public      | late_bind_view      | VIEW
 db1           | public      | mat_view            | VIEW
 db1           | public      | mv_tbl__mat_view__0 | TABLE
 db1           | public      | regular_view        | VIEW
 db1           | public      | table1              | TABLE
 db2           | public      | table2              | TABLE
(6 rows)
```

The following example lists all Amazon Redshift and external schemas of all databases on the cluster\.

```
select database_name, schema_name, schema_owner, schema_type 
from svv_all_schemas where database_name in ('db1', 'db2') ;

 database_name |    schema_name     | schema_owner | schema_type 
---------------+--------------------+--------------+-------------
 db1           | pg_catalog         |            1 | local
 db1           | public             |            1 | local
 db1           | information_schema |            1 | local
 db2           | pg_catalog         |            1 | local
 db2           | public             |            1 | local
 db2           | information_schema |            1 | local
 db2           | db1_public_sch     |            1 | external
(7 rows)
```

The following example lists all Amazon Redshift and external tables of all databases on the cluster\.

```
select database_name, schema_name, table_name, table_type 
from svv_all_tables 
where database_name in ('db1', 'db2') and schema_name in ('public');

 database_name | schema_name |     table_name      | table_type 
---------------+-------------+---------------------+------------
 db1           | public      | regular_view        | VIEW
 db1           | public      | mv_tbl__mat_view__0 | TABLE
 db1           | public      | mat_view            | VIEW
 db1           | public      | late_bind_view      | VIEW
 db1           | public      | table1              | TABLE
 db2           | public      | table2              | TABLE
(6 rows)
```