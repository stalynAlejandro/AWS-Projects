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
