# Working with Amazon S3 – Secure File Sharing and Event Notifications (AWS Lab)

## Overview
In this lab, I created and configured an **Amazon S3 bucket** to securely share files with an **external user** from a media company.  
Additionally, I configured **event notifications** so that any change in the bucket contents automatically triggers an **email alert** using **Amazon SNS**.

This lab simulates a real-world scenario where third-party users require **restricted access** to upload and manage files, while administrators maintain **control, security, and visibility**.

---

## Architecture
The solution architecture consists of:

- An **Amazon S3 bucket** used as a shared image repository
- An **IAM group and user (mediacouser)** representing an external media company
- **Fine-grained IAM policies** restricting access to a specific S3 prefix
- An **Amazon SNS topic** for event notifications
- Email subscriptions to receive alerts when objects are created or deleted

<img width="1068" height="724" alt="image" src="https://github.com/user-attachments/assets/41fcbc34-7dd3-4318-91fd-c2a9629b5ce1" />

---

## Objectives
- Create and configure an S3 bucket using AWS CLI
- Grant controlled write access to an external IAM user
- Validate IAM permissions through real operations
- Configure S3 event notifications
- Send email alerts using Amazon SNS

---

## Services and Technologies Used
- Amazon S3
- AWS Identity and Access Management (IAM)
- Amazon SNS
- AWS CLI (`s3` and `s3api`)
- Amazon EC2 (CLI Host)
- JSON (IAM policies and notification configuration)

---

## Lab Implementation

### 1. Connecting to the CLI Host and Configuring AWS CLI
I connected to the **CLI Host EC2 instance** using EC2 Instance Connect and configured AWS CLI with the provided credentials.

To validate that AWS CLI was correctly configured, the following command was executed:

```bash
aws sts get-caller-identity
```

<img width="904" height="108" alt="image" src="https://github.com/user-attachments/assets/995316d3-27d5-428f-80c6-0beddbbafbb1" />

---

### 2. Creating and Initializing the S3 Bucket
Using AWS CLI, I created an S3 bucket with a unique name and uploaded initial images under the `images/` prefix.

```bash
aws s3 mb s3://cafe-pincheira --region us-west-2
aws s3 sync ~/initial-images/ s3://cafe-pincheira/images
aws s3 ls s3://cafe-pincheira/images/ --human-readable --summarize
```

<img width="925" height="104" alt="image" src="https://github.com/user-attachments/assets/da1ef0b7-f633-4a30-a01c-2f9dbd61ca14" />

<img width="863" height="116" alt="image" src="https://github.com/user-attachments/assets/2d5b7158-29e7-4652-9380-893ee046bec3" />

---

### 3. Reviewing IAM Group and User Permissions
I reviewed the **mediaco IAM group** and its associated policy, which grants object-level permissions only within the `images/` prefix, enforcing the **principle of least privilege**.

<details>
<summary>View IAM applied policy (mediaCoPolicy)</summary>

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Action": [
				"s3:ListAllMyBuckets",
				"s3:GetBucketLocation"
			],
			"Resource": [
				"arn:aws:s3:::*"
			],
			"Effect": "Allow",
			"Sid": "AllowGroupToSeeBucketListInTheConsole"
		},
		{
			"Action": [
				"s3:ListBucket"
			],
			"Resource": [
				"arn:aws:s3:::cafe-*",
				"arn:aws:s3:::cafe-*/*"
			],
			"Effect": "Allow",
			"Sid": "AllowRootLevelListingOfTheBucket"
		},
		{
			"Action": [
				"s3:PutObject",
				"s3:GetObject",
				"s3:GetObjectVersion",
				"s3:DeleteObject",
				"s3:DeleteObjectVersion"
			],
			"Resource": "arn:aws:s3:::cafe-*/images/*",
			"Effect": "Allow",
			"Sid": "AllowUserSpecificActionsOnlyInTheSpecificPrefix"
		}
	]
}
```
</details>

---

### 4. Testing External User Access (mediacouser)
I validated the allowed actions (upload, view, and delete objects) and confirmed that unauthorized actions, such as modifying bucket permissions or object visibility, were correctly blocked.

<img width="1772" height="656" alt="image" src="https://github.com/user-attachments/assets/e6b1ef01-d5af-41e6-85d1-c1db214fddf1" />

<img width="1773" height="762" alt="image" src="https://github.com/user-attachments/assets/56d463c0-be7d-49b4-9709-84d95b58c70b" />

<img width="817" height="760" alt="image" src="https://github.com/user-attachments/assets/a3ae9995-405f-409e-9ccf-c7ded6d4707d" />

---

## Event Notifications with Amazon SNS

### 5. Creating and Configuring the SNS Topic
An SNS topic named `s3NotificationTopic` was created, and an email subscription was confirmed to receive notifications.

<img width="1914" height="582" alt="image" src="https://github.com/user-attachments/assets/de186382-22e5-44db-90bc-d45a61d65f2c" />

---

### 6. Configuring S3 Event Notifications
An event notification configuration was applied so that Amazon S3 publishes events when objects are created or deleted under the `images/` prefix.

<img width="1458" height="375" alt="image" src="https://github.com/user-attachments/assets/d1346e0e-2a60-4deb-9f95-92d3b96b3a2b" />

---

### 7. Testing Event Notifications
Object upload and deletion actions triggered email notifications, while read-only operations did not, confirming the correct behavior of the configuration.

<img width="654" height="691" alt="image" src="https://github.com/user-attachments/assets/9391dc2e-23ca-43c3-b392-f7927bff7131" />

<img width="658" height="671" alt="image" src="https://github.com/user-attachments/assets/9b2985e8-0bb5-4b6e-95e6-53f8e550dc7e" />

---

### CLI Validation as External User
Additional validation was performed using AWS CLI as **mediacouser**, including:

- Credential verification using STS

<img width="509" height="108" alt="image" src="https://github.com/user-attachments/assets/bc047f3a-ff61-4b2f-abe2-1b4d96948f2a" />

- PUT and GET object operations

<img width="1323" height="258" alt="image" src="https://github.com/user-attachments/assets/e04cbcb3-946d-4e8e-98c4-298ab4c6e565" />

<img width="978" height="174" alt="image" src="https://github.com/user-attachments/assets/e2c54f09-bf9f-4da9-82fc-94d73d2aaa5d" />

- DELETE object operations

<img width="861" height="73" alt="image" src="https://github.com/user-attachments/assets/71efa94a-e67e-4b54-a823-29846cb3059a" />

- Attempted unauthorized permission changes (blocked as expected)

<img width="1902" height="72" alt="image" src="https://github.com/user-attachments/assets/38111691-02d0-42ac-8db2-843e9ad3f9fd" />

---

## Results
- Secure S3 bucket configured for external collaboration
- IAM permissions correctly scoped by object prefix
- Unauthorized actions successfully blocked
- Real-time email notifications triggered by file changes

---

## Key Learnings
- Prefix-based IAM policies are essential for secure S3 sharing
- External users must not be granted bucket-level permissions
- S3 event notifications provide a simple and effective auditing mechanism
- Amazon SNS enables real-time operational alerts

---

## What I Would Do Differently in Production
- Use **Infrastructure as Code** (CloudFormation or Terraform)
- Replace IAM users with **federated access and IAM roles**
- Enable **S3 Server Access Logging or CloudTrail data events**
- Encrypt objects using **SSE-KMS**
- Integrate notifications with centralized monitoring or incident management tools

---

## Conclusion
This lab demonstrates hands-on experience in **secure file sharing with Amazon S3**, **IAM access control**, and **event-driven notifications using Amazon SNS**, aligned with real-world cloud security and operational best practices.
