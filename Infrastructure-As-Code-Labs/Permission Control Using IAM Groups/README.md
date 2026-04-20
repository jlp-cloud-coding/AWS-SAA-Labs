
## Overview
This lab is an extension of previous identity management exercise. The goal is to refactor the security model by removing inline/managed policies from a specific user and transitioning to a Group-based permission model.

## Objectives
Using a ClodFormation template:

Create a specialized IAM User (Sally).

Create an IAM Group with specific custom managed Policies.

Demonstrate that a User inherits permissions through Group membership.

Verify access to AWS S3 resources using the Group-assigned permissions.

## Architecture
Below resources are created using the CloudFormation template

Identity: IAM User Sally

Bucket: cat pics bucket, animal pics bucket

Policy: AllowAllS3ExceptCats (custom managed policy) attached to the Group.

## Implementation Steps
1. User Creation
Created an IAM User named Sally using CFN template. At this stage, Sally has Zero Permissions (the "Implicit Deny" state).

2. Group Configuration
Created an IAM User Group using CFN template. Instead of attaching a policy directly to Sally's user object, the policy was attached to the Group.

Group Name: developers

Attached Policy: AllowAllS3ExceptCats

3. Membership Assignment
Added Sally to the developerss group.

4. Verification
Action: Logged in as Sally.

Result: Sally was able to list and delete the objects in animal pics S3 bucket but not the cat pics bucket.

Logic: Sally's identity inherited the permissions from the Group container.

🧠 Key Technical Learnings
Groups are NOT Identities: You cannot log in as a Group. A Group is purely a management container for Users.

Inheritance: When a user is added to a group, they immediately inherit all policies attached to that group. If they are removed, those permissions are instantly revoked.

Scalability: If we need to add 50 more S3 administrators, we simply add them to this Group rather than attaching 50 individual policies.

Policy Types: We used an AWS Managed Policy, which is a standalone policy managed by AWS that provides permissions for common use cases.
