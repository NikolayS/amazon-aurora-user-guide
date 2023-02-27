# Preparing for Amazon Aurora MySQL\-Compatible Edition version 1 end of life<a name="Aurora.MySQL56.EOL"></a>

Amazon Aurora MySQL\-Compatible Edition version 1 \(with MySQL 5\.6 compatibility\) is planned to reach end of life on February 28, 2023\. Amazon advises that you upgrade all clusters \(provisioned and Aurora Serverless\) running Aurora MySQL version 1 to Aurora MySQL version 2 \(with MySQL 5\.7 compatibility\) or Aurora MySQL version 3 \(with MySQL 8\.0 compatibility\)\. Do this before Aurora MySQL version 1 reaches the end of its support period\.

For Aurora provisioned DB clusters, you can complete upgrades from Aurora MySQL version 1 to Aurora MySQL version 2 by several methods\. You can find instructions for the in\-place upgrade mechanism in [How to perform an in\-place upgrade](AuroraMySQL.Updates.MajorVersionUpgrade.md#AuroraMySQL.Upgrading.Procedure)\. Another way to complete the upgrade is to take a snapshot of an Aurora MySQL version 1 cluster and restore the snapshot to an Aurora MySQL version 2 cluster\. Or you can follow a multistep process that runs the old and new clusters side by side\. For more details about each method, see [Upgrading from Aurora MySQL 1\.x to 2\.x](AuroraMySQL.Updates.MajorVersionUpgrade.md#AuroraMySQL.Updates.MajorVersionUpgrade.1to2)\.

For Aurora Serverless v1 DB clusters, you can perform an in\-place upgrade from Aurora MySQL version 1 to Aurora MySQL version 2\. For more details about this method, see [Modifying an Aurora Serverless v1 DB cluster](aurora-serverless.modifying.md)\.

For Aurora provisioned DB clusters, you can complete upgrades from Aurora MySQL version 1 to Aurora MySQL version 3 by using a two\-stage upgrade process:

1. Upgrade from Aurora MySQL version 1 to Aurora MySQL version 2 using the methods described preceding\.

1. Upgrade from Aurora MySQL version 2 to Aurora MySQL version 3 using the same methods as for upgrading from version 1 to version 2\. For more details, see [Upgrading from Aurora MySQL 2\.x to 3\.x](AuroraMySQL.Updates.MajorVersionUpgrade.md#AuroraMySQL.Updates.MajorVersionUpgrade.2to3)\. Note the [Feature differences between Aurora MySQL version 2 and 3](Aurora.AuroraMySQL.Compare-v2-v3.md#AuroraMySQL.Compare-v2-v3-features)\.

You can find upcoming end\-of\-life dates for Aurora major versions in [Amazon Aurora versions](Aurora.VersionPolicy.md)\. Amazon automatically upgrades any clusters that you don't upgrade yourself before the end\-of\-life date\. After the end\-of\-life date, these automatic upgrades to the subsequent major version occur during a scheduled maintenance window for clusters\. 

The following are additional milestones for upgrading Aurora MySQL version 1 clusters \(provisioned and Aurora Serverless\) that are reaching end of life\. For each, the start time is 00:00 Universal Coordinated Time \(UTC\)\. 

1. Now through February 28, 2023 – You can at any time start upgrades of Aurora MySQL version 1 \(with MySQL 5\.6 compatibility\) clusters to Aurora MySQL version 2 \(with MySQL 5\.7 compatibility\)\. From Aurora MySQL version 2, you can do a further upgrade to Aurora MySQL version 3 \(with MySQL 8\.0 compatibility\) for Aurora provisioned DB clusters\. 

1. January 16, 2023 – After this time, you can't create new Aurora MySQL version 1 clusters or instances from either the AWS Management Console or the AWS Command Line Interface \(AWS CLI\)\. You also can't add new secondary Regions to an Aurora global database\. This might affect your ability to recover from an unplanned outage as outlined in [Recovering an Amazon Aurora global database from an unplanned outage](aurora-global-database-disaster-recovery.md#aurora-global-database-failover), because you can't complete steps 5 and 6 after this time\. You will also be unable to create a new cross\-Region read replica running Aurora MySQL version 1\. You can still do the following for existing Aurora MySQL version 1 clusters until February 28, 2023:
   + Restore a snapshot taken of an Aurora MySQL version 1 cluster to the same version as the original snapshot cluster\.
   + Add read replicas \(not applicable for Aurora Serverless DB clusters\)\.
   + Change instance configuration\.
   + Perform point\-in\-time restore\.
   + Create clones of existing version 1 clusters\.
   + Create a new cross\-Region read replica running Aurora MySQL version 2 or higher\.

1.  February 28, 2023 – After this time, we plan to automatically upgrade Aurora MySQL version 1 clusters to the default version of Aurora MySQL version 2 within a scheduled maintenance window that follows\. Restoring Aurora MySQL version 1 DB snapshots results in an automatic upgrade of the restored cluster to the default version of Aurora MySQL version 2 at that time\. 

Upgrading between major versions requires more extensive planning and testing than for a minor version\. The process can take substantial time\. For a detailed discussion of the v1\-to\-v2 upgrade process, see [Upgrade Amazon Aurora MySQL\-Compatible Edition version 1 \(with MySQL 5\.6 compatibility\)](http://aws.amazon.com/blogs/database/upgrade-amazon-aurora-mysql-compatible-edition-version-1-with-mysql-5-6-compatibility)\.

For situations where the top priority is to reduce downtime, you can also use [blue/green deployments](https://aws.amazon.com/blogs/aws/new-fully-managed-blue-green-deployments-in-amazon-aurora-and-amazon-rds/) for performing the major version upgrade in provisioned Amazon Aurora DB clusters\. A blue/green deployment creates a staging environment that copies the production environment\. You can make changes to the Aurora DB cluster in the green \(staging\) environment without affecting production workloads\. The switchover typically takes under a minute with no data loss and no need for application changes\. For more information, see [Overview of Amazon RDS Blue/Green Deployments for Aurora](blue-green-deployments-overview.md)\.

After the upgrade is finished, you also might have follow\-up work to do\. For example, you might need to follow up due to differences in SQL compatibility, the way certain MySQL\-related features work, or parameter settings between the old and new versions\.

To learn more about the methods, planning, testing, and troubleshooting of Aurora MySQL major version upgrades, be sure to thoroughly read [Upgrading the major version of an Amazon Aurora MySQL DB cluster](AuroraMySQL.Updates.MajorVersionUpgrade.md)\.

## Finding clusters affected by this end\-of\-life process<a name="find-cluster"></a>

To find clusters affected by this end\-of\-life process, use the following procedures\.

**Important**  
Be sure to perform these instructions in every AWS Region and for each AWS account where your resources are located\.

### Console<a name="aurora-find-mysqlv1-cluster.CON"></a>

**To find an Aurora MySQL version 1 cluster**

1. Sign in to the AWS Management Console and open the Amazon RDS console at [https://console\.aws\.amazon\.com/rds/](https://console.aws.amazon.com/rds/)\.

1.  In the navigation pane, choose **Databases**\.

1.  In the **Filter by databases** box, enter **5\.6**\.

1. Check for Aurora MySQL in the engine column\.

### AWS CLI<a name="aurora-find-mysqlv1-cluster.CLI"></a>

To find clusters affected by this end\-of\-life process using the AWS CLI, call the [describe\-db\-clusters](https://docs.aws.amazon.com/cli/latest/reference/rds/describe-db-clusters.html) command\. You can use the sample script following\.

**Example**  

```
aws rds describe-db-clusters --include-share --query 'DBClusters[?Engine==`aurora`].{EV:EngineVersion, DBCI:DBClusterIdentifier, EM:EngineMode}' --output table --region us-east-1     
        
        +------------------------------------------+
        |            DescribeDBClusters            |
        +---------------+--------------+-----------+
        |     DBCI      |     EM       |    EV     |
        +---------------+--------------+-----------+
        |  my-database-1|  serverless  |  5.6.10a  |
        +---------------+--------------+-----------+
```

### RDS API<a name="Aurora-find-mysqlv1-cluster.API"></a>

To find Aurora MySQL DB clusters running Aurora MySQL version 1, use the RDS [DescribeDBClusters](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_DescribeDBClusters.html) API operation with the following required parameters: 
+  `DescribeDBClusters`
  + Filters\.Filter\.N
    + Name
      + engine
    + Values\.Value\.N
      + \['aurora'\]