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

### Create Lambda Function

**To create the function**

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

### Now to test the Lambda Function

Now we will test our newly created function. Since we haven't created our DynamoDB or the API yet, we will do a simple echo operation. This function should output whatever intput we pass.
1. We click the arrow on "Select a test event" and click the "configure test events'

2. Paste the following JSON into the event. The operation dictates what the lambda function will perform for us. In this case, it would simply return the payload from input event as output. Then we click Create to save it.
```
{
    "operation": "echo",
    "payload": {
        "somekey1": "somevalue1",
        "somekey2": "somevalue2"
    }
}
```

3. Click Test and it will execute the test event. We should see the following output in the console.

```
{
    "somekey1": "somevalue1",
    "somekey2": "somevalue2"
}
```

## Create DynamoDB Table
Create the DynamoDB table that the Lambda function will use.
### To Create a DynamoDB table
    1. Open the DynamoDB console.
    2. Choose Create table.
    3. Create a table with the following settings.
        -Table name - lambda-apigateway
        -Primary Key - id (string)
    4. Choose Create.

## Create API
### To create the API
1. Go to API Gateway console
2. Click Create API
3. Select the REST API and then "Build"
4. Give the API name as "DynamoDBOperations", we will keep everything else as is and then click "Create API".
5. Now lets add a resource next.
Click "Actions", then click "Create Resource"
6. Input "DunamoDBManager" in the resource Name, Resource Path will get populated. Then click "Create Resource".
7. Let's Create a POST Method for our API. With the "/dynamodbmanager" resource selected, Click "Actions" again and click "Create Method".
8. Select "POST" from the drop down, then click checkmark
9. The integration will come up automatically with "Lambda Function" option selected. Select "LambdaFunctionOverHttps" function that we created earlier. As you start typing the name your function name will show up. Select and click "Save". A popup window will come up to add resource ploicy to the lambda to be invoked by this API. Then click "OK".
Our API-Lambda integration is now done!

## Deploy the API
in this step we deploy the API that we created to a stage called prod.
1. Click "Actions", select "Deploy API"
2. Now it is going to ask you about stage. Select "[New Stage]" for "Deployment stage". Give "Prod" as "Stage name". Click "Deploy".
3. We are now all set to run our solution! To invoke our API endpoint, we need the endpoint URL. In the "Stages" screen expand the stage "Prod", select "POST" method, and copy the "Invoke URL" from screen.

## Running our solution
1. The Lambda funtion supports using the create operation to create an item in our DynamoDB table. To request this operation, use the following JSON:

```
{
    "operation": "create",
    "tableName": "lambda-apigateway",
    "payload": {
        "Item": {
            "id": "1234ABCD",
            "number": 5
        }
    }
}
```
    2. To execute our API from our local machine I chose to use Postman. 
        - To Run this from Postman, select "POST", paste the API invoke URL. Then under "Body" select "raw" and paste the above JSON. Click "Send". API should execute and return "HTTPStatusCode" 200.

    3. To Validate that the item is inserted properly and everything was working fine into our DynamoDB table, we go to Dynamo console, select "lambda-apigateway" table, select "items" tab, and the newly inserted item should be displayed.

    4. To retrive the inserted items fro mthe table, we can use the "list" operatiom of Lambda using the same API. Use the following JSON to the API, and it will return the items from the Dynamo table.
```
{
    "operation": "list",
    "tableName": "lambda-apigateway",
    "payload": {
    }
}
```
