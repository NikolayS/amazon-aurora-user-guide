# synch/cond/innodb/row\_lock\_wait\_cond<a name="ams-waits.row-lock-wait-cond"></a>

The `synch/cond/innodb/row_lock_wait_cond` event occurs when one session has locked a row for an update, and another session tries to update the same row\. For more information, see [InnoDB locking](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html) in the *MySQL Reference*\.



## Supported engine versions<a name="ams-waits.row-lock-wait-cond.versions"></a>

This wait event information is supported for the following engine versions:
+ Aurora MySQL version 2: 2\.10\.3, 2\.11\.0, 2\.11\.1

## Likely causes of increased waits<a name="ams-waits.row-lock-wait-cond.causes"></a>

Multiple data manipulation language \(DML\) statements are accessing the same row or rows simultaneously\.

## Actions<a name="ams-waits.row-lock-wait-cond.actions"></a>

We recommend different actions depending on the other wait events that you see\.

**Topics**
+ [Find and respond to the SQL statements responsible for this wait event](#ams-waits.row-lock-wait-cond.actions.id)
+ [Find and respond to the blocking session](#ams-waits.row-lock-wait-cond.actions.blocker)

### Find and respond to the SQL statements responsible for this wait event<a name="ams-waits.row-lock-wait-cond.actions.id"></a>

Use Performance Insights to identify the SQL statements responsible for this wait event\. Consider the following strategies:
+ If row locks are a persistent problem, consider rewriting the application to use optimistic locking\.
+ Use multirow statements\.
+ Spread the workload over different database objects\. You can do this through partitioning\.
+ Check the value of the `innodb_lock_wait_timeout` parameter\. It controls how long transactions wait before generating a timeout error\.

For a useful overview of troubleshooting using Performance Insights, see the blog post [Analyze Amazon Aurora MySQL Workloads with Performance Insights](https://aws.amazon.com/blogs/database/analyze-amazon-aurora-mysql-workloads-with-performance-insights/)\.

### Find and respond to the blocking session<a name="ams-waits.row-lock-wait-cond.actions.blocker"></a>

Determine whether the blocking session is idle or active\. Also, find out whether the session comes from an application or an active user\.

To identify the session holding the lock, you can run `SHOW ENGINE INNODB STATUS`\. The following example shows sample output\.

```
mysql> SHOW ENGINE INNODB STATUS;

---TRANSACTION 2771110, ACTIVE 112 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 2 lock struct(s), heap size 1136, 1 row lock(s)
MySQL thread id 24, OS thread handle 70369573642160, query id 13271336 172.31.14.179 reinvent Sending data
select id1 from test.t1 where id1=1 for update
------- TRX HAS BEEN WAITING 43 SEC FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 11 page no 3 n bits 0 index GEN_CLUST_INDEX of table test.t1 trx id 2771110 lock_mode X waiting
Record lock, heap no 2 PHYSICAL RECORD: n_fields 5; compact format; info bits 0
```

Or you can use the following query to extract details on current locks\.

```
mysql> SELECT p1.id waiting_thread,
              p1.user waiting_user,
              p1.host waiting_host,
              it1.trx_query waiting_query,        
              ilw.requesting_trx_id waiting_transaction, 
              ilw.blocking_lock_id blocking_lock, 
              il.lock_mode blocking_mode,
              il.lock_type blocking_type,
              ilw.blocking_trx_id blocking_transaction,
              CASE it.trx_state 
                WHEN 'LOCK WAIT' 
                THEN it.trx_state 
                ELSE p.state 
              END blocker_state, 
              il.lock_table locked_table,        
              it.trx_mysql_thread_id blocker_thread, 
              p.user blocker_user, 
              p.host blocker_host 
       FROM information_schema.innodb_lock_waits ilw 
       JOIN information_schema.innodb_locks il 
         ON ilw.blocking_lock_id = il.lock_id 
        AND ilw.blocking_trx_id = il.lock_trx_id
       JOIN information_schema.innodb_trx it 
         ON ilw.blocking_trx_id = it.trx_id
       JOIN information_schema.processlist p 
         ON it.trx_mysql_thread_id = p.id 
       JOIN information_schema.innodb_trx it1 
         ON ilw.requesting_trx_id = it1.trx_id 
       JOIN information_schema.processlist p1 
         ON it1.trx_mysql_thread_id = p1.id\G

*************************** 1. row ***************************
      waiting_thread: 3561959471
        waiting_user: reinvent
        waiting_host: 123.456.789.012:20485
       waiting_query: select id1 from test.t1 where id1=1 for update
 waiting_transaction: 312337314
       blocking_lock: 312337287:261:3:2
       blocking_mode: X
       blocking_type: RECORD
blocking_transaction: 312337287
       blocker_state: User sleep
        locked_table: `test`.`t1`
      blocker_thread: 3561223876
        blocker_user: reinvent
        blocker_host: 123.456.789.012:17746
1 row in set (0.04 sec)
```

When you identify the session, your options include the following:
+ Contact the application owner or the user\.
+ If the blocking session is idle, consider ending the blocking session\. This action might trigger a long rollback\. To learn how to end a session, see [Ending a session or query](mysql-stored-proc-ending.md)\.

For more information about identifying blocking transactions, see [Using InnoDB Transaction and Locking Information](https://dev.mysql.com/doc/refman/5.7/en/innodb-information-schema-examples.html) in the *MySQL Reference Manual*\.