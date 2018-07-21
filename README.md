# Cloudify

Ad-hoc serverless batch processing for nodejs.

## Install
[Yarn](https://yarnpkg.com/en/docs/install)
```
npm install -g yarn
```
[AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/installing.html)
```
pip install awscli --upgrade --user
```

## Building

```
$ yarn install
$ yarn build
```

## Testing live cloud services

### GCP
Setup credentials for [GCP](https://cloud.google.com/sdk/docs/authorizing)
* Create a project
* Create a google [service account](https://console.cloud.google.com/iam-admin/serviceaccounts)
* Assign Owner permissions for the service account
* Enable Cloud functions API
* Run basic Google test
```
npx jest build/test/google-https.test.js
```
### AWS
Setup credentials for [AWS](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html).
* Add credentials to system using aws cli (aws configure)
* Ensure AWS user has AdministratorAccess role and Administrator group
* Run basic AWS test
```
npx jest build/test/aws-basic.test.js
```

The test target also runs the test-local test cases, so installation of the
Google cloud functions emulator is required.

## Local Testing

First install google cloud functions emulator (unfortunately it must be
installed globally to work):

```
$ npm install -g @google-cloud/functions-emulator
```

Then run the test script:

```
$ DEBUG=cloudify yarn test-local
```



```
$ yarn test
```

To just run remote tests for a specific service provider:

```
yarn build
DEBUG=cloudify npx jest build/test/aws.test.js
DEBUG=cloudify npx jest build/test/google.test.js
```

# Principles

- Ephemeral functions: Cloudify cleans up after itself. It removes functions, roles, logs, and log groups. By default it will leave no trace in your infrastructure once cleanup is completed.
- Avoids resource exhaustion: doesn't take up log space, function name space, and other limits. Only currently executing functions take up infrastructure namespace.
- Independence: two separate jobs can be run at the same time and they will not interfere with each other.
- Works with AWS and Google Cloud Platform.

# AWS Notes

## IAM Roles

Cloudify will create an IAM role for the lambda function it creates. By default this role will have administrator access. The role will be created dynamically and then deleted when the cleanup function is called. Dynamically creating the role takes some time - up to 5-6 seconds. To speed this up, and also to allow for less permissive roles, you can create a persistent IAM role manually and restrict its permissions. Having a cached role will also make function startup faster because the role will not have to be dynamically created.

```typescript
const RoleName = "...cached role name...";
let cloud = cloudify.create("aws");
let service = await cloud.createFunction("./functions", {
 RoleName
});
```

There are a minimum set of policy permissions required, namely the ones in `arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole`:

```json
{
 "Version": "2012-10-17",
 "Statement": [
  {
   "Effect": "Allow",
   "Action": ["logs:CreateLogGroup", "logs:CreateLogStream", "logs:PutLogEvents"],
   "Resource": "*"
  }
 ]
}
```

If Cloudify cannot find the role name specified, it will revert back to dynamically creating a role for you.

If you want to have dynamically created role with custom permissions, specify the `PolicyArn` option. Cloudify will attach this policy to the role it dynamically creates.

# Google Cloud Notes

## Node version

Google Cloud Functions only supports node 6.11.5 at the moment (as of 6/2016).
If your code uses any Node APIs that were introduced after this version, it will
fail when run on Google Cloud. This can happen, for example, if your TypeScript
target uses features introduced in later node/V8 versions than 6.11.5. Though
not strictly required, it can be helpful to synchronize the node version on your
local machine with the remote cloud, which can be accomplished by adding the
following to your `package.json`:

```json
"engines": {
  "node": "6.11.5"
}
```

# Cleaning up stray resources

Use the `cloudify-cleanup` utility to clean up stray resources that may be left by cloudify in some rare instances (e.g. crashes where cleanup is not invoked):

```
$ node build/src/cloudify-cleanup.js aws
```

By default the utility runs in dry-run mode, only printing the actions it will
perform. To actually execute the cleanup, specify the `-x` option:

```
$ node build/src/cloudify-cleanup.js aws -x
```

# Concurrency

Response queue funnel: create a request listener for every 20 outstanding requests. With a minimum of 2.
