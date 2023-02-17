# Week 0 — Billing and Architecture

## Homework Hard Assignments
1. Set a Billing alarm
    - setup a billing alarm using the user interface as shown by Chirag
    - also setup a billing alarm using CLI as shown by Andrew
    ![Setup Alarm](/journal/images/Week0-SetupAlarm.png)
2. Set a AWS Budget   
    - Setup a zero spend budget in AWS account as shown by Chirag 
    - Setup a budget using CLI as shown by Andrew
    ![Setup Budget](/journal/images/Week0-SetupBudget.png)
3. Generating AWS Credentials 
    - Created IAM user 
    ![IAM user](/journal/images/Week0-CreateIAMUser.png)
4. Using CloudShell 
    - Install cli in gitpod
    - setup gitpod environment variables for AWS IAM user access keys
    - updated the gitpod.yml file so that everytime the environment is launched cli is available
    - setup environment variable for aws account id
    - created budget using CLI
    - created an sns using CLI
    - created alarm using CLI
5. Conceptual Architecture Diagram or your Napkins 
    Conceptual Design Diagram 
    (https://lucid.app/lucidchart/2890b70a-aa69-4be6-817b-ac03deecad79/edit?viewport_loc=-247%2C-11%2C2219%2C979%2C0_0&invitationId=inv_85b5de19-4240-48bf-8be0-7eaba2b11e37)

## Homework Stretch Assignments
1. Destroy your root account credentials, Set MFA, IAM role
- created IAM Role and setup MFA
2. Use EventBridge to hookup Health Dashboard to SNS and send notification when there is a service health issue.
(ref:https://docs.aws.amazon.com/health/latest/ug/cloudwatch-events-health.html)
![Event Bridge 1](/journal/images/Week0-Amazon EventBridge.png)
![Event Bridge 2](/journal/images/Week0-Amazon EventBridge-SNSTopic.png)
3. Review all the questions of each pillars in the Well Architected Tool (No specialized lens)
4. Create an architectural diagram (to the best of your ability) the CI/CD logical pipeline in Lucid Charts
Cruddur Logical Diagram (https://lucid.app/lucidchart/24d50af4-522a-45e9-9c3d-c8eed74bc2aa/edit?view_items=jcjxtnjjyBer&invitationId=inv_5947252d-342b-43cf-8d25-eba34828548c)
5. Research the technical and service limits of specific services and how they could impact the technical path for technical flexibility. 
6. Open a support ticket and request a service limit

## Bonus
![Napkin Design](/journal/images/week0-NapkinDesign.jpg)

## Misc Notes
- /usr/bin - system installs  vs /usr/local/bin - logged in user install
- when aws packages is installed the location is stored in the $PATH env variable
- command to get current identity
```
aws sts get-caller-identity
```
- to reset the aws idenity keys
```
export AWS_ACCESS_KEY_ID=""
export AWS_SECRET_ACCESS_KEY=""
export AWS_DEFAULT_REGION=us-east-1
```
- to set the aws identity keys 
```
export AWS_ACCESS_KEY_ID="{value of IAM Access Key}"
export AWS_SECRET_ACCESS_KEY="{value of IAM Access Key Secret}"
export AWS_DEFAULT_REGION="{value of region}"
```
- once aws identity key the Account should automatically be set to your account
- to set account id as a environment variable
```
export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text )
```

## Pricing Module - Chirag
- AWS services varies according to region. Always select region where you want to spin up service
- Bill will be in USD and local currency
- Conversion rate applies on date of invoice with latest exchange rate
- 750 Hours free utilization 
- AWS Billing Alerts vs AWS Budgets
    AWS Billing Alerts is an older feature released in 2012 that leverages Amazon Cloudwatch alarm and metrics to monitor charges and Amazon SNS to send email alerts. It was AWS first attempt to provide proactive alerting and alarms on your total AWS charges or a specific AWS service. The feature is relatively basic when it comes to pure costs/budgeting and there isn’t a way to setup in-depth budget monitoring with the filters you have in Cost Explorer.On the flip-side, since it is uses CloudWatch, you can create an alarm that correlates between RDS CPU utilization and it’s total monthly budget. The same can be applied across many other CloudWatch metrics.

    AWS Budgets -Released 7 years later in 2019, AWS Budgets builds upon AWS billing alerts and is integrated into Amazon’s billing dashboard. Overall, it provides a better and more robust way to monitor your AWS costs. All the dimensions you are familiar with in Cost Explorer can be developed into monitoring daily / weekly or monthly AWS Budgets via email. In addition, AWS Budgets gives you a way to trigger specific event if a threshold is reach through “Budget Actions”.
- AWS Cost Explorer
    AWS Cost Explorer has an easy-to-use interface that lets you visualize, understand, and manage your AWS costs and usage over time.

## Security Module - Ashish
- Organization Unit
    - acts like a governor 
    - offered as part of Organization service
    - Management account should not have any services, it should be used to simply create Organization account
    - Best practice is to create Active Account/Standby Accounts and then Business Units under Active Accounts. Create multiple accounts under standby. When a new BU comes in move account from Standby to Active/BU 

- AWS Cloud Trail
    - service to monitor data security and residence
    - under region vs avaialbility zone vs Global service Concept
        - Region = Us east coast = large 
        - Availability Zones = within region is subnetworks
        - Global service = Cloud trail by default will only log activity in the region it is activated in unless you enable it for all your services in all regions
    - Audit Logs for IR/Forenscis
    - S3 bucket<=>Dropbox
    - AWS KMS = private key to encrypt/decrypt

- Create IAM User
    - 3 kinds of users  - human IAM User (username/password) , federated users(on premise environment), System IAM User( )
    - Enable MFA for all human users
    - Principle of least previledge
    - IAM user is global service not restricted to region
    - Best practice to use IAM user and never root user

- Create IAM Role & IAM Policy
    - 2 types - AWS Managed Policy
    - IAM Role = what can and cannot be done. a role is a type of IAM identity that can be authenticated and authorized to utilize an AWS resource.  IAM Roles manage who has access to your AWS resources
    - IAM Policy = can be attached to a group of users or individual or role. a policy defines the permissions of the IAM identity. IAM policies control their permissions.
    - User or EC2 instance can use a role

- Enable AWS Organization SCP(service control policies)
    - example policies in https://github.com/hashishrajan/aws-scp-best-practice-policies

- What's the difference between an AWS Organizations service control policy and an IAM policy?
    https://aws.amazon.com/premiumsupport/knowledge-center/iam-policy-service-control-policy/

-