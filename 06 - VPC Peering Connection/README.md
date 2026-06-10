# VPC Peering
A hands-on AWS networking project that establishes a secure, private VPC Peering connection between two isolated Virtual Private Clouds, enabling direct inter-VPC communication via private IP addresses — Securely connect to the public internet when necessary, with precise control over how traffic flows in and out.

**Project Author:** [View Project](http://learn.nextwork.org/projects/aws-networks-peering)\
**Author:** Ngurah Gede Wisnu Gudakesa  
**Email:** ngurahgedewisnugk@gmail.com

---

## Architecture Overview

This project deploys a **dual-VPC architecture** in the AWS Sydney (`ap-southeast-2`) region. Two fully isolated VPCs — `NextWork-1-vpc` (`10.1.0.0/16`) and `NextWork-2-vpc` (`10.2.0.0/16`) — are provisioned using the AWS VPC Wizard. Each VPC is self-contained with its own public subnet, route table, and Internet Gateway. A **VPC Peering Connection** (`pcx-*`) bridges the two networks, and route tables on both sides are updated to funnel cross-VPC traffic through the peering link rather than the public internet. EC2 instances are deployed in each VPC to serve as test endpoints.
 
The security model follows a **least-privilege, defense-in-depth** approach. Non-overlapping CIDR blocks prevent routing conflicts. Security group inbound rules on Instance 2 are scoped specifically to the `10.1.0.0/16` CIDR range — meaning only resources originating from VPC 1 can send ICMP traffic to VPC 2. EC2 Instance Connect, backed by **Elastic IP addresses**, provides secure browser-based SSH access without requiring locally-managed key pairs or bastion hosts.
 
![alt text](<assets/Architecture Overview.png>)


---
## What I Built
 
A production-ready **AWS multi-VPC private networking** solution that provides:
 
| Pillar | Description |
|---|---|
| **Network Isolation** | Two fully independent VPCs with non-overlapping **IPv4 CIDR blocks** (`10.1.0.0/16` and `10.2.0.0/16`), preventing routing conflicts and ensuring clean network boundaries |
| **Private Peering** | A **VPC Peering Connection** that enables direct, encrypted communication between VPCs over AWS's private backbone — no traffic traverses the public internet |
| **Secure Access** | **EC2 Instance Connect** with **Elastic IP addresses** provides static, browser-based SSH access without local key pair management |
| **Validated Connectivity** | End-to-end ICMP ping testing between EC2 instances confirmed zero packet loss (`9 packets transmitted, 9 received, 0% packet loss`) across the peering link |
| **Total completion time** | ~8 hours, included dedicated time for understanding each topic's concepts of "EC2 Instance can have multiple Private IPs" and "Bring Your Own IP"|
 
---

## Key Technologies
 
| Service / Technology | Role in Project |
|---|---|
| **Amazon VPC** | Core isolated network environment for each deployment unit |
| **VPC Peering Connection** | Private, direct routing channel between the two VPCs |
| **Amazon EC2** | Compute endpoints used to generate and receive test traffic |
| **Elastic IP Address** | Static public IPv4 required for EC2 Instance Connect access |
| **Internet Gateway (IGW)** | Outbound internet path for public subnets in each VPC |
| **Route Tables** | Traffic steering logic directing cross-VPC packets through the peering link |
| **Security Groups** | Stateful firewall rules scoped to specific CIDR sources for ICMP and SSH |
| **EC2 Instance Connect** | Browser-based SSH, eliminating local key pair dependencies |
 
---

## Action: Step-by-Step Implementation

### Phase 1 — Provision Both VPCs

Used the **AWS VPC Wizard ("VPC and more")** to create two VPCs in a single, streamlined flow. Each VPC was provisioned with:
 
- 1 × Public Subnet
- 1 × Route Table (public)
- 1 × Internet Gateway
  
| Resource | VPC 1 | VPC 2 |
|---|---|---|
| **Name** | `NextWork-1-vpc` | `NextWork-2-vpc` |
| **CIDR Block** | `10.1.0.0/16` | `10.2.0.0/16` |
| **Subnet** | `ap-southeast-2a` | `ap-southeast-2a` |
| **Internet Gateway** | Attached | Attached |
 
> **Why non-overlapping CIDRs?** Overlapping IP address ranges make routing ambiguous — AWS cannot distinguish which VPC a packet is destined for, causing routing failures. CIDR uniqueness is a hard prerequisite for VPC Peering.

![Provision VPC](<assets/VPC 2's Connection.png>)
 
---

### Phase 2 - Create the VPC Peering Connection

Initiated a **Peering Connection Request** from VPC 1 (Requester) to VPC 2 (Accepter) within the same AWS account and region.
 
| Field | Value |
|---|---|
| **Requester VPC** | `vpc-000ff151bddebb46e` / `NextWork-1-vpc` |
| **Accepter VPC** | `vpc-011e7313a18d81c21` / `NextWork-2-vpc` |
| **Requester CIDR** | `10.1.0.0/16` |
| **Accepter CIDR** | `10.2.0.0/16` |
| **Region** | `ap-southeast-2` (Sydney) |
| **Connection ID** | `pcx-08fe3d920a4dededb` |
 
> **Handshake model:** The peering connection is not active until the Accepter VPC explicitly approves the request. This two-sided consent model ensures no network can be silently peered without mutual agreement.

![Image](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-peering_88727bef)

---

### Phase 3 — Update Route Tables in Both VPCs
 
After acceptance, both route tables require an explicit entry pointing cross-VPC traffic at the peering connection. Without this, packets have no path to the peer and the connection is non-functional.
 
**VPC 1 Route Table (`NextWork-1-rtb-public`)**
 
| Destination | Target | Purpose |
|---|---|---|
| `10.1.0.0/16` | local | Local VPC traffic |
| `10.2.0.0/16` | `pcx-08fe3d920a4dededb` | Route to VPC 2 via peering |
| `0.0.0.0/0` | `igw-*` | Internet-bound traffic |

![VPC 1 Route Table](<assets/VPC 1's route table .png>)
 
**VPC 2 Route Table (`NextWork-2-rtb-public`)**
 
| Destination | Target | Purpose |
|---|---|---|
| `10.2.0.0/16` | local | Local VPC traffic |
| `10.1.0.0/16` | `pcx-08fe3d920a4dededb` | Route to VPC 1 via peering |
| `0.0.0.0/0` | `igw-*` | Internet-bound traffic |

![VPC 2 Route Table](<assets/VPC 2's route table .png>)
 
---

### Phase 4 - Launch EC2 Instances

In this step, I will launch an EC2 instance in each VPC because these instances will be used to test the VPC peering connection later in the project.

- **Key Pair**: Not configured — EC2 Instance Connect handles authentication
- **Public IP**: Auto-assign disabled by default (resolved in Step 5)
- **Security Groups**: Default inbound SSH rule; ICMP rule added in Step 7

![Launch EC2 Instance](<assets/instance summary.png>)

---

### Phase 5 & 6 — Connect via EC2 Instance Connect + Troubleshooting
 
**Problem encountered:**
 
```
Error: No public IPv4 address assigned.
With no public IPv4 address, you can't use EC2 Instance Connect.
```
![Error Response](<assets/showing yellow error.png>)

**Root cause:** Auto-assign public IP was disabled at subnet level, leaving Instance 1 with no public IPv4.
 
**Resolution — Elastic IP Address:**
 
Allocated a new **Elastic IP** from Amazon's IPv4 pool and associated it with Instance 1.

![elastic ip](<assets/elastic ip address.png>)
 
| Attribute | Detail |
|---|---|
| **Why Elastic IP?** | Provides a *static* public IPv4 that persists across instance reboots, unlike the default dynamic IP |
| **Business value** | Prevents DNS update cycles every time an instance restarts; critical for production availability |
| **Region** | `ap-southeast-2` |
 
> This resolved the Instance Connect error immediately — the static IPv4 gave the browser-based SSH client a stable endpoint to negotiate the connection.

![ec2 instance connect](<assets/ec2 instance connect.png>)

---
 
### Phase 7 — Test VPC Peering (Ping Validation)
 
From Instance 1, executed a ping targeting Instance 2's **private IP address**:
 
```bash
ping 10.2.8.120
```

![Test Ping](<assets/test ping.png>)
 
**Initial result:** Request timed out — no response from Instance 2.
 
**Root cause:** Instance 2's security group had no inbound rule permitting ICMP traffic.

![sg instance 2](<assets/sg instance 2 inbound rule.png>)
 
**Fix — Security Group Inbound Rule on Instance 2:**
 
| Field | Value |
|---|---|
| **Type** | All ICMP - IPv4 |
| **Protocol** | ICMP |
| **Source** | `10.1.0.0/16` (VPC 1's CIDR only) |
 
**Final result:**

![Image](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-peering_7a29d352)

✅ **0% packet loss** — VPC Peering validated and fully operational.

## Problem Solving
 
### Challenge Matrix
 
| # | Problem | Root Cause | Resolution | Impact |
|---|---|---|---|---|
| 1 | EC2 Instance Connect failure | Auto-assign public IP disabled; no IPv4 present | Allocated & associated an **Elastic IP** | Static public access restored; SSH operational |
| 2 | Ping timeout between instances | Security group on Instance 2 blocked ICMP | Added `All ICMP - IPv4` inbound rule scoped to `10.1.0.0/16` | Cross-VPC ICMP confirmed; 0% packet loss achieved |
 
### Mental Model: Why Two Fixes Were Needed
 
```
EC2 Instance Connect failure            Ping timeout
        │                                     │
        ▼                                     ▼
  No public IPv4              Route table ✔  BUT  Security Group ✘
  → Elastic IP fix            Traffic reaches VPC 2 BUT
                              Instance 2 SG drops ICMP by default
                              → Add ICMP rule scoped to 10.1.0.0/16
```
 
> **Key insight:** The route table controls *where* traffic goes. The security group controls *whether* it's allowed in. Both layers must be configured correctly — this is the core of AWS's defense-in-depth model.

## Key Learnings & Concepts Explored

---
 
| Concept | Insight |
|---|---|
| **VPC Peering Handshake** | Both Requester and Accepter must explicitly consent — peering is never unilaterally forced |
| **Bidirectional Route Table Updates** | Accepting a peering request alone is insufficient; routes must be manually added on both sides |
| **Elastic IP vs. Dynamic IP** | Default EC2 public IPs are ephemeral (lost on restart); Elastic IPs provide production-grade address stability |
| **Security Groups as Stateful Firewalls** | ICMP must be explicitly permitted per-source; default-deny means silently dropped packets, not visible errors |
| **EC2 Multiple Private IPs** | A single EC2 instance can host multiple private IP addresses — enabling multi-application hosting on one compute node |
| **Bring Your Own IP (BYOIP)** | AWS supports importing on-premises IP ranges, enabling seamless IP continuity during cloud migrations |

---
