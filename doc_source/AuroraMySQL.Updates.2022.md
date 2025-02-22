# Aurora MySQL database engine updates 2018\-06\-04 \(version 2\.02\.2\) \(deprecated\)<a name="AuroraMySQL.Updates.2022"></a>

**Version:** 2\.02\.2

Aurora MySQL 2\.02\.2 is generally available\. Aurora MySQL 2\.x versions are compatible with MySQL 5\.7 and Aurora MySQL 1\.x versions are compatible with MySQL 5\.6\.

When creating a new Aurora MySQL DB cluster, you can choose compatibility with either MySQL 5\.7 or MySQL 5\.6\. When restoring a MySQL 5\.6\-compatible snapshot, you can choose compatibility with either MySQL 5\.7 or MySQL 5\.6\.

You can restore snapshots of Aurora MySQL 1\.14\*, 1\.15\*, 1\.16\*, 1\.17\*, 2\.01\*, and 2\.02 into Aurora MySQL 2\.02\.2\. You can also perform an in\-place upgrade from Aurora MySQL 2\.01\* or 2\.02 to Aurora MySQL 2\.02\.2\.

We don't allow in\-place upgrade of Aurora MySQL 1\.\* clusters into Aurora MySQL 2\.02\.2 or restore to Aurora MySQL 2\.02\.2 from an Amazon S3 backup\. We plan to remove these restrictions in a later Aurora MySQL 2\.\* release\.

The performance schema is disabled for this release of Aurora MySQL 5\.7\. Upgrade to Aurora 2\.03 for performance schema support\.

**Note**  
 This version is currently not available in the AWS GovCloud \(US\-West\) \[us\-gov\-west\-1\] and China \(Beijing\) \[cn\-north\-1\] regions\. There will be a separate announcement once it is made available\. 

If you have any questions or concerns, AWS Support is available on the community forums and through [AWS Premium Support](http://aws.amazon.com/support)\. For more information, see [Maintaining an Amazon Aurora DB cluster](USER_UpgradeDBInstance.Maintenance.md)\.

## Comparison with Aurora MySQL 5\.6<a name="AuroraMySQL.Updates.2022.Compare56"></a>

The following Amazon Aurora MySQL features are supported in Aurora MySQL 5\.6, but these features are currently not supported in Aurora MySQL 5\.7\.
+ Asynchronous key prefetch \(AKP\)\. For more information, see [Optimizing Amazon Aurora indexed join queries with asynchronous key prefetch](AuroraMySQL.BestPractices.md#Aurora.BestPractices.AKP)\.
+ Hash joins\. For more information, see [Optimizing large Aurora MySQL join queries with hash joins](AuroraMySQL.BestPractices.md#Aurora.BestPractices.HashJoin)\.
+ Native functions for synchronously invoking AWS Lambda functions\. For more information, see [Invoking a Lambda function with an Aurora MySQL native function](AuroraMySQL.Integrating.Lambda.md#AuroraMySQL.Integrating.NativeLambda)\.
+ Scan batching\. For more information, see [Aurora MySQL database engine updates 2017\-12\-11 \(version 1\.16\) \(deprecated\)](AuroraMySQL.Updates.20171211.md)\.
+ Migrating data from MySQL using an Amazon S3 bucket\. For more information, see [Migrating data from MySQL by using an Amazon S3 bucket](AuroraMySQL.Migrating.ExtMySQL.md#AuroraMySQL.Migrating.ExtMySQL.S3)\.

Currently, Aurora MySQL 2\.01 does not support features added in Aurora MySQL version 1\.16 and later\. For information about Aurora MySQL version 1\.16, see [Aurora MySQL database engine updates 2017\-12\-11 \(version 1\.16\) \(deprecated\)](AuroraMySQL.Updates.20171211.md)\.

## MySQL 5\.7 compatibility<a name="AuroraMySQL.Updates.2022.Compatibility"></a>

Aurora MySQL 2\.02\.2 is wire\-compatible with MySQL 5\.7 and includes features such as JSON support, spatial indexes, and generated columns\. Aurora MySQL uses a native implementation of spatial indexing using z\-order curves to deliver >20x better write performance and >10x better read performance than MySQL 5\.7 for spatial datasets\.

Aurora MySQL 2\.02\.2 does not currently support the following MySQL 5\.7 features:
+ Global transaction identifiers \(GTIDs\)\. Aurora MySQL supports GTIDs in version 2\.04 and higher\.
+ Group replication plugin
+ Increased page size
+ InnoDB buffer pool loading at startup
+ InnoDB full\-text parser plugin
+ Multisource replication
+ Online buffer pool resizing
+ Password validation plugin
+ Query rewrite plugins
+ Replication filtering
+ The `CREATE TABLESPACE` SQL statement
+ X Protocol

## CLI differences between Aurora MySQL 2\.x and Aurora MySQL 1\.x<a name="AuroraMySQL.Updates.20180206.CLI"></a>
+ The engine name for Aurora MySQL 2\.x is `aurora-mysql`; the engine name for Aurora MySQL 1\.x continues to be `aurora`\.
+ The engine version for Aurora MySQL 2\.x is `5.7.12`; the engine version for Aurora MySQL 1\.x continues to be `5.6.10ann`\.
+ The default parameter group for Aurora MySQL 2\.x is `default.aurora-mysql5.7`; the default parameter group for Aurora MySQL 1\.x continues to be `default.aurora5.6`\.
+ The DB cluster parameter group family name for Aurora MySQL 2\.x is `aurora-mysql5.7`; the DB cluster parameter group family name for Aurora MySQL 1\.x continues to be `aurora5.6`\.

Refer to the Aurora documentation for the full set of CLI commands and differences between Aurora MySQL 2\.x and Aurora MySQL 1\.x\.

## Improvements<a name="AuroraMySQL.Updates.2022.Improvements"></a>
+ Fixed an issue where an Aurora Writer can occasionally restart when tracking Aurora Replica progress\.
+ Fixed an issue where an Aurora Replica restarts or throws an error when a partitioned table is accessed after running index create or drop statements on the table on the Aurora Writer\.
+ Fixed an issue where a table on an Aurora Replica is inaccessible while it is applying the changes caused by running ALTER table ADD/DROP column statements on the Aurora Writer\.