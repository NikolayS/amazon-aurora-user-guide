# Overview of parameter groups<a name="parameter-groups-overview"></a>

A *DB cluster parameter group* acts as a container for engine configuration values that apply to every DB instance in an Aurora DB cluster\. For example, the Aurora shared storage model requires that every DB instance in an Aurora cluster use the same setting for parameters such as `innodb_file_per_table`\. Thus, parameters that affect the physical storage layout are part of the cluster parameter group\. The DB cluster parameter group also includes default values for all instance\-level parameters\.

A *DB parameter group* acts as a container for engine configuration values that are applied to one or more DB instances\. DB parameter groups apply to DB instances in both Amazon RDS and Aurora\. These configuration settings apply to properties that can vary among the DB instances within an Aurora cluster, such as the sizes for memory buffers\. 

**Topics**
+ [Default and custom parameter groups](#parameter-groups-overview.custom)
+ [Static and dynamic DB cluster parameters](#parameter-groups-overview.db-cluster)
+ [Static and dynamic DB instance parameters](#parameter-groups-overview.db-instance)
+ [Character set parameters](#parameter-groups-overview.char-sets)
+ [Supported parameters and parameter values](#parameter-groups-overview.supported)

## Default and custom parameter groups<a name="parameter-groups-overview.custom"></a>

If you create a DB instance without specifying a DB parameter group, the DB instance uses a default DB parameter group\. Likewise, if you create an Aurora DB cluster without specifying a DB cluster parameter group, the DB cluster uses a default DB cluster parameter group\. Each default parameter group contains database engine defaults and Amazon RDS system defaults based on the engine, compute class, and allocated storage of the instance\.

You can't modify the parameter settings of a default parameter group\. Instead, you can do the following:

1. Create a new parameter group\.

1. Change the settings of your desired parameters\. Not all DB engine parameters in a parameter group are eligible to be modified\.

1. Modify your DB instance or DB cluster to use the custom parameter group\. For information about modifying a DB cluster or DB instance, see [Modifying an Amazon Aurora DB cluster](Aurora.Modifying.md)\.
**Note**  
If you have modified your DB instance to use a custom parameter group, and you start the DB instance, RDS automatically reboots the DB instance as part of the startup process\.

If you update parameters within a DB parameter group, the changes apply to all DB instances that are associated with that parameter group\. Likewise, if you update parameters within an Aurora DB cluster parameter group, the changes apply to all Aurora DB clusters that are associated with that DB cluster parameter group\.

If you don't want to create a parameter group from scratch, you can copy an existing parameter group with the AWS CLI [copy\-db\-parameter\-group](https://docs.aws.amazon.com/cli/latest/reference/rds/copy-db-parameter-group.html) command or [copy\-db\-cluster\-parameter\-group](https://docs.aws.amazon.com/cli/latest/reference/rds/copy-db-cluster-parameter-group.html) command\. You might find that copying a parameter group is useful in some cases\. For example, you might want to include most of an existing parameter group's custom parameters and values in a new parameter group\.

## Static and dynamic DB cluster parameters<a name="parameter-groups-overview.db-cluster"></a>

DB cluster parameters are either static or dynamic\. They differ in the following ways:
+ When you change a static parameter and save the DB cluster parameter group, the parameter change takes effect after you manually reboot the DB instances in each associated DB cluster\. When you use the AWS Management Console to change static DB cluster parameter values, it always uses `pending-reboot` for the `ApplyMethod`\.
+ When you change a dynamic parameter, by default the parameter change takes effect immediately, without requiring a reboot\. When you use the console, it always uses `immediate` for the `ApplyMethod`\. To defer the parameter change until after you reboot the DB instances in an associated DB cluster, use the AWS CLI or RDS API\. Set the `ApplyMethod` to `pending-reboot` for the parameter change\.

For more information about using the AWS CLI to change a parameter value, see [modify\-db\-cluster\-parameter\-group](https://docs.aws.amazon.com/cli/latest/reference/rds/modify-db-cluster-parameter-group.html)\. For more information about using the RDS API to change a parameter value, see [ModifyDBClusterParameterGroup](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_ModifyDBClusterParameterGroup.html)\.

If you change the DB cluster parameter group associated with a DB cluster, reboot the DB instances in the DB cluster\. The reboot applies the changes to all DB instances in the DB cluster\. To determine whether the DB instances of a DB cluster must be rebooted to apply changes, run the following AWS CLI command\.

`aws rds describe-db-clusters --db-cluster-identifier db_cluster_identifier`

Check the `DBClusterParameterGroupStatus` value for the primary DB instance in the output\. If the value is `pending-reboot`, then reboot the DB instances of the DB cluster\.

## Static and dynamic DB instance parameters<a name="parameter-groups-overview.db-instance"></a>

DB instance parameters are either static or dynamic\. They differ as follows:
+ When you change a static parameter and save the DB parameter group, the parameter change takes effect after you manually reboot the associated DB instances\. For static parameters, the console always uses `pending-reboot` for the `ApplyMethod`\.
+ When you change a dynamic parameter, by default the parameter change takes effect immediately, without requiring a reboot\. When you use the AWS Management Console to change DB instance parameter values, it always uses `immediate` for the `ApplyMethod` for dynamic parameters\. To defer the parameter change until after you reboot an associated DB instance, use the AWS CLI or RDS API\. Set the `ApplyMethod` to `pending-reboot` for the parameter change\.

For more information about using the AWS CLI to change a parameter value, see [modify\-db\-parameter\-group](https://docs.aws.amazon.com/cli/latest/reference/rds/modify-db-parameter-group.html)\. For more information about using the RDS API to change a parameter value, see [ModifyDBParameterGroup](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_ModifyDBParameterGroup.html)\.

When you associate a new DB parameter group with a DB instance, RDS applies the modified static and dynamic parameters only after the DB instance is rebooted\. However, if you modify dynamic parameters in the DB parameter group after you associate it with the DB instance, these changes are applied immediately without a reboot\. For more information about changing the DB parameter group, see [Modifying an Amazon Aurora DB cluster](Aurora.Modifying.md)\.

If a DB instance isn't using the latest changes to its associated DB parameter group, the console shows a status of **pending\-reboot** for the DB parameter group\. This status doesn't result in an automatic reboot during the next maintenance window\. To apply the latest parameter changes to that DB instance, manually reboot the DB instance\.

## Character set parameters<a name="parameter-groups-overview.char-sets"></a>

Before you create a DB cluster, set any parameters that relate to the character set or collation of your database in your parameter group\. Also do so before you create a database in it\. In this way, you ensure that the default database and new databases use the character set and collation values that you specify\. If you change character set or collation parameters, the parameter changes aren't applied to existing databases\.

For some DB engines, you can change character set or collation values for an existing database using the `ALTER DATABASE` command, for example:

```
ALTER DATABASE database_name CHARACTER SET character_set_name COLLATE collation;
```

For more information about changing the character set or collation values for a database, check the documentation for your DB engine\.

## Supported parameters and parameter values<a name="parameter-groups-overview.supported"></a>

To determine the supported parameters for your DB engine, view the parameters in the DB parameter group and DB cluster parameter group used by the DB instance or DB cluster\. For more information, see [Viewing parameter values for a DB parameter group](USER_WorkingWithDBInstanceParamGroups.md#USER_WorkingWithParamGroups.Viewing) and [Viewing parameter values for a DB cluster parameter group](USER_WorkingWithDBClusterParamGroups.md#USER_WorkingWithParamGroups.ViewingCluster)\.

In many cases, you can specify integer and Boolean parameter values using expressions, formulas, and functions\. Functions can include a mathematical log expression\. However, not all parameters support expressions, formulas, and functions for parameter values\. For more information, see [Specifying DB parameters](USER_ParamValuesRef.md)\.

For an Aurora global database, you can specify different configuration settings for the individual Aurora clusters\. Make sure that the settings are similar enough to produce consistent behavior if you promote a secondary cluster to be the primary cluster\. For example, use the same settings for time zones and character sets across all the clusters of an Aurora global database\.

Improperly setting parameters in a parameter group can have unintended adverse effects, including degraded performance and system instability\. Always be cautious when modifying database parameters, and back up your data before modifying a parameter group\. Try parameter group setting changes on a test DB instance or DB cluster before applying those parameter group changes to a production DB instance or DB cluster\.