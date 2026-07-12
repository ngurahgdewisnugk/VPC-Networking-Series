# Testing VPC Connectivity

A hands-on AWS networking project that validates end-to-end traffic flow within an Amazon VPC by configuring **EC2 Instance Connect**, enforcing **multi-layer firewall rules** across Security Groups and Network ACLs, and verifying both inter-instance and internet connectivity using industry-standard diagnostic tools.

**Project Author:** [View Project](http://learn.nextwork.org/projects/aws-networks-connectivity) </br>
**Author:** Ngurah Gede Wisnu Gudakesa  
**Email:** ngurahgedewisnugk@gmail.com

---

## Architecture Overview

![architecture overview](<assets/architecture overview after.png>)

- **Network Boundary:** An isolated **Amazon VPC** divided into a **public subnet** and a **private subnet**, each hosting one dedicated EC2 instance — the *NextWork Public Server* and the *NextWork Private Server* respectively.
- **Traffic Entry Point:** Inbound internet traffic reaches the Public Server exclusively through an **Internet Gateway**, while the Private Server remains shielded from any direct external access.
- **Diagnostic Tools Used:**
  - **SSH via EC2 Instance Connect** — browser-based terminal access to the Public Server
  - **ICMP `ping`** — inter-server reachability testing between Public and Private Server
  - **`curl` over HTTPS** — internet egress validation from the public subnet
  
Security was enforced through a **defense-in-depth** model operating at two distinct layers. 
1. At the subnet boundary, **Network ACLs (NACLs)** act as stateless packet filters controlling traffic entering and leaving each subnet. 
2. At the instance boundary, **Security Groups** act as stateful firewalls controlling which protocols and sources are permitted. Both layers required explicit configuration for each traffic type tested — SSH, ICMP, and HTTP/HTTPS — ensuring no implicit trust existed anywhere in the topology.

--- 
### This project took me...

This project took me 5 hours, including the time spent on understand each concept.

---

## What I Built
 
A production-ready **AWS VPC connectivity testing environment** that provides:
 
- **Isolated Multi-Tier Network**: A VPC with clearly separated **public and private subnets**, ensuring the Private Server is never directly reachable from the internet while maintaining controlled internal communication paths.
- **Granular Access Control**: A **defense-in-depth security model** using both **Network ACLs** (stateless, subnet-level) and **Security Groups** (stateful, instance-level) to enforce least-privilege access for every protocol tested.
- **Verified Inter-Instance Communication**: Confirmed bidirectional **ICMP reachability** between the Public Server and Private Server using `ping`, validating that the internal network path is open and functional after proper NACL and Security Group tuning.
- **Validated Internet Egress**: Confirmed outbound internet connectivity from the Public Server using `curl` over **HTTPS**, retrieving live HTML content from an external web server and proving the routing and firewall configuration is production-ready.

## Action: Step-by-Step Implementation
 
### Phase 1 — Establishing SSH Access to the Public Server

![error EC2 Instance](<assets/error ec2 instance connect.png>)
 
The first objective was to access the **NextWork Public Server** directly from the AWS Management Console using **EC2 Instance Connect** — a browser-based SSH client that manages ephemeral key injection without requiring local key-pair setup.
 
**Initial State:**  

![Public Security Group](<assets/public Security Group.png>)

The Public Server's Security Group was pre-configured with an inbound rule permitting **HTTP (port 80)** traffic only. No SSH rule existed.
 
**Action Taken:**  
![inbound rule sg](<assets/security group inbound rule.png>)
Added a new **inbound rule** to the Public Server's Security Group:
 
| Field      | Value              |
|------------|--------------------|
| Type       | SSH                |
| Protocol   | TCP                |
| Port       | 22                 |
| Source     | Anywhere-IPv4 (0.0.0.0/0) |
 
**Outcome:**  

![alt text](<assets/success connect ssh.png>)

EC2 Instance Connect successfully established a terminal session to the Public Server, confirming the firewall change was applied correctly.


---

### Phase 2 — Testing Inter-Server Connectivity via Ping
 
With shell access to the Public Server established, the next step was to verify **ICMP reachability** to the Private Server using the `ping` command.
 
**Ping Command Executed:**
```bash
ping 10.0.1.180
```
 
Where `10.0.1.180` is the **private IPv4 address** of the NextWork Private Server.
 
**Initial Result:**  
![alt text](<assets/ping traffic stuck.png>)

The command returned only a single line with no replies — the Public Server transmitted ICMP Echo Requests, but the Private Server returned no responses.
 
**Root Cause Analysis:**  

![Inbound outbound NACL](<assets/subnet private server.png>)

ICMP traffic was being blocked at **two independent security layers**:
 
| Layer | Resource | Missing Rule |
|---|---|---|
| Subnet-level | Private Network ACL | No inbound/outbound rule for All ICMP – IPv4 |
| Instance-level | Private Server Security Group | No inbound rule for All ICMP – IPv4 |
 
**Fixes Applied:**
 
**1. Private Network ACL — Inbound Rule Added:**

![!\[SG Inbound Rule\](<assets/security group inbound rule.png>)](<assets/NACL inboundrule.png>)
 
| Rule # | Type | Protocol | Source CIDR | Action |
|---|---|---|---|---|
| 100 | All ICMP – IPv4 | ICMP | 10.0.0.0/24 (public subnet) | ALLOW |
 
**Private Network ACL — Outbound Rule Added:**

![NACL Outbound Rule](<assets/NACL outbound rule.png>)
 
| Rule # | Type | Protocol | Destination CIDR | Action |
|---|---|---|---|---|
| 100 | All ICMP – IPv4 | ICMP | 10.0.0.0/24 (public subnet) | ALLOW |
 
> **Note:** NACLs are **stateless** — both inbound and outbound rules must be explicitly defined for bidirectional traffic.
 
**2. Private Server Security Group — Inbound Rule Added:**

![SG Inbound Rule](<assets/security group inbound rule-new.png>)

| Field | Value |
|---|---|
| Type | All ICMP – IPv4 |
| Protocol | ICMP |
| Source | NextWork Public Security Group (SG ID) |
 
> **Note:** Security Groups are **stateful** — only an inbound rule was required; return traffic is automatically permitted.
 
**Outcome:**  
`ping 10.0.1.180` began returning consistent ICMP Echo Replies, confirming bidirectional communication between the two servers was fully operational.

![Image](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-connectivity_4a9e8014)
 
---
### Phase 3 — Validating Internet Connectivity via Curl
 
The final test confirmed that the Public Server had functional **outbound internet access** by retrieving live web content using `curl`.
 
**Curl Command Executed:**
```bash
curl https://learn.nextwork.org/projects/aws-host-a-website-on-s3
```
 
**Outcome:**  
![Image](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-connectivity_8ee57662)

The command returned a large volume of raw HTML content from NextWork's web application, confirming the Public Server could:
1. Resolve an external DNS name
2. Establish a **HTTP** connection over port 80
3. Successfully retrieve data from a remote web server via the **Internet Gateway**
   
## Key Technologies
 
| Technology | Category | Role in Project |
|---|---|---|
| Amazon VPC | Networking | Isolated virtual network environment |
| Amazon EC2 | Compute | Public and Private Server instances |
| Security Groups | Firewall (Instance-level) | Stateful traffic filtering per instance |
| Network ACLs | Firewall (Subnet-level) | Stateless traffic filtering per subnet |
| EC2 Instance Connect | Access Management | Browser-based SSH without local key setup |
| ICMP / `ping` | Diagnostics | Inter-instance reachability testing |
| `curl` | Diagnostics | HTTP/HTTPS internet egress validation |
| Internet Gateway | Networking | Outbound internet access for public subnet |
 
---
 
## Key Takeaways
 
- **Defense-in-depth is non-negotiable.** Both the Network ACL and the Security Group must be correctly configured — passing one layer is not sufficient. Each layer independently blocks traffic if not explicitly permitted.
- **Stateless vs. Stateful matters.** NACLs require both inbound *and* outbound rules for bidirectional traffic. Security Groups handle return traffic automatically, reducing configuration overhead but requiring careful mental modeling.
- **Least-privilege sourcing.** Scoping the Private Server's Security Group rule to the *Public Security Group ID* (rather than `0.0.0.0/0`) ensures only traffic originating from the Public Server is permitted — a key best practice for internal communication rules.
- **Layered diagnostics accelerate troubleshooting.** Using `ping` to validate Layer 3 reachability and `curl` to validate Layer 7 application connectivity provides a complete picture of network health at different protocol levels.
---
