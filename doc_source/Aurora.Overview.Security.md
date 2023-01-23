# Amazon Aurora security<a name="Aurora.Overview.Security"></a>

 Security for Amazon Aurora is managed at three levels: 
+  To control who can perform Amazon RDS management actions on Aurora DB clusters and DB instances, you use AWS Identity and Access Management \(IAM\)\. When you connect to AWS using IAM credentials, your AWS account must have IAM policies that grant the permissions required to perform Amazon RDS management operations\. For more information, see [Identity and access management for Amazon Aurora](UsingWithRDS.IAM.md)\. 

   If you are using IAM to access the Amazon RDS console, you must first log on to the AWS Management Console with your user credentials, and then go to the Amazon RDS console at [https://console\.aws\.amazon\.com/rds](https://console.aws.amazon.com/rds)\. 
+  Aurora DB clusters must be created in a virtual private cloud \(VPC\) based on the Amazon VPC service\. To control which devices and Amazon EC2 instances can open connections to the endpoint and port of the DB instance for Aurora DB clusters in a VPC, you use a VPC security group\. You can make these endpoint and port connections using Transport Layer Security \(TLS\)/Secure Sockets Layer \(SSL\)\. In addition, firewall rules at your company can control whether devices running at your company can open connections to a DB instance\. For more information on VPCs, see [Amazon VPC VPCs and Amazon Aurora](USER_VPC.md)\. 
+  To authenticate logins and permissions for an Amazon Aurora DB cluster, you can take either of the following approaches, or a combination of them\. 
  +  You can take the same approach as with a stand\-alone DB instance of MySQL or PostgreSQL\. 

     Techniques for authenticating logins and permissions for stand\-alone DB instances of MySQL or PostgreSQL, such as using SQL commands or modifying database schema tables, also work with Aurora\. For more information, see [Security with Amazon Aurora MySQL](AuroraMySQL.Security.md) or [Security with Amazon Aurora PostgreSQL](AuroraPostgreSQL.Security.md)\. 
  +  You can use IAM database authentication\. 

     With IAM database authentication, you authenticate to your Aurora DB cluster by using a user or IAM role and an authentication token\. An *authentication token* is a unique value that is generated using the Signature Version 4 signing process\. By using IAM database authentication, you can use the same credentials to control access to your AWS resources and your databases\. For more information, see [IAM database authentication](UsingWithRDS.IAMDBAuth.md)\. 
  +  You can use Kerberos authentication for Aurora PostgreSQL\. 

     You can use Kerberos to authenticate users when they connect to your Aurora PostgreSQL DB cluster\. In this case, your DB cluster works with AWS Directory Service for Microsoft Active Directory to enable Kerberos authentication\. AWS Directory Service for Microsoft Active Directory is also called AWS Managed Microsoft AD\. Keeping all of your credentials in the same directory can save you time and effort\. You have a centralized place for storing and managing credentials for multiple DB clusters\. Using a directory can also improve your overall security profile\. For more information, see [Using Kerberos authentication with Aurora PostgreSQL](postgresql-kerberos.md) 

 For information about configuring security, see [Security in Amazon Aurora](UsingWithRDS.md)\. 

## Using SSL with Aurora DB clusters<a name="Aurora.Overview.Security.SSL"></a>

 Amazon Aurora DB clusters support Secure Sockets Layer \(SSL\) connections from applications using the same process and public key as Amazon RDS DB instances\. For more information, see [Security with Amazon Aurora MySQL](AuroraMySQL.Security.md), [Security with Amazon Aurora PostgreSQL](AuroraPostgreSQL.Security.md), or [Using TLS/SSL with Aurora Serverless v1](aurora-serverless.md#aurora-serverless.tls)\. 