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

`{
    "operation": "create",
    "tableName": "lambda-apigateway",
    "payload": {
        "Item": {
            "id": "1",
            "name": "Bob"
        }
    }
}`

sample request payload for a DynamoDB read item

`{
    "operation": "read",
    "tableName": "lambda-apigateway",
    "payload": {
        "Key": {
            "id": "1"
        }
    }
}`

# Configuration steps

o create an execution role

1. Open the roles page in the IAM console.
2. Choose Create role.
3. Create a role with the following properties.
4. Trusted entity – Lambda.
5. Role name – lambda-apigateway-role.
6. Permissions – Custom policy with permission to DynamoDB and CloudWatch Logs. This custom policy has the permissions that the function needs to write data to   DynamoDB and upload logs.

`{
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
}`

# Create Lambda Function

Steps involes:

1. Click "Create function" in AWS Lambda Console

![create-lambda-functions](https://user-images.githubusercontent.com/52368773/215882284-96c03460-5562-455c-970e-3b5ba5d9a9bd.png)

2. Select "Author from scratch". Use name LambdaFunctionOverHttps , select Python 3.7 as Runtime. Under Permissions, select "Use an existing role", and select lambda-apigateway-role that we created, from the drop down




