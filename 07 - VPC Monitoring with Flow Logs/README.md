<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# VPC Monitoring with Flow Logs

**Project Link:** [View Project](http://learn.nextwork.org/projects/aws-networks-monitoring)

**Author:** Ngurah Gede Wisnu Gudakesa  
**Email:** ngurahgedewisnugk@gmail.com

---

## VPC Monitoring with Flow Logs

![Image](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-monitoring_3e1e79a1)

---

## Introducing Today's Project!

### What is Amazon VPC?

An Amazon VPC is an isolated virtual network within AWS that you fully control. It's incredibly useful because it allows your resources to communicate privately within the VPC, securely connect with other VPCs using peering connections, and safely access the public internet when needed.

### How I used Amazon VPC in this project

In this project, I leveraged Amazon VPC to build a robust network monitoring solution. I started by creating two isolated VPC environments using the VPC Resource Map and launched EC2 instances within them. To enable secure communication between these environments, I established a VPC Peering Connection. The core of the project involved setting up VPC Flow Logs to capture all network traffic, granting necessary permissions via IAM policies and roles to send these logs to CloudWatch. Finally, I used CloudWatch Logs Insights to analyze the collected flow logs, gaining valuable insights into network activity and troubleshooting communication patterns.

### One thing I didn't expect in this project was...

One thing I didn't expect in this project was how crucial a deep understanding of CloudWatch Log Groups is for effective network monitoring. It's not just about collecting logs, but about how they are organized within specific groups, which then directly enables the powerful visual insights we can gain using CloudWatch Logs Insights. Realizing how these visual insights can be tailored to our needs was particularly impressive and impactful.

### This project took me...

This project took me 8 hours, and I dedicated that time to thoroughly understanding each topic's concepts. I've noticed that with each project series, the field becomes more complex, truly requiring a commitment to in-depth study.

---

## In the first part of my project...

### Step 1 - Set up VPCs

In this step, I will create two VPCs from scratch using the VPC wizard. This approach helps reduce setup time and minimize manual configuration, making the process faster and less prone to errors. We are setting up two VPCs to also revise some learnings from the VPC peering project.

### Step 2 - Launch EC2 instances

In this step, I will launching an EC2 instance in each VPC. These instances are essential for two reasons: first, they will generate the network activity that our VPC Flow Logs will monitor, and second, they will be used to test the VPC peering connection between the two VPCs later in the project. I’m also configuring them to have public IPv4 addresses and setting up their security groups to allow ICMP traffic for my connectivity tests.

### Step 3 - Set up Logs

In this step, I will set up VPC Flow Logs to track all inbound and outbound network traffic within my VPC, and store these records for analysis.

### Step 4 - Set IAM permissions for Logs

In this step, I will create an IAM policy to define the permissions VPC Flow Logs needs to write logs to CloudWatch. Then, I will create an IAM role with a custom trust policy, attach the newly created IAM policy to it, and finally assign this role to VPC Flow Logs to enable log delivery. This also allows me to finish setting up the subnet's flow log.

---

## Multi-VPC Architecture

In this step, I launched two isolated VPCs (NextWork-1 and NextWork-2) using the VPC wizard's "VPC and more" option. Each VPC has a unique IPv4 CIDR block (10.1.0.0/16 and 10.2.0.0/16 respectively). For each VPC, I created one public subnet, one route table, and one Internet Gateway. In total, I created two subnets.


The IPv4 CIDR blocks for VPCs 1 and 2 are unique because overlapping IP addresses between them would cause routing conflicts and connectivity issues for their resources.

### I also launched EC2 instances in each subnet

My EC2 instances' security groups were configured with an inbound rule to allow All ICMP - IPv4 traffic from 0.0.0.0/0 (all IP addresses). This was necessary to ensure that the upcoming ping test, which requires ICMP traffic to be allowed from any source, would function correctly.

![Image](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-monitoring_e7fa8775)

---

## Logs

Logs are like a diary for your computer systems, recording everything that happens within an operating system or application. They are crucial for security audits, helping to gather information about system activities, and are essential for troubleshooting problems to meet service-level agreements.

Log groups are like big folders in AWS CloudWatch where you keep related log data together. Logs are region-specific, meaning they are created and stored in the region where they originate. However, CloudWatch dashboards can be used to bring together and view logs from different regions.

### I also set up a flow log for VPC 1

![Image](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-monitoring_e8398869)

---

## IAM Policy and Roles

I created an IAM policy because VPC Flow Logs need explicit permission to send network traffic data to CloudWatch Log Groups. This policy allows Flow Logs to create log groups and streams, and then put log events into them, ensuring our network activity is recorded.

I created an IAM role to assign the permissions defined in my IAM policy to VPC Flow Logs. The custom trust policy ensures that only VPC Flow Logs can assume this role, giving it the specific actions it needs (like sending logs to CloudWatch) while maintaining strong security.

A custom trust policy is define who can assume or use this role, In this case, it specifically allows only VPC Flow Logs to use the role that I'm creating. This is a crucial security measure to prevent other services from accidentally or maliciously using the role.

//please review my answer, is it relatable with questions on Step 4 : Set Up a Flow Log IAM Policy and Role. "What is a custom trust policy?" If there is something part missing or out-of-topic, please correct it and simplify.

![Image](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-monitoring_4334d777)

---

## In the second part of my project...

### Step 5 - Ping testing and troubleshooting

In this step, I will test the VPC peering connection by having Instance 1 send test messages to Instance 2. This will ensure that communication is established between the two isolated VPC environments.

### Step 6 - Set up a peering connection

In this step, I will set up a VPC Peering Connection and configure route tables to enable direct communication between VPC 1 and VPC 2 using their private IP addresses. This ensures traffic can flow securely between them.

### Step 7 - Analyze flow logs

In this step, I will deep dive into VPC Flow Logs by using CloudWatch Logs Insights to analyze recorded network activity. My goal is to review the flow logs to understand traffic patterns and identify key insights, such as the top data transfers between IP addresses.

---

## Connectivity troubleshooting

My first ping test between my EC2 instances had no replies with Private Ipv4 Address, which means there's a problem with the connection.

![Image](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-monitoring_99d4ba42)

Receiving ping replies when using the other instance's public IP address means that Instance 2 is configured to respond to ping requests, and Instance 1 can communicate with it, but this communication is happening over the public internet. This confirms that the instances themselves are reachable, but not necessarily through a private peering connection.

---

## Connectivity troubleshooting

The ping test using Instance 2's private address failed because there was no VPC peering connection established between VPC 1 and VPC 2. Without this direct connection, traffic destined for VPC 2's private IP addresses was routed through the Internet Gateway via the public internet, instead of directly between the VPCs. This exposes the traffic publicly and prevents private IP communication.

### To solve this, I set up a peering connection between my VPCs

I updated both VPCs' route tables to explicitly direct traffic destined for the other VPC through the newly established VPC peering connection. Without these new routes, the VPCs wouldn't know to use the peering connection, and traffic between them wouldn't be able to flow using private IP addresses.

![Image](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-monitoring_7316a13d)

---

## Connectivity troubleshooting

The successful ping replies from Instance 2's private IP address indicate that the VPC peering connection is correctly configured. This allows VPC 1 and VPC 2 to communicate directly using their private IP addresses, with traffic routed effectively by each VPC's route table.

![Image](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-monitoring_4ec7821f)

---

## Analyzing flow logs

Flow logs tell meabout network traffic, including the amount of data (in bytes) transferred, the source and destination IP addresses, the protocol used, the destination port, the number of packets transferred, and the status of the traffic (whether it was accepted or rejected).

For example, the flow log I've captured tells me about specific network traffic, including whether it was accepted or rejected. It details the amount of data transferred (in bytes), the source IP address, the destination IP address, the protocol used, the port, and the number of packets transferred.

![Image](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-monitoring_d116818e)

---

## Logs Insights

Logs Insights is a CloudWatch feature that analyzes network logs. It uses queries to filter, process, and combine data, which helps you troubleshoot problems and better understand your network traffic, especially for the network resources in this project.

I ran the query “Top 10 byte transfers by source and destination IP addresses”. This query helps discover the top 10 biggest data transfers between IP addresses in my network, which uncovers the busiest traffic routes.

![Image](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-monitoring_3e1e79a1)

---

---
