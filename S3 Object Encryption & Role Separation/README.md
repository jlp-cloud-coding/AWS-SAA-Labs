# 🔐 S3 Object Encryption Lab (SSE-S3, SSE-KMS, AWS-Managed vs Customer-Managed KMS Keys, Role Separation)

This lab demonstrates Amazon S3 encryption behavior using:

* SSE-S3 (S3-managed encryption)
* SSE-KMS with AWS-managed key (`aws/s3`)
* SSE-KMS with customer-managed KMS key (CMK)
* Bucket default encryption
* IAM + KMS role separation

---

# 📌 Overview

We explored:

* How Amazon S3 encrypts objects at rest
* Difference between SSE-S3 and SSE-KMS
* AWS-managed vs customer-managed KMS keys
* IAM + KMS permission interaction
* Role separation enforcement
* Bucket-level default encryption behavior

---

# Step 1: Create S3 Bucket

Created a bucket in Amazon Web Services S3:

* Block Public Access → disabled (lab only)
* Versioning → optional
* Uploaded test images

---

# Step 2: SSE-S3 Encryption Object

Uploaded Object 1 with:

* Encryption: **SSE-S3 (AES-256)**

### What happens:

* S3 automatically:

  * generates and manages encryption keys
  * encrypts object at rest

✔️ No KMS involvement

---

# 🔐 Step 3: SSE-KMS with AWS-Managed Key

Uploaded Object 2 with:

* Encryption: **SSE-KMS**
* Key: `aws/s3` (AWS-managed key)

### Key characteristics:

* Managed by AWS Key Management Service
* No visibility into key policy
* Fully automated key lifecycle

### Flow:

```text id="kms_aws_managed_flow"
S3 → requests DEK from KMS (aws/s3) → encrypts object → stores encrypted object + encrypted DEK
```

---

# Step 4: SSE-KMS with Customer-Managed Key (CMK)

Uploaded Object 3 using:

* Custom KMS key (created manually)

### Key characteristics:

* You control:

  * key policy
  * IAM access
  * usage permissions
* Supports auditing via CloudTrail

### Flow:

```text id="kms_cmk_flow"
S3 → KMS (CMK) → generate DEK → encrypt object → store encrypted object + encrypted DEK
```

---

# AWS-Managed vs Customer-Managed KMS Keys

| Feature            | AWS-Managed Key (`aws/s3`) | Customer-Managed Key (CMK)    |
| ------------------ | -------------------------- | ----------------------------- |
| Created by         | AWS                        | You                           |
| Key policy control | ❌ No                       | ✅ Yes                         |
| IAM control        | Limited                    | Full                          |
| Rotation           | Automatic                  | Optional (enabled by default) |
| Best for           | Simple use cases           | Security / compliance         |

---

# Important Clarification

👉 Both key types are:

* **Equally secure cryptographically**
* Difference is **control, not strength**

---

# Step 5: IAM Explicit DENY Test

Added inline IAM policy:

* Explicit DENY for KMS actions

### Results:

| Action        | SSE-S3    | SSE-KMS   |
| ------------- | --------- | --------- |
| Read object   | ✅ Allowed | ❌ Denied  |
| Delete object | ✅ Allowed | ✅ Allowed |

---

# Key Insight

👉 SSE-KMS requires:

* S3 permission (`s3:GetObject`)
* KMS permission (`kms:Decrypt`)

---

# Step 6: Bucket Default Encryption

Configured bucket-level encryption:

* SSE-S3 OR SSE-KMS as default option

### Behavior:

* If object upload has no encryption specified:

  * bucket default encryption is applied

---

# Step 7: KMS Key Policy Behavior

When creating CMK:

* Key administrators = manage key
* Key users = use key
* If left blank → account-level trust applies

### Important rule:

👉 KMS requires explicit trust via:

* Key policy AND IAM policy

---

# Key Takeaways

## SSE-S3

* Fully managed by S3
* No KMS involvement

## SSE-KMS

* Uses AWS KMS
* Requires IAM + KMS permissions

## AWS-Managed KMS Key

* Fully automated (`aws/s3`)
* No control over policy

## Customer-Managed KMS Key

* Full control
* Best for compliance + auditing

## Role Separation

* S3 stores object
* KMS controls encryption keys
