<!--
title: 'Serverless Framework Node Express API service backed by DynamoDB on AWS and localstack'
description: 'This template demonstrates how to develop and deploy a simple Node Express API service backed by DynamoDB running on AWS Lambda using the traditional Serverless Framework. It demonstrates local deployment with localstack community'
layout: Doc
framework: v3
platform: AWS / Localstack
language: nodeJS
priority: 1
authorLink: 'https://github.com/opengeekv2'
authorName: 'Marc Mauri'
authorAvatar: 'https://avatars.githubusercontent.com/u/901282?v=4'
-->

# Serverless Framework Node Express API on AWS and Localstack

This template demonstrates how to develop and deploy a simple Node Express API service, backed by DynamoDB database, running on AWS Lambda using the traditional Serverless Framework.

It demonstrates local deployment with localstack community


## Anatomy of the template

This template configures a single function, `api`, which is responsible for handling all incoming requests thanks to the `http` event. To learn more about `http` event configuration options, please refer to [http event docs](https://www.serverless.com/framework/docs/providers/aws/events/apigateway). As the event is configured in a way to accept all incoming requests, `express` framework is responsible for routing and handling requests internally. Implementation takes advantage of `serverless-http` package, which allows you to wrap existing `express` applications. To learn more about `serverless-http`, please refer to corresponding [GitHub repository](https://github.com/dougmoscrop/serverless-http). Additionally, it also handles provisioning of a DynamoDB database that is used for storing data about users. The `express` application exposes two endpoints, `POST /users` and `GET /user/{userId}`, which allow to create and retrieve users.

## Usage

### Deployment

Install dependencies with:

```
npm install
```

and then deploy with:

```
serverless deploy
```

After running deploy, you should see output similar to:

```bash
Deploying aws-node-express-dynamodb-api-project to stage dev (us-east-1)

✔ Service deployed to stack aws-node-express-dynamodb-api-project-dev (196s)

endpoint: ANY - https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com
functions:
  api: aws-node-express-dynamodb-api-project-dev-api (766 kB)
```

_Note_: In current form, after deployment, your API is public and can be invoked by anyone. For production deployments, you might want to configure an authorizer. For details on how to do that, refer to [`http` event docs](https://www.serverless.com/framework/docs/providers/aws/events/apigateway). Additionally, in current configuration, the DynamoDB table will be removed when running `serverless remove`. To retain the DynamoDB table even after removal of the stack, add `DeletionPolicy: Retain` to its resource definition.

### Invocation

After successful deployment, you can create a new user by calling the corresponding endpoint:

```bash
curl --request POST 'https://xxxxxx.execute-api.us-east-1.amazonaws.com/users' --header 'Content-Type: application/json' --data-raw '{"name": "John", "userId": "someUserId"}'
```

Which should result in the following response:

```bash
{"userId":"someUserId","name":"John"}
```

You can later retrieve the user by `userId` by calling the following endpoint:

```bash
curl https://xxxxxxx.execute-api.us-east-1.amazonaws.com/users/someUserId
```

Which should result in the following response:

```bash
{"userId":"someUserId","name":"John"}
```

If you try to retrieve user that does not exist, you should receive the following response:

```bash
{"error":"Could not find user with provided \"userId\""}
```

### Local development

It is also possible to emulate DynamoDB, API Gateway and Lambda locally using the `serverless-localstack` plugin.

To do that you will need first to [install localstack](https://docs.localstack.cloud/getting-started/installation/).

After installing run it with:

```bash
localstack start
```

DynamoDB setup has been tweaked to support both environments:

```js
let dynamoConfig = {}
if (process.env.LOCALSTACK_HOSTNAME) {
  dynamoConfig = {endpoint: "http://" + process.env.LOCALSTACK_HOSTNAME + ":4566" }
}
const client = new DynamoDBClient(dynamoConfig);
```

Running the following command with start both local API Gateway emulator as well as local instance of emulated DynamoDB:

```bash
serverless deploy --stage local
```
After running deploy, you should see output similar to:

```bash
Deploying aws-node-express-dynamodb-api-project to stage local (us-east-1, "serverless" provider)
Using serverless-localstack
Using serverless-localstack
serverless-localstack: Reconfigured endpoints
serverless-localstack: Reconfigured endpoints

...

✔ Service deployed to stack aws-node-express-dynamodb-api-project-local (30s)

dashboard: https://app.serverless.com/opengeekv2/apps/aws-node-express-dynamodb-api-project/aws-node-express-dynamodb-api-project/local/us-east-1
endpoint: http://localhost:4566/restapis/k8c26o9vfx/local/_user_request_
functions:
  api: aws-node-express-dynamodb-api-project-local-api (1.1 MB)
```

k8c26o9vfx is the apiId
local is the stage

### Local Invocation

After successful deployment, you can create a new user by calling the corresponding endpoint:

```bash
curl --request POST 'https://<apiId>.execute-api.localhost.localstack.cloud:4566/<stage>/users' --header 'Content-Type: application/json' --data-raw '{"name": "John", "userId": "someUserId"}'
```

Which should result in the following response:

```bash
{"userId":"someUserId","name":"John"}
```

You can later retrieve the user by `userId` by calling the following endpoint:

```bash
curl https://<apiId>.execute-api.localhost.localstack.cloud:4566/<stage>/users/someUserId
```

Which should result in the following response:

```bash
{"userId":"someUserId","name":"John"}
```

If you try to retrieve user that does not exist, you should receive the following response:

```bash
{"error":"Could not find user with provided \"userId\""}
```