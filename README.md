# Workshop: Building a React PWA Chat Application

## Quicklinks

- [Introduction](#introduction)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Back End Setup](#back-end-setup)
  - [Interacting with Chatbots](#interacting-with-chatbots)
  - [Interacting with other AWS AI Services](#interacting-with-other-aws-ai-services)
- [Building, Deploying and Publishing with the Amplify CLI](#building-deploying-and-publishing-with-the-amplify-cli)
- [Back End Setup, Back End and Front End Building, Deploying and Publishing with the Amplify Console](#back-end-setup-back-end-and-front-end-building-deploying-and-publishing-with-the-amplify-console)
- [Clean Up](#clean-up)

## Introduction

This is a Starter React Progressive Web Application (PWA) that uses AWS AppSync to implement offline and real-time capabilities in a chat application with AI/ML features such as image recognition, text-to-speech, language translation, sentiment analysis as well as conversational chatbots developed as part of the re:Invent session [Bridging the Gap Between Real Time/Offline and AI/ML Capabilities in Modern Serverless Apps](https://www.youtube.com/watch?v=0H5F0PI2-SU). In the chat app, users can search for users and messages, have conversations with other users, upload images and exchange messages.

### Architecture

![ChatQL Overview](/media/ChatQLv2.png)

The application demonstrates GraphQL Mutations, Queries and Subscriptions with AWS AppSync integrating with other AWS Services:

- Amazon Cognito for user management as well as Authentication and Authorisation (*AuthN/Z*)
- Amazon DynamoDB with multiple data sources (Users, Messages, Conversations, ConvoLink)
- Amazon Elasticsearch data source for full text search on messages and users
- Amazon S3 for Media Storage
- AWS Lambda as a Serverless integration layer for connecting to AI Services
  - Amazon Comprehend for sentiment and entity analysis as well as language detection
  - Amazon Rekognition for object, scene and celebrity detection on images
  - Amazon Lex for conversational chatbots
  - Amazon Polly for text-to-speech on messages
  - Amazon Translate for language translation

## Getting Started

### Prerequisites

- [AWS Account](https://aws.amazon.com/mobile/details) with appropriate permissions to create the related resources
- [NodeJS](https://nodejs.org/en/download/) with [NPM](https://docs.npmjs.com/getting-started/installing-node)
- [AWS CLI](http://docs.aws.amazon.com/cli/latest/userguide/installing.html) with output configured as JSON `(pip install awscli --upgrade --user)`
- [AWS Amplify CLI](https://github.com/aws-amplify/amplify-cli) configured for a region where [AWS AppSync](https://docs.aws.amazon.com/general/latest/gr/rande.html) and all other services in use are available `(npm install -g @aws-amplify/cli)`
- [AWS SAM CLI](https://github.com/awslabs/aws-sam-cli) `(pip install --user aws-sam-cli)`
- [AWS SAM Install Instructions](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html) 
- [Create React App](https://github.com/facebook/create-react-app) `(npm install -g create-react-app)`
- [Install JQ](https://stedolan.github.io/jq/) **macOS** `(brew install jq)` or **Windows** `(chocolatey install jq)`
- **If using Windows**, you'll need the [Windows Subsystem for Linux (WSL)](https://docs.microsoft.com/en-us/windows/wsl/install-win10)


### Region Selection

*Please Note: If you wish to integrate the AI features as part of this workshop. It's recommended that you launch the entire solution in one of these regions:*

* *us-east-1*
* *us-west-1*
* *eu-west-1*

*At the time of writing this workshop, Amazon Lex is only available in these regions.*

## Instructions
###Launch a Serverless Chat Application with AWS Amplify

1. First, clone this repository and navigate to the created folder:

   ```bash
   git clone https://github.com/StefanBuchman/aws-appsync-chat-workshop.git
   cd aws-appsync-chat-workshop
   ```

2. Install the required modules:

   ```bash
   npm install
   ```

3. Init the directory as an amplify **Javascript** app using the **React** framework:

#### The amplify init command is a one-time initialization step for your Amplify powered cloud app. You run this once for each project (JavaScript, iOS, or Android) to connect your app with an AWS backend.

   ```bash
   amplify init
   ```

#### Example Output:
![Amplify Init](/media/Init.png)


   Set the region we are deploying resources to:

   ```bash
   export AWS_REGION=$(jq -r '.providers.awscloudformation.Region' amplify/#current-cloud-backend/amplify-meta.json)
   echo $AWS_REGION
   ```

   Make sure [**ALL**](https://docs.aws.amazon.com/general/latest/gr/rande.html) services are supported in this region or else you'll get errors in the next steps.

4. Log into the console and head over to [CloudFormation](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks).

   You should see a new stack has been created with a:
   - Deployment bucket
   - Two IAM roles

5. Add an **Amazon Cognito User Pool** auth resource. 

   ```bash
   amplify add auth
   ```
   When prompted:

   "_Do you want to use the default authentication and security configuration?_"
   Answer: **Use the default configuration**

   This will create a local resource for a Cognito user pool.  We'll push this configuration up to the cloud shortly.
 

6. Add an **AppSync GraphQL** API with **Amazon Cognito User Pool** for the API Authentication. Follow the default options. When prompted with "_Do you have an annotated GraphQL schema?_", select **"YES"** and provide the schema file path `backend/schema.graphql`

   ```bash
   amplify add api
   ```

Amplify will create your API for you according to the schema defined.  It will additionally build out the DynamoDB tables and populate the data sources when Step 7 is executed.

#### Example Output:
![Amplify add api](/media/addApi.png)

6. Add S3 Private Storage for **Content (Images, audio, video, etc.)** to the project with the default options. Select private **read/write** access for **Auth users only**:

   ```bash
   amplify add storage
   ```

7. Now it's time to provision your cloud resources based on the local setup and configured features. When asked:

   **Do you want to generate code for your newly created GraphQL API**, answer **"No"** as it would overwrite the current custom files in the `src/graphql` folder.

   ```bash
   amplify push
   ```

   Wait for the provisioning to complete. Once done, a `src/aws-exports.js` file with the resources information is created.

At this point, go check out the resources that have been created as part of the `amplify push` command:

In your `src` directory you should have a file called: `aws-exports.js`.  This file contains the references to the series of resources we created in the cloud.

- [AppSync](https://console.aws.amazon.com/appsync/home?region=us-east-1#/apis) you should see your new API, made up of the Schema and Data Sources.

- [DynamoDB](https://console.aws.amazon.com/dynamodb/home?region=us-east-1#tables:) you should see 4 new tables to support the chat application.  These were created by Amplify based off of the GraphQL schema.

- [Cognito](https://console.aws.amazon.com/cognito/users/?region=us-east-1) a new Cognito User Pool should have been created to store credentials for your chat users.

- [S3](https://s3.console.aws.amazon.com/s3/home?region=us-east-1) there should be a deployment bucket as well as a new bucket to hold media from the chat application.

- [CloudFormation](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks) finally, you should see 4 new stacks, a primary and 3 nested stacks.  These ultimately provisioned the resouces during `init` and `push`.  When cleaning up, these stacks will also tear down the provisioned resources.

---

## Testing the chat app before adding AI features

1. Execute the following command to install your project package dependencies and run the application locally:

    ```bash
    amplify serve
    ```

2. Access your chat app at http://localhost:3000

3. Use two different browsers or one in Incognito/InPrivate mode.  Sign up at least 2 different users, authenticate with each user to get them registered in the backend Users table.

4. Search for your new users to start a conversation and test real-time/offline messaging.

5. Try to send an image, you should be able to go to your S3 bucket where you'll see the file being committed to.

6. Head back to Cognito and validate you can see your two new users.  You'll be able to view their details, validate that they confimed their identity and even initiate a reset of their password.

---

# Add AI Features to your Serverless Chat Application
## Getting AI supporting resources

1. Look up the S3 bucket name created for user storage:

   ```bash
   export USER_FILES_BUCKET=$(sed -n 's/.*"aws_user_files_s3_bucket": "\(.*\)".*/\1/p' src/aws-exports.js)
   echo $USER_FILES_BUCKET
   ```

2. Retrieve the API ID of your AppSync GraphQL endpoint

   ```bash
   export GRAPHQL_API_ID=$(jq -r '.api[(.api | keys)[0]].output.GraphQLAPIIdOutput' ./amplify/#current-cloud-backend/amplify-meta.json)
   echo $GRAPHQL_API_ID
   ```

3. Retrieve the project's deployment bucket and stackname . *It will be used for packaging and deployment with SAM*

    ```bash
    export DEPLOYMENT_BUCKET_NAME=$(jq -r '.providers.awscloudformation.DeploymentBucketName' ./amplify/#current-cloud-backend/amplify-meta.json)
    export STACK_NAME=$(jq -r '.providers.awscloudformation.StackName' ./amplify/#current-cloud-backend/amplify-meta.json)
    echo $DEPLOYMENT_BUCKET_NAME
    echo $STACK_NAME
    ```

## Lambda Functions

Now we need to deploy 3 Lambda functions (one for AppSync and two for Lex) and configure the AppSync Resolvers to use Lambda accordingly. 

First, we install the npm dependencies for each lambda function. We then package and deploy the changes with SAM. 

**Please Note:** If you have defined an **AWS Profile** for the AWS CLI remember to add `--profile profile-name` to the SAM or CLI commands below.

1. First, head to the Lambda functions under the backend directory.  Take a look through the functions and see what they're doing.

2. Let's get the dependencies installed and the functions packaged.

    ```bash
    cd ./backend/chuckbot-lambda; npm install; cd ../..
    cd ./backend/moviebot-lambda; npm install; cd ../..

    sam package --template-file ./backend/deploy.yaml --s3-bucket $DEPLOYMENT_BUCKET_NAME --output-template-file packaged.yaml

    export STACK_NAME_AIML="$STACK_NAME-extra-aiml"

    sam deploy --template-file ./packaged.yaml --stack-name $STACK_NAME_AIML --capabilities CAPABILITY_IAM --parameter-overrides appSyncAPI=$GRAPHQL_API_ID s3Bucket=$USER_FILES_BUCKET --region $AWS_REGION
    ```

3. Head over to [CloudFormation](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks) and validate that the stack has been created. *Wait for the stack to complete deploying.*

   Head over to your new Lambda functions and validate they have been created.


4. We'll now retrieve the ARN for the Lambda functions.  We'll need the ARN's to allow Lex to point to the correct function to fullfil an intent.

    ```bash
    export CHUCKBOT_FUNCTION_ARN=$(aws cloudformation describe-stacks --stack-name  $STACK_NAME_AIML --query "Stacks[0].Outputs" --region $AWS_REGION | jq -r '.[] | select(.OutputKey == "ChuckBotFunction") | .OutputValue')

    export MOVIEBOT_FUNCTION_ARN=$(aws cloudformation describe-stacks --stack-name  $STACK_NAME_AIML --query "Stacks[0].Outputs" --region $AWS_REGION | jq -r '.[] | select(.OutputKey == "MovieBotFunction") | .OutputValue')

    echo $CHUCKBOT_FUNCTION_ARN
    echo $MOVIEBOT_FUNCTION_ARN
    ```

   Your ARN's should look something like the following:
   `arn:aws:lambda:us-east-1:0123456789:function:awstest1-20190208143507-extra-aiml-ChuckBot-TZJDK8UUOJCC`

## Lex Chatbots

5. Let's set up Lex. We will create 2 chatbots: ChuckBot and MovieBot. 

   Execute the following commands to add permissions so Lex can invoke the chatbot related Lambda functions you created in the previous section:

    ```bash
    aws lambda add-permission --statement-id Lex --function-name $CHUCKBOT_FUNCTION_ARN --action lambda:\* --principal lex.amazonaws.com --region $AWS_REGION
    aws lambda add-permission --statement-id Lex --function-name $MOVIEBOT_FUNCTION_ARN --action lambda:\* --principal lex.amazonaws.com --region $AWS_REGION
    ```

    Update the bots intents with the Lambda ARN:

    ```bash
    jq '.fulfillmentActivity.codeHook.uri = $arn' --arg arn $CHUCKBOT_FUNCTION_ARN backend/ChuckBot/intent.json -M > tmp.txt ; cp tmp.txt backend/ChuckBot/intent.json; rm tmp.txt
    jq '.fulfillmentActivity.codeHook.uri = $arn' --arg arn $MOVIEBOT_FUNCTION_ARN backend/MovieBot/intent.json -M > tmp.txt ; cp tmp.txt backend/MovieBot/intent.json; rm tmp.txt
    ```

    And, deploy the slot types, intents and bots:

    ```bash
    aws lex-models put-slot-type --cli-input-json file://backend/ChuckBot/slot-type.json --region $AWS_REGION
    aws lex-models put-intent --cli-input-json file://backend/ChuckBot/intent.json --region $AWS_REGION
    aws lex-models put-bot --cli-input-json file://backend/ChuckBot/bot.json --region $AWS_REGION
    aws lex-models put-slot-type --cli-input-json file://backend/MovieBot/slot-type.json --region $AWS_REGION
    aws lex-models put-intent --cli-input-json file://backend/MovieBot/intent.json --region $AWS_REGION
    aws lex-models put-bot --cli-input-json file://backend/MovieBot/bot.json --region $AWS_REGION
    ```

6. Head over to [Lex]() and walk-through what has been created.
   
   You should see two Lex chatbots (ChuckBot & MovieBot), the bots may still be building.  While they are building take a look at:
   - The intents, these are the actions your chatbot users will attempt to have the chatbot undertake.  We can take slots (variables) as part of the intent to make the chatbot more dynamic
   - The slots, these are pieces of information we may need to fullfil an intent.
   - Finally, make sure each chatbot has a Lambda function defined in the Fullfilment section.  This is what will be executed once when know what our chatbot user is looking for.

---

## Interacting with Chatbots

_The chatbots retrieve information online via API calls from Lambda to [The Movie Database (TMDb)](https://www.themoviedb.org/) (MovieBot, which is based on this [chatbot sample](https://github.com/aws-samples/aws-lex-convo-bot-example)) and [chucknorris.io ](https://api.chucknorris.io/) (ChuckBot)_

1. Execute the following command to install your project package dependencies and run the application locally:

    ```bash
    amplify serve
    ```

2. Access your chat app at http://localhost:3000

3. Resume your previous conversation or start a new one.

4. In order to initiate or respond to a chatbot conversation, you need to start the message with either `@chuckbot` or `@moviebot` to trigger or respond to the specific bot, for example:

   - _@chuckbot Give me a Chuck Norris fact_
   - _@moviebot Tell me about a movie_

5. Each subsequent response needs to start with the bot handle (@chuckbot or @moviebot) so the app can detect the message is directed to Lex and not to the other user in the same conversation. Both users will be able to view Lex chatbot responses in real-time powered by GraphQL subscriptions.

6. Alternatively you can start a chatbot conversation from the message drop-down menu:

   - Just selecting `ChuckBot` will display options for further interaction
   - Send a message with a nothing but a movie name and selecting `MovieBot` subsequently will retrieve the details about the movie

### Interacting with other AWS AI Services

1. Click or select uploaded images to trigger Amazon Rekognition object, scene and celebrity detection.
2. From the drop-down menu, select LISTEN -> TEXT TO SPEECH to trigger Amazon Polly and listen to messages in different voices based on the message automatically detected source language (supported languages: English, Mandarin, Portuguese, French and Spanish).
3. To perform entity and sentiment analysis on messages via Amazon Comprehend, select ANALYZE -> SENTIMENT from the drop-down menu.
4. To translate the message select the desired language under TRANSLATE in the drop-down menu (supported languages: English, Mandarin, Portuguese, French and Spanish). In the translation pane, click on the microphone icon to listen to the translated message.

## Building, Deploying and Publishing with the Amplify CLI

1. Execute `amplify add hosting` from the project's root folder and follow the prompts to create an S3 bucket (DEV) and/or a CloudFront distribution (PROD).

2. Build, deploy, upload and publish the application with a single command:

   ```bash
   amplify publish
   ```

3. If you are deploying a CloudFront distribution, be mindful it needs to be replicated across all points of presence globally and it might take up to 15 minutes to do so.

4. Access your public ChatQL application using the S3 Website Endpoint URL or the CloudFront URL returned by the `amplify publish` command. Share the link with friends, sign up some users, and start creating conversations, uploading images, translating, executing text-to-speech in different languages, performing sentiment analysis and exchanging messages. Be mindful PWAs require SSL, in order to test PWA functionality access the CloudFront URL (HTTPS) from a mobile device and add the site to the mobile home screen.

## Back End Setup, Back End and Front End Building, Deploying and Publishing with the Amplify Console

1. Fork this repository into your own GitHub account and clone it
2. Install the Amplify CLI with multienv support:

    ```bash
    npm install -g @aws-amplify/cli@multienv
    ```

    More info [here](https://docs.aws.amazon.com/amplify/latest/userguide/deploy-backend.html).

3. Repeat Steps 3 to 6 from the [Back End Setup](#back-end-setup) in the previous section. Do not perform step 7 (`amplify push`).
4. Commit the changes to your forked repository. A new folder `amplify` will be commited with the project details.
5. Connect your repository to the [Amplify Console](https://console.aws.amazon.com/amplify/home?#/create) as per the instructions [here](https://docs.aws.amazon.com/amplify/latest/userguide/getting-started.html), making sure the name of the branch in your repository matches the name of the environment configured on `amplify init` (i.e. master). When prompted with "_We detected a backend created with the Amplify Framework. Would you like Amplify Console to deploy these resources with your frontend?_", select **"YES"** and provide or create an IAM role with appropriate permissions to build the backend resources
6. Wait for the build, deployment and verification steps

---

**_At this point you have an usable serverless chat application with no AI features. The next steps are only needed to deploy and configure the integration with services that provide image recognition, text-to-speech, language translation, sentiment analysis as well as conversational chatbots. From here you can skip to step 8 if there's no interest to setup the AI integration._**

---

7. Now perform steps 7 to 12 from the [Back End Setup](#back-end-setup)
8. Access your app from the hosted site generated by the Amplify Console(https://master.xxxxxxxx.amplifyapp.com)

## Clean Up

To clean up the project, you can simply delete the bots, delete the stack created by the SAM CLI:

```bash
aws lex-models delete-bot --name `jq -r .name backend/ChuckBot/bot.json` --region $AWS_REGION
aws lex-models delete-bot --name `jq -r .name backend/MovieBot/bot.json` --region $AWS_REGION
aws lex-models delete-intent --name `jq -r .name backend/ChuckBot/intent.json` --region $AWS_REGION
aws lex-models delete-intent --name `jq -r .name backend/MovieBot/intent.json` --region $AWS_REGION
aws lex-models delete-slot-type --name `jq -r .name backend/ChuckBot/slot-type.json` --region $AWS_REGION
aws lex-models delete-slot-type --name `jq -r .name backend/MovieBot/slot-type.json` --region $AWS_REGION

aws cloudformation delete-stack --stack-name $STACK_NAME_AIML --region $AWS_REGION
```

and use:

```bash
amplify delete
```

to delete the resources created by the Amplify CLI.
