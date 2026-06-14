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

## 🛠️ Step-by-Step Lab Execution

### Phase 1: Initializing and Mounting the EBS Volume (Instance 1)
When a raw EBS volume is first attached to an EC2 instance, it has no file system layout. We must verify its block status, format it, and create a logical mount directory.

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
# [Added test message inside nano editor, then saved and exited]

# 8. List directory permissions and contents to verify file placement
ls -la

```

### Phase 2: Configuring Persistence Across Reboots
By default, manual mounts are temporary and disappear when the OS restarts. To ensure the operating system automatically maps the EBS volume on boot, we must configure /etc/fstab using the device's UUID (Universally Unique Identifier).

```bash

# 1. Trigger a safe reboot of the instance to demonstrate standard mount loss
sudo reboot

# [After reconnecting to Instance 1 via EC2 Instance Connect browser terminal]

# 2. Check disk space usage (Notice the /ebstest mount is missing from active paths)
df -k

# 3. Query the system to find the persistent hardware UUID of the volume
sudo blkid

# 4. Open the system's filesystem table configuration
sudo nano /etc/fstab

```

## Configuration addition appended to the bottom of /etc/fstab:
UUID=YOUR_VOLUME_UUID_HERE  /ebstest  xfs  defaults,nofail  0  2

## 💡 The nofail Flag: Adding nofail ensures that if this specific EBS volume is ever detached from the instance in the AWS console, the EC2 instance will still boot up smoothly rather than crashing during the boot phase due to a missing disk map.

```bash
# 5. Verify the fstab configuration file has no errors and mount the drive paths
sudo mount -a

# 6. Verify data integrity inside the permanent directory path
cd /ebstest
ls -la  # The amazingtestfile.txt is successfully retained!
```

## 🔄 Multi-Instance Portability (Instances 2 & 3)
Because EBS volumes are network-isolated components detached from individual CPU hosts, they can be unmounted and passed around between different instances within the same Availability Zone.

When attaching this pre-formatted disk to clean nodes (Instance 2 and Instance 3), we do not run mkfs again (which would wipe the data). Instead, we simply mount it directly:

```bash

# 1. Confirm the newly attached volume is visible via block lists
lsblk 

# 2. Check file type structure (Should instantly display existing XFS format)
sudo file -s /dev/nvme1n1

# 3. Create a matching directory track on the new instance
sudo mkdir /ebstest

# 4. Mount the volume directly to view existing files
sudo mount /dev/nvme1n1 /ebstest

# 5. Confirm original data created on Instance 1 is safely readable
cd /ebstest
ls -la
```

### Key Takeaways
🎯 Key Takeaways
Storage Decoupling: EBS data operates outside the lifecycle of the instance OS. It acts completely independently of single host architectures.

Device Translations: Understanding the transition from traditional hypervisors (Xen) to modern cloud arrays (AWS Nitro) ensures engineering commands are targeted at correct device mappings (/dev/nvme*).

Fstab Resiliency: Production cloud instances should always leverage the nofail parameter on secondary storage volumes to maintain server availability during block modifications.
