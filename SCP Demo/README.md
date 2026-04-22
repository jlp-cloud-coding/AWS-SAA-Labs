## Overview
This lab demonstrates how to use Service Control Policies (SCPs) to restrict S3 access for a member account by applying the policy at the Organizational Unit (OU) level.

## Step 1: Create Organizational Unit (OU)
a. Go to AWS Organizations → Organizational Units
b. Create a new OU:
Name: Dev
<img width="945" height="372" alt="Cretae a DEV OU" src="https://github.com/user-attachments/assets/abcf03d7-ea86-4b98-8306-a29619a998a6" />


## Step 2: Move Account to OU
a. Go to Accounts
b. Select your Developer account
c. Click Move
d. Place it inside:
"Dev" OU just created in above step

<img width="953" height="377" alt="move to dev ou" src="https://github.com/user-attachments/assets/a0f8a015-dec1-4437-b0df-44a924900f12" />

## Step 3: Baseline Test (Before SCP)
a. From management account → Switch Role into Dev account
b. Go to S3
c. Perform actions:
  - Create a bucket named catpicscpdemo
  - Upload files
  - List buckets
✔ Confirm S3 access is working normally

<img width="926" height="353" alt="step 4 s3 bucket with a file in it" src="https://github.com/user-attachments/assets/c3fb9538-7266-4bed-bba0-104b3cfed876" />

## Step 4: Create SCP (Allow All Except S3)
a. Go to AWS Organizations → Policies
b. Enable Service Control Policies
c. Click Create policy
Add:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "*",
            "Resource": "*"
        },
        {
            "Effect": "Deny",
            "Action": "s3:*",
            "Resource": "*"
        }
    ]
}
```
<img width="958" height="379" alt="step 5 - create scp" src="https://github.com/user-attachments/assets/f5ea6767-8d49-4187-a80e-0b2f86807a1e" />

## Step 5: Attach SCP to OU
Attach the SCP to Dev OU
## Step 6: Test SCP Restriction
Switch role → Developer account
Try:
List S3 buckets

❌ Result: Access Denied
✔ SCP successfully blocks S3 even with admin access
<img width="956" height="377" alt="step 7 -attach this scp to the DEV OU" src="https://github.com/user-attachments/assets/e7c94b4d-2915-44fd-8473-6980f69159a8" />


## Step 7: Restore Access (Cleanup)
a. Go to AWS Organizations → Policies
b. Either: Detach the SCP and reattach FullAWSAccess
✔ Result: S3 access restored
c. Empty and delete the S3 bucket "catpicscpdemo"
d. Delete the SCP "AllowAllExceptS3"

## Key Learnings
1. SCP applied at OU level affects all accounts inside it
2. Explicit Deny overrides IAM permissions
3. IAM AdministratorAccess does NOT bypass SCP
4. SCP acts as a guardrail, not a permission grant
