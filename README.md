# AWS Security StepFunction Integration

## Description
With the release of [AWS SDK Service Integrations](https://aws.amazon.com/about-aws/whats-new/2021/09/aws-step-functions-200-aws-sdk-integration/), 
it is now easier than ever to orchestrate services. As *Security is our highest priority* and AWS's security services required
by regulated industries are not enabled out of the box, this project aims to orchestrate and choreograph foundational
security requirements to secure a singular account in a for development purposes, which would typically be done manually or
require several layers of tooling to automate. The use of function-less hugely reduces the need for support/maintenance of the
solution while it can be presented as a single CloudFormation file that can easily be consumed. The solution will demonstrate
idempotency, having the ability to run over itself to enforce the controls over time, and provide alerting on non-compliance.  

The end result is modular state machines targeting core AWS account security controls which would allow a user to start
experimenting in the cloud with a level of guardrails, alerting and managed services. Each modular component could be taken
and used independently to resolve a security risk, but ultimately act as a public example of the power of AWS Step Functions
service integrations in account operations and security context. 

## Deployment
This project is a simple single YAML CloudFormation script. Deployment is easy, when logged into your AWS account use the one click deployment to open up the template loaded into the CloudFormation Console.

[One Click deploy (v0.0.1)](https://console.aws.amazon.com/cloudformation/home?#/stacks/create/review?templateURL=https://aws-security-stepfunction-integration.s3.eu-west-2.amazonaws.com/release/ssi-deploy-0.0.1.yml)

It can also be deployed using the [CloudFormation Console](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html) or using the [AWS CLI](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-using-cli.html).

*Note: The deployment is regional - Please check the right hand corner to ensure you are in the region you would like to use.*

Once in the Console you should see the following:


![Deploy](./img/example-deploy.PNG?raw=true "Deploy")

This is where you can tailor the security controls for your account. Make sure you understand the controls and costs associated with this deployment.    

All logs will be sent to an S3 bucket in your account which has the name of `ssi-logging-${AWS::AccountId}-${AWS::Region}`. You may optionally add another layer of security by configuring a bucket to enable MFA (multi-factor authentication) delete. For more information see here: [Configuring MFA delete](https://docs.aws.amazon.com/AmazonS3/latest/userguide/MultiFactorAuthenticationDelete.html)

Once deployed, Navigate to the `SSI-Main Step` function which is now set to run on a schedule. Alternatively, you can trigger the step function manually at any time.  The step function can be run with the default deployment input.

## Usage

![Main SF](./img/main-sf.PNG?raw=true "Main SF")

The output of this step function will give you a report on the status of your account.

Please see below an Example output:
```json
[
  {
    "LoggingBucket": {
      "Description": "S3 Bucket created as part of inital cloudformation deployment used to store logs",
      "Name": "ssi-logging-1234567890-eu-west-2",
      "Arn": "arn:aws:s3:::ssi-logging-1234567890-eu-west-2"
    },
    "SecurityHub": {
      "Status": "AWS Security Hub is subscribed"
    },
    "CloudTrail": {
      "Status": {
        "Status": "Trail exists and logging"
      }
    },
    "Config": {
      "Status": {
        "Config": "AWS Config is enabled",
        "ConformancePack": "Conformance pack deployed"
      }
    },
    "GuardDuty": {
      "Status": "Amazon GuardDuty enabled"
    },
    "Macie": {
      "Status": "Macie enabled"
    },
    "AuditManager": {
      "Status": "Audit Manager enabled"
    },
    "Inspector": {
      "Status": "AWS Inspector is enabled"
    },
    "AccountLevelSettings": {
      "Status": [
        {
          "S3PublicAccess": "S3 BPA enabled"
        },
        {
          "DefaultEbsEncryption": "Enabled"
        }
      ]
    }
  }
]
```
This shows you the status of the security controls within your account at the time of the step function execution.

## Deployment Inputs

The list of parameters for this template:

| Parameter Name | Description |
|----------------|-------------|
|Deployment Settings/Notification Email| Email address that will be alerted if deployment fails.|
|Deployment Settings/Deployment Frequency| Deployment Frequency (minutes):|
|AWS Security Hub/AWS Foundational Security Best Practices [Mandatory]|The standard is defined by AWS security experts. This curated set of controls helps improve your security posture in AWS, and covers AWS?s most popular and foundational services.|
|AWS Security Hub/CIS AWS Foundations Benchmark|The Center for Internet Security (CIS) AWS Foundations Benchmark v1.2.0 is a set of security configuration best practices for AWS.|
|AWS Security Hub/PCI DSS|The Payment Card Industry Data Security Standard (PCI DSS) v3.2.1 is an information security standard for entities that store, process, and/or transmit cardholder data.|
|AWS CloudTrail/Enable multi region trail| You can configure AWS CloudTrail to deliver log files from multiple regions to a single S3 bucket for a single account.|
|AWS Config/Conformance Pack|A collection of AWS Config rules and remediation actions|
|AWS GuardDuty/Enable Guardduty|Amazon GuardDuty is a monitoring service that analyzes core services to generate security findings for your account.|
|Amazon Macie/Enable Macie|Amazon Macie is a fully managed data security and data privacy service that uses machine learning and pattern matching to discover and protect your sensitive data in AWS.|
|AWS Audit Manager/Enable Audit Manager|AWS Audit Manager helps you continuously audit your AWS usage to simplify how you assess risk and compliance with regulations and industry standards.|
|Amazon Inspector/Enable Amazon Inspector|Automated and continual vulnerability management at scale.|
|Account Level Settings/Block public access to your Amazon S3 storage|The Amazon S3 Block Public Access feature provides settings for access points, buckets, and accounts to help you manage public access to Amazon S3 resources.|
|Account Level Settings/Enforce EBS encryption|Enforce the encryption of new EBS volumes and snapshot copies that you create|


## Solution design

![Architecture Diagram](./img/architecture-diagram.png?raw=true "Architecture Diagram")

## Elevator Pitch: 
 You want to start using the cloud but are overwhelmed by the different security services. You don't want to have to manually go into each service and configure it
 but nor do you want to deploy a complex software package over the top. You either looking to build out a landing zone but want to start getting your engineers
 hands on while you finish defining requirements. What you need is AWS Security StepFunction Integration [SSI]. It's a one click lightweight StepFunction that contains
 a few lines of mark-up that configures security best practices and guardrails to get up and running. It is no means a landing zone, we have Control Tower and AFT for 
 more long term production applications, but SSI will enable basic monitoring, alerting, logging and threat detection to give you a safer place to start your cloud
 journey.  

## Who is the customer? 
### Customer 1:
You are an engineer working for a large regulated company. Your CEO has heard of the great opportunities the cloud has to offer and has handed you the golden credit card to create an AWS account to start exploring cloud services. You want to get a demo together quickly. Although the application is not sensitive in any way, you still want to make sure your account cannot easily be exploited.
### Customer 2:
You are a small organisation who requires the use of 1 or two AWS services for a single workload. You want to ensure that you have security services offered by AWS but you are overwhelmed by the scattered different services and are not sure how to configure them.
### Customer 3:
You are a Security consultant wanting to demonstrate to a client the different AWS services there are to offer. You want a quick way to turn on services and get them running to show them in action to your client.
### Customer 4:
You have an existing security solution deployed into your account. However, one of the security services is not supported or enabled. You want a lightweight option to enable that one service in and a mechanism to ensure its stayed turned on.   
### Customer 5:
You are working in a Cloud Centre of Excellence (CCoE) with an AWS partner who is currently assisting you build out for first Landing Zone. Its taking some time to get your requirements together with your auditors and operations teams. Meanwhile, your application teams are sat waiting for an environment to be ready to start experimenting with sandbox workloads.  

## What is the prevailing problem? 

### Manual activity
The first thing a user should do with a new account is lock it down. Each service requires multiple key strokes to be configured. Steps creating IAM users or creating buckets requires cloud knowledge.
### Large packages 
Although many solutions have tried to do this in the past, they have been complex requiring vast prerequisite and dependences on cli/python/npm etc…
### Limited auditing
The auditing out of the box for a fresh account does not give you centralised logging of all activity within your account. 
### Complex/Overwhelming  
Long term having a landing zone is the solution, but this can be overwhelming for new customers and building one out to meeting the organisation needs can slow down time it takes a developer to create their first workload in the cloud.
### Support
Support has always been an issue for security accelerators. Using step function service integrations the run time environment is fully managed.  
### Insecure 
Options such as account level EBS encryption and S3 block public access are not enabled out of the box.

## FAQ

### Q: I need a landing Zone
A: This solution is for a single AWS account. Although this could be deployed as a stackset or adapted for more than 1 account, its not recommended for a multi account environment as all the logging in local to that account and there is no centralised delegation. 

### Q: I already have a solution, I only need 1 additional service enabled
A: This solution may still help! Each service has its owns state machine so you can extract the one you want!

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

## Uninstall
The uninstall step is currently manual. To do this, you will need to remove the CloudFormation stack removing the orchestration resources then follow up by deleting any changes you want to reverse to the services through the console.

To delete the orchestration CloudFormation, please see: [Deleting a stack on the AWS CloudFormation console](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-delete-stack.html)

Find below instructions on disabling the enabled services:

- [Disabling AWS Security Hub](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-disable.html)
- [AWS Cloudtrail - Deleting a trail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-delete-trails-console.html)
- [Managing Conformance Packs Across all Accounts in Your Organization](https://docs.aws.amazon.com/config/latest/developerguide/conformance-pack-organization-apis.html)
- [Managing the Configuration Recorder](https://docs.aws.amazon.com/config/latest/developerguide/stop-start-recorder.html)
- [Suspending or stopping GuardDuty](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_suspend-disable.html)
- [Suspending or disabling Amazon Macie](https://docs.aws.amazon.com/macie/latest/user/macie-suspend-disable.html)
- [Disable AWS Audit Manager](https://docs.aws.amazon.com/audit-manager/latest/userguide/console-settings.html#disable)
- [Disabling Amazon Inspector](https://docs.aws.amazon.com/inspector/latest/user/disabling-best-practices.html)
- [Configuring block public access settings for your account](https://docs.aws.amazon.com/AmazonS3/latest/userguide/configuring-block-public-access-account.html)
- [EBS Encryption by default](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html#encryption-by-default)