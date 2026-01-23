# AWS Security StepFunction Integration

Automated AWS account security configuration using Step Functions. Orchestrates foundational security services and controls without requiring Lambda functions or complex tooling.

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

## What This Is NOT

**This is not a landing zone.** This solution is designed for short-term experimentation and development in standalone AWS accounts, not for production multi-account organizations.

**Not for Organizations:** All logging is local to the deployed account with no centralized delegation. For multi-account environments, use AWS Control Tower or [AWS Landing Zone Accelerator](https://github.com/awslabs/landing-zone-accelerator-on-aws)

**Not for Production Workloads:** While this solution implements security best practices, it's intended for sandbox, development, and experimentation accounts. Production environments should use enterprise landing zone solutions with centralized governance.

**Use this solution when:**
- Setting up a sandbox account for experimentation
- Quickly securing a development account while your landing zone is being built
- Learning AWS security services in a safe environment
- Demonstrating security capabilities to stakeholders
- Enabling security controls in a single standalone account

## Deployment

**Recommended: Use the [CloudFormation Console](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html)**
1. Download `deploy-modernized.yml` from this repository
2. Navigate to CloudFormation in your AWS Console
3. Create stack → Upload template file
4. Configure parameters (see Deployment Inputs section below)
5. Review and create

*Note: The deployment is regional - Please check the right hand corner to ensure you are in the region you would like to use.*

Once in the Console you should see the following:

![Deploy](./img/example-deploy.PNG?raw=true "Deploy")

This is where you can tailor the security controls for your account. Make sure you understand the controls and costs associated with this deployment.

**Alternative: AWS CLI**
```bash
# Clone the repository
git clone <repository-url>
cd aws-security-stepfunction-integration-main

# Deploy using AWS CLI
aws cloudformation create-stack \
  --stack-name aws-security-ssi \
  --template-body file://deploy-modernized.yml \
  --parameters ParameterKey=pNotificationEmail,ParameterValue=your-email@example.com \
  --capabilities CAPABILITY_IAM \
  --region us-east-1
```    

All logs will be sent to an S3 bucket in your account which has the name of `ssi-logging-${AWS::AccountId}-${AWS::Region}`. You may optionally add another layer of security by configuring a bucket to enable MFA (multi-factor authentication) delete. For more information see here: [Configuring MFA delete](https://docs.aws.amazon.com/AmazonS3/latest/userguide/MultiFactorAuthenticationDelete.html)

Once deployed, Navigate to the `SSI-Main` Step Function which is set to run on a schedule. You can also trigger it manually at any time with the default input.

## Key Features

### ðŸ”’ Security Hub with CSPM
- Security Hub V2 with centralized security posture management
- CSPM enabled by default for continuous posture monitoring
- Support for AWS Foundational, CIS, PCI DSS, and NIST 800-53 standards
- Scalable architecture for adding standards over time

### ðŸ›¡ï¸ IAM Security
- Password policy with complexity requirements (14+ chars, 90-day expiration, 24 password history)
- IAM Access Analyzer to identify resources shared externally
- Dedicated state machine for IAM controls

### ðŸ“Š CloudWatch Logs Protection
- Kinesis Firehose delivery stream for log aggregation to S3
- Account-level subscription filter policy preventing data exfiltration
- Automatic log compression (GZIP) and encryption (KMS)
- Logs stored in centralized S3 bucket with lifecycle policies

### Security Findings Capture
- **Security Hub findings** automatically captured and archived (CRITICAL, HIGH, MEDIUM severity)
- **Inspector findings** captured for vulnerability management
- **IAM Access Analyzer findings** captured for external access detection
- All findings flow through EventBridge → CloudWatch Logs → Firehose → S3
- 90-day retention in CloudWatch Logs with long-term S3 storage

### ðŸ” Account Level Settings (7 Parallel Controls)
- S3 Block Public Access
- EBS Encryption by default
- IMDSv2 required for EC2 instances
- RDS Encryption enforcement
- EC2 Serial Console disabled
- EBS Snapshot Block Public Access
- RDS Public Accessibility disabled

### ðŸ“¦ Additional Services
- **CloudTrail**: Multi-region trail with S3 and KMS encryption, optional data events for S3/Lambda
- **Config**: Configuration recorder with conformance packs (CIS, PCI DSS, NIST, HIPAA, Control Tower)
- **GuardDuty**: Threat detection with S3 protection and automatic publishing to S3
- **Macie**: Sensitive data discovery
- **Audit Manager**: Compliance auditing (region-dependent)
- **Inspector**: Vulnerability management

### Reliability Features
- **SNS Dead Letter Queue**: 14-day retention for failed error notifications
- **Firehose Error Logging**: Dedicated CloudWatch Logs for delivery failures
- **Exponential Backoff**: Automatic retry logic for rate-limited API calls
- **Idempotent Execution**: Safe to run repeatedly without side effects

### ðŸŒ Operational Excellence
- Graceful handling of region-specific service availability
- Exponential backoff retry logic for rate limiting
- Comprehensive error handling and logging
- Idempotent execution - safe to run repeatedly
- Scheduled execution with configurable frequency

### ðŸ’° Cost Optimizations
- S3 Bucket Keys (99% reduction in KMS costs)
- Lifecycle policies for automatic storage tiering
- Optimized log retention (90 days)
- Efficient resource tagging for cost allocation

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
      "Status": "AWS Security Hub V2 with CSPM enabled",
      "Standards": {
        "AWS Foundational": "Enabled",
        "CIS": "Enabled",
        "PCI DSS": "Enabled",
        "NIST 800-53": "Enabled"
      }
    },
    "CloudTrail": {
      "Status": {
        "Status": "Trail exists and logging"
      }
    },
    "CloudWatchLogs": {
      "Status": "CloudWatch subscription filter policy configured successfully"
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
    "IAMSettings": {
      "Status": {
        "PasswordPolicy": "Configured",
        "AccessAnalyzer": "Enabled"
      }
    },
    "AccountLevelSettings": {
      "Status": [
        {
          "S3PublicAccess": "S3 BPA enabled"
        },
        {
          "DefaultEbsEncryption": "Enabled"
        },
        {
          "IMDSv2": "Required"
        },
        {
          "RDSEncryption": "Enforced"
        },
        {
          "EC2SerialConsole": "Disabled"
        },
        {
          "EBSSnapshotBlockPublic": "Enabled"
        },
        {
          "RDSPublicAccess": "Disabled for all instances"
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
|AWS Security Hub/AWS Foundational Best Practices|Curated set of controls for AWS security posture (recommended).|
|AWS Security Hub/CIS Benchmark|Security configuration best practices for AWS.|
|AWS Security Hub/PCI DSS|Information security standard for entities that store, process, or transmit cardholder data.|
|AWS Security Hub/NIST 800-53|Catalog of security and privacy controls for information systems and organizations.|
|AWS CloudTrail/Enable multi region trail| You can configure AWS CloudTrail to deliver log files from multiple regions to a single S3 bucket for a single account.|
|AWS CloudTrail/Enable CloudTrail Data Events|Enable data events for S3 object-level operations and Lambda invocations. Note: Increases logging costs.|
|AWS CloudWatch Logs/Configure CloudWatch Subscription Filter Policy|Configure account-level CloudWatch Logs subscription filter policy to prevent data exfiltration.|
|AWS Config/Conformance Pack|A collection of AWS Config rules and remediation actions. Select 'None' to skip conformance pack deployment.|
|AWS GuardDuty/Enable Guardduty|Amazon GuardDuty is a monitoring service that analyzes core services to generate security findings for your account.|
|Amazon Macie/Enable Macie|Amazon Macie is a fully managed data security and data privacy service that uses machine learning and pattern matching to discover and protect your sensitive data in AWS.|
|AWS Audit Manager/Enable Audit Manager|AWS Audit Manager helps you continuously audit your AWS usage. NOTE: Not available in all regions.|
|Amazon Inspector/Enable Amazon Inspector|Automated and continual vulnerability management at scale.|
|IAM Security Settings/Configure IAM Password Policy|Configure IAM password policy: 14+ chars, requires symbols/numbers/upper/lower case, 90-day expiration, 24 password history.|
|IAM Security Settings/Enable IAM Access Analyzer|Enable IAM Access Analyzer to identify resources shared with external entities.|
|Account Level Settings/Block public access to your Amazon S3 storage|The Amazon S3 Block Public Access feature provides settings for access points, buckets, and accounts to help you manage public access to Amazon S3 resources.|
|Account Level Settings/Enforce EBS encryption|Enforce the encryption of new EBS volumes and snapshot copies that you create.|
|Account Level Settings/Require IMDSv2 for EC2 instances|Require IMDSv2 (Instance Metadata Service Version 2) for enhanced EC2 instance security.|
|Account Level Settings/Enforce RDS encryption|Enforce encryption for new RDS database instances and clusters.|
|Account Level Settings/Disable EC2 Serial Console|Disable EC2 Serial Console access for enhanced security.|
|Account Level Settings/Block Public EBS Snapshots|Block public access to EBS snapshots at the account level to prevent accidental data exposure.|
|Account Level Settings/Disable RDS Public Accessibility|Disable public accessibility for new RDS database instances and Aurora clusters.|


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
Although many solutions have tried to do this in the past, they have been complex requiring vast prerequisite and dependences on cli/python/npm etcâ€¦
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

### Q: What's the time-to-value compared to manual configuration?
A: Manual configuration of these security services typically takes 2-3 days of engineering time and requires deep AWS expertise. This solution deploys in under 30 minutes with a single CloudFormation stack. The scheduled execution ensures controls stay enabled and compliant over time, eliminating ongoing manual verification work and reducing operational overhead by 90%.

### Q: Where are my security findings stored?
A: Security findings from Security Hub, Inspector, and IAM Access Analyzer are automatically captured via EventBridge and stored in CloudWatch Logs (90-day retention), then aggregated to your S3 logging bucket via Kinesis Firehose for long-term storage. All logs are encrypted with KMS and compressed with GZIP.

### Q: What happens if the Step Function fails?
A: If any state machine execution fails, an SNS notification is sent to your configured email address. Failed notifications are retained in an SQS Dead Letter Queue for 14 days, ensuring you never miss critical alerts. The solution is idempotent, so you can safely re-run it to remediate issues.


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
- [CloudWatch Logs - Deleting Account Policies](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CloudWatch-Logs-Subscription-Filter-Policies.html)
- [Managing Conformance Packs Across all Accounts in Your Organization](https://docs.aws.amazon.com/config/latest/developerguide/conformance-pack-organization-apis.html)
- [Managing the Configuration Recorder](https://docs.aws.amazon.com/config/latest/developerguide/stop-start-recorder.html)
- [Suspending or stopping GuardDuty](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_suspend-disable.html)
- [Suspending or disabling Amazon Macie](https://docs.aws.amazon.com/macie/latest/user/macie-suspend-disable.html)
- [Disable AWS Audit Manager](https://docs.aws.amazon.com/audit-manager/latest/userguide/console-settings.html#disable)
- [Disabling Amazon Inspector](https://docs.aws.amazon.com/inspector/latest/user/disabling-best-practices.html)
- [Deleting IAM Access Analyzer](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-getting-started.html#access-analyzer-delete)
- [Configuring block public access settings for your account](https://docs.aws.amazon.com/AmazonS3/latest/userguide/configuring-block-public-access-account.html)
- [EBS Encryption by default](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html#encryption-by-default)
- [EBS Snapshot Block Public Access](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/block-public-access-snapshots.html)
- [Modifying RDS Instance Public Accessibility](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_VPC.WorkingWithRDSInstanceinaVPC.html#USER_VPC.Hiding)