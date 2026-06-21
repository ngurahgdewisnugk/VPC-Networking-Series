# AWS VPC Endpoints — Private S3 Connectivity for EC2

A production-grade AWS networking project that establishes private, internet-free connectivity between an EC2 instance and Amazon S3 using a **VPC Gateway Endpoint**, reinforced by a defense-in-depth security model that layers **S3 bucket policies** and **VPC endpoint policies** to eliminate public internet exposure for service-to-service traffic.

**Project Author:** [View Project](http://learn.nextwork.org/projects/aws-networks-endpoints)</br>
**Author:** Ngurah Gede Wisnu Gudakesa  
**Email:** ngurahgedewisnugk@gmail.com

---

## Architecture Overview

![Architecture Overview](<Assets/architecture overview.png>)

### How I used Amazon VPC in this project

In this project, I used Amazon VPC to establish secure and private communication between my VPC resources and external AWS services, specifically Amazon S3.

I began by setting up an isolated VPC environment and launched EC2 instances within it. I then configured authentication for the EC2 instance to interact with S3 while maintaining VPC network isolation.

Crucially, I configured traffic using VPC endpoints to ensure that this communication with S3 was secure and private, without needing to traverse the public internet.

### One thing I didn't expect in this project was...

One thing I didn't expect in this project was the powerful, defense-in-depth security that can be achieved with both S3 Bucket Policies and VPC Endpoint Policies. It was surprising to see how these policies could restrict access so granularly, even blocking the AWS Management Console itself. This level of control is incredibly useful for minimizing risk and enhancing security in a production environment.

### This project took me...

This project took me 3 hours. I really enjoyed dedicating this time to thoroughly understanding each concept, and I'm excited to see more projects like this one!

---
 
## What I Built
 
A production-ready **private AWS service connectivity** solution that provides:
 
* **Isolated VPC Network Foundation**: A custom VPC with a public subnet, scoped security group, and an EC2 instance configured for secure, auditable access via **EC2 Instance Connect**.
* **Authenticated Programmatic Access**: IAM **Access Keys** configured on the EC2 instance to authenticate AWS CLI calls against S3, with **IAM roles** identified as the recommended production alternative to avoid storing long-lived credentials on the instance.
* **Private S3 Connectivity via Gateway Endpoint**: A **VPC Gateway Endpoint** and updated **route table** that direct all S3-bound traffic privately within the AWS network backbone, fully bypassing the public internet.
* **Defense-in-Depth Access Control**: Layered **S3 bucket policies** and **VPC endpoint policies** that restrict access exclusively to traffic from the designated endpoint, validated by deliberately toggling policy effects from `Allow` to `Deny` to confirm enforcement at each layer.
---
 
## Technology Stack
 
| Service / Tool | Purpose |
|---|---|
| **Amazon VPC** | Isolated network environment (`10.0.0.0/16`) |
| **Amazon EC2** | Compute instance (Amazon Linux 2023, `t2.micro`) hosting the AWS CLI client |
| **Amazon S3** | Object storage target for private connectivity testing |
| **VPC Gateway Endpoint** | Private routing path between the VPC and S3, bypassing the internet |
| **Route Tables** | Redirect S3-bound traffic from the Internet Gateway to the VPC endpoint |
| **IAM Access Keys & Policies** | Programmatic authentication and fine-grained access control |
| **S3 Bucket Policy** | Identity-based control restricting bucket access to the VPC endpoint |
| **Security Groups** | Instance-level firewall permitting SSH for EC2 Instance Connect |
| **AWS CLI** | Command-line interface used to validate connectivity (`ls`, `cp`) |
 
---

## Action: Step-by-Step Implementation

### Phase 1 — Architecture Setup & Baseline Connectivity (Public Internet)
 
**Step 1: VPC & EC2 Launch**

- Created `NextWork-vpc` (`10.0.0.0/16`) with a single public subnet at this stage. 
  ![Create VPC](<Assets/01 - Create VPC.png>)
- Launched `Instance - NextWork VPC Project` (Amazon Linux 2023, `t2.micro`) into the public subnet with auto-assigned public IP, secured by `SG - NextWork VPC Project` allowing SSH. 
- Created the `nextwork-vpc-project-wsworkspace` S3 bucket and uploaded two `.png` objects.
  ![upload object](<Assets/upload objects.png>)
 
**Step 2: Connect to the EC2 Instance** </br>
Connected to the instance using **EC2 Instance Connect** to establish a CLI session capable of reaching S3 over the public internet. </br>
![ec2 instance](<Assets/02 - connect ec2 instace.png>)
 
**Step 3: Set Up IAM Access Keys** </br>
Generated an **Access Key ID** and **Secret Access Key** and configured them on the instance via `aws configure` (Access Key ID, Secret Access Key, default region, default output format), giving the EC2 instance its own credentials to authenticate independently with AWS services.
![aws configure](<Assets/03 - aws configure.png>)
 
**Step 4: Validate S3 Access via AWS CLI** 
- `aws s3 ls` → listed all S3 buckets in the account, confirming successful authentication.
  ![Image](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-endpoints_4334d778)

- `aws s3 ls s3://nextwork-vpc-project-wsworkspace` → listed the two uploaded `.png` objects.
  ![Image](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-endpoints_4334d779)

- `sudo touch /tmp/nextwork.txt` → created a blank test file on the instance.
- `aws s3 cp /tmp/nextwork.txt s3://nextwork-vpc-project-wsworkspace` → uploaded the file as a new object.
- `aws s3 ls s3://nextwork-vpc-project-wsworkspace` → confirmed the new object was present.
  ![Image](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-endpoints_3e1e79a2)
  
---

### Phase 2 — Enforcing Private Connectivity (VPC Endpoint)
 
**Step 5: Set Up a Gateway Endpoint** </br>
Created an **VPC Endpoint**, adding a route in the VPC route table that directs traffic bound for Amazon S3 directly to the endpoint instead of the internet. Gateway endpoints manage routing only and carry no inherent identity-based access control — that layer is added separately via policies.

![Image](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-endpoints_09bcaa8a)

**Step 6: Configure the S3 Bucket Policy** </br>
- Applied a bucket policy denying all access to the bucket except requests originating from the specified VPC endpoint.</br> 
  ![Image](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-endpoints_7316a13d)
- Immediately after saving, the bucket showed **Access Denied** for all non-endpoint sources, including the AWS Management Console.
  ![Image](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-endpoints_4ec7821f)
  
**Step 7: Update the Route Table** </br>
- Navigated to the Endpoints page, selected the VPC endpoint, and managed its route table associations by attaching the public route table. Verified the update in the public subnet's route table tab. 
  ![Image](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-endpoints_d116818e)
- Re-ran `aws s3 ls s3://nextwork-vpc-project-wsworkspace` from the EC2 instance, which now succeeded — confirming traffic was being routed privately through the endpoint rather than failing over the (now-blocked) public path.
  ![Image](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-endpoints_4334d779)
  
 
**Step 8: Validate the Endpoint Connection & Endpoint Policy** </br>
- Re-tested S3 connectivity from the EC2 instance to confirm the endpoint route was functioning correctly. 
  ![vpc endpoint](<Assets/04 - endpoint policies.png>)
- Then edited the **VPC endpoint policy**, changing `"Effect": "Allow"` to `"Effect": "Deny"`. 
  ![vpc endpoint deny](<Assets/04 - endpoint policies - deny.png>)
- Re-running `aws s3 ls` immediately returned an **Access Denied** error — this time enforced at the VPC endpoint level rather than the bucket level, confirming that both policy layers were independently effective.
  ![Image](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-endpoints_3e1e79a3)

---

## Problem Solving
 
The most instructive moment in this project was discovering that a restrictive **S3 bucket policy** blocked *every* access path except the VPC endpoint — including the AWS Management Console itself, which is normally treated as an implicitly trusted, "break-glass" access method.
 
| Access Path | Before Bucket Policy | After Bucket Policy (Endpoint-Only) |
|---|---|---|
| EC2 instance via VPC Endpoint | ✅ Allowed | ✅ Allowed |
| AWS Management Console | ✅ Allowed | ❌ Denied |
| Public Internet (any source) | ✅ Allowed | ❌ Denied |
 
**Mental model:** Security here operates as two independent gates that both must open for a request to succeed:
 
1. **Network-layer gate (Route Table → Gateway Endpoint):** determines *whether traffic can reach S3 privately at all.*
2. **Identity-layer gates (Bucket Policy + Endpoint Policy):** determine *whether that traffic is authorized*, evaluated from both the resource side (bucket policy) and the network-boundary side (endpoint policy).
By the end of Phase 2, **100% of S3 traffic from the EC2 instance was eliminated from the public internet path**, with both policy layers verified independently by toggling their `Effect` between `Allow` and `Deny` and observing the access failure at the expected layer each time. This dual-toggle test methodology proved that the controls were genuinely enforced rather than just configured.

---

## Lessons Learned
 
- **IAM roles over access keys**: While access keys were used here for learning purposes, attaching an **IAM role** to the EC2 instance is the recommended best practice in production, removing the need to manage long-lived credentials on the instance.
- **Defense-in-depth is layered, not redundant**: Route tables, bucket policies, and endpoint policies each control a different dimension of access (path, resource, and boundary), and combining them closes gaps that any single control would leave open.
- **Console access is not a trusted default**: A sufficiently restrictive bucket policy can — and should, in sensitive environments — deny even the AWS Management Console, proving that access control should be based on explicit trust boundaries rather than assumed administrative privilege.
- **Verify by breaking, not just by building**: Deliberately switching policy effects from `Allow` to `Deny` was the clearest way to confirm each security layer was actually functioning as intended.

---

## Conclusion
 
This project demonstrates a complete, verifiable workflow for moving AWS service traffic off the public internet and onto private AWS network paths, while layering identity-based controls to enforce least-privilege access. The result is an EC2-to-S3 connectivity pattern that is private by default, explicitly authorized at multiple layers, and resilient against unintended access — a foundational pattern for any production VPC architecture handling sensitive data.

---
