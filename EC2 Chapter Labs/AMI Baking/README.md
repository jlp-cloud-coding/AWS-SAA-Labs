# AWS Infrastructure Optimization: AMI Baking & Stateful-to-Stateless Migration

## Project Overview
This repository documents a hands-on technical lab focused on **Amazon Machine Images (AMIs)** and the process of **AMI Baking (Golden Images)**. 

In production environments, bootstrapping instances from scratch using launch scripts (User Data) can cause critical delays during auto-scaling events. This project demonstrates how to eliminate initialization latency, eliminate external package repository dependencies, and ensure 100% deployment consistency by "baking" a fully configured application stack directly into a custom AMI template.

---

## Architecture

### 1. The Starting Point: Monolithic & Stateful (Anti-Pattern)
The lab begins by deploying a standard web server architecture where the web engine (**Apache**), application code (**WordPress**), and database management system (**MariaDB**) run concurrently on a single EC2 instance. 
* **The Problem:** This establishes a Single Point of Failure (SPOF), locks the infrastructure into restrictive vertical scaling models, and creates an environment where compute layers are tightly coupled to stateful local disk resources.

### 2. The Production Objective: Decoupled & Stateless (Target Pattern)
By using the AMI generated in this project, the architecture can seamlessly transition to a production-ready cloud deployment:
* **Compute Tier:** Stateless EC2 instances managed dynamically by an **Auto Scaling Group** behind an **Application Load Balancer (ALB)**.
* **Storage Tier:** User uploads and media offloaded to **Amazon S3**.
* **Database Tier:** Relational states migrated to an **Amazon RDS Multi-AZ** cluster for high availability and automatic failover.

---

## Steps for Implementation

### Step 1: Base Configuration & Stack Deployment
Launched a base Linux EC2 instance and manually provisioned the complete LAMP stack, application parameters, and custom operating system banners (`cowsay` custom MOTD configuration).

```bash
# Core software provisioning snippet executed during initialization:
sudo dnf install wget php-mysqlnd httpd php-fpm php-mysqli mariadb105-server php-json php php-devel -y
sudo systemctl enable httpd && sudo systemctl enable mariadb
sudo systemctl start httpd && sudo systemctl start mariadb
```

### Step 2: Ensuring Data Consistency
To prevent snapshot fragmentation and file system corruption, the source EC2 instance was gracefully Stopped prior to image creation. This completely freezes disk I/O operations and guarantees a clean, stable block capture.

### Step 3: The Bake
Initiated the CreateImage workflow inside the EC2 environment. AWS concurrently generated:

An EBS Snapshot representing the exact point-in-time configuration of the 8 GB root volume.

A Custom AMI metadata mapping that wraps the snapshot into a reusable, regional deployment template.

### Step 4: Immutable Deployment Verification
Launched a completely fresh target EC2 instance referencing the newly baked AMI. The instance booted up instantly functional, completely bypassing the software installation and code setup phases.

### Validation
1. Reference Source Instance Configuration
Description: The baseline operational EC2 instance summary and assigned public access configurations.

2. Initiating Image Capture (The Bake Action)
Description: Defining metadata attributes and configuring the state-capture criteria for the custom Golden Image template.

3. Structural Mapping: AMI & Snapshot Association
Description: Verification of the registered AMI within the management console, explicitly displaying its logical link to the underlying backend EBS storage snapshot.

4. End-to-End Verification: Immutable Cloning Success
Description: Accessing the newly cloned server via its distinct public IPv4 address, demonstrating that the full WordPress initialization pipeline and database mappings inherited flawlessly without any manual server-side intervention.

### Cleanup Steps:
To adhere to cloud cost-optimization best practices and strictly maintain AWS Free Tier compliance boundaries, a mandatory deletion sequence was strictly executed immediately following verification:

1. Terminate all deployed operational EC2 compute instances.

2. Deregisterthe customized AMI metadata template to break the asset mapping lock.

3. Delete the underlying 8 GB block-level EBS Snapshot from storage components to eliminate residual storage fees.
