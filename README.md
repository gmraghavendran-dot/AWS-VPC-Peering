AWS Intra-Region VPC Peering & Security Infrastructure

📌 Executive Summary

This project demonstrates the design, deployment, security validation, and complete decommissioning of an intra-region, cross-VPC architecture within AWS Region ap-south-1 (Mumbai).

The infrastructure establishes secure, private IPv4 communication between two isolated Virtual Private Clouds (ewb-vpc-a and ewb-vpc-b) across the internal AWS backbone without routing traffic through the public internet. Administrative access is managed using SSH ProxyJump through a public Bastion host, keeping private application servers completely isolated without public IPv4 addresses.

👨‍💻 Project & Student Information

Student Name: Raghavendran G M

AICTE Student ID: STUc171e6fe139bf1789379999

Program: AWS Cloud Computing Intern

Batch & Mode: Batch 1 (Virtual)

Offered By: EWB Edutech Private Limited

AWS Region: Mumbai (ap-south-1)

Completion Date: October 24, 2026

📐 Architecture & Network Plan

                   AWS REGION: ap-south-1 (Mumbai)
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│   ┌───────────────────────────┐      SSH ProxyJump Path                 │
│   │   LOCAL MACHINE / LAPTOP  │ - - - - - - - - - - - - - ┐             │
│   └─────────────┬─────────────┘                           │             │
│                 │ SSH (22)                                │             │
│                 ▼                                         ▼             │
│   ┌──────────────────────────────────────────┐  ┌───────────────────┐  │
│   │ VPC A: ewb-vpc-a (10.10.0.0/16)          │  │ VPC B: ewb-vpc-b  │  │
│   │                                          │  │ (10.20.0.0/16)   │  │
│   │  ┌────────────────────────────────────┐  │  │                   │  │
│   │  │ Public Subnet: ewb-a-public        │  │  │ ┌───────────────┐ │  │
│   │  │ (10.10.1.0/24)                     │  │  │ │Private Subnet │ │  │
│   │  │  [EC2] ewb-bastion                 │  │  │ │ewb-b-private  │ │  │
│   │  │  IP: 10.10.1.10 (Pub: 3.111.196.90)│  │  │ │(10.20.1.0/24)  │ │  │
│   │  └──────────────────┬─────────────────┘  │  │ │               │ │  │
│   │                     │                    │  │ │ [EC2]         │ │  │
│   │  ┌──────────────────▼─────────────────┐  │  │ │ ewb-app-b     │ │  │
│   │  │ Private Subnet: ewb-a-private      │  │  │ │ IP: 10.20.1.10│ │  │
│   │  │ (10.10.2.0/24)                     │  │  │ │ (No Public IP)│ │  │
│   │  │  [EC2] ewb-app-a                   │  │  │ └───────▲───────┘ │  │
│   │  │  IP: 10.10.2.10 (No Public IP)     │  │  │         │         │  │
│   │  └──────────────────┬─────────────────┘  │  └─────────┼─────────┘  │
│   └─────────────────────┼────────────────────┘            │            │
│                         │        VPC Peering Connection   │            │
│                         └───────────► pcx-... ◄───────────┘            │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘


Resource Inventory & Addressing Scheme

Resource Name

Type

Resource ID

IPv4 CIDR / IP

Notes

ewb-vpc-a

VPC

vpc-0c8a1eb58534143f4

10.10.0.0/16

Administrative & App VPC

ewb-a-public

Public Subnet

subnet-0c8b6afd632a13049

10.10.1.0/24

Hosts Bastion Server

ewb-a-private

Private Subnet

subnet-00920c6b4f6f7272e

10.10.2.0/24

Hosts App Server A

ewb-vpc-b

VPC

vpc-0929404d783f5b4df

10.20.0.0/16

Workload VPC

ewb-b-private

Private Subnet

subnet-0df374ef519f84b01

10.20.1.0/24

Hosts App Server B

ewb-igw-a

Internet Gateway

igw-0604da1c62268fb6a

N/A

Attached to ewb-vpc-a

ewb-a-b-peer

VPC Peering

pcx-0a404ca9b9d4cb5b3

N/A

Active intra-region link

ewb-bastion

EC2 Instance

i-044063495f2a1c410

10.10.1.10 / 3.111.196.90

Public Entry Point

ewb-app-a

EC2 Instance

i-0475a8621a443524b

10.10.2.10

Isolated Application Host

ewb-app-b

EC2 Instance

i-0ca2fda18cf4b3b0d

10.20.1.10

Isolated Application Host

⚙️ Routing & Security Group Rules

1. Route Table Configurations

ewb-rt-a-public (rtb-0fdd0e7393f3e2036):

0.0.0.0/0 ➔ igw-0604da1c62268fb6a (Internet Access)

10.10.0.0/16 ➔ local

10.20.0.0/16 ➔ pcx-0a404ca9b9d4cb5b3 (Bastion-to-VPC B routing)

ewb-rt-a-private (rtb-093782ba5cbb0a4f0):

10.10.0.0/16 ➔ local

10.20.0.0/16 ➔ pcx-0a404ca9b9d4cb5b3 (Forward path to VPC B)

ewb-rt-b-private (rtb-0ac9b53da56b4cc9c):

10.20.0.0/16 ➔ local

10.10.0.0/16 ➔ pcx-0a404ca9b9d4cb5b3 (Return path to VPC A)

2. Inbound Security Group Rules

ewb-sg-bastion (sg-0404695246aba0328):

SSH (22) ➔ My-Public-IP/32

ewb-sg-app-a (sg-00dbabc32af40a3e7):

SSH (22) ➔ 10.10.1.10/32 (Bastion Private IP)

Custom ICMP Echo Request ➔ 10.20.0.0/16 (VPC B Subnet)

ewb-sg-app-b (sg-027b07feaadc4fdf8):

SSH (22) ➔ 10.10.1.10/32 (Bastion Private IP)

Custom ICMP Echo Request ➔ 10.10.0.0/16 (VPC A Subnet)

🔄 Project Implementation Workflow

Network Provisioning: Created two dual-subnet VPCs with non-overlapping IPv4 CIDRs (10.10.0.0/16 and 10.20.0.0/16). Attached an Internet Gateway to VPC A.

Compute Deployment: Launched three Amazon Linux 2023 EC2 instances with static private IPs. Configured private hosts with zero public IPv4 addresses.

Peering Setup: Created the peering request (ewb-a-b-peer) from VPC A to VPC B and accepted the request to activate the peering link (pcx-0a404ca9b9d4cb5b3).

Symmetric Routing: Configured forward routes in VPC A and return routes in VPC B pointing to the peering connection target.

Zero-Trust Client Access: Configured local OpenSSH ~/.ssh/config using ProxyJump ewb-bastion to securely reach private hosts without embedding private keys on intermediate servers.

Validation & Failure Testing:

Baseline Test: Confirmed ICMP failures prior to configuring cross-VPC routes.

Bidirectional Reachability: Verified 0% packet loss between ewb-app-a and ewb-app-b.

Controlled Failure: Removed ICMP rules in ewb-sg-app-b to observe stateful drop (100% packet loss).

Recovery: Restored security rules and confirmed full connectivity (0.178 ms average latency).

Teardown & Cost Auditing: Terminated all compute instances, removed peering links, route tables, subnets, and VPCs, verifying a $0.00 total balance in AWS Billing.

🔑 Key Engineering Takeaways

Private Backbone Routing: AWS VPC Peering routes internal network traffic securely across the AWS global backbone without traversing the public internet.

Symmetric Network Requirement: Cross-VPC traffic requires explicit destination routes in both the requester and accepter route tables for bidirectional packet delivery.

Stateful Firewall Enforcement: Security Groups operate at the Elastic Network Interface (ENI) level; dropping unauthorized traffic even if VPC route targets are active.

Zero-Trust SSH Bastion: Utilizing ProxyJump allows administrators to reach isolated private workloads securely without copying .pem private keys to public jump servers.

Cloud FinOps Practice: Decommissioning dependencies in strict order prevents lingering billable allocations.

📄 License & Acknowledgments

This project was developed as part of the EWB Edutech AWS Cloud Computing Virtual Internship Program in alignment with AICTE curriculum standards.
