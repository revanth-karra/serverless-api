# Building serverless API using API Gateway, Lambda, and DynamoDB

![NEWIMG](https://user-images.githubusercontent.com/52368773/215879828-3738c773-e190-43d8-8dc6-30954b0e4310.png)

Create (DynamoDBManager) and (POST) method on it. The method is backed by a Lambda function (LambdaFunctionOverHttps). That is, when we call the API through an HTTPS endpoint, Amazon API Gateway invokes the Lambda function.

The POST method on the DynamoDBManager supports the following DynamoDB operations:

Create, update, and delete an item.
Read an item.
Scan an item.
Other operations (echo, ping), not related to DynamoDB, that you can use for testing.
