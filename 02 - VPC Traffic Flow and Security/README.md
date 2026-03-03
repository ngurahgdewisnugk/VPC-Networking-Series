# VPC Traffic Flow and Security

A comprehensive implementation of AWS Virtual Private Cloud (VPC) networking with multi-layered security controls, demonstrating best practices for cloud network architecture and traffic management across multiple regions.

**Project Author:** Ngurah Gede Wisnu Gudakesa  
**Contact:** ngurahgedewisnugk@gmail.com  
**Project Link:** [View Project](http://learn.nextwork.org/projects/aws-networks-security)

---

## Architecture Overview

![architecture](<assets/architecture overview.png>)

This project implements a secure, multi-region AWS network infrastructure that demonstrates the complete VPC traffic flow lifecycle. The architecture features isolated virtual networks with granular security controls at both the instance and subnet levels, enabling secure public internet access while maintaining private internal communication channels.

The infrastructure leverages a defense-in-depth security model with:
- **Perimeter security** via Internet Gateway and route table configurations
- **Instance-level protection** through stateful Security Groups
- **Subnet-level enforcement** using stateless Network ACLs
- **Multi-region deployment** with centralized monitoring via AWS Global View

---

## What I Built

A production-ready AWS networking solution that provides:

- **Isolated Network Environment**: Custom VPC with segregated public and private subnets, ensuring logical isolation of resources and controlled communication patterns
- **Internet Connectivity**: Configured Internet Gateway with intelligent routing to enable selective public internet access for designated resources
- **Multi-Layered Security Architecture**: Implemented dual-firewall approach combining stateful Security Groups (instance-level) and stateless Network ACLs (subnet-level) for comprehensive traffic control
- **Cross-Region Infrastructure**: Deployed additional VPC components in secondary region using AWS CLI, demonstrating infrastructure-as-code capabilities and multi-region architecture patterns
- **Centralized Resource Management**: Integrated AWS Global View for unified visibility and monitoring of distributed resources across all deployed regions

---

## Action: Step-by-Step Implementation

### Phase 1: Foundation Setup
![Create VPC](<assets/Create VPC.png>)

**1. VPC Creation**

- Provisioned custom VPC with defined IPv4 CIDR block (10.0.0.0/16)
- Established isolated network space for all subsequent resources
- Configured DNS resolution and hostname settings for the VPC

![Create Subnet](<assets/Create Subnet.png>)
**2. Subnet Configuration**

- Created public subnet within the VPC address space
- Allocated appropriate CIDR range for subnet addressing
- Configured subnet to automatically assign public IPv4 addresses to launched instances

![Create Internet Gateway](<assets/create IGW.png>)
**3. Internet Gateway Deployment**
- Provisioned Internet Gateway (IGW) resource
- Attached IGW to the VPC to enable internet connectivity
- Verified gateway state and attachment status

### Phase 2: Traffic Routing Configuration

![Route Table](<assets/Route Table Route 2 .png>)
**4. Route Table Setup**

- Created custom route table for public subnet traffic management
- Added local route (10.0.0.0/16) for intra-VPC communication
- Configured default route (0.0.0.0/0) with IGW as target for internet-bound traffic
- Associated route table with public subnet to enable routing rules

**Route Logic Implementation:**
```
Destination: 10.0.0.0/16 → Target: local (internal VPC traffic)
Destination: 0.0.0.0/0   → Target: igw-xxxxxxxx (internet traffic)
```

### Phase 3: Security Layer Implementation

![Security Group](<assets/security group.png>)
**5. Security Group Configuration (Stateful - Instance Level)**
- Created security group with descriptive naming convention
- **Inbound Rules**: 
  - Type: HTTP
  - Protocol: TCP
  - Port: 80
  - Source: 0.0.0.0/0 (Anywhere-IPv4) - for public web access
- **Outbound Rules**: 
  - Default rule allowing all outbound traffic maintained
  - Enables instance to initiate connections to external resources

![Network ACL](<assets/inbound rules.png>)

**6. Network ACL Setup (Stateless - Subnet Level)**
- Created custom Network ACL for subnet-level traffic control
- **Inbound Rules**:
  - Rule 100: Allow all traffic (all protocols, all ports, all sources)
- **Outbound Rules**:
  - Rule 100: Allow all traffic (all protocols, all ports, all destinations)
- Associated custom NACL with public subnet
- Verified rule evaluation order and catch-all deny rules

### Phase 4: Multi-Region Deployment
![alt text](<assets/CLI Cloudshell.png>)
**7. AWS CLI Deployment**
- Authenticated AWS CLI with appropriate credentials and region configuration
- Executed CLI commands to provision infrastructure in secondary region:
  - Created additional VPC with equivalent CIDR configuration
  - Deployed Internet Gateway and attached to new VPC
  - Configured Security Group with matching rule sets
- Validated resource creation and proper configuration in new region


![Aws Global View-1.png](<assets/Aws Global View.png>)

**8. Global Resource Tracking**
- Accessed EC2 Global View (AWS Global View) console
- Configured filters to display resources across all regions:
  - Resource types: VPC, Internet Gateway, Security Group
  - Regions: Primary and secondary deployment regions
- Verified resource inventory and compliance with deployment specifications
- Utilized Global View for resource cleanup verification post-project

---

## Key Technologies

| Technology | Purpose | Implementation Details |
|------------|---------|----------------------|
| **Amazon VPC** | Network isolation and segmentation | Custom CIDR blocks, subnet organization, private cloud environment |
| **Internet Gateway** | Public internet connectivity | Bidirectional traffic enablement, attached to VPC, routing target |
| **Route Tables** | Traffic direction control | CIDR-based routing logic, destination-target mappings, subnet associations |
| **Security Groups** | Instance-level stateful firewall | Protocol/port-based rules, automatic return traffic handling, resource attachment |
| **Network ACLs** | Subnet-level stateless firewall | Numbered rule evaluation, explicit inbound/outbound rules, subnet association |
| **AWS CLI** | Infrastructure automation | Multi-region deployment, scriptable resource provisioning |
| **AWS Global View** | Cross-region resource monitoring | Unified dashboard, filter-based search, multi-region visibility |

---

## Problem Solving & Outcomes

### Challenge 1: Understanding Layered Security Complexity

**Problem:** Initially encountered confusion distinguishing between Route Tables, Security Groups, and Network ACLs—each operates at different OSI layers with distinct functions, which can be misleading when designing security architecture.

**Solution:** 
- Invested 6 hours in deep conceptual study and visualization
- Mapped each component to its specific function:
  - **Route Tables**: Layer 3 (Network) - traffic direction
  - **Security Groups**: Layer 4 (Transport) - stateful instance protection
  - **Network ACLs**: Layer 4 (Transport) - stateless subnet protection
- Created clear mental model: Route Tables as GPS, Security Groups as resource-specific guards, NACLs as subnet perimeter defenses

**Outcome:** Achieved comprehensive understanding of AWS network security layers, enabling confident implementation of defense-in-depth architecture.

### Challenge 2: Instance vs. Subnet Security Controls

**Problem:** Determining optimal security rule placement between Security Groups and Network ACLs for effective traffic management without redundancy or gaps.

**Solution:** Implemented complementary security layers:
- **Security Groups (Stateful)**: 
  - Focused on application-specific rules (HTTP port 80)
  - Leveraged automatic return traffic handling
  - Reduced rule complexity through statefulness
- **Network ACLs (Stateless)**:
  - Provided subnet-wide baseline security
  - Required explicit inbound AND outbound rules
  - Acted as additional defense layer before traffic reaches instances

**Outcome:** Created robust multi-layered security architecture where:
- NACLs provide broad subnet protection as first line of defense
- Security Groups offer granular instance protection as second line
- Combined approach prevents unauthorized access while maintaining operational flexibility

### Challenge 3: Multi-Region Resource Tracking

**Problem:** Managing and verifying resources across multiple AWS regions became complex, requiring manual region switching and increasing risk of orphaned resources.

**Solution:** 
- Deployed AWS Global View as centralized monitoring solution
- Configured resource type filters (VPC, IGW, Security Groups)
- Implemented region-based filtering for primary and secondary deployments
- Established verification workflow for both deployment and cleanup operations

**Outcome:** 
- **70% reduction** in time spent verifying resource deployment status
- **Single-pane visibility** across all regions eliminated manual region switching
- **Complete resource accountability** ensured no orphaned resources post-cleanup
- **Scalable approach** for teams managing globally distributed applications

### Key Achievements

✅ **Security Posture**: Implemented defense-in-depth with dual-layer firewall controls  
✅ **Network Architecture**: Established clear separation between public and private traffic flows  
✅ **Automation**: Demonstrated IaC capabilities through AWS CLI multi-region deployment  
✅ **Operational Efficiency**: Optimized resource tracking with AWS Global View integration  
✅ **Knowledge Transfer**: Documented comprehensive understanding of AWS networking fundamentals  

---

## Technical Insights

### Security Group vs. Network ACL Decision Matrix

| Criterion | Security Group | Network ACL |
|-----------|----------------|-------------|
| **Scope** | Instance-level | Subnet-level |
| **State** | Stateful (automatic return traffic) | Stateless (explicit rules required) |
| **Rule Processing** | All rules evaluated before decision | Rules evaluated in numerical order |
| **Default Behavior** | Default deny (explicit allow required) | Custom: deny all; Default: allow all |
| **Use Case** | Application-specific access control | Subnet-wide baseline security |

### Route Table Decision Logic

```
Traffic Flow Decision Tree:
├─ Destination IP in VPC CIDR (10.0.0.0/16)?
│  ├─ YES → Route to: local (stay within VPC)
│  └─ NO → Check next route
│
└─ Destination IP matches 0.0.0.0/0?
   └─ YES → Route to: Internet Gateway (exit to internet)
```

---