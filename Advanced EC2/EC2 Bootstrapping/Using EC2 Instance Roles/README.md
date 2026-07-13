# AWS EC2 Instance Roles (IAM Roles for EC2)

## 📌 Project Overview

This lab demonstrates the recommended AWS practice for granting permissions to EC2 instances.

Instead of manually configuring long-term AWS credentials using `aws configure`, an **IAM Role** is attached to the EC2 instance through an **Instance Profile**. AWS automatically provides temporary credentials to the instance, allowing the AWS CLI and SDKs to securely access AWS services.

---

# Architecture

```
          IAM Role
     (A4LInstanceRole)
             │
             ▼
     Instance Profile
             │
             ▼
        EC2 Instance
             │
             ▼
 Temporary Credentials
   (via IMDS)
             │
             ▼
 AWS CLI / AWS SDK
             │
             ▼
 Amazon S3
```

---

# Lab Environment

CloudFormation provisions:

- Custom A4L VPC
- Public subnet
- Amazon Linux EC2 instance

Initially, the EC2 instance has **no IAM Role attached**.

---

# Step 1 — Verify No AWS Credentials

After connecting using **EC2 Instance Connect** via the browser terminal, run:

```bash
aws s3 ls
```

Output:

```
Unable to locate credentials. You can configure credentials by running: ```aws configure ```
```
<img width="958" height="380" alt="1" src="https://github.com/user-attachments/assets/43f6db9f-6423-437b-b0b3-e389e22505a4" />


At this point, the EC2 instance has no AWS credentials available.

> Although `aws configure` would solve this problem, storing long-term access keys on an EC2 instance is **not recommended** and is considered a security anti-pattern.

---

# Step 2 — Create an IAM Role

Create an IAM Role:

```
A4LInstanceRole
```

Grant the required permissions (for this lab, S3 read only access).

AWS automatically creates an **Instance Profile** associated with the role.

---

# Step 3 — Attach the Role to the EC2 Instance

Attach the **Instance Profile** to the running EC2 instance.

No reboot is required.

Within a short time, AWS automatically makes temporary credentials available to the instance.

<img width="958" height="377" alt="2" src="https://github.com/user-attachments/assets/d58e03cc-99ae-4f55-8f04-2186f6fb2d75" />

<img width="959" height="260" alt="3" src="https://github.com/user-attachments/assets/7067b609-909e-4ee4-9ed5-de8131fa919a" />

---

# Step 4 — Test Again

Run the same command:

```bash
aws s3 ls
```

This time the command succeeds because the AWS CLI automatically retrieves temporary credentials from the EC2 Instance Metadata Service (IMDS).

<img width="959" height="374" alt="4" src="https://github.com/user-attachments/assets/b60df45f-0b5f-415b-b11f-2c77ee72bb97" />


---

# Inspecting Temporary Credentials

The credentials provided by the IAM Role can be viewed through the **Instance Metadata Service (IMDS)**.

## List Attached IAM Role

```bash
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

Output:

```
A4LInstanceRole
```
<img width="956" height="379" alt="5" src="https://github.com/user-attachments/assets/e089894a-5636-441f-84dd-e31bfcbe305e" />



---

## View Temporary Credentials

```bash
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/A4LInstanceRole
```

Example output:

```json
{
  "AccessKeyId": "...",
  "SecretAccessKey": "...",
  "Token": "...",
  "Expiration": "..."
}
```

AWS automatically rotates these temporary credentials before they expire.

<img width="956" height="379" alt="5" src="https://github.com/user-attachments/assets/0fbe3227-1139-43b3-9d00-bcc5d128e255" />


---

# Why Use IAM Roles Instead of `aws configure`?

| `aws configure` | IAM Role |
|-----------------|----------|
| Stores long-term credentials | Uses temporary credentials |
| Manual credential management | Automatically managed by AWS |
| Keys may be leaked or forgotten | No embedded credentials |
| Requires key rotation | Automatic credential rotation |
| Not recommended for EC2 | AWS Best Practice ✅ |

---

# Key Takeaways

- Never store long-term AWS access keys on EC2 instances.
- Use **IAM Roles** to grant permissions securely.
- AWS automatically provides temporary credentials through the **Instance Metadata Service (IMDS)**.
- The AWS CLI and SDKs retrieve these credentials automatically, so no manual configuration is required.
- This approach improves security, simplifies credential management, and follows AWS best practices.
