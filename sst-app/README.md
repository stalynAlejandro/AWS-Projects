# SST

### What are we covering

We've structured this tutorial in a way that shows you what the development workflow looks like while working with SST. We start with setting up your local development environment and go all the way to "git push to deploy" in production.

Covering:

1. Creating a new SST app

2. Setting up your local dev environment

3. Writing to a database

4. Connecting it to your API

5. Rendering it in the frontend

6. Deploying to production

At a very high level, we are using the following:

- SST's constructs to define our infraestructure

- PostgreSQL for our database

- GraphQL for the API

- React for our frontend

## Create a New Project

> npx create-sst@latest --template=graphql/rds

## Start Live Lambda Dev

> npx sst dev

The `sst dev` command, as you might've guessed, deploys to your AWS account. It does a couple of interesting things:

- 1.  Bootstraps your AWS account for SST.

- 2.  Setups up the _Live Lambda Dev environment_

- 3.  Deploys your app to AWS.

- 4.  Runs a local server to:

- 4.1 Proxy Lambda requests to your local machine.

- 4.2 Power the SST Console.

# Project Structure - Monorepo

## stacks/

The `stacks` directory contains the app's infrastructure as defined as code. Or what is known as **Infraestructure as Code(IaC)**.

> `Database.ts` creates a PostgreSQL database cluster.

```ts
//  stacks/Database.ts
export function Database({ stack }: StackContext) {
  const rd = new RDS(stack, "db", {
    engine: "postgresql11.13",
    //
  });
}
```

Stacks also allows us to return props that we can reference in other stacks.

```ts
return rds;
```

> `Api.ts` creates an API with GraphQL endpoint at /graphql using _API Gateway_

```ts
//  stacks/Api.ts

export function Api({ stack }: StackContext) {
  const rd = use(Database);

  const api = new ApiGateway(stack, "api", {
    /// ...
  });
}
```

The `use(Database)` call gives this stack access to the props that the `Database` stack returns. So `rds` is coming from the return statement of our `Database` stack.

We _bind_ the database to our API so that the functions that power our API have access to it.

```ts

function: {
    bind: [rds],
}

```

The `bind` prop does two things for us. It gives our functions permissions to access the database. Also our functions are loaded with the database required to query it.

> `Web.ts` cretes a `Vite` static site hosted on S3, and serves the content through a CDN using `CloudFront`.

```ts
//  stacks/Web.ts

export function Web({ stack }: StackContext) {
  const api = use(Api);

  const site = new StaticSite(stack, "site", {
    // ...
  });
}
```

We get the `Api` stack to set the GraphQL API Url as an environment variable for our frontend to use.

```ts
environment: {
  VITE_GRAPHQL_URL: api.url + "/graphql";
}
```

## packages/

The `packages/` directory houses everything that powers our backend. This includes our **GraphQL API**,but also all your business logic, and whatever else you need.

- `packages/core` contains all of your business logic. The `create sst` setup encourages **Domain Driven Desing**. It helps you keep your business logic separate from your API and Lambda functions. This allows you to write simple, maintainable code. It implements all the things your application can do. These are then called by external facing services - like an API.

- `packages/core/migrations` contains a React application created with `Vite`. It's already wired up to be able to talk to the **GraphQL API**. If you are using a different frontend, for example NextJs, you can delete this folder and provision it yourself.

- `packages/functions` is where you can place all the code for your **Lambda functions**. Your functions should generally be fairly simple. They should mostly be calling into code previously defined in `services/core`.

- `packages/graphql` contains the outputs of GraphQL related code generation. Typically you won't be touching this but it needs to be comminted to Git. It contains code shared between the frontend and backend.

## packages.json

Is relatively simple. But there are couple of things of note

> **Workspaces**

As we had mentioned above, we are using **workspaces** to organize our monorepo setup.

Workspaces help you manage dependencies for separate `packages` inside your repo that have their own `package.json` files.

We have workspaces in our setup.

```ts
//  package.json

"workspaces":[
  "packages/*",
]

```

You'll notice that all these directories have their own `package.json` file.

So when you need to install/uninstall a dependency in on of those workspaces, you can do the following from the project root.

```ts
$ npm install <package> -W <workspace>
$ npm uninstall <package> -W <workspace>
```

Or you can do the regular `npm install` in the workspace's directory.

## Scripts

Our starter also comes with a few helpful scripts.

```ts
//  package.json

"scripts": {
  "dev":"sst dev",
  "build":"sst build",
  "deploy":"sst deploy",
  "remove":"sst remove",
  "console":"sst console",
  "typecheck":"tsc --noEmit",
  "test":"sst bind -- vitest run",
  "gen":"hygen"
}

```

> `dev`: **Start the Live Lambda Dev** environment for the _default_ stage.

> `build`: Build the **CloudFormation** for the Infraestructure of the app for the _default_ stage. It converts the SST constructs to _CloudFormation_ and packages the necessary assets, but it doesn't deploy them.

> `deploy`: Build the infrastructure and deploy the app to AWS.

> `remove`: Completely remove the app's infraestructure from AWS for the _default_ stage. Use with caution!

> `console`: Start the **SST Console** for the _default stage_. Useful for managing _non-local_ stages.

> `typecheck`: Run typecheck for the entire project. By default, our editor should automatically typechek our code using the `tsconfig.json` in our project root. However, this script lets you explicitly run typecheck as a part of our CI process.

> `test`: Load our `Config` and run our tests. Our starter uses `Vitest`.

> `gen`: Uses **Hygen** to run built-in code gen tasks. Currently only supports `npm run gen migration new`. This will help you code gen a new migration.

- Note: The _default_ stage that we are referring to above; is the one that you selected while first creating the app.

## sst.config.ts

Finally, the `sst.config.ts` defines the project config and the stacks in the app.

```ts
//  sst.config.ts

export default {
  config(_input){
    return{
      name:"my-sst-app",
      region:"us-east-1",
    };
  },
  stacks(app){
    app.stack(Database).stack(Api).stack(web)
  }
} satisfies SSTConfig;

```

By now your `sst dev` process should be complete. So let'srun our first migration and initialize our database.

# Initialize the Database

After the `sst dev` command.

Once your local development is up and running, you should see the following printed out in the terminal.

```

SST v2.5.5 ready!

=> App:     my-sst-app
   Stage:   sa
   Console: https://console.sst.dev/my-sst-app/sa
```

We are now ready to initialize our database. We are using RDS with PostgreSQL in this setup.

## RDS

RDS is a fully-managed database offering from AWS. It supports PostgreSQL and MySQL engines.

SST provisions a serverless flavour of it with the `RDS` construct. RDS will automatically scale up and down based on the load it's experiencing.

- Note. Serverless RDS can take a few minutes to autoscale up and down.

We'll use RDS with PostgreSQL in this tutorial because it is the most familiar option. We'll do a deep dive into a true serverless database like `DynamoDB` at a later date.

## Open the Console

Head over to the _Console_ link in your browser - `https://console.sst.dev/ss-app/sa/local`

- The SST Console is a web based dashboard to manage your SST apps.

Then navigate to the RDS tab.

To add tables in our database we are going to run a **migration**.

### What is a migration

Migrations are a set of files that contain the queries necessary to make updates to our database schema. They have an `up` function, that's run while applying the migration. And a `now` function, that's run while rolling back the migration.

Recall from the _Project Structure_ chapter that the migration files are placed in `packages/core/migrations`.

The starter creates the first migration for you. It's called `article` and you'll find it in `package/core/migrations/165000012557_article.mjs`

We use **Kysely** to build our SQL queries in a typesafe way. We use that for our migrations as well.

```js
//  packages/core/migrations/16500000012557_article.mjs

import { Kysely } from "kysely";

export async function up(db) {
  await db.schema
    .createTable("article")
    .addColumn("articleID", "text", (col) => col.primaryKey())
    .addColumn("title", "text", (col) => col.notNull())
    .addColumn("url", "text", (col) => col.notNull())
    .addColumn("created", "timestamp", (col) => col.defaultTo("now()"))
    .execute();

  await db.schema
    .createIndex("idx_article_created")
    .on("article")
    .column("created")
    .execute();
}

export async function down(db) {
  await db.schema.dropIndex("idx_article_created").execute();
  await db.schema.dropIndex("idx_article_created").execute();
}
```

In this case, our migration is creating a table, called `article`, to store the links that are submitted. We are also adding an index to fetch them. The `down` function just removes the table and the index.

- Migration files are named with a timestamp to prevent naming conflicts when you are working with your team.

You can create a new migration by running `npm run gen migration new`. This command will ask for the name of a migration and it'll generate a new file with the current timestamp.

> Click on the **Migrations** button on the top right. And click the **Apply** button on the **article** migration.

This will create a table named `article`.

In the **Migrations** tab you'll see all the migrations in our app, and their status.

## Run a query

> To verify that the table has been created successfully; enter the following query into the query editor, and hit **Execute**.

```sql

SELECT * FROM article

```

You should see the query returns 0 rows.

### Behind The Scenes

1. We ran `sst dev` to start **Live Lambda Dev** environment and the **SST Console**.

2. Deployed the infraestructure for our app to AWS. Including a **RDS PostgreSQL** database based on `stacks/Database.ts`.

3. We then opened up the Console and ran a migration in `packages/core/migrations`.

4. It created an `article` table that we'll use to store the links our users will submit.

## Start The Frontend

You can start your frontend app locally, like you normally would. And it can connect to the API that's running using **SST's Live Lambda Dev**. That wey you can make changes live in your API and it'll reflect right away in the frontend.

> cd packages/web

> npm run dev

You should see the following in your terminal.

```
vite ... dev server running at:

> Local: http://localhost:3000/
> Network: use `--host` to expose

ready in ms

```

You should see the homepage of our app.

Over on the **Console**; you'll find the **Live Lambda** logs in the **Local** tab.

There, should see a `POST /graphql` request that was made. And the response body should say `"articles":[]`.

### Behind The Scenes

This seemingly simple workflow:

1. Your frontend is running locally.

2. It makes a request to a GraphQL endpoint that's running in AWS.

3. That invokes a Lambda function in AWS.

4. The Lambda function request is then proxied to your local machine.

5. The local version of that function is run.

6. It makes a query to an RDS Postgres database that's in AWS.

7. The logs for the function execution are displayed in the Console.

8. The results of that execution are sent back to AWS.

9. Your frontend then renders those results.

Note that everything here happens in real-time. There's no polling or syncing!

### Post an article

Type in `Learning sst` as the title and `https://sst.dev` for the URL. Click **Submit**.

You should see a page with the new article.

Again if we head back to the Console, you should se a new `POST /graphql` request. This time, creating the new article.

## Set a breakpoint

Open `packages/core/src/article.ts` and set a breakpoint in the `list` function.

The `list` domain function is called by our GraphQL API to get the list of all the articles that have been submitted. So it'll get run when we load our app homepage.

We currently have `sst dev` running in our terminal. Let's switch over to debugging through VS Code.

First stop the `sst dev` CLI.

Then select the **Run and Debug** tab in the top left menu in VSCode, and click **Start Debugging** at the top.

Go back to our frontend and refresh the homepage. You should see it hit our breakpoint.

Now you can browse the values of the variables in our code. You can also inspect the call stack leading up to the function call.

Once you are done debugging, hit **Continue** to resume the execution.

Now you have a good feel for SST's local development environment. Let's start working on our app.

# Domain Driven Desing

So we are ready to start working on our app. We'll be adding a simple comments feature.

And the setup that `create sst` generates reflects **Domain Driven Design(DDD)** pattern.

### What is DDD

In practice, this looks like creating a collection of modules and functions in the `packages/core` directory that implements the capabilities of your system.

> The idea of DDD is to have a separate layer that holds your business logic.

In the starter we provide a `core/src/article.ts` module which contains actions you can take that are related to the business concept of _Articles_. It exposes hihg level functions that handle the work of talking to the database, storing and retrieving them, and allows for more complex functionality in the future - like publishing notifications to an event bus.

The API and Lambda function code are unaware of these details and simply call into these modules to compose the logic together.

# Write to the Database

### Scaffod business logic

We'll start by scaffolding the domain code first. We'll add this to our `core` package.

Open up `packages/core/src/article.ts` and add the following two functions to the bottom of the file.

```ts
export function addComment(articleID: string, text: string) {
  // code for adding a comment to an article
}

export function comments(articleID: string) {
  // code for getting a list of comments of an article
}
```

Before we can implement them, we'll need to create a new table to store the comments.

### Create a migration

Run this in the **root** of the project to create a new migration.

```sh
npm run gen migration new

```

It'll aks you to name your migration.

```sh
? Migration name > comment
```

Once the migration is created, you should see the following in your terminal.

```sh
  Migration name - comment

  Loaded templates: _templates
    added: packages/core/migrations/99974749...mjs
```

Open up the new migration script and replace its content with:

```ts
// packages/core/migration/9091_comment.mjs

import { Kysely } from "kysely";

export async function up(db) {
  await db.schema
    .createTable("comment")
    .addColumn("commentID", "text", (col) => col.primaryKey())
    .addColumn("articleID", "text", (col) => col.notNull())
    .addColumn("text", "text", (col) => col.notNull());
    .execute()
}

export async function down(db){
  await db.schema.dropTable("comment").execute()
}
```

This migration will create a new table called `comment`. While undoing the migration will drop the table.

### Run a migration

Go to the RDS tab in SST Console and Click **Apply** on our `comment` migration.

To verify: `SELECT * FROM comment`

You should see 0 row being returned.

### Query the table

Ready to implement the `addComment` and `comments` functions.

```ts
//  packages/core/src/article.ts

export function addComment(articleID: string, text: string) {
  return SQL.DB.insertInto("comment")
    .values({
      commentID: ulid(),
      articleID,
      text,
    })
    .returningAll()
    .executeTakeFirstOrThrow();
}

export function comments(articleID: string) {
  return SQL.DB.selectFrom("comment")
    .selectAll()
    .where("articleID", "=", articleID)
    .execute();
}
```

We are using _Kysely_ to run typesafe queries against our database.

1. `SQL.DB` is the kysely instance imported from `packages/core/src/sql.ts`

2. `RDS` is comming from the SST Node client package.

```ts
//  packages/core/src/sql.ts

import { RDS } from "sst/node/rds";
```

It has access to the config of our database, thanks to _Resource Binding_. You might recall us **binding** our database to the functions in our API.

```ts
//  stacks/Api.ts

function: {
  bind:[rds],
}
```

By binding the `rds` cluster to our API in `stacks/Api.ts`, our API can access the database ARN (an ARN is an AWS identifier), database name, and ARN of the secret to access the database in our functions.

3. The Kysely instance also needs a `Database` type. This is comming from `packages/core/src/sql.generated.ts`

```ts
//  packages/core/src/sql.generated.ts

export interface Database {
  article: Article;
  comment: Comment;
}
```

The keys of this interface are the table names in our database. And they in turn point to other interfaces that list the column types of the respective tables. For example, here's the new `Comment` table we just created:

```ts
export interface Comment {
  articleID: string;
  commentID: string;
  text: string;
}
```

4. The `sql.generated.ts` types file, as you might've guessed in auto-generated. Our infraestructure code generates this when a new migration is run!

It's defined in `stacks/Database.ts`

```ts
const rds = new RDS(stack, "rds", {
  engine: "postgresql11.13",
  migrations: "packages/core/migrations",
  types: "packages/core/src/sql.generated.ts",
  defaultDatabaseName: "main",
});
```

Event though this file is auto-generated, you should check it into Git.

Now with our business logic and database queries implemented, we are ready to hook up our API.
