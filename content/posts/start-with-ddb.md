+++ 
draft = false
date = 2018-06-29T21:57:17+02:00
title = "Ressources to start with DynamoDB"
slug = "start-with-ddb" 
tags = ["serverless", "aws", "dynamodb", "amazon"]
categories = ["backend"]
+++

I am used to MongoDB when it comes to store data. Easy to install with great tools to help, easy to use especially with [Mongoose](http://mongoosejs.com/), it's a great solution.  
But I'm playing with AWS at the moment with the [serverless framework](http://www.serverless.com) and the use of DynamoDB, the NoSQL database of Amazon, is part of the stack with API Gateway, Lambda, etc... So it would be stupid to not try it.  
If you are using MondoDB, DynamoDB will not be unknown to you, it's a NoSQL database based on the document model. You can manage your tables with the GUI of amazon on your AWS account or you could also use the CloudFormation syntax to create your tables and set-up the different settings that you need. Since serverless framework mostly works with a big file (the size depends on application of course, nothing to be scared of!) called `serverless.yml`, you can setup your DynamoDB in this file among other stuff to be able to deploy easily a full functional back-end service.  
The problem here is the CloudFormation syntax which may slow you down a little. Being new with all the amazon web services, it was kind of a struggle to find my way among all these pages. So here are the links that will help you take rapid advantage of the DynamoDB and the CloudFormation Syntax for your `serverless.yml` file:

* [To learn how DynamoDB works and the Amazon jargon](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.CoreComponents.html)
* [API of DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_CreateTable.html)
* [Using CloudFormation to create DynamoDB table](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-dynamodb-table.html)

