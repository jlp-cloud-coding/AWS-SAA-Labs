# AWS Hands-On Lab: Automated PII & Sensitive Data Detection with Amazon Macie & EventBridge

A hands-on implementation project demonstrating automated sensitive data discovery in Amazon S3 using **Amazon Macie**, event-driven notification routing via **Amazon EventBridge**, and email alerts delivered through **Amazon Simple Notification Service (SNS)**.

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

## Key Features Demonstrated
* **Automated S3 Scanning:** Using Amazon Macie to continuously monitor and scan S3 buckets for Personally Identifiable Information (PII), financial records, and credentials.
* **Managed vs. Custom Data Identifiers:** Leveraging AWS built-in classifiers (`cc.txt` detection) and evaluating regex-based custom identifiers for proprietary data formats.
* **Event-Driven Security Automation:** Routing security findings dynamically from Macie to EventBridge without polling overhead.
* **Instant Alerting Pipeline:** Triggering SNS notifications to deliver real-time security alerts directly to an administrator's email.

---

## Step-by-Step Implementation Guide

### Step 1: Set Up the Storage & Notification Pipeline
1. **Create an S3 Bucket:** Provision a dedicated S3 bucket to hold your test datasets.
2. **Create an SNS Topic:** 
   * Navigate to **Amazon SNS** and create a standard topic named `macie-alerts`.
   * Create an email subscription and confirm the endpoint via the confirmation link sent to your inbox.

### Step 2: Enable Amazon Macie & Upload Test Data
1. Navigate to the **Amazon Macie** console and click **Enable Macie** (utilizing the 30-day free trial).
2. Upload test files into your S3 bucket (e.g., sample text files containing mock credit card numbers like `cc.txt`).
3. Allow Macie's background automated discovery engine to evaluate the bucket contents.

### Step 3: Configure EventBridge Integration
1. Navigate to the **Amazon EventBridge** console and click **Create rule** on the default event bus.
2. Set **Event source** to **AWS services** (`Amazon Macie`).
3. Set **Event type** to **Macie Finding**.
4. Configure the target as your **SNS topic** (`macie-alerts`).

### Step 4: Verify & Test
* Modify or trigger data discovery so Macie raises a finding.
* Confirm that the raw JSON security payload successfully routes through EventBridge and arrives in your email inbox via SNS.

---

## Cleanup Instructions (Cost Control)

To ensure no ongoing charges or active monitoring costs accrue in your AWS account, execute the following cleanup steps:
1. **Disable Macie:** Go to Macie Settings and click **Disable Macie**.
2. **Empty & Delete S3 Bucket:** Empty all objects from your test bucket and delete the bucket.
3. **Delete EventBridge Rule:** Remove the `macie-finding-rule` from the Default event bus.
4. **Delete SNS Topic:** Delete the `macie-alerts` topic and associated subscriptions.
