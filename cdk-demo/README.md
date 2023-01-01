# Welcome to your CDK TypeScript project

This is a blank project for CDK development with TypeScript.

The `cdk.json` file tells the CDK Toolkit how to execute your app.

## Useful commands

- `npm run build` compile typescript to js
- `npm run watch` watch for changes and compile
- `npm run test` perform the jest unit tests
- `cdk deploy` deploy this stack to your default AWS account/region
- `cdk diff` compare deployed stack with current state
- `cdk synth` emits the synthesized CloudFormation template

# Important files

Here are some of the important files and what they are used for:

- **bin/cdk-demo.ts** - This is the entry point to your CDK application. This will load/create all the stacks we define under **lib/**.

- **lib/cdk-demo-stack.ts** - This is where your main CDK application stack is defined. Your resource and its properties can go here.

- **package.json** - This is where you define your project dependencies, as well as some additional information, and build scripts (npm build, npm test, npm watch).

- **cdk.json** - This file tells the toolkit to run your application and also contains some additional settings and parameters related to CDK and your project.

With **lib/cdk-demo-stack.ts** and **bin/cdk-demo.ts** files we can create our infraestructure.

## Create the infraestructure

To start building out a project, a common starting point is to create a logically isolated virtual network that you define, called **Amazon Virtual Private Cloud (VPC)**

While the CDK pulls this information from your local AWS CLI configuration.

Settins will be defining in our VPC. If you do not specify this, the stack will be environment-agnostic, but some features and contexts lookups will not work.

```
#!/usr/bin/env node
import 'source-map-support/register';
import * as cdk from 'aws-cdk-lib';
import {CdkDemoStack} from '../lib/cdk-demo-'

const app = new cdk.App();
new CdkDemoStack(app, 'CdkDemoStack', {
  env:{account:'AccountNumber',
  }
})

```

For this tutorial we will create a VPC with two public-facingsubnets, spread across two Availability Zones.

Before we can dive into writting the code, we need to explain and install AWS CDK library modules.

A construct can represent a single AWS resource, or it can be a higher-level abstraction consisting of multiple AWS related resources.

The AWS construct library is organized into several modules. For this project we need the **Amazon EC2** module, which also includes support for **Amazon VPCs**.

To install EC2 module use npm.

```
npm install @aws-cdk/aws-ec2
```

Now we are ready to create our VPC.

We want to create a very simple setup spanning two AZs, and with a public subnet for each. Please read this document for a more detailed explanation of the differences.

## Deploy

The **cdk deploy** command compiles your TypeScript into javascript and creates a **CloudFormation** change set to deploy this change.

CDK manages all of this for you, along with uploading the template file to S3 and using **CloudFormation** to run it.

After a few minutes, you should get a green check mark along with an **ARN (Amazon Resource Name)** of your newly created **CloudFormation** stack.

Your new VPC has now been deployed and is ready to be used.

The VPC created in this tutorial does not cost anything per month, but there is a quota that limits each region in an account to only allow five VPCs.

## Clean up resources

If you wish to remove resources you can run the **cdk destroy** command and it will remove all the resources via the CloudFormation stack it created.
