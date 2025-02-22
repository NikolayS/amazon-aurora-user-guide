# Client:ClientWrite<a name="apg-waits.clientwrite"></a>

The `Client:ClientWrite` event occurs when Aurora PostgreSQL is waiting to write data to the client\.

**Topics**
+ [Supported engine versions](#apg-waits.clientwrite.context.supported)
+ [Context](#apg-waits.clientwrite.context)
+ [Likely causes of increased waits](#apg-waits.clientwrite.causes)
+ [Actions](#apg-waits.clientwrite.actions)

## Supported engine versions<a name="apg-waits.clientwrite.context.supported"></a>

This wait event information is supported for Aurora PostgreSQL version 10 and higher\.

## Context<a name="apg-waits.clientwrite.context"></a>

A client process must read all of the data received from an Aurora PostgreSQL DB cluster before the cluster can send more data\. The time that the cluster waits before sending more data to the client is a `Client:ClientWrite` event\.

Reduced network throughput between the Aurora PostgreSQL DB cluster and the client can cause this event\. CPU pressure and network saturation on the client can also cause this event\. *CPU pressure* is when the CPU is fully utilized and there are tasks waiting for CPU time\. *Network saturation* is when the network between the database and client is carrying more data than it can handle\. 

## Likely causes of increased waits<a name="apg-waits.clientwrite.causes"></a>

Common causes for the `Client:ClientWrite` event to appear in top waits include the following: 

**Increased network latency**  
There might be increased network latency between the Aurora PostgreSQL DB cluster and client\. Higher network latency increases the time required for the client to receive the data\.

**Increased load on the client**  
There might be CPU pressure or network saturation on the client\. An increase in load on the client delays the reception of data from the Aurora PostgreSQL DB cluster\.

**Large volume of data sent to the client**  
The Aurora PostgreSQL DB cluster might be sending a large amount of data to the client\. A client might not be able to receive the data as fast as the cluster is sending it\. Activities such as a copy of a large table can result in an increase in `Client:ClientWrite` events\.

## Actions<a name="apg-waits.clientwrite.actions"></a>

We recommend different actions depending on the causes of your wait event\.

**Topics**
+ [Place the clients in the same Availability Zone and VPC subnet as the cluster](#apg-waits.clientwrite.actions.az-vpc-subnet)
+ [Use current generation instances](#apg-waits.clientwrite.actions.db-instance-class)
+ [Reduce the amount of data sent to the client](#apg-waits.clientwrite.actions.reduce-data)
+ [Scale your client](#apg-waits.clientwrite.actions.scale-client)

### Place the clients in the same Availability Zone and VPC subnet as the cluster<a name="apg-waits.clientwrite.actions.az-vpc-subnet"></a>

To reduce network latency and increase network throughput, place clients in the same Availability Zone and virtual private cloud \(VPC\) subnet as the Aurora PostgreSQL DB cluster\.

### Use current generation instances<a name="apg-waits.clientwrite.actions.db-instance-class"></a>

In some cases, you might not be using a DB instance class that supports jumbo frames\. If you're running your application on Amazon EC2, consider using a current generation instance for the client\. Also, configure the maximum transmission unit \(MTU\) on the client operating system\. This technique might reduce the number of network round trips and increase network throughput\. For more information, see [ Jumbo frames \(9001 MTU\)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/network_mtu.html#jumbo_frame_instances) in the *Amazon EC2 User Guide for Linux Instances*\.

For information about DB instance classes, see [Aurora DB instance classes](Concepts.DBInstanceClass.md)\. To determine the DB instance class that is equivalent to an Amazon EC2 instance type, place `db.` before the Amazon EC2 instance type name\. For example, the `r5.8xlarge` Amazon EC2 instance is equivalent to the `db.r5.8xlarge` DB instance class\.

### Reduce the amount of data sent to the client<a name="apg-waits.clientwrite.actions.reduce-data"></a>

When possible, adjust your application to reduce the amount of data that the Aurora PostgreSQL DB cluster sends to the client\. Making such adjustments relieves CPU and network contention on the client\.

### Scale your client<a name="apg-waits.clientwrite.actions.scale-client"></a>

Using Amazon CloudWatch or other host metrics, determine if your client is currently constrained by CPU or network bandwidth, or both\. If the client is constrained, scale your client accordingly\.