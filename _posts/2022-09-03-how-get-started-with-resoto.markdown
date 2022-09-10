---
layout: post
title: How to get started with Resoto on AWS
---
<img src="{{site.baseurl}}/images/resoto/resoto_cli.png">
# Introduction
[Resoto](https://resoto.com/about) is an [Open Source](https://github.com/someengineering/resoto)  CLI tool by [someengineering](https://some.engineering/)  that not only helps to simplify querying your entire cloud infrastructures inventory at 
the fingertips but also eases the mundane cloud janitor tasks such as finding and deleting the unused cloud resources across multiple AWS accounts which could be costing the business an arm and a leg and documenting cloud inventory for audit purposes in a pretty format.

# How does Resoto work?
Resoto indexes the infrastrcure resources of your AWS Account(s) by capturing the resource dependencies and maps out the entire
infrastrcure in a graph database, this graph database can then be easily queried using Resoto's powerful search sytanx.

**To list a few examples :-**

A. `search is(aws_account)`   
*Lookup AWS 12 Digit Account id and name* 

<img src="{{site.baseurl}}/images/resoto/aws_account.png">

B. `search is(aws_ec2_instance) and age > 2year` 

*Find EC2 Instances older than 2 years* 

<img src="{{site.baseurl}}/images/resoto/ec2_instance_2year.png">

C. `search is(aws_ec2_instance) and age > 10days and instance_status=running | count instance_type`

*Find EC2 Instance older than 10 Days in running status and print the output with instance type count* 
<img src="{{site.baseurl}}/images/resoto/ec2_instance_count.png">

D. `search is(aws_ec2_instance) and /ancestors.region.reported.name=="ap-southeast-2" and instance_status=running|count instance_type`

*Find the count of the EC2 instances by instance type filtered by AWS Region id* 

<img src="{{site.baseurl}}/images/resoto/ec2_instance_by_region.png">


As demostrated in the example above the Resoto query syntax makes it easy to look up cloud resources and save admin time writing and maintaining your own customer CLI scripts.

# What makes up Resoto ?
Resoto is made up of the multiple components - such as Resoto Core , Resoto Worker , Resoto metrics and resotoshell also, ArangoDB which servers as the graph database backend.

so what are these components about ? [source](https://resoto.com/docs/getting-started/install-resoto/pip#installing-resoto) 

 - *resotocore* maintains the infrastructure graph.
 - *resotoworker* collects infrastructure data from the cloud provider APIs.
 - *resotometrics* exports metrics in Prometheus format.
 - *resotoshell* is the command-line interface (CLI) used to interact with Resoto.
 - *resoto-plugins* a collection of worker plugins

# Bootstrap Resoto
#### Installation

Resoto can be installed using the following methods –
1.	Docker and docker-compose
2.	Kubernetes
3.	Python pip package manager 

Resoto website has detailed instructions on how to get started with the above methods - [Link to official doc](https://resoto.com/docs/getting-started/install-resoto) 

#### Deploy Resoto on EC2 using Terraform
For my own experimentation with Resoto - I  have setup a Terraform template to spin up the Resoto on an EC2 Instance with an IAM Profile - The project is Open Source on my Github Profile - Link to Github Repo --> 
[terraform-template-resoto](https://github.com/msharma24/terraform-template-resoto) 

#### How to deploy Resoto on EC2 using Terraform

1. Git Clone Reo Git clone the Github Repo [https://github.com/msharma24/terraform-template-resoto](https://github.com/msharma24/terraform-template-resoto)  
2. Export the AWS Credentials 
3. Run terraform init
4. terraform apply [-auto-approve]
5. Once the deployment completes the Terraform outputs will print the ssh command to EC2 Instance .
6. SSH to the EC2 Instance using the command in the Terraform output
7. Run the following commands to access the `resotoshell` if you chose to install resoto via `pip` with `install_method` as `local`

```
sudo su
cd /root
pre_shared_key=$(< ~/resoto/.pre-shared-key)
source ~/resoto/resoto-venv/bin/activate
resh --resotocore-uri https://localhost:8900 --psk "$pre_shared_key"
```
and you will presented with the resoto shell prommpt `>`

8  At the resoto shell prompt - Run the following command `config edit resoto.worker` and set the `collector` to `aws` as seen below:
<img src="{{site.baseurl}}/images/resoto/resoto_worker_config.png">

9 Once the config has been updated, Run the following command in the resotoshell to kick off collecting the resource information and map the dependencies in the ArangoDB.

`workflow run collect`

10 If this is your first run it can take few minutes for resoto to collect the information and this job will then re-run every hour to add/update new resource data collection
11 verify resoto is now collecting resource information by running `search is(aws_account)` , which print your account information.

#### Now, Whats next ?
Browse through the official Resoto documentation with How to Guides and examples to query the resources in your own AWS Account.
If you find Resoto valuable for your purposes - Star the Resoto Github Repo [https://github.com/someengineering/resoto](https://github.com/someengineering/resoto) 

#### Resoto Community
Join the official Resoto Discord Community [https://discord.gg/someengineering](https://discord.gg/someengineering)  to hang out with the Resoto Dev Team and other community members , including myself :)



