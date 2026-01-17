# Chapter 1 — VPC Networking (Hands-on)

## Goal
Create the base AWS network for the project (zero-cost):
- VPC
- Public Subnet
- Internet Gateway
- Route Table (public route)
- Security Group (web)

## Region
✅ Region used: us-east-1

## Resources created
### VPC
- Name: project-vpc
- CIDR: 10.0.0.0/16

### Subnet
- Name: public-subnet
- AZ: us-east-1a
- CIDR: 10.0.1.0/24

### Internet Gateway
- Name: project-igw
- Attached to: project-vpc

### Route Table
- Name: public-rt
- Route: 0.0.0.0/0 -> project-igw
- Associated subnet: public-subnet

### Security Group
- Name: web-sg
Inbound rules:
- SSH (22) from My IP
- HTTP (80) from 0.0.0.0/0
Outbound:
- Allow all (default)

## Why this design
- VPC provides network isolation
- Public subnet + IGW allows internet access
- Route table enables outbound/inbound internet routing for public subnet
- Security group restricts access to only required ports

## Notes
✅ Any issues faced:
✅ What you learned:
