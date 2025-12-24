# Security Auditing and Incident Analysis with AWS CloudTrail (AWS Lab)

## Overview
In this lab, **AWS CloudTrail** is implemented to audit actions performed within an AWS account in order to **investigate a security incident** that affected the Café website.

The lab simulates a real-world scenario where an attacker modifies security rules, gains operating system access, and alters web server files. Through log analysis, the responsible actor is identified and corrective actions are applied.

---

## Architecture
The lab architecture includes:

- **EC2 Instance – Café Web Server**
- **AWS CloudTrail Trail** with logs stored in Amazon S3
- **S3 Bucket (monitoring3333)** for audit logs
- **Amazon Athena** for SQL-based log analysis
- **IAM Users** (legitimate and malicious)
- **Security Groups** protecting the web server

<img width="814" height="437" alt="image" src="https://github.com/user-attachments/assets/748a9aaf-dbec-49a1-affc-67d8a0164eec" />

---

## Objectives
- Configure an AWS CloudTrail trail
- Detect unauthorized changes to Security Groups
- Analyze logs using Linux tools, AWS CLI, and Amazon Athena
- Identify the user responsible for the incident
- Remediate vulnerabilities at both AWS and OS levels

---

## Services Used
- Amazon EC2
- AWS CloudTrail
- Amazon S3
- Amazon Athena
- AWS IAM
- Amazon VPC (Security Groups)

---

## Lab Implementation

### 1. Initial Website Observation
The Café website was accessed to confirm that its content was legitimate prior to the incident.

<img width="493" height="877" alt="image" src="https://github.com/user-attachments/assets/f0c39a16-3593-4cab-af1a-946c076f793f" />

---

### 2. CloudTrail Trail Creation
A CloudTrail trail named `monitor` was created to record account activity and store logs in a dedicated S3 bucket.

<img width="1666" height="304" alt="image" src="https://github.com/user-attachments/assets/1f58f4c2-bef5-4bdd-b140-abe35481629e" />

---

### 3. Detection of Website Compromise
After enabling CloudTrail, the website content was maliciously modified.

<img width="531" height="934" alt="image" src="https://github.com/user-attachments/assets/2238175d-708c-461a-9d20-892365323d83" />

---

### 4. Security Group Analysis
A suspicious inbound rule allowing SSH access from `0.0.0.0/0` was identified.

<img width="1486" height="157" alt="image" src="https://github.com/user-attachments/assets/e0874fe0-88f7-4d6c-9a12-42b7a51b3f61" />

---

### 5. Log Download and Analysis Using grep
CloudTrail logs were downloaded from S3, decompressed, and analyzed using `grep` to identify relevant events.

<img width="1903" height="460" alt="image" src="https://github.com/user-attachments/assets/c2ab717e-5193-4aca-af98-689414875909" />

<img width="1039" height="995" alt="image" src="https://github.com/user-attachments/assets/82c05d99-b502-484b-8a31-a587279fe0de" />

<img width="988" height="1003" alt="image" src="https://github.com/user-attachments/assets/3883349b-06a4-4844-9858-ee02e031f39c" />

---

### 6. Log Analysis Using AWS CLI (lookup-events)
The `aws cloudtrail lookup-events` command was used to filter Security Group–related events.

<img width="1900" height="1006" alt="image" src="https://github.com/user-attachments/assets/d2de02a9-c234-4e11-a2fa-2c1b98497e0b" />

---

### 7. Advanced Analysis with Amazon Athena
An Athena table was created to query CloudTrail logs using SQL in order to identify the responsible user.

<img width="330" height="516" alt="image" src="https://github.com/user-attachments/assets/c5547a32-7684-449c-b4fe-729e0ca6b8b3" />

<img width="1869" height="822" alt="image" src="https://github.com/user-attachments/assets/21f41d08-84a1-4c15-9cdf-3eb451b73115" />

---

### 8. Attacker Identification
Using SQL queries, the following information was identified:
- Responsible IAM user
- Exact time of the change
- Source IP address
- Access method (CLI or console)

<img width="1467" height="714" alt="image" src="https://github.com/user-attachments/assets/fdd25f0c-a5b7-44d7-a0d1-51b7d2608821" />

**Findings:**
- **User:** chaos  
- **Time:** 2025-12-24T00:02:53Z  
- **IP Address:** 44.243.94.245  
- **Event Name:** AuthorizeSecurityGroupIngress  
- **User Agent:** aws-cli/1.18.147 Python/2.7.18 Linux/4.14.355-280.710.amzn2.x86_64 botocore/1.18.6  
- **Request Parameters:** SSH (port 22) opened to 0.0.0.0/0  
- **Access Method:** Programmatic (CLI/SDK)

---

### 9. OS-Level Remediation
An unauthorized operating system user (`chaos-user`) was detected and removed from the EC2 instance.

<img width="986" height="917" alt="image" src="https://github.com/user-attachments/assets/fb5e6d8a-070a-457f-9ddb-0ff29b119841" />

<img width="534" height="117" alt="image" src="https://github.com/user-attachments/assets/e27da953-f8cd-4d57-8baf-82b2e715a24a" />

---

### 10. SSH Security Hardening
Password-based SSH authentication was disabled and the Security Group rules were corrected.

<img width="622" height="90" alt="image" src="https://github.com/user-attachments/assets/c7ec16fc-3fc1-45a8-aea8-16a20780a42c" />

<img width="1650" height="267" alt="image" src="https://github.com/user-attachments/assets/9bc1dbb8-d688-4118-a55c-17b8c948a90b" />

---

### 11. Website Restoration
The original website image was restored from a local backup.

<img width="803" height="425" alt="image" src="https://github.com/user-attachments/assets/b807c230-ffdc-42bd-a28e-495a2abd6e89" />

<img width="491" height="870" alt="image" src="https://github.com/user-attachments/assets/01e390f8-61c6-4299-b0a1-e1e8737f6f32" />

---

### 12. Removal of the Malicious IAM User
The IAM user responsible for the incident was deleted from the AWS account.

<img width="1607" height="302" alt="image" src="https://github.com/user-attachments/assets/eb3e6854-6a00-4a29-b160-d640327285a4" />

---

## Results
- Security incident identified and documented
- Attacker traced through audit logs
- Infrastructure and operating system secured
- Website successfully restored

---

## What I Would Do Differently in Production
- Enable CloudTrail at the organization level
- Centralize logs in a dedicated security account
- Integrate with Amazon GuardDuty and AWS Security Hub
- Enforce strict least-privilege IAM policies
- Automate incident response using AWS Lambda

---

## Conclusion
This lab demonstrates how **AWS CloudTrail** is a critical service for **auditing, forensic investigation, and incident response**, accurately replicating a real-world cloud security scenario.
