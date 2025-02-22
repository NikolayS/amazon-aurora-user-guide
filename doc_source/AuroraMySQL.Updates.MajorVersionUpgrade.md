# Upgrading the major version of an Amazon Aurora MySQL DB cluster<a name="AuroraMySQL.Updates.MajorVersionUpgrade"></a><a name="mvu"></a>

In an Aurora MySQL version number such as 2\.11\.1, the 2 represents the major version\. Aurora MySQL version 2 is compatible with MySQL 5\.7\. Aurora MySQL version 3 is compatible with MySQL 8\.0\.

 Upgrading between major versions requires more extensive planning and testing than for a minor version\. The process can take substantial time\. After the upgrade is finished, you also might have followup work to do\. For example, this might occur because of differences in SQL compatibility or the way certain MySQL\-related features work\. Or it might occur because of differing parameter settings between the old and new versions\. 

**Topics**
+ [Upgrading from Aurora MySQL 2\.x to 3\.x](#AuroraMySQL.Updates.MajorVersionUpgrade.2to3)
+ [Planning a major version upgrade for an Aurora MySQL cluster](#AuroraMySQL.Upgrading.Planning)
+ [Aurora MySQL major version upgrade paths](#AuroraMySQL.Upgrading.Compatibility)
+ [How the Aurora MySQL in\-place major version upgrade works](#AuroraMySQL.Upgrading.Sequence)
+ [How to perform an in\-place upgrade](#AuroraMySQL.Upgrading.Procedure)
+ [How in\-place upgrades affect the parameter groups for a cluster](#AuroraMySQL.Upgrading.ParamGroups)
+ [Changes to cluster properties between Aurora MySQL versions](#AuroraMySQL.Upgrading.Attrs)
+ [In\-place major upgrades for global databases](#AuroraMySQL.Upgrading.GlobalDB)
+ [After the upgrade](#AuroraMySQL.Upgrading.PostUpgrade)
+ [Troubleshooting for Aurora MySQL in\-place upgrade](#AuroraMySQL.Upgrading.Troubleshooting)
+ [Aurora MySQL in\-place upgrade tutorial](#AuroraMySQL.Upgrading.Tutorial)
+ [Alternative blue\-green upgrade technique](#AuroraMySQL.UpgradingMajor.BlueGreen)

## Upgrading from Aurora MySQL 2\.x to 3\.x<a name="AuroraMySQL.Updates.MajorVersionUpgrade.2to3"></a>

If you have a MySQL 5\.7–compatible cluster and want to upgrade it to a MySQL–8\.0 compatible cluster, you can do so by running an upgrade process on the cluster itself\. This kind of upgrade is an *in\-place upgrade*, in contrast to upgrades that you do by creating a new cluster\. This technique keeps the same endpoint and other characteristics of the cluster\. The upgrade is relatively fast because it doesn't require copying all your data to a new cluster volume\. This stability helps to minimize any configuration changes in your applications\. It also helps to reduce the amount of testing for the upgraded cluster\. This is because the number of DB instances and their instance classes all stay the same\.

The in\-place upgrade mechanism involves shutting down your DB cluster while the operation takes place\. Aurora performs a clean shutdown and completes outstanding operations such as transaction rollback and undo purge\. For more information, see [How the Aurora MySQL in\-place major version upgrade works](#AuroraMySQL.Upgrading.Sequence)\.

The in\-place upgrade method is convenient, because it is simple to perform and minimizes configuration changes to associated applications\. For example, an in\-place upgrade preserves the endpoints and set of DB instances for your cluster\. However, the time needed for an in\-place upgrade can vary depending on the properties of your schema and how busy the cluster is\. Thus, depending on the needs for your cluster, you might want to choose among the multiple upgrade techniques\. These include in\-place upgrade and snapshot restore, described in [Restoring from a DB cluster snapshot](aurora-restore-snapshot.md)\. They also include other upgrade techniques such as the one described in [Alternative blue\-green upgrade technique](#AuroraMySQL.UpgradingMajor.BlueGreen)\.

**Note**  
If you use the AWS CLI or RDS API for the snapshot restore upgrade method, you must run a subsequent operation to create a writer DB instance in the restored DB cluster\.

For general information about Aurora MySQL version 3 and its new features, see [Aurora MySQL version 3 compatible with MySQL 8\.0](AuroraMySQL.MySQL80.md)\. For details about planning an upgrade, see [Upgrade planning for Aurora MySQL version 3](AuroraMySQL.mysql80-upgrade-procedure.md#AuroraMySQL.mysql80-planning) and [Upgrading to Aurora MySQL version 3](AuroraMySQL.mysql80-upgrade-procedure.md)\.

**Tip**  
When you upgrade the major version of your cluster from 2\.x to 3\.x, the original cluster and the upgraded one both use the same `aurora-mysql` value for the `engine` attribute\.

## Planning a major version upgrade for an Aurora MySQL cluster<a name="AuroraMySQL.Upgrading.Planning"></a>

To make sure that your applications and administration procedures work smoothly after upgrading a cluster between major versions, do some advance planning and preparation\. To see what sorts of management code to update for your AWS CLI scripts or RDS API–based applications, see [How in\-place upgrades affect the parameter groups for a cluster](#AuroraMySQL.Upgrading.ParamGroups)\. Also see [Changes to cluster properties between Aurora MySQL versions](#AuroraMySQL.Upgrading.Attrs)\.

To learn the sorts of issues that you might encounter during the upgrade, see [Troubleshooting for Aurora MySQL in\-place upgrade](#AuroraMySQL.Upgrading.Troubleshooting)\. For issues that might cause the upgrade to take a long time, you can test those conditions in advance and correct them\.

You can check application compatibility, performance, maintenance procedures, and similar considerations for the upgraded cluster\. To do so, you can perform a simulation of the upgrade before doing the real upgrade\. This technique can be especially useful for production clusters\. Here, it's important to minimize downtime and have the upgraded cluster ready to go as soon as the upgrade has finished\.

Use the following steps:

1. Create a clone of the original cluster\. Follow the procedure in [Cloning a volume for an Amazon Aurora DB cluster](Aurora.Managing.Clone.md)\.

1. Set up a similar set of writer and reader DB instances as in the original cluster\.

1. Perform an in\-place upgrade of the cloned cluster\. Follow the procedure in [How to perform an in\-place upgrade](#AuroraMySQL.Upgrading.Procedure)\.

   Start the upgrade immediately after creating the clone\. That way, the cluster volume is still identical to the state of the original cluster\. If the clone sits idle before you do the upgrade, Aurora performs database cleanup processes in the background\. In that case, the upgrade of the clone isn't an accurate simulation of upgrading the original cluster\.

1. Test application compatibility, performance, administration procedures, and so on, using the cloned cluster\.

1. If you encounter any issues, adjust your upgrade plans to account for them\. For example, adapt any application code to be compatible with the feature set of the higher version\. Estimate how long the upgrade is likely to take based on the amount of data in your cluster\. You might also choose to schedule the upgrade for a time when the cluster isn't busy\.

1. After you're satisfied that your applications and workload work properly with the test cluster, you can perform the in\-place upgrade for your production cluster\.

1. Work to minimize the total downtime of your cluster during a major version upgrade\. To do so, make sure that the workload on the cluster is low or zero at the time of the upgrade\. In particular, make sure that there are no long running transactions in progress when you start the upgrade\.

For information specific to upgrading to Aurora MySQL version 3, see [Upgrade planning for Aurora MySQL version 3](AuroraMySQL.mysql80-upgrade-procedure.md#AuroraMySQL.mysql80-planning)\.

## Aurora MySQL major version upgrade paths<a name="AuroraMySQL.Upgrading.Compatibility"></a>

Not all kinds or versions of Aurora MySQL clusters can use the in\-place upgrade mechanism\. You can learn the appropriate upgrade path for each Aurora MySQL cluster by consulting the following table\.


|  Type of Aurora MySQL DB cluster  | Can it use in\-place upgrade?  |  Action  | 
| --- | --- | --- | 
|   Aurora MySQL provisioned cluster, 2\.0 or higher   |  Yes  |  In\-place upgrade is supported for 5\.7\-compatible Aurora MySQL clusters\. For information about upgrading to Aurora MySQL version 3, see [Upgrade planning for Aurora MySQL version 3](AuroraMySQL.mysql80-upgrade-procedure.md#AuroraMySQL.mysql80-planning) and [Upgrading to Aurora MySQL version 3](AuroraMySQL.mysql80-upgrade-procedure.md)\.  | 
|   Aurora MySQL provisioned cluster, 3\.01\.0 or higher   |  N/A  |  Use a minor version upgrade procedure to upgrade between Aurora MySQL version 3 versions\.  | 
|  Aurora Serverless v1 cluster  | Yes |  You can upgrade from a MySQL 5\.6\-compatible Aurora Serverless v1 version to a MySQL 5\.7\-compatible one by performing an in\-place upgrade\. For more information, see [Modifying an Aurora Serverless v1 DB cluster](aurora-serverless.modifying.md)\.  | 
|  Aurora Serverless v2 cluster  |  N/A  | Currently, Aurora Serverless v2 is supported for Aurora MySQL only on version 3\. | 
|  Cluster in an Aurora global database  |  Yes  |  To upgrade Aurora MySQL from version 2 to version 3, follow the [procedure for doing an in\-place upgrade](#AuroraMySQL.Upgrading.Procedure) for clusters in an Aurora global database\. Perform the upgrade on the global cluster\. Aurora upgrades the primary cluster and all the secondary clusters in the global database at the same time\. If you use the AWS CLI or RDS API, call the `modify-global-cluster` command or `ModifyGlobalCluster` operation instead of `modify-db-cluster` or `ModifyDBCluster`\. You can't perform an in\-place upgrade from Aurora MySQL version 2 to version 3 if the `lower_case_table_names` parameter is turned on\. For more information, see [Major version upgrades](aurora-global-database-upgrade.md#aurora-global-database-upgrade.major)\.  | 
|  Parallel query cluster  |  Yes  |  You can perform an in\-place upgrade\. In this case, choose 2\.09\.1 or higher for the Aurora MySQL version\.  | 
|  Cluster that is the target of binary log replication  |  Maybe  |  If the binary log replication is from an Aurora MySQL cluster, you can perform an in\-place upgrade\. You can't perform the upgrade if the binary log replication is from an RDS for MySQL or an on\-premises MySQL DB instance\. In that case, you can upgrade using the snapshot restore mechanism\.  | 
|  Cluster with zero DB instances  |  No  |  Using the AWS CLI or the RDS API, you can create an Aurora MySQL cluster without any attached DB instances\. In the same way, you can also remove all DB instances from an Aurora MySQL cluster while leaving the data in the cluster volume intact\. While a cluster has zero DB instances, you can't perform an in\-place upgrade\. The upgrade mechanism requires a writer instance in the cluster to perform conversions on the system tables, data files, and so on\. In this case, use the AWS CLI or the RDS API to create a writer instance for the cluster\. Then you can perform an in\-place upgrade\.  | 
|  Cluster with backtrack enabled  |  Yes  |  You can perform an in\-place upgrade for an Aurora MySQL cluster that uses the backtrack feature\. However, after the upgrade, you can't backtrack the cluster to a time before the upgrade\.  | 

## How the Aurora MySQL in\-place major version upgrade works<a name="AuroraMySQL.Upgrading.Sequence"></a>

 Aurora MySQL performs a major version upgrade as a multistage process\. You can check the current status of an upgrade\. Some of the upgrade steps also provide progress information\. As each stage begins, Aurora MySQL records an event\. You can examine events as they occur on the **Events** page in the RDS console\. For more information about working with events, see [Working with Amazon RDS event notification](USER_Events.md)\. 

**Important**  
 Once the process begins, it runs until the upgrade either succeeds or fails\. You can't cancel the upgrade while it's underway\. If the upgrade fails, Aurora rolls back all the changes and your cluster has the same engine version, metadata, and so on as before\. 

 The upgrade process consists of these stages: 

1.  Aurora performs a series of checks before beginning the upgrade process\. Your cluster keeps running while Aurora does these checks\. For example, the cluster can't have any XA transactions in the prepared state or be processing any data definition language \(DDL\) statements\. For example, you might need to shut down applications that are submitting certain kinds of SQL statements\. Or you might simply wait until certain long\-running statements are finished\. Then try the upgrade again\. Some checks test for conditions that don't prevent the upgrade but might make the upgrade take a long time\. 

    If Aurora detects that any required conditions aren't met, modify the conditions identified in the event details\. Follow the guidance in [Troubleshooting for Aurora MySQL in\-place upgrade](#AuroraMySQL.Upgrading.Troubleshooting)\. If Aurora detects conditions that might cause a slow upgrade, plan to monitor the upgrade over an extended period\. 

1.  Aurora takes your cluster offline\. Then Aurora performs a similar set of tests as in the previous stage, to confirm that no new issues arose during the shutdown process\. If Aurora detects any conditions at this point that would prevent the upgrade, Aurora cancels the upgrade and brings the cluster back online\. In this case, confirm when the conditions no longer apply and start the upgrade again\. 

1.  Aurora creates a snapshot of your cluster volume\. Suppose that you discover compatibility or other kinds of issues after the upgrade is finished\. Or suppose that you want to perform testing using both the original and upgraded clusters\. In such cases, you can restore from this snapshot to create a new cluster with the original engine version and the original data\. 
**Tip**  
 This snapshot is a manual snapshot\. However, Aurora can create it and continue with the upgrade process even if you have reached your quota for manual snapshots\. This snapshot remains permanently until you delete it\. After you finish all post\-upgrade testing, you can delete this snapshot to minimize storage charges\. 

1.  Aurora clones your cluster volume\. Cloning is a fast operation that doesn't involve copying the actual table data\. If Aurora encounters an issue during the upgrade, it reverts to the original data from the cloned cluster volume and brings the cluster back online\. The temporary cloned volume during the upgrade isn't subject to the usual limit on the number of clones for a single cluster volume\. 

1.  Aurora performs a clean shutdown for the writer DB instance\. During the clean shutdown, progress events are recorded every 15 minutes for the following operations\. You can examine events as they occur on the **Events** page in the RDS console\. 
   +  Aurora purges the undo records for old versions of rows\. 
   +  Aurora rolls back any uncommitted transactions\. 

1.  Aurora upgrades the engine version on the writer DB instance: 
   +  Aurora installs the binary for the new engine version on the writer DB instance\. 
   +  Aurora uses the writer DB instance to upgrade your data to MySQL 5\.7\-compatible format\. During this stage, Aurora modifies the system tables and performs other conversions that affect the data in your cluster volume\. In particular, Aurora upgrades the partition metadata in the system tables to be compatible with the MySQL 5\.7 partition format\. This stage can take a long time if the tables in your cluster have a large number of partitions\. 

      If any errors occur during this stage, you can find the details in the MySQL error logs\. After this stage starts, if the upgrade process fails for any reason, Aurora restores the original data from the cloned cluster volume\. 

1.  Aurora upgrades the engine version on the reader DB instances\. 

1.  The upgrade process is completed\. Aurora records a final event to indicate that the upgrade process completed successfully\. Now your DB cluster is running the new major version\. 

## How to perform an in\-place upgrade<a name="AuroraMySQL.Upgrading.Procedure"></a>

We recommend that you review the background material in [How the Aurora MySQL in\-place major version upgrade works](#AuroraMySQL.Upgrading.Sequence)\.

Perform any preupgrade planning and testing, as described in the following:
+ [Planning a major version upgrade for an Aurora MySQL cluster](#AuroraMySQL.Upgrading.Planning)
+ [Upgrade planning for Aurora MySQL version 3](AuroraMySQL.mysql80-upgrade-procedure.md#AuroraMySQL.mysql80-planning)

### Console<a name="AuroraMySQL.Upgrading.ModifyingDBCluster.CON"></a>

**To upgrade the major version of an Aurora MySQL DB cluster**

1. Sign in to the AWS Management Console and open the Amazon RDS console at [https://console\.aws\.amazon\.com/rds/](https://console.aws.amazon.com/rds/)\.

1. If you used a custom parameter group for the original DB cluster, create a corresponding parameter group compatible with the new major version\. Make any necessary adjustments to the configuration parameters in that new parameter group\. For more information, see [How in\-place upgrades affect the parameter groups for a cluster](#AuroraMySQL.Upgrading.ParamGroups)\.

1.  In the navigation pane, choose **Databases**\. 

1.  In the list, choose the DB cluster that you want to modify\. 

1.  Choose **Modify**\. 

1.  For **Version**, choose a new Aurora MySQL major version\. We generally recommend using the latest minor version of the major version\.  
![\[In-place upgrade of an Aurora MySQL DB cluster from version 2 to version 3\]](http://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/images/ams-upgrade-v2-v3.png)

1.  Choose **Continue**\. 

1.  On the next page, specify when to perform the upgrade\. Choose **During the next scheduled maintenance window** or **Immediately**\. 

1.  \(Optional\) Periodically examine the **Events** page in the RDS console during the upgrade\. Doing so helps you to monitor the progress of the upgrade and identify any issues\. If the upgrade encounters any issues, consult [Troubleshooting for Aurora MySQL in\-place upgrade](#AuroraMySQL.Upgrading.Troubleshooting) for the steps to take\. 

1. If you created a new parameter group at the start of this procedure, associate the custom parameter group with your upgraded cluster\. For more information, see [How in\-place upgrades affect the parameter groups for a cluster](#AuroraMySQL.Upgrading.ParamGroups)\.
**Note**  
 Performing this step requires you to restart the cluster again to apply the new parameter group\. 

1.  \(Optional\) After you complete any post\-upgrade testing, delete the manual snapshot that Aurora created at the beginning of the upgrade\. 

### AWS CLI<a name="AuroraMySQL.Upgrading.ModifyingDBCluster.CLI"></a>

 To upgrade the major version of an Aurora MySQL DB cluster, use the AWS CLI [modify\-db\-cluster](https://docs.aws.amazon.com/cli/latest/reference/rds/modify-db-cluster.html) command with the following required parameters: 
+  `--db-cluster-identifier` 
+  `--engine aurora-mysql` 
+  `--engine-version` 
+  `--allow-major-version-upgrade` 
+  `--apply-immediately` or `--no-apply-immediately` 

 If your cluster uses any custom parameter groups, also include one or both of the following options: 
+  `--db-cluster-parameter-group-name`, if the cluster uses a custom cluster parameter group 
+  `--db-instance-parameter-group-name`, if any instances in the cluster use a custom DB parameter group 

The following example upgrades the `sample-cluster` DB cluster to Aurora MySQL version 2\.10\.2\. The upgrade happens immediately, instead of waiting for the next maintenance window\.

**Example**  
For Linux, macOS, or Unix:  

```
aws rds modify-db-cluster \
          --db-cluster-identifier sample-cluster \
          --engine aurora-mysql \
          --engine-version 5.7.mysql_aurora.2.10.2 \
          --allow-major-version-upgrade \
          --apply-immediately
```
For Windows:  

```
aws rds modify-db-cluster ^
          --db-cluster-identifier sample-cluster ^
          --engine aurora-mysql ^
          --engine-version 5.7.mysql_aurora.2.10.2 ^
          --allow-major-version-upgrade ^
          --apply-immediately
```
You can combine other CLI commands with `modify-db-cluster` to create an automated end\-to\-end process for performing and verifying upgrades\. For more information and examples, see [Aurora MySQL in\-place upgrade tutorial](#AuroraMySQL.Upgrading.Tutorial)\.

**Note**  
If your cluster is part of an Aurora global database, the in\-place upgrade procedure is slightly different\. You call the [modify\-global\-cluster](https://docs.aws.amazon.com/cli/latest/reference/rds/modify-global-cluster.html) command operation instead of `modify-db-cluster`\. For more information, see [In\-place major upgrades for global databases](#AuroraMySQL.Upgrading.GlobalDB)\.

### RDS API<a name="AuroraMySQL.Upgrading.ModifyingDBCluster.API"></a>

To upgrade the major version of an Aurora MySQL DB cluster, use the RDS API operation [ModifyDBCluster](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_ModifyDBCluster.html) with the following required parameters:
+ `DBClusterIdentifier`
+ `Engine`
+ `EngineVersion`
+ `AllowMajorVersionUpgrade`
+ `ApplyImmediately` \(set to `true` or `false`\)

**Note**  
If your cluster is part of an Aurora global database, the in\-place upgrade procedure is slightly different\. You call the [ModifyGlobalCluster](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_ModifyGlobalClusterParameterGroup.html) operation instead of `ModifyDBCluster`\. For more information, see [In\-place major upgrades for global databases](#AuroraMySQL.Upgrading.GlobalDB)\.

## How in\-place upgrades affect the parameter groups for a cluster<a name="AuroraMySQL.Upgrading.ParamGroups"></a>

Aurora parameter groups have different sets of configuration settings for clusters that are compatible with MySQL 5\.7 or 8\.0\. When you perform an in\-place upgrade, the upgraded cluster and all its instances must use the corresponding cluster and instance parameter groups:

Your cluster and instances might use the default 5\.7\-compatible parameter groups\. If so, the upgraded cluster and instance start with the default 8\.0\-compatible parameter groups\. If your cluster and instances use any custom parameter groups, make sure to create corresponding or 8\.0\-compatible parameter groups\. Also make sure to specify those during the upgrade process\.

**Important**  
 If you specify any custom parameter group during the upgrade process, make sure to manually reboot the cluster after the upgrade finishes\. Doing so makes the cluster begin using your custom parameter settings\. 

## Changes to cluster properties between Aurora MySQL versions<a name="AuroraMySQL.Upgrading.Attrs"></a>

When you upgrade from Aurora MySQL version 2 to version 3, make sure to check any applications or scripts that you use to set up or manage Aurora MySQL clusters and DB instances\.

Also, change your code that manipulates parameter groups to account for the fact that the default parameter group names are different for 5\.7\- and 8\.0\-compatible clusters\. The default parameter group names for Aurora MySQL version 2 and 3 clusters are `default.aurora-mysql5.7` and `default.aurora-mysql8.0`, respectively\.

For example, you might have code like the following that applies to your cluster before an upgrade\.

```
# Check the default parameter values for MySQL 5.7–compatible clusters.
aws rds describe-db-parameters --db-parameter-group-name default.aurora-mysql5.7 --region us-east-1
```

 After upgrading the major version of the cluster, modify that code as follows\.

```
# Check the default parameter values for MySQL 8.0–compatible clusters.
aws rds describe-db-parameters --db-parameter-group-name default.aurora-mysql8.0 --region us-east-1
```

## In\-place major upgrades for global databases<a name="AuroraMySQL.Upgrading.GlobalDB"></a>

 For an Aurora global database, you upgrade the global database cluster\. Aurora automatically upgrades all of the clusters at the same time and makes sure that they all run the same engine version\. This requirement is because any changes to system tables, data file formats, and so on, are automatically replicated to all the secondary clusters\. 

Follow the instructions in [How the Aurora MySQL in\-place major version upgrade works](#AuroraMySQL.Upgrading.Sequence)\. When you specify what to upgrade, make sure to choose the global database cluster instead of one of the clusters it contains\.

If you use the AWS Management Console, choose the item with the role **Global database**\.

![\[Upgrading global database cluster\]](http://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/images/aurora-global-databases-major-upgrade-global-cluster.png)

 If you use the AWS CLI or RDS API, start the upgrade process by calling the [modify\-global\-cluster](https://docs.aws.amazon.com/cli/latest/reference/rds/modify-global-cluster.html) command or [ModifyGlobalCluster](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_ModifyGlobalCluster.html) operation\. You use one of these instead of `modify-db-cluster` or `ModifyDBCluster`\.

**Note**  
You can't specify a custom parameter group for the global database cluster while you're performing a major version upgrade of that Aurora global database\. Create your custom parameter groups in each Region of the global cluster\. Then apply them manually to the Regional clusters after the upgrade\.

## After the upgrade<a name="AuroraMySQL.Upgrading.PostUpgrade"></a>

 If the cluster that you upgraded had the Backtrack feature enabled, you can't backtrack the upgraded cluster to a time that's before the upgrade\. 

## Troubleshooting for Aurora MySQL in\-place upgrade<a name="AuroraMySQL.Upgrading.Troubleshooting"></a>

Use the following tips to help troubleshoot problems with Aurora MySQL in\-place upgrades\. These tips don't apply to Aurora Serverless DB clusters\.


|  Reason for in\-place upgrade being canceled or slow  | Effect |  Solution to allow in\-place upgrade to complete within maintenance window  | 
| --- | --- | --- | 
|  Cluster has XA transactions in the prepared state  |  Aurora cancels the upgrade\.  |  Commit or roll back all prepared XA transactions\.  | 
|  Cluster is processing any data definition language \(DDL\) statements  |  Aurora cancels the upgrade\.  |  Consider waiting and performing the upgrade after all DDL statements are finished\.  | 
|  Cluster has uncommitted changes for many rows  |  Upgrade might take a long time\.  |  The upgrade process rolls back the uncommitted changes\. The indicator for this condition is the value of `TRX_ROWS_MODIFIED` in the `INFORMATION_SCHEMA.INNODB_TRX` table\. Consider performing the upgrade only after all large transactions are committed or rolled back\.  | 
|  Cluster has high number of undo records  |  Upgrade might take a long time\.  |  Even if the uncommitted transactions don't affect a large number of rows, they might involve a large volume of data\. For example, you might be inserting large BLOBs\. Aurora doesn't automatically detect or generate an event for this kind of transaction activity\. The indicator for this condition is the history list length\. The upgrade process rolls back the uncommitted changes\. Consider performing the upgrade only after the history list length is smaller\.  | 
|  Cluster is in the process of committing a large binary log transaction  |  Upgrade might take a long time\.  |  The upgrade process waits until the binary log changes are applied\. More transactions or DDL statements could start during this period, further slowing down the upgrade process\. Schedule the upgrade process when the cluster isn't busy with generating binary log replication changes\. Aurora doesn't automatically detect or generate an event for this condition\.  | 

 You can use the following steps to perform your own checks for some of the conditions in the preceding table\. That way, you can schedule the upgrade at a time when you know the database is in a state where the upgrade can complete successfully and quickly\. 
+  You can check for open XA transactions by executing the `XA RECOVER` statement\. You can then commit or roll back the XA transactions before starting the upgrade\. 
+  You can check for DDL statements by executing a `SHOW PROCESSLIST` statement and looking for `CREATE`, `DROP`, `ALTER`, `RENAME`, and `TRUNCATE` statements in the output\. Allow all DDL statements to finish before starting the upgrade\. 
+  You can check the total number of uncommitted rows by querying the `INFORMATION_SCHEMA.INNODB_TRX` table\. The table contains one row for each transaction\. The `TRX_ROWS_MODIFIED` column contains the number of rows modified or inserted by the transaction\. 
+  You can check the length of the InnoDB history list by executing the `SHOW ENGINE INNODB STATUS SQL` statement and looking for the `History list length` in the output\. You can also check the value directly by running the following query: 

  ```
  SELECT count FROM information_schema.innodb_metrics WHERE name = 'trx_rseg_history_len';
  ```

   The length of the history list corresponds to the amount of undo information stored by the database to implement multi\-version concurrency control \(MVCC\)\. 

## Aurora MySQL in\-place upgrade tutorial<a name="AuroraMySQL.Upgrading.Tutorial"></a>

The following Linux examples show how you might perform the general steps of the in\-place upgrade procedure using the AWS CLI\.

This first example creates an Aurora DB cluster that's running a 2\.x version of Aurora MySQL\. The cluster includes a writer DB instance and a reader DB instance\. The `wait db-instance-available` command pauses until the writer DB instance is available\. That's the point when the cluster is ready to be upgraded\.

```
aws rds create-db-cluster --db-cluster-identifier mynewdbcluster --engine aurora-mysql \
  --db-cluster-version 5.7.mysql_aurora.2.10.2
...
aws rds create-db-instance --db-instance-identifier mynewdbcluster-instance1 \
  --db-cluster-identifier mynewdbcluster --db-instance-class db.t4g.medium --engine aurora-mysql
...
aws rds wait db-instance-available --db-instance-identifier mynewdbcluster-instance1
```

The Aurora MySQL 3\.x versions that you can upgrade the cluster to depend on the 2\.x version that the cluster is currently running and on the AWS Region where the cluster is located\. The first command, with `--output text`, just shows the available target version\. The second command shows the full JSON output of the response\. In that response, you can see details such as the `aurora-mysql` value that you use for the `engine` parameter\. You can also see the fact that upgrading to 3\.02\.0 represents a major version upgrade\.

```
aws rds describe-db-clusters --db-cluster-identifier mynewdbcluster \
  --query '*[].{EngineVersion:EngineVersion}' --output text
5.7.mysql_aurora.2.10.2

aws rds describe-db-engine-versions --engine aurora-mysql --engine-version 5.7.mysql_aurora.2.10.2 \
  --query '*[].[ValidUpgradeTarget]'
...
{
    "Engine": "aurora-mysql",
    "EngineVersion": "8.0.mysql_aurora.3.02.0",
    "Description": "Aurora MySQL 3.02.0 (compatible with MySQL 8.0.23)",
    "AutoUpgrade": false,
    "IsMajorVersionUpgrade": true,
    "SupportedEngineModes": [
        "provisioned"
    ],
    "SupportsParallelQuery": true,
    "SupportsGlobalDatabases": true,
    "SupportsBabelfish": false
},
...
```

This example shows how if you enter a target version number that isn't a valid upgrade target for the cluster, Aurora doesn't perform the upgrade\. Aurora also doesn't perform a major version upgrade unless you include the `--allow-major-version-upgrade` parameter\. That way, you can't accidentally perform an upgrade that has the potential to require extensive testing and changes to your application code\.

```
aws rds modify-db-cluster --db-cluster-identifier mynewdbcluster \
  --engine-version 5.7.mysql_aurora.2.09.2 --apply-immediately
An error occurred (InvalidParameterCombination) when calling the ModifyDBCluster operation: Cannot find upgrade target from 5.7.mysql_aurora.2.10.2 with requested version 5.7.mysql_aurora.2.09.2.

aws rds modify-db-cluster --db-cluster-identifier mynewdbcluster \
  --engine-version 8.0.mysql_aurora.3.02.0 --region us-east-1 --apply-immediately
An error occurred (InvalidParameterCombination) when calling the ModifyDBCluster operation: The AllowMajorVersionUpgrade flag must be present when upgrading to a new major version.

aws rds modify-db-cluster --db-cluster-identifier mynewdbcluster \
  --engine-version 8.0.mysql_aurora.3.02.0 --apply-immediately --allow-major-version-upgrade
{
  "DBClusterIdentifier": "mynewdbcluster",
  "Status": "available",
  "Engine": "aurora-mysql",
  "EngineVersion": "5.7.mysql_aurora.2.10.2"
}
```

 It takes a few moments for the status of the cluster and associated DB instances to change to `upgrading`\. The version numbers for the cluster and DB instances only change when the upgrade is finished\. Again, you can use the `wait db-instance-available` command for the writer DB instance to wait until the upgrade is finished before proceeding\. 

```
aws rds describe-db-clusters --db-cluster-identifier mynewdbcluster \
  --query '*[].[Status,EngineVersion]' --output text
upgrading 5.7.mysql_aurora.2.10.2

aws rds describe-db-instances --db-instance-identifier mynewdbcluster-instance1 \
  --query '*[].{DBInstanceIdentifier:DBInstanceIdentifier,DBInstanceStatus:DBInstanceStatus} | [0]'
{
    "DBInstanceIdentifier": "mynewdbcluster-instance1",
    "DBInstanceStatus": "upgrading"
}

aws rds wait db-instance-available --db-instance-identifier mynewdbcluster-instance1
```

 At this point, the version number for the cluster matches the one that was specified for the upgrade\. 

```
aws rds describe-db-clusters --db-cluster-identifier mynewdbcluster \
  --query '*[].[EngineVersion]' --output text

8.0.mysql_aurora.3.02.0
```

The preceding example did an immediate upgrade by specifying the `--apply-immediately` parameter\. To let the upgrade happen at a convenient time when the cluster isn't expected to be busy, you can specify the `--no-apply-immediately` parameter\. Doing so makes the upgrade start during the next maintenance window for the cluster\. The maintenance window defines the period during which maintenance operations can start\. A long\-running operation might not finish during the maintenance window\. Thus, you don't need to define a larger maintenance window even if you expect that the upgrade might take a long time\.

The following example upgrades a cluster that's initially running Aurora MySQL version 2\.10\.2\. In the `describe-db-engine-versions` output, the `False` and `True` values represent the `IsMajorVersionUpgrade` property\. From version 2\.10\.2, you can upgrade to some other 2\.\* versions\. Those upgrades aren't considered major version upgrades and so don't require an in\-place upgrade\. In\-place upgrade is only available for upgrades to the 3\.\* versions that are shown in the list\.

```
aws rds describe-db-clusters --db-cluster-identifier mynewdbcluster \
  --query '*[].{EngineVersion:EngineVersion}' --output text
5.7.mysql_aurora.2.10.2

aws rds describe-db-engine-versions --engine aurora-mysql --engine-version 5.7.mysql_aurora.2.10.2 \
  --query '*[].[ValidUpgradeTarget]|[0][0]|[*].[EngineVersion,IsMajorVersionUpgrade]' --output text

5.7.mysql_aurora.2.10.3 False
5.7.mysql_aurora.2.11.0 False
5.7.mysql_aurora.2.11.1 False
8.0.mysql_aurora.3.01.1 True
8.0.mysql_aurora.3.02.0 True
8.0.mysql_aurora.3.02.2 True


aws rds modify-db-cluster --db-cluster-identifier mynewdbcluster \
  --engine-version 8.0.mysql_aurora.3.02.0 --no-apply-immediately --allow-major-version-upgrade
...
```

When a cluster is created without a specified maintenance window, Aurora picks a random day of the week\. In this case, the `modify-db-cluster` command is submitted on a Monday\. Thus, we change the maintenance window to be Tuesday morning\. All times are represented in the UTC time zone\. The `tue:10:00-tue:10:30` window corresponds to 2:00\-2:30 AM Pacific time\. The change in the maintenance window takes effect immediately\.

```
aws rds describe-db-clusters --db-cluster-identifier mynewdbcluster --query '*[].[PreferredMaintenanceWindow]'
[
    [
        "sat:08:20-sat:08:50"
    ]
]

aws rds modify-db-cluster --db-cluster-identifier mynewdbcluster --preferred-maintenance-window tue:10:00-tue:10:30"
aws rds describe-db-clusters --db-cluster-identifier mynewdbcluster --query '*[].[PreferredMaintenanceWindow]'
[
    [
        "tue:10:00-tue:10:30"
    ]
]
```

The following example shows how to get a report of the events generated by the upgrade\. The `--duration` argument represents the number of minutes to retrieve the event information\. This argument is needed because by default, `describe-events` only returns events from the last hour\.

```
aws rds describe-events --source-type db-cluster --source-identifier mynewdbcluster --duration 20160
{
    "Events": [
        {
            "SourceIdentifier": "mynewdbcluster",
            "SourceType": "db-cluster",
            "Message": "DB cluster created",
            "EventCategories": [
                "creation"
            ],
            "Date": "2022-11-17T01:24:11.093000+00:00",
            "SourceArn": "arn:aws:rds:us-east-1:123456789012:cluster:mynewdbcluster"
        },
        {
            "SourceIdentifier": "mynewdbcluster",
            "SourceType": "db-cluster",
            "Message": "Upgrade in progress: Performing online pre-upgrade checks.",
            "EventCategories": [
                "maintenance"
            ],
            "Date": "2022-11-18T22:57:08.450000+00:00",
            "SourceArn": "arn:aws:rds:us-east-1:123456789012:cluster:mynewdbcluster"
        },
        {
            "SourceIdentifier": "mynewdbcluster",
            "SourceType": "db-cluster",
            "Message": "Upgrade in progress: Performing offline pre-upgrade checks.",
            "EventCategories": [
                "maintenance"
            ],
            "Date": "2022-11-18T22:57:59.519000+00:00",
            "SourceArn": "arn:aws:rds:us-east-1:123456789012:cluster:mynewdbcluster"
        },
        {
            "SourceIdentifier": "mynewdbcluster",
            "SourceType": "db-cluster",
            "Message": "Upgrade in progress: Creating pre-upgrade snapshot [preupgrade-mynewdbcluster-5-7-mysql-aurora-2-10-2-to-8-0-mysql-aurora-3-02-0-2022-11-18-22-55].",
            "EventCategories": [
                "maintenance"
            ],
            "Date": "2022-11-18T23:00:22.318000+00:00",
            "SourceArn": "arn:aws:rds:us-east-1:123456789012:cluster:mynewdbcluster"
        },
        {
            "SourceIdentifier": "mynewdbcluster",
            "SourceType": "db-cluster",
            "Message": "Upgrade in progress: Cloning volume.",
            "EventCategories": [
                "maintenance"
            ],
            "Date": "2022-11-18T23:01:45.428000+00:00",
            "SourceArn": "arn:aws:rds:us-east-1:123456789012:cluster:mynewdbcluster"
        },
        {
            "SourceIdentifier": "mynewdbcluster",
            "SourceType": "db-cluster",
            "Message": "Upgrade in progress: Purging undo records for old row versions. Records remaining: 164",
            "EventCategories": [
                "maintenance"
            ],
            "Date": "2022-11-18T23:02:25.141000+00:00",
            "SourceArn": "arn:aws:rds:us-east-1:123456789012:cluster:mynewdbcluster"
        },
        {
            "SourceIdentifier": "mynewdbcluster",
            "SourceType": "db-cluster",
            "Message": "Upgrade in progress: Purging undo records for old row versions. Records remaining: 164",
            "EventCategories": [
                "maintenance"
            ],
            "Date": "2022-11-18T23:06:23.036000+00:00",
            "SourceArn": "arn:aws:rds:us-east-1:123456789012:cluster:mynewdbcluster"
        },
        {
            "SourceIdentifier": "mynewdbcluster",
            "SourceType": "db-cluster",
            "Message": "Upgrade in progress: Upgrading database objects.",
            "EventCategories": [
                "maintenance"
            ],
            "Date": "2022-11-18T23:06:48.208000+00:00",
            "SourceArn": "arn:aws:rds:us-east-1:123456789012:cluster:mynewdbcluster"
        },
        {
            "SourceIdentifier": "mynewdbcluster",
            "SourceType": "db-cluster",
            "Message": "Database cluster major version has been upgraded",
            "EventCategories": [
                "maintenance"
            ],
            "Date": "2022-11-18T23:10:28.999000+00:00",
            "SourceArn": "arn:aws:rds:us-east-1:123456789012:cluster:mynewdbcluster"
        }
    ]
}
```

## Alternative blue\-green upgrade technique<a name="AuroraMySQL.UpgradingMajor.BlueGreen"></a>

In some situations, your top priority is to perform an immediate switchover from the old cluster to an upgraded one\. In such situations, you can use a multistep process that runs the old and new clusters side\-by\-side\. Here, you replicate data from the old cluster to the new one until you are ready for the new cluster to take over\. For details, see [Using Amazon RDS Blue/Green Deployments for database updates](blue-green-deployments.md)\.