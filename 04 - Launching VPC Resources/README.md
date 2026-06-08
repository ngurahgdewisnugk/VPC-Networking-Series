# 🚀 Launching VPC Resources on AWS

**Project Author:** Ngurah Gede Wisnu Gudakesa  
**Contact:** ngurahgedewisnugk@gmail.com  
**Project Link:** [View Project](http://learn.nextwork.org/projects/aws-networks-private)

![Image](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-ec2_8ee57662)

This project walks through building a complete, production-style virtual private network on AWS — from the ground up. Think of it like setting up a secure office building in the cloud: some rooms (**public subnets**) are accessible from the street, while others (**private subnets**) are locked away and only reachable from trusted internal sources.
 
The goal was to design and deploy a **custom Amazon VPC** with properly configured subnets, routing rules, gateways, and two EC2 servers — one publicly accessible, one completely private — and verify that the network works correctly through live connectivity testing.

> ⏱️ **Total time invested:** 6 hours (including in-depth concept study)



---

## Architecture Overview

![architecture](<assets/ARCHITECTURE OVERVIEW.png>)

I used VPC resource map! to Creating component network resources at a lightning fast pace, with details : 
1. Creating Subnet. 

2. Configurating route table, 

3. and make network connection like internet gateways. 

Also launch a EC2 Instances for private server under private subnet configuration and public server under public subnet configurations.

It's a really useful and handy tool for visualizing complex network architectures and understanding how each component needs to be configured.

---

## 🏗️ What I Built

A secure, multi-availability-zone AWS network infrastructure featuring:
 
- A **custom VPC** with CIDR block `10.0.0.0/16` spanning two Availability Zones in `ap-southeast-2`
- **6 subnets** (2 public + 4 private) distributed across `ap-southeast-2a` and `ap-southeast-2b`
- **5 route tables** for fine-grained traffic control per subnet
- An **Internet Gateway** (`nextwork-igw`) enabling public internet access
- **2 EC2 instances** — one public-facing server, one fully private server
- **Dedicated security groups** with least-privilege inbound rules
- **SSH key pairs** (`.pem`) for secure, encrypted remote access

---

## Action: Step-by-Step Implementation

### Phase 1: 🔑 SSH Key Pairs (`.pem` format)

-  Direct VM access is essential for tasks like installing software, editing configuration files, or running connectivity test scripts — actions that cannot be performed through the AWS Management Console. SSH (Secure Shell) is used for this encrypted remote access. Key pairs consist of a **public key** installed on the EC2 instance and a **private key** kept locally as a `.pem` (Privacy Enhanced Mail) file. When connecting, the EC2 instance issues a cryptographic challenge that only the private key can unlock — ensuring only authorized users gain access.

### Phase 2: Launching a public server

![Image](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-ec2_88727bef)

I had to change my EC2 instance's networking settings by  ensuring my EC2 instance is launch on VPC Isolation environment, therefore, i configure : 

1. Choosing the NextWork-vpc that I had recently created.

2. Configuring the subnet connection to my public subnet.

3. Enabling the Auto-assign public IP setting, which is necessary for a public instance and for using EC2 Instance Connect.

4. Selecting the existing NextWork Public Security Group for the firewall settings.

### Phase 3: Launching a private server

![Image](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-ec2_4a9e8014)

My private server has its own dedicated security group because to restrict access to smaller group of trusted resources and securing resource to still private from public and only communicate with internal resource under VPC isolation environment. 

My private server's security group's source is set to NextWork Public Security Group. This means only resources that are part of the NextWork Public Security Group (like My public server) can initiate a connection with my private server. This significantly enhances security by preventing unauthorized access from the broader internet.

### Phase 4: Speeding up VPC creation (VPC Resource Map)

![Image](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-ec2_1cbb1b88)

- The VPC Resource Map is a visual, wizard-driven tool in the AWS Console that lets you create all VPC components — subnets, route tables, and gateways — simultaneously in a single configuration pass. It provides a "helicopter view" of the entire architecture, making it significantly faster to design and easier to troubleshoot, especially for complex multi-subnet setups like this one.
- Six subnets were created across two Availability Zones to achieve both redundancy and security segmentation.
 
  **Subnets created:**
 
| Name | Type | Availability Zone |
|---|---|---|
| `nextwork-subnet-public1-ap` | Public | ap-southeast-2a |
| `nextwork-subnet-private1-ap` | Private | ap-southeast-2a |
| `nextwork-subnet-private3-ap` | Private | ap-southeast-2a |
| `nextwork-subnet-public2-ap` | Public | ap-southeast-2b |
| `nextwork-subnet-private2-ap` | Private | ap-southeast-2b |
| `nextwork-subnet-private4-ap` | Private | ap-southeast-2b |

**Route tables created:**
 
| Name | Associated Subnets |
|---|---|
| `nextwork-rtb-public` | Public subnets (both AZs) |
| `nextwork-rtb-private1-ap-southeast-2a` | Private Subnet 1 |
| `nextwork-rtb-private2-ap-southeast-2b` | Private Subnet 2 |
| `nextwork-rtb-private3-ap-southeast-2a` | Private Subnet 3 |
| `nextwork-rtb-private4-ap-southeast-2b` | Private Subnet 4 |

---

### ✅ Outcomes
 
- ✔ Deployed a fully functional **dual-AZ custom VPC** with complete subnet, routing, and gateway configuration
- ✔ Successfully launched a **publicly accessible EC2 instance** via EC2 Instance Connect with a public IP
- ✔ Launched a **hardened private EC2 instance** accessible only through the public server (bastion pattern)
- ✔ Completed the entire architecture in approximately **6 hours**, with deep concept understanding at each step


---
