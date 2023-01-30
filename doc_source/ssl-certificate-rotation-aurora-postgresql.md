# Updating applications to connect to Aurora PostgreSQL DB clusters using new SSL/TLS certificates<a name="ssl-certificate-rotation-aurora-postgresql"></a>

As of September 19, 2019, Amazon RDS has published new Certificate Authority \(CA\) certificates for connecting to your Aurora DB clusters using Secure Socket Layer or Transport Layer Security \(SSL/TLS\)\. Following, you can find information about updating your applications to use the new certificates\.

This topic can help you to determine whether any client applications use SSL/TLS to connect to your DB clusters\. If they do, you can further check whether those applications require certificate verification to connect\. 

**Note**  
Some applications are configured to connect to Aurora PostgreSQL DB clusters only if they can successfully verify the certificate on the server\.   
For such applications, you must update your client application trust stores to include the new CA certificates\. 

After you update your CA certificates in the client application trust stores, you can rotate the certificates on your DB clusters\. We strongly recommend testing these procedures in a development or staging environment before implementing them in your production environments\.

For more information about certificate rotation, see [Rotating your SSL/TLS certificate](UsingWithRDS.SSL-certificate-rotation.md)\. For more information about downloading certificates, see [Using SSL/TLS to encrypt a connection to a DB cluster](UsingWithRDS.SSL.md)\. For information about using SSL/TLS with PostgreSQL DB clusters, see [Securing Aurora PostgreSQL data with SSL/TLS](AuroraPostgreSQL.Security.md#AuroraPostgreSQL.Security.SSL)\.

**Topics**
+ [Determining whether applications are connecting to Aurora PostgreSQL DB clusters using SSL](#ssl-certificate-rotation-aurora-postgresql.determining-server)
+ [Determining whether a client requires certificate verification in order to connect](#ssl-certificate-rotation-aurora-postgresql.determining-client)
+ [Updating your application trust store](#ssl-certificate-rotation-aurora-postgresql.updating-trust-store)
+ [Using SSL/TLS connections for different types of applications](#ssl-certificate-rotation-aurora-postgresql.applications)

## Determining whether applications are connecting to Aurora PostgreSQL DB clusters using SSL<a name="ssl-certificate-rotation-aurora-postgresql.determining-server"></a>

Check the DB cluster configuration for the value of the `rds.force_ssl` parameter\. By default, the `rds.force_ssl` parameter is set to `0` \(off\)\. If the `rds.force_ssl` parameter is set to `1` \(on\), clients are required to use SSL/TLS for connections\. For more information about parameter groups, see [Working with parameter groups](USER_WorkingWithParamGroups.md)\.

If `rds.force_ssl` isn't set to `1` \(on\), query the `pg_stat_ssl` view to check connections using SSL\. For example, the following query returns only SSL connections and information about the clients using SSL\.

```
select datname, usename, ssl, client_addr from pg_stat_ssl inner join pg_stat_activity on pg_stat_ssl.pid = pg_stat_activity.pid where ssl is true and usename<>'rdsadmin';
```

Only rows using SSL/TLS connections are displayed with information about the connection\. The following is sample output\.

```
 datname  | usename | ssl | client_addr
----------+---------+-----+-------------
 benchdb  | pgadmin | t   | 53.95.6.13
 postgres | pgadmin | t   | 53.95.6.13
(2 rows)
```

The preceding query displays only the current connections at the time of the query\. The absence of results doesn't indicate that no applications are using SSL connections\. Other SSL connections might be established at a different time\.

## Determining whether a client requires certificate verification in order to connect<a name="ssl-certificate-rotation-aurora-postgresql.determining-client"></a>

When a client, such as psql or JDBC, is configured with SSL support, the client first tries to connect to the database with SSL by default\. If the client can't connect with SSL, it reverts to connecting without SSL\. The default `sslmode` mode used is different between libpq\-based clients \(such as psql\) and JDBC\. The libpq\-based clients default to `prefer`, where JDBC clients default to `verify-full`\. The certificate on the server is verified only when `sslrootcert` is provided with `sslmode` set to `require`, `verify-ca`, or `verify-full`\. An error is thrown if the certificate is invalid\.

Use `PGSSLROOTCERT` to verify the certificate with the `PGSSLMODE` environment variable, with `PGSSLMODE` set to `require`, `verify-ca`, or `verify-full`\. 

```
PGSSLMODE=require PGSSLROOTCERT=/fullpath/rds-ca-2019-root.pem psql -h pgdbidentifier.cxxxxxxxx.us-east-2.rds.amazonaws.com -U primaryuser -d postgres
```

Use the `sslrootcert` argument to verify the certificate with `sslmode` in connection string format, with `sslmode` set to `require`, `verify-ca`, or `verify-full`\. 

```
psql "host=pgdbidentifier.cxxxxxxxx.us-east-2.rds.amazonaws.com sslmode=require sslrootcert=/full/path/rds-ca-2019-root.pem user=primaryuser dbname=postgres"
```

For example, in the preceding case, if you use an invalid root certificate, you see an error similar to the following on your client\.

```
psql: SSL error: certificate verify failed
```

## Updating your application trust store<a name="ssl-certificate-rotation-aurora-postgresql.updating-trust-store"></a>

For information about updating the trust store for PostgreSQL applications, see [Secure TCP/IP connections with SSL](https://www.postgresql.org/docs/current/ssl-tcp.html) in the PostgreSQL documentation\.

**Note**  
When you update the trust store, you can retain older certificates in addition to adding the new certificates\.

### Updating your application trust store for JDBC<a name="ssl-certificate-rotation-aurora-postgresql.updating-trust-store.jdbc"></a>

You can update the trust store for applications that use JDBC for SSL/TLS connections\.

For information about downloading the root certificate, see [Using SSL/TLS to encrypt a connection to a DB cluster](UsingWithRDS.SSL.md)\.

For sample scripts that import certificates, see [Sample script for importing certificates into your trust store](UsingWithRDS.SSL-certificate-rotation.md#UsingWithRDS.SSL-certificate-rotation-sample-script)\.

## Using SSL/TLS connections for different types of applications<a name="ssl-certificate-rotation-aurora-postgresql.applications"></a>

The following provides information about using SSL/TLS connections for different types of applications:
+ **psql**

  The client is invoked from the command line by specifying options either as a connection string or as environment variables\. For SSL/TLS connections, the relevant options are `sslmode` \(environment variable `PGSSLMODE`\), `sslrootcert` \(environment variable `PGSSLROOTCERT`\)\.

  For the complete list of options, see [Parameter key words](https://www.postgresql.org/docs/11/libpq-connect.html#LIBPQ-PARAMKEYWORDS) in the PostgreSQL documentation\. For the complete list of environment variables, see [Environment variables](https://www.postgresql.org/docs/current/libpq-envars.html) in the PostgreSQL documentation\.
+ **pgAdmin**

  This browser\-based client is a more user\-friendly interface for connecting to a PostgreSQL database\.

  For information about configuring connections, see the [pgAdmin documentation](https://www.pgadmin.org/docs/pgadmin4/latest/server_dialog.html)\.
+ **JDBC**

  JDBC enables database connections with Java applications\.

  For general information about connecting to a PostgreSQL database with JDBC, see [Connecting to the database](https://jdbc.postgresql.org/documentation/use/) in the PostgreSQL documentation\. For information about connecting with SSL/TLS, see [Configuring the client](https://jdbc.postgresql.org/documentation/ssl/) in the PostgreSQL documentation\. 
+ **Python**

  A popular Python library for connecting to PostgreSQL databases is `psycopg2`\.

  For information about using `psycopg2`, see the [psycopg2 documentation](https://pypi.org/project/psycopg2/)\. For a short tutorial on how to connect to a PostgreSQL database, see [Psycopg2 tutorial](https://wiki.postgresql.org/wiki/Psycopg2_Tutorial)\. You can find information about the options the connect command accepts in [The psycopg2 module content](https://www.psycopg.org/docs/module.html)\.

**Important**  
After you have determined that your database connections use SSL/TLS and have updated your application trust store, you can update your database to use the rds\-ca\-2019 certificates\. For instructions, see step 3 in [Updating your CA certificate by modifying your DB instance](UsingWithRDS.SSL-certificate-rotation.md#UsingWithRDS.SSL-certificate-rotation-updating)\.