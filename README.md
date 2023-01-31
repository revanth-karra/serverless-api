# Building serverless API using API Gateway, Lambda, and DynamoDB

![NEWIMG](https://user-images.githubusercontent.com/52368773/215879828-3738c773-e190-43d8-8dc6-30954b0e4310.png)

Create (DynamoDBManager) and (POST) method on it. The method is backed by a Lambda function (LambdaFunctionOverHttps). That is, when we call the API through an HTTPS endpoint, Amazon API Gateway invokes the Lambda function.

The POST method on the DynamoDBManager supports the following DynamoDB operations:

1. Create, update, and delete an item.
2. Read an item.
3. Scan an item.
4. Other operations (echo, ping) can be used for testing purpose(not related to DynamoDB)

# Request payload you send in the POST request identifies the DynamoDB operation

sample request payload for a DynamoDB create item

```
{
    "operation": "create",
    "tableName": "lambda-apigateway",
    "payload": {
        "Item": {
            "id": "1",
            "name": "Bob"
        }
    }
}

```

sample request payload for a DynamoDB read item

```
{
    "operation": "read",
    "tableName": "lambda-apigateway",
    "payload": {
        "Key": {
            "id": "1"
        }
    }
}
```

# Configuration steps

o create an execution role

1. Open the roles page in the IAM console.
2. Choose Create role.
3. Create a role with the following properties.
4. Trusted entity – Lambda.
5. Role name – lambda-apigateway-role.
6. Permissions – Custom policy with permission to DynamoDB and CloudWatch Logs. This custom policy has the permissions that the function needs to write data to   DynamoDB and upload logs.

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

Steps involes:

1. Click "Create function" in AWS Lambda Console

![create-lambda-functions](https://user-images.githubusercontent.com/52368773/215882284-96c03460-5562-455c-970e-3b5ba5d9a9bd.png)

2. Select "Author from scratch". Use name LambdaFunctionOverHttps , select Python 3.8 as Runtime. Under Permissions, select "Use an existing role", and select lambda-apigateway-role that we created, from the drop down

3. Click "Create function"

![create-lambda-functionss](https://user-images.githubusercontent.com/52368773/215885630-976f2d59-7e71-457b-8a09-0b63bb5d6c14.png)

4. Replace the boilerplate coding with the following code and click "Deploy"

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
 
![lambda-function-code](https://user-images.githubusercontent.com/52368773/215887018-4cdf96c4-6cc2-4783-ab26-f12f18f379cb.png)

#   Testing Lambda Function

1. Click on "Select a test event" and click "Configure test events"

![configuring-test-event](https://user-images.githubusercontent.com/52368773/215889078-6cc3c16e-6acc-4122-8c04-7a022d825aef.png)

2. Paste the following JSON into the event. The field "operation" indictates what this lambda function will perform. Here, it's simply returns the payload from input event as output. Click "Create" to save

```

{
    "operation": "echo",
    "payload": {
        "somekey1": "somevalue1",
        "somekey2": "somevalue2"
    }
}

```

![configuring-test-event-code](https://user-images.githubusercontent.com/52368773/215889762-5dd78ba6-23fc-4fa6-8ec9-8b065667b638.png)

3. Click "Test", and it will execute the test event. You should see the output in the console

We're all set to create DynamoDB table and an API using our lambda !!!

# Create DynamoDB Table

Steps to create a DynamoDB table

1. Open the DynamoDB console.
2. Choose Create table.
3. Create a table with the following settings.
4. Table name – lambda-apigateway
5. Primary key – id (string)
6. Choose Create.

![dynamo-create-table](https://user-images.githubusercontent.com/52368773/215890798-8afcd25a-71fc-48cd-af55-19a147836c6e.png)

![dynamodb-table-creation](https://user-images.githubusercontent.com/52368773/215891199-923ed684-209a-4bcd-9ef3-d2b8facc5169.png)
