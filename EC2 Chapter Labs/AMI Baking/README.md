# AWS Infrastructure Optimization: AMI Baking & Stateful-to-Stateless Migration

## Project Overview
This repository documents a hands-on technical lab focused on **Amazon Machine Images (AMIs)** and the process of **AMI Baking (Golden Images)**. 

In production, installing your application from scratch using boot scripts (User Data) can cause major delays when trying to scale out during a traffic spike. It also relies on the internet to safely download packages every single time. This project demonstrates how to eliminate that setup delay and remove external dependencies by 'baking' the fully configured application directly into a custom AMI. This ensures your servers boot up instantly and are 100% identical every single time.

---

## Architecture

### 1. The Manual Setup (The Source Instance)
* **What was built:** A single EC2 instance running a standard web application LAMP stack. LAMP is an acronym for the operating system, Linux; the web server, Apache; the database server, MySQL/Maria DB; and the programming language, PHP/Python/Perl (In this lab **Apache**, **PHP**, **WordPress**, and a **MariaDB** database are used). 
* **The Problem:** Setting this up required typing out 11 steps of manual Linux configurations. If we needed to build say 5 more identical servers, repeating this manual installation process for every single one would take too much time and invite human error.

### 2. The Automated Setup (The Baked AMI Clone)
* **The Solution:** Instead of repeating manual installations, we configured the source server perfectly *once* and then "baked" it into a custom **Amazon Machine Image (AMI)**.
* **The Result:** The custom AMI acts as a standard template. When we launched a brand-new EC2 instance from this AMI, it booted up with WordPress, Apache, the database, and even our custom `cowsay` login screen already fully installed and working. No manual setup scripts were required on the new machine.

---

## Steps for Implementation

### Step 1: Base Configuration & Stack Deployment
Launcha base Linux EC2 instance and manually provisioned the complete LAMP stack, application parameters, and custom operating system banners (`cowsay` custom MOTD configuration).

# 1: Set Up Configuration Variables
These variables store your database settings in the terminal's memory so you don't have to re-type them in later commands.

``` 
DBName='a4lwordpress'
DBUser='a4lwordpress'
DBPassword='4n1m4l$L1f3'
DBRootPassword='4n1m4l$L1f3'

```

# 2: Install System Software Stack (LAMP)
This downloads and installs the web server (Apache), the programming language runtime (PHP), and the database management engine (MariaDB).

```
sudo dnf install wget php-mysqlnd httpd php-fpm php-mysqli mariadb105-server php-json php php-devel -y
```
# 3. Start and Enable Services
This turns on Apache (httpd) and MariaDB, and configures Linux to turn them back on automatically if the server reboots.

```
sudo systemctl enable httpd
sudo systemctl enable mariadb
sudo systemctl start httpd
sudo systemctl start mariadb
```

# 4: Set the Database Root Password
Secures the core database management system engine with an administrative password.
```
sudo mysqladmin -u root password $DBRootPassword
```
# 5: Download and Extract WordPress
Downloads the official WordPress application package to the web directory, unpacks it, cleans up the leftover zip files, and moves the folders to the correct server path.
```
sudo wget http://wordpress.org/latest.tar.gz -P /var/www/html
cd /var/www/html
sudo tar -zxvf latest.tar.gz
sudo cp -rvf wordpress/* .
sudo rm -R wordpress
sudo rm latest.tar.gz
```

# 6: Configure the Application Settings
Copies the sample configuration file, dynamically replaces the placeholder text with your database variables using sed (stream editor), and hands file ownership over to the Apache web server.
```
sudo cp ./wp-config-sample.php ./wp-config.php
sudo sed -i "s/'database_name_here'/'$DBName'/g" wp-config.php
sudo sed -i "s/'username_here'/'$DBUser'/g" wp-config.php
sudo sed -i "s/'password_here'/'$DBPassword'/g" wp-config.php   
sudo chown apache:apache * -R
```

# 7: Create the Application Database
Writes the core structured query commands (SQL) to a temporary file to create the a4lwordpress database and assign user permissions, feeds that file directly into MariaDB, and wipes the temporary setup file clean.
```
echo "CREATE DATABASE $DBName;" >> /tmp/db.setup
echo "CREATE USER '$DBUser'@'localhost' IDENTIFIED BY '$DBPassword';" >> /tmp/db.setup
echo "GRANT ALL ON $DBName.* TO '$DBUser'@'localhost';" >> /tmp/db.setup
echo "FLUSH PRIVILEGES;" >> /tmp/db.setup
mysql -u root --password=$DBRootPassword < /tmp/db.setup
sudo rm /tmp/db.setup
```

# 8: Install and Test cowsay
Installs the fun terminal customization tool and runs a basic test command.
```
sudo dnf install -y cowsay
cowsay "hey hello"
```
# 9: Configure Custom Message of the Day (MOTD)
Creates an automated text script file using the nano terminal text editor so that anyone who logs into the server is greeted by the animal banner, updates the system permission rules so it can run, applies the changes, and restarts the operating system.

```
sudo nano /etc/update-motd.d/40-cow

# (Inside the nano editor window, paste the following script code):
#!/bin/sh
cowsay "Amazon Linux 2023 AMI - Animals4Life"

# (After saving and closing out of nano, run):
sudo chmod 755 /etc/update-motd.d/40-cow
sudo update-motd
sudo reboot

<img width="951" height="349" alt="AFterRebooting ActualEC2Instance" src="https://github.com/user-attachments/assets/f91fc55b-0a1c-44f4-8e19-564dcf4f4ddb" />


```

### Step 2: Ensuring Data Consistency
To prevent snapshot fragmentation and file system corruption, the source EC2 instance must be Stopped prior to image creation. This completely freezes disk I/O operations and guarantees a clean, stable block capture.

### Step 3: The Bake
Initiate the CreateImage workflow inside the EC2 environment. AWS parallely generates:

An EBS Snapshot representing the exact point-in-time configuration of the 8 GB root volume.

A Custom AMI metadata mapping that wraps the snapshot into a reusable, regional deployment template.



### Step 4: Deployment Verification
Launch a completely fresh target EC2 instance referencing the newly baked AMI. The instance will boot up instantly functional, bypassing the software installation and code setup process.

### Validation
1. Reference Source Instance Configuration
Description: The baseline operational EC2 instance summary and assigned public access configurations.

<img width="956" height="376" alt="EC2_Running_instance" src="https://github.com/user-attachments/assets/33146018-6e33-4696-99e3-0dfd6eba94c8" />

<img width="951" height="349" alt="AFterRebooting ActualEC2Instance" src="https://github.com/user-attachments/assets/5d64f416-28e8-452b-8d92-3e216e6468e8" />

<img width="896" height="469" alt="ActualEC2Instance_WPSite" src="https://github.com/user-attachments/assets/0c6805d6-64f9-481c-b548-a4630a0eba8d" />


3. Initiating Image Capture (The Bake Action)
Description: Defining metadata attributes and configuring the state-capture criteria for the custom AMI Image template.

<img width="959" height="396" alt="CreateAMI-a" src="https://github.com/user-attachments/assets/b53c4f6b-091b-4303-9382-65d825e52b7c" />

<img width="959" height="325" alt="Create AMI-b" src="https://github.com/user-attachments/assets/d40e0e96-b7ab-444f-8b02-60ffb389d9bd" />

<img width="956" height="380" alt="create ami-c" src="https://github.com/user-attachments/assets/7de6f99c-0d07-4c3c-9ce7-b50c5751d9fc" />

5. Structural Mapping: AMI & Snapshot Association
Description: Verification of the registered AMI within the management console, explicitly displaying its logical link to the underlying backend EBS storage snapshot.

<img width="959" height="397" alt="LaunchInstanceFromAMI" src="https://github.com/user-attachments/assets/d117cda6-59b4-49bc-8743-151af60332cb" />

<img width="956" height="380" alt="SnapshotFromAMI" src="https://github.com/user-attachments/assets/f36f76d4-8054-445c-b87a-84e385606f21" />

7. End-to-End Verification:
Description: Accessing the newly cloned server via its distinct public IPv4 address, with full WordPress initialization and database mappings inherited without any manual server-side intervention.

<img width="958" height="373" alt="InstanceFromAMICreatedSuccess" src="https://github.com/user-attachments/assets/4829dd1d-bfa0-4aa4-b334-b88b77309818" />

<img width="959" height="412" alt="InstanceFromAMIValidateBrowserSuccess" src="https://github.com/user-attachments/assets/531866d3-44c4-415b-b857-4620916be1ee" />

<img width="866" height="470" alt="InstanceFromAMIValidateSuccessWPDialog" src="https://github.com/user-attachments/assets/2ac56b97-b2d2-4c1b-9907-45e1c0245cd3" />


### Cleanup Steps:
To adhere to cloud cost-optimization best practices and strictly maintain AWS Free Tier compliance boundaries, a mandatory deletion sequence was strictly executed immediately following verification:

1. Terminate all deployed operational EC2 compute instances.

2. Deregisterthe customized AMI metadata template to break the asset mapping lock.

3. Delete the underlying 8 GB block-level EBS Snapshot from storage components to eliminate residual storage fees.
