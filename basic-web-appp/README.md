# Modules

1. Create Web App. Deploy static resources for your web application using the AWS Amplify Console. 

2. Build Serverless Function. Build a serverless function using AWS Lambda. 

3. Link Serverless Function to Web App. Deploy your serverless function with API Gateway. 

4. Create Data Table. Perist data in Amazon DynamoDB table. 

5. Add Interactivity to Web App. Modify your web app to invoke your API. 

# 1 AWS Amplify Overview

We use **AWS Amplify** to deploy static resources for your web app. In subsequent modules, you will add dynamic functionality to these pages using **AWS Lambda** and **Amazon API Gateway** to remote **RESTful APIs**. 

REST (Representational State Transfer) is an architectural pattern for creating web services. API stands for *Application programming interface*. 


## Key concepts

- static website. A static website has fixed content, unlike dynamic websites. Static websites are the most basic type and are the easiest to create. All that is required is creating a few HTML pages and publishing them to a web server. 

- web hosting. Provides the technologies/services for the website to be viewed on the internet. 

- AWS Regions. Separate geographic areas that AWS uses to house its infraestructure. These are distributed around the world so that customers can choose a Region closest to them to host their cloud infraestructure there. 


# 2 AWS Lambda Overview

You will be writing a small piece of code, in python, javascript or java, to be used in later module to add Interactivity to your web page. 

You will use **AWS Lambda**, a compute service that let's you create serverless functions, eliminating the need for you to manage software and hardware. Instead, applicactions are broken up into individual functions that can be invoked and scaled individually. 


