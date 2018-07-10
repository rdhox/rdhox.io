+++ 
draft = false
date = 2018-07-09T17:09:09+02:00
title = "Building a GraphQL API with serverless framework, AWS and Apollo Server 2"
slug = "graphql-api-with-aws-serverless-and-apollo2" 
tags = ["serverless", "aws", "dynamodb", "amazon", "graphql", "apollo-server"]
categories = ["backend", "serverless"]
+++

Okay, let's build an API with the help of the [serverless framework](http://www.serverless.com), GraphQL and Apollo Server 2, with a Dynamo database and AWS. We will build a table where users will be recorded, and lambda functions to create those users, to request informations from them, and update them. All that using a graphQL layer, and apollo-server-lambda to make our job easier.  


You can find the final code [here](https://github.com/rdhox/graphql-api-with-aws-serverless-and-apollo2).

Before anything to happen, we have to subscribe to AWS, to install serverless framework and to set-up AWS credentials.
I let you subscibe to AWS, that simple enough. I assume that we have nodeJS and NPM installed, so install serverless:
```bash
$ npm i -g serverless
```
You can check this [video](https://www.youtube.com/watch?v=HSd9uYj2LJA) to set-up your credentials.
I recommand you to follow the quick start if you're new to serverless framework => [here](https://serverless.com/framework/docs/providers/aws/guide/quick-start/).
First let's create a folder and install some packages :  
```bash
$ mkdir api-graphql
$ cd api-graphql/
$ npm i aws-sdk graphql apollo-server-lambda@rc
```
Open your favorite IDE and let's create two files : `serverless.yml` and `handler.js`:  

-api-graphql/  
: |--serverless.yml  
: |--handler.js  

<br />

## Create the _serverless.yml_ file
---

<br />

Copy this code into the `serverless.yml`:
```yml
#serverless.yml

service: api-graphql

provider:
  name: aws
  runtime: nodejs8.10
  stage: dev
  environment:
    USER_TABLE: users-table-${self:provider.stage}
  iamRoleStatements:
    - Effect: Allow
      Action:
        - dynamodb:Query
        - dynamodb:Scan
        - dynamodb:GetItem
        - dynamodb:PutItem
        - dynamodb:UpdateItem
        - dynamodb:DeleteItem
      Resource: "arn:aws:dynamodb:${opt:region, self:provider.region}:*:table/${self:provider.environment.USER_TABLE}"

resources:
  Resources:
    UserTable:
      Type: 'AWS::DynamoDB::Table'
      Properties:
        AttributeDefinitions:
          - AttributeName: ID
            AttributeType: S
        KeySchema:
          - AttributeName: ID
            KeyType: HASH
        ProvisionedThroughput:
          ReadCapacityUnits: 1
          WriteCapacityUnits: 1
        TableName: ${self:provider.environment.USER_TABLE}

functions:
  graphql:
    handler: handler.graphql
    events:
      - http:
          path: graphql
          method: post
          cors: true
      - http:
          path: graphql
          method: get
          cors: true
```
Ok let's take 5 minutes to explain what we have here. The `serverless.yml` file is the place where we set-up our back-end environment. In this case, the AWS environment. It's a yaml syntax and we can use the CloudFormation syntax, which accept the yaml, for setting-up aws services, like dynamoDB. 


__The _provider_ object__ describe the back-end environment. Variables passed under `environment` are constants that we can find in our JS files with `process.env.YOUR_VARIABLE`. The `iamRoleStatements` is where we give permissions to our aws profile. Here we `Allow` some DynamoDB actions on a specific resource that is our table. `IAM` stands for 'Identity and Access Management'. We can create and set different profiles on our AWS console and be very specific about which profile is allowed to do specifics tasks, which lambda functions is allowed to access specifics resources, etc...  


__The _resources_ object__ set-up the dynamoDB table in this exemple. You can find how the syntax work [here](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-dynamodb-table.html). NoSQL databases don't work like SQL databases (which make sense!). Here we define only the attribute `ID` (type String define by the `S`) of the table named `users-table-dev` which gonna be the Primary Key, define by `HASH`. But we are totally allowed to add in the future other attributes to our user item which is an instance of our user table. And items from our user table can have different attributes between each other. Like you see there is no rule, because the structure is not defined in advance. We are far from, for example the Symfony framework in PHP, where we define our tables attributes and generate our PHP classes with an ORM doing the link. It can be great, but it can be dangerously messy too. You'll find a lot to read about NoSQL database, but for starter, you can read [this nice introduction to DynamoDB](https://www.dynamodbguide.com/what-is-dynamo-db).  


Last part is __the _functions_ object__, where we define our lambda functions. We use only one entry point, which is our apollo server. The name of the function is 'graphql'. The handler, which is the local path where to find the function, is located in the `handler.js` file where we export the graphql function, which explain the `handler.graphql`. Protocol `http` is the way we trigger our function, and we can add some details, like the path in the url to use, the request method, etc...  

<br />


## Create the _handler.js_ file
---

<br />

Let's take a look now to our `handler.js` file :

```javascript
//handler.js

const { ApolloServer, gql } = require('apollo-server-lambda');
const {userTypeDef, userResolvers} = require('./models/user');

const typeDefs = gql`${userTypeDef}`;
const resolvers = userResolvers;

const server = new ApolloServer({
    typeDefs,
    resolvers,
    context: ({ event, context }) => ({
        headers: event.headers,
        functionName: context.functionName,
        event,
        context,
    })
});

exports.graphql = server.createHandler({
    cors: {
      origin: '*',
      credentials: true,
    },
  });
```

We just set-up our apollo server and linked it to the 'graphql' end-point that we export. Every request of the type "https://your_amazon_aws_url/graphql" will trigger the apollo server. But as you can see, there is nothing about the user schema here and no back-end logic, we just imported some stuff from `user.js`. Let's create a few other things and discuss about it.

<br />

## Create the back-end logic
---

<br />

-api-graphql/  
: |--serverless.yml  
: |--handler.js  
: |--functions/
: &nbsp;&nbsp;&nbsp;|--promisify.js
: |--models/  
: &nbsp;&nbsp;&nbsp;|--user.js

```javascript
// ./functions/promisfy.js

module.exports = foo => new Promise((resolve, reject) => {
    foo((error, result) => {
        if (error) {
            reject(error)
        } else {
            resolve(result)
        }
    })
})
```

```javascript
// ./models/user.js

const AWS = require('aws-sdk');
const promisify = require('../functions/promisify');
const crypto = require('crypto');

dynamoDb = new AWS.DynamoDB.DocumentClient();

//Schema of user

exports.userTypeDef = `
    type User {
        ID: String
        email: String
        country: String
    }
    type Query {
        user(ID: String!): User
    }
    type Mutation {
        createUser(email: String): Boolean
        updateUser(ID: String, country: String): User
    }
`;

exports.userResolvers = {
    Query: {
        user: (_, { ID }) => getUser(ID),
    },
    Mutation: {
        createUser: (_, { email }) => createUser(email),
        updateUser: (_, { ID, country }) => updateUser(ID, country),
    }
};

// Lambda functions of user

const createUser = email => promisify(callback => 
    dynamoDb.put({
        TableName: process.env.USER_TABLE,
        Item: {
            ID: crypto.createHash('md5').update(email).digest('hex').toString(),
            email: email,
        },
        ConditionExpression: 'attribute_not_exists(#u)',
        ExpressionAttributeNames: {'#u': 'ID'},
        ReturnValues: 'ALL_OLD',
    }, callback))
    .then( (result) => true)
    .catch(error => {
        console.log(error)
        return false;
    })

const getUser = ID => promisify(callback =>
    dynamoDb.get({
        TableName: process.env.USER_TABLE,
        Key: { ID },
    }, callback))
    .then(result => {
        if(!result.Item){ return ID; }
        return result.Item;
    })
    .catch(error => console.error(error))

const updateUser = (ID, country) => promisify(callback => 
    dynamoDb.update({
        TableName: process.env.USER_TABLE,
        Key: { ID },
        UpdateExpression: 'SET #foo = :bar',
        ExpressionAttributeNames: {'#foo' : 'country'},
        ExpressionAttributeValues: {':bar' : country},
        ReturnValues: 'ALL_NEW'
    }, callback))
    .then(result => result.Attributes)
    .catch(error => console.log(error))
```

__promisify.js:__
In version 10 of nodeJS, there is a function call promisify that transform functions into promises easily, which is very cool. AWS is not running nodeJS 10 yet, so we create and export our own promisify function.  

__user.js:__
We finally find the code that we are interested here. We can put everything into the `handler.js` file, but since your project is not gonna be as simple as a tutorial, it's good to see how we can organize ourself (and you can read [that](https://blog.apollographql.com/modularizing-your-graphql-schema-code-d7f71d5ed5f2) to learn more about structuring an apollo project). We can find the schema and the resolvers of the user at the start. With apollo-server our job is nicely simplified. The resolvers take promises where we use the aws-sdk library to manipulate our dynamoDB table.  


Little detail here, we can't ask dynamoDB to generate automatically a unique ID for our table, due to the nature of a NoSQL database. There are some solutions that you can find, here we simply hash the email with the crypto library of Node to get our ID, and we checked if the ID already exist in our table. If yes, an error is thrown, if no, everything's alright. This method is relatively basic, and may not work if you expand to different regions. Be warn.  


Now let the serverless framework do is magic:
```bash
$ sls deploy
```

<br />

## Test the API
---

<br />

Once our project is deployed, serverless should give us the end-points of our functions. Here we got 2 identics urls, for post and get requests, in a form of `https://******.execute-api.us-east-1.amazonaws.com/dev/graphql`.  
We can test our api directly with the curl command, or by going to our aws account in the API Gateway service. There we can see our end-points, see the functions attached and test them. It's very nice so we will do that.  
By the way, the aws console is really something important that you have to understand if you want to use aws services. There really is a lot of stuff to see, lot of services, you can really be lost checking everything, but it's worth it.  


Ok I assume that you are in your API Gateway page. If nothing appeared, check that you are in the right region, here _us-east-1_.
Click on _dev-api-graphql_, _post_, and _test_. In the header text-area, put `Content-Type: application/json`. In the Request Body, paste :

```json
{
	"query": "mutation CreateUser($arg: String!){ createUser(email: $arg)}",
	"operationName": "CreateUser",
	"variables": {
		"arg": "test-dev@test.fr"
	}
}
```
I vividly encourage you to test your json with https://jsonlint.com/. It can save you a lot of time if you're like me and you're mistaken `:` with `=` and you passed the last 2 hours without noticing it... Anyway, click _Test_, and if the result look like :

```json
{
  "data": {
    "createUser": true
  }
}
```
Well done, you created your first user! If you click again on _Test_, the result should be false this time because we can't create two user with the same email. We can go check your dynamoDB table on our aws console to see our item with our two attributes, ID and email. Let's try to get the email of the user:

```json
{
	"query": "query gUser($arg: String!){ user(ID: $arg){ email }}",
	"operationName": "gUser",
	"variables": {
		"arg": "86250407fc87f3d297e3076b08133cfd"
	}
}
```
The response is :

```json
{
  "data": {
    "user": {
      "email": "test-dev@test.fr"
    }
  }
}
```

Perfect it's exactly what we wanted! Ok now let's update the user :

```json
{
	"query": "mutation UpdateUser($id: String!, $country: String! ){ updateUser(ID: $id, country: $country){ ID email country }}",
	"operationName": "UpdateUser",
	"variables": {
		"id": "86250407fc87f3d297e3076b08133cfd",
		"country": "France"
	}
}
```

And the response is:

```json
{
  "data": {
    "updateUser": {
      "ID": "86250407fc87f3d297e3076b08133cfd",
      "email": "test-dev@test.fr",
      "country": "France"
    }
  }
}
```

And our item has now a _country_ attribute.  


Ok I let you find how we can delete this user, you should have the logic now!

This method of building an API is not the best. Why? Because there is no communication between our graphQL schemas where we define our types and the functions where we interact with our tables. You always have to be careful about what you are doing with the data and what graphQL can do with it. Fortunately there is something call AWS AppSync that gonna make our job easier, and I plan to write about it soon!








