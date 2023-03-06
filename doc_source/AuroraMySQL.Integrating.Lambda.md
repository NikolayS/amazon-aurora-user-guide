# Invoking a Lambda function from an Amazon Aurora MySQL DB cluster<a name="AuroraMySQL.Integrating.Lambda"></a><a name="lambda"></a>

You can invoke an AWS Lambda function from an Amazon Aurora MySQL\-Compatible Edition DB cluster with the native function `lambda_sync` or `lambda_async`\. Before invoking a Lambda function from an Aurora MySQL, the Aurora DB cluster must have access to Lambda\. For details about granting access to Aurora MySQL, see [Giving Aurora access to Lambda](#AuroraMySQL.Integrating.LambdaAccess)\. For information about the `lambda_sync` and `lambda_async` stored functions, see [Invoking a Lambda function with an Aurora MySQL native function](#AuroraMySQL.Integrating.NativeLambda)\. 

 You can also call an AWS Lambda function by using a stored procedure\. However, using a stored procedure is deprecated\. We strongly recommend using an Aurora MySQL native function if you are using one of the following Aurora MySQL versions: 
+ Aurora MySQL version 2, for MySQL 5\.7\-compatible clusters\.
+ Aurora MySQL version 3\.01 and higher, for MySQL 8\.0\-compatible clusters\. The stored procedure isn't available in Aurora MySQL version 3\.

**Topics**
+ [Giving Aurora access to Lambda](#AuroraMySQL.Integrating.LambdaAccess)
+ [Invoking a Lambda function with an Aurora MySQL native function](#AuroraMySQL.Integrating.NativeLambda)
+ [Invoking a Lambda function with an Aurora MySQL stored procedure \(deprecated\)](#AuroraMySQL.Integrating.ProcLambda)

## Giving Aurora access to Lambda<a name="AuroraMySQL.Integrating.LambdaAccess"></a>

Before you can invoke Lambda functions from an Aurora MySQL DB cluster, make sure to first give your cluster permission to access Lambda\.

**To give Aurora MySQL access to Lambda**

1. Create an AWS Identity and Access Management \(IAM\) policy that provides the permissions that allow your Aurora MySQL DB cluster to invoke Lambda functions\. For instructions, see [Creating an IAM policy to access AWS Lambda resources](AuroraMySQL.Integrating.Authorizing.IAM.LambdaCreatePolicy.md)\.

1. Create an IAM role, and attach the IAM policy you created in [Creating an IAM policy to access AWS Lambda resources](AuroraMySQL.Integrating.Authorizing.IAM.LambdaCreatePolicy.md) to the new IAM role\. For instructions, see [Creating an IAM role to allow Amazon Aurora to access AWS services](AuroraMySQL.Integrating.Authorizing.IAM.CreateRole.md)\.

1. Set the `aws_default_lambda_role` DB cluster parameter to the Amazon Resource Name \(ARN\) of the new IAM role\.

   If the cluster is part of an Aurora global database, apply the same setting for each Aurora cluster in the global database\. 

   For more information about DB cluster parameters, see [Amazon Aurora DB cluster and DB instance parameters](USER_WorkingWithDBClusterParamGroups.md#Aurora.Managing.ParameterGroups)\.

1. To permit database users in an Aurora MySQL DB cluster to invoke Lambda functions, associate the role that you created in [Creating an IAM role to allow Amazon Aurora to access AWS services](AuroraMySQL.Integrating.Authorizing.IAM.CreateRole.md) with the DB cluster\. For information about associating an IAM role with a DB cluster, see [Associating an IAM role with an Amazon Aurora MySQL DB cluster](AuroraMySQL.Integrating.Authorizing.IAM.AddRoleToDBCluster.md)\.

    If the cluster is part of an Aurora global database, associate the role with each Aurora cluster in the global database\. 

1. Configure your Aurora MySQL DB cluster to allow outbound connections to Lambda\. For instructions, see [Enabling network communication from Amazon Aurora MySQL to other AWS services](AuroraMySQL.Integrating.Authorizing.Network.md)\.

    If the cluster is part of an Aurora global database, enable outbound connections for each Aurora cluster in the global database\. 

## Invoking a Lambda function with an Aurora MySQL native function<a name="AuroraMySQL.Integrating.NativeLambda"></a>

**Note**  
You can call the native functions `lambda_sync` and `lambda_async` when you use Aurora MySQL version 2, or Aurora MySQL version 3\.01 and higher\. For more information about Aurora MySQL versions, see [Database engine updates for Amazon Aurora MySQL](AuroraMySQL.Updates.md)\.

You can invoke an AWS Lambda function from an Aurora MySQL DB cluster by calling the native functions `lambda_sync` and `lambda_async`\. This approach can be useful when you want to integrate your database running on Aurora MySQL with other AWS services\. For example, you might want to send a notification using Amazon Simple Notification Service \(Amazon SNS\) whenever a row is inserted into a specific table in your database\.

### Working with native functions to invoke a Lambda function<a name="AuroraMySQL.Integrating.NativeLambda.lambda_functions"></a>

The `lambda_sync` and `lambda_async` functions are built\-in, native functions that invoke a Lambda function synchronously or asynchronously\. When you must know the result of the Lambda function before moving on to another action, use the synchronous function `lambda_sync`\. When you don't need to know the result of the Lambda function before moving on to another action, use the asynchronous function `lambda_async`\.

In Aurora MySQL version 3, the user invoking a native function must be granted the `AWS_LAMBDA_ACCESS` role\. To grant this role to a user, connect to the DB instance as the administrative user, and run the following statement\.

```
GRANT AWS_LAMBDA_ACCESS TO user@domain-or-ip-address
```

You can revoke this role by running the following statement\.

```
REVOKE AWS_LAMBDA_ACCESS FROM user@domain-or-ip-address
```

**Tip**  
When you use the role technique in Aurora MySQL version 3, you also activate the role by using the `SET ROLE role_name` or `SET ROLE ALL` statement\. If you aren't familiar with the MySQL 8\.0 role system, you can learn more in [Role\-based privilege model](Aurora.AuroraMySQL.Compare-80-v3.md#AuroraMySQL.privilege-model)\. You can also find more details in [Using roles](https://dev.mysql.com/doc/refman/8.0/en/roles.html) in the *MySQL Reference Manual*\.  
This only applies to the current active session\. When you reconnect, you have to run the `SET ROLE` statement again to grant privileges\. For more information, see [SET ROLE statement](https://dev.mysql.com/doc/refman/8.0/en/set-role.html) in the *MySQL Reference Manual*\.  
You can also use the `activate_all_roles_on_login` DB cluster parameter to automatically activate all roles when a user connects to a DB instance\. When this parameter is set, you don't have to call the SET ROLE statement explicitly to activate a role\. For more information, see [activate\_all\_roles\_on\_login](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_activate_all_roles_on_login) in the *MySQL Reference Manual*\.

In Aurora MySQL version 2, the user invoking a native function must be granted the `INVOKE LAMBDA` privilege\. To grant this privilege to a user, connect to the DB instance as the administrative user, and run the following statement\.

```
GRANT INVOKE LAMBDA ON *.* TO user@domain-or-ip-address
```

You can revoke this privilege by running the following statement\.

```
REVOKE INVOKE LAMBDA ON *.* FROM user@domain-or-ip-address
```

#### Syntax for the lambda\_sync function<a name="AuroraMySQL.Integrating.NativeLambda.lambda_functions.Sync.Syntax"></a>

You invoke the `lambda_sync` function synchronously with the `RequestResponse` invocation type\. The function returns the result of the Lambda invocation in a JSON payload\. The function has the following syntax\.

```
lambda_sync (
  lambda_function_ARN,
  JSON_payload
)
```

#### Parameters for the lambda\_sync function<a name="AuroraMySQL.Integrating.NativeLambda.lambda_functions.Sync.Parameters"></a>

The `lambda_sync` function has the following parameters\.

* lambda\_function\_ARN *  
The Amazon Resource Name \(ARN\) of the Lambda function to invoke\.

* JSON\_payload *  
The payload for the invoked Lambda function, in JSON format\.

**Note**  
Aurora MySQL version 3 supports the JSON parsing functions from MySQL 8\.0\. However, Aurora MySQL version 2 doesn't include those functions\. JSON parsing isn't required when a Lambda function returns an atomic value, such as a number or a string\.

#### Example for the lambda\_sync function<a name="AuroraMySQL.Integrating.NativeLambda.lambda_functions.Sync.Example"></a>

The following query based on `lambda_sync` invokes the Lambda function `BasicTestLambda` synchronously using the function ARN\. The payload for the function is `{"operation": "ping"}`\.

```
SELECT lambda_sync(
    'arn:aws:lambda:us-east-1:123456789012:function:BasicTestLambda',
    '{"operation": "ping"}');
```

#### Syntax for the lambda\_async function<a name="AuroraMySQL.Integrating.NativeLambda.lambda_functions.Async.Syntax"></a>

You invoke the `lambda_async` function asynchronously with the `Event` invocation type\. The function returns the result of the Lambda invocation in a JSON payload\. The function has the following syntax\.

```
lambda_async (
  lambda_function_ARN,
  JSON_payload
)
```

#### Parameters for the lambda\_async function<a name="AuroraMySQL.Integrating.NativeLambda.lambda_functions.Async.Parameters"></a>

The `lambda_async` function has the following parameters\.

* lambda\_function\_ARN *  
The Amazon Resource Name \(ARN\) of the Lambda function to invoke\.

* JSON\_payload *  
The payload for the invoked Lambda function, in JSON format\.

**Note**  
Aurora MySQL version 3 supports the JSON parsing functions from MySQL 8\.0\. However, Aurora MySQL version 2 doesn't include those functions\. JSON parsing isn't required when a Lambda function returns an atomic value, such as a number or a string\.

#### Example for the lambda\_async function<a name="AuroraMySQL.Integrating.NativeLambda.lambda_functions.Async.Example"></a>

The following query based on `lambda_async` invokes the Lambda function `BasicTestLambda` asynchronously using the function ARN\. The payload for the function is `{"operation": "ping"}`\.

```
SELECT lambda_async(
    'arn:aws:lambda:us-east-1:123456789012:function:BasicTestLambda',
    '{"operation": "ping"}');
```

#### Invoking a Lambda function within a trigger<a name="AuroraMySQL.Integrating.NativeLambda.lambda_functions.trigger"></a>

You can use triggers to call Lambda on data\-modifying statements\. The following example uses the `lambda_async` native function and stores the result in a variable\.

```
mysql>SET @result=0;
mysql>DELIMITER //
mysql>CREATE TRIGGER myFirstTrigger
      AFTER INSERT
          ON Test_trigger FOR EACH ROW
      BEGIN
      SELECT lambda_async(
          'arn:aws:lambda:us-east-1:123456789012:function:BasicTestLambda',
          '{"operation": "ping"}')
          INTO @result;
      END; //
mysql>DELIMITER ;
```

**Note**  
Triggers aren't run once per SQL statement, but once per row modified, one row at a time\. When a trigger runs, the process is synchronous\. The data\-modifying statement only returns when the trigger completes\.  
Be careful when invoking an AWS Lambda function from triggers on tables that experience high write traffic\. `INSERT`, `UPDATE`, and `DELETE` triggers are activated per row\. A write\-heavy workload on a table with `INSERT`, `UPDATE`, or `DELETE` triggers results in a large number of calls to your AWS Lambda function\.

## Invoking a Lambda function with an Aurora MySQL stored procedure \(deprecated\)<a name="AuroraMySQL.Integrating.ProcLambda"></a>

You can invoke an AWS Lambda function from an Aurora MySQL DB cluster by calling the `mysql.lambda_async` procedure\. This approach can be useful when you want to integrate your database running on Aurora MySQL with other AWS services\. For example, you might want to send a notification using Amazon Simple Notification Service \(Amazon SNS\) whenever a row is inserted into a specific table in your database\. 

### Aurora MySQL version considerations<a name="AuroraMySQL.Integrating.ProcLambda.caveats"></a>

Starting in Aurora MySQL version 2, you can use the native function method instead of these stored procedures to invoke a Lambda function\. For more information about the native functions, see [Working with native functions to invoke a Lambda function](#AuroraMySQL.Integrating.NativeLambda.lambda_functions)\.

In Aurora MySQL version 2, the stored procedure `mysql.lambda_async` is no longer supported\. We strongly recommend that you work with native Lambda functions instead\. In Aurora MySQL version 3, the stored procedure isn't available\.

### Working with the mysql\.lambda\_async procedure to invoke a Lambda function \(deprecated\)<a name="AuroraMySQL.Integrating.Lambda.mysql_lambda_async"></a>

The `mysql.lambda_async` procedure is a built\-in stored procedure that invokes a Lambda function asynchronously\. To use this procedure, your database user must have `EXECUTE` privilege on the `mysql.lambda_async` stored procedure\.

#### Syntax<a name="AuroraMySQL.Integrating.Lambda.mysql_lambda_async.Syntax"></a>

The `mysql.lambda_async` procedure has the following syntax\.

```
CALL mysql.lambda_async (
  lambda_function_ARN,
  lambda_function_input
)
```

#### Parameters<a name="AuroraMySQL.Integrating.Lambda.mysql_lambda_async.Parameters"></a>

The `mysql.lambda_async` procedure has the following parameters\.

* lambda\_function\_ARN *  
The Amazon Resource Name \(ARN\) of the Lambda function to invoke\.

* lambda\_function\_input *  
The input string, in JSON format, for the invoked Lambda function\.

#### Examples<a name="AuroraMySQL.Integrating.Lambda.mysql_lambda_async.Examples"></a>

As a best practice, we recommend that you wrap calls to the `mysql.lambda_async` procedure in a stored procedure that can be called from different sources such as triggers or client code\. This approach can help to avoid impedance mismatch issues and make it easier to invoke Lambda functions\. 

**Note**  
Be careful when invoking an AWS Lambda function from triggers on tables that experience high write traffic\. `INSERT`, `UPDATE`, and `DELETE` triggers are activated per row\. A write\-heavy workload on a table with `INSERT`, `UPDATE`, or `DELETE` triggers results in a large number of calls to your AWS Lambda function\.   
Although calls to the `mysql.lambda_async` procedure are asynchronous, triggers are synchronous\. A statement that results in a large number of trigger activations doesn't wait for the call to the AWS Lambda function to complete, but it does wait for the triggers to complete before returning control to the client\.

**Example: Invoke an AWS Lambda function to send email**  
The following example creates a stored procedure that you can call in your database code to send an email using a Lambda function\.  
**AWS Lambda Function**  

```
import boto3

ses = boto3.client('ses')

def SES_send_email(event, context):

    return ses.send_email(
        Source=event['email_from'],
        Destination={
            'ToAddresses': [
            event['email_to'],
            ]
        },

        Message={
            'Subject': {
            'Data': event['email_subject']
            },
            'Body': {
                'Text': {
                    'Data': event['email_body']
                }
            }
        }
    )
```
**Stored Procedure**  

```
DROP PROCEDURE IF EXISTS SES_send_email;
DELIMITER ;;
  CREATE PROCEDURE SES_send_email(IN email_from VARCHAR(255),
                                  IN email_to VARCHAR(255),
                                  IN subject VARCHAR(255),
                                  IN body TEXT) LANGUAGE SQL
  BEGIN
    CALL mysql.lambda_async(
         'arn:aws:lambda:us-west-2:123456789012:function:SES_send_email',
         CONCAT('{"email_to" : "', email_to,
             '", "email_from" : "', email_from,
             '", "email_subject" : "', subject,
             '", "email_body" : "', body, '"}')
     );
  END
  ;;
DELIMITER ;
```
**Call the Stored Procedure to Invoke the AWS Lambda Function**  

```
mysql> call SES_send_email('example_from@amazon.com', 'example_to@amazon.com', 'Email subject', 'Email content');
```

**Example: Invoke an AWS Lambda function to publish an event from a trigger**  
The following example creates a stored procedure that publishes an event by using Amazon SNS\. The code calls the procedure from a trigger when a row is added to a table\.  
**AWS Lambda Function**  

```
import boto3

sns = boto3.client('sns')

def SNS_publish_message(event, context):

    return sns.publish(
        TopicArn='arn:aws:sns:us-west-2:123456789012:Sample_Topic',
        Message=event['message'],
        Subject=event['subject'],
        MessageStructure='string'
    )
```
**Stored Procedure**  

```
DROP PROCEDURE IF EXISTS SNS_Publish_Message;
DELIMITER ;;
CREATE PROCEDURE SNS_Publish_Message (IN subject VARCHAR(255),
                                      IN message TEXT) LANGUAGE SQL
BEGIN
  CALL mysql.lambda_async('arn:aws:lambda:us-west-2:123456789012:function:SNS_publish_message',
     CONCAT('{ "subject" : "', subject,
            '", "message" : "', message, '" }')
     );
END
;;
DELIMITER ;
```
**Table**  

```
CREATE TABLE 'Customer_Feedback' (
  'id' int(11) NOT NULL AUTO_INCREMENT,
  'customer_name' varchar(255) NOT NULL,
  'customer_feedback' varchar(1024) NOT NULL,
  PRIMARY KEY ('id')
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
**Trigger**  

```
DELIMITER ;;
CREATE TRIGGER TR_Customer_Feedback_AI
  AFTER INSERT ON Customer_Feedback
  FOR EACH ROW
BEGIN
  SELECT CONCAT('New customer feedback from ', NEW.customer_name), NEW.customer_feedback INTO @subject, @feedback;
  CALL SNS_Publish_Message(@subject, @feedback);
END
;;
DELIMITER ;
```
**Insert a Row into the Table to Trigger the Notification**  

```
mysql> insert into Customer_Feedback (customer_name, customer_feedback) VALUES ('Sample Customer', 'Good job guys!');
```