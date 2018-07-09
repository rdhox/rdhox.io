+++ 
draft = true
date = 2018-07-09T17:09:09+02:00
title = "Building a GraphQL API with serverless framework, AWS and Apollo Server 2"
slug = "graphql-api-with-aws-serverless-and-apollo2" 
tags = ["serverless", "aws", "dynamodb", "amazon", "graphql", "apollo-server"]
categories = ["backend", "serverless"]
+++

Okay, let's build an API with the help of the [serverless framework](http://www.serverless.com), GraphQL and Apollo Server 2, with a Dynamo database and AWS. We will build a table where users will be record, and lambda functions to create those users, to request informations from them, and update them. All that using a graphQL layer, and apollo-server-lambda to make our job easier.  
Before anything to happen, you have to subscribe to AWS, to install serverless framework and to set-up your AWS credentials.
I let you subscibe to AWS, that simple enough. I assume that you have nodeJS and NPM installed, so install serverless:
```
$ npm i -g serverless
```
You can check this [video](https://www.youtube.com/watch?v=HSd9uYj2LJA) to set-up your credentials.
I recommand you to follow the quick start if you're new to serverless framework => [here](https://serverless.com/framework/docs/providers/aws/guide/quick-start/).
First let's create a folder and install some packages :  
```
$ mkdir api-graphql
$ cd api-graphql/
$ npm i aws-sdk graphql apollo-server-lambda@rc
```
Open your favorite IDE and let's create two files : `serverless.yml` and `handler.js`:  

-api-graphql  
: |--serverless.yml  
: |--handler.js  

Copy this code into the `serverless.yml`:
```yml
#serverless.yml

service: api-graphql

provider:
  name: aws
  runtime: nodejs8.10
  region: eu-west-3
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
Ok let's take 5 minutes to explain what we have here. The `serverless.yml` file is the place where you gonna set-up your back-end environment. In this case, the AWS environment. It's a yaml syntax and you can use the CloudFormation syntax, which accept the yaml, for setting-up aws services, like dynamoDB.  
The provider object describe the back-end environment. The variables passed under `environment` are constants that you can find in your JS files with `process.env.YOUR_VARIABLE`. The `iamRoleStatements` is where you give permissions. Here we "Allow" some DynamoDB actions on a specific resource that is our table. `IAM` stand for 'Identity and Access Management'. You can create and set different profiles on your AWS console and be very specific about which profile is allowed to do specifics tasks, which lambda functions is allowed to access specifics resources, etc...  
The resources object set-up the dynamoDB table in this exemple. You can find how the syntax work [here](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-dynamodb-table.html). NoSQL database don't work like SQL database (which make sense!). Here we define only the attribut `ID` (type String define by the `S`) of the table named `users-table-dev` which gonna be the Primary Key, define here by `HASH`. But we are totally allowed to add in the future other attributs to our user item which is an instance of our user table. And items from our user table can have different attributs between each others. Like you see there is no rule, because the structure is not define in advance. We are far from, for example the Symfony framework in PHP, where we define our tables attributs and generate our PHP classes with an ORM doing the link. It can be great, but it can be dangerously messy too. You'll find a lot to read about NoSQL database, but for starter, you can read [this nice introduction to DynamoDB](https://www.dynamodbguide.com/what-is-dynamo-db).  
Last part is the functions object, where you define your lambda functions. We gonna use only one entry point, which gonna be our apollo server. The name of the function is 'graphql'. The handler, which is the path where to find the function, is located in the `handler.js` file where we gonna export the graphql function, which explain the `handler.graphql`. The `http` is the way that we gonna use to trigger our function, and we can add some details, like the path in the url to use, the request method, etc...  


Let's take a look now to our `handler.js` file :

```javascript
//handler.js

const { ApolloServer, gql } = require('apollo-server-lambda');
const User = require('./user');

const typeDefs = gql`${User.typeDef}`;
const resolvers = User.resolvers;

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

We just set-up our apollo server and linked it to the 'graphql' end-point that we export. Every request of the type "https://your_amazon_aws_url/graphql" will trigger the apollo server. But as you can see, there is nothing about the user schema here and no back-end logic, we just imported an `user.js` file. Let's create a few other things and discuss about it:

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
const promisify = require('./promisify');
const crypto = require('crypto');

//Schema of user

export const typeDefs = `
    type User {
        ID: String!
        email: String!
    }
    extend type Query {
        user(ID: String!): User
    }
    extend type Mutation {
        createUser(email: String!): Boolean
    }
`;

export const resolvers = {
    Query: {
        user: (_, { email }) => getUser(email),
    },
    Mutation: {
        createUser: (_, { email }) => createUser(email),
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
        ReturnValues: 'ALL_OLD',
    }, callback))
    .then( () => true)
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
```

__promisify.js:__
In version 10 of nodeJS, there is a function call promisify that transform functions into promises easily, which is very cool. AWS is not running nodeJS 10 yet, so we create and export our own promisify function.  

__users.js:__
We finally find the code that we are all interested here. You can wrote everything into the `handler.js` file, but since your project is not gonna be as simple than a tutorial, it's good to see how you can organize yourself (and you can read [that](https://blog.apollographql.com/modularizing-your-graphql-schema-code-d7f71d5ed5f2) to learn more about structuring a project). 





