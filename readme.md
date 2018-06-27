# Serverless CI/CD with AWS

- [Overview](#overview)

# Overview
Serverless is fast becoming the new default compute model for many software projects. The attractive pay as you go pricing model, rapid scaling and reduced ops overhead are very compelling advantages to any organisation shipping software.

While applications are moving to this new compute model, we're often still left with some traditional infrastructure in the form of build servers and nodes. This remains a burden for ops due to having to perform OS patching, build server updates and managing fleets of build nodes as well as monitoring for performance and right-sizing both server and build nodes on an ongoing basis. This leaves also a potential single point of failure where build servers are shared amongst different services being deployed - downtime of the server/nodes can mean delays in shippinh. 

In this playground we're going to remove some of that burden by creating a serverless CI/CD pipeline on AWS using CodePipeline, CodeBuild and CloudFormation. This pipeline will be deploying a simple nodejs express application to lambda.

# Hands On

### Logging in to AWS
You can skip this section if you are using your own AWS account and have appropriate IAM access (detailed in addendum)

First we need to login to the console (credentials provided separately) - also ensure that we are in the eu-west-1 region.
https://ecsd-training.signin.aws.amazon.com/console

The user that we logged into won't have many permissions at this stage so we need to assume an appropriate role. Follow this link in order to do that

https://signin.aws.amazon.com/switchrole?roleName=pg23meetuprole&account=ecsd-training

You should now see an indicator in the top right showing that you are currently assuming a role.

### Creating the pipeline

Next we need to navigate to the CodePipeline user interface
![](images/codepipeline-ui.png)

Click the "Create Pipeline" button.

You now need to give your pipeline a name - this needs to be unique so try and follow convention e.g. firstnamelastinitial-pg23-pipeline (e.g. paulf-pg23-pipeline)

Click the "Next Step" button to continue

### Define the source

The first step in our pipeline is to grab our source code. In our case this is provided in a codecommit repository but other options are available for github and AWS S3. The branch we'll be deploying is the master branch.

Choose CodeCommit.

Pick the repository "pg23repo"

Pick the branch "master"

Leave the "Change detection options" at their default - for CodeCommit and s3 this will use CloudWatch events to detect changes in order to trigger the pipeline, github will use webhooks.

Click the "Next Step" button to continue

## Define the Build

### Deployment

For now select No Deployment - we'll come back and add this step later.

### Observe

Your pipeline should be triggered on creation but will likely fail at this point

### Fix the build!

The build will have failed as one of the steps requires permission to perform a PutObject request on s3. Let's fix this.

Navigate to the IAM user interface and select Roles on the left.
Find the role you created for your CodeBuild configuration.

Click the "Attach Policy" button.

Type "S3" in the search box.
For this demo let's just assign "AmazonS3FullAccess" - in a real environment you'd define an appropriate policy for your bucket for least privilege. Select the "AmazonS3FullAccess" role by clicking the checkbox to its left.

Click the "Attach Policy" button down the bottom right.

Observe that the Permissions tab now lists "AmazonS3FullAccess" in addition to existing policies.

### Re-run the pipeline

Go back to your pipeline and click the "Release" button. This time the pipeline should succeed and we are ready to define our deployment steps!

### Add a new stage

Deployment of a SAM template needs to consist of two steps as CloudFormation stack creation currently does not support template transforms. 

We instead need to generate a change set and then apply the change set to a new or existing CloudFormation stack.

This will require two pipeline actions.

First, lets add a new stage to the pipeline

In your pipeline, click the "Edit" button.

You will see the UI change to show additional controls which allow you to modify your pipeline's workflow.

### Add a generate changeset action

### Add an apply changeset action

### Re-run the pipeline

### Success!!

### Verify the deployment

### Commit a change

### Verify the deployment

### Conclusion

So now we have a pipeline which can continuously deploy changes from our master branch to our deployment environment. 

Other additions we can make to this pipeline include additional stages (such as uat), manual approval (requiring human intervention in order to continue) as well as other actions such as running tests. 


### One more thing

As a sidenote, CodePipeline and CodeBuild also generate events which can be consumed and fed into CloudWatch as metrics. This can be used to build dashboards showing the performance of your pipeline and development process. This can be a powerful tool in diagnosing issues with your SDLC by examining the various metrics with regards to feedback, failure and deployment rate.





