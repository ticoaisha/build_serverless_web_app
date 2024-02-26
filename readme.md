-------------
# Building Serverless Web App

In this workshop I deployed a simple web application that enables users to request unicorn rides from the [Wild Rydes][wildrydes] fleet. The application will present users with an HTML based user interface for indicating the location where they would like to be picked up and will interface on the backend with a RESTful web service to submit the request and dispatch a nearby unicorn. The application will also provide facilities for users to register with the service and log in before requesting rides.

### Application Architecture
* AWS Lambda
* Amazon API Gateway
* Amazon S3
* Amazon DynamoDB
* Amazon Cognito
* AWS Amplify Console

#### Architecture explained
Amplify Console hosts static web resources including HTML, CSS, JavaScript, and image files which are loaded in the user’s browser via S3. JavaScript executed in the browser sends and receives data from a public backend API built using Lambda and API Gateway. Amazon Cognito provides user management and authentication functions to secure the backend API. Finally, DynamoDB provides a persistence layer where data can be stored by the API’s Lambda function.

![Applicaiton architecture](/images/app_arch.png)

## Setup
AWS Cloud9

1. Go to the AWS Management Console, Select Services then select Cloud9 under Developer Tools. From the top-right of the Console, select an available region for this workshop. Once you have selected a region for Cloud9, use the same region for the entirety of this workshop.

![AWS Cloud9 Create environment](/images/aws_cloud9_create_env.png)

2. Select Create environment.

3. Enter wildrydes-webapp-development into Name and optionally provide a Description.

![Create environment](/images/create_env.png)

4. Select Next step.

5. Review the environment settings and select Create environment. It will take a few minutes for your Cloud9 environment to be provisioned and prepared.

6. Once ready, your IDE will open to a welcome screen. Below that, you should see a terminal prompt. Close the Welcome tab and drag up the terminal window to give yourself more space to work in.

![Cloud9 Terminal Window](/images/Cloud9_Terminal_Window.png)

You can run AWS CLI commands in here just like you would on your local computer. Remember for this workshop to run all commands within the Cloud9 terminal window rather than on your local computer.

Keep your AWS Cloud9 IDE opened in a browser tab throughout this workshop.

Verify that your user is logged in by running the command aws sts get-caller-identity. Copy and paste the command into the Cloud9 terminal window.

*Note. Keep an open scratch pad in Cloud9 or a text editor on your local computer for notes. When the step-by-step directions tell you to note something such as an ID or Amazon Resource Name (ARN), copy and paste that into your scratch pad.*

## Static Web Hosting
This section explains the frontend structure. You’ll deploy the static website using AWS Amplify Console by first creating a git repository (in either CodeCommit or GitHub) and then pushing the site code.

##### Overview
In this module you’ll configure AWS Amplify Console to host the static resources for your web application. In subsequent modules you’ll add dynamic functionality to these pages using JavaScript to call remote RESTful APIs built with AWS Lambda and Amazon API Gateway.

###### Architecture Overview
The architecture for this module is very straightforward. All of your static web content including HTML, CSS, JavaScript, images and other files will be managed by AWS Amplify Console and served via Amazon CloudFront. Your end users will then access your site using the public website URL exposed by AWS Amplify Console. You don’t need to run any web servers or use other services in order to make your site available.

![Staic Web Hosting Architecture Overview](/images/static_web_hosting_arch_overview.png)

Region

Region Selection
This workshop step can be deployed in any AWS region that supports the following services:

AWS Cognito
AWS Amplify Console
AWS CodeCommit

You can refer to the [AWS region table](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/) in the AWS documentation to see which regions have the supported services.

Once you’ve chosen a region, you should deploy all of the resources for this workshop there. Make sure you select your region from the drop down in the upper right corner of the AWS Console before getting started.

Repository

Create the git repository
We will use AWS CodeCommit to host your site’s repository. CodeCommit is included in the AWS Free Tier.

CodeCommit helper
The AWS Cloud9 development environment comes with AWS managed temporary credentials that are associated with your IAM user. You use these credentials with the AWS git-remote-codecommit tool (A Git Remote Helper that makes it easier to interact with AWS CodeCommit). This tool is installed in Cloud9 by default. You can install it on your own machine by following the installation instructions.

Setting up your CodeCommit repository
1. First, create a new CodeCommit repository from within your Cloud9 terminal window:

```
aws codecommit create-repository \
  --repository-name wild-rydes


```

![Create a new CodeCommit repository within Cloud9](/images/create_codecommit_repo-cloud9.png)

2. Clone the existing (not new) workshop repository from GitHub:

```
git clone https://github.com/aws-samples/aws-serverless-webapp-workshop.git

```
![Clone existing workshop repo directory](/images/clone_exist_workshop.png)

3. Change into the workshop repository directory:

```
cd aws-serverless-webapp-workshop
```
4. Split out the WildRydesVue code into its own branch:

```
sudo yum install git-subtree -y
git subtree split -P resources/code/WildRydesVue -b WildRydesVue
```
![Split out WildRydesVue code into its own branch](/images/split_code_into_branch.png)

5. Create a new directory for your CodeCommit repo:
```
mkdir ../wild-rydes && cd ../wild-rydes
```
6. Initialize a new git repository:
```
git init
```
![Initialize a new git repository](/images/git_init.png)

7. Pull the WildRydesVue branch into your new repo:
```
git pull ../aws-serverless-webapp-workshop WildRydesVue

```
![Pull the WildRydesVue branch into your new repo](/images/git_pull.png)

8. Add your CodeCommit repository as a remote:
```
git remote add origin codecommit://wild-rydes
```

![Add your CodeCommit repository as a remote]()

9. Push the code to your new CodeCommit repository:
```
git push -u origin master
```
![Add repo as a remote, push the code to new repo](/images/add_push.png)

10. Remove the temporary local repository you created in step 2: rm -rf ../aws-serverless-webapp-workshop

Deploy

Deploy the site with AWS Amplify Console
Next you’ll use the AWS Amplify Console to deploy the website you’ve just commited to git. The Amplify Console takes care of the work of setting up a place to store your static web application code and provides a number of helpful capabilities to simplify both the lifecycle of that application as well as enable best practices.

1. Launch the Amplify Console
2. Underneath Get Started, you’ll find a section for Amplify Hosting titled Host your web app. Click the Get started button within that section. If you are starting from the All apps page, choose New app, then Host web app in the upper right corner.

![Amplify get started](/images/amplify_get_started.png)

3. Connect a repository: Select AWS CodeCommit and choose Continue

![Connect a repo](/images/connect_repo.png)
4. From the drop down select the Repository and master Branch created today and select Next

![Add repo branch](/images/add_repo_branch.png)

5. Amplify will detect that the application has an existing Amplify backend. Select Create New Environment and name it prod

Now you need to create a new service role with the permissions to deploy the application backend.

1. Click on Create new role, check that Amplify is selected and click Next permissions, click Next: Tags, click Next: Review.

2. Give the Role a new name: wildrydes-backend-role and click Create role.

![Create a role](/images/create_role.png)

Click Attach policies under the ** Add Permissions** tab, search for AWSCodeCommitReadOnly policy, click on the checkbox next to the policy name, and click **Add premissions**.

![Attach policies](/images/attach_policies.png)

![AWSCodeCommitReadOnly policy](/images/AWSCodeCommitReadOnly_policy.png)

3. Close this tab, go back to AWS Amligy Build settings, click Refrsh existing roles, search for wildrydes-backend-role from the search filter, and click the role name.

4. Refresh the role list by clicking on the circular arrow button, and select the role created in the step above.

![Choose a role](/images/choose_role.png)

5. Select Next

On the Review page select Save and deploy

![Review repo details](/images/review_repo.png)

This initial build and deploy process may take up to five minutes for Amplify Console to create the neccesary resources and to deploy your code.

![Provision](/images/provision.png)

![Provision, build and deploy completed](/images/completed.png)

Once completed, click the site URL to launch your Wild Rydes site.

![Website opened](/images/website_opened.png)

![Just website](/images/just_website.png)

Modify

Modify the website
The AWS Amplify Console will rebuild and redeploy the app when it detects changes to the connected repository. Make a change to the main page to test out this process.

1. From your Cloud9 environment open the index.html file in the /wild-rydes/public/ directory of the repository.

![Open index.html](/images/open_index_html.png)

2. Modify the title line so that is says:

```
  <title>Wild Rydes - Rydes of the Future!</title>

```
![Modify index.html title](/images/modify_index_title.png)

Save the file

3. Commit again to your git repository the changes:
```
git add . 

git commit -m "updated title"
   
git push
```

![Commit the changes](/images/commit_changes.png)

Amplify Console will begin to build the site again soon after it notices the update to the repository. This happens pretty quickly! Head back to the Amplify Console to watch the process.

![Amplify will begin to build site again](/images/Amlify_build_site_again.png)

4. Once completed, re-open the Wild Rydes site and notice the title change.

![Title changes](/images/title_changed.png)

Recap
AWS Amplify Console makes it easy to deploy static websites with continuous integration and continuous delivery. It is capable of building more complicated JavaScript framework-based applications and shows you a preview of your application as it renders on popular mobile platforms.

In this module, you’ve created a static website which will be the base for our Wild Rydes business.

## User Management

This section explains how to create users. You’ll configure user management for the website using Amazon Cognito.

OVERVIEW
In this module you’ll use the AWS Amplify CLI to create an Amazon Cognito User Pool to manage your users' accounts. You’ll deploy pages that enable customers to register as a new user, verify their email address, and sign into the site.

Architecture Overview
When users visit your website they will first register a new user account. For the purposes of this workshop we’ll only require them to provide an email address and password to register. However, you can configure Amazon Cognito to require additional attributes in your own applications.

After users submit their registration, Amazon Cognito will send a confirmation email with a verification code to the address they provided. To confirm their account, users will return to your site and enter their email address and the verification code they received. You can also confirm user accounts using the Amazon Cognito console if you want to use fake email addresses for testing.

After users have a confirmed account (either using the email verification process or a manual confirmation through the console), they will be able to sign in. When users sign in, they enter their username (or email) and password. A JavaScript function then communicates with Amazon Cognito, authenticates using the Secure Remote Password protocol (SRP), and receives back a set of JSON Web Tokens (JWT). The JWTs contain claims about the identity of the user and will be used in the next module to authenticate against the RESTful API you build with Amazon API Gateway.

![User Mgmt Architecture Overview](/images/user_mgmt_arch_overview.png)

AMPLIFY CLI
Initialize AWS Amplify CLI
Background
The AWS Amplify Command Line Interface (CLI) is a unified toolchain to create, integrate, and manage the AWS cloud services for your app. The Amplify CLI toolchain is designed to work with the Amplify JavaScript library as well as the AWS Mobile SDKs for iOS and Android.

AWS Amplify Authentication module provides Authentication APIs and building blocks for developers who want to create user authentication experiences.

1. Install the Amplify CLI by running the following command from within your Cloud9 terminal window:
```
npm install -g @aws-amplify/cli
```
![Install Amplify CLI](/images/install_amplify_cli.png)

More information can be found in the documention here

2. Configure your default AWS profile.
```
echo '[profile default]' > ~/.aws/config
```


3. Make sure you are in the root wild-rydes directory of the repository:

4. Initialize amplify CLI by executing the following command:
```
amplify init
```

The terminal will now take a few moments to initialize your project:

*Note. Be sure to provide the same environment name that you provided earlier, for example, prod.*

Here is the output in the terminal:

![Initializing a project](/images/init_project.png)

Verify that the initialization has finished by entering the following command. Version 4.29.3 or greater should be installed.
```
amplify version

```
![Amplify version](/images/amplify_v.png)

Next you will add an Amazon Cognito category to your AWS Amplify configuration, via the AWS Amplify CLI.

USER POOL
Create an Amazon Cognito User Pool using AWS Amplify CLI
Background
The AWS Amplify Authentication module provides Authentication APIs and building blocks for developers who want to create user authentication experiences.

Amazon Cognito User Pools is a full-featured user directory service to handle user registration, authentication, and account recovery. Amazon Cognito Federated Identities on the other hand, is a way to authorize your users to use AWS services.

Amplify interfaces with User Pools to store your user information. This includes federation with other OpenID providers like Facebook & Google. Amplify also uses Federated Identities to manage user access to AWS Resources, like allowing a user to upload a file to an S3 bucket. The Amplify CLI automates the access control policies for these AWS resources and provides fine grained access controls via GraphQL for protecting data in your APIs.

In this section you use the Amplify CLI to create a new Cognito User Pool with the default settings. Then you use the Amazon Cognito Console to manage the new User Pool.

Amazon Cognito
Execute the following commands to add the Amazon Cognito User Pool from within the Cloud9 terminal window:

```
amplify add auth
```
The AWS Amplify CLI will now run through the set up for Amazon Cognito, select the following:

![Setting up Amazon Cognito](/images/setting_up_amazon_cognito.png)

Once configuration completes you see the following confirmation:

![Configuration completed](/images/config_completed.png)

Committing your code updates to provision your Amplify backend resources in the cloud and kick off a new build.

Commit the changes to your git repository:

```
git add .

git commit -m "Configure Cognito"

git push

```
Amplify Console picks up the changes and begins building and deploying your web application. User Pool will get created in Amazon Cognito console once the application is deployed successfully.

![Commit the changes](/images/commit_new_changes.png)

APP CLIENT
Check Your User Pool’s App Client
A Cognito Userpool and a new App client has been created by the AWS Amplify build. Let’s take a look at this app client.

1. Go to the Amazon Cognito Console

2. Choose User Pools.

Here you will see a new userpool generated by the AWS Amplify CLI that looks something similar to the example below:

![Choose User pools](/images/amazon_cognito_user_pool.png)

![New user pool](/images/new_user_pool.png)

3. Click the new user pool to open the Pool Details page

4. Click "App integration" tab:

![App intagration tab](/images/app_integ.png)

5. You will see that a new App client has been generated. Your web application is configured to use this App client via a config file located in wild-rydes/src/aws-exports.js.

![Apps](/images/apps.png)

How it Works:
Rather than configuring each service through a constructor or constants file, Amplify supports configuration through a centralized file called aws-exports.js which defines all the regions and service endpoints to communicate. Whenever you run amplify push or rebuild your web application by running a git commit, this file is automatically created, allowing you to focus on your application code. The Amplify CLI places this file in the appropriate source directory configured with amplify init.

❗ You won’t see updates to this file in your local file store because it is included in the .gitignore file.


CREATE A USER
Create a new user for your user pool
Note: Instead of having you write the browser-side code for managing the registration, verification, and sign in flows, we provide a working implementation in the assets you deployed in the first module by using the AWS Amplify Authentication UI component.

The Authenticator component provides basic login/logout functionality for an application, as well as confirmation steps for new user registration and user login.

Usage: <amplify-authenticator></amplify-authenticator>
Implementation Validation

1. Visit /auth under your website domain, or choose the Giddy Up! button on the homepage of your site.

![Giidy up!](/images/giddy_up.png)

2. Click on the Create Account link at the bottom of the sign in box.

![Create acct](/images/create_acct.png)

3. Complete the registration form and choose Create Account. You can use your own email or enter a fake email. Make sure to choose a password that contains at least one upper-case letter, a number, and a special character. Don’t forget the password you entered for later. You should see an alert that confirms that your user has been created.

⚠️ If you get an Authentication Error this is likely because your changes have not finished deploying.

4. Confirm your new user using one of the two following methods.

If you used an email address you control, you can complete the account verification process by entering the verification code that is emailed to you. Please note, the verification email may end up in your spam folder. For real deployments we recommend configuring your user pool to use Amazon Simple Email Service to send emails from a domain you own.

If you used a dummy email address, you must confirm the user manually through the Cognito console.

From the AWS console, click Services then select Cognito under Security, Identity & Compliance.
Choose Manage your User Pools
Select the user pool prefixed with wildrydes and click Users and groups in the left navigation bar.
You should see a user corresponding to the email address that you submitted through the registration page. Choose that username to view the user detail page.
Choose Confirm user to finalize the account creation process.

5. After confirming the new user using either the verrification code or the Cognito console, click on the back to sign in link or refresh the /auth page and log in using the email address and password you entered during the registration step.

6. If successful you should be redirected to /ride. 

![Redirected to /ride](/images/redirect_ride.png)

You should see a notification that the API is not configured.

![API is not configured](/images/api_not_config.png)

Recap
Amazon Cognito provides two different capabilities for managing users, federated identities and user pools. Amazon Cognito user pools handle almost any aspect about managing users, including login credentials, password resets, multifactor authentication and much more!

In this module you used user pools to create a fully-managed user management system that allows you to authenticate your users and manage their user information. You updated your website to use the user pool, and used the AWS SDKs to provide a signin form on the site.

Next
After you have successfully logged into your web application, you can proceed to the next module, Serverless Backend.

Extra
Try copying the auth_token you’ve received and paste that into an online JWT Decoder to understand what this token means for your application

![Decoder of auth token](/images/token_decode.png)

## Serverless Backend
This section explains how to create an AWS Lambda function that will persist data to an Amazon DynamoDB table.

OVERVIEW
In this module you’ll use AWS Lambda and Amazon DynamoDB to build a backend process for handling requests from your web application. The browser application that you deployed in the first module allows users to request that a unicorn be sent to a location of their choice. In order to fulfill those requests, the JavaScript running in the browser invokes a service running in the cloud.

You’ll implement a Lambda function that will be invoked each time a user requests a unicorn. The function selects a unicorn from the fleet, records the request in a DynamoDB table, and responds to the front-end application with details about the dispatched unicorn.

![Serverless Backend Architecture Overview](/images/serverless_be_arch.png)

The function is invoked from the browser using Amazon API Gateway. You implement that connection in the next module. For this module, you test your function in isolation.

DYNAMODB
Create an Amazon DynamoDB Table
Use the Amazon DynamoDB Console to create a new DynamoDB table. Call your table Rides and give it a partition key called RideId with type String. The table name and partition key are case sensitive. Make sure you use the exact IDs provided. Use the defaults for all other settings.

After you’ve created the table, record the ARN for use in the next step.

Go to the Amazon DynamoDB Console
Choose Create table.

![Create a table](/images/create_table.png)

Enter Rides for the Table name. This field is case sensitive.
Enter RideId for the Partition key and select String for the key type. This field is case sensitive.
Choose the Default settings button for Table settings and choose Create table.

![Creatong table](/images/creating_tbl.png)

![Creation in process](/images/tbl_progress.png)

![Table created](/images/tbl_created.png)

Once the table is Active, Click on “Rides” to open the table settings and under Overview > General information > Additional info section, you will find the ARN of the table. Record this ARN as you will use this in the next section.

![Table ARN](/images/tbl_arn.png)

IAM
Create an IAM Role for Your Lambda Function
Background
Every Lambda function has an IAM role associated with it. This role defines what other AWS services the function is allowed to interact with. In this workshop, you create an IAM role that grants your Lambda function permission to write logs to Amazon CloudWatch Logs and access to write items to your DynamoDB table.

High-Level Instructions
Use the IAM console to create a new role. Name it WildRydesLambda and select AWS Lambda for the role type. You’ll need to attach policies that grant your function permissions to write to Amazon CloudWatch Logs and put items to your DynamoDB table.

Attach the managed policy called AWSLambdaBasicExecutionRole to this role to grant the necessary CloudWatch Logs permissions. Also, create a custom inline policy for your role that allows the dynamodb:PutItem action for the table you created in the previous section.

1. Go to the AWS IAM Console
2. Select Roles in the left navigation bar and then choose Create role.
3. Under Use case, Select Lambda from the AWS services, then click Next

![IAM role Lambda](/images/iam_role_lambda.png)

4. Begin typing AWSLambdaBasicExecutionRole in the Filter text box and check the box next to that managed role, then Click Next

![AWS Lambda Basic Exec Role](/images/lambda_basic_exec_role_lambda.png)

5. Enter WildRydesLambda for the Role name. Add any tags that you wish.

![Name role](/images/name_role.png)

6. Choose Create role.

![Role has been created](/images/lambda_role_created.png)

Next you need to add permissions to the role so that it can access your DynamoDB table.

1. Open WildRydesLambda role, under the Add Permissions tab, choose the create inline policy

![Create inline policy](/images/create_inline_policy.png)

2. Select Choose a service.
3. Begin typing DynamoDB into the search box labeled Find a service and select DynamoDB when it appears.

Choose Actions allowed.
Begin typing PutItem into the search box labeled Filter actions and check the box next to PutItem when it appears.

![DynamoDB permiss PutItem](/images/DynamoDb_put_item.png)


Select the Resources section.

With the Specific option selected, choose the Add ARN link in the table section.
Paste the ARN of the table you created in the previous section in the Specify ARN for table field, and choose Add.

![Specify ARN](/images/arn_tbl.png)

Click Next
Under the "Review and create" tab enter DynamoDBWriteAccess for the policy name and choose Create policy:

![Create DynamoDb policy](/images/create_dynamodb_policy.png)

LAMBDA
Create a Lambda Function for Handling Requests
Background
AWS Lambda runs your code in response to events such as an HTTP request. In this step you build a function that processes API requests from the web application to dispatch a unicorn. In the next module you use Amazon API Gateway to create a RESTful API that exposes an HTTP endpoint that can be invoked from your users' browsers. Then you connect the Lambda function you create in this step to that API to create a fully functional backend for your web application.

High-Level Instructions
Use the AWS Lambda console to create a new Lambda function called RequestUnicorn that processes API requests. Copy and paste this example implementation into the AWS Lambda console’s editor for your function code.

Configure your function to use the WildRydesLambda IAM role you created in the previous section.

1. Go to the AWS Lambda console
Click Create function.

![Create lambda function](/images/create_lambda_function.png)

Keep the default Author from scratch card selected.
Enter RequestUnicorn in the Name field.
Select Node.js 18.x for the Runtime.
Expand Change default execution role under Permissions.
Ensure Use an existing role is selected from the Role dropdown.
Select WildRydesLambda from the Existing Role dropdown.

![Create Lambda function details](/images/create_lambda_f_details.png)

Choose Create function.
Scroll down to the Function code section and replace the existing code in the index.js code editor with the contents of requestUnicorn.js.

![Exicting code](/images/existing_code.png)

![Replaced code](/images/replaced_code.png)

Click Deploy in the upper right above the code editor.

Implementation Validation
For this module you will test the function that you built using the AWS Lambda console. In the next module you will add a REST API with API Gateway so you can invoke your function from the browser-based application that you deployed in the first module.

From Test Tab, Configure test event.
Keep Create new test event selected.
Enter TestRequestEvent in the Event name field
Copy and paste the following test event into the editor

```
{
    "path": "/ride",
    "httpMethod": "POST",
    "headers": {
        "Accept": "*/*",
        "Authorization": "eyJraWQiOiJLTzRVMWZs",
        "content-type": "application/json; charset=UTF-8"
    },
    "queryStringParameters": null,
    "pathParameters": null,
    "requestContext": {
        "authorizer": {
            "claims": {
                "cognito:username": "the_username"
            }
        }
    },
    "body": "{\"PickupLocation\":{\"Latitude\":47.6174755835663,\"Longitude\":-122.28837066650185}}"
}
```
![Test event](/images/test_event.png)

Click Save. Click Test.

![Test succeeded](/images/test_success.png)

From Test Tab, expand the Details section of the Execution result section.
Verify that the execution succeeded and that the function result looks like the following:

![Test details](/images/test_details.png)

Recap
AWS Lambda is a serverless Functions-as-a-Service (FaaS) product that removes the burden of managing servers to run your applications. You configure a trigger and set the role that the function can use and then can interface with almost anything you want from databases, to datastores, to other services either publicly on the internet or in your own Amazon Virtual Private Cloud (VPC). Amazon DynamoDB is a non-relational serverless database that can scale automatically to handle massive amounts of traffic and data without the need to manage any servers.

In this module you created a DynamoDB table and a Lambda function to write data into it. In the next module, you create an Amazon API Gateway REST API and connect it to your application to capture ride details from your users.

Next
After testing your new function using the Lambda console, you can move on to the next module, RESTful APIs.

## RESTful APIs
This section explains how to expose the Lambda function via an Amazon API Gateway as a RESTful API that the static website can call.

OVERVIEW
In this module you use API Gateway to expose the Lambda function you built in the previous module as a RESTful API. This API will be accessible on the public Internet. It will be secured using the Amazon Cognito user pool you created in the User Management module. Using this configuration you will then turn your statically hosted website into a dynamic web application by adding client-side JavaScript that makes AJAX calls to the exposed APIs.

![RESTfulAPIs Architecture Overview](/images/RESTful_arct.png)

The preceding diagram shows how API Gateway integrates with the existing components you built previously.

The static website you deployed in the first module already has a page configured to interact with the API you’ll build in this module. The ride route has a simple map-based interface for requesting a unicorn ride. After authenticating using the /signin route, your users select their pickup location by clicking a point on the map and request a ride by choosing the Request Unicorn button in the upper right corner.

This module focuses on the steps required to build the cloud components of the API, but if you’re interested in how the browser code works that calls this API, you can inspect the ride.js source. In this case the application uses jQuery’s ajax() method to make the remote request.

REST API
Create a New REST API
Use the Amazon API Gateway console to create a new API named WildRydes.

Go to the Amazon API Gateway Console, click Create API

![API Gateway Console](/images/api_gwy_console.png)

On the REST API card, choose Build.

In the section Create new API select New API to clear the example API definition.

Enter WildRydes for the API Name.

Select Regional from the Endpoint Type dropdown.


*Note. Edge optimized APIs are best for public services being accessed from the Internet. Regional endpoints are typically used for APIs that are accessed primarily from within the same AWS Region. Private APIs are for internal services inside of an Amazon VPC.*

Choose Create API

![Create API](/images/create_api.png)

![API Created](/images/api_created.png)

COGNITO
Create a Cognito User Pools Authorizer
Background
Amazon API Gateway can use the JWT tokens returned by Cognito User Pools to authenticate API calls. In this step you’ll configure an authorizer for your API to use the user pool you created in User Management.

High-Level Instructions
In the Amazon API Gateway console, create a new Cognito user pool authorizer for your API. Configure it with the details of the user pool that you created in the previous module. You can test the configuration in the console by copying and pasting the auth token presented to you after you log in via the /signin route of your current website.

Under your newly created API, choose Authorizers.

Choose Create New Authorizer.

![Create New Authorizer](/images/create_new_auth.png)

Enter WildRydes for the Authorizer name.

Select Cognito for the type.

In the Region drop-down under Cognito User Pool, select the Region where you created your Cognito user pool in the User Management module (by default the current region should be selected).

Enter wildrydes in the Cognito User Pool input, the name will auto-complete and allow you to select the name of the user pool that was generated when the user pool was created.

Enter Authorization for the Token Source.

Choose Create.

![Creating new authorizer](/images/create_new_auth.png)

![Authorixer created](/images/auth_created.png)

Verify your authorizer configuration
Open a new browser tab and visit /ride under your website’s domain.

If you are redirected to the sign-in page, sign in with the user you created in the last module. You will be redirected back to /ride.

Copy the auth token from the notification on the /ride,

Go back to previous tab where you have just finished creating the Authorizer

Click Authorizer, paste the auth token into the Authorization Token value field in the popup dialog.

![Auth token value](/images/auth_token_val.png)

Click Test Authorizer button and verify that the response code is 200 and that you see the claims for your user displayed.

![Test success](/images/code_201.png)

API RESOURCE
Create a new resource and method
Create a new resource called /ride within your API. Then create a POST method for that resource and configure it to use a Lambda proxy integration backed by the RequestUnicorn function you created in the first step of this module.

In the left nav, click on Resources under your WildRydes API.

![Resources](/images/res.png)

From the Actions dropdown select Create Resource.

Enter ride as the Resource Name.

Ensure the Resource Path is set to ride.

Select Enable API Gateway CORS for the resource.

Choose Create Resource.

![Create resource](/images/create_res.png)

![Resource created](/images/res_created.png)

With the newly created /ride resource selected, from the Action dropdown select Create Method.

Select POST from the new dropdown that appears, then click the checkmark.

Select Lambda Function for the integration type.

Check the box for Use Lambda Proxy integration.

Select the Region you are using for Lambda Region.

![Creating method](/images/creating_method.png)

Enter the name of the function you created in the previous module, RequestUnicorn, for Lambda Function.

Choose Save. Please note, if you get an error that you function does not exist, check that the region you selected matches the one you used in the previous module.

Select the Method Request card, click Edit, select the WildRydes Cognito user pool authorizer from the drop-down list, and click the checkmark icon.

![Method request settings](/images/method_req_setttings.png)

![After updates](/images/meth_req_upd.png)

DEPLOY
Deploy Your API
From the Amazon API Gateway console, choose Actions, Deploy API. You’ll be prompted to create a new stage. You can use prod for the stage name.

In the Actions drop-down list select Deploy API.

![Deploy API](/images/deploy_api.png)

Select [New Stage] in the Deployment stage drop-down list.
Enter prod for the Stage Name.
Choose Deploy.

![Choose deploy API](/images/choose_deploy_api.png)

Record the Invoke URL. You will use it in the next section.

![Stage details](/images/stage_details.png)

UPDATE CONFIG
Update the Website Config
Update the /src/config.js file in your website deployment to include the invoke URL of the stage you just created. You should copy the invoke URL directly from the top of the stage editor page on the Amazon API Gateway console and paste it into the _config.api.invokeUrl key of your site’s /src/config.js file. Make sure when you update the config file it still contains the updates you made in the previous module for your Cognito user pool.

On your Cloud9 development environment open src/config.js
Update the invokeUrl setting under the api key in the config.js file. Set the value to the Invoke URL for the deployment stage your created in the previous section. An example of a complete config.js file is included below. Note: The actual URL in your file will be different.

![Invoke url to config file](/images/invoke_url_to_config.png)


Save the modified file making sure the filename is still config.js.

Commit the changes to your git repository:

```
git add src/config.js 


```
![Updating per invoke url](/images/update_invoke_url.png)

Amplify Console should pick up the changes and begin building and deploying your web application. Watch it to verify the completion of the deployment.

![Amplify updating](/images/aplify_upd_1.png)

![Amplify updating](/images/ampl_upd_2.png)

![Amplify updating](/images/ampl_upd_3.png)

Implementation Validation
Visit /ride under your website domain.
If you are redirected to the sign in page, sign in with the user you created in the previous module.
After the map has loaded, click anywhere on the map to set a pickup location.

![Chose pick up location](/images/choose_pick_up.png)

Choose Request Unicorn. You should see a notification in the right sidebar that a unicorn is on its way and then see a unicorn icon fly to your pickup location.

![Unicorn arrived]()

![Screen that unicorn arrived]()

Recap
Amazon API Gateway is a fully managed service that makes it easy for developers to create, publish, maintain, monitor, and secure APIs at any scale. You can easily plug in Authorization via Amazon Cognito and backends such as AWS Lambda to create completely serverless APIs.

In this module you’ve used API Gateway to provide a REST API to the Lambda function created in the previous module. From there you’ve updated the website to use the API endpoint so that you can request rides and the information about the ride is saved in the DynamoDB table created earlier.

Congratulations, you have completed the Wild Rydes Web Application Workshop! Check out our other workshops covering additional serverless use cases.

Next
See this workshop’s cleanup guide for instructions on how to delete the resources you’ve created.


## Cleanup
Once you have finished with the workshop, delete the associated resources to prevent incurring ongoing charges in your AWS account.

OVERVIEW
This page provides instructions for cleaning up the resources created during the preceding modules.

REST API CLEANUP
REST API Cleanup
Delete the REST API created in RESTful APIs step. There is a Delete API option in the Actions drop-down when you select your API in the Amazon API Gateway Console.

Go to the Amazon API Gateway Console
Select the API you created in RESTful APIs step.
Expand the Actions drop-down and choose Delete API.
Enter the name of your API when prompted and choose Delete API.
























