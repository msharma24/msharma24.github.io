---
layout: post
title: Terraform Backend for multi-account AWS Architecture
---
{% seo %}
TL;DR How to create  S3 Bucket and DynamoDB Table for Terraform backend in a multi-account AWS environment.


I'm working with a customer who has deployed  a multitude of AWS Accounts in their AWS Organisation and have arranged the AWS accounts in multiple Organisational Units (OU) . They have also leveraged the [AWS Control Tower](https://aws.amazon.com/controltower/?control-blogs.sort-by=item.additionalFields.createdDate&control-blogs.sort-order=desc) to easily set up the governance , security and the best practices across the AWS organisation.

They have also adopted Terraform as the preferred  [Infrastructure as Code (IAC)](https://en.wikipedia.org/wiki/Infrastructure_as_code) tool, and using a separate AWS S3 Bucket and a DynamoDb Table as the [Terraform backend ](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/what-is-cfnstacksets.html) per AWS account is one of the community best practices.

This created the need to having a fully automated solution in place to deploy an AWS S3 Bucket and a DynamoDB Table in each of the existing and future  AWS Accounts with a standard configuration  , so that the teams across the organisation can use the same AWS S3 Bucket and the DynamoDB Table for the Terraform State  management of their application stack environment(s).

In order to solve this problem, we decided to write a CloudFormation Template which will deploy a S3 Bucket and DynamoDB Table in every AWS Accounts using the [CloudFormation StackSets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/what-is-cfnstacksets.html) - which lets you provision resources across Multiple AWS Accounts and Regions. 	<img src="{{site.baseurl}}/images/blog01/TerraformRemoteState-with-CloudFormationStack.png">

## Solution Deployment
When you deploy the AWS Control Tower service across the AWS Organisation , it creates `AWSControlTowerExecution` IAM Role in every child account enrolled and uses the `AWSControlTowerStackSetRole` IAM Role in the Management AWS Account to deploy stack sets in all the AWS accounts created by AWS Control Tower.


<details>
  <summary>CloudFormation Template - Click to expand!</summary>
{% highlight yaml linenos %}
---
AWSTemplateFormatVersion: 2010-09-09
Description: >
  [Do Not Delete]
  CloudFormation Template to create  a S3 Bucket and a DynamoDB Table
  for Terraform tfstate file management.
Resources:
  TerraformStateBucket:
    Type: 'AWS::S3::Bucket'
    DeletionPolicy: Retain
    UpdateReplacePolicy: Retain
    Properties:
      AccessControl: Private
      BucketName: !Sub 'terraform-state-backend-${AWS::AccountId}'
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: AES256
      PublicAccessBlockConfiguration:
        BlockPublicAcls: True
        BlockPublicPolicy: True
        IgnorePublicAcls: True
        RestrictPublicBuckets: True
      VersioningConfiguration:
        Status: Enabled
      Tags:
        - Key: "Name"
          Value: Terraform State Backend
        - Key: "Managed  By"
          Value: CloudFormation Stack

  TerraformLockTable:
    Type: 'AWS::DynamoDB::Table'
    Properties:
      AttributeDefinitions:
        - AttributeName: LockID
          AttributeType: S
      KeySchema:
        - AttributeName: LockID
          KeyType: HASH
      ProvisionedThroughput:
        ReadCapacityUnits: 5
        WriteCapacityUnits: 5
      TableName: !Sub 'terraform-state-backend-lock-${AWS::AccountId}'
      Tags:
        - Key: "Name"
          Value: Terraform State Backend Lock
        - Key: "Managed By"
          Value: CloudFormation stack

Outputs:
  StackName:
    Description: CloudFormation Stack Name
    Value: !Ref AWS::StackName
  TerraformStateBucketName:
    Description: S3 Bucket for Terraform State Storage
    Value: !Ref TerraformStateBucket
  TerraformLockTable:
    Description: DynamoDB for Terraform Stack Lock
    Value: !Ref TerraformLockTable
{% endhighlight %}
</details>

## Steps to deploy the above CloudFormation Template
1. 	Login to the AWS Management accounts (*Root Account*) console and go to the AWS Organisation service page and make a copy of the of the Organisational Units  `id` in which you wish to create the AWS S3 Bucket and AWS DynamoDB Table using the CloudFormation Stackset. <img src="{{site.baseurl}}/images/blog01/aws-org.png">

2. Download the CloudFormation Template from this blog and save as `terraform-state-backend-CloudFormation.yaml` to your local computer.


3. Next, Go to the AWS CloudFormation Service page and enable the CloudFormation Stackset trusted access with the AWS Organisations by following this [link](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-orgs-enable-trusted-access.html) to AWS documentation.


4. Click on StackSets after expanding the menu on the left ,and then click the "create stackset" button and upload the `terraform-state-backend-CloudFormation.yaml` and then click `Next`	<img src="{{site.baseurl}}/images/blog01/cfn-upload.png">		 On the next screen, Enter the stackset name - example: `terraform-backend-stackset` and click `Next` again.
5. Choose the "Service-managed permissions" as default option and click `Next`


6. On the "Set deployment options" form , choose the "Deploy to Organizational Units (OU)" Radio option and enter the Organisational Unit ids in which you wish to deploy, make sure the *Account removal behavior* Radio option is set to *Retain* and then *Automatic deployment* is *Enabled* ,  also select the AWS Region from the "Specify region" drop-down menu. <img src="{{site.baseurl}}/images/blog01/dep-options.png">
7. On the Next screen, Review the configuration and Click the *Submit* Button to begin the deployment.


8. Once the deployment begins you will the stackset Operation in Running status and once completed a S3 Bucket and a DynamoDB table will created in every account of your organisational units and this will be a good starting point to start writing the Terraform stack configurations using the S3 and DynanmoDB as backend configuration, and if any more new accounts are created in those organisation units this stackset will be automatically deployed to them. <img src="{{site.baseurl}}/images/blog01/stack-set-op.png">

## Results
Once the CloudFormation Stackset completes, each AWS account in the targeted Organisational Unit (OU) will have a S3 bucket  and a DynamoDB  called,

* S3 Bucket:**`terraform-state-backend-<12 Digit AWS Account id>`**
* Dynamodb: **`terraform-state-backend-lock-<12 Digit AWS Account id>`**

and they can be  referenced in the Terraform `backend` configuration as follows:-
{% highlight php lineos %}
// backend.tf
terraform {
  required_version = ">= 1.0.0"

  backend "s3" {
    region         = "ap-southeast-2"
    bucket         = "terraform-state-backend-111110011111"
    key            = "test-project-key/dev/terraform.tfstate"
    dynamodb_table = "terraform-state-backend-lock-111110011111"
    encrypt        = "true"
  }
}
{% endhighlight %}



