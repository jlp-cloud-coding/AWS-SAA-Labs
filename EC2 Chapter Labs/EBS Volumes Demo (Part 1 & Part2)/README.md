# AWS EC2 Storage Deep Dive: EBS Volumes & Lifecycle Persistence

This repository documents a practical lab demonstrating how to initialize, format, mount, and manage **Amazon Elastic Block Store (EBS)** volumes on an EC2 instance. It highlights the differences between volatile and persistent storage configurations in Linux using the filesystem table (`fstab`).

---

## 🏗️ Architecture Overview



* **Elastic Block Store (EBS):** Network-attached virtual block storage. It acts like a network-accessible USB flash drive. Data persists independently of the life of the EC2 instance (survives instance stops and starts).
* **EC2 Instance Store:** Ephemeral, physical storage directly attached to the underlying hardware host. High performance, but data is permanently wiped if the instance is stopped or terminated.

---

## 📌 Architectural Note: Xen vs. AWS Nitro Device Naming

During this lab, the attached EBS volume appeared in the operating system as `/dev/nvme1n1` instead of the legacy `/dev/xvdf` seen in older documentation. 

* **Why?** Legacy AWS instances (like the `t2` family) utilize the Xen hypervisor, which registers storage devices as Xen Virtual Disks (`/dev/xvd*`). 
* **Modern Reality:** Current AWS CloudFormation templates deploy modern, current-generation instances (like the `t3` family) built on the **AWS Nitro System**. Nitro exposes virtual media using native hardware **NVMe** (Non-Volatile Memory Express) controllers, which Linux registers as `/dev/nvme*`. The commands below reflect this optimized Nitro path.

---

## Prerequisite:
This lab was performed using CloudFormation template shared from the EBS volumes demo chapter from adrian cantrill's saa course. A bunch of resources were created after importing this template such as EC2 Instances 2 in AZ-A and 1 in AZ-B, EBS Volumes, Security Groups etc.

<img width="959" height="356" alt="CF Stack" src="https://github.com/user-attachments/assets/40e7ae5f-ff90-4618-8d2a-1430b025f569" />

<img width="956" height="353" alt="ec2 instances" src="https://github.com/user-attachments/assets/4211fd00-8bb1-4322-b3e3-99d52dcd4156" />

## 🛠️ Step-by-Step Lab Execution

## Create an EBS Volume manually via console and attach it to EC2 instance 1 in AZ-A:

EBS Volume Creation:

<img width="959" height="369" alt="step 3 - ebs create vol 1" src="https://github.com/user-attachments/assets/a15f4fb5-3b30-4eac-b8fe-6f6f1157fc86" />

<img width="959" height="377" alt="step3 - ebs create vol 2" src="https://github.com/user-attachments/assets/d7ff1974-4ad0-476d-a606-7715f46bbc73" />

Attach to EC2 instance 1:

<img width="959" height="385" alt="Attach EBS Vol to EC2 1 instance" src="https://github.com/user-attachments/assets/becd3667-2f13-4822-a9b1-13bed53fba57" />

<img width="988" height="377" alt="attach vol step 2" src="https://github.com/user-attachments/assets/445561c7-fd84-461b-8d2d-cee131a935dd" />

Connect to EC2 instance 1 in AZ-A via EC2 Instance Connect in the browser terminal:

<img width="959" height="376" alt="step 4 - ec2 instance connect" src="https://github.com/user-attachments/assets/71c1d838-412c-402b-930b-54c8127f8dda" />

### Phase 1: Initializing and Mounting the EBS Volume (Instance 1)
When a raw EBS volume is first attached to an EC2 instance, it has no file system layout. We must verify its block status, format it, and create a logical mount directory. Run below commands in the browser terminal by connecting via EC2 Instance Connect:

```bash
# 1. List all available block storage devices to find the new unformatted volume
lsblk

# 2. Inspect the raw device status (Should return "data", meaning it's completely empty)
sudo file -s /dev/nvme1n1

# 3. Create an XFS filesystem on the raw NVMe block device
sudo mkfs -t xfs /dev/nvme1n1

# 4. Verify the filesystem creation (Should now show XFS filesystem data)
sudo file -s /dev/nvme1n1

# 5. Create a target mount directory in the root partition
sudo mkdir /ebstest

# 6. Mount the physical volume to the newly created directory track
sudo mount /dev/nvme1n1 /ebstest

# 7. Move into the storage track and write a mock application data file
cd /ebstest
sudo nano mysuccessfile.txt
# [Added test message "My first text message!!" inside nano editor, then saved and exited]

# 8. List directory permissions and contents to verify file placement
ls -la

```

### Phase 2: Configuring Persistence Across Reboots
By default, manual mounts are temporary and disappear when the OS restarts. To ensure the operating system automatically maps the EBS volume on boot, we must configure /etc/fstab using the device's UUID (Universally Unique Identifier).

```bash

# 1. Trigger a safe reboot of the instance to demonstrate standard mount loss
sudo reboot

# [After reconnecting to Instance 1 via EC2 Instance Connect browser terminal]

# 2. Check disk space usage - checks the amount of disk space available on the filesystem with each file name's argument 
df -k
After running the above command notice the /ebstest mount is missing from active paths. Because we used mount command to manually mount our ebs volume into the ebs test folder,so it doesn't automatically mount the file system when the instance restarts. So we need to configure it to auto mount once the instance restarts. To do that we need to get the unique id of the ebs volume using the next command.

# 3. Query the system to find the persistent hardware UUID of the volume
sudo blkid
UUID: ebadbc61-3577-43e2-92af-d3efb622aa6a

# 4. Open the system's filesystem table configuration
sudo nano /etc/fstab

```

## Configuration addition appended to the bottom of /etc/fstab:
UUID=YOUR_VOLUME_UUID_HERE  /ebstest  xfs  defaults,nofail 
UUID=ebadbc61-3577-43e2-92af-d3efb622aa6a /ebstest xfs defaults,nofail

## 💡 The nofail Flag: Adding nofail ensures that if this specific EBS volume is ever detached from the instance in the AWS console, the EC2 instance will still boot up smoothly rather than crashing during the boot phase due to a missing disk map.

```bash
# 5. Verify the fstab configuration file has no errors and mount the drive paths. This will perform mount of all the volumes listed in the fstab file
sudo mount -a

#5a. Run df -k and now we can see the ebs volume 

# 6. Verify data integrity inside the permanent directory path
cd /ebstest
ls -la  # The mysuccessfile.txt is successfully retained!
This shows that the data on this file system is persistent and its available even after reboot this ec2 instance.
```

## 🔄 Multi-Instance Portability (Instances 2 & 3)
Because EBS volumes are network-isolated components detached from individual CPU hosts, they can be unmounted and passed around between different instances within the same Availability Zone.

Now stop the first EC2 instance in AZ-A and detach the EBS volume from this instance.

<img width="955" height="377" alt="step5-stop ec2 instance 1 in azA" src="https://github.com/user-attachments/assets/ce13e3a9-aeb7-4cb1-804b-a5b7055286e9" />

<img width="959" height="374" alt="step6-detach ebs volume from instance1 in azA" src="https://github.com/user-attachments/assets/0b896075-035d-4192-8c4c-133f2ab59827" />

And attach this ebs volume to another EC2 instance 2 in AZ-A and connect to it via EC2 Instance connect in the browser window.

<img width="959" height="373" alt="step7-reattch the ebs volume to instance2 in azA" src="https://github.com/user-attachments/assets/32de5868-fab9-4c06-a6f2-d0169d8e8981" />


```bash

# 1. Optional - Confirm the newly attached volume is visible via block lists
lsblk 

# 2. Check file type structure (Should instantly display existing XFS format)
When attaching this pre-formatted disk to clean nodes (Instance 2 in AZ-A and Instance 3 in AZ-B), we do not run mkfs again (which would wipe the data). Instead, we simply mount it directly.

If we run the below command we can see our file system. We dont need to go through the proces of creating the file system again, as EBS volumes persist past the lifecycle of an EC2 instance.

sudo file -s /dev/nvme1n1

# 3. Create a matching directory track on the new instance
sudo mkdir /ebstest

# 4. Mount the volume directly to view existing files
sudo mount /dev/nvme1n1 /ebstest

# 5. Confirm original data created on Instance 1 is safely readable
cd /ebstest
ls -la
We can see that our text file mysuccessfile.txt and its contents exist.
```
Now stop EC2 instance 2 in AZ-A and detach the EBS volume. Now we can also attach this EBS volume to EC2 intance 3 in AZ-B using Snapshots. (We cannot directly attach without Snapshots to instance 3 which is in AZ-B because EBS volume is created in AZ-A and it is strictly AZ specific. Inorder to associate this volume outside the AZ we can create a Snapshot using S3).

### 🏗️ The Cross-AZ Migration Architecture - Snapshots<img width="959" height="377" alt="step8- create snapshot" src="https://github.com/user-attachments/assets/1e298d7d-8735-4f68-88d3-00c2eed6e73b" />

When you take a snapshot of an EBS volume, AWS takes a point-in-time backup of the data and stores it incrementally inside Amazon S3. Because S3 is a region-wide service (not locked to a single AZ), that snapshot becomes accessible from any Availability Zone within that region.

To attach the data to Instance 3 in AZ-B, you follow a 3-step pipeline:

Snapshot: Take a snapshot of the volume in AZ-A (copies data to S3).

<img width="959" height="377" alt="step8- create snapshot" src="https://github.com/user-attachments/assets/eae313b4-637b-457e-b543-d7ab816dfb40" />

<img width="959" height="356" alt="step 8b - create snapshot" src="https://github.com/user-attachments/assets/0ccf4b4d-7296-4d68-b4bc-2c3cc5970117" />

<img width="956" height="373" alt="step 8c- create snapshot" src="https://github.com/user-attachments/assets/53b28176-1eff-498b-a2b3-6cf35503f102" />

Restore: Create a brand new EBS volume from that snapshot, but explicitly choose AZ-B as the target location.

<img width="959" height="379" alt="step 9- Create volume from snapshot" src="https://github.com/user-attachments/assets/50b6891c-f691-46cb-96b3-dee31ddab87c" />

<img width="959" height="376" alt="step9b- create vol from snapshot" src="https://github.com/user-attachments/assets/49fd744d-d920-48c4-bb9a-c3157cefb670" />

<img width="955" height="378" alt="step9c" src="https://github.com/user-attachments/assets/bed9e9ab-5d52-4d45-95e8-9a9640f9af71" />

<img width="959" height="373" alt="step9d" src="https://github.com/user-attachments/assets/24395c30-3497-466b-8d77-3a25fdd68ae0" />

Attach: Plug that new volume directly into Instance 3.

<img width="956" height="374" alt="step10-attach ebstestvolumeAZB to ec2 instance in AZB" src="https://github.com/user-attachments/assets/bef2aa32-3290-4f40-8d00-ce806c8c463d" />

<img width="959" height="379" alt="step10b" src="https://github.com/user-attachments/assets/35715022-b4fb-4883-b007-311f2d1e1985" />

## Phase 1: Verification and Mounting
Because this new volume was created from a snapshot of your original volume, it already has a filesystem and your files on it. You do not run mkfs (formatting) because that would wipe your data!

```bash
# 1. Verify the new volume is attached and recognized by the OS
lsblk

# 2. Check the file system structure
# (Unlike the first lab, this will immediately say "XFS filesystem data" because it's a clone)
sudo file -s /dev/nvme1n1

# 3. Create the mount directory on Instance 3
sudo mkdir /ebstestAZB

# 4. Mount the volume directly
sudo mount /dev/nvme1n1 /ebstestAZB

# 5. Check your files
cd /ebstestAZB
ls -la  # We can see mysuccessfile.txt here in AZ-B
```

### Key Takeaways
🎯 Key Takeaways
Storage Decoupling: EBS data operates outside the lifecycle of the instance OS. It acts completely independently of single host architectures.

Device Translations: Understanding the transition from traditional hypervisors (Xen) to modern cloud arrays (AWS Nitro) ensures engineering commands are targeted at correct device mappings (/dev/nvme*).

Fstab Resiliency: Production cloud instances should always leverage the nofail parameter on secondary storage volumes to maintain server availability during block modifications.
