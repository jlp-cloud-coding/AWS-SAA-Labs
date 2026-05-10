# Overview
This project demonstrates the implementation of a highly available static website using **Amazon S3 Cross-Region Replication (CRR)**. By configuring automated, asynchronous object replication between two geographically distant AWS regions, this setup ensures data redundancy and disaster recovery capabilities.

### Key Objectives:
* Configure source and destination S3 buckets in separate regions (`us-east-1` and `us-west-1`).
* Implement **S3 Versioning** as a prerequisite for replication.
* Establish an **IAM Role** for secure cross-region permission management.
* Validate real-time synchronization of static website content across global endpoints.

---

## Architecture
1.  **Source Bucket:** `us-east-1` (N. Virginia) – Hosts the primary static website.
2.  **Destination Bucket:** `us-west-1` (N. California) – Receives replicated objects.
3.  **Replication Engine:** Triggered by S3 Versioning events to copy new objects and metadata.

---

## Implementation Steps

### 1. Bucket Setup & Versioning
* Create two S3 buckets with unique global names
  
<img width="919" height="367" alt="sourceanddest" src="https://github.com/user-attachments/assets/52f6007f-a9a1-4c08-84fd-8c3f727c64b4" />

### 2. Cross-Region Replication (CRR) Configuration
* Configure a Replication Rule on the source bucket to target the destination bucket.

<img width="956" height="370" alt="enableBucketVersioning" src="https://github.com/user-attachments/assets/69d03af4-5c44-4701-9b8a-90a604fbf6d3" />

<img width="932" height="374" alt="enableversioningstep2" src="https://github.com/user-attachments/assets/b8242de0-b740-4414-b7f4-4f651d53a94e" />


* Enable **Bucket Versioning** on both source and destination buckets

<img width="884" height="374" alt="destinationbukcetstep3" src="https://github.com/user-attachments/assets/e416e973-d270-4b6d-a1d2-66a542906ac5" />

<img width="938" height="376" alt="destinationbucketstep4-versioning enabled" src="https://github.com/user-attachments/assets/d58f5119-5602-47d5-9ce9-b262e6800606" />

* Note: Versioning is mandatory for CRR to track object states and ensure consistency across regions.*
* Create a new IAM Role to allow S3 to perform `ReplicateObject` actions from source to destination.

 <img width="938" height="374" alt="create anew iam role-step5" src="https://github.com/user-attachments/assets/b63846dd-f1dd-414e-907c-7c47bb1ee145" />
 * CRR successfully created:

<img width="910" height="378" alt="finalstep" src="https://github.com/user-attachments/assets/7157f5f7-daf2-4f2a-99dc-2220399ffb2d" />

<img width="907" height="358" alt="success" src="https://github.com/user-attachments/assets/a5438480-5ac0-463d-82fa-9f9ee28e2bf4" />


### 3. Static Website Hosting
* Enable Static Website Hosting on both buckets.
* Disable Block Public Access Settings
* Configure Bucket Policies to allow `s3:GetObject` for public read access to the website files.

---

## Validation

The following screenshots demonstrates the same asset (`cat.jpg`) being served from two different AWS regional endpoints. This confirms the replication was successful and the metadata (Public Read access) was maintained.

<img width="760" height="467" alt="finalTestsource" src="https://github.com/user-attachments/assets/0f33ce21-3c49-4660-8837-ccce2fe333e1" />

<img width="731" height="463" alt="finalTestDest" src="https://github.com/user-attachments/assets/5540b9b0-9adc-471c-91be-f522da817df1" />

### Versioning & Event-Driven Updates
When a new version of the image was uploaded to the source bucket, S3 automatically triggered a replication task. Both the primary and failover sites updated to the latest version without manual intervention.

<img width="769" height="461" alt="sourceTruffles" src="https://github.com/user-attachments/assets/0d777f69-83d4-4d76-b287-ad8114c58f08" />

<img width="724" height="464" alt="destTruffles" src="https://github.com/user-attachments/assets/0e634e7d-ab81-4874-9c6a-729f0f7db6d7" />

---

## Cleanup
Immediately after testing:
1.  **Empty and Delete S3 Buckets:** Permanently delete all object versions and buckets.
2.  **Remove IAM Role:** Delete the IAM role created during CRR creation 

---
