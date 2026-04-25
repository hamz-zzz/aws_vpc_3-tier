# aws_vpc-3-tier

## Architecture Diagram

![Architecture](architecture/3-Tier-Architecture.svg)

This project implements a highly available 3-tier architecture in AWS using a custom VPC.  
Traffic enters through an Application Load Balancer, is distributed across EC2 instances in private subnets, and connects to a Multi-AZ RDS MySQL database.  
Outbound internet access is handled via a Regional NAT Gateway, while the database layer remains isolated.

---

## Key Design Decisions

- **Multi-AZ Design**  
  Resources are distributed across two Availability Zones for high availability.

- **Subnet Segmentation**  
  Public, App, and DB tiers are isolated using dedicated subnets.

- **Security Group Chaining**  
  - ALB → App (HTTP)
  - App → DB (MySQL)
  - No direct internet access to App or DB

- **RDS Multi-AZ Deployment**  
  Primary in us-east-1a, standby in us-east-1b with synchronous replication and automatic failover.

- **Regional NAT Gateway**  
  Provides outbound internet access for private subnets with automatic multi-AZ expansion.

---

## Deployment Steps

1. Created VPC (10.0.0.0/16) with DNS enabled  
2. Created 6 subnets across 2 AZs  
3. Configured Internet Gateway and route tables  
4. Configured Regional NAT Gateway for outbound traffic  
5. Defined security groups for tier isolation  
6. Launched EC2 instances (App + Bastion)  
7. Deployed RDS MySQL (Multi-AZ)  
8. Configured Application Load Balancer and target groups  

---

## Load Balancing Validation

![Browser Test](screenshots/9-1-browser-test.png)

Traffic is distributed across both App Server A and App Server B via the ALB.

---

## Instance Failure Simulation

![Stopped Instance](screenshots/9-2-1-stopped-instance-a.png)  
![Health Check](screenshots/9-2-2-instance-a-health-check.png)  
![Failover](screenshots/9-2-3-failover-to-instance-b.png)

After stopping one instance, traffic continued to be served by the remaining healthy instance.

---

## App → RDS Connectivity

![DB Connection](screenshots/7-6-connecting-to-db.png)

Application instances successfully connect to the RDS endpoint over port 3306 using private networking.

---

## Outbound Internet Access

![NAT](screenshots/9-3-app-instance-bastion-host-nat.png)

Private app instances access the internet via the Regional NAT Gateway.

---

## Failure Scenarios

### ALB → App Security Group Misconfiguration

![SG Change](screenshots/10-1-2-updated-app-sg.png)  
![Error](screenshots/10-1-3-504-error.png)

Removing the ALB security group from the app tier results in a 504 Gateway Timeout, demonstrating dependency on correct security group chaining.

---

## What This Project Demonstrates

- VPC design and subnetting strategy  
- Route table configuration and traffic flow control  
- Security group-based segmentation  
- High availability across Availability Zones  
- Load balancing (Layer 7)  
- RDS Multi-AZ architecture  
- Controlled failure testing and validation  

---

## Supporting Artifacts

This repository includes supporting materials used to validate and document the deployment:

- **screenshots/**  
  Contains evidence of architecture behavior, including load balancing, failover testing, database connectivity, NAT gateway routing, and security group validation.

- **configs/**  
  Contains the EC2 user data scripts used to initialize and configure application instances during deployment.

These artifacts provide reproducibility and verification of the implemented 3-tier architecture.