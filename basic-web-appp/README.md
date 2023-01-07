# Modules

1. Create Web App. Deploy static resources for your web application using the AWS Amplify Console.

2. Build Serverless Function. Build a serverless function using AWS Lambda.

3. Link Serverless Function to Web App. Deploy your serverless function with API Gateway.

4. Create Data Table. Perist data in Amazon DynamoDB table.

5. Add Interactivity to Web App. Modify your web app to invoke your API.

# 1 AWS Amplify Overview

We use **AWS Amplify** to deploy static resources for your web app. In subsequent modules, you will add dynamic functionality to these pages using **AWS Lambda** and **Amazon API Gateway** to remote **RESTful APIs**.

REST (Representational State Transfer) is an architectural pattern for creating web services. API stands for _Application programming interface_.

## Key concepts

- static website. A static website has fixed content, unlike dynamic websites. Static websites are the most basic type and are the easiest to create. All that is required is creating a few HTML pages and publishing them to a web server.

- web hosting. Provides the technologies/services for the website to be viewed on the internet.

- AWS Regions. Separate geographic areas that AWS uses to house its infraestructure. These are distributed around the world so that customers can choose a Region closest to them to host their cloud infraestructure there.

# 2 AWS Lambda Overview

You will be writing a small piece of code, in python, javascript or java, to be used in later module to add Interactivity to your web page.

You will use **AWS Lambda**, a compute service that let's you create serverless functions, eliminating the need for you to manage software and hardware. Instead, applicactions are broken up into individual functions that can be invoked and scaled individually.

These serverless functions are triggered based on a specific event you will define the code. It is also a very affordable service, because you are only charged for the number of events you process, not the idle time. Best of all, you do not have to worry about managing any servers.

- Create a Lambda function from scratch using the AWS console (in python, javascript or java)

- Create (JSON) events in the AWS console to test your function

## key concepts

- compute service - A service that provides computational processing power.

- serverless function - Piece of code that will be executed by a compute service, on demand.

- lambda trigger - The type of event that will make a lambda (serverless) function run. This can be another AWS service or external input.

You will notice that we added the _AWS Lambda_ service to the diagram, but it does not yet have a connection to AWS Amplify.

# Link a Serverless Function To a Web App

We will use _Amazon API Gateway_ to create a RESTful API that will allow us to make calls to our Lambda function from a web client (typically refers to a user's web browser).

API Gateway will act as a middle layer between the HTML client we created in module one and the serverless backend we created in module two.

- Create a new API using API Gateway

- Define HTTP methods on your API

- Trigger a Lambda function from an API

- Enable cross-origin resource sharing (CORS) on an API so you can consume resources from a different origin (domain)

- Test an API created with API Gateway from the AWS Management Console

## Key Concepts

- RESTful API - Rest stands for "Representational State Transfer" and is an architectural pattern for creating web services. API stands for 'application programming interface'. A RESTful API is one that implements the REST architectural pattern.

- HTTP request methods - HTTP methods are designed to enable communications between clients and servers. Methods like GET or PUT defined by the HTTP protocol, are used to indicate what action to take on a resource.

- CORS - The CORS browser security feature uses HTTP headers to tell a browser to allow a given web application running at one origin (domain) to access selected resources from a server a different origin.

- Edge optimized - A resource that uses AWS global infraestructure to better serve geographically diverse clients. Edge-optimized endpoints are best for geographically distributed clients. This makes them a good choice for public servicess being accessed from the internet. Regional endpoints are typically used for APIs that are accessed primarily from within the same AWS region.

# Create a Data Table

You will create an Amazon DynamoDB table and enable your Lambda function to store data in it.

We will create a table to persist data using _Amazon DynamoDB_.

DynamoDB is key-value database service, so we do not need to create a schema for our data. It has consistent performance at any scale and there are no servers to manage when using it.

Additionally, we will use the _AWS Identiity and Access Management (IAM) service_ to securely give our services the required permissions to interact with each other.

Specifically, we are going to allow the Lambda function we created in module two to write to our newly created DynamoDB table using an IAM policy. To do this, we will use the AWS SDK(python, javascript) from our Lambdan function.

- Create a DynamoDB table using the AWS Management Console.

- Create a role and manage permissions with IAM.

- Write to a DynamoDB table using the AWS SDK (python, javascript)

## key concepts

- Persisting data - Storing data so we can access it in the future, independently from program execution.

- Non-relational database - Non-relational database do no use a tabular schema of rows and columns. Instead, they use a storage model that is optimized for the specific requirements of the type of data being stored.

- Key-value database - A type of non-relational database that stores data as a collection of key-value pairs in which a key serves as a unique identifier.

- Primary key - The valu that will identify each piece of data in a DynamoDB table. This value will also serve to partition the table to make it scalable.

- Schema - The organization of data that serves as a blueprint for how a database should be constructed.

- AWS SDK - Software Development Kits, provide a set of tools, libraries, documentation, code samples, processes, and guides that allow developers to cerate software applications on a specific platform.

- IAM Policy - A document that defines what AWS resources an entity, such as a service, user, or group has access to.

We added two services in this module: DynamoDB(for storage) and IAM(for managing permissions securely). Both are connected to our Lambda function, so that it can write to our database. The final step is to add code to our client to cal the API Gateway.

# Add Interactivity to Your Web App

You will modify your static website to invoke your API and display custom text. 

We will update the static website we created in module 1 to invoke the REST API we created in module 3. This will add the ability to display text based on what you input. 

- Call an API Gateway API from an HTML page 

- Upload a new version of a web app to the Amplify console

## Key concepts

- Deploying a website - Making a website available to users.

- Environment - A stage such as 'prod', 'dev' or 'staging', where an application or website can be executed. 

- Invoking an API - Sending an event to an API to trigger a specific behaviour. 


