# Connecting to your DB cluster using IAM authentication and the AWS SDK for Python \(Boto3\)<a name="UsingWithRDS.IAMDBAuth.Connecting.Python"></a>

You can connect to an Aurora MySQL or Aurora PostgreSQL DB cluster with the AWS SDK for Python \(Boto3\) as described following\.

**Prerequisites**  
The following are prerequisites for connecting to your DB cluster using IAM authentication:
+ [Enabling and disabling IAM database authentication](UsingWithRDS.IAMDBAuth.Enabling.md)
+ [Creating and using an IAM policy for IAM database access](UsingWithRDS.IAMDBAuth.IAMPolicy.md)
+ [Creating a database account using IAM authentication](UsingWithRDS.IAMDBAuth.DBAccounts.md)

In addition, make sure the imported libraries in the sample code exist on your system\.

**Examples**  
The code examples use profiles for shared credentials\. For information about the specifying credentials, see [Credentials](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/credentials.html) in the AWS SDK for Python \(Boto3\) documentation\.

The following code examples show how to generate an authentication token, and then use it to connect to a DB cluster\. 

To run this code example, you need the [AWS SDK for Python \(Boto3\)](http://aws.amazon.com/sdk-for-python/), found on the AWS site\.

Modify the values of the following variables as needed:
+ `ENDPOINT` – The endpoint of the DB cluster that you want to access
+ `PORT` – The port number used for connecting to your DB cluster
+ `USER` – The database account that you want to access
+ `REGION` – The AWS Region where the DB cluster is running
+ `DBNAME` – The database that you want to access
+ `SSLCERTIFICATE` – The full path to the SSL certificate for Amazon Aurora

  For `ssl_ca`, specify an SSL certificate\. To download an SSL certificate, see [Using SSL/TLS to encrypt a connection to a DB cluster](UsingWithRDS.SSL.md)\.

This code connects to an Aurora MySQL DB cluster\.

Before running this code, install the PyMySQL driver by following the instructions in the [ Python Package Index](https://pypi.org/project/PyMySQL/)\.

```
import pymysql
import sys
import boto3
import os

ENDPOINT="mysqlcluster.cluster-123456789012.us-east-1.rds.amazonaws.com"
PORT="3306"
USER="jane_doe"
REGION="us-east-1"
DBNAME="mydb"
os.environ['LIBMYSQL_ENABLE_CLEARTEXT_PLUGIN'] = '1'

#gets the credentials from .aws/credentials
session = boto3.Session(profile_name='default')
client = session.client('rds')

token = client.generate_db_auth_token(DBHostname=ENDPOINT, Port=PORT, DBUsername=USER, Region=REGION)

try:
    conn =  pymysql.connect(host=ENDPOINT, user=USER, passwd=token, port=PORT, database=DBNAME, ssl_ca='SSLCERTIFICATE')
    cur = conn.cursor()
    cur.execute("""SELECT now()""")
    query_results = cur.fetchall()
    print(query_results)
except Exception as e:
    print("Database connection failed due to {}".format(e))
```

This code connects to an Aurora PostgreSQL DB cluster\.

Before running this code, install `psycopg2` by following the instructions in [Psycopg documentation](https://pypi.org/project/psycopg2/)\.

```
import psycopg2
import sys
import boto3
import os

ENDPOINT="postgresmycluster.cluster-123456789012.us-east-1.rds.amazonaws.com"
PORT="5432"
USER="jane_doe"
REGION="us-east-1"
DBNAME="mydb"

#gets the credentials from .aws/credentials
session = boto3.Session(profile_name='RDSCreds')
client = session.client('rds')

token = client.generate_db_auth_token(DBHostname=ENDPOINT, Port=PORT, DBUsername=USER, Region=REGION)

try:
    conn = psycopg2.connect(host=ENDPOINT, port=PORT, database=DBNAME, user=USER, password=token, sslrootcert="SSLCERTIFICATE")
    cur = conn.cursor()
    cur.execute("""SELECT now()""")
    query_results = cur.fetchall()
    print(query_results)
except Exception as e:
    print("Database connection failed due to {}".format(e))
```

If you want to connect to a DB cluster through a proxy, see [Connecting to a proxy using IAM authentication](rds-proxy-setup.md#rds-proxy-connecting-iam)\.