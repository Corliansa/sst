# Serverless Stack Toolkit (SST) [![npm](https://img.shields.io/npm/v/@serverless-stack/cli.svg)](https://www.npmjs.com/package/@serverless-stack/cli) [![Build Status](https://github.com/serverless-stack/serverless-stack/workflows/CI/badge.svg)](https://github.com/serverless-stack/serverless-stack/actions)

<img alt="Logo" align="right" src="https://raw.githubusercontent.com/serverless-stack/identity/main/sst.svg" width="20%" />

Serverless Stack Toolkit (SST) is an extension of [AWS CDK](https://aws.amazon.com/cdk/) that:

- Includes a complete [local development environment for Lambda](#local-lambda-development)
  - Supports remotely invoking local functions
  - Zero-config ES and TypeScript support using [esbuild](https://esbuild.github.io)
- Allows you to use [CDK with Serverless Framework](https://serverless-stack.com/chapters/using-aws-cdk-with-serverless-framework.html)

Getting help: [**Slack**][slack] / [**Twitter**](https://twitter.com/ServerlessStack) / [**Forums**](https://discourse.serverless-stack.com/)

## Quick Start

Create your first SST app.

```bash
$ npx create-serverless-stack@latest my-sst-app
$ cd my-sst-app
$ npx sst start
```

<p>
<img src="https://d1ne2nltv07ycv.cloudfront.net/SST/sst-start-demo/sst-start-demo-1356x790.gif" width="600" alt="sst start" />
</p>

## Table of Contents

- [Background](#background)
- [Usage](#usage)
  - [Creating an app](#creating-an-app)
  - [Working on your app](#working-on-your-app)
  - [Developing locally](#developing-locally)
  - [Building your app](#building-your-app)
  - [Testing your app](#testing-your-app)
  - [Deploying your app](#deploying-your-app)
  - [Removing an app](#removing-an-app)
  - [Package scripts](#package-scripts)
  - [Linting, type checking](#linting-type-checking)
- [Example Project](#example-project)
- [Migrating From CDK](#migrating-from-cdk)
- [Known Issues](#known-issues)
- [Future Roadmap](#future-roadmap)
- [Contributing](#contributing)
- [Running Locally](#running-locally)
- [References](#references)
  - [`@serverless-stack/cli`](https://github.com/serverless-stack/serverless-stack/tree/master/packages/cli)
  - [`create-serverless-stack`](https://github.com/serverless-stack/serverless-stack/tree/master/packages/create-serverless-stack)
  - [`@serverless-stack/resources`](https://github.com/serverless-stack/serverless-stack/tree/master/packages/resources)
- [Community](#community)

---

## Background

### Local Lambda Development

Developing Lambdas locally is painful, you either:

1. Locally mock all the AWS services you are using
2. Or, constantly deploy your changes to test them

Both these approaches don't work well in practice. Locally mocking all the AWS services can be hard to do and most setups are really flaky. While, constantly deploying your Lambda functions or infrastructure can be simply too slow.

The `sst start` command starts up a local development environment that opens a WebSocket connection to your deployed app and proxies any Lambda requests to your local machine. This allows you to:

- Work on your Lambda functions locally
- While, interacting with your entire deployed AWS infrastructure
- Supports all Lambda triggers, so there's no need to mock API Gateway, SQS, SNS, etc.
- Supports real Lambda environment variables and Lambda IAM permissions
- So if a Lambda fails on AWS due to lack of IAM permissions, it would fail locally as well
- And it's fast. There's nothing to deploy when you make a change!

You can read more about the [**sst start** command here](https://github.com/serverless-stack/serverless-stack/tree/master/packages/cli#start) and [try out a demo here](https://github.com/serverless-stack/sst-start-demo).

### Using Serverless Framework with CDK

[Serverless Framework](https://github.com/serverless/serverless) is great but deploying any other AWS resources requires you to write CloudFormation templates in YAML. CloudFormation templates are incredibly verbose and even creating simple resources can take hundreds of lines of YAML. AWS CDK solves this by allowing you to generate CloudFormation templates using modern programming languages. Making it truly, _infrastructure as code_.

However, to use AWS CDK alongside your Serverless Framework services, requires you to follow certain conventions.

- **Deploying all the stacks to the same region and AWS account**

  Serverless Framework apps are deployed to multiple environments using the `--region` and `AWS_PROFILE=profile` options. CDK apps on the other hand, contain CloudFormation stacks that are deployed to multiple regions and AWS accounts simultaneously.

- **Prefixing stage and resource names**

  Since the same app is deployed to multiple environments, Serverless Framework adopts the practice of prefixing the stack names with the stage name. On the other hand, to deploy a CDK app to multiple stages, you'd need to manually ensure that the stack names and resource names don't thrash.

SST provides the above out-of-the-box. So you can deploy your Serverless services using:

```bash
$ AWS_PROFILE=production serverless deploy --stage prod --region us-east-1
```

And use CDK for the rest of your AWS infrastructure:

```bash
$ AWS_PROFILE=production npx sst deploy --stage prod --region us-east-1
```

You can [read more about this here](https://serverless-stack.com/chapters/using-aws-cdk-with-serverless-framework.html).

### And more

As a bonus, SST also supports deploying your CloudFormation stacks asynchronously. So you don't have to waste CI build minutes waiting for CloudFormation to complete. [Seed](https://seed.run) natively supports concurrent asynchronous deployments for your SST apps. Making it 5x faster than other CI services. And SST deployments on Seed are free!

SST also comes with a few other niceties:

- Zero-config support for ES and TypeScript using [esbuild](http://esbuild.github.io)
- Automatically lints your code using [ESLint](https://eslint.org/)
- Runs your unit tests using [Jest](https://jestjs.io/)

Behind the scenes, SST uses [a lightweight fork of AWS CDK](https://github.com/serverless-stack/sst-cdk) to programmatically invoke the various CDK commands.

## Usage

### Creating an app

Create a new project using.

```bash
$ npx create-serverless-stack@latest my-sst-app
```

Or alternatively, with a newer version of npm or Yarn.

```bash
# With npm 6+
$ npm init serverless-stack@latest my-sst-app
# Or with Yarn 0.25+
$ yarn create serverless-stack my-sst-app
```

This by default creates a JavaScript/ES project. If you instead want to use **TypeScript**.

```bash
$ npm init serverless-stack@latest my-sst-app --language typescript
```

By default your project is using npm as the package manager, if you'd like to use **Yarn**.

```bash
$ npm init serverless-stack@latest my-sst-app --use-yarn
```

You can read more about the [**create-serverless-stack** CLI here](https://github.com/serverless-stack/serverless-stack/tree/master/packages/create-serverless-stack).

### Working on your app

Your app starts with a simple project structure.

```
my-sst-app
├── README.md
├── node_modules
├── .gitignore
├── package.json
├── sst.json
├── test
│   └── MyStack.test.js
├── lib
|   ├── MyStack.js
|   └── index.js
└── src
    └── lambda.js
```

It includes a config file in `sst.json`.

```json
{
  "name": "my-sst-app",
  "stage": "dev",
  "region": "us-east-1"
}
```

The **stage** and the **region** are defaults for your app and can be overridden using the `--stage` and `--region` options. The **name** is used while prefixing your stack and resource names.

The `lib/index.js` file is the entry point for your app. It has a default export function to add your stacks.

```jsx
import MyStack from "./MyStack";

export default function main(app) {
  new MyStack(app, "my-stack");

  // Add more stacks
}
```

Here you'll be able to access the stage, region, and name of your app using.

```js
app.stage; // "dev"
app.region; // "us-east-1"
app.name; // "my-sst-app"
```

In the sample `lib/MyStack.js` you can add the resources to your stack.

```jsx
import * as sst from "@serverless-stack/resources";

export default class MyStack extends sst.Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    // Define your stack
  }
}
```

Note that the stacks in SST use `sst.Stack` as imported from `@serverless-stack/resources`. As opposed to `cdk.Stack`. This is what allows SST to make sure that your stack names are prefixed with the stage names and are deployed to the region and AWS account that's specified through the CLI.

You can access the stage, region, and name of your app using.

```js
this.node.root.stage; // "dev"
this.node.root.region; // "us-east-1"
this.node.root.name; // "my-sst-app"
```

And if you need to prefix certain resource names so that they don't thrash when deployed to multiple stages, you can do the following in your stacks.

```js
this.node.root.logicalPrefixedName("MyResource"); // "dev-my-sst-app-MyResource"
```

The sample stack also comes with a Lambda function and API endpoint. The Lambda function is in the `src/` directory.

```js
new sst.Function(this, "Lambda", {
  entry: "src/lambda.js",
});
```

Notice that we are using the `sst.Function` instead of the `cdk.lambda.NodejsFunction`. This allows SST to locally invoke a deployed Lambda function.

You can read more about [**@serverless-stack/resources** here](https://github.com/serverless-stack/serverless-stack/tree/master/packages/resources).

### Developing locally

Let's start the local development environment.

```bash
# With npm
$ npx sst start
# Or with Yarn
$ yarn sst start
```

The first time you run this, it'll deploy your app and a stack that sets up the debugger. This can take a couple of minutes.

#### Making changes

The sample stack will deploy a Lambda function with an API endpoint. You'll see something like this in the output.

```bash
Outputs:
  ApiEndpoint: https://s8gecmmzxf.execute-api.us-east-1.amazonaws.com
```

If you head over to the endpoint, it'll invoke the Lambda function in `src/lambda.js`. You can try changing this file and hitting the endpoint again. You should **see your changes reflected right away**!

### Building your app

Once you are ready to build your app and convert your CDK code to CloudFormation, run the following from your project root.

```bash
# With npm
$ npx sst build
# Or with Yarn
$ yarn sst build
```

This will compile your ES (or TS) code to the `.build/` directory in your app. And the synthesized CloudFormation templates are outputted to `.build/cdk.out/`. Note that, you shouldn't commit the `.build/` directory to source control and it's ignored by default in your project's `.gitignore`.

### Testing your app

You can run your tests using.

```bash
# With npm
$ npm test
# Or with Yarn
$ yarn test
```

Internally, SST uses [Jest](https://jestjs.io/). You'll just need to add your tests to the `test/` directory.

### Deploying your app

Once your app has been built and tested successfully. You are ready to deploy it to AWS.

```bash
# With npm
$ npx sst deploy
# Or with Yarn
$ yarn sst deploy
```

This uses your **default AWS Profile**. And the **region** and **stage** specified in your `sst.json`. You can deploy using a specific AWS profile, stage, and region by running.

```bash
$ AWS_PROFILE=my-profile npx sst deploy --stage prod --region eu-west-1
```

### Removing an app

Finally, you can remove all your stacks and their resources from AWS using.

```bash
# With npm
$ npx sst remove
# Or with Yarn
$ yarn sst remove
```

Note that, this permanently removes your resources from AWS. It also removes the stack that's created as a part of the debugger.

### Package scripts

The above commands (`start`, `build`, `deploy`, and `remove`) are also available in your `package.json`. So you can run them using.

```bash
# With npm
$ npm run <command>
# Or with Yarn
$ yarn run <command>
```

Just note that for `npm run`, you'll need to use an extra `--` for the options. For example:

```bash
$ npm run build -- --stage alpha
```

### Linting, type checking

Your code is automatically linted when building or deploying. If you'd like to customize the lint rules, add a `.eslintrc.json` in your project root. If you'd like to turn off linting, add `*` to an `.eslintignore` file in your project root.

If you are using TypeScript, SST also runs a separate TypeScript process to type check your code. It uses the `tsconfig.json` in your project root for this.

Note that, this applies to the Lambda functions in your app as well.

## Example Project

We use SST as a part of the [Serverless Stack guide](https://serverless-stack.com). We build a [simple notes app](http://demo2.serverless-stack.com/) in the guide and the backend for it is created using Serverless Framework and CDK with SST. You can check out the repo here — [serverless-stack-demo-api](https://github.com/AnomalyInnovations/serverless-stack-demo-api).

## Migrating From CDK

It's fairly simple to move a CDK app to SST. There are a couple of small differences between the two:

1. There is no `cdk.json`

   If you have a `context` block in your `cdk.json`, you can move it to a `cdk.context.json`. You can [read more about this here](https://docs.aws.amazon.com/cdk/latest/guide/context.html). You'll also need to add a `sst.json` config file, as talked about above. Here is a sample config for reference.

   ```json
   {
     "name": "my-sst-app",
     "stage": "dev",
     "region": "us-east-1"
   }
   ```

2. There is no `bin/*.js`

   Instead there is a `lib/index.js` that has a default export function where you can add your stacks. SST creates the App object for you. This is what allows SST to ensure that the stage, region, and AWS accounts are set uniformly across all the stacks. Here is a sample `lib/index.js` for reference.

   ```js
   import MyStack from "./MyStack";

   export default function main(app) {
     new MyStack(app, "my-stack");

     // Add more stacks
   }
   ```

3. Stacks extend `sst.Stack`

   Your stack classes extend `sst.Stack` instead of `cdk.Stack`. Here is what the JavaScript version looks like.

   ```js
   import * as sst from "@serverless-stack/resources";

   export default class MyStack extends sst.Stack {
     constructor(scope, id, props) {}
   }
   ```

   And in TypeScript.

   ```ts
   import * as sst from "@serverless-stack/resources";

   export class MyStack extends sst.Stack {
     constructor(scope: sst.App, id: string, props?: sst.StackProps) {}
   }
   ```

4. Lambdas use `sst.Function`

   Use the `sst.Function` construct instead to the `cdk.lambda.NodejsFunction`. You can read more about this over on [`@serverless-stack/resources`](https://github.com/serverless-stack/serverless-stack/tree/master/packages/resources) docs.

5. Include the right packages

   You don't need the `aws-cdk` package in your `package.json`. Instead you'll need `@serverless-stack/cli` and `@serverless-stack/resources`.

## Known Issues

### CDK Version Mismatch

There is a known issue in AWS CDK when using mismatched versions of their NPM packages. This means that all your AWS CDK packages in your `package.json` should use the same exact version. And since SST uses a forked version of AWS CDK internally, this means that your app needs to use the same versions as well.

To help with this, SST will show a message to let you know if you might potentially run into this issue. And help you fix it.

```bash
Mismatched versions of AWS CDK packages. Serverless Stack currently supports 1.55.0. Fix using:

  npm install @aws-cdk/aws-cognito@1.55.0 --save-exact
```

We also created a convenience method to help install the CDK npm packages with the right version — [`sst add-cdk`](https://github.com/serverless-stack/serverless-stack/tree/master/packages/cli#add-cdk-packages).

So instead of:

```bash
$ npm install @aws-cdk/aws-s3 @aws-cdk/aws-iam
```

You can do:

```bash
$ npx sst add-cdk @aws-cdk/aws-s3 @aws-cdk/aws-iam
```

And it'll install those packages using the right CDK versions.

You can learn more about these issues [here](https://github.com/aws/aws-cdk/issues/9578) and [here](https://github.com/aws/aws-cdk/issues/542).

## Future Roadmap

Check out [**the public SST roadmap here**][roadmap].

## Contributing

Check out our [roadmap][roadmap] and [join our Slack][slack] to get started.

- Open [a new issue](https://github.com/serverless-stack/serverless-stack/issues/new) if you've found a bug or have some suggestions.
- Or submit a pull request!

## Running Locally

To run this project locally, clone the repo and initialize the project.

```bash
$ git clone https://github.com/serverless-stack/serverless-stack.git
$ cd serverless-stack
$ yarn
```

Run all the tests.

```bash
$ yarn test
```

## References

- [`@serverless-stack/cli`](https://github.com/serverless-stack/serverless-stack/tree/master/packages/cli)
- [`create-serverless-stack`](https://github.com/serverless-stack/serverless-stack/tree/master/packages/create-serverless-stack)
- [`@serverless-stack/resources`](https://github.com/serverless-stack/serverless-stack/tree/master/packages/resources)

## Community

[Follow us on Twitter](https://twitter.com/ServerlessStack), [join us on Slack][slack], [post on our forums](https://discourse.serverless-stack.com), and [subscribe to our newsletter](https://emailoctopus.com/lists/1c11b9a8-1500-11e8-a3c9-06b79b628af2/forms/subscribe).

## Thanks

This project extends [AWS CDK](https://github.com/aws/aws-cdk) and is based on the ideas from [Create React App](https://www.github.com/facebook/create-react-app).

---

Brought to you by [Anomaly Innovations](https://anoma.ly/); makers of [Seed](https://seed.run/) and the [Serverless Stack Guide](https://serverless-stack.com/).

[slack]: https://join.slack.com/t/serverless-stack/shared_invite/zt-kqna615x-AFoTXvrglivZqJZcnTzKZA
[roadmap]: https://github.com/serverless-stack/serverless-stack/milestones?direction=asc&sort=due_date&state=open
