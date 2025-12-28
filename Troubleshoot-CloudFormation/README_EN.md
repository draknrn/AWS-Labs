# AWS CloudFormation Troubleshooting – Infrastructure as Code (IaC)

## Overview
This lab focuses on **troubleshooting AWS CloudFormation deployments** and reinforces key **Infrastructure as Code (IaC)** concepts.  
The activity simulates real-world scenarios where CloudFormation stacks fail during creation, update, or deletion, requiring a systematic analysis process using **AWS CLI**, **JMESPath**, and **EC2 instance log analysis**.

The lab is presented as a proof of concept (POC) for the **Café** application, enabling an understanding of how CloudFormation facilitates reliable deployments, drift detection, and large-scale AWS infrastructure management.

---

## Architecture

<img width="921" height="410" alt="image" src="https://github.com/user-attachments/assets/065a7cef-b5b3-46e0-af0c-39b19028aaae" />

The architecture used throughout the lab includes:

- Custom Amazon VPC (**VPC2**)
- Public subnet
- Amazon EC2 instances:
  - **CLI Host** (administration and troubleshooting)
  - **Web Server** (created by CloudFormation)
- Amazon S3 bucket (created by the stack)
- AWS CloudFormation stack and associated resources
- Security Groups

This architecture is used in Tasks 1, 2, and 3 to demonstrate deployments, drift detection, and challenges when deleting stacks.

---

## Objectives
- Practice querying JSON data using **JMESPath**.
- Diagnose failures in AWS CloudFormation stack deployments.
- Analyze **cloud-init** logs on EC2 instances to identify root causes.
- Detect out-of-sync configurations (drift).
- Resolve failed stack deletions while preserving critical resources.
- Reinforce **Infrastructure as Code** best practices.

---

## Services Used
- AWS CloudFormation
- Amazon EC2
- Amazon VPC
- Amazon S3
- AWS CLI
- Linux (Amazon Linux)
- JMESPath

---
## Task 1: Troubleshooting CloudFormation Stack Creation

### 1.1 AWS CLI Configuration
The AWS CLI was configured using temporary lab credentials, and the region was detected via the EC2 metadata service.

<img width="971" height="215" alt="image" src="https://github.com/user-attachments/assets/02f8a1ff-6b02-4a5a-a976-bba0c823143b" />

### 1.2 Initial Stack Creation Failure
The first stack creation attempt failed and entered the **ROLLBACK_COMPLETE** state.

<img width="1229" height="592" alt="image" src="https://github.com/user-attachments/assets/5c8d1e0a-d825-4b25-85e4-a9999bd000ab" />

<img width="1899" height="325" alt="image" src="https://github.com/user-attachments/assets/08af2e5f-e8fe-4785-bfce-659bf367bec0" />

Event analysis revealed:
- Timeout on a **WaitCondition**
- Failure in the EC2 instance userdata script

### 1.3 Rollback Prevention for Analysis
The stack was recreated using the `--on-failure DO_NOTHING` option, allowing inspection of the EC2 instance logs.

<img width="1031" height="176" alt="image" src="https://github.com/user-attachments/assets/88afdabd-3d43-4b43-9637-e4f3ea1698a1" />

<img width="1899" height="197" alt="image" src="https://github.com/user-attachments/assets/73480fd6-5a73-43a2-997c-2acacced6c75" />

<img width="1907" height="195" alt="image" src="https://github.com/user-attachments/assets/a9d33798-10c9-48b9-b4ec-bb0bc2b06514" />

Log analysis confirmed:
- Use of a non-existent package (`http`).
- Immediate script termination due to the `-e` flag.
- The WaitCondition never received a success signal.

### 1.4 Template Fix and Successful Deployment
The CloudFormation template was corrected by replacing `http` with `httpd`.

<img width="804" height="182" alt="image" src="https://github.com/user-attachments/assets/295008bf-5e7a-4d8e-b50e-533fe64834fd" />

<img width="1340" height="344" alt="image" src="https://github.com/user-attachments/assets/d1e57fab-6999-4ff3-b16e-f55e6e0b77d1" />

<img width="546" height="105" alt="image" src="https://github.com/user-attachments/assets/42e9561f-587f-421c-9e49-2ca74ecda41d" />

After recreating the stack:
- All resources were created successfully.
- The stack reached the **CREATE_COMPLETE** state.
- The web server responded correctly.

---

## Task 2: Manual Changes and Drift Detection

### 2.1 Manual Modifications
A Security Group was manually modified to restrict SSH access to a specific IP address.

<img width="1653" height="275" alt="image" src="https://github.com/user-attachments/assets/9971eb7e-c40a-4b13-91b0-4a7619d50059" />

Additionally, an object was uploaded to the S3 bucket created by the stack.

<img width="565" height="249" alt="image" src="https://github.com/user-attachments/assets/b1088fe2-7385-4956-b175-8f050cf4e4e2" />

### 2.2 Drift Detection
Drift detection was executed, resulting in:
- Security Group in **MODIFIED** state
- S3 bucket in **IN_SYNC** state

<img width="1043" height="268" alt="image" src="https://github.com/user-attachments/assets/89e131f7-6c15-46db-979b-1c1ddcde261f" />

<img width="1903" height="518" alt="image" src="https://github.com/user-attachments/assets/d2436125-495f-4ca2-a85a-994615f037b5" />

<img width="946" height="381" alt="image" src="https://github.com/user-attachments/assets/9a2634c3-4926-4302-8464-7452ebdeb933" />

This demonstrates that CloudFormation detects configuration changes but does not consider bucket contents as drift.

---

## Task 3: Stack Deletion Failure
<img width="1231" height="280" alt="image" src="https://github.com/user-attachments/assets/cc32564f-46a8-4861-9866-8302dde595fd" />

<img width="601" height="306" alt="image" src="https://github.com/user-attachments/assets/ad7bf03e-c594-4c83-aa16-c4539b0025f3" />

When attempting to delete the stack, it entered the **DELETE_FAILED** state.

Cause:
- CloudFormation does not delete S3 buckets that contain objects.

---

## Challenge: Preserve the S3 Bucket and Delete the Stack
<img width="1050" height="326" alt="image" src="https://github.com/user-attachments/assets/cec40414-23aa-416c-acab-61801d2a0bdd" />

Using AWS CLI:
- The bucket LogicalResourceId was identified.
- The stack was deleted using the `--retain-resources` option.

Result:
- Successful stack deletion.
- Preservation of the S3 bucket and its contents.

---

## Results
- Identification and resolution of CloudFormation deployment failures.
- EC2 log analysis to determine root causes.
- Effective use of JMESPath with AWS CLI.
- Drift detection and analysis.
- Stack deletion while preserving critical resources.
- Application of real-world IaC troubleshooting practices.

---

## What I Would Do Differently in Production
- Template validation and linting (cfn-lint).
- Use of Change Sets before applying changes.
- Stack policies to protect critical resources.
- Centralized logging and monitoring.
- Strict controls to prevent changes outside IaC.

---

## Conclusion
This lab demonstrates the importance of **methodical troubleshooting** when working with **Infrastructure as Code**.  
The combined use of AWS CloudFormation, AWS CLI, JMESPath, and log analysis enables the construction of **reliable, auditable, and production-ready environments**, aligned with AWS best practices.
