# Managing transactions<a name="stored-procedure-transaction-management"></a>

You can create a stored procedure with default transaction management behavior or nonatomic behavior\. 

## Default mode stored procedure transaction management<a name="stored-procedure-transaction-management-default-mode"></a>

The default transaction mode automatic commit behavior causes each SQL command that runs separately to commit individually\. A call to a stored procedure is treated as a single SQL command\. The SQL statements inside a procedure behave as if they are in a transaction block that implicitly begins when the call starts and ends when the call finishes\. A nested call to another procedure is treated like any other SQL statement and operates within the context of the same transaction as the caller\. For more information about automatic commit behavior, see [Serializable isolation](c_serial_isolation.md)\.

However, suppose that you call a stored procedure from within a user specified transaction block \(defined by BEGIN\.\.\.COMMIT\)\. In this case, all statements in the stored procedure run in the context of the user\-specified transaction\. The procedure doesn't commit implicitly on exit\. The caller controls the procedure commit or rollback\.

If any error is encountered while running a stored procedure, all changes made in the current transaction are rolled back\.

You can use the following transaction control statements in a stored procedure:
+ COMMIT – commits all work done in the current transaction and implicitly begins a new transaction\. For more information, see [COMMIT](r_COMMIT.md)\. 
+ ROLLBACK – rolls back the work done in the current transaction and implicitly begins a new transaction\. For more information, see [ROLLBACK](r_ROLLBACK.md)\. 

TRUNCATE is another statement that you can issue from within a stored procedure and influences transaction management\. In Amazon Redshift, TRUNCATE issues a commit implicitly\. This behavior stays the same in the context of stored procedures\. When a TRUNCATE statement is issued from within a stored procedure, it commits the current transaction and begins a new one\. For more information, see [TRUNCATE](r_TRUNCATE.md)\. 

All statements that follow a COMMIT, ROLLBACK, or TRUNCATE statement run in the context of a new transaction\. They do so until a COMMIT, ROLLBACK, or TRUNCATE statement is encountered or the stored procedure exits\.

When you use a COMMIT, ROLLBACK, or TRUNCATE statement from within a stored procedure, the following constraints apply:
+ If the stored procedure is called from within a transaction block, it can't issue a COMMIT, ROLLBACK, or TRUNCATE statement\. This restriction applies within the stored procedure's own body and within any nested procedure call\.
+ If the stored procedure is created with `SET config` options, it can't issue a COMMIT, ROLLBACK, or TRUNCATE statement\. This restriction applies within the stored procedure's own body and within any nested procedure call\. 
+ Any cursor that is open \(explicitly or implicitly\) is closed automatically when a COMMIT, ROLLBACK, or TRUNCATE statement is processed\. For constraints on explicit and implicit cursors, see [Limits and differences for stored procedure support](stored-procedure-constraints.md)\.

Additionally, you can't run COMMIT or ROLLBACK using dynamic SQL\. However, you can run TRUNCATE using dynamic SQL\. For more information, see [Dynamic SQL](c_PLpgSQL-statements.md#r_PLpgSQL-dynamic-sql)\. 

When working with stored procedures, consider that the BEGIN and END statements in PL/pgSQL are only for grouping\. They don't start or end a transaction\. For more information, see [Block](c_PLpgSQL-structure.md#r_PLpgSQL-block)\. 

The following example demonstrates transaction behavior when calling a stored procedure from within an explicit transaction block\. The two insert statements issued from outside the stored procedure and the one from within it are all part of the same transaction \(3382\)\. The transaction is committed when the user issues the explicit commit\.

```
CREATE OR REPLACE PROCEDURE sp_insert_table_a(a int) LANGUAGE plpgsql
AS $$
BEGIN
  INSERT INTO test_table_a values (a);
END;
$$;

Begin;
  insert into test_table_a values (1);
  Call sp_insert_table_a(2);
  insert into test_table_a values (3);
Commit; 

select userid, xid, pid, type, trim(text) as stmt_text 
from svl_statementtext where pid = pg_backend_pid() order by xid , starttime , sequence;

 userid | xid  | pid |  type   |               stmt_text
--------+------+-----+---------+----------------------------------------
    103 | 3382 | 599 | UTILITY | Begin;
    103 | 3382 | 599 | QUERY   | insert into test_table_a values (1);
    103 | 3382 | 599 | UTILITY | Call sp_insert_table_a(2);
    103 | 3382 | 599 | QUERY   | INSERT INTO test_table_a values ( $1 )
    103 | 3382 | 599 | QUERY   | insert into test_table_a values (3);
    103 | 3382 | 599 | UTILITY | COMMIT
```

In contrast, take an example when the same statements are issued from outside of an explicit transaction block and the session has autocommit set to ON\. In this case, each statement runs in its own transaction\.

```
insert into test_table_a values (1);
Call sp_insert_table_a(2);
insert into test_table_a values (3);

select userid, xid, pid, type, trim(text) as stmt_text 
from svl_statementtext where pid = pg_backend_pid() order by xid , starttime , sequence;

 userid | xid  | pid |  type   |                                                                    stmt_text
--------+------+-----+---------+-------------------------------------------------------------------------------------------------------------------------------------------------
    103 | 3388 | 599 | QUERY   | insert into test_table_a values (1);
    103 | 3388 | 599 | UTILITY | COMMIT
    103 | 3389 | 599 | UTILITY | Call sp_insert_table_a(2);
    103 | 3389 | 599 | QUERY   | INSERT INTO test_table_a values ( $1 )
    103 | 3389 | 599 | UTILITY | COMMIT
    103 | 3390 | 599 | QUERY   | insert into test_table_a values (3);
    103 | 3390 | 599 | UTILITY | COMMIT
```

The following example issues a TRUNCATE statement after inserting into `test_table_a`\. The TRUNCATE statement issues an implicit commit that commits the current transaction \(3335\) and starts a new one \(3336\)\. The new transaction is committed when the procedure exits\.

```
CREATE OR REPLACE PROCEDURE sp_truncate_proc(a int, b int) LANGUAGE plpgsql
AS $$
BEGIN
  INSERT INTO test_table_a values (a);
  TRUNCATE test_table_b;
  INSERT INTO test_table_b values (b);
END;
$$;

Call sp_truncate_proc(1,2);

select userid, xid, pid, type, trim(text) as stmt_text 
from svl_statementtext where pid = pg_backend_pid() order by xid , starttime , sequence;

 userid | xid  |  pid  |  type   |                                                                                             stmt_text
--------+------+-------+---------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    103 | 3335 | 23636 | UTILITY | Call sp_truncate_proc(1,2);
    103 | 3335 | 23636 | QUERY   | INSERT INTO test_table_a values ( $1 )
    103 | 3335 | 23636 | UTILITY | TRUNCATE test_table_b
    103 | 3335 | 23636 | UTILITY | COMMIT
    103 | 3336 | 23636 | QUERY   | INSERT INTO test_table_b values ( $1 )
    103 | 3336 | 23636 | UTILITY | COMMIT
```

The following example issues a TRUNCATE from a nested call\. The TRUNCATE commits all work done so far in the outer and inner procedures in a transaction \(3344\)\. It starts a new transaction \(3345\)\. The new transaction is committed when the outer procedure exits\.

```
CREATE OR REPLACE PROCEDURE sp_inner(c int, d int) LANGUAGE plpgsql
AS $$
BEGIN
  INSERT INTO inner_table values (c);
  TRUNCATE outer_table;
  INSERT INTO inner_table values (d);
END;
$$;

CREATE OR REPLACE PROCEDURE sp_outer(a int, b int, c int, d int) LANGUAGE plpgsql
AS $$
BEGIN
  INSERT INTO outer_table values (a);
  Call sp_inner(c, d);
  INSERT INTO outer_table values (b);
END;
$$;

Call sp_outer(1, 2, 3, 4);

select userid, xid, pid, type, trim(text) as stmt_text 
from svl_statementtext where pid = pg_backend_pid() order by xid , starttime , sequence;

 userid | xid  |  pid  |  type   |                                                                                              stmt_text
--------+------+-------+---------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    103 | 3344 | 23636 | UTILITY | Call sp_outer(1, 2, 3, 4);
    103 | 3344 | 23636 | QUERY   | INSERT INTO outer_table values ( $1 )
    103 | 3344 | 23636 | UTILITY | CALL sp_inner( $1 , $2 )
    103 | 3344 | 23636 | QUERY   | INSERT INTO inner_table values ( $1 )
    103 | 3344 | 23636 | UTILITY | TRUNCATE outer_table
    103 | 3344 | 23636 | UTILITY | COMMIT
    103 | 3345 | 23636 | QUERY   | INSERT INTO inner_table values ( $1 )
    103 | 3345 | 23636 | QUERY   | INSERT INTO outer_table values ( $1 )
    103 | 3345 | 23636 | UTILITY | COMMIT
```

The following example shows that cursor `cur1` was closed when the TRUNCATE statement committed\.

```
CREATE OR REPLACE PROCEDURE sp_open_cursor_truncate()
LANGUAGE plpgsql
AS $$
DECLARE
  rec RECORD;
  cur1 cursor for select * from test_table_a order by 1;
BEGIN
  open cur1;
  TRUNCATE table test_table_b;
  Loop
    fetch cur1 into rec;
    raise info '%', rec.c1;
    exit when not found;
  End Loop;
END
$$;

call sp_open_cursor_truncate();
ERROR: cursor "cur1" does not exist
CONTEXT: PL/pgSQL function "sp_open_cursor_truncate" line 8 at fetch
```

The following example issues a TRUNCATE statement and can't be called from within an explicit transaction block\.

```
CREATE OR REPLACE PROCEDURE sp_truncate_atomic() LANGUAGE plpgsql
AS $$
BEGIN
  TRUNCATE test_table_b;
END;
$$;

Begin;
  Call sp_truncate_atomic();
ERROR: TRUNCATE cannot be invoked from a procedure that is executing in an atomic context.
HINT: Try calling the procedure as a top-level call i.e. not from within an explicit transaction block. 
Or, if this procedure (or one of its ancestors in the call chain) was created with SET config options, recreate the procedure without them.
CONTEXT: SQL statement "TRUNCATE test_table_b"
PL/pgSQL function "sp_truncate_atomic" line 2 at SQL statement
```

The following example shows that a user who is not a superuser or the owner of a table can issue a TRUNCATE statement on the table\. The user does this using a `Security Definer` stored procedure\. The example shows the following actions: 
+ The user1 creates table `test_tbl`\. 
+ The user1 creates stored procedure `sp_truncate_test_tbl`\. 
+ The user1 grants `EXECUTE` privilege on the stored procedure to user2\. 
+ The user2 runs the stored procedure to truncate table `test_tbl`\. The example shows the row count before and after the `TRUNCATE` command\. 

```
set session_authorization to user1;
create table test_tbl(id int, name varchar(20));
insert into test_tbl values (1,'john'), (2, 'mary');
CREATE OR REPLACE PROCEDURE sp_truncate_test_tbl() LANGUAGE plpgsql
AS $$
DECLARE
  tbl_rows int;
BEGIN
  select count(*) into tbl_rows from test_tbl;
  RAISE INFO 'RowCount before Truncate: %', tbl_rows;
  TRUNCATE test_tbl;
  select count(*) into tbl_rows from test_tbl;
  RAISE INFO 'RowCount after Truncate: %', tbl_rows;
END;
$$ SECURITY DEFINER;
grant execute on procedure sp_truncate_test_tbl() to user2;
reset session_authorization;


set session_authorization to user2;
call sp_truncate_test_tbl();
INFO:  RowCount before Truncate: 2
INFO:  RowCount after Truncate: 0
CALL
reset session_authorization;
```

The following example issues COMMIT twice\. The first COMMIT commits all work done in transaction 10363 and implicitly starts transaction 10364\. Transaction 10364 is committed by the second COMMIT statement\. 

```
CREATE OR REPLACE PROCEDURE sp_commit(a int, b int) LANGUAGE plpgsql
AS $$
BEGIN
  INSERT INTO test_table values (a);
  COMMIT;
  INSERT INTO test_table values (b);
  COMMIT;
END;
$$;

call sp_commit(1,2);

select userid, xid, pid, type, trim(text) as stmt_text
from svl_statementtext where pid = pg_backend_pid() order by xid , starttime , sequence;
 userid |  xid  | pid  |  type   |                                                                                    stmt_text
--------+-------+------+---------+-----------------------------------------------------------------------------------------------------------------
    100 | 10363 | 3089 | UTILITY | call sp_commit(1,2);
    100 | 10363 | 3089 | QUERY   | INSERT INTO test_table values ( $1 )
    100 | 10363 | 3089 | UTILITY | COMMIT
    100 | 10364 | 3089 | QUERY   | INSERT INTO test_table values ( $1 )
    100 | 10364 | 3089 | UTILITY | COMMIT
```

The following example issues a ROLLBACK statement if `sum_vals` is greater than 2\. The first ROLLBACK statement rolls back all the work done in transaction 10377 and starts a new transaction 10378\. Transaction 10378 is committed when the procedure exits\. 

```
CREATE OR REPLACE PROCEDURE sp_rollback(a int, b int) LANGUAGE plpgsql
AS $$
DECLARE
  sum_vals int;
BEGIN
  INSERT INTO test_table values (a);
  SELECT sum(c1) into sum_vals from test_table;
  IF sum_vals > 2 THEN
    ROLLBACK;
  END IF;
  
  INSERT INTO test_table values (b);
END;
$$;

call sp_rollback(1, 2);

select userid, xid, pid, type, trim(text) as stmt_text
from svl_statementtext where pid = pg_backend_pid() order by xid , starttime , sequence;

userid |  xid  | pid  |  type   |                                                                                    stmt_text
--------+-------+------+---------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    100 | 10377 | 3089 | UTILITY | call sp_rollback(1, 2);
    100 | 10377 | 3089 | QUERY   | INSERT INTO test_table values ( $1 )
    100 | 10377 | 3089 | QUERY   | SELECT sum(c1) from test_table
    100 | 10377 | 3089 | QUERY   | Undoing 1 transactions on table 133646 with current xid 10377 : 10377
    100 | 10378 | 3089 | QUERY   | INSERT INTO test_table values ( $1 )
    100 | 10378 | 3089 | UTILITY | COMMIT
```

## Nonatomic mode stored procedure transaction management<a name="stored-procedure-transaction-management-nonatomic-mode"></a>

A stored procedure created in NONATOMIC mode has different transaction control behavior from a procedure created in default mode\. Similar to the automatic commit behavior of SQL commands outside stored procedures, each SQL statement inside a NONATOMIC procedure runs in its own transaction and commits automatically\. If a user begins an explicit transaction block within a NONATOMIC stored procedure, then the SQL statements within the block do not automatically commit\. The transaction block controls commit or rollback of statements within it\. 

In NONATOMIC stored procedures, you can open an explicit transaction block inside the procedure using the START TRANSACTION statement\. However, if there is already an open transaction block, this statement will do nothing because Amazon Redshift does not support sub transactions\. The previous transaction continues\.

When you work with cursor FOR loops inside a NONATOMIC procedure, make sure you open an explicit transaction block before iterating through the results of a query\. Otherwise, the cursor is closed when the SQL statement inside the loop is automatically committed\.

Some of the considerations when using NONATOMIC mode behavior are as follows:
+ Each SQL statement inside the stored procedure is automatically committed if there is no open transaction block, and the session has autocommit set to ON\.
+ You can issue a COMMIT/ROLLBACK/TRUNCATE statement to end the transaction if the stored procedure is called from within a transaction block\. This is not possible in default mode\.
+ You can issue a START TRANSACTION statement to begin a transaction block inside the stored procedure\.

The following examples demonstrate transaction behavior when working with NONATOMIC stored procedures\. The session for all the following examples has autocommit set to ON\.

In the following example, a NONATOMIC stored procedure has two INSERT statements\. When the procedure is called outside of a transaction block, every INSERT statement within the procedure automatically commits\. 

```
CREATE TABLE test_table_a(v int); 
CREATE TABLE test_table_b(v int); 

CREATE OR REPLACE PROCEDURE sp_nonatomic_insert_table_a(a int, b int) NONATOMIC AS
$$
BEGIN
    INSERT INTO test_table_a values (a);
    INSERT INTO test_table_b values (b);
END;
$$ 
LANGUAGE plpgsql;

Call sp_nonatomic_insert_table_a(1,2);

Select userid, xid, pid, type, trim(text) as stmt_text 
from svl_statementtext where pid = pg_backend_pid() order by xid , starttime , sequence;

 userid | xid  |    pid     |  type   |               stmt_text                
--------+------+------------+---------+----------------------------------------
      1 | 1792 | 1073807554 | UTILITY | Call sp_nonatomic_insert_table_a(1,2);
      1 | 1792 | 1073807554 | QUERY   | INSERT INTO test_table_a values ( $1 )
      1 | 1792 | 1073807554 | UTILITY | COMMIT
      1 | 1793 | 1073807554 | QUERY   | INSERT INTO test_table_b values ( $1 )
      1 | 1793 | 1073807554 | UTILITY | COMMIT
(5 rows)
```

However, when the procedure is called from within a BEGIN\.\.COMMIT block, all the statements are part of the same transaction \(xid=1799\)\. 

```
Begin;
  INSERT INTO test_table_a values (10);
  Call sp_nonatomic_insert_table_a(20,30);
  INSERT INTO test_table_b values (40);
Commit; 

Select userid, xid, pid, type, trim(text) as stmt_text 
from svl_statementtext where pid = pg_backend_pid() order by xid , starttime , sequence;

 userid | xid  |    pid     |  type   |                stmt_text                 
--------+------+------------+---------+------------------------------------------
      1 | 1799 | 1073914035 | UTILITY | Begin;
      1 | 1799 | 1073914035 | QUERY   | INSERT INTO test_table_a values (10);
      1 | 1799 | 1073914035 | UTILITY | Call sp_nonatomic_insert_table_a(20,30);
      1 | 1799 | 1073914035 | QUERY   | INSERT INTO test_table_a values ( $1 )
      1 | 1799 | 1073914035 | QUERY   | INSERT INTO test_table_b values ( $1 )
      1 | 1799 | 1073914035 | QUERY   | INSERT INTO test_table_b values (40);
      1 | 1799 | 1073914035 | UTILITY | COMMIT
(7 rows)
```

In this example, two INSERT statements are between START TRANSACTION\.\.\.COMMIT\. When the procedure is called outside of a transaction block, the two INSERT statements are in the same transaction \(xid=1866\)\. 

```
CREATE OR REPLACE PROCEDURE sp_nonatomic_txn_block(a int, b int) NONATOMIC AS
$$
BEGIN
    START TRANSACTION;
    INSERT INTO test_table_a values (a);
    INSERT INTO test_table_b values (b);
    COMMIT;
END;
$$ 
LANGUAGE plpgsql;

Call sp_nonatomic_txn_block(1,2);

Select userid, xid, pid, type, trim(text) as stmt_text 
from svl_statementtext where pid = pg_backend_pid() order by xid , starttime , sequence;

 userid | xid  |    pid     |  type   |               stmt_text                
--------+------+------------+---------+----------------------------------------
      1 | 1865 | 1073823998 | UTILITY | Call sp_nonatomic_txn_block(1,2);
      1 | 1866 | 1073823998 | QUERY   | INSERT INTO test_table_a values ( $1 )
      1 | 1866 | 1073823998 | QUERY   | INSERT INTO test_table_b values ( $1 )
      1 | 1866 | 1073823998 | UTILITY | COMMIT
(4 rows)
```

When the procedure is called from within a BEGIN\.\.\.COMMIT block, the START TRANSACTION inside the procedure does nothing because there is already an open transaction\. The COMMIT inside the procedure commits the current transaction \(xid=1876\) and starts a new one\.

```
Begin;
  INSERT INTO test_table_a values (10);
  Call sp_nonatomic_txn_block(20,30);
  INSERT INTO test_table_b values (40);
Commit; 

Select userid, xid, pid, type, trim(text) as stmt_text 
from svl_statementtext where pid = pg_backend_pid() order by xid , starttime , sequence;

 userid | xid  |    pid     |  type   |               stmt_text                
--------+------+------------+---------+----------------------------------------
      1 | 1876 | 1073832133 | UTILITY | Begin;
      1 | 1876 | 1073832133 | QUERY   | INSERT INTO test_table_a values (10);
      1 | 1876 | 1073832133 | UTILITY | Call sp_nonatomic_txn_block(20,30);
      1 | 1876 | 1073832133 | QUERY   | INSERT INTO test_table_a values ( $1 )
      1 | 1876 | 1073832133 | QUERY   | INSERT INTO test_table_b values ( $1 )
      1 | 1876 | 1073832133 | UTILITY | COMMIT
      1 | 1878 | 1073832133 | QUERY   | INSERT INTO test_table_b values (40);
      1 | 1878 | 1073832133 | UTILITY | COMMIT
(8 rows)
```

This example shows how to work with cursor loops\. Table test\_table\_a has three values\. The objective is to iterate through the three values and insert them into table test\_table\_b\. If a NONATOMIC stored procedure is created in the following way, it will throw the error cursor "cur1" does not exist after executing INSERT statement in the first loop\. This is because the auto commit of the INSERT closes the open cursor\.

```
insert into test_table_a values (1), (2), (3);

CREATE OR REPLACE PROCEDURE sp_nonatomic_cursor() NONATOMIC
LANGUAGE plpgsql
AS $$
DECLARE
  rec RECORD;
  cur1 cursor for select * from test_table_a order by 1;
BEGIN
  open cur1;
  Loop
    fetch cur1 into rec;
    exit when not found;
    raise info '%', rec.v;
    insert into test_table_b values (rec.v);
  End Loop;
END
$$;

CALL sp_nonatomic_cursor();

INFO:  1
ERROR:  cursor "cur1" does not exist
CONTEXT:  PL/pgSQL function "sp_nonatomic_cursor" line 7 at fetch
```

To make the cursor loop work, put it between START TRANSACTION\.\.\.COMMIT\.

```
insert into test_table_a values (1), (2), (3);

CREATE OR REPLACE PROCEDURE sp_nonatomic_cursor() NONATOMIC
LANGUAGE plpgsql
AS $$
DECLARE
  rec RECORD;
  cur1 cursor for select * from test_table_a order by 1;
BEGIN
  START TRANSACTION;
  open cur1;
  Loop
    fetch cur1 into rec;
    exit when not found;
    raise info '%', rec.v;
    insert into test_table_b values (rec.v);
  End Loop;
  COMMIT;
END
$$;

CALL sp_nonatomic_cursor();

INFO:  1
INFO:  2
INFO:  3
CALL
```