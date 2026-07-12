<div align="center">
  <img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" width="300" />

  # 🚀 AWS VPC Networks: End-to-End Cloud Networking Series

  ![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
  ![Amazon VPC](https://img.shields.io/badge/Amazon%20VPC-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)
  ![Amazon EC2](https://img.shields.io/badge/EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white)
  ![Amazon S3](https://img.shields.io/badge/S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white)
  ![CloudWatch](https://img.shields.io/badge/CloudWatch-FF4F8B?style=for-the-badge&logo=amazoncloudwatch&logoColor=white)
  ![IAM](https://img.shields.io/badge/IAM-DD344C?style=for-the-badge&logo=amazoniam&logoColor=white)
  ![AWS CLI](https://img.shields.io/badge/AWS%20CLI-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)

  <hr>

  **Author:** Ngurah Gede Wisnu Gudakesa</br>
  **Email:** ngurahgedewisnugk@gmail.com

</div>

---

## 📋 Table of Contents
- 🎯 [Introduction](#-introduction)
- 🎓 [Learning Objectives](#-learning-objectives)
- 📚 [Core Component](#-core-components)
- 📖 [Step-by-Step Build Guide](#-step-by-step-build-guide)
- 🛠 [Challenges & Solutions](#-challenges--solutions)
- ❓ [Why This Project?](#-why-this-project)

---

# 🎯 Introduction

This project series documents the process of designing, securing, and operating a production-style network on AWS — starting from a single isolated VPC and building up to multi-VPC peering, private S3 access, and full traffic observability. Across nine hands-on projects, I built the networking backbone of AWS from the ground up: defining CIDR blocks, segmenting public and private subnets, layering firewalls, connecting isolated VPCs, and monitoring live traffic — all using the AWS Management Console, CLI, and IAM.

# 🎓 Learning Objectives
- 🌐 **Network Isolation** — Design a custom Amazon VPC with defined CIDR blocks, public/private subnets, and Availability Zone redundancy
- 🔐 **Defense-in-Depth Security** — Apply layered protection using stateful Security Groups and stateless Network ACLs
- 🔗 **Cross-Network Connectivity** — Establish VPC Peering and VPC Endpoints for private, internet-free communication between resources and AWS services
- 📊 **Traffic Visibility** — Capture and analyze network activity using VPC Flow Logs and CloudWatch Logs Insights

---


# 📚 Core Components

| Component | Role | Description |
|---|---|---|
| **Amazon VPC** | Network Foundation | Provides an isolated virtual network with a custom IPv4 CIDR block (`10.0.0.0/16`) dedicated to the account |
| **Subnets (Public/Private)** | Network Segmentation | Divide the VPC into isolated sections; public subnets route to the internet, private subnets stay internal |
| **Internet Gateway** | Internet Access | Connects the VPC to the public internet, allowing public subnet resources to send and receive traffic |
| **NAT Gateway** | Private Outbound Access | Lets private subnet instances reach the internet for updates/patches without accepting inbound connections |
| **Route Tables** | Traffic Direction | Define where network traffic from each subnet is directed, based on destination CIDR matching |
| **Security Groups** | Instance-Level Firewall | Stateful rules that control inbound/outbound traffic for individual EC2 instances |
| **Network ACLs** | Subnet-Level Firewall | Stateless rules that control traffic entering and leaving an entire subnet |
| **VPC Peering** | Cross-VPC Communication | Directly connects two VPCs so resources can communicate via private IP addresses |
| **VPC Endpoints** | Private AWS Service Access | Enables private connectivity to AWS services like S3 without traversing the public internet |
| **VPC Flow Logs** | Network Monitoring | Captures IP traffic metadata and ships it to CloudWatch for analysis |
| **IAM Roles & Policies** | Access Control | Grants least-privilege permissions so services like Flow Logs and EC2 can interact with AWS resources |

### 🔧 Key Technologies
- **Amazon VPC** — isolated network foundation for every project in the series
- **Amazon EC2** — public and private compute instances used for connectivity testing
- **Amazon S3** — object storage accessed both publicly and privately via VPC Endpoints
- **AWS IAM** — roles and policies securing Flow Logs delivery and EC2 authentication
- **Amazon CloudWatch** — Flow Log storage and Logs Insights query analysis
- **AWS CLI** — command-line interaction with AWS services directly from EC2

## 🎯 Outcome
- ✅ Deployed a fully isolated VPC with public and private subnets across a configurable CIDR range
- ✅ Enabled secure internet access for public resources via an Internet Gateway and NAT Gateway
- ✅ Enforced defense-in-depth security using Security Groups and Network ACLs together
- ✅ Connected two independent VPCs through VPC Peering using private IP communication
- ✅ Captured and analyzed live network traffic using VPC Flow Logs and CloudWatch Logs Insights
- ✅ Enabled private, internet-free access to Amazon S3 using a VPC Gateway Endpoint

---

# 📖 Step-by-Step Build Guide

The series is structured as nine progressive builds, each layering a new networking or security concept on top of the last.

### 🛤️ Build Process

| Step | Topic | Description | Link |
|---|---|---|---|
| **01** | **Build a Virtual Private Cloud** | Created a custom VPC (`10.0.0.0/16`), configured subnets across Availability Zones, and attached an Internet Gateway | [View Project](https://github.com/ngurahgdewisnugk/AWS-Beginners-Challange/blob/e0f74c6ffffbc37f94b588493b7d47c97fd17cef/Project-Portofolio/04.%20Build%20a%20Virtual%20Private%20Cloud%20on%20AWS.md) |
| **02** | **VPC Traffic Flow & Security** | Configured route tables to make a subnet public, and layered Security Groups with Network ACLs for defense-in-depth | [View Project](https://github.com/ngurahgdewisnugk/VPC-Networking-Series/tree/e435b8dee3a032a49e601de9c8ffc56ddb5099d3/02%20-%20VPC%20Traffic%20Flow%20and%20Security) |
| **03** | **Creating a Private Subnet** | Built a dedicated route table and custom Network ACL to fully isolate a private subnet from the internet | [View Project](https://github.com/ngurahgdewisnugk/VPC-Networking-Series/tree/e435b8dee3a032a49e601de9c8ffc56ddb5099d3/03%20-%20Creating%20a%20Private%20Subnet) |
| **04** | **Launching VPC Resources** | Used the VPC resource map to launch public and private EC2 servers, and set up a NAT Gateway | [View Project](https://github.com/ngurahgdewisnugk/VPC-Networking-Series/tree/e435b8dee3a032a49e601de9c8ffc56ddb5099d3/04%20-%20Launching%20VPC%20Resources) |
| **05** | **Testing VPC Connectivity** | Verified instance-to-instance and internet connectivity using EC2 Instance Connect, ping, and curl | [View Project](https://github.com/ngurahgdewisnugk/VPC-Networking-Series/tree/e435b8dee3a032a49e601de9c8ffc56ddb5099d3/05%20-%20Testing%20VPC%20Connectivity) |
| **06** | **VPC Peering** | Connected two isolated VPCs via a peering connection and updated route tables for private IP communication | [View Project](https://github.com/ngurahgdewisnugk/VPC-Networking-Series/tree/e435b8dee3a032a49e601de9c8ffc56ddb5099d3/06%20-%20VPC%20Peering%20Connection) |
| **07** | **VPC Monitoring with Flow Logs** | Set up IAM roles/policies and VPC Flow Logs, then analyzed traffic patterns with CloudWatch Logs Insights | [View Project](https://github.com/ngurahgdewisnugk/VPC-Networking-Series/tree/e435b8dee3a032a49e601de9c8ffc56ddb5099d3/07%20-%20VPC%20Monitoring%20with%20Flow%20Logs) |
| **08** | **Access S3 from a VPC** | Configured IAM access keys so an EC2 instance could authenticate and interact with Amazon S3 over the internet | [View Project](https://github.com/ngurahgdewisnugk/VPC-Networking-Series/tree/e435b8dee3a032a49e601de9c8ffc56ddb5099d3/08%20-%20Access%20S3%20from%20VPC) |
| **09** | **VPC Endpoints** | Replaced public S3 access with a VPC Gateway Endpoint and enforced access via S3 bucket and endpoint policies | [View Project](https://github.com/ngurahgdewisnugk/VPC-Networking-Series/tree/e435b8dee3a032a49e601de9c8ffc56ddb5099d3/09%20-%20VPC%20Endpoints) |

---

# 🛠 Challenges & Solutions

| 🔧 Tool | ⚠️ Challenge | ✅ Solution |
|---|---|---|
| [EC2 Instance Connect](https://github.com/ngurahgdewisnugk/VPC-Networking-Series/tree/main/05%20-%20Testing%20VPC%20Connectivity#phase-1--establishing-ssh-access-to-the-public-server) | SSH connection failed because the Security Group only allowed HTTP traffic | Added an inbound rule permitting SSH (port 22) from Anywhere-IPv4 |
| [Network ACL / Security Group](https://github.com/ngurahgdewisnugk/VPC-Networking-Series/tree/main/05%20-%20Testing%20VPC%20Connectivity#phase-2--testing-inter-server-connectivity-via-ping) | Private Server sent no ping replies to the Public Server | Added inbound/outbound rules for All ICMP – IPv4 on both the Network ACL and Security Group |
| [EC2 Instance Connect (Peering)](https://github.com/ngurahgdewisnugk/VPC-Networking-Series/tree/main/06%20-%20VPC%20Peering%20Connection#phase-5--6--connect-via-ec2-instance-connect--troubleshooting) | "No public IPv4 address assigned" error blocked access to the instance | Allocated and associated an Elastic IP for a static public IPv4 address |
| [S3 Bucket Policy](https://github.com/ngurahgdewisnugk/VPC-Networking-Series/tree/main/08%20-%20Access%20S3%20from%20VPC#problem-solving--key-learnings) | Bucket denied all access, including the AWS Console, right after the policy update | Routed EC2 traffic through a VPC Gateway Endpoint and updated the subnet's route table to restore S3 access |
| [VPC Endpoint Policy](https://github.com/ngurahgdewisnugk/VPC-Networking-Series/tree/main/09%20-%20VPC%20Endpoints#problem-solving) | Switching the policy `Effect` to `Deny` blocked all S3 access through the endpoint | Reverted the `Effect` to `Allow` to restore intended least-privilege access |

---

# ❓ Why This Project?

### 🔴 The Pain Point
- Resources deployed without network isolation carry unnecessary and avoidable security risk
- A single flat firewall makes it hard to enforce least-privilege access at both the instance and subnet level
- Cross-VPC and hybrid architectures require careful CIDR planning to avoid IP conflicts and routing failures
- Sending traffic to AWS services like S3 over the public internet increases exposure and can add data transfer overhead
- Without traffic visibility, diagnosing connectivity issues (blocked ICMP, missing routes) is slow and manual

### 💚 The Solution
This series builds a layered, auditable network from first principles — isolating resources by subnet, enforcing security at both the instance and subnet level, connecting environments privately through peering and endpoints, and closing the loop with Flow Logs for full traffic visibility.

#### ✨ Key Benefits

| Feature | Benefit |
|---|---|
| Multi-AZ Subnet Design | Increases high availability — public resources stay reachable even if one Availability Zone fails |
| Layered Security (SG + NACL) | Reduces blast radius by enforcing rules at both the instance and subnet level |
| VPC Peering | Enables secure, low-latency communication between isolated environments without public internet exposure |
| VPC Endpoints | Removes the public internet dependency for AWS service access, improving security and reducing data transfer cost |
| VPC Flow Logs + CloudWatch Insights | Provides traffic visibility for faster troubleshooting and stronger security audits |