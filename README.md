# Multi-Cloud High Availability & Disaster Recovery (AWS–Azure DNS Failover) 

## Project Overview

In this project, I implemented a DNS-based active–passive multi-cloud failover architecture using AWS as the primary environment and Azure as the disaster recovery (DR) environment.

The goal was to simulate a real-world production scenario where:

- AWS serves traffic under normal conditions
- If AWS becomes unavailable, traffic automatically fails over to Azure
- No manual switching is required

To verify which cloud is serving traffic, the application dynamically displays:

Instance: <Instance-ID> | Region: <Region>

This information is fetched directly from each cloud provider’s metadata service during instance boot.

---

# Architecture Summary

Under normal operation:

Client  
- Route 53 (Primary record)  
- AWS Application Load Balancer  
- EC2 Instance  

If AWS becomes unhealthy:

Client  
- Route 53 health check detects failure  
- DNS automatically switches to Azure endpoint  
- Azure VM serves traffic  

This is implemented as an active-passive model using DNS failover routing.

---

# Application Layer

The web application itself is very simple. It is a static HTML template stored in a GitHub repository.

The repository contains:

- A basic `index.html`
- A placeholder value `CLOUD_NAME`
- A minimal layout to display instance and region information

The GitHub repository only stores the application code. It does not configure infrastructure.

---

# Bootstrap Logic (Infrastructure Automation)

The key part of this project is how each VM configures itself automatically during launch.

Instead of manually logging into servers, I used:

- AWS **User Data** (for EC2)
- Azure **Custom Data / Cloud-Init** (for VM)

These scripts run automatically at first boot.

---

# AWS Primary Setup

When launching the EC2 instance (or Launch Template for scaling), I provided the following User Data script:

```bash
#!/bin/bash
set -e

yum update -y
yum install -y nginx git

systemctl enable nginx
systemctl start nginx

rm -rf /usr/share/nginx/html/*

git clone https://github.com/<your-username>/multi-cloud-ha-dr-aws-azure-dns-failover.git /tmp/app
cp -r /tmp/app/site/* /usr/share/nginx/html/

INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
REGION=$(curl -s http://169.254.169.254/latest/meta-data/placement/region)

sed -i "s/CLOUD_NAME/AWS PRIMARY/" /usr/share/nginx/html/index.html
echo "<p style='font-size:22px;'>Instance: $INSTANCE_ID | Region: $REGION</p>" >> /usr/share/nginx/html/index.html

systemctl restart nginx
```

What this script does:

1. Installs nginx and git
2. Clones the GitHub repository
3. Copies the HTML files to nginx web directory
4. Calls the AWS metadata endpoint:
   - http://169.254.169.254/latest/meta-data/instance-id
   - http://169.254.169.254/latest/meta-data/placement/region
5. Injects the instance ID and region into the webpage
6. Restarts nginx

As a result, every EC2 instance automatically configures itself and displays its own metadata.

---

# Azure Disaster Recovery Setup

For Azure, I used a Virtual Machine (or VM Scale Set). During creation, I passed a Cloud-Init script as Custom Data:

```bash
#!/bin/bash
set -e

apt-get update -y
apt-get install -y nginx git jq

systemctl enable nginx
systemctl start nginx

rm -rf /var/www/html/*

git clone https://github.com/<your-username>/multi-cloud-ha-dr-aws-azure-dns-failover.git /tmp/app
cp -r /tmp/app/site/* /var/www/html/

AZ_JSON=$(curl -s -H "Metadata:true" \
"http://169.254.169.254/metadata/instance?api-version=2021-02-01")

VM_ID=$(echo $AZ_JSON | jq -r '.compute.vmId')
REGION=$(echo $AZ_JSON | jq -r '.compute.location')

sed -i "s/CLOUD_NAME/AZURE DR/" /var/www/html/index.html
echo "<p style='font-size:22px;'>Instance: $VM_ID | Region: $REGION</p>" >> /var/www/html/index.html

systemctl restart nginx
```

Azure metadata is slightly different:

- It requires the header: `Metadata:true`
- It returns JSON
- It must be parsed using `jq`

This difference between AWS and Azure metadata services was an important learning point in this project.

---

# DNS Failover Configuration

Route 53 was configured with:

- Primary record → AWS ALB
- Secondary record → Azure Public IP
- Health check monitoring the AWS endpoint

If the health check fails, Route 53 automatically switches traffic to Azure.

This design keeps the solution cloud-agnostic and avoids cross-cloud networking complexity.

---

# Failover Test Procedure

To test failover:

1. Access the domain and confirm AWS is serving traffic.
2. Stop the AWS EC2 instance.
3. Wait for Route 53 health check to detect failure.
4. Refresh the domain.
5. Confirm Azure DR is serving traffic.

This validates the active-passive failover logic.

---

# Key Learnings

- DNS-based failover is simple and cloud-independent.
- Metadata services differ significantly between AWS and Azure.
- Bootstrapping at launch ensures repeatability.
- Active-passive is cost-efficient but not zero-downtime.

---

## 📸 Screenshots & Implementation Proof

All configuration steps and failover validation are documented inside the `screenshots/` directory.

### Browse by Section

- [01 – AWS Network](./screenshots/01-aws-network/)
- [02 – AWS Compute & ALB](./screenshots/02-aws-compute-alb/)
- [03 – Azure Network](./screenshots/03-azure-network/)
- [04 – Azure VMSS & Load Balancer](./screenshots/04-azure-compute-lb/)
- [05 – Route 53 DNS Failover](./screenshots/05-route53-dns-failover/)



Each folder contains a dedicated `README.md` displaying the screenshots for that implementation phase.


---

# Conclusion

This project demonstrates a practical implementation of multi-cloud resiliency using DNS failover and automated instance bootstrapping.

It reflects real-world disaster recovery patterns and emphasizes automation, reproducibility, and cross-cloud design.



