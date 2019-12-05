## re:Invent 2019 SVS217

Welcome to the re:Invent SVS217 builder session! This README walks you through steps to use the AWS SAM CLI to create, package, and publish an [AWS SAM](https://aws.amazon.com/serverless/sam/) serverless application to the [AWS Serverless Application Repository](https://aws.amazon.com/serverless/serverlessrepo/) (AWS SAR).

## Prerequisites

In order to complete this session, you must first complete the following prerequisites:

1. If you do not already have an AWS account, [create one](https://portal.aws.amazon.com/billing/signup#/start).
1. [Install](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html) and [configure](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html) the AWS CLI with credentials to make calls to your AWS account.
1. [Install](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html) the AWS SAM CLI. Please note:
    1. You do not need to install Docker to complete the steps of this tutorial.
    1. If you already have AWS SAM CLI installed, ensure you are running version 0.34.0 or later by running `sam --version`. If you're on an older version, upgrade your SAM CLI installation.

## Hello World App

First, let's get familiar with SAM CLI by using it to initialize, build, and deploy a simple hello world app. Walk through Steps 1-3 of [Tutorial: Deploying a Hello World Application](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-getting-started-hello-world.html) from the AWS SAM Getting Started documentation.

By the end of this tutorial, you should have deployed a sample hello world endpoint and been able to make a request to it using the `curl` command.

## Publishing your app to SAR

Now that you've created a sample application using SAM, you can use SAM CLI to publish it to the AWS Serverless Application Repository (AWS SAR), allowing you to share your application with others who can deploy it without needing any special developer tools.

### Add application metadata

The first step is to add a `Metadata` section to your AWS SAM template file providing the required application information. Copy/paste the following into the template.yaml file in your project directory and replace `<your name>` with your author name:

```yaml
Metadata:
  AWS::ServerlessRepo::Application:
    Name: sam-app
    Description: hello world
    Author: <your name>
    SpdxLicenseId: MIT-0
    LicenseUrl: LICENSE
    ReadmeUrl: README.md
    Labels: ['hello-world', 'reinvent2019']
    HomePageUrl: https://github.com/aws-samples/reinvent2019-svs217
    SemanticVersion: 0.0.1
    SourceCodeUrl: https://github.com/aws-samples/reinvent2019-svs217
```

### Add a license file

In the above sample metadata section, it specifies the MIT-0 (MIT no attribution) license as the license for the project. The LicenseUrl field specifies that the text of the license can be found in a file called `LICENSE` in the same directory as the SAM template. Go ahead and add that file. You can find the text of the MIT-0 license in the README of [this GitHub project](https://github.com/aws/mit-0).

NOTE: You do not have to specify a license and license url in order to publish an app to SAR, but apps without an open source license cannot be shared publicly. We're adding this license here since we'd like to be able to share the app publicly later on in this tutorial.

### Create a S3 bucket for packaging your SAR apps

Next, use the AWS CLI to create a S3 bucket that will be used for packaging your application for publishing to SAR.

```sh
$ aws s3 mb s3://<your bucket name>
```

NOTE: You must choose a bucket name that is globally unique, e.g., `<your username>-sar-publishing`

### Give SAR access to get objects from your bucket

Next, you need to give SAR access to get objects from your packaging bucket. You can do this by adding a bucket policy to the bucket giving SAR `GetObject` permissions. Copy/paste the following bucket policy into a file, e.g., bucket-policy.json:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service":  "serverlessrepo.amazonaws.com"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::<your bucket name>/*"
        }
    ]
}
```

NOTE: Make sure to replace `<your bucket name>` in the above policy with the actual name of your bucket.

Now use the AWS CLI to put the bucket policy to your bucket:

```sh
$ aws s3api put-bucket-policy --bucket <your bucket name> --policy file://bucket-policy.json
```

You can verify that the bucket policy operation succeeded by running the following command and verifying the returned policy is the one you put on the bucket:

```sh
$ aws s3api get-bucket-policy --bucket <your bucket name>
```

### Build and package your template

The next step is to build and package your template:

```sh
$ sam build
```

This should build a template and place it in `.aws-sam/build/template.yaml` by default. Next, we'll package the built template using our publishing S3 bucket:

```sh
$ sam package --template-file .aws-sam/build/template.yaml --s3-bucket <your bucket name> --output-template-file packaged-template.yaml
```

### Publish your template to SAR

Finally, we'll publish the packaged template to SAR:

```sh
$ sam publish --template-file packaged-template.yaml
```

You can view your published app by going to the SAR console: https://console.aws.amazon.com/serverlessrepo/home

## Deploying your application

So far, you've been deploying your application to your AWS account by building, packaging, and deploying it using AWS SAM CLI. This requires that you have the source code and the appropriate development tools installed on your system. However, once you publish your app to SAR, anyone with access to the app can deploy it to their AWS account without having any tools installed.

Let's deploy your app from SAR:

1. Visit the [AWS Serverless Application Repository Console](https://console.aws.amazon.com/serverlessrepo/home)
1. Click "Available applications" on the left navigation bar.
1. Click the "Private applications" tab since, by default, your published application is private to your account.
1. Click on your app. You will be redirected to a page to Review, configure and deploy your application.
1. Enter a name for your application (this will be part of the stack name), fill in any required parameters, and click "Deploy".

This will deploy a CloudFormation stack to your AWS account containing an instance of your SAR application. By publishing your app to SAR, others can now easily deploy your application into their AWS account with just a few clicks, no developer tools required.

## Sharing your application

By default, apps published to SAR are private to the publishing account, meaning only the publishing account has search and deploy permissions to the app. However, you can privately share your app with a specified list of AWS account ids or you can share your app publicly with any AWS account.

1. Visit the [AWS Serverless Application Repository Console](https://console.aws.amazon.com/serverlessrepo/home)
1. Click on the application you've published to see details for that application.
1. Look for the section titled "Share with AWS accounts" in the lower right corner. You can use this section to share your application with individual AWS account ids or toggle your app to be public. Experiment with both options. You can specify any 12-digit number as an account id, e.g., `123456789012`. However, note, the account id could be a real AWS account id, so you should remove it before you're finished.
1. When you toggle an app to be shared publicly, you will be able to search and deploy it from the AWS SAR [public application search](https://serverlessrepo.aws.amazon.com/).

## Updating your application

We will now use `sam publish` to publish updates to your application.

### Updating application metadata

You can update certain application metadata fields simply by updating the Metadata section of the template, building, packaging, and publishing. For example, to update the app description, update the `Description` field in the application template metadata, then run the build, package, and publish commands like before:

```sh
$ sam build
$ sam package --template-file .aws-sam/build/template.yaml --s3-bucket <your bucket name> --output-template-file packaged-template.yaml
$ sam publish --template-file packaged-template.yaml
```

### Publishing a new application version

Now we're going to add a new feature to our application and publish a new version of the application to SAR. The initial version of our app creates a serverless REST API that returns the a JSON object with the message "hello world". We're going to add a feature to our app that allows the user to pass in a parameter containing the message text they would like the REST API to return.

#### Add a template parameter for the message

In your `template.yaml` file, add a `Parameters` section with a string parameter called `Message` with a default value of `hello world`:

```yaml
Parameters:
  Message:
    Type: String
    Default: 'hello world'
```

NOTE: The default value is added for backwards-compatibility. This way if any consumers have already deployed the previous version of the app, they can deploy the new version of the app without it breaking the previous functionality. If we had not included a default value, the app would now require this new parameter to be given a value by the consumer, which would break existing installations of the app.

#### Pass the parameter to your app lambda function

Now we will pass this new parameter's value to the API Lambda function using Lambda environment variables. In your `template.yaml` file, add the following within the `Properties` section of `HelloWorldFunction`:

```yaml
      Environment:
        Variables:
          MESSAGE: !Ref Message
```

This initializes the Lambda function with an environment variable `MESSAGE` containing the template parameter value passed as Message.

#### Update the app logic

Now we will update the logic of the application Lambda function to return the contents of the environment variable instead of the hard-coded 'hello world' message. The exact code will vary depending on the language, but for example, if you chose python3.7, you would update `hello_world/app.py` adding `import os` to the top and updating the return statement from this:

```python
    return {
        "statusCode": 200,
        "body": json.dumps({
            "message": "hello world",
            # "location": ip.text.replace("\n", "")
        }),
    }
```

to this:

```python
    return {
        "statusCode": 200,
        "body": json.dumps({
            "message": os.environ['MESSAGE'],
            # "location": ip.text.replace("\n", "")
        }),
    }
```

`os.environ['MESSAGE']` is how you retrieve an environment variable's value in python. Other languages will have different syntax for accessing environment variables. For example, in Java you would use `System.getenv("MESSAGE")`

#### Verify the new feature works

Before publishing a new version of the app to SAR, we will deploy and test it to make sure our changes work as expected.

```sh
$ sam build
$ sam deploy --guided
$ curl https://<restapiid>.execute-api.us-east-1.amazonaws.com/Prod/hello/
```

#### Publish the updated version to SAR

Once the new feature is verified to work, the next step is to publish it to SAR as a new version of the app. The first step is to update the SemanticVersion of the app in the template Metadata. Since the new feature was added in a backwards-compatible way, we can do a minor version update and change the SemanticVersion from 0.0.1 to 0.1.0.

```yaml
Metadata:
  AWS::ServerlessRepo::Application:
    Name: sam-app
    ...
    SemanticVersion: 0.1.0
```

NOTE: To learn more about semantic versioning, visit https://semver.org/

Finally, build, package, publish:

```sh
$ sam build
$ sam package --template-file .aws-sam/build/template.yaml --s3-bucket <your bucket name> --output-template-file packaged-template.yaml
$ sam publish --template-file packaged-template.yaml
```

## Extra Credit: Nested Applications

AWS SAR apps can easily be deployed via the AWS Console with just a few clicks. However, there is a more advanced way to consume SAR apps by embedding them in your SAM template as a serverless application. In this step, we'll create a new SAM template that embeds a SAR app as a nested application.

First, create a new SAM template called `nested-app.yaml` with this content:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: SAR nested application example

Resources:
  Topic:
    Type: AWS::SNS::Topic
```

This template defines a serverless application containing an [Amazon Simple Notification Service (SNS)](https://aws.amazon.com/sns) topic. Now we will embed a SAR app that connects to the Amazon SNS topic, and saves any messages published to the topic to an S3 bucket.

1. Go to the AWS SAR [public application search](https://serverlessrepo.aws.amazon.com/applications) and search for "event storage", and look for an app called `fork-event-storage-backup-pipeline` published by AWS in the search results. Note, you will have to check the "Show apps that create custom IAM roles or resource policies" checkbox just below the search text input in order to find this app. Once you find the app, click on it to see its details. If you have trouble finding this application, here is a [direct link](https://serverlessrepo.aws.amazon.com/applications/arn:aws:serverlessrepo:us-east-1:077246666028:applications~fork-event-storage-backup-pipeline).
1. Click the Deploy button to be redirected to the AWS Console.
1. Click on the "Copy as SAM Resource" button. This copies the code necessary to embed this app as a nested application in your SAM template.
1. Paste the nested application in the resources section of your SAM template. It will look something like this:

```yaml
  forkeventstoragebackuppipeline:
    Type: AWS::Serverless::Application
    Properties:
      Location:
        ApplicationId: arn:aws:serverlessrepo:us-east-1:077246666028:applications/fork-event-storage-backup-pipeline
        SemanticVersion: 1.0.0
      Parameters:
        # [Optional] The ARN of the S3 bucket to which incoming events are loaded. If you don't enter any value, then a new S3 bucket is created in your account.
        # BucketArn: '' # Uncomment to override default value
        # [Optional] The ARN of the Lambda function used for transforming the incoming events. If you don’t enter any value, then data transformation is disabled.
        # DataTransformationFunctionArn: '' # Uncomment to override default value
        # [Optional] The level used for logging the execution of the Lambda function that polls events from the SQS queue. Four options are available, namely DEBUG, INFO, WARNING, and ERROR. If you don’t enter any value, then INFO is used.
        # LogLevel: 'INFO' # Uncomment to override default value
        # [Optional] The amount of seconds for which the stream should buffer incoming events before delivering them to the destination. Any integer value from 60 to 900 seconds. If you don't enter any value, then 300 is used.
        # StreamBufferingIntervalInSeconds: '300' # Uncomment to override default value
        # [Optional] The amount of data, in MB, that the stream should buffer before delivering them to the destination. Any integer value from 1 to 100. If you don't enter any value, then 5 is used.
        # StreamBufferingSizeInMBs: '5' # Uncomment to override default value
        # [Optional] The format used for compressing the incoming events. Three options are available, namely GZIP, ZIP, and SNAPPY. If you don’t enter any value, then data compression is disabled.
        # StreamCompressionFormat: 'UNCOMPRESSED' # Uncomment to override default value
        # [Optional] The string prefix used for naming files stored in the S3 bucket. If you don’t enter any value, then no prefix is used.
        # StreamPrefix: '' # Uncomment to override default value
        # [Optional] The SNS subscription filter policy, in JSON format, used for filtering the incoming events. The filter policy decides which events are processed by this pipeline. If you don’t enter any value, then no filtering is used, meaning all events are processed.
        # SubscriptionFilterPolicy: '' # Uncomment to override default value
        # The ARN of the SNS topic to which this instance of the pipeline should be subscribed.
        TopicArn: YOUR_VALUE
```

This application has many parameters, but only `TopicArn` is required, which is why the other parameters are commented out. For `TopicArn`, we will reference the Amazon SNS topic created in the template. Also, let's lower the `StreamBufferingIntervalInSeconds` parameter from the default 300 seconds to 60 seconds so we don't have to wait as long for the events to be saved to the S3 bucket. We'll also remove the other commented out parameters to reduce clutter in the template. After these changes, your template should look something like this:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: SAR nested application example

Resources:
  Topic:
    Type: AWS::SNS::Topic

  forkeventstoragebackuppipeline:
    Type: AWS::Serverless::Application
    Properties:
      Location:
        ApplicationId: arn:aws:serverlessrepo:us-east-1:077246666028:applications/fork-event-storage-backup-pipeline
        SemanticVersion: 1.0.0
      Parameters:
        # The ARN of the SNS topic to which this instance of the pipeline should be subscribed.
        TopicArn: !Ref Topic
        # [Optional] The amount of seconds for which the stream should buffer incoming events before delivering them to the destination. Any integer value from 60 to 900 seconds. If you don't enter any value, then 300 is used.
        StreamBufferingIntervalInSeconds: '60'
```

Now we can deploy this template using SAM CLI.

```sh
$ sam deploy --template-file nested-app.yaml --stack-name nested-app --capabilities CAPABILITY_IAM CAPABILITY_AUTO_EXPAND
```

The deployed stack will contain a SNS topic as well as a *nested* CloudFormation stack containing the `fork-event-storage-backup-pipeline` app. The app creates an S3 bucket, subscribes to the SNS topic and saves any messages published to the topic to the S3 bucket. Let's publish some test messages to the SNS topic and verify the app saves them to the S3 bucket as expected.

1. Go to the [S3 Console](https://s3.console.aws.amazon.com/s3/home) and find the bucket created by the app. If you named your stack `nested-app`, you can type `nested-app` into the search to find the bucket. It will be named something like `nested-app-forkeventstoragebackuppip-backupbucket-1np5b3deqw6wv`. Click on the bucket and verify it is empty.
1. In a separate tab, go to the [SNS Console](https://console.aws.amazon.com/sns). Click "Topics" in the left navigation bar. You should see the SNS topic created by CloudFormation. If you named your stack `nested-app`, it will be named something like `nested-app-Topic-141M7JSS8Q4TY`. Click on it to view details about the topic.
1. Click the "Publish message" button in the upper left side of the screen. Publish a few test messages. You can enter any text, e.g., "test message 1", "test message 2", etc.
1. Go back to the S3 Console tab and refresh the bucket until you see it contains the test messages. Note, it can take up to a minute for messages to start appearing in the S3 bucket due to the `StreamBufferingIntervalInSeconds` parameter setting on the app.

## What's Next?

Congratulations! You've learned how to create a serverless application using AWS SAM, build, deploy, and publish it to the AWS Serverless Application Repository so other AWS accounts can easily discover and deploy it. From here, you can:

1. Continue to modify your application and publish new versions using SAM CLI. See the [AWS SAM Specification](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-specification.html) to learn more about SAM template anatomy and supported resource types.
1. Study an example [realworld serverless application](https://github.com/awslabs/realworld-serverless-application). This project serves as a case study of how to build a real world application using a combination of serverless technologies and approaches.

## References

1. [AWS SAM Documentation](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html)
    1. [Publishing Serverless Applications Using the AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-template-publishing-applications.html)
    1. [AWS SAM Template Metadata Section Properties](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-template-publishing-applications-metadata-properties.html)
    1. [AWS Serverless Application Model (AWS SAM) Specification](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-specification.html)
1. [AWS Serverless Application Repository Documentation](https://docs.aws.amazon.com/serverlessrepo/latest/devguide/what-is-serverlessrepo.html)
1. AWS Serverless Application Repository [Public Application Search](https://serverlessrepo.aws.amazon.com/applications)
1. [Realworld serverless application example](https://github.com/awslabs/realworld-serverless-application)

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

