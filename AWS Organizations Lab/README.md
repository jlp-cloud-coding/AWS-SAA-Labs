## Overview
This lab demonstrates how to use AWS Organizations to manage multiple AWS accounts and enable cross-account access using IAM roles.
## Architecture
Management Account (Payer Account)
Email: pavani.gorthy93@gmail.com
IAM user: iamadmin
Controls AWS Organization
Member Account (Dev Account)
Email: pavani.gorthy93+developer@gmail.com
Created inside AWS Organizations
## Step 1: Create AWS Organization
Login to management account (iamadmin)
Go to AWS Organizations
Click Create Organization
Choose All features
## Step 2: Create new Developer Account
Go to AWS Organizations → Accounts
Click Add account → Create account
Enter:
Account name: Developer
Email: pavani.gorthy93+developer@gmail.com
Submit

✔ AWS automatically creates:

OrganizationAccountAccessRole
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

Creates the account
Adds it to the organization
Assigns **OrganizationAccountAccessRole**

## Step 4: Switch Role (Automatic Org Role)

From management account:

Click IAM user → Switch Role
Enter:
Account ID: Developer account ID you just created
Role name: OrganizationAccountAccessRole
Switch role

✔ You are now in Dev account

## Step 5: Create Manual IAM Role (Dev Account)
I didn't have another existing account to invite. So to simulate the CrossAccount Access behaviour I used the same developer account by manually creating the role and adding general iamadmin account as the trusted entity
Inside Dev account:

Go to IAM → Roles → Create Role
Select Another AWS account
Enter management account ID
Attach policy:
AdministratorAccess
Role name:
CrossAccountAdminRole
Create role
## Step 6: Switch Role (Manual Role)

From management account:

Click IAM user → Switch Role
Enter:
Account ID: Developer account ID
Role name: CrossAccountAdminRole
Click Switch

✔ Access Dev account via manual role

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
