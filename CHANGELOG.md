# Changelog

All notable changes to this project will be documented in this file.

## [0.0.2] - 2026

### 🎉 Major Modernization and Feature Expansion

This release represents a comprehensive modernization and expansion of the AWS Security StepFunction Integration project, bringing it up to date with current AWS security best practices and adding numerous new features.

### Added - Security Hub V2 with CSPM
- **Security Hub V2 with CSPM**: Unified security platform with Cloud Security Posture Management
  - Security Hub V2: Prioritizes critical security issues with real-time risk analytics, automated correlation, and enriched context
  - CSPM: Always enabled (AWS recommendation) for automated security best practice checks and compliance monitoring
  - Uses OCSF (Open Cybersecurity Schema Framework) format for findings
- **Security Hub Standards** (optional):
  - AWS Foundational Security Best Practices (parameter `pSecHubStandardAwsFun`, default: true)
  - CIS AWS Foundations Benchmark v3.0 (parameter `pSecHubStandardCIS`, default: true)
  - PCI DSS v4.0 (parameter `pSecHubStandardPCI`, default: false)
  - NIST 800-53 Rev. 5 (parameter `pSecHubStandardNIST`, default: false)
- Security Hub state machine completely refactored for scalability and idempotency
- Security Hub standards are enabled sequentially with exponential backoff retry logic

### Added - Account Level Settings Expansion
- **Account Level Settings**: Five new security settings (now 7 total, running in parallel):
  - IMDSv2 requirement for EC2 instances (parameter `pImdsv2`)
  - RDS encryption enforcement (parameter `pRdsEncrypt`)
  - EC2 Serial Console disable (parameter `pEc2SerialConsole`)
  - EBS snapshot public access blocking (parameter `pEbsSnapshotBlockPublic`)
  - RDS/Aurora public accessibility disable (parameter `pRdsPublicAccess`)
- Account Level Settings state machine expanded from 2 to 7 parallel branches
- IAM AccountLevelSettings policy expanded with new permissions:
  - `ec2:ModifyInstanceMetadataDefaults`, `ec2:GetInstanceMetadataDefaults`
  - `rds:ModifyDBCluster`, `rds:ModifyDBInstance`, `rds:DescribeDBClusters`, `rds:DescribeDBInstances`
  - `ec2:DisableSerialConsoleAccess`, `ec2:EnableSerialConsoleAccess`, `ec2:GetSerialConsoleAccessStatus`
  - `ec2:EnableSnapshotBlockPublicAccess`, `ec2:DisableSnapshotBlockPublicAccess`, `ec2:GetSnapshotBlockPublicAccessState`

### Added - IAM Security Settings
- **IAM Security Settings**: New dedicated state machine (SSI-IAMSettings)
  - IAM Password Policy configuration (parameter `pIamPasswordPolicy`)
    - 14+ character minimum, complexity requirements
    - 90-day expiration, 24 password history
  - IAM Access Analyzer enablement (parameter `pIamAccessAnalyzer`)
    - Account-level analyzer for external access detection
- IAM IAMSettings policy with permissions:
  - `iam:UpdateAccountPasswordPolicy`, `iam:GetAccountPasswordPolicy`
  - `access-analyzer:CreateAnalyzer`, `access-analyzer:GetAnalyzer`, `access-analyzer:ListAnalyzers`, `access-analyzer:TagResource`

### Added - Findings Capture and Logging
- **Security Hub findings** captured via EventBridge and sent to CloudWatch Logs
  - EventBridge rule filters CRITICAL, HIGH, and MEDIUM severity findings
  - Dedicated log group `/aws/securityhub/findings` with 90-day retention
  - Findings flow: EventBridge → CloudWatch Logs → Firehose → S3
- **Inspector findings** captured via EventBridge
  - Dedicated log group `/aws/inspector/findings` with 90-day retention
  - All Inspector2 findings automatically captured and archived
- **IAM Access Analyzer findings** captured via EventBridge
  - Dedicated log group `/aws/access-analyzer/findings` with 90-day retention
  - External access findings automatically logged for audit

### Added - CloudWatch Logs Aggregation
- Kinesis Firehose delivery stream (SSI-CloudWatchLogs-to-S3) for centralized log aggregation
- Server-side encryption enabled for Firehose stream using customer-managed KMS key
- CloudWatch error logging enabled for Firehose with dedicated log group `/aws/kinesisfirehose/SSI-CloudWatchLogs-to-S3`
- Firehose IAM role with CloudWatch Logs and KMS permissions
- Logs compressed with GZIP and stored in `cloudwatch-logs/` prefix
- Error logs stored in `cloudwatch-logs-errors/` prefix
- 5-minute buffer interval and 5 MB buffer size for S3 delivery
- EventBridge resource policies for all log groups to allow event delivery

### Added - CloudWatch Logs Protection
- **CloudWatch Logs**: New dedicated state machine (SSI-CloudWatchLogs)
  - CloudWatch Logs subscription filter policy (parameter `pCloudWatchSubscriptionFilter`)
  - Prevents log data exfiltration to external accounts
  - Restricts log subscription destinations to same account
- IAM CloudWatchSettings policy with permissions:
  - `logs:PutAccountPolicy`, `logs:DescribeAccountPolicies`, `logs:DeleteAccountPolicy`

### Added - AWS Config Enhancements
- **AWS Config**: Additional Conformance Pack options:
  - None (skip conformance pack deployment)
  - Operational-Best-Practices-for-PCI-DSS (PCI DSS v4.0)
  - Operational-Best-Practices-for-NIST-800-53 (NIST 800-53 Rev. 5)
  - Operational-Best-Practices-for-HIPAA-Security (HIPAA Security Rule)
- Config state machine now supports skipping conformance pack deployment when "None" is selected

### Added - CloudTrail Data Events
- Optional CloudTrail data events logging (parameter `pCloudTrailDataEvents`, default: false)
- Captures S3 object-level operations (GetObject, PutObject, DeleteObject) across all buckets
- Captures Lambda function invocations across all functions in account
- Event selectors configured via Step Functions state machine
- IAM permission added: `cloudtrail:PutEventSelectors`

### Added - SNS Dead Letter Queue
- SQS Dead Letter Queue (SSI-Error-Notifications-DLQ) for failed SNS notifications
- 14-day message retention for failed notification recovery
- KMS encryption for messages in DLQ
- Queue policy allowing SNS to send failed messages
- SNS topic now includes RedrivePolicy pointing to DLQ
- SNS topic KMS encryption enabled

### Added - Infrastructure Improvements
- S3 Bucket Keys for cost optimization (reduces KMS costs by ~99%)
- S3 lifecycle policies for automatic storage tiering
- Comprehensive resource tagging (Environment, ManagedBy)
- KMS encryption for CloudWatch Logs
- KMS encryption for SNS topics
- Enhanced error handling in Step Functions
- Condition keys for IAM policy restrictions
- S3 Lifecycle Configuration:
  - Transition to Standard-IA after 90 days
  - Transition to Glacier Instant Retrieval after 180 days
  - Expire old versions after 90 days
- Comprehensive Outputs section with user-facing resources only (removed child state machines and KMS keys)

### Changed - Breaking Changes
- **BREAKING**: Updated CIS Benchmark from v1.2.0 to v3.0.0
- **BREAKING**: Updated PCI DSS from v3.2.1 to v4.0.0
- **BREAKING**: Removed deprecated S3 `AccessControl` property
- Replaced S3 `AccessControl` with `OwnershipControls` and `BucketOwnerEnforced`

### Changed - IAM and Security Improvements
- Improved IAM policies with least privilege principles
- Replaced wildcard resources with specific ARNs where possible
- Enhanced S3 bucket policy with encryption enforcement
- Added `aws:SourceAccount` conditions to all service principals
- Added `iam:PassedToService` condition for PassRole permissions
- Added `kms:ViaService` conditions for KMS permissions
- Scoped service-linked role creation to specific services
- Enhanced bucket policy with unencrypted upload denial
- Improved IAM role trust policies with account conditions

### Changed - Configuration Updates
- Updated CloudWatch Logs retention from 30 to 90 days
- Changed KMS MultiRegion from true to false (best practice for single region)
- Updated Config Conformance Pack names to latest versions
- Updated EventBridge terminology (from CloudWatch Events)
- Account Level Settings state machine renamed parallel state from "Run EBS and S3 settings in parallel" to "Run account settings in parallel"

### Changed - SNS and Email Handling
- SNS topic now always created, but subscription only added when email is provided
- Email parameter now validates email format with regex pattern
- Added Conditions section with `HasNotificationEmail` condition

### Fixed - GuardDuty Publishing Destination
- GuardDuty state machine now checks if publishing destination exists for existing detectors
- Creates publishing destination for existing detectors if missing
- Added `guardduty:ListPublishingDestinations` permission to Step Functions worker role
- Idempotent with error handling

### Fixed - Security Hub Rate Limiting
- Added exponential backoff retry to PCI DSS and NIST standard enablement
- Retry configuration: 2s, 4s, 8s, 16s, 32s, 64s (6 attempts, backoff rate 2.0)
- Handles `TooManyRequestsException` and `SecurityHubException`

### Fixed - Firehose KMS Permissions
- Separate KMS policy statements for Firehose stream encryption vs S3 encryption
- Added CloudWatch Logs permissions to Firehose role (logs:PutLogEvents, logs:CreateLogStream)

### Security Enhancements
- IMDSv2 enforcement prevents SSRF attacks on EC2 instances
- Serial Console disable reduces attack surface for EC2 instances
- RDS encryption ensures data-at-rest protection for databases
- EBS snapshot public access blocking prevents accidental data exposure
- RDS public accessibility disable prevents database exposure to the internet
- IAM password policy enforces strong passwords (14+ chars, complexity, 90-day expiration, 24 password history)
- IAM Access Analyzer identifies resources shared with external entities
- CloudWatch subscription filter policy prevents log data exfiltration to external accounts

---

## [0.0.1] - 2022

### Initial Release

### Added
- AWS Security Hub integration with basic standards
- CloudTrail configuration for audit logging
- AWS Config with conformance packs
- GuardDuty enablement for threat detection
- Macie enablement for data discovery
- Audit Manager integration for compliance
- Inspector v2 enablement for vulnerability scanning
- Account-level security settings:
  - S3 Block Public Access
  - EBS encryption by default
- Step Functions orchestration for automated deployment
- SNS error notifications
- Centralized logging bucket with KMS encryption
- Basic IAM roles and policies
- CloudWatch Logs for state machine execution logging
