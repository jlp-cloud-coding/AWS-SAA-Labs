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
* **Implementation Steps:** * Create a custom VPC container utilizing a `/16` network prefix (`10.16.0.0/16`), encapsulating `65,536` total IP addresses.
  * Toggle the DNS settings: `enableDnsSupport` and `enableDnsHostnames` checked as `true` to ensure seamless name-resolution behaviors.
  * Linked a native, AWS-allocated IPv6 CIDR block directly to the VOC container to establish dual-stack capability compatibility.

Create VPC:

<img width="898" height="352" alt="Create VPC - 2" src="https://github.com/user-attachments/assets/b5ae5a7a-1e6f-4a22-85cd-7c6a0744eb7a" />

<img width="856" height="353" alt="Create VPC -1" src="https://github.com/user-attachments/assets/4eadb2e4-3b49-4811-b273-dfbf5514457a" />

<img width="931" height="338" alt="VPC Created - 3" src="https://github.com/user-attachments/assets/6db161f4-f8c9-4091-9791-6f11e2bdefec" />

Enable DNS flags:
 
<img width="941" height="205" alt="vpc edit actions - 4" src="https://github.com/user-attachments/assets/d467ec0f-7c62-4723-a78c-98598ae5dcfa" />

<img width="896" height="333" alt="enable dns - 5" src="https://github.com/user-attachments/assets/b6adb317-6c58-4ee4-8e16-c1101a553d47" />

 #### Phase 2: Multi-Tier Subnet Segmentation (12-Subnet Matrix)
* **Objective:** Architect strict network isolation boundaries across compute (web), business logic (app), data persistence layers (db), and future expansion fields (reserved).
* **Implementation Steps:** * Manually configured **12 distinct subnets** utilizing tight `/20` masks, allocating `4,096` unique IP signatures per network segment.
  * Distributed subnets across **3 Availability Zones** (AZ-A, AZ-B, and AZ-C) to satisfy zero single-point-of-failure metrics:
    * **WEB Tier (3 Public Subnets):** Dedicated zone for external load-balancers, ingress proxies, and internet front doors (`sn-web-A`, `sn-web-B`, `sn-web-C`).
    * **APP Tier (3 Private Subnets):** Area hosting internal application processing units (`sn-app-A`, `sn-app-B`, `sn-app-C`).
    * **DB Tier (3 Private Subnets):** Optimized for database workloads, strictly isolated from external visibility (`sn-db-A`, `sn-db-B`, `sn-db-C`).
    * **Reserved Tier (3 Private Subnets):** Preserved for customized networking appliances or future edge integration vectors (`sn-reserved-A`, `sn-reserved-B`, `sn-reserved-C`).
    * **Dual-Stack Automation Rule:** Explicitly enabled the `Enable auto-assign IPv6 address setting` to auto-assign native IPv6 addresses at the subnet level. This ensures that any compute resource launched into these boundaries automatically provisions a globally unique IPv6 address without requiring manual resource-level overrides.
  * **Scalability Purpose:** Allocated spare block space to allow the attachment of a 4th physical Availability Zone in future, without breaking existing configurations.

Multi-tier VPC Subnets Creation:

Below screenshots show subnets creations in AZs A,B and C for the reserved tier. Likewise I did manual creation of the web, app and db subnets in each of the 3 AZs with a total of 12 subnets. (9 private subnets for app/db/reserved tiers and 3 public subnets for the web tier)

Subnet-A:
--------
<img width="742" height="308" alt="Create Subnet - step 1" src="https://github.com/user-attachments/assets/18832937-d673-4745-95fd-9810ee202caa" />

<img width="785" height="351" alt="step 2a" src="https://github.com/user-attachments/assets/390e8bb4-0630-424f-aa45-89323604c54d" />

<img width="765" height="327" alt="step 2b" src="https://github.com/user-attachments/assets/7cab98f6-4be5-47dc-b426-66b8c98ddfb0" />

Subnet-B:
--------
<img width="794" height="359" alt="AZ_B - subnet Step 1" src="https://github.com/user-attachments/assets/6b01e170-4018-4e13-8285-16ae36183d43" />

<img width="787" height="322" alt="AZ-B STEP 2" src="https://github.com/user-attachments/assets/96569d29-5898-48df-bea9-22376b1c66a2" />

Subnet-C:
---------
<img width="796" height="355" alt="AZ-C STEP1" src="https://github.com/user-attachments/assets/6d9bcd1f-47ad-43de-85d5-a1ef8ef0885d" />

<img width="770" height="355" alt="AZ-C STEP2" src="https://github.com/user-attachments/assets/131d74a2-52e7-42cb-b50d-fd95493de536" />

Subnets in all tiers (including app,db,web):

<img width="953" height="355" alt="Success- All 12 subnets" src="https://github.com/user-attachments/assets/97fa2ddc-4761-4c28-9a1d-adf12659a163" />

Enable auto-assign IPv6 in subnet settings:

<img width="946" height="358" alt="Step 4 - Edit SUbnet Setting Enable IPV6 for Subnets" src="https://github.com/user-attachments/assets/fbb02447-a614-4221-ad3b-763fe1712c0a" />

<img width="761" height="356" alt="step 4b - enable ipv6 on created subnets" src="https://github.com/user-attachments/assets/ec9270a4-28c6-4487-b59f-3ac0368ac99c" />

#### Phase 3 & 4: Configuring VPC public subnets and Jumpbox & Boundary Handshake Verification
* **Objective:** We implement an Internet Gateway, Route Tables and Routes within the VPC to support the WEB public subnets. Once the WEB subnets are public, we create a bastion host with public IPv4 addressing and connect to it to test.
* **Implementation Steps:**
* Create and attach an Internet Gateway (`a4l-vpc1-igw`) onto the VPC container(created in Phase 1).
 
<img width="937" height="348" alt="Create IGW" src="https://github.com/user-attachments/assets/29ae6c63-8b39-4a67-8e87-5188d5909037" />

<img width="958" height="363" alt="IGW Success" src="https://github.com/user-attachments/assets/a12e89a2-1125-4214-b7e4-be0738459d0a" />

<img width="958" height="250" alt="3  Attach IGW to vpc" src="https://github.com/user-attachments/assets/38e54b0c-d638-4866-aaf9-67d0b76e171d" />

<img width="956" height="247" alt="attach igw to vpc 3b" src="https://github.com/user-attachments/assets/e5e796a9-c024-4c47-9574-fb6e244cc445" />

* Create a distinct web Route Table (`a4l-vpc1-rt-web`) to isolate edge policies away from the default Main Route Table.

<img width="898" height="344" alt="Step2 5-Create a RT" src="https://github.com/user-attachments/assets/6f642bdd-3d42-4755-999a-cf0133b16b58" />

* Edit routes inside the Route table pointing to the edge device (public internet):
    * **IPv4 Gateway Egress:** `0.0.0.0/0` &rarr; `a4l-vpc1-igw`
    * **IPv6 Gateway Egress:** `::/0` &rarr; `a4l-vpc1-igw`
   
<img width="947" height="354" alt="step 3 - add routes" src="https://github.com/user-attachments/assets/84617377-b494-4e81-8d58-8e9b53abd4d8" />

<img width="971" height="351" alt="step 3 - 8 add custom routes for public internet" src="https://github.com/user-attachments/assets/f97e6abb-ef61-473c-8185-ccb79f0c400e" />

* Associate the 3 public-facing WEB subnets to this new custom route table. Also update the subnet properties to mandate public IPv4 auto-assignment at the subnet-level to prevent doing it later on at resource level for each individual resource created in that specific subnet.

<img width="950" height="350" alt="step 4 - 9  edit subnet settings" src="https://github.com/user-attachments/assets/a8406d0e-698e-4921-abb7-47df890b4cbc" />

<img width="948" height="353" alt="step2 - 7 attach subnets success" src="https://github.com/user-attachments/assets/ff4f4a80-313b-4f48-9b6a-13181d4b5241" />

<img width="950" height="350" alt="step 4 - 9  edit subnet settings" src="https://github.com/user-attachments/assets/d2c73ccb-6c58-472a-a3bd-ca0bf3e68461" />

<img width="949" height="334" alt="step 4 - 10  enable auto assign ipv4" src="https://github.com/user-attachments/assets/6c9f2174-db7c-415b-b198-fb8f2144b6ca" />

  * **Validation:** * Created a `t3.micro` EC2 instance named `a4l-bastion` inside `sn-web-B` (AZ-B) to validate cross-zone high-availability. Create a Security Group (`A4L-BASTION-SG`) while creating the EC2 instance to enforce explicit inbound port constraints.

<img width="929" height="353" alt="step 1 launch ec2" src="https://github.com/user-attachments/assets/ea44e92d-dfe1-4c02-a25a-11bb53acaf83" />

<img width="947" height="349" alt="step2" src="https://github.com/user-attachments/assets/e41e203c-c6c2-4b0a-985f-7fdbfc6f6e1b" />

<img width="626" height="311" alt="step 3 " src="https://github.com/user-attachments/assets/5dc6099f-96ff-4c83-94bf-df77e2d28795" />

<img width="940" height="350" alt="step 4 " src="https://github.com/user-attachments/assets/4b2b9380-14a0-47e5-bb79-b5baf2827b20" />

<img width="932" height="350" alt="step5" src="https://github.com/user-attachments/assets/709035a9-1a70-4569-8ee1-31a86abd2e48" />

<img width="959" height="332" alt="step 6 instance created" src="https://github.com/user-attachments/assets/420eefe3-77dc-4346-9ba5-89a804c4017c" />


* Connect to this instance remotely from our local machine shell utilizing an asymmetric cryptographic keypair (`A4L.pem` via `ssh -i`).

 <img width="943" height="356" alt="step 8 connect to instance" src="https://github.com/user-attachments/assets/a951e1ab-87bc-42be-8ce6-758f4941abeb" />

 <img width="923" height="358" alt="step 9 connect using ssh client" src="https://github.com/user-attachments/assets/6412b102-e0a1-4fa4-b5d0-213138149163" />

* **Success connection:** Observe that the local machine prompt connect to and display the internal, private IP of the host node (`ec2-user@ip-10-16-112-x`).

<img width="862" height="460" alt="step 10 connect to ec2 successful" src="https://github.com/user-attachments/assets/cd80b96a-a402-4333-a5aa-ed7cebc2619e" />

#### Phase 5: NAT Gateway Deployment - implementing private internet access

Prior to executing this phase, all baseline infrastructure components from the previous phases were deleted to maintain zero active cloud costs. To establish an identical, production-ready baseline for this lab, an automated infrastructure-as-code deployment script via **AWS CloudFormation** was executed to instantly reconstruct the complete Phase 1 through Phase 4 structure. The NAT Gateway components, route tables, and subnet associations are done as part of Phase 5 separately.

* **Objective:** Enable highly secure outbound-only internet connectivity for backend servers sitting in private subnets, allowing them to download security patches or system dependencies while strictly blocking the public internet from initiating direct inbound connections.
* **Implementation Steps:**
* Before creating any NAT Gatways we first validate by connecting the private EC2 instance created (which has been created using the CFN template along with the other resources from Phases 1 to 4) in the sn-app-A subnet. This instance doesn't have an IPV4 public address associated with it, so we connect to it via the SSH Session Manager inorder to access the EC2 console.

<img width="952" height="368" alt="step1-ec2-ssh connect" src="https://github.com/user-attachments/assets/dc2927a5-2943-4d5e-98be-806d24708ed1" />

<img width="936" height="339" alt="step 2 - ssm session manager" src="https://github.com/user-attachments/assets/c5d8a45a-7276-4948-a886-75bd7d4c1376" />

* Next we using a ping command to ping to a public IPv4 address. **ping 1.1.1.1**

<img width="956" height="269" alt="step3-ping" src="https://github.com/user-attachments/assets/ef44889c-6c18-4bcb-9dd2-ebfb32280a88" />

* Since we don't have a NAT Gateway created yet, this private EC2 instance cannot send any connection request to the public internet yet.
  
* Now create **3 distinct NAT Gateways** across the three public WEB subnets (`sn-web-A`, `sn-web-B`, and `sn-web-C`).
 
<img width="959" height="393" alt="step4-create natgw-a" src="https://github.com/user-attachments/assets/9edc4eb0-406b-4f22-8d42-051058dd2758" />

<img width="985" height="372" alt="step4a-create natgwa" src="https://github.com/user-attachments/assets/34b4212a-e565-4aca-93c5-2c7519417942" />

<img width="967" height="374" alt="step4b-create natgwa" src="https://github.com/user-attachments/assets/05e3344d-0454-461a-9cd7-9812550c56bd" />

All the NAT Gateways:

<img width="956" height="371" alt="step5- all natgws created" src="https://github.com/user-attachments/assets/ab5192e2-3bca-4c3d-8cfc-a1c88676352a" />

Elastic IPs (public IPv4 addresses) for all the NAT Gateways:

<img width="953" height="377" alt="step6 - elastic IPs" src="https://github.com/user-attachments/assets/d12164aa-6829-468f-a563-897d9e5f496d" />

* *Design Rationale:* Deploying a separate NAT Gateway in each individual Availability Zone satisfies AWS high-availability best practices. If a single physical data center zone suffers a major failure, the other two zones maintain independent outbound public exit tracks without dealing with costly cross-AZ data processing delays or fatal single-point failure vectors.

* Create **3 private Route Tables** (one dedicated for each independent Availability Zone block) to maintain total isolation boundaries.
  
<img width="959" height="374" alt="step 7-create RTa" src="https://github.com/user-attachments/assets/0a754be9-2c0a-46bc-9b09-2ea3aec7cad9" />

* Embed a default outbound IPv4 route path within each route table targeting its respective, local-zone NAT Gateway:
    * **Private Egress Policy:** `0.0.0.0/0` &rarr; `NAT Gateway (Specific to that active AZ below a4l-vpc1-natgw-A)`
   
<img width="959" height="368" alt="step8 - add default routes" src="https://github.com/user-attachments/assets/ad11ad78-fc29-4e0a-b51e-e08ad9b6257d" />

<img width="957" height="358" alt="step8a - add natgwa in the target" src="https://github.com/user-attachments/assets/9d73c1ec-046f-45d0-b5b5-81bcb8af9e8b" />

Similarly edit and add default routes for other RTs in different AZs, pointing to the corresponding NAT Gatways in those AZs:

<img width="959" height="377" alt="step 8b - default route added for b" src="https://github.com/user-attachments/assets/24d13fda-4418-4ce0-8477-ea76b4524005" />

<img width="959" height="376" alt="step 8b- add natgw b " src="https://github.com/user-attachments/assets/a027ebf9-d655-48a7-a255-a795aedf7f6b" />

* Link the custom route tables to the corresponding private subnets (including `sn-app-A`, `sn-db-A`, and `sn-reserved-A`).

<img width="959" height="375" alt="step 9-subnet association a" src="https://github.com/user-attachments/assets/01be9bab-ef0e-45d7-9c6d-4476f3f05d3d" />

<img width="957" height="370" alt="step 9a" src="https://github.com/user-attachments/assets/99418405-ac03-4984-be26-a19893287cba" />

* **Validation:** Re-ran the test (`ping 1.1.1.1`). Observed live network tracking returns successfully streaming back into the private console terminal session.

<img width="959" height="413" alt="step10-ping success to internet ipv4" src="https://github.com/user-attachments/assets/1241e488-438f-4440-abe1-7762fcc6c472" />












