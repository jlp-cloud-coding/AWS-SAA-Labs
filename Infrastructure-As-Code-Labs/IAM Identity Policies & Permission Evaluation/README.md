# AWS IAM Identity Policies & Permission Evaluation Lab

## 🎯 Objective
This lab demonstrates the **Evaluation Logic** of AWS IAM policies. It specifically proves how an **Explicit Deny** overrides any **Explicit Allow**, and how permissions are calculated when multiple policy types (Managed vs. Inline) are attached to a user.

## 🏗️ Architecture & Resources
The environment was deployed using AWS CloudFormation and included:
* **IAM User:** `Sally` (Representing a person with limited administrative duties).
* **S3 Buckets:** * `catpics` (High-sensitivity bucket).
  * `animalpics` (General-access bucket).
* **Managed Policy:** `AllowAllS3ExceptCats` — Allows full S3 access but explicitly denies access to the `catpics` bucket.

## 🧪 The Testing Process

### Phase 1: Implicit Deny
* **Action:** Logged in as user `Sally` with no policies attached.
* **Observation:** Sally could not see any buckets or EC2 resources.
* **Lesson:** AWS follows a "Zero Trust" model; without an explicit allow, the result is an **Implicit Deny**.

### Phase 2: Explicit Allow (Inline Policy)
* **Action:** Attached an Inline Policy `s3_fulladmin.json` granting `s3:*` on all resources.
* **Observation:** Sally successfully viewed and uploaded images to both the `catpics` and `animalpics` buckets.
* s3_fulladmin.json:
  ```
  {
   "Version":"2012-10-17",
   "Statement":[
      {
         "Sid":"statement1",
         "Effect":"Allow",
         "Action":[
            "s3:*"
         ],
         "Resource":[
            "arn:aws:s3:::*"
         ]
       }
    ]
}

### Phase 3: The "DAD" Rule (Managed Policy)
* **Action:** Deleted the inline policy and attached the Managed Policy `AllowAllS3ExceptCats`.
* **Observation:** Sally could access `animalpics` but received a **Permission Denied** error when trying to access `catpics`.
* **Lesson:** **Explicit Deny always wins.** Even though the policy granted access to all S3 buckets (`s3:*` on `*`), the specific Deny on the `catpics` ARN took precedence.

## 🛠️ Key Technical Concepts
* **Policy JSON Structure:** Utilized `Effect`, `Action`, and `Resource` (ARN) fields.
* **ARN Formatting:** Used `arn:aws:s3:::bucket-name` and `arn:aws:s3:::bucket-name/*` to cover both bucket and object level permissions.
* **Infrastructure as Code:** Leveraged CloudFormation for repeatable resource deployment.

## 🧹 Cleanup
To avoid costs, all resources were deleted by:
1. Emptying S3 buckets.
2. Deleting the CloudFormation Stack (which automatically removed the IAM user and policies).
