### Overview
A comprehensive, hands-on engineering implementation of a highly available, secure, and fault-tolerant Virtual Private Cloud (VPC) topology deployed across multiple Availability Zones (AZs) in the `us-east-1` region.

### Final Architectural Blueprint
The final physical and logical deployment state achieved at the conclusion:

+-----------------------------------------------------------------------------------------------------------------------+
| AWS REGION: us-east-1 (VPC CIDR: 10.16.0.0/16)                                                                        |
|                                                                                                                       |
|       +------------------------------------+    +------------------------------------+    +------------------------------------+ |
|       |        AVAILABILITY ZONE A         |    |        AVAILABILITY ZONE B         |    |        AVAILABILITY ZONE C         | |
|       |                                    |    |                                    |    |                                    | |
|       |  [WEB TIER] Public Subnet:         |    |  [WEB TIER] Public Subnet:         |    |  [WEB TIER] Public Subnet:         | |
|       |  sn-web-A (10.16.48.0/20)          |    |  sn-web-B (10.16.112.0/20)         |    |  sn-web-C (10.16.176.0/20)         | |
|       |  +------------------------------+  |    |  +------------------------------+  |    |  +------------------------------+  | |
|       |  |  💥 NAT Gateway A             |  |    |  |  💥 NAT Gateway B             |  |    |  |  💥 NAT Gateway C             |  | |
|       |  +------------------------------+  |    |  +------------------------------+  |    |  +------------------------------+  | |
|       +-----------------+------------------+    +-----------------+------------------+    +-----------------+------------------+ |
|                         |                                         |                                         |                    |
|       +-----------------v------------------+    +-----------------v------------------+    +-----------------v------------------+ |
|       |  [APP TIER] Private Subnet:        |    |  [APP TIER] Private Subnet:        |    |  [APP TIER] Private Subnet:        | |
|       |  sn-app-A (10.16.32.0/20)          |    |  sn-app-B (10.16.160.0/20)         |    |  sn-app-C (10.16.160.0/20)         | |
|       |  🖥️ Private Test EC2 (Ping Source) |    |                                    |    |                                    | |
|       +-----------------+------------------+    +-----------------+------------------+    +-----------------+------------------+ |
|                         |                                         |                                         |                    |
|       +-----------------v------------------+    +-----------------v------------------+    +-----------------v------------------+ |
|       |  [DB TIER] Private Subnet:         |    |  [DB TIER] Private Subnet:         |    |  [DB TIER] Private Subnet:         | |
|       |  sn-db-A (10.16.16.0/20)           |    |  sn-db-B (10.16.80.0/20)           |    |  sn-db-C (10.16.144.0/20)          | |
|       +-----------------+------------------+    +-----------------+------------------+    +-----------------+------------------+ |
|                         |                                         |                                         |                    |
|       +-----------------v------------------+    +-----------------v------------------+    +-----------------v------------------+ |
|       |  [RESERVED TIER] Private Subnet:   |    |  [RESERVED TIER] Private Subnet:   |    |  [RESERVED TIER] Private Subnet:   | |
|       |  sn-reserved-A (10.16.0.0/20)      |    |  sn-reserved-B (10.16.64.0/20)     |    |  sn-reserved-C (10.16.128.0/20)    | |
|       +------------------------------------+    +------------------------------------+    +------------------------------------+ |
|                                                                                                                       |
|       =============================================================================================================   |
|                                       SPARE CAPACITY BLOCK FOR FUTURISTIC 4TH AZ EXPANSION                           |
|       =============================================================================================================   |
+-----------------------------------------------------------+-----------------------------------------------------------+
|
[VPC Core Virtual Router]
|
+---------------------+---------------------+
| (Public Route: 0.0.0.0/0)                 | (Private Route: 0.0.0.0/0 Target NAT-GW)
v                                           v
[Internet Gateway (a4l-vpc-1-igw)]             [AZ-Specific NAT Gateways]
|                                           |
+---------------------+---------------------+
| (Dynamic Inbound Translation Layer)
v
🌐 PUBLIC INTERNET
(Target: 1.1.1.1)

### Steps

#### Phase 1: Custom Baseline VPC Blueprint (The Container Skeleton)
* **Objective:** Establish an isolated, logically partitioned administrative network boundary.
* **Implementation:** * Provisioned a custom VPC wrapper utilizing a `/16` network prefix (`10.16.0.0/16`), encapsulating `65,536` total IP addresses.
  * Explicitly toggled core DNS flags: `enableDnsSupport` and `enableDnsHostnames` turned to `true` to guarantee seamless name-resolution behaviors.
  * Provisioned and linked a native, AWS-allocated IPv6 CIDR block directly to the infrastructure container to establish modern dual-stack capability compatibility.

    Create VPC:

    <img width="898" height="352" alt="Create VPC - 2" src="https://github.com/user-attachments/assets/b5ae5a7a-1e6f-4a22-85cd-7c6a0744eb7a" />

    <img width="856" height="353" alt="Create VPC -1" src="https://github.com/user-attachments/assets/4eadb2e4-3b49-4811-b273-dfbf5514457a" />

    <img width="931" height="338" alt="VPC Created - 3" src="https://github.com/user-attachments/assets/6db161f4-f8c9-4091-9791-6f11e2bdefec" />

    Enable DNS flags:
 
    <img width="941" height="205" alt="vpc edit actions - 4" src="https://github.com/user-attachments/assets/d467ec0f-7c62-4723-a78c-98598ae5dcfa" />

    <img width="896" height="333" alt="enable dns - 5" src="https://github.com/user-attachments/assets/b6adb317-6c58-4ee4-8e16-c1101a553d47" />

 #### Phase 2: Multi-Tier Subnet Segmentation (12-Subnet Matrix)
* **Objective:** Architect strict network isolation boundaries across compute (web), business logic (app), data persistence layers (db), and future expansion fields (reserved).
* **Implementation:** * Manually configured **12 distinct subnets** utilizing tight `/20` masks, allocating `4,096` unique IP signatures per network segment.
  * Distributed subnets across **3 Availability Zones** (AZ-A, AZ-B, and AZ-C) to satisfy zero single-point-of-failure metrics:
    * **WEB Tier (3 Public Subnets):** Dedicated zone for external load-balancers, ingress proxies, and internet front doors (`sn-web-A`, `sn-web-B`, `sn-web-C`).
    * **APP Tier (3 Private Subnets):** Area hosting internal application processing units (`sn-app-A`, `sn-app-B`, `sn-app-C`).
    * **DB Tier (3 Private Subnets):** Optimized for database workloads, strictly isolated from external visibility (`sn-db-A`, `sn-db-B`, `sn-db-C`).
    * **Reserved Tier (3 Private Subnets):** Preserved for customized networking appliances or future edge integration vectors (`sn-reserved-A`, `sn-reserved-B`, `sn-reserved-C`).
    * **Dual-Stack Automation Rule:** Explicitly enabled the `Enable auto-assign IPv6 address setting` to auto-assign native IPv6 addresses at the subnet level. This ensures that any compute resource launched into these boundaries automatically provisions a globally unique IPv6 address without requiring manual resource-level overrides.
  * **Scalability Purpose:** Spare block space within the overall allocation map to allow the attachment of a 4th physical Availability Zone without breaking existing configurations.

Multi-tier VPC Subnets Creation:

Below screenshots show subnets creations in AZs A,B and C for the reserved tier. Likewise I did manual creation of the web, app and db subnets in each of the 3 AZs with a total of 12 subnets. (9 private subnets for app/db/reserved tiers and 3 public subnets for the web tier)

AZ-A:
-----
<img width="742" height="308" alt="Create Subnet - step 1" src="https://github.com/user-attachments/assets/18832937-d673-4745-95fd-9810ee202caa" />

<img width="785" height="351" alt="step 2a" src="https://github.com/user-attachments/assets/390e8bb4-0630-424f-aa45-89323604c54d" />

<img width="765" height="327" alt="step 2b" src="https://github.com/user-attachments/assets/7cab98f6-4be5-47dc-b426-66b8c98ddfb0" />

AZ-B:
-----
<img width="794" height="359" alt="AZ_B - subnet Step 1" src="https://github.com/user-attachments/assets/6b01e170-4018-4e13-8285-16ae36183d43" />

<img width="787" height="322" alt="AZ-B STEP 2" src="https://github.com/user-attachments/assets/96569d29-5898-48df-bea9-22376b1c66a2" />

AZ-C:
-----
<img width="796" height="355" alt="AZ-C STEP1" src="https://github.com/user-attachments/assets/6d9bcd1f-47ad-43de-85d5-a1ef8ef0885d" />

<img width="770" height="355" alt="AZ-C STEP2" src="https://github.com/user-attachments/assets/131d74a2-52e7-42cb-b50d-fd95493de536" />

Subnets in all tiers (including app,db,web):

<img width="953" height="355" alt="Success- All 12 subnets" src="https://github.com/user-attachments/assets/97fa2ddc-4761-4c28-9a1d-adf12659a163" />

Enable auto-assign IPv6 in subnet settings:

<img width="946" height="358" alt="Step 4 - Edit SUbnet Setting Enable IPV6 for Subnets" src="https://github.com/user-attachments/assets/fbb02447-a614-4221-ad3b-763fe1712c0a" />

<img width="761" height="356" alt="step 4b - enable ipv6 on created subnets" src="https://github.com/user-attachments/assets/ec9270a4-28c6-4487-b59f-3ac0368ac99c" />











