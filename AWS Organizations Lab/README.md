## Overview
This lab demonstrates how to use AWS Organizations to manage multiple AWS accounts and enable cross-account access using IAM roles.
## Architecture
1. Management Account (Payer Account)
2. Email: pavani.gorthy93@gmail.com
3. IAM user: iamadmin
4. Controls AWS Organization
  -Member Account (Dev Account)
  - Email: pavani.gorthy93+developer@gmail.com
  - Created inside AWS Organizations
    
## Step 1: Create AWS Organization
- Login to management account (iamadmin)
- Go to AWS Organizations
- Choose All needed features
- Click Create Organization

## Step 2: Create new Developer Account
1. Go to AWS Organizations → Accounts
2. Click Add account → Create account
3.Enter:
  - Account name: Developer
  - Email: pavani.gorthy93+developer@gmail.com
4. Click Submit

<img width="1905" height="722" alt="org-1" src="https://github.com/user-attachments/assets/324bed12-e18c-4a16-b169-5e1d3000ae72" />

## Step 3: Send invitation (conditional step)
This step depends on how the account was added to AWS Organizations.
🔹 Case 1: Invited Existing Account

If an already existing AWS account was invited:
Open Gmail inbox
Accept AWS invitation
🔹 Case 2: Created New Account via AWS Organizations

If the account was created from within AWS Organizations:

✔ No email acceptance is required
✔ AWS automatically:

- Creates the account
- Adds it to the organization
- Assigns **OrganizationAccountAccessRole**

## Step 4: Switch Role (Automatic Org Role)

From management account:

- Click IAM user → Switch Role
Enter:
  - Account ID: Developer account ID you just created
  - Role name: OrganizationAccountAccessRole
  - Switch role

<img width="667" height="388" alt="ORG-2" src="https://github.com/user-attachments/assets/610a0735-a124-4a17-87c3-2a7765d418ed" />

✔ You are now in Dev account

<img width="952" height="312" alt="ORG-3" src="https://github.com/user-attachments/assets/513acea0-feea-49e4-bbef-c2f6af5bd48b" />


## Step 5: Create Manual IAM Role (Dev Account)
I didn't have another existing account to send an invite. So to simulate the CrossAccount Access behaviour I used the same developer account by manually creating the role and adding general iamadmin account as the trusted entity.

- Inside Dev account:

1. Go to IAM → Roles → Create Role
2.Select Another AWS account
3.Enter management account ID
4.Attach policy: AdministratorAccess
5. Role name: CrossAccountAdminRole
6. Create role
7. 
<img width="1913" height="691" alt="crossacc-1" src="https://github.com/user-attachments/assets/46444c95-3010-41ba-8be5-4128e90b4c4e" />

<img width="1867" height="746" alt="crossacc-2" src="https://github.com/user-attachments/assets/e07c0082-d351-40d3-bbcf-1c52e461bd9f" />


## Step 6: Switch Role (Manual Role)

From management account:

1. Click IAM user → Switch Role
Enter:
2. Account ID: Developer account ID
3. Role name: CrossAccountAdminRole
4. Click Switch

✔ Access Dev account via custom role you created
<img width="1905" height="774" alt="crossacc-3" src="https://github.com/user-attachments/assets/113e3a96-59e6-4dde-8625-6ea9a7483bf7" />

## Step 7: Delete Manual Role (Cleanup)

Inside Developer account:

Go to IAM → Roles
Delete CrossAccountAdminRole

## Note:

Observed that though Role disappears from AWS it may still appear in console switch history (UI cache)
## Key Concepts Learned
1. AWS Organizations
Centralized multi-account management
Simplifies Dev/Test/Prod separation
2. Automatic Role
OrganizationAccountAccessRole
Created automatically for new accounts in org
3. Manual Role
Created in existing accounts
Requires trust relationship setup
4. Cross-account Access
Uses IAM roles (not IAM users)
Based on trust between accounts
5. Role Switching
Allows access between accounts without logging out
