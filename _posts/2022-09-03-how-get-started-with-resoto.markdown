---
layout: post
title: How to get started with Resoto on AWS
---
<img src="{{site.baseurl}}/images/resoto/resoto_cli.png">
# Introduction
[Resoto](https://resoto.com/about) is an [Open Source](https://github.com/someengineering/resoto)  CLI tool by [someengineering](https://some.engineering/)  that puts  your entire cloud infrastructures inventory at your fingertips and also eases mundane cloud janitor tasks such as finding and deleting unused cloud resources across multiple AWS accounts (*which could be costing the business an arm and a leg*).
To Top it off, Resoto also  enables you to document the  cloud inventory for audit purposes in a pretty format.	

# How does Resoto work?
Resoto indexes the infrastrcure resources of your AWS account(s) by capturing resource dependencies and mapping your entire
Infrastructure in a graph database. This graph database can then be easily queried using Resoto's powerful search syntax.

**To list a few examples-**

A. `search is(aws_account)`   
*Lookup AWS 12-digit account ID and name* 

<img src="{{site.baseurl}}/images/resoto/aws_account.png">

B. `search is(aws_ec2_instance) and age > 2year` 

*Find EC2 instances older than 2 years* 

<img src="{{site.baseurl}}/images/resoto/ec2_instance_2year.png">

C. `search is(aws_ec2_instance) and age > 10days and instance_status=running | count instance_type`

*Find running EC2 instances older than 10 days  and print the output with instance type counts* 
<img src="{{site.baseurl}}/images/resoto/ec2_instance_count.png">

D. `search is(aws_ec2_instance) and /ancestors.region.reported.name=="ap-southeast-2" and instance_status=running|count instance_type`

*Find the  EC2 instance type  count filtered by AWS region ID* 

<img src="{{site.baseurl}}/images/resoto/ec2_instance_by_region.png">


As demonstrated in the example above the Resoto search syntax makes it easy to look up cloud resources and save admin time previously spent  writing and maintaining your own custom CLI scripts.

# What makes up Resoto?
Resoto is made up of several components - including as Resoto Core, Resoto Worker, Resoto Metrics and Resoto Shell also, ArangoDB servers as the graph database backend.

So what does each component do? [source](https://resoto.com/docs/getting-started/install-resoto/pip#installing-resoto) 

 - *resotocore* maintains the infrastructure graph.
 - *resotoworker* collects infrastructure data from the cloud provider APIs.
 - *resotometrics* exports metrics in Prometheus format.
 - *resotoshell* is the command-line interface (CLI) used to interact with Resoto.
 - *resoto-plugins* a collection of worker plugins

# Bootstrap Resoto
#### Installation

Resoto can be installed using the following methods–
1.	Docker Compose
2.	Kubernetes
3.	Python pip package manager 

[Resoto website has detailed instructions on how to get started with the above methods](https://resoto.com/docs/getting-started/install-resoto)

#### Deploy Resoto on EC2 using Terraform
For my own experimentation with Resoto,  I have created a Terraform template to spin up Resoto on an EC2 Instance with an IAM Profile - I  Open Sourced the  on  Github at URL --> 
[terraform-template-resoto](https://github.com/msharma24/terraform-template-resoto) 

#### How to deploy Resoto on EC2 using Terraform

1. `git clone git@github.com:msharma24/terraform-template-resoto.git`
2. Export your AWS IAM Credentials 
3. Run `terraform init`
4. `terraform apply [-auto-approve]`
5. Once the deployment completes,  Terraform will print an ssh command.
6. SSH into the EC2 Instance using the command from the Terraform output.
7. Run the following commands to access the `Resoto Shell` (if you installed Resoto via `pip` with `install_method` as `local`)

```
sudo su
cd /root
pre_shared_key=$(< ~/resoto/.pre-shared-key)
source ~/resoto/resoto-venv/bin/activate
resh --resotocore-uri https://localhost:8900 --psk "$pre_shared_key"
```
You will presented with the Resoto Shell prompt `>`

8  At the Resoto Shell prompt - run the following command `config edit resoto.worker` and set the `collector` to `aws`:
<img src="{{site.baseurl}}/images/resoto/resoto_worker_config.png">

9 Once the config has been updated, Run the following command in the Resoto Shell to kick off resource and dependency collection.

`workflow run collect`

10 If this is your first collect run it can take few minutes for Resoto to collect the information. This job will run every hour to add/update new resource data collection.

11 To verify Resoto has successfully collected resource information by running `search is(aws_account)` , which prints your account information.

#### Whats next?
Browse through the official Resoto documentation which includes  [How to guides](https://resoto.com/docs/how-to-guides) and examples to query the resources in your AWS Account.
If you find Resoto valuable please - Star ★ the Resoto Github repository  to support the project [https://github.com/someengineering/resoto](https://github.com/someengineering/resoto) 

#### Resoto Community
Join the official Resoto Discord Server at [https://discord.gg/someengineering](https://discord.gg/someengineering)  to hang out with the Resoto dev team and other community members,including myself :)



