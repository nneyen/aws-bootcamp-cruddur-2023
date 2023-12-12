# Week 0 — Billing and Architecture
---
I am currently taking the cloud boot camp project with [@andrewbrown](https://www.linkedin.com/in/andrew-wc-brown/) and I have committed to writing a journal as instructed by andrew. 

Before diving into the project, we have to first set up Billing and Alarms. This is good practice so we do not end up owing AWS Loads of money 💰. 

Now the fun part of this task is we will be using aws-cli for everything!

## Task 1 : Setting Up AWS CLI
---
### Install AWS CLI
- Installing AWS CLI needs to be automoted, else it will have to be done each time you load your workspace

- To install AWS CLI, simply update `.gitpod.yml` with the following commands:

```
tasks:
 - name: aws-cli
   env:
    AWS_CLI_AUTO_PROMPT: on-partial
   init:
    cd /workspace
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    sudo ./aws/install
    cd $THEIA_WORKSPACE_ROOT
```

### Create a New User and Generate AWS Credentials
This is necessary to allow us work on AWS from console. 
- Navigate to IAM Console
- Create a new user, and `enable console access`
- Edit the new user and set up security credentials i.e. `access key` and `secret access key` with AWS CLI Access
- Keep the credentials safe as you would need it

### Set up Gitpod Workspace ENVS

Note : Even after setting this up, I find that I have to use aws configure each time. Not sure why however here are the commands you can enter on your terminal (note you can also do this straight from your gitpod dashboard but that's boring 😝)

```
    gp env AWS_ACCESS_KEY="youraccesskey"
    go env AWS_SECRET_ACCESS_KEY="yoursecretaccesskey"
    gp env AWS_DEFAULT_REGION="yourpreferedregion"
```
You can also go ahead and include your account ID as an env using similar commands. 

To test that your AWS CLI is working, run the following commands

```
aws sts get-caller-identity

```

here's what it would look like 

```
    "UserId": "AIFBZRJIQN2ONP4ET5BK6",
    "Account": "123456789098",
    "Arn": "arn:aws:iam::123456789098:user/youruser"

```
NOTE: DO NOT EXPOSE YOUR SECRET OR ACCESS KEYS ! 

---

## Task 2: Create an AWS Budget
---

I got the commands for this section here [create aws budgets](https://docs.aws.amazon.com/cli/latest/reference/budgets/create-budget.html)

From the page above copy the example for `budget.json` and `notifications-with-subscribers` and store in /aws/json. You can edit the amount and the unit to suit your purposes

Next run the following CLI command: 
```
aws budgets create-budget \
    --account-id $AccountID \
    --budget file://aws/json/budget.json \
    --notifications-with-subscribers file://aws/json/budget.json

```
Note if you stored your account ID as an env, then you can use the following `$ACCOUNT_ID`.

If you are unsure how to pull up your accout ID, use the command

```
aws sts get-caller-identity --query Account --output text

```

DO NOT EXPOSE YOUR ACCOUNT ID !

---

## Task 3: Enable Billing Alarm

---

The last task for this week is configuring AWS Billing alarm. First you have to enable billing in your ROOT account so you can recieve alerts.
- Go to your `Billing Page`, and find `Billing Preferences`
- Check `CloudWatch Billing Alerts`
- Save your preferences

### Create a Billing Alarm

In order to create a billing alarm, you have to first set up an SNS topic. SNS Topics are used to send notifications or messages. [The instructions for creating an sns topic from cli can be found here](https://docs.aws.amazon.com/cli/latest/reference/sns/create-topic.html)

In your terminal enter the following commands:
```
    aws sts create-topic --name billing-alarm

```
This will produce a TopicARN which we will use to when creating a subscription for our alerts. 

This command is used for creating a subscription:

```
aws sns subscribe \
    --topic-arn TopicARN \
    --protocol email \
    --notification-endpoint your@email.com

```
You will recieve an email in your inbox, and you can confirm your subscription. It also helps confirm this is visible in your aws console as well. 

Once subscription is confirmed, the next step is to create an alarm
[instructions can be found here](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-alarm.html)

From the page above copy the example for `alarm-config.json` and store in /aws/json.

Next use the following you commands to create an alarm

```
 aws cloudwatch put-metric-alarm --cli-input-json file://aws/json/alarm-config.json
```

Confirm that your metric has been creating by Navigating to AWS CloudWatch Dashboard

And that's it! We are done!