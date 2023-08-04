---
title: AWS EC2 Jumphost Setup
layout: post
post-image: "/assets/images/blog/aws_logo.svg"
description: Set up Jumphost/Bastion to access private subnets from Internet
tags:
- aws
- ec2
- jumphost
- bastion
- blog
- intra-vpc routing
---

# Problem Statement
<br>
<br>
![AWS Jumphost/Bastion Diagram](/assets/images/blog/aws_ec2_jumphost_bastion.drawio.svg "AWS Jumphost/Bastion Diagram")
<br>
<br>
- Jumphosts/Bastions can be accessible from Internet via EC2 Instance Connect and SSH
- They can be used to access EC2 instances in private subnets via SSH

# Implementation

## Infrastructure

### VPC
Create the VPC with the following settings:
- `DNS hostnames`: `Enabled`
- `DNS resolution`: `Enabled`
- `Main network ACL`: `VPC-NACL`
- `IPv4 CIDR`: `172.16.0.0/16`

### Network ACL

#### Inbound Rules

| Type | Protocol | Port range | Source | Allow/Deny | Comment                               |
| ---- | -------- | ---------- | ------ | ---------- | ------------------------------------- |
| HTTP | TCP(6)   | 80         | 0.0.0.0/0 | Allow   | To test static webpage on jumphosts   |
| HTTPS | TCP(6)   | 443        | 0.0.0.0/0 | Allow   | To test static webpage on jumphosts  |
| SSH | TCP(6)   | 22         | 0.0.0.0/0 | Allow   | To access jumphosts and EC2 instances in private subnets |
| Custom TCP | TCP(6)   | 1024 - 65535 | 0.0.0.0/0 | Allow   | To allow returning traffic of outbound traffic from EC2 instances like `sudo yum update -y`|

#### Outbound Rules

| Type | Protocol | Port range | Source | Allow/Deny | Comment                               |
| ---- | -------- | ---------- | ------ | ---------- | ------------------------------------- |
| All traffic | All   | All         | 0.0.0.0/0 | Allow   | Allow outbound                   |

### Subnets

Create two public subnets `Public Subnet 1` and `Public Subnet 2`

Create two private subnets `Private Subnet 1` and `Private Subnet 2`

### Route Tables

Create `Public RT` for the two public subnets with two routes:

- Route for destination `0.0.0.0/0` via target Internet Gateway
- Route for destination `172.16.0.0/16` via target local

`Private RTs` one for each private subnet is created by default

### Security Groups

#### Public Security Group

Create security group `sg-pubsub` for public subnets with the following rules:

**Inbound**

| Type | Protocol | Port range | Source | Allow/Deny | Comment                               |
| ---- | -------- | ---------- | ------ | ---------- | ------------------------------------- |
| HTTP  | TCP     | 80         | 0.0.0.0/0 | Allow   | Allow access to static webpage on jumphost |
| SSH  | TCP      | 22         | 0.0.0.0/0 | Allow   | Allow SSH to jumphosts                |

**Outbound**

| Type         | Protocol | Port range | Source    | Allow/Deny | Comment                               |
| -----------  | -------- | ---------- | --------- | ---------- | ------------------------------------- |
| All traffic  | All      | All        | 0.0.0.0/0 | Allow      | Allow access Internet                 |

#### Public Security Group

Create security group `sg-pubsub` for public subnets with the following rules:

**Inbound**

| Type | Protocol | Port range | Source | Allow/Deny | Comment                               |
| ---- | -------- | ---------- | ------ | ---------- | ------------------------------------- |
| SSH  | TCP      | 22         | 0.0.0.0/0 | Allow   | Allow SSH from jumphosts                |

**Outbound**

| Type         | Protocol | Port range | Source    | Allow/Deny | Comment                               |
| -----------  | -------- | ---------- | --------- | ---------- | ------------------------------------- |
| All traffic  | All      | All        | 0.0.0.0/0 | Allow      | Allow access Internet                 |

## EC2 Instance Deployment and Setup

### Jumphost/Bastion

#### Deployment

This machine is deployed as follows:

- Assign a public IP address for SSH and web inbound access
- Assign a private IP address in the public subnet for the SSH access to the other EC2 instances in the private networks
- Create a key pair for SSH access

#### Setup

Connect to Bastion using `EC2 Instance Connect` or `SSH client` as follows:
```
1. Open an SSH client
2. Locate your private key file. The key used to launch this instance is <key_pair_name>.pem
3. Run this command, if necessary, to ensure your key is not publicly viewable.
   chmod 400 <key_pair_name>.pem
4. Connect to your instance using its Public DNS or IP address:
   ssh -i "<key_pair_name>.pem" ec2-user@<Public IP>
5. Once the SSH session is established, open <key_pair_name>.pem on the SSH client machine, copy and paste its content to .ssh/<key_pair_name>.pem on the Bastion
```

### EC2 Instances

These machines are deployed as follows:

- Assign a private IP address in the private subnet for the SSH access from the Bastion
- Re-use the same key pair for SSH access

After that confirm that these instances are accessible via SSH from the Bastion.

# Appendix

## AWS Routing

### Routing between subnets inside of a local VPC

By default, each VPC comes with 1 route table pre-configured with a “local” route. The scope of the “local” route is only within the subnet defined for the entire VPC. For example, if your VPC was set up to have the address space of 172.16.0.0/16, your “local” route would be defined as “172.16.0.0/16”. This allows all of the resources created within the VPC to talk to each other without any additional configuration. You cannot delete the “local” route from your route table, and anytime a new route table is created within a VPC, the “local” route is included by default.

Within a VPC, route tables are assigned to individual subnets. With only 1 route table created in a VPC, all of the subnets would be assigned to that route table. You can create multiple route tables in a VPC, or you can leave the 1 default route table.

### 2-tier routing tables (1 for public subnets, and 1 for private subnets)

For environments that are broken apart into public and private subnets, it is best practice to have 1 route table for public subnets, and 1 route table for private subnets.
In this scenario, you will need to further separate your route tables into availability zones, as well. The reason for this is: you will want systems in AZ A to use a NAT gateway and any other systems which are also within that AZ; same for systems in AZ B. This ensures that if one availability zone becomes unavailable, the systems in the other availability zone are not relying on a system that is now non-functional. The diagram below shows this configuration.

# References

1. [AWS Routing 101](https://medium.com/@mda590/aws-routing-101-67879d23014d)