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
