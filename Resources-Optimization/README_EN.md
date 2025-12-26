# Resource Utilization Optimization – Café Web Application (AWS Lab)

## Overview
In this lab, the AWS resources used by the **Café** web application are optimized with the goal of **reducing operational costs** without impacting functionality.

The main optimizations are:
- Removal of the decommissioned local database.
- Downsizing the EC2 instance type from **t3.small** to **t3.micro**.
- Cost savings estimation using **AWS Pricing Calculator**.

---

## Architecture

The Café application topology includes:
- Amazon VPC
- EC2 Instance **CafeInstance**
- Amazon RDS (MariaDB)
- Amazon EBS
- CLI Host

<img width="1232" height="574" alt="image" src="https://github.com/user-attachments/assets/fcc35b33-356b-41f6-884f-6f7a4f426bf3" />

---

## Objectives
- Optimize an Amazon EC2 instance to reduce costs.
- Remove unnecessary components after a migration.
- Change the EC2 instance type.
- Estimate costs before and after optimization.
- Calculate the projected monthly savings.

---

## Services Used
- Amazon EC2
- Amazon EBS
- Amazon RDS (MariaDB)
- AWS CLI
- AWS Pricing Calculator

---

## Task 1: Optimize the EC2 Instance

### 1.1 AWS CLI Configuration
The AWS CLI was configured using the lab credentials.

```bash
aws configure
```

<img width="496" height="115" alt="image" src="https://github.com/user-attachments/assets/3a305d08-6026-436c-ad95-e35967c87fb2" />

---

### 1.2 Local Database Removal

The local MariaDB database was stopped and uninstalled:

```bash
sudo systemctl stop mariadb
sudo yum -y remove mariadb-server
```

<img width="1096" height="994" alt="image" src="https://github.com/user-attachments/assets/cb45e4a1-42ce-463a-996c-fc3b69cf29ab" />

---

### 1.3 EC2 Instance Identification

From the **CLI Host**, the Instance ID was retrieved:

```bash
aws ec2 describe-instances --filters "Name=tag:Name,Values=CafeInstance" --query "Reservations[*].Instances[*].InstanceId"
```

<img width="477" height="153" alt="image" src="https://github.com/user-attachments/assets/2ea7fac6-cde4-44bc-b534-2607a4051047" />

---

### 1.4 Instance Resizing

The instance was stopped, the type was modified, and then restarted:

```bash
aws ec2 stop-instances --instance-ids <instance-id>

aws ec2 modify-instance-attribute --instance-id <instance-id> --instance-type "{\"Value\": \"t3.micro\"}"

aws ec2 start-instances --instance-ids <instance-id>
```

<img width="733" height="366" alt="image" src="https://github.com/user-attachments/assets/ea8219c6-4438-4075-b1fc-f5253100f870" />
<img width="1574" height="150" alt="image" src="https://github.com/user-attachments/assets/2e49af73-4175-4895-bfd5-fc1872e73280" />

---

### 1.5 Functionality Verification

The instance status was verified and the website was accessed:

```bash
aws ec2 describe-instances --instance-ids <instance-id> --query "Reservations[*].Instances[*].[InstanceType,PublicDnsName,PublicIpAddress,State.Name]"
```

<img width="1893" height="172" alt="image" src="https://github.com/user-attachments/assets/4189adf9-e825-4758-9221-33a9ec06de93" />

Tested URL:
```
http://<Public-DNS>/cafe
```

<img width="1084" height="961" alt="image" src="https://github.com/user-attachments/assets/c7a2b303-2485-439f-806e-d454800158eb" />

---

## Task 2: Cost Estimation with AWS Pricing Calculator

### 2.1 Costs Before Optimization

Configuration:
- EC2: t3.small
- EBS: 40 GB gp2
- RDS: db.t3.micro (20 GB)

<img width="1864" height="751" alt="image" src="https://github.com/user-attachments/assets/170523d5-b898-47b0-b0d2-209efea1bfbd" />

**Estimated monthly cost:** $74.04

---

### 2.2 Costs After Optimization

Changes:
- EC2: t3.micro
- EBS: 20 GB gp2

<img width="1863" height="756" alt="image" src="https://github.com/user-attachments/assets/c7114245-bd14-476d-b40b-137d907a2ae4" />

**Estimated monthly cost:** $64.45

---

### 2.3 Projected Savings

| State | Monthly Cost |
|------|---------------|
| Before | $74.04 |
| After | $64.45 |
| **Savings** | **$9.59** |

---

## Results
- Successful instance downsizing.
- Removal of unnecessary resources.
- Demonstrable monthly savings.
- Application of cost optimization best practices.

---

## What I Would Do Differently in Production
- Automate right-sizing with **AWS Compute Optimizer**
- Use **Elastic IP** to avoid IP changes
- Implement **Infrastructure as Code**
- Monitor costs with **AWS Budgets**
- Apply automated shutdown policies

---

## Conclusion
This lab demonstrates how small technical optimizations can generate a **positive financial impact**, reinforcing the importance of **cost optimization** as a pillar of the AWS Well-Architected Framework.
