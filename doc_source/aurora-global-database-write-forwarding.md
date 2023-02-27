# Using write forwarding in an Amazon Aurora global database<a name="aurora-global-database-write-forwarding"></a>

You can reduce the number of endpoints that you need to manage for applications running on your Aurora global database, by using *write forwarding*\. This feature of Aurora MySQL lets secondary clusters in an Aurora global database forward SQL statements that perform write operations to the primary cluster\. The primary cluster updates the source and then propagates resulting changes back to all secondary AWS Regions\. 

The write forwarding configuration saves you from implementing your own mechanism to send write operations from a secondary AWS Region to the primary Region\. Aurora handles the cross\-Region networking setup\. Aurora also transmits all necessary session and transactional context for each statement\. The data is always changed first on the primary cluster and then replicated to the secondary clusters in the Aurora global database\. This way, the primary cluster is the source of truth and always has an up\-to\-date copy of all your data\. 

**Note**  
 Write forwarding requires Aurora MySQL version 2\.08\.1 or later\. 

**Topics**
+ [Enabling write forwarding](#aurora-global-database-write-forwarding-enabling)
+ [Checking if a secondary cluster has write forwarding enabled](#aurora-global-database-write-forwarding-describing)
+ [Application and SQL compatibility with write forwarding](#aurora-global-database-write-forwarding-compatibility)
+ [Isolation and consistency for write forwarding](#aurora-global-database-write-forwarding-isolation)
+ [Running multipart statements with write forwarding](#aurora-global-database-write-forwarding-multipart)
+ [Transactions with write forwarding](#aurora-global-database-write-forwarding-txns)
+ [Configuration parameters for write forwarding](#aurora-global-database-write-forwarding-params)
+ [Amazon CloudWatch metrics for write forwarding](#aurora-global-database-write-forwarding-cloudwatch)

## Enabling write forwarding<a name="aurora-global-database-write-forwarding-enabling"></a>

 By default, write forwarding isn't enabled when you add a secondary cluster to an Aurora global database\. 

 To enable write forwarding using the AWS Management Console, choose the **Enable read replica write forwarding** option when you add a Region for a global database\. For an existing secondary cluster, modify the cluster to use the **Enable read replica write forwarding** option\. To turn off write forwarding, clear the **Enable read replica write forwarding** check box when adding the Region or modifying the secondary cluster\. 

 To enable write forwarding using the AWS CLI, use the `--enable-global-write-forwarding` option\. This option works when you create a new secondary cluster using the `create-db-cluster` command\. It also works when you modify an existing secondary cluster using the `modify-db-cluster` command\. It requires that the global database uses an Aurora version that supports write forwarding\. You can turn write forwarding off by using the `--no-enable-global-write-forwarding` option with these same CLI commands\. 

 To enable write forwarding using the Amazon RDS API, set the `EnableGlobalWriteForwarding` parameter to `true`\. This parameter works when you create a new secondary cluster using the `CreateDBCluster` operation\. It also works when you modify an existing secondary cluster using the `ModifyDBCluster` operation\. It requires that the global database uses an Aurora version that supports write forwarding\. You can turn write forwarding off by setting the `EnableGlobalWriteForwarding` parameter to `false`\. 

**Note**  
For a database session to use write forwarding, specify a setting for the `aurora_replica_read_consistency` configuration parameter\. Do this in every session that uses the write forwarding feature\. For information about this parameter, see [Isolation and consistency for write forwarding](#aurora-global-database-write-forwarding-isolation)\.   
The RDS Proxy feature doesn't support the `SESSION` value for the `aurora_replica_read_consistency` variable\. Setting this value can cause unexpected behavior\.

The following CLI examples show how you can set up an Aurora global database with write forwarding enabled or disabled\. The highlighted items represent the commands and options that are important to specify and keep consistent when setting up the infrastructure for an Aurora global database\. 

 The following example creates an Aurora global database, a primary cluster, and a secondary cluster with write forwarding enabled\. Substitute your own choices for the user name, password, and primary and secondary AWS Regions\. 

```
# Create overall global database.
aws rds create-global-cluster --global-cluster-identifier write-forwarding-test \
  --engine aurora-mysql --engine-version 5.7.mysql_aurora.2.08.1 \
  --region us-east-1

# Create primary cluster, in the same AWS Region as the global database.
aws rds create-db-cluster --global-cluster-identifier write-forwarding-test \
  --db-cluster-identifier write-forwarding-test-cluster-1 \
  --engine aurora-mysql --engine-version 5.7.mysql_aurora.2.08.1 \
  --master-username my_user_name --master-user-password my_password \
  --region us-east-1

aws rds create-db-instance --db-cluster-identifier write-forwarding-test-cluster-1 \
  --db-instance-identifier write-forwarding-test-cluster-1-instance-1 \
  --db-instance-class db.r5.large \
  --engine aurora-mysql --engine-version 5.7.mysql_aurora.2.08.1 \
  --region us-east-1

aws rds create-db-instance --db-cluster-identifier write-forwarding-test-cluster-1 \
  --db-instance-identifier write-forwarding-test-cluster-1-instance-2 \
  --db-instance-class db.r5.large \
  --engine aurora-mysql --engine-version 5.7.mysql_aurora.2.08.1 \
  --region us-east-1

# Create secondary cluster, in a different AWS Region than the global database,
# with write forwarding enabled.
aws rds create-db-cluster --global-cluster-identifier write-forwarding-test \
  --db-cluster-identifier write-forwarding-test-cluster-2 \
  --engine aurora-mysql --engine-version 5.7.mysql_aurora.2.08.1 \
  --region us-east-2 \
  --enable-global-write-forwarding

aws rds create-db-instance --db-cluster-identifier write-forwarding-test-cluster-2 \
  --db-instance-identifier write-forwarding-test-cluster-2-instance-1 \
  --db-instance-class db.r5.large \
  --engine aurora-mysql --engine-version 5.7.mysql_aurora.2.08.1 \
  --region us-east-2

aws rds create-db-instance --db-cluster-identifier write-forwarding-test-cluster-2 \
  --db-instance-identifier write-forwarding-test-cluster-2-instance-2 \
  --db-instance-class db.r5.large \
  --engine aurora-mysql --engine-version 5.7.mysql_aurora.2.08.1 \
  --region us-east-2
```

 The following example continues from the previous one\. It creates a secondary cluster without write forwarding enabled, then enables write forwarding\. After this example finishes, all secondary clusters in the global database have write forwarding enabled\. 

```
# Create secondary cluster, in a different AWS Region than the global database,
# without write forwarding enabled.
aws rds create-db-cluster --global-cluster-identifier write-forwarding-test \
  --db-cluster-identifier write-forwarding-test-cluster-2 \
  --engine aurora-mysql --engine-version 5.7.mysql_aurora.2.08.1 \
  --region us-west-1

aws rds create-db-instance --db-cluster-identifier write-forwarding-test-cluster-2 \
  --db-instance-identifier write-forwarding-test-cluster-2-instance-1 \
  --db-instance-class db.r5.large \
  --engine aurora-mysql --engine-version 5.7.mysql_aurora.2.08.1 \
  --region us-west-1

aws rds create-db-instance --db-cluster-identifier write-forwarding-test-cluster-2 \
  --db-instance-identifier write-forwarding-test-cluster-2-instance-2 \
  --db-instance-class db.r5.large \
  --engine aurora-mysql --engine-version 5.7.mysql_aurora.2.08.1 \
  --region us-west-1

aws rds modify-db-cluster --db-cluster-identifier write-forwarding-test-cluster-2 \
  --region us-east-2 \
  --enable-global-write-forwarding
```

## Checking if a secondary cluster has write forwarding enabled<a name="aurora-global-database-write-forwarding-describing"></a>

 To determine whether you can use write forwarding from a secondary cluster, you can check whether the cluster has the attribute `"GlobalWriteForwardingStatus": "enabled"`\. 

 In the AWS Management Console, you see `Read replica write forwarding` on the **Configuration** tab of the details page for the cluster\. To see the status of the global write forwarding setting for all of your clusters, run the following AWS CLI command\. 

 A secondary cluster shows the value `"enabled"` or `"disabled"` to indicate if write forwarding is turned on or off\. A value of `null` indicates that write forwarding isn't available for that cluster\. Either the cluster isn't part of a global database, or is the primary cluster instead of a secondary cluster\. The value can also be `"enabling"` or `"disabling"` if write forwarding is in the process of being turned on or off\. 

**Example**  

```
aws rds describe-db-clusters --query '*[].{DBClusterIdentifier:DBClusterIdentifier,GlobalWriteForwardingStatus:GlobalWriteForwardingStatus}'
[
    {
        "GlobalWriteForwardingStatus": "enabled",
        "DBClusterIdentifier": "aurora-write-forwarding-test-replica-1"
    },
    {
        "GlobalWriteForwardingStatus": "disabled",
        "DBClusterIdentifier": "aurora-write-forwarding-test-replica-2"
    },
    {
        "GlobalWriteForwardingStatus": null,
        "DBClusterIdentifier": "non-global-cluster"
    }
]
```

 To find all secondary clusters that have global write forwarding enabled, run the following command\. This command also returns the cluster's reader endpoint\. You use the secondary cluster's reader endpoint when you use write forwarding from the secondary to the primary in your Aurora global database\. 

**Example**  

```
aws rds describe-db-clusters --query 'DBClusters[].{DBClusterIdentifier:DBClusterIdentifier,GlobalWriteForwardingStatus:GlobalWriteForwardingStatus,ReaderEndpoint:ReaderEndpoint} | [?GlobalWriteForwardingStatus == `enabled`]'
[
    {
        "GlobalWriteForwardingStatus": "enabled",
        "ReaderEndpoint": "aurora-write-forwarding-test-replica-1.cluster-ro-cnpexample.us-west-2.rds.amazonaws.com",
        "DBClusterIdentifier": "aurora-write-forwarding-test-replica-1"
    }
]
```

## Application and SQL compatibility with write forwarding<a name="aurora-global-database-write-forwarding-compatibility"></a>

 Certain statements aren't allowed or can produce stale results when you use them in a global database with write forwarding\. Thus, the `EnableGlobalWriteForwarding` setting is turned off by default for secondary clusters\. Before turning it on, check to make sure that your application code isn't affected by any of these restrictions\. 

 You can use the following kinds of SQL statements with write forwarding: 
+  Data manipulation language \(DML\) statements, such as `INSERT`, `DELETE`, and `UPDATE`\. There are some restrictions on the properties of these statements that you can use with write forwarding, as described following\. 
+  `SELECT ... LOCK IN SHARE MODE` and `SELECT FOR UPDATE` statements\. 
+  `PREPARE` and `EXECUTE` statements\. 

 The following restrictions apply to the SQL statements you use with write forwarding\. In some cases, you can use the statements on secondary clusters with write forwarding enabled at the cluster level\. This approach works if write forwarding isn't turned on within the session by the `aurora_replica_read_consistency` configuration parameter\. Trying to use a statement when it's not allowed because of write forwarding causes an error message with the following format\. 

```
ERROR 1235 (42000): This version of MySQL doesn't yet support 'operation with write forwarding'.
```

**Data definition language \(DDL\)**  
 Connect to the primary cluster to run DDL statements\. 

**Updating a permanent table using data from a temporary table**  
 You can use temporary tables on secondary clusters with write forwarding enabled\. However, you can't use a DML statement to modify a permanent table if the statement refers to a temporary table\. For example, you can't use an `INSERT ... SELECT` statement that takes the data from a temporary table\. The temporary table exists on the secondary cluster and isn't available when the statement runs on the primary cluster\. 

**XA transactions**  
 You can't use the following statements on a secondary cluster when write forwarding is turned on within the session\. You can use these statements on secondary clusters that don't have write forwarding enabled, or within sessions where the `aurora_replica_read_consistency` setting is empty\. Before turning on write forwarding within a session, check if your code uses these statements\.   

```
XA {START|BEGIN} xid [JOIN|RESUME]
XA END xid [SUSPEND [FOR MIGRATE]]
XA PREPARE xid
XA COMMIT xid [ONE PHASE]
XA ROLLBACK xid
XA RECOVER [CONVERT XID]
```

**LOAD statements for permanent tables**  
 You can't use the following statements on a secondary cluster with write forwarding enabled\.   

```
LOAD DATA INFILE 'data.txt' INTO TABLE t1;
        LOAD XML LOCAL INFILE 'test.xml' INTO TABLE t1;
```
 You can load data into a temporary table on a secondary cluster\. However, make sure that you run any `LOAD` statements that refer to permanent tables only on the primary cluster\. 

**Plugin statements**  
 You can't use the following statements on a secondary cluster with write forwarding enabled\.   

```
INSTALL PLUGIN example SONAME 'ha_example.so';
UNINSTALL PLUGIN example;
```

**SAVEPOINT statements**  
 You can't use the following statements on a secondary cluster when write forwarding is turned on within the session\. You can use these statements on secondary clusters that don't have write forwarding enabled, or within sessions where the `aurora_replica_read_consistency` setting is blank\. Check if your code uses these statements before turning on write forwarding within a session\.   

```
SAVEPOINT t1_save;
ROLLBACK TO SAVEPOINT t1_save;
RELEASE SAVEPOINT t1_save;
```

## Isolation and consistency for write forwarding<a name="aurora-global-database-write-forwarding-isolation"></a>

 In sessions that use write forwarding, you can only use the `REPEATABLE READ` isolation level\. Although you can also use the `READ COMMITTED` isolation level with read\-only clusters in secondary AWS Regions, that isolation level doesn't work with write forwarding\. For information about the `REPEATABLE READ` and `READ COMMITTED` isolation levels, see [Aurora MySQL isolation levels](AuroraMySQL.Reference.IsolationLevels.md)\. 

 You can control the degree of read consistency on a secondary cluster\. The read consistency level determines how much waiting the secondary cluster does before each read operation to ensure that some or all changes are replicated from the primary cluster\. You can adjust the read consistency level to ensure that all forwarded write operations from your session are visible in the secondary cluster before any subsequent queries\. You can also use this setting to ensure that queries on the secondary cluster always see the most current updates from the primary cluster\. This is so even for those submitted by other sessions or other clusters\. To specify this type of behavior for your application, you choose a value for the session\-level parameter `aurora_replica_read_consistency`\. 

**Important**  
Always set the `aurora_replica_read_consistency` parameter for any session for which you want to forward writes\. If you don't, Aurora doesn't enable write forwarding for that session\. This parameter has an empty value by default, so choose a specific value when you use this parameter\. The `aurora_replica_read_consistency` parameter has an effect only on secondary clusters that have write forwarding enabled\. 

 For the `aurora_replica_read_consistency` parameter, you can specify the values `EVENTUAL`, `SESSION`, and `GLOBAL`\. 

 As you increase the consistency level, your application spends more time waiting for changes to be propagated between AWS Regions\. You can choose the balance between fast response time and ensuring that changes made in other locations are fully available before your queries run\. 

 With the read consistency set to `EVENTUAL`, queries in a secondary AWS Region that uses write forwarding might see data that is slightly stale due to replication lag\. Results of write operations in the same session aren't visible until the write operation is performed on the primary Region and replicated to the current Region\. The query doesn't wait for the updated results to be available\. Thus, it might retrieve the older data or the updated data, depending on the timing of the statements and the amount of replication lag\. 

 With the read consistency set to `SESSION`, all queries in a secondary AWS Region that uses write forwarding see the results of all changes made in that session\. The changes are visible regardless of whether the transaction is committed\. If necessary, the query waits for the results of forwarded write operations to be replicated to the current Region\. It doesn't wait for updated results from write operations performed in other Regions or in other sessions within the current Region\. 

 With the read consistency set to `GLOBAL`, a session in a secondary AWS Region sees changes made by that session\. It also sees all committed changes from both the primary AWS Region and other secondary AWS Regions\. Each query might wait for a period that varies depending on the amount of session lag\. The query proceeds when the secondary cluster is up\-to\-date with all committed data from the primary cluster, as of the time that the query began\. 

 For more information about all the parameters involved with write forwarding, see [Configuration parameters for write forwarding](#aurora-global-database-write-forwarding-params)\. 

### Examples of using write forwarding<a name="aurora-global-database-write-forwarding-examples"></a>

In the following example, the primary cluster is in the US East \(N\. Virginia\) Region\. The secondary cluster is in the US East \(Ohio\) Region\. The example shows the effects of running `INSERT` statements followed by `SELECT` statements\. Depending on the value of the `aurora_replica_read_consistency` setting, the results might differ depending on the timing of the statements\. To achieve higher consistency, you might wait briefly before issuing the `SELECT` statement\. Or Aurora can automatically wait until the results finish replicating before proceeding with `SELECT`\. 

In this example, there is a read consistency setting of `eventual`\. Running an `INSERT` statement immediately followed by a `SELECT` statement still returns the value of `COUNT(*)`\. This value reflects the number of rows before the new row is inserted\. Running the `SELECT` again a short time later does return the updated row count\. The `SELECT` statements don't do any waiting\. 

```
mysql> set aurora_replica_read_consistency = 'eventual';
mysql> select count(*) from t1;
+----------+
| count(*) |
+----------+
|        5 |
+----------+
1 row in set (0.00 sec)
mysql> insert into t1 values (6); select count(*) from t1;
+----------+
| count(*) |
+----------+
|        5 |
+----------+
1 row in set (0.00 sec)
mysql> select count(*) from t1;
+----------+
| count(*) |
+----------+
|        6 |
+----------+
1 row in set (0.00 sec)
```

 With a read consistency setting of `session`, a `SELECT` statement immediately after an `INSERT` does wait until the changes from the `INSERT` statement are visible\. Subsequent `SELECT` statements don't do any waiting\. 

```
mysql> set aurora_replica_read_consistency = 'session';
mysql> select count(*) from t1;
+----------+
| count(*) |
+----------+
|        6 |
+----------+
1 row in set (0.01 sec)
mysql> insert into t1 values (6); select count(*) from t1; select count(*) from t1;
Query OK, 1 row affected (0.08 sec)
+----------+
| count(*) |
+----------+
|        7 |
+----------+
1 row in set (0.37 sec)
+----------+
| count(*) |
+----------+
|        7 |
+----------+
1 row in set (0.00 sec)
```

 With the read consistency setting still set to `session`, introducing a brief wait after performing an `INSERT` statement makes the updated row count available by the time the next `SELECT` statement runs\. 

```
mysql> insert into t1 values (6); select sleep(2); select count(*) from t1;
Query OK, 1 row affected (0.07 sec)
+----------+
| sleep(2) |
+----------+
|        0 |
+----------+
1 row in set (2.01 sec)
+----------+
| count(*) |
+----------+
|        8 |
+----------+
1 row in set (0.00 sec)
```

 With a read consistency setting of `global`, each `SELECT` statement waits to ensure that all data changes as of the start time of the statement are visible before performing the query\. The amount of waiting for each `SELECT` statement varies, depending on the amount of replication lag between the primary and secondary clusters\. 

```
mysql> set aurora_replica_read_consistency = 'global';
mysql> select count(*) from t1;
+----------+
| count(*) |
+----------+
|        8 |
+----------+
1 row in set (0.75 sec)
mysql> select count(*) from t1;
+----------+
| count(*) |
+----------+
|        8 |
+----------+
1 row in set (0.37 sec)
mysql> select count(*) from t1;
+----------+
| count(*) |
+----------+
|        8 |
+----------+
1 row in set (0.66 sec)
```

## Running multipart statements with write forwarding<a name="aurora-global-database-write-forwarding-multipart"></a>

 A DML statement might consist of multiple parts, such as a `INSERT ... SELECT` statement or a `DELETE ... WHERE` statement\. In this case, the entire statement is forwarded to the primary cluster and run there\. 

## Transactions with write forwarding<a name="aurora-global-database-write-forwarding-txns"></a>

 Whether the transaction is forwarded to the primary cluster depends on the access mode of the transaction\. You can specify the access mode for the transaction by using the `SET TRANSACTION` statement or the `START TRANSACTION` statement\. You can also specify the transaction access mode by changing the value of the Aurora MySQL session variable `tx_read_only`\. You can only change this session value while you're connected to a secondary cluster that has write forwarding enabled\. 

 If a long\-running transaction doesn't issue any statement for a substantial period of time, it might exceed the idle timeout period\. This period has a default of one minute\. You can increase it up to one day\. A transaction that exceeds the idle timeout is canceled by the primary cluster\. The next subsequent statement you submit receives a timeout error\. Then Aurora rolls back the transaction\. 

 This type of error can occur in other cases when write forwarding becomes unavailable\. For example, Aurora cancels any transactions that use write forwarding if you restart the primary cluster or if you turn off the write forwarding configuration setting\. 

## Configuration parameters for write forwarding<a name="aurora-global-database-write-forwarding-params"></a>

 The Aurora cluster parameter groups include settings for the write forwarding feature\. Because these are cluster parameters, all DB instances in each cluster have the same values for these variables\. Details about these parameters are summarized in the following table, with usage notes after the table\.


|  Name  |  Scope  |  Type  |  Default value  |  Valid values  | 
| --- | --- | --- | --- | --- | 
|  aurora\_fwd\_master\_idle\_timeout \(Aurora MySQL version 2\)  |  Global  |  unsigned integer  |  60  |  1–86,400  | 
|  aurora\_fwd\_master\_max\_connections\_pct \(Aurora MySQL version 2\)  |  Global  |  unsigned long integer  |  10  |  0–90  | 
|  aurora\_fwd\_writer\_idle\_timeout \(Aurora MySQL version 3\)  |  Global  |  unsigned integer  |  60  |  1–86,400  | 
|  aurora\_fwd\_writer\_max\_connections\_pct \(Aurora MySQL version 3\)  |  Global  |  unsigned long integer  |  10  |  0–90  | 
|  aurora\_replica\_read\_consistency  |  Session  |  Enum  |  ''  |  EVENTUAL, SESSION, GLOBAL  | 

To control incoming write requests from secondary clusters, use these settings on the primary cluster: 
+  `aurora_fwd_master_idle_timeout`, `aurora_fwd_writer_idle_timeout`: The number of seconds the primary cluster waits for activity on a connection that's forwarded from a secondary cluster before closing it\. If the session remains idle beyond this period, Aurora cancels the session\. 
+  `aurora_fwd_master_max_connections_pct`, `aurora_fwd_writer_max_connections_pct`: The upper limit on database connections that can be used on a writer DB instance to handle queries forwarded from readers\. It's expressed as a percentage of the `max_connections` setting for the writer DB instance in the primary cluster\. For example, if `max_connections` is 800 and `aurora_fwd_master_max_connections_pct` or `aurora_fwd_writer_max_connections_pct` is 10, then the writer allows a maximum of 80 simultaneous forwarded sessions\. These connections come from the same connection pool managed by the `max_connections` setting\. 

   This setting applies only on the primary cluster, when one or more secondary clusters have write forwarding enabled\. If you decrease the value, existing connections aren't affected\. Aurora takes the new value of the setting into account when attempting to create a new connection from a secondary cluster\. The default value is 10, representing 10% of the `max_connections` value\. If you enable query forwarding on any of the secondary clusters, this setting must have a nonzero value for write operations from secondary clusters to succeed\. If the value is zero, the write operations receive the error code `ER_CON_COUNT_ERROR` with the message `Not enough connections on writer to handle your request`\. 

The `aurora_replica_read_consistency` parameter is a session\-level parameter that enables write forwarding\. You use it in each session\. You can specify `EVENTUAL`, `SESSION`, or `GLOBAL` for read consistency level\. To learn more about consistency levels, see [Isolation and consistency for write forwarding](#aurora-global-database-write-forwarding-isolation)\. The following rules apply to this parameter:
+  This is a session\-level parameter\. The default value is '' \(empty\)\. 
+  Write forwarding is available in a session only if `aurora_replica_read_consistency` is set to `EVENTUAL` or `SESSION` or `GLOBAL`\. This parameter is relevant only in reader instances of secondary clusters that have write forwarding enabled and that are in an Aurora global database\. 
+  You can't set this variable \(when empty\) or unset \(when already set\) inside a multistatement transaction\. However, you can change it from one valid value \(`EVENTUAL`, `SESSION`, or `GLOBAL`\) to another valid value \(`EVENTUAL`, `SESSION`, or `GLOBAL`\) during such a transaction\. 
+  The variable can't be `SET` when write forwarding isn't enabled on the secondary cluster\. 
+  Setting the session variable on a primary cluster doesn't have any effect\. If you try to modify this variable on a primary cluster, you receive an error\. 

## Amazon CloudWatch metrics for write forwarding<a name="aurora-global-database-write-forwarding-cloudwatch"></a>

 The following Amazon CloudWatch metrics apply to the primary cluster when you use write forwarding on one or more secondary clusters\. These metrics are all measured on the writer DB instance in the primary cluster\. 


| CloudWatch Metric\(Aurora MySQL status variable\) | Units and description | 
| --- | --- | 
|  `ForwardingMasterDMLLatency` \(–\)  | Milliseconds\. Average time to process each forwarded DML statement on the writer DB instance\. It doesn't include the time for the secondary cluster to forward the write request\. It also doesn't include the time to replicate changes back to the secondary cluster\. For Aurora MySQL version 2\.   | 
|  `ForwardingMasterOpenSessions` \(`Aurora_fwd_master_open_sessions`\)  | Count\. Number of forwarded sessions on the writer DB instance\. For Aurora MySQL version 2\.  | 
| `ForwardingMasterDMLThroughput` \(–\)   | Count, per second\. Number of forwarded DML statements processed each second by this writer DB instance\. For Aurora MySQL version 2\.  | 
| – \(`Aurora_fwd_master_dml_stmt_duration`\) | Microseconds\. Total duration of DML statements forwarded to this writer DB instance\. For Aurora MySQL version 2\.  | 
| –\(`Aurora_fwd_master_dml_stmt_count`\) | Count\. Total number of DML statements forwarded to this writer DB instance\. For Aurora MySQL version 2\. | 
| – \(`Aurora_fwd_master_select_stmt_duration`\) | Microseconds\. Total duration of SELECT statements forwarded to this writer DB instance\. For Aurora MySQL version 2\.  | 
| – \(`Aurora_fwd_master_select_stmt_count`\) | Count\. Total number of SELECT statements forwarded to this writer DB instance\. For Aurora MySQL version 2\.  | 
|  `ForwardingWriterDMLLatency` \(–\)  | Milliseconds\. Average time to process each forwarded DML statement on the writer DB instance\. It doesn't include the time for the secondary cluster to forward the write request\. It also doesn't include the time to replicate changes back to the secondary cluster\. For Aurora MySQL version 3 and higher\.   | 
|  `ForwardingWriterOpenSessions` \(`Aurora_fwd_writer_open_sessions`\)  | Count\. Number of forwarded sessions on the writer DB instance\. For Aurora MySQL version 3 and higher\.  | 
| `ForwardingWriterDMLThroughput` \(–\)   | Count, per second\. Number of forwarded DML statements processed each second by this writer DB instance\. For Aurora MySQL version 3 and higher\.  | 
| – \(`Aurora_fwd_writer_dml_stmt_duration`\) | Microseconds\. Total duration of DML statements forwarded to this writer DB instance\.  | 
| –\(`Aurora_fwd_writer_dml_stmt_count`\) | Count\. Total number of DML statements forwarded to this writer DB instance\. For Aurora MySQL version 3 and higher\. | 
| – \(`Aurora_fwd_writer_select_stmt_duration`\) | Microseconds\. Total duration of SELECT statements forwarded to this writer DB instance\. For Aurora MySQL version 3 and higher\.  | 
| – \(`Aurora_fwd_writer_select_stmt_count`\) | Count\. Total number of SELECT statements forwarded to this writer DB instance\. For Aurora MySQL version 3 and higher\.  | 

 The following CloudWatch metrics apply to each secondary cluster\. These metrics are measured on each reader DB instance in a secondary cluster with write forwarding enabled\. 


| CloudWatch Metric\(Aurora MySQL status variable\) | Unit and description | 
| --- | --- | 
|  `ForwardingReplicaDMLLatency` \(–\) |  Milliseconds\. Average response time in milliseconds of forwarded DMLs on replica\.  | 
|  `ForwardingReplicaReadWaitLatency` \(–\) |  Milliseconds\. Average wait time in milliseconds that a SELECT statement on a reader DB instance waits to catch up to the primary cluster\. The degree to which the reader DB instance waits before processing a query depends on the aurora\_replica\_read\_consistency setting\.  | 
|  `ForwardingReplicaDMLThroughput` \(–\) |  Count \(per second\)\. Number of forwarded DML statements processed each second\.  | 
|  `ForwardingReplicaReadWaitThroughput` \(–\) |  Count \(SELECT statements per second\)\. Total number of SELECT statements processed each second in all sessions that are forwarding writes\.  | 
|  `ForwardingReplicaOpenSessions` \(`Aurora_fwd_replica_open_sessions`  |  Count\. The number of sessions that are using write forwarding on a reader DB instance\.  | 
|   `ForwardingReplicaSelectLatency`   `(–)`   | Milliseconds\. Forwarded SELECT latency, average over all forwarded SELECT statements within the monitoring period\. | 
|   `ForwardingReplicaSelectThroughput`   `(–)`   | Count per second\. Forwarded SELECT throughput, per second average within the monitoring period\. | 
| – \(`Aurora_fwd_replica_dml_stmt_count`\)  | Count\. Total number of DML statements forwarded from this reader DB instance\.  | 
| – \(`Aurora_fwd_replica_dml_stmt_duration`\)  |  Microseconds\. Total duration of all DML statements forwarded from this reader DB instance\.  | 
| – \(`Aurora_fwd_replica_select_stmt_duration`\) | Microseconds\. Total duration of SELECT statements forwarded from this reader DB instance\.  | 
| – \(`Aurora_fwd_replica_select_stmt_count` | Count\. Total number of SELECT statements forwarded from this reader DB instance\.  | 
| – \(`Aurora_fwd_replica_read_wait_duration`  |  Microseconds\. Total duration of waits due to the read consistency setting on this reader DB instance\.  | 
| – \(`Aurora_fwd_replica_read_wait_count`\)  | Count\. Total number of read\-after\-write waits on this reader DB instance\.  | 
| – \(`Aurora_fwd_replica_errors_session_limit`\) | Count\. Number of sessions rejected by the primary cluster due to the error conditions writer full or Too many forwarded statements in progress\.  | 