# Serverless-project
My serverless project to learn more about using API Gateway, Lambda, and DynamoDB

This is my first standalone project to implement some of the practical knowledge that I have learned through my certification and online studies!

This is a microservice product test using API Gateway a backend on Lambda and a database on DynamoDB.

I used this service to insert data, read the data, update the data, and delete data.


![Serverless project graphic](https://github.com/user-attachments/assets/a23d5d71-bb0a-4c22-a463-0fa9e18dc0af)

## Setup

### Creating the Lambda IAM Role

First I created an execution role that gives me function permission to access the AWS resources we need.

To create the role I went through the following steps
1. Open the roles page in the IAM console.
2. Choose Create a role.
3. Create a role with the following properties.
   - Trusted Entity - Lambda
   - Role name - lambda-apigateway-role.
   - Permissions - Use a custom policy with permission to DynamoDB and CloudWatch Logs. The following policy will give permissions to the functions we need to write data to DynamoDB and upload logs.

   ```
   {
   "Version": "2012-10-17",
   "Statement": [
   {
     "Sid": "Stmt1428341300017",
     "Action": [
       "dynamodb:DeleteItem",
       "dynamodb:GetItem",
       "dynamodb:PutItem",
       "dynamodb:Query",
       "dynamodb:Scan",
       "dynamodb:UpdateItem"
   ],
     "Effect": "Allow",
     "Resource": "*"
    },
    {
       "Sid": "",
       "Resource": "*",
       "Action": [
       "logs:CreateLogGroup",
       "logs:CreateLogStream",
       "logs:PutLogEvents"
     ],
     "Effect": "Allow"
   }
   ]
   }
```


# Create Lambda Function

## To create the function

1. Click "Create function" in AWS Console.
2. Selected "Author from scratch". We used the name **LambdaFunctionOverHttps**, then **Python 3.12** as Runtime. Under Permissions, we use the existing role that we created named **lambda-apigateway-role** in the drop down.
3. Create the function
4. Replace the boilerplate coding with the following code snippet and click "Save".

```
from __future__ import print_function

import boto3
import json

print('Loading function')


def lambda_handler(event, context):
    '''Provide an event that contains the following keys:

      - operation: one of the operations in the operations dict below
      - tableName: required for operations that interact with DynamoDB
      - payload: a parameter to pass to the operation being performed
    '''
    #print("Received event: " + json.dumps(event, indent=2))

    operation = event['operation']

    if 'tableName' in event:
        dynamo = boto3.resource('dynamodb').Table(event['tableName'])

    operations = {
        'create': lambda x: dynamo.put_item(**x),
        'read': lambda x: dynamo.get_item(**x),
        'update': lambda x: dynamo.update_item(**x),
        'delete': lambda x: dynamo.delete_item(**x),
        'list': lambda x: dynamo.scan(**x),
        'echo': lambda x: x,
        'ping': lambda x: 'pong'
    }

    if operation in operations:
        return operations[operation](event.get('payload'))
    else:
        raise ValueError('Unrecognized operation "{}"'.format(operation))

```
