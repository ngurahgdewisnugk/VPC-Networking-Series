# Secure S3 Access from an Isolated AWS VPC via EC2

A hands-on AWS networking project that provisions an isolated Amazon VPC with a public-subnet EC2 instance, then securely authenticates that instance to Amazon S3 using IAM access keys and the AWS CLI — demonstrating how compute resources inside a private network boundary can safely interact with managed storage services that live outside it.

**Project Author:** [View Project](http://learn.nextwork.org/projects/aws-networks-s3)</br>
**Author:** Ngurah Gede Wisnu Gudakesa  
**Email:** ngurahgedewisnugk@gmail.com

---

## Architecture Overview
![Architecture Overview](<Assets/Architecture Overview.png>)

1. The infrastructure begins with a custom Amazon VPC containing a public subnet. An EC2 instance is launched into this subnet with **auto-assigned public IP** enabled, and a dedicated **Security Group** restricts inbound access to SSH only, allowing connection through EC2 Instance Connect. 
2. Separately, an Amazon S3 bucket is provisioned outside the VPC boundary and seeded with sample objects. 
3. Traffic flows from the EC2 instance, over the AWS network, to the S3 service endpoint via authenticated AWS CLI API calls — there is no direct network-layer connection between the VPC and S3; access is governed entirely by IAM credential authentication.
4. The security model applied here centers on **credential-based authentication** rather than network-level trust: the EC2 instance proves its identity to AWS using an Access Key ID and Secret Access Key configured through `aws configure`, rather than relying on implicit network access. 
5. This mirrors a defense-in-depth posture — network isolation (VPC + Security Group) restricts who can reach the instance, while IAM credentials independently govern what that instance can do once it's running. 
6. The project also surfaces the tradeoff between this access-key approach and the AWS-recommended best practice of attaching an **IAM Role** directly to the EC2 instance, which removes the need to manage long-lived static credentials altogether.

---
 
## What I Built
 
A production-style AWS networking and storage integration that provides:
 
* **Isolated Network Foundation**: A custom **Amazon VPC** with a public subnet, giving full control over IP addressing and traffic flow for hosted resources.
* **Controlled Compute Access**: An **EC2 instance** with a public IP and a tightly scoped **Security Group** permitting only SSH traffic, accessed via **EC2 Instance Connect**.
* **External Storage Integration**: An **Amazon S3 bucket** provisioned outside the VPC, populated with objects and managed entirely through the **AWS CLI**.
* **Credential-Based Authentication**: Programmatic access configured via **IAM Access Keys** (Access Key ID + Secret Access Key), enabling secure, auditable communication between EC2 and S3.

---

## Technology Stack
 
| Category | Service / Tool |
|---|---|
| Networking | Amazon VPC, Public Subnet, Security Groups |
| Compute | Amazon EC2, EC2 Instance Connect |
| Storage | Amazon S3 |
| Identity & Access | IAM Access Keys |
| Tooling | AWS CLI |

---

## Action: Step-by-Step Implementation

### Step 1 — VPC & EC2 Architecture Setup

In this step, I will create a VPC from scratch and launch an EC2 instance into its public subnet. I'm doing this to establish an isolated network environment and prepare to connect with external AWS services. Additionally, I will set up an Amazon S3 bucket and upload files, which my EC2 instance will interact with.

![VPC Set Up](<Assets/01 - VPC Set Up.png>)

### Step 2 - Connecting to the EC2 Instance

Connected to the running EC2 instance using **EC2 Instance Connect** to gain a terminal session for running AWS CLI commands.

### Step 3 - Setting Up IAM Access Keys

Generated an IAM **Access Key ID** and **Secret Access Key** to serve as the credentials the EC2 instance would use to authenticate against AWS services.

![Set Up IAM Access Key](<Assets/03 - set up IAM Access Key.png>)

### Step 4 — S3 Bucket Setup

1. Provisioned an Amazon S3 bucket (`nextwork-vpc-project-wsworskpace`) 
   ![Set Up S3 Buckets](<Assets/04 - Set Up S3 Buckets.png>)

2. And uploaded two local files as objects, preparing the storage target the EC2 instance would later interact with.
   ![Image](http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-s3_4334d777)

### Step 5 — Configuring the AWS CLI

Ran `aws configure` on the EC2 instance and supplied the Access Key ID, Secret Access Key, and default region (output format left blank), giving the instance authenticated access to the AWS account.

![aws configure](<Assets/05 - aws configure.png>)

### Step 6 — Validating & Interacting with S3
Used the AWS CLI to confirm connectivity, list bucket contents, and upload a new object, validating that the EC2 instance could fully read from and write to S3.
 
<table>
    <thead>
        <tr>
            <th>#</th>
            <th>Command</th>
            <th>Purpose</th>
            <th>Image</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>1</td>
            <td><code>aws s3 ls</code></td>
            <td>
                Lists all S3 buckets in the account — first run confirmed CLI installation;
                second run (post-configuration) confirmed successful authentication.
            </td>
            <td>
                <img src="Assets/terminal aws s3 response.png" alt="aws s3 ls">
            </td>
        </tr>
        <tr>
            <td>2</td>
            <td><code>aws s3 ls s3://nextwork-vpc-project-wsworskpace</code></td>
            <td>
                Lists objects inside the target bucket, confirming the two previously
                uploaded <code>.png</code> files were visible.
            </td>
            <td>
                <img src="http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-s3_4334d779" alt="Bucket Contents">
            </td>
        </tr>
        <tr>
            <td>3</td>
            <td><code>sudo touch /tmp/test.txt</code></td>
            <td>
                Creates a blank local test file on the EC2 instance.
            </td>
            <td rowspan="3">                
                <img src="http://learn.nextwork.org/motivated_teal_shy_hog/uploads/aws-networks-s3_3e1e79a2" alt="Create Test File">
            </td>
        </tr>
        <tr>
            <td>4</td>
            <td><code>aws s3 cp /tmp/test.txt s3://nextwork-vpc-project-wsworskpace</code></td>
            <td>
                Uploads the local file to the S3 bucket as a new object.
            </td>
        </tr>
        <tr>
            <td>5</td>
            <td><code>aws s3 ls s3://nextwork-vpc-project-wsworskpace</code></td>
            <td>
                Re-lists bucket contents to validate that the new upload succeeded.
            </td>
        </tr>
    </tbody>
</table>

---

## Problem Solving & Key Learnings
 
**Challenge:** Enabling an EC2 instance — which lives inside an isolated VPC — to securely reach Amazon S3, a service that exists entirely outside that network boundary.

![unable credential](<Assets/aws cli credential.png>)
 
**Resolution path:**
 
```
VPC Isolation (default: no AWS access)
        │
        ▼
Generate IAM Access Key + Secret Key
        │
        ▼
Run `aws configure` on EC2 instance
        │
        ▼
CLI authenticates each S3 API call
        │
        ▼
Full read/write access to S3 confirmed (aws s3 ls / cp)
```
The unexpected learning was discovering that **access keys are just one of several valid authentication mechanisms** for AWS CLI — and not always the best one. While access keys solved the immediate problem and unblocked the EC2-to-S3 connection in one configuration pass, this project highlighted that **IAM Roles** are the AWS-recommended alternative for EC2-based workloads, since they eliminate the operational risk of storing static, long-lived credentials directly on an instance.
 
| Approach | Credential Lifetime | Operational Overhead | AWS Recommendation |
|---|---|---|---|
| **IAM Access Keys** (used in this project) | Long-lived, static | Manual rotation required | Acceptable for learning/testing |
| **IAM Role attached to EC2** | Temporary, auto-rotated | None — managed by AWS | ✅ Best practice for production |

---

## Key Takeaways
 
* Successfully isolated compute resources in a custom VPC while still enabling secure, authenticated access to external AWS services like S3.
* Gained hands-on fluency with core AWS CLI commands (`ls`, `cp`, `configure`) for managing S3 from the command line.
* Understood the practical difference between **access keys** and **IAM roles**, and why roles are the preferred credential strategy for EC2-to-AWS-service communication in production environments.


---

---
