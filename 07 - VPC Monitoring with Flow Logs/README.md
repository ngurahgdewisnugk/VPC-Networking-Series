# VPC Monitoring with Flow Logs

A multi-VPC AWS network built with isolated CIDR blocks, secured with least-privilege IAM roles for log delivery, connected via VPC Peering, and continuously monitored using VPC Flow Logs and CloudWatch Logs Insights for traffic visibility and troubleshooting.

**Project Author:** [View Project](http://learn.nextwork.org/projects/aws-networks-monitoring)</br>
**Author:** Ngurah Gede Wisnu Gudakesa  
**Email:** ngurahgedewisnugk@gmail.com

---

## Architecture Overview

![architecture overview](<Assets/architecture overview.png>)
 
- This project provisions two fully isolated Virtual Private Clouds, **NextWork-1** (`10.1.0.0/16`) and **NextWork-2** (`10.2.0.0/16`), each built using the VPC wizard's "VPC and more" option to automatically generate a public subnet, route table, and Internet Gateway. An EC2 instance with a public IPv4 address is launched in each VPC's subnet, with security groups configured to allow inbound ICMP traffic from `0.0.0.0/0`. 

- This setup enables both the generation of real network traffic for monitoring purposes and connectivity testing between the two environments. A **VPC Peering Connection** links the two VPCs, with route tables on both sides updated to direct traffic destined for the peer VPC's CIDR block through the peering connection rather than out to the public internet.
 
- From a security and observability standpoint, the architecture follows a **least-privilege, defense-in-depth model**. 
- VPC Flow Logs are enabled on NextWork-1 to capture all accepted and rejected traffic at the ENI/subnet level, and a dedicated IAM policy and role — governed by a custom trust policy that permits only the VPC Flow Logs service to assume it — are created to deliver these logs to CloudWatch Log Groups. 
- This ensures log delivery permissions are scoped exclusively to the Flow Logs service, eliminating the risk of other services misusing the role. On top of this, **CloudWatch Logs Insights** is used to query the collected flow logs, transforming raw network records into actionable insights such as top talkers by byte volume — supporting both security auditing and performance troubleshooting.

---
 
## What I Built
 
A production-ready **multi-VPC network monitoring solution** that provides:
 
* **Isolated Multi-VPC Network Design**: Two VPCs (NextWork-1 and NextWork-2) provisioned with **unique, non-overlapping CIDR blocks** (`10.1.0.0/16` and `10.2.0.0/16`) to prevent routing conflicts, each with its own public subnet, route table, and Internet Gateway.
* **Private Inter-VPC Connectivity via VPC Peering**: A **VPC Peering Connection** with updated route tables on both VPCs, enabling direct, private IP-to-IP communication between resources in NextWork-1 and NextWork-2 without traversing the public internet.
* **Comprehensive Network Visibility with VPC Flow Logs**: **VPC Flow Logs** capturing source/destination IPs, ports, protocols, packet counts, byte transfers, and accept/reject status for all traffic, delivered to a **CloudWatch Log Group**.
* **Least-Privilege IAM Access for Log Delivery**: A custom **IAM policy and role**, secured with a **custom trust policy**, that grants the VPC Flow Logs service exactly the permissions it needs to create log groups/streams and write log events — nothing more.
* **Traffic Analytics with CloudWatch Logs Insights**: **CloudWatch Logs Insights** queries (e.g., "Top 10 byte transfers by source and destination IP addresses") to identify the busiest traffic routes and support network troubleshooting.

---

## Technology Stack
 
| Category | Service / Tool | Purpose |
|---|---|---|
| Networking | Amazon VPC | Isolated virtual networks (NextWork-1, NextWork-2) |
| Networking | VPC Peering | Private connectivity between the two VPCs |
| Networking | Internet Gateway & Route Tables | Public internet access and inter-VPC routing |
| Compute | Amazon EC2 | Instances for generating and testing network traffic |
| Monitoring | VPC Flow Logs | Capturing inbound/outbound traffic metadata |
| Monitoring | Amazon CloudWatch Logs | Centralized storage for flow log data |
| Monitoring | CloudWatch Logs Insights | Querying and analyzing flow log data |
| Security/IAM | IAM Policy & Role | Least-privilege permissions for Flow Logs to CloudWatch delivery |
| Security/IAM | Custom Trust Policy | Restricts role assumption to the VPC Flow Logs service only |
| Security | Security Groups | ICMP inbound rules for connectivity testing |
 
---

## Action: Step-by-Step Implementation

### Step 1 — Set Up Two VPCs

![Create VPC](<Assets/01 - Create VPC.png>)
 
Using the **VPC wizard's "VPC and more" option**, two VPCs were provisioned from scratch — NextWork-1 (`10.1.0.0/16`) and NextWork-2 (`10.2.0.0/16`) — each automatically generating one public subnet, one route table, and one Internet Gateway. This approach minimizes manual configuration and setup time while keeping the CIDR ranges unique to avoid IP overlap, routing conflicts, and connectivity issues between the two environments.

### Step 2 — Launch EC2 Instances
 
An EC2 instance was launched in each VPC's subnet, configured with:
 
* A **public IPv4 address** for direct reachability and connectivity testing.
* A **security group inbound rule allowing All ICMP - IPv4 traffic from `0.0.0.0/0`**, enabling ping-based connectivity tests between instances.
  
These instances serve a dual purpose: generating the network traffic that VPC Flow Logs will capture, and acting as test endpoints for the upcoming VPC Peering connection.

### Step 3 — Set Up VPC Flow Logs

![Create Flow Logs](<Assets/03 - VPC Create Flow Logs.png>)
 
**VPC Flow Logs** were configured on NextWork-1 to record all inbound and outbound traffic for the VPC, with logs stored in a **CloudWatch Log Group** for retention and analysis. Each captured record includes:
 
* Source and destination IP addresses
* Protocol and destination port
* Number of packets and bytes transferred
* Accept or reject status

### Step 4 — Configure IAM Permissions for Flow Logs
 
To allow Flow Logs to deliver data to CloudWatch, the following IAM resources were created:
 
* **IAM Policy** — grants permissions to create log groups/streams and put log events into CloudWatch Logs.
  ![IAM Policy](<04 - VPC Flow IAM Policy.png>)
* **IAM Role with a Custom Trust Policy** — the trust policy restricts role assumption to the **VPC Flow Logs service only**, preventing other AWS services from using the role. The IAM policy is attached to this role.
  ![IAM Custom Policy](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-monitoring_4334d777)
* **Role Assignment** — the completed role is assigned to the subnet's Flow Log configuration, finalizing the log delivery pipeline.
  ![IAM Role](<Assets/04 - VPC Flow IAM Role.png>)

### Step 5 — Initial Connectivity (Ping) Test
 
A ping test was run from Instance 1 to Instance 2 using **private IPv4 addresses** to validate cross-VPC connectivity prior to establishing peering.

![VPC Testing](<Assets/05 - VPC Testing.png>)
 
### Step 6 — Establish a VPC Peering Connection
 
- A **VPC Peering Connection** was created between NextWork-1 and NextWork-2.
  ![vpc peering connection](<Assets/vpc peering connection.png>) 
- and the **route tables of both VPCs were updated** to direct traffic destined for the peer VPC's CIDR block through the peering connection — enabling secure, private IP-based communication between the two networks.
  ![Route Table](<Assets/VPC 1 Route Table .png>)
 
### Step 7 — Analyze Flow Logs with CloudWatch Logs Insights
 
Using **CloudWatch Logs Insights**, the collected flow log data was queried to identify traffic patterns. The query **"Top 10 byte transfers by source and destination IP addresses"** was executed to surface the busiest traffic routes in the network.

![Log Insight](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-monitoring_3e1e79a1)

---

## Problem Solving

### Issue: Private IP Connectivity Failure Before Peering
 
**Symptom:** The initial ping test from Instance 1 to Instance 2's **private IPv4 address** returned no replies.

![Ping Instance 1](<Assets/ping instance 1 .png>) 
 
**Root Cause:** No VPC Peering Connection existed between NextWork-1 and NextWork-2. As a result, traffic destined for VPC 2's private IP range had no direct path and was instead routed out through the Internet Gateway to the public internet — meaning private-IP communication between the VPCs was not yet possible.
 
**Diagnostic Step:** A follow-up ping test using Instance 2's **public IP address** succeeded, confirming the instances were individually reachable, but only over the public internet — isolating the problem specifically to the lack of a private cross-VPC path.

![Ping Public Ip](<Assets/07 - ping public ip.png>)
 
**Resolution:** A **VPC Peering Connection** was established **on step 6**, and **route tables in both VPCs were updated** to point traffic destined for the peer VPC's CIDR block at the peering connection.
 
**Result:** A repeated ping test using Instance 2's **private IP address** succeeded, confirming that:

![ping instance 1 private address](<Assets/ping instance 1 private address.png>)

* Private IP-to-IP communication between VPC 1 and VPC 2 was restored.
* Traffic between the VPCs is now routed directly via the peering connection rather than over the public internet — improving both performance and security posture.
```
Before: Instance 1 → Internet Gateway → Public Internet → Instance 2 (public IP only, exposed traffic)
After:  Instance 1 → VPC Peering Connection → Instance 2 (private IP, isolated from public internet)
```
 
---
 
## Key Takeaways
 
* **CIDR planning matters**: Assigning unique, non-overlapping CIDR blocks (`10.1.0.0/16` vs `10.2.0.0/16`) up front prevented routing conflicts between the two VPCs.
* **Peering requires both route tables**: Establishing a peering connection alone is not enough — both VPCs' route tables must explicitly point traffic to the peering connection for private communication to work.
* **Least-privilege IAM is foundational to observability**: A custom trust policy ensures only the VPC Flow Logs service can assume the role used to deliver logs to CloudWatch, reducing the attack surface of the monitoring pipeline.
* **Log organization drives insight quality**: How CloudWatch Log Groups are structured directly affects how effectively CloudWatch Logs Insights can be used to query (on Step 7) and visualize network activity.
---
 
## Project Reflection
 
This project took approximately **8 hours**, with time dedicated to building a thorough conceptual understanding of each topic — VPC isolation, peering, Flow Logs, IAM trust policies, and Logs Insights queries — rather than just completing the configuration steps. The most significant takeaway was recognizing how critical CloudWatch Log Group organization is: it's not just about collecting flow log data, but about structuring it in a way that unlocks meaningful, query-driven insights into network behavior.

![flow logs](<Assets/expanded flow logs.png>)

---
---
