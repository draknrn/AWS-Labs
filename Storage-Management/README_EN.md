# Storage Management – AWS Lab

## Overview
In this lab, I worked with **Amazon EBS and Amazon S3** to learn how to **automate backup management** and **securely synchronize files**, using **AWS CLI, Python, and core AWS services**.

The main objectives were:
- Create EBS volume snapshots
- Automate snapshot creation and cleanup
- Retain only the most recent backups
- Synchronize files from an EC2 instance to Amazon S3
- Recover deleted files using S3 versioning

This lab simulates a real-world cloud infrastructure management scenario.

---

## Architecture
The environment consisted of:
- A **VPC** with a public subnet
- Two **EC2 instances**:
  - **Command Host**: used to manage AWS resources via AWS CLI
  - **Processor**: instance with an attached EBS volume and sample files
- An **EBS volume** attached to the Processor instance
- An **Amazon S3 bucket**, initially without versioning, later enabled via AWS CLI for file recovery

<img width="1538" height="970" alt="image" src="https://github.com/user-attachments/assets/027c3651-e700-4177-b8bf-09ede5084de2" />

---

## Objectives
- Create manual and automated EBS snapshots
- Schedule tasks using `cron`
- Automate snapshot retention with Python
- Sync files from EC2 to S3
- Recover deleted files using S3 versioning

---

## Services & Technologies Used
- Amazon EC2
- Amazon EBS
- Amazon S3
- AWS CLI
- IAM Roles
- Python 3
- Linux Cron

---

## Lab Implementation

### 1. S3 Bucket Creation
An S3 bucket was created to serve as the destination for file synchronization from the EC2 instance.

<img width="1373" height="649" alt="image" src="https://github.com/user-attachments/assets/95d8e3cf-2c40-4d55-ad41-2b85dbd05c05" />

---

### 2. IAM Role Assignment
An **IAM Role** was attached to the **Processor** instance to allow access to Amazon S3 and other AWS services without hardcoding credentials.

<img width="1441" height="518" alt="image" src="https://github.com/user-attachments/assets/654ef631-df0b-4068-871e-d8c338be5cac" />

This follows AWS security best practices.

---

### 3. Initial EBS Snapshot Creation
From the **Command Host** instance, AWS CLI was used to:
- Identify the EBS volume attached to the Processor instance
- Stop the instance to ensure snapshot consistency
- Create the initial snapshot
- Restart the instance

<img width="761" height="224" alt="image" src="https://github.com/user-attachments/assets/cf10af3d-40da-47dc-813d-9998537ff0a0" />

---

### 4. Snapshot Automation with Cron
A scheduled `cron` job was configured to create EBS snapshots every minute (for demonstration purposes only).

<img width="1204" height="599" alt="image" src="https://github.com/user-attachments/assets/aeeefa77-b5b6-4722-9b62-f7bbe3de41e3" />

<img width="1301" height="137" alt="image" src="https://github.com/user-attachments/assets/84355dd7-6bce-4d1c-a1ff-3dad1ad5320a" />

This simulates an automated backup system.

---

### 5. Automated Snapshot Cleanup (Python)
A Python script was executed to:
- List all snapshots for the EBS volume
- Sort them by creation date
- Delete all but the **two most recent snapshots**

<img width="1300" height="171" alt="image" src="https://github.com/user-attachments/assets/55676a2b-d452-45c3-9985-42c0d1803ffb" />

This helps control costs and maintain relevant backups only.

---

## Challenge: Amazon S3 File Synchronization

### 6. Enabling S3 Versioning
Versioning was enabled on the S3 bucket to allow recovery of deleted or overwritten files.

<img width="1109" height="86" alt="image" src="https://github.com/user-attachments/assets/13b1f015-ca55-498b-a8bf-d23b888f9990" />

---

### 7. File Synchronization Using `aws s3 sync`
A local directory was synchronized from the EC2 instance to the S3 bucket using:

```bash
aws s3 sync files s3://my-bucket/files/
```

<img width="536" height="52" alt="image" src="https://github.com/user-attachments/assets/e28e0181-c4b5-44ba-bc84-c886d4e95f91" />

---

### 8. File Deletion and Recovery
The following workflow was performed:
- A file was deleted locally
- The S3 bucket was synchronized using `--delete`
- The file was recovered from a previous S3 version
- The restored file was re-synced to the bucket

<img width="706" height="241" alt="image" src="https://github.com/user-attachments/assets/82336c58-a366-4b07-997d-adaaf9f9972b" />

<img width="942" height="578" alt="image" src="https://github.com/user-attachments/assets/e989016f-c7ac-4f5f-ab42-bded11194872" />

<img width="1384" height="256" alt="image" src="https://github.com/user-attachments/assets/bec5ab36-86e2-429a-b390-04b81a5be462" />

<img width="626" height="102" alt="image" src="https://github.com/user-attachments/assets/3750208e-852d-409a-9995-dcb65b3a1147" />

This demonstrates recovery from human error, a common real-world scenario.

---

## Results
- Automated EBS snapshots
- Automatic backup retention
- Secure file synchronization with Amazon S3
- Recovery of deleted files using versioning
- Proper use of IAM Roles and AWS CLI

---

## Key Learnings
- Automation reduces errors and operational costs
- EBS snapshots are essential for EC2 backups
- S3 versioning provides strong data protection
- AWS CLI enables infrastructure management without the console
- Python is a powerful tool for cloud automation

---

## What I Would Do Differently in Production
- Use **Infrastructure as Code** (CloudFormation or Terraform)
- Apply **least-privilege IAM policies**
- Replace cron-based snapshots with **AWS Data Lifecycle Manager**
- Add **CloudWatch monitoring and alerts**
- Implement **S3 lifecycle policies** for cost optimization
- Use **multi-AZ and managed services** for higher availability

---

## Conclusion
This lab demonstrates hands-on experience in **AWS storage management**, backup automation, and data recovery, aligned with real-world cloud operations and best practices.
