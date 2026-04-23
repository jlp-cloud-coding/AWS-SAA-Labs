## Overview:
This demo documents the setup of an Organization Trail for auditing and monitoring API activity across the entire account.

## Resource Summary
* **Trail Name:** `MyFirstCloudTrailDemo`
* **S3 Bucket Name:** `cloudtrail-myfirstdemo-s3-123`
* **CloudWatch Log Group:** `aws-cloudtraillogs-cw-myfirstdemo`
* **IAM Role:** `CloudTrailRoleForCloudwatchLogs-MyFirstCloudTrailDemo`

---

## Step-by-Step Implementation

### 1. Trail & S3 Configuration
1.  **Navigate** to the **CloudTrail** console.
2.  **Create Trail** using the name `MyFirstCloudTrailDemo`.
3.  **Storage:** Select **Create new S3 bucket** and enter `cloudtrail-myfirstdemo-s3-123`.
4.  **Log File Validation:** Enabled (Checked). This provides a digital signature (SHA-256) for each log file to ensure it hasn't been modified.

<img width="2108" height="3722" alt="Cloudtrail-create-page1" src="https://github.com/user-attachments/assets/495a6a5c-2e5c-4797-b9e1-f514f845d71b" />


## 2. CloudWatch Logs Integration
1.  **CloudWatch Logs:** Enabled.
2.  **Log Group Name:** `aws-cloudtraillogs-cw-myfirstdemo`.
3.  **IAM Role:** Select **Create new** and use the name `CloudTrailRoleForCloudwatchLogs-MyFirstCloudTrailDemo`.
4.  **Policy Details:** This role allows CloudTrail to communicate with CloudWatch by granting `logs:CreateLogStream` and `logs:PutLogEvents` permissions.

### 3. Event Selection
1.  **Event Type:** Selected **Management events**.
2.  **API Activity:** Both **Read** and **Write** are enabled.
3.  *Note: Data Events (like S3 object-level actions) are excluded to maintain Free Tier eligibility.*

<img width="2108" height="1454" alt="CT-Screen2" src="https://github.com/user-attachments/assets/f3374323-5a6c-4c44-a25b-981bb346bc20" />

<img width="1893" height="750" alt="CT-success-created" src="https://github.com/user-attachments/assets/96937b9c-a04b-4e64-a127-436500c08327" />


### 4. Verification & Testing
1.  **Wait:** Initial log delivery to S3 and CloudWatch typically takes 10–15 minutes.
2.  **S3 Verification:** Check the bucket `cloudtrail-myfirstdemo-s3-123`. You should see the `AWSLogs` folder structure appearing.
3.    - AWSLogs/ → [Your_Organization_ID]/ → [Account_ID]/
4.  **CloudWatch Verification:** Navigate to **Log groups > aws-cloudtraillogs-cw-myfirstdemo**. Open a Log Stream to view raw JSON events.
5.  **Event History:** Use the CloudTrail console's **Event History** tab to quickly search for recent actions like `CreateTrail` or `CreateBucket`.

---

## Cleanup Procedure:
1.  **Stop Logging:** In the Trails console, select `MyFirstCloudTrailDemo` and click **Stop logging**.
2.  **Delete Trail:** Delete the trail `MyFirstCloudTrailDemo`.
3.  **Empty & Delete S3 Bucket:** `cloudtrail-myfirstdemo-s3-123`.
4.  **Delete CloudWatch Log Group:** `aws-cloudtraillogs-cw-myfirstdemo`.
5.  **Delete IAM Role:** `CloudTrailRoleForCloudwatchLogs-MyFirstCloudTrailDemo`.

---

## Key Takeaways:
* **CloudTrail** provides the "Audit Trail" (Who did what?).
* **CloudWatch Logs** allows for real-time alerting based on CloudTrail events (e.g., alert when an unauthorized user tries to stop a trail).
* **Management Events** are the default and record operations like creating an EC2 instance or changing IAM policies.
