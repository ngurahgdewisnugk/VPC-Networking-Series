# Creating a Private Subnet

This project is about setting up a **secure, isolated section of a cloud network** inside AWS — called a **private subnet**.

Think of it like building a locked back room inside a building. The front entrance (public subnet) is open to visitors, but the back room (private subnet) only allows people already inside the building to access it. No strangers can walk in directly from the street (the internet).

The goal of this project is to configure AWS so that sensitive resources (like databases) are protected from direct internet access, while still being reachable by trusted internal systems.

**Project Author:** Ngurah Gede Wisnu Gudakesa  
**Contact:** ngurahgedewisnugk@gmail.com  
**Project Link:** [View Project](http://learn.nextwork.org/projects/aws-networks-private)

---

## Architecture Overview

![architecture](<assets/architecture overview.png>)

The infrastructure leverages a defense-in-depth security model with:
- Public traffic from the internet flows through the **Internet Gateway → Public Route Table → Public Subnet**
- The **Private Subnet has no internet gateway route**, so external traffic cannot reach it directly
- Internal VPC traffic (e.g., from the public subnet to the private subnet) passes through the **Private Route Table → Network ACL → Private Subnet**, subject to ACL rules

---

## 🏗️ What I Built

A secure, isolated AWS network infrastructure featuring:

- A **Virtual Private Cloud (VPC)** as the overarching private network boundary (`10.0.0.0/16`)
- A **Private Subnet** (`10.0.1.0/24`) dedicated to hosting sensitive, internal-only resources — isolated from the public internet
- A **Public Subnet** (`10.0.0.0/24`) for resources that need internet connectivity
- A **Dedicated Private Route Table** that routes traffic only within the VPC — with no route to an internet gateway
- A **Custom Network ACL (Access Control List)** for the private subnet that denies all inbound and outbound traffic by default, enforcing a strict security posture

All resources were provisioned in the **Asia Pacific (Sydney) / `ap-southeast-2b`** Availability Zone.

---

## Action: Step-by-Step Implementation

### Phase 1: Creating a Private Subnet
![Private Subnet ](<assets/private subnet.png>)

- Private Subnet: A private subnet does not have direct internet access and is used for internal resources that don't need to be publicly accessible, such as databases storing sensitive data.
- Private subnets exist to host sensitive resources, like databases containing customer data, securely away from direct internet access. They allow internal resources, such as web servers in a public subnet, to communicate with these private resources in a controlled manner, enhancing security by preventing unauthorized external access.

### Phase 2: Creating a dedicated route table 
![private rtb](<assets/nextwork private route table.png>)

- By default, my private subnet is associated with public route table that connected to internet gateway, which means this settings can accessly public, this is not recommended because I'm gonna setting for private subnet for sensitive resources. 

- I’m setting up a new route table because the existing default route table for my VPC has a route to an internet gateway. If my private subnet were associated with that default route table, it would inadvertently become public, which defeats the purpose of a private subnet.

- My private subnet's dedicated route table only has one default route that allows traffic within the 10.0.0.0/16 CIDR block to communicate with local resources under the VPC.

### Phase 3: Security Layer Implementation (network ACL)
![Network ACL](<assets/NACL inbound rules.png>)

- By default, my private subnet is associated with the default network ACL for the VPC. This default ACL allows all inbound and outbound traffic, which exposes my private subnet to unrestricted access from the internet or other untrusted networks.

- I set up a dedicated network ACL for my private subnet because the default network ACL allows all traffic, which exposes my private subnet to unrestricted access. A new custom network ACL starts by denying all inbound and outbound traffic, providing a stronger security posture to restrict unwanted traffic and protect my private resources.

- My new network ACL, by default, has rules that deny all inbound and outbound traffic to my private subnet.

---

## Key Technologies

| Technology | Purpose | Implementation Details |
|------------|---------|----------------------|
| **Amazon VPC** | Network isolation and segmentation | Custom CIDR blocks, subnet organization, private cloud environment |
| **Internet Gateway** | Public internet connectivity | Bidirectional traffic enablement, attached to VPC, routing target |
| **Route Table (Private)** | A route table is a set of rules (routes) that determine where network traffic from a subnet is directed. | By default, a new subnet inherits the VPC's main route table — which in this project already had a route pointing to an internet gateway. Associating the private subnet with that default table would have accidentally made it publicly accessible, defeating its entire purpose. A **dedicated private route table** was created with only one route: local traffic within the `10.0.0.0/16` CIDR block. This ensures no traffic from the private subnet can leak out to the internet.|
| **Security Groups** | Instance-level stateful firewall | Protocol/port-based rules, automatic return traffic handling, resource attachment |
| **Network ACLs** | A Network ACL is a stateless firewall that controls inbound and outbound traffic at the subnet level. | The VPC's **default Network ACL allows all traffic**, which would expose the private subnet to unrestricted access. A **custom Network ACL** was created specifically for the private subnet. Unlike the default, a new custom ACL starts by **denying all inbound and outbound traffic**, giving a secure baseline. Specific rules can then be added selectively to allow only the traffic that's explicitly needed. |
| **Subnets (Public & Private)** | A subnet is a subdivision of the VPC's IP address space, scoped to a specific Availability Zone. | Subnets let you segment your network by purpose. The **public subnet** (`10.0.0.0/24`) handles resources that need internet access. The **private subnet** (`10.0.1.0/24`) isolates sensitive resources — like databases — away from direct internet exposure. Each subnet must have a **unique CIDR block** to prevent IP address conflicts and ensure traffic is correctly routed between them. |

---

### Outcome

The result is a properly segmented AWS network where:
- Public-facing resources sit in an internet-accessible subnet
- Sensitive internal resources are isolated in a private subnet with no direct internet route
- All traffic into and out of the private subnet is blocked by default, with access granted only through explicit ACL rules

> ⏱️ This project took approximately **3 hours**, including time spent studying the underlying networking concepts.

---
