# Building CI/CD workflows for Serverless Applications

This lab is provided as part of **[AWS Innovate Modern Applications Edition](https://aws.amazon.com/events/aws-innovate/apj/modern-apps/)**. 

Click [here](https://github.com/phonghuule/aws-innovate-modern-applications-2022) to explore the full list of hands-on labs.

ℹ️ You will run this lab in your own AWS account. Please follow directions at the end of the lab to remove resources to avoid future costs.

This lab walks you through creating a CI/CD workflow for serveress applications. 
## Content 

- [Getting Started](#getting-started)
- [Step 1. Deploying a lambda application](#step-1-deploying-a-lambda-application)
- [Step 2. Setting up the Code Repo and IDE](#step-2-setting-up-the-code-repo-and-ide)
- [Step 3. Safe Deployments](#step-3-safe-deployments)
- [Cleanup](#cleanup)
- [Summary](#summary)



### Getting started
In software engineering, **CI/CD** or **CICD** is the combined practices of continuous integration (CI) and either continuous delivery or continuous deployment (CD).

CI/CD bridges the gaps between development and operation activities and teams by enforcing automation in building, testing and deployment of applications. The process contrasts with traditional methods where all updates were integrated into one large batch before rolling out the newer version.

**Serverless applications** feature automatic scaling, built-in high availability, and a pay-for-use billing model to increase agility and optimize costs. These technologies also eliminate infrastructure management tasks like capacity provisioning and patching, so you can focus on writing code that serves your customers. Serverless applications start with AWS Lambda, an event-driven compute service natively integrated with over 200 AWS services and software as a service (SaaS) applications.

### Step 1. Deploying a lambda application
1. Navigate to the [AWS Lambda service](https://us-east-1.console.aws.amazon.com/lambda/home?region=us-east-1#/applications) and select the **Applications** option on the left pane.
2. Select Create application and select the **Serverless API backend** option.

![serverlessapi](/images/serverlessapibackendapp.png)

3. This will give us a RESTful web API that uses DynamoDB to manage state. Click on Next.
4. Enter a name for the application as ``serverless-start``
5. Select runtime as Node.js 14.x
6. We will use [AWS SAM](https://aws.amazon.com/serverless/sam/) for this application. Leave the Template format as AWS SAM(YAML).
7. In the Source control section we will use CodeStar Connections to create a private repository in your GitHub account. Select ``Create new connection`` and click on Connect with CodeStar Connections and follow the screenshot.
    
    ![codestartogithub](/images/codestartogithub.png)

8. Check the ``Create roles and permissions boundary`` option under Permissions and click on Create. This will take a few minutes for the application to create. An API endpoint is created which is backed by a Lambda function and a Dynamo DB. 
    
    ![serverlessappsetup](/images/serverlessappsetup.png)

    The Architecture created by the ``serverless-start`` app is as follows:

    ![serverlessArchitecture](/images/serverlessArchitecture.png)

9. Navigate to [AWS IAM](https://us-east-1.console.aws.amazon.com/iamv2/home?region=us-east-1#/home) and select Role on the left panel to display a list of roles
Look for CloudFormation role for project **serverless-start**

    ![iamrole](/images/iamrole-serverless.png)

Click on the Role **CloudFormationRole-serverless-start-...**. Choose **Add Permissions** -> **Create Inline Policy** and select **JSON** tab when it is opened.

![iamrole-inline-policy](/images/iamrole-inline-policy.png)

Paste in the following JSON object into the input field.

```
    {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                    "iam:GetRole",
                    "iam:PassRole",
                    "iam:DetachRolePolicy",
                    "iam:DeleteRolePolicy",
                    "iam:CreateRole",
                    "iam:DeleteRole",
                    "iam:AttachRolePolicy",
                    "iam:PutRolePolicy",
                    "iam:GetRolePolicy"
            ],
            "Resource": "*"
        }
    ]
    }
```

Name **Policy** as **CloudFormation-serverless-start-create-role** and choose **Create Policy** 

### Step 2. Setting up the Code Repo and IDE
1. Navigate to the github repository that is created in your github account. The repository is a SAM application. Please refer to the [SAM documentation](https://docs.aws.amazon.com/serverless-application-model/) for more information.
    Some key files to note here are:
    |Folder/File name|Purpose|
    |--------|-------|
    |template.yml| An AWS SAM template file closely follows the format of an AWS CloudFormation template file. The template file specifies the resources and the events thats will be created by the SAM CLI|
    |buildspec.yml| The buildspec.yml specifies the phases within the CodeBuild project |
    |src/handlers| This folder specifies the code for the lamda handlers|
    |event| The folder specifies the API endpoints that will be created by the SAM|

2. Navigate to the serverless [application](https://us-east-1.console.aws.amazon.com/lambda/home?region=us-east-1#/applications) created and access the ``serverless-start`` application. Copy the API Endpoint for the serverless application.

3. Now we will navigate to [API Gateway](https://us-east-1.console.aws.amazon.com/apigateway/main/apis?region=us-east-1), select **serverless-start API** and choose **Test** to create a few records using the API Endpoint:

    ![API GW](/images/APIGWTest.png)

    Put the followings in the **Request Body** and **Test**:
    ```
    {
        "id":"1",
        "name":"We"
    }
    ```

    ![Test API](/images/TestResult.png)
    
    Feel free to post additional rows.

    These operations will create Items in the DynamoDB ``serverless-start-SampleTable-xxxxxxx`` table.
    
    ![dynamoDBItems](/images/dynamoDBItems.png)

4. We will now get the list of items. Copy the API Endpoint in your browser console and hit enter. e.g. ``https://xxxxxxxxx.execute-api.ap-southeast-2.amazonaws.com/Prod/``. The list of items in the ``serverless-start-SampleTable-xxxxxxx`` table will be displayed as below:

    ![listallItems](/images/listallItems.png)

5. Next lets try to fetch an item that does not exist in the table. In the below screenshot we are fetching item with an id = 4. Currently when an item is not found in the backend a blank page is displayed.
    
    ![emptystringwhenpickingid](/images/emptystringwhenpickingid.png)

6. Next we will use an IDE to update the code to return an error when items are not found in the dynamodb backend. For this lab we will use the Cloud9 IDE as it comes pre-installed with AWS SAM. Navigate to the [Cloud9 service](https://us-east-1.console.aws.amazon.com/cloud9/home?region=us-east-1#) and create a new environment with the below settings.
    
    ![createCloudEnv](/images/createCloudEnv.png)

7. Clone your private github repository for the ``serverless-start`` application. 
**Note:** GitHub now uses a personal access token to access repositories as username and password usage has been discontinued. Please see [link](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) to create a personal access token.
    
    ![cloneserverless-start-repo](/images/cloneserverless-start-repo.png)

8. Open ``get-by-id.js`` under ``src/handlers`` and replace

````
    const { Item } = await docClient.get(params).promise();

    const response = {
        statusCode: 200,
        body: JSON.stringify(Item),
    };
````
with 
```    
    const { Item } = await docClient.get(params).promise();
    let bodyString
    if (Item) bodyString = JSON.stringify(Item)
    else bodyString = JSON.stringify({"error": "Please check Item entered. Item not found in the catalog" })

    const response = {
        statusCode: 200,
        body: bodyString,
    };
```
This results in code looking as below:

![cloud9updateCode](/images/cloud9updateCode.png)

### Step 3. Safe Deployments

1. We will now perform a safe/gradual deployment. Using SAM you can take advantage of gradual deployments. 
    Safe depoloyments are broadly classified as:
    - Canary Deployments
    - Linear Deployments

    We define this within the ``template.yaml`` file of SAM application, by defining 2 properties:
    - AutoPublishAlias
    - Deployment Preference Type

    The various values for the Deployment Preference Type are:
    |Deployment Preference Type|
    |--|
    |Canary10Percent30Minutes|
    |Canary10Percent5Minutes|
    |Canary10Percent10Minutes|
    |Canary10Percent15Minutes|
    |Linear10PercentEvery10Minutes|
    |Linear10PercentEvery1Minute|
    |Linear10PercentEvery2Minutes|
    |Linear10PercentEvery3Minutes|
    |AllAtOnce|

    You can read more about 'safe deployments' [here](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/automating-updates-to-serverless-apps.html).

    In our lab we will be using ``Canary10Percent5Minutes`` deployment preference.
    This instructs AWS CodeDeploy to start with 10% traffic on new version and after that shift the remaining traffic. 
    Changes to the ``template.yaml`` will look as below:
    ![definingcanary](/images/definingcanary.png)

2. Once the code changes are made, these can pushed to the GitHub repository via Terminal window of Cloud9 IDE.
    ````
        cd ~/environment/serverless-start 
        git add *
        git commit -m "Return an error when Item not found"
        git push
    ````
    Remember to use the GitHub [Personal Access Token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) to push the code.
3. This will kick off the code pipeline as progress through the various steps that include:
    - Code Checkout
    - Code Build
    - Code Deploy
    Navigate to [AWS Lambda Applications](https://us-east-1.console.aws.amazon.com/lambda/home?region=us-east-1#/applications), select **serverless-start** and choose **View in CodePipeline** to see the progress

    ![codepipeline](/images/serverless-codepipeline.png)


    ![codepipelinekickoff](/images/codepipelinekickoff.png)


4. While the deployment progresses, you can navigate to the ``serverless-start`` cloudformation stack update.
    
    ![serverlessstartdeploycfn](/images/serverlessstartdeploycfn.png)

5. Now that the changes have been deployed, fetching an item with an ID that does not exist in the dynamodb should return a more verbose error as below:
    
    ![idnotfounderror](/images/idnotfounderror.png)

## Cleanup
1. Navigate to Lambda Applications and delete ``serverless-start`` application. This will prompt you with detailed instructions on deleting the application. Follow the instructions to delete all associated resources.
    ![deleteserverless](/images/deleteserverless.png)
2. Navigate to the Cloud9 service and delete the ``cicdworkflowIDE`` environment.

## Summary
In this lab you learnt:
- how to create a sample serverless application.
- how to implement a CI/CD workflow using Code pipeline.
- how to perform safe deployments using a deployment preference with SAM template.
