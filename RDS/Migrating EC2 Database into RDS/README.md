# AWS Architecture Decoupling: Migrating EC2 Web Application Database to Amazon RDS

## 📌 Project Overview
Demonstrates the architectural decoupling of a monolithic web application stack into a scalable, high-availability multi-tier environment. 

Initially, the application and database run on a single Amazon EC2 instance. This lab migrates the active mariaDB database from the EC2 instance to a fully managed Amazon RDS (MySQL) instance, updates application configuration files, and verifies database health without data loss.

---

## Architecture

### Initial Architecture (Monolithic)
* Single EC2 Instance hosting both Apache Web Server and local Maria DB instance (`localhost`).

### Target Architecture (Decoupled Multi-Tier)
* **Web Tier:** Apache Web Server running on EC2 in a Public Subnet.
* **Database Tier:** Amazon RDS MySQL Instance in dedicated Database Subnets.
* **Security & Access:** EC2 Security Group allows inbound web traffic (HTTP/80) and outbound access to RDS. RDS Security Group allows inbound MySQL traffic (Port 3306) specifically from the EC2 Security Group ID.

---

## 🛠️ Step-by-Step Implementation & Commands
### 1a. Create a sample WP blog for testing
Copy the public IP of the EC2 instance on which apache web server where the WordPress site is hosted. 
Login with the admin user and DB password and create a sample test blog inside WP.

<img width="767" height="472" alt="Screenshot 2026-08-03 161326" src="https://github.com/user-attachments/assets/10f0d298-3ae1-44fa-831c-85f077a77cae" />


### 1b. Export Database Dump
Connect to the EC2 host via EC2 browser Instance Connect and export the local MySQL database using `mysqldump`:

```bash
# Backup of Source Database
mysqldump -h PRIVATEIPOFMARIADBINSTANCE -u a4lwordpress -p a4lwordpress > a4lwordpress.sql
Run a ls -la to verify the dump file has been succesfully retrieved

<img width="686" height="337" alt="Screenshot 2026-08-03 164039" src="https://github.com/user-attachments/assets/ae18566a-59e2-4780-9f30-7d9dace24a14" />

```

### 2a. Create DB subnet group 
The DB subnet group tells RDS into which subnets should the DB instances be created. (Useful For a multi-tier application not a single AZ/single-tier application)

<img width="770" height="311" alt="Screenshot 2026-08-03 162047" src="https://github.com/user-attachments/assets/960a080d-e687-43c7-9aa9-80c9672d7f3d" />

<img width="627" height="320" alt="Screenshot 2026-08-03 162124" src="https://github.com/user-attachments/assets/c813e994-f2f6-4416-bdb0-ecf3fe0a9837" />

<img width="954" height="245" alt="Screenshot 2026-08-03 162227" src="https://github.com/user-attachments/assets/3960f814-2e45-4ebe-b457-d668694fa741" />

### 2b. Provision RDS Instance & Configure Security Groups
Provision an Amazon RDS MySQL database instance.

<img width="945" height="298" alt="Screenshot 2026-08-03 162327" src="https://github.com/user-attachments/assets/e7bb45e1-37eb-48a5-b796-2cb517d46372" />

<img width="922" height="253" alt="Screenshot 2026-08-03 162349" src="https://github.com/user-attachments/assets/e12b7182-ebab-4533-ad16-c79bc5f15181" />

<img width="923" height="329" alt="Screenshot 2026-08-03 162424" src="https://github.com/user-attachments/assets/21e6f77c-39bb-45dc-ac8c-5992fdb4b169" />

<img width="904" height="354" alt="Screenshot 2026-08-03 162603" src="https://github.com/user-attachments/assets/9f76ddb8-e6dd-49d6-b632-0f493d410c39" />

<img width="930" height="328" alt="Screenshot 2026-08-03 162636" src="https://github.com/user-attachments/assets/ee759232-8254-40db-9268-30ee6541d477" />

<img width="865" height="311" alt="Screenshot 2026-08-03 162747" src="https://github.com/user-attachments/assets/5a94519f-20ef-45eb-bb6d-5ffd807bf6f0" />

<img width="743" height="286" alt="Screenshot 2026-08-03 162758" src="https://github.com/user-attachments/assets/1cb73f43-2a16-40f7-8b6a-1e9cbcea0c6b" />

<img width="908" height="352" alt="Screenshot 2026-08-03 162826" src="https://github.com/user-attachments/assets/24686298-f1dd-4f74-b664-dd2675a84dc4" />

<img width="650" height="331" alt="Screenshot 2026-08-03 162851" src="https://github.com/user-attachments/assets/d0d7b872-db43-42b5-8ce7-668425a24739" />

<img width="935" height="355" alt="Screenshot 2026-08-03 163037" src="https://github.com/user-attachments/assets/a39760ea-c929-4fb4-8190-36e2134c9a46" />

<img width="888" height="289" alt="Screenshot 2026-08-03 163142" src="https://github.com/user-attachments/assets/7ac586c0-3828-40a5-ac2b-cef2df8c08bf" />

<img width="789" height="350" alt="Screenshot 2026-08-03 163221" src="https://github.com/user-attachments/assets/e474ae4d-c35f-4bfb-8630-40ffea1d16d4" />

<img width="957" height="185" alt="db success created" src="https://github.com/user-attachments/assets/d38f114c-afa2-4b6f-99a2-41824c506a83" />

# Update inbound security rules on the RDS Security Group to allow MySQL traffic (TCP Port 3306) originating strictly from the EC2 Instance Security Group.

<img width="959" height="352" alt="inbound sg of rds modified" src="https://github.com/user-attachments/assets/2d9b1eea-a017-4091-b231-915d41b01554" />

<img width="790" height="160" alt="Screenshot 2026-08-03 161050" src="https://github.com/user-attachments/assets/c1f90584-00c5-48af-af42-2eca720e52aa" />

### 3. Migrate Database to RDS
# Restore to Destination Database
```bash
mysql -h CNAMEOFRDSINSTANCE -u a4lwordpress -p a4lwordpress < a4lwordpress.sql
```
### 4. Update Application Configuration
Modify the WordPress configuration file (wp-config.php) to point application database queries from local host to the newly created RDS endpoint:

# Edit application configuration file
cd /var/www/html
sudo nano wp-config.php
ctrl+o(save), enter, ctrl+x(exit)

# Update Database Host Definition:
```bash
replace
/** MySQL hostname */
define('DB_HOST', 'PRIVATEIPOFMARIADBINSTANCE');

with 
/** MySQL hostname */
define('DB_HOST', 'REPLACEME_WITH_RDSINSTANCEENDPOINTADDRESS');
```
### 5. Verify Decoupled Application
Stop local MySQL service on EC2 to confirm no remaining dependency on local storage:
Right click EC2 instance and click on Stop
Once instance is stopped, test and verify live web application rendering content, media links, and database query executions directly via the browser.

<img width="957" height="337" alt="Screenshot 2026-08-05 162738" src="https://github.com/user-attachments/assets/4ff25e19-28f4-40cf-869a-1566d9499e16" />

<img width="805" height="402" alt="Screenshot 2026-08-05 162517" src="https://github.com/user-attachments/assets/309b0570-cfad-4bf0-9cf5-50b68506991e" />
