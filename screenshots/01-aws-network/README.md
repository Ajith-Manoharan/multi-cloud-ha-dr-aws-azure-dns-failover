
# AWS Network Architecture

This section covers the foundational AWS networking setup for the multi-cloud high availability architecture.

## Architecture Components

- Custom VPC (10.0.0.0/16)
- Public and Private Subnets
- Internet Gateway
- NAT Gateway
- Route Tables
- Security Groups

---

## 1️⃣ VPC Overview

![VPC Overview](./01-vpc-overview.png)

---

## 2️⃣ Public & Private Subnets

![Subnets](./02-public-private-subnets.png)

---

## 3️⃣ Internet Gateway Attached

![Internet Gateway](./03-internet-gateway-attached.png)

---

## 4️⃣ NAT Gateway

![NAT Gateway](./04-nat-gateway-created.png)

---

## 5️⃣ Public Route Table

![Public Route Table](./05-public-route-table.png)

---

## 6️⃣ Private Route Table

![Private Route Table](./06-private-route-table.png)

---

## 7️⃣ Security Groups Configuration

![Security Groups](./08-security-groups-alb-ec2.png)

---

### Outcome

AWS network is fully segmented with public-facing ALB and private compute layer.
