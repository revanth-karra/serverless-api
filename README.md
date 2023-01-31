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

# Create an API

Steps to create an API

1. Go to API Gateway console
2. Click Create API

![build-rest-api](https://user-images.githubusercontent.com/52368773/215892209-8994abab-cf50-4421-abe6-0b12ef41124f.png)

3. Scroll down and select "Build" for REST API

![create-first-api](https://user-images.githubusercontent.com/52368773/215892985-950bdb00-bf66-4873-a320-e93f75717cc4.png)

4. Give the API name as "DynamoDBOperations", keep everything as is, click "Create API"

![create-new-api](https://user-images.githubusercontent.com/52368773/215893479-efae9b84-b4ba-456a-a450-913be3ede638.png)

5. Each API is collection of resources and methods that are integrated with backend HTTP endpoints, Lambda functions, or other AWS services. Typically, API resources are organized in a resource tree according to the application logic.

6. let's add a resource next

![actions-creating-resources](https://user-images.githubusercontent.com/52368773/215894134-e4e318ff-9654-4137-95a3-f14c109bc2e3.png)

7. Use "DynamoDBManager" in the Name section, Resource Path will get auto-populated. Click "Create Resource"

![creating-resource](https://user-images.githubusercontent.com/52368773/215894391-369eeecb-d7bf-4ea4-be05-49b0569225f5.png)

8. Let's create a " POST Method " for our API. With the "/dynamodbmanager" resource selected, Click "Actions" again and click "Create Method".

![create-methods](https://user-images.githubusercontent.com/52368773/215894788-04f68bf2-0f31-493d-ac7e-a4d1af4084eb.png)

![dynamodbmanager-box](https://user-images.githubusercontent.com/52368773/215895093-0659e3ec-dde0-46a3-8e95-114b2c735551.png)

![POST-METHOD](https://user-images.githubusercontent.com/52368773/215895272-a58bf655-87fc-4fe7-a71e-8c75cce6eb82.png)

9. The integration will come up automatically with "Lambda Function" option selected. Select "LambdaFunctionOverHttps" function that we created earlier. As you start typing the name, your function name will show up.Select and click "Save". A popup window will come up to add resource policy to the lambda to be invoked by this API. Click "Ok"

![post-setup](https://user-images.githubusercontent.com/52368773/215895560-0388ecb6-6af7-44b0-8595-0afa0ff18415.png)

![add-permissions-to-lambda-function](https://user-images.githubusercontent.com/52368773/215895740-4fd7e008-0928-44ae-a04a-9d0aa456154d.png)

10. click "ok" to add the permissions to lambda functions
11. Our API-Lambda integration is Completed !!

# Deploy the API

We need to deploy the API that we created to a stage called prod.

1. Click "Actions", select "Deploy API"

![deploy-API](https://user-images.githubusercontent.com/52368773/215896450-ed9ad424-00e7-4764-8f31-bc8822915a98.png)

2. Now it is going to ask you about a stage. Select "[New Stage]" for "Deployment stage". Give "Prod" as "Stage name". Click "Deploy"

![deploying-API](https://user-images.githubusercontent.com/52368773/215896916-314bdcfb-7c8f-4fd2-93c1-c64d732ce155.png)

3. To invoke our API endpoint, we need the endpoint url. In the "Stages" screen, expand the stage "Prod", select "POST" method, and copy the "Invoke URL" from screen

# Running our solution

# Open POSTMAN

1. The Lambda function supports using the create operation to create an item in your DynamoDB table. To request this operation, use the following JSON

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
2. To execute our API from local machine, we are going to use Postman and Curl command. You can choose either method based on your convenience and familiarity.

To run this from Postman, select "POST" , paste the API invoke url. Then under "Body" select "raw" and paste the above JSON. Click "Send". API should execute and return "HTTPStatusCode" 200.

![postman](https://user-images.githubusercontent.com/52368773/215897670-e1bed7ad-d5d5-444c-b876-fc4e2ad8c04f.png)

3. To validate that the item is indeed inserted into DynamoDB table, go to Dynamo console, select "lambda-apigateway" table, select "Items" tab, and the newly inserted item should be displayed.

![validate-items-dynamodb](https://user-images.githubusercontent.com/52368773/215898621-9314dac5-308f-4ac4-9c48-4fb7eefdf051.png)

4. To get all the inserted items from the table, we can use the "list" operation of Lambda using the same API. Pass the following JSON to the API, and it will return all the items from the Dynamo table

![lambda-api-gateway](https://user-images.githubusercontent.com/52368773/215899333-1c7d2340-43d5-4c22-8d7b-8054e3285ae7.png)

Payload

```

{
    "operation": "list",
    "tableName": "lambda-apigateway",
    "payload": {
    }
}

```

![postman-list](https://user-images.githubusercontent.com/52368773/215900682-adee865d-4679-4ab1-a2dd-f34cda2f6f96.png)

Response

![postman-response](https://user-images.githubusercontent.com/52368773/215900630-3cb313c3-0133-43f4-977f-c086a023d6cb.png)

We have successfully created a serverless API using API Gateway, Lambda, and DynamoDB !!!
