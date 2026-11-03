# Automated PII & Sensitive Data Detection with Amazon Macie & EventBridge

A hands-on lab demonstrating automated sensitive data discovery in Amazon S3 using **Amazon Macie**, event-driven notification routing via **Amazon EventBridge**, and email alerts delivered through **Amazon Simple Notification Service (SNS)**.

---

## Architecture Overview
```
S3 Bucket (Sample Data)
│
▼
Amazon Macie (Automated Discovery & Managed Data Identifiers)
│
▼
Amazon EventBridge (Event Rule Match: "Macie Finding")
│
▼
Amazon SNS (macie-alerts Topic)
│
▼
Email Notification (Inbox Alert)
```

---

## Key Features
* **Automated S3 Scanning:** Using Amazon Macie to continuously monitor and scan S3 buckets for Personally Identifiable Information (PII), financial records, and credentials.
* **Managed vs. Custom Data Identifiers:** Leveraging AWS built-in classifiers (`cc.txt` detection) and evaluating regex-based custom identifiers for custom data formats such as license plates, employee details etc.
* **Event-Driven Security Automation:** Routing security findings from Macie to EventBridge without polling overhead.
* **Instant Alerting Pipeline:** Triggering SNS notifications to deliver real-time security alerts directly to the user's email.

---

## Step-by-Step Implementation

### Step 1: Set Up the Storage & Notification Pipeline
1. **Create an S3 Bucket:** Create an S3 bucket to hold the sample sensitive test data.

<img width="959" height="371" alt="s3-bucket" src="https://github.com/user-attachments/assets/cab33d64-2bb2-4ddb-b500-487d2c084d34" />

2. **Create an SNS Topic:** 
   * Create an SNS topic named `macie-alerts`.
   * Create an email subscription and confirm the endpoint via the confirmation link received in your  provided email inbox.
  
<img width="956" height="379" alt="Screenshot 2026-10-31 165551" src="https://github.com/user-attachments/assets/7f4a5971-8cbc-412f-85b3-da83f457fd67" />

<img width="959" height="385" alt="Screenshot 2026-10-31 165633" src="https://github.com/user-attachments/assets/b9099883-c2f4-4c86-9ca4-99bc5880019d" />

<img width="959" height="380" alt="Screenshot 2026-10-31 165726" src="https://github.com/user-attachments/assets/36d3a553-a7bd-407f-bdbf-526f1ce8fb92" />

<img width="958" height="379" alt="Screenshot 2026-10-31 165819" src="https://github.com/user-attachments/assets/9cb729af-78b4-41a0-b077-80f4c14e90a4" />

<img width="959" height="380" alt="Screenshot 2026-10-31 165851" src="https://github.com/user-attachments/assets/5852aef8-47d4-4b8f-9732-f908fb1a3095" />

<img width="731" height="364" alt="Screenshot 2026-10-31 165956" src="https://github.com/user-attachments/assets/0412afd0-4b07-4f74-b1e7-51debce62d8b" />

<img width="641" height="245" alt="Screenshot 2026-10-31 170026" src="https://github.com/user-attachments/assets/ef4efa70-c233-4113-a606-3dd5bfb8b9cd" />

### Step 2: Enable Amazon Macie & Upload Test Data
1. Navigate to the **Amazon Macie** console and click **Enable Macie** (utilizing the 30-day free trial).
2. Upload test files into your S3 bucket (e.g., sample text files containing mock credit card numbers like `cc.txt`).
3. Allow Macie's background automated discovery engine to evaluate the bucket content.

<img width="952" height="373" alt="amazon-macie-1" src="https://github.com/user-attachments/assets/25c5b59b-56ec-4c29-a384-2c550248eaca" />

<img width="955" height="380" alt="amazon-macie-2" src="https://github.com/user-attachments/assets/b1fddfd0-255e-451c-818a-d192216b62af" />

<img width="954" height="382" alt="macie-3" src="https://github.com/user-attachments/assets/ab19f19e-711b-450e-933b-aa04827a847e" />

<img width="954" height="382" alt="macie-4" src="https://github.com/user-attachments/assets/b3717166-b9f6-4316-acea-d9b126e3ef17" />

<img width="959" height="387" alt="Screenshot 2026-10-31 163558" src="https://github.com/user-attachments/assets/781527e9-0364-49ef-9d76-03249920ac7e" />

<img width="959" height="364" alt="Screenshot 2026-10-31 163636" src="https://github.com/user-attachments/assets/6ef955a6-f519-427d-aae2-902e2267b95d" />

<img width="959" height="388" alt="Screenshot 2026-10-31 163659" src="https://github.com/user-attachments/assets/4ecf011c-85e7-4a72-9b8b-9c01e50d2555" />

<img width="959" height="378" alt="Screenshot 2026-10-31 163716" src="https://github.com/user-attachments/assets/db04db80-58b9-41ee-901d-49e84b046d5e" />

<img width="959" height="384" alt="Screenshot 2026-10-31 163747" src="https://github.com/user-attachments/assets/9d30740a-fd6c-4866-8075-49afe5118aba" />

<img width="959" height="382" alt="Screenshot 2026-10-31 171647" src="https://github.com/user-attachments/assets/20631953-9866-4fc6-89e2-5471b7adbb72" />

<img width="959" height="383" alt="Screenshot 2026-10-31 171803" src="https://github.com/user-attachments/assets/5fa0df35-336b-4e84-b851-af508c0c07aa" />

<img width="959" height="377" alt="Screenshot 2026-10-31 171848" src="https://github.com/user-attachments/assets/aeb71d4f-d038-4085-b775-49a39db0aedb" />

<img width="959" height="377" alt="Screenshot 2026-10-31 171934" src="https://github.com/user-attachments/assets/e5a682b8-ab72-4e12-8546-e4e2e901f58e" />

<img width="959" height="382" alt="Screenshot 2026-10-31 171951" src="https://github.com/user-attachments/assets/85f7c0e6-7dcc-4d85-9a09-c3b7d191ff3c" />

### Step 3: Configure EventBridge Integration
1. Navigate to the **Amazon EventBridge** console and click **Create rule** on the default event bus.
2. Set **Event source** to **AWS services** (`Amazon Macie`).
3. Set **Event type** to **Macie Finding**.
4. Configure the target as the **SNS topic** (`macie-alerts`) created in previous step.

<img width="959" height="377" alt="Screenshot 2026-10-31 170236" src="https://github.com/user-attachments/assets/68e72860-54e8-4f82-b7bc-7a4b9156f7f6" />

<img width="959" height="374" alt="Screenshot 2026-10-31 171316" src="https://github.com/user-attachments/assets/7896d550-a239-44cc-b224-14d744e95b63" />

<img width="959" height="385" alt="Screenshot 2026-10-31 171330" src="https://github.com/user-attachments/assets/52fc2c39-5c44-4706-ae64-6d4455f63ac6" />

<img width="955" height="384" alt="Screenshot 2026-10-31 171357" src="https://github.com/user-attachments/assets/cab39af8-9473-47a3-8063-d6f7cc021e19" />

<img width="959" height="383" alt="Screenshot 2026-10-31 171427" src="https://github.com/user-attachments/assets/2e45f2bd-f90f-4288-a738-2eba99e18e74" />

<img width="959" height="380" alt="Screenshot 2026-10-31 171458" src="https://github.com/user-attachments/assets/13f5f12b-a01d-4d5d-aee5-acb77da8734f" />

<img width="959" height="380" alt="Screenshot 2026-10-31 171521" src="https://github.com/user-attachments/assets/7c50c0d6-092d-48b3-a40f-2051028ed8f4" />

### Step 4: Verify & Test
* Once Macie findings are complete select the S3 bucket and click on the Analyze findings button beside the bucket.
* Confirm that the raw JSON security payload successfully routes through EventBridge and arrives in your email inbox via SNS.

<img width="959" height="375" alt="Screenshot 2026-10-31 163815" src="https://github.com/user-attachments/assets/14720fc0-ba0b-49d2-93df-af9970f67511" />

<img width="959" height="380" alt="Screenshot 2026-10-31 165247" src="https://github.com/user-attachments/assets/7e011d5c-8049-4a87-a04e-b3c6ac6714cb" />

<img width="959" height="380" alt="Screenshot 2026-10-31 165357" src="https://github.com/user-attachments/assets/d717c36b-5615-4245-9f92-dae8c3936697" />

<img width="959" height="365" alt="Screenshot 2026-10-31 165440" src="https://github.com/user-attachments/assets/21f534e7-78a9-4f02-b50a-ee1ec0165966" />

<img width="697" height="317" alt="Screenshot 2026-11-03 145939" src="https://github.com/user-attachments/assets/6c728a63-b687-43b3-8e00-5b583ee7d475" />


---

## Screenshots

**S3**:


**Macie**:

<img width="952" height="373" alt="amazon-macie-1" src="https://github.com/user-attachments/assets/9e13f6ed-b5b2-4a49-8e1d-b0e6448ae986" />

<img width="955" height="380" alt="amazon-macie-2" src="https://github.com/user-attachments/assets/12dd8ee6-8cd9-4c55-be42-ac3f0d7f34b6" />

<img width="954" height="382" alt="macie-3" src="https://github.com/user-attachments/assets/09aa00fa-c8e5-4d23-8a1b-2881820608f6" />

<img width="954" height="382" alt="macie-4" src="https://github.com/user-attachments/assets/b3c5fd66-374e-4dda-926b-47fe43750330" />

<img width="959" height="387" alt="Screenshot 2026-10-31 163558" src="https://github.com/user-attachments/assets/7b8057b3-56b7-42e8-a810-cd610713d4c0" />

<img width="959" height="364" alt="Screenshot 2026-10-31 163636" src="https://github.com/user-attachments/assets/ce2e8349-c6f8-4f60-be93-09b861f675e7" />

<img width="959" height="388" alt="Screenshot 2026-10-31 163659" src="https://github.com/user-attachments/assets/809dcb19-aa19-499d-8cb1-9045313559c3" />

<img width="959" height="378" alt="Screenshot 2026-10-31 163716" src="https://github.com/user-attachments/assets/b671c434-c719-4913-b56e-9e2c25d6820f" />

<img width="959" height="384" alt="Screenshot 2026-10-31 163747" src="https://github.com/user-attachments/assets/59a5c27c-96a6-4cd7-8c3b-b3c31e254340" />

## Cleanup

Execute the following cleanup steps to ensure to costs incur in the AWS account:
1. **Disable Macie:** Go to Macie Settings and click **Disable Macie**.
2. **Empty & Delete S3 Bucket:** Empty all objects from your test bucket and delete the bucket.
3. **Delete EventBridge Rule:** Remove the `macie-finding-rule` from the Default event bus.
4. **Delete SNS Topic:** Delete the `macie-alerts` topic and associated subscriptions.
