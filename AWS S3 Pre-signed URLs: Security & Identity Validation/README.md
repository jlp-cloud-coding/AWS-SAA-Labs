# Overview

This lab demonstrates how Amazon S3 Presigned URLs work, including:

- Temporary object access
- URL expiration behavior
- IAM permission dependency
- Explicit deny impact
- Behavior with non-existent objects

---

## Architecture Concept

A presigned URL allows temporary access to a private S3 object using the permissions of the IAM identity that generated the URL.

Important concepts:

- Bucket can remain private
- URL inherits creator permissions
- URL stops working if permissions are removed
- URL expiration does NOT override IAM credential validity

---

## Services Used

- Amazon S3
- IAM
- AWS CLI / AWS CloudShell

---

# Lab Steps

## 1. Create an S3 Bucket

```
aws s3 mb s3://demofruitsbucket100
```

<img width="927" height="349" alt="s3Bucket" src="https://github.com/user-attachments/assets/669d6248-1f59-4451-b6c8-3921d14e6e5b" />


2. Upload an Object
```
aws s3 cp image.jpg s3://demofruitsbucket100/
```
Verify that the image object was uploaded successfully using the console Open.

<img width="934" height="358" alt="step1 Open Authenticated" src="https://github.com/user-attachments/assets/ac5e909e-f8ad-4663-92cf-0c2e54235cf4" />

Also verify that Object URL doe not authenticate user to view the image since bucket is not made public

<img width="575" height="212" alt="step2 Object URL Unauthenticated" src="https://github.com/user-attachments/assets/99a7686e-9816-40ea-aa86-fc6af8ee66d3" />

3. Generate a Presigned URL (180 seconds)
```
aws s3 presign s3://demofruitsbucket100/image.jpg --expires-in 180
```

<img width="917" height="396" alt="Test 1 Presign URL Generate" src="https://github.com/user-attachments/assets/c0ccf6b5-5202-486b-ac56-e1c3cf5d4e4b" />


## Observation:

Successfully opened the private image using browser
Bucket itself remained private
Access worked temporarily

4. Generate Another Presigned URL (with expiry time set to longer duration in my case 1 hour)
```
aws s3 presign s3://demofruitsbucket100/image.jpg --expires-in 3600
```

<img width="940" height="272" alt="test 5 unknown object presigned url cloudshell" src="https://github.com/user-attachments/assets/54a4c67b-f772-432e-9139-e4ff8dc5a4c4" />


## Observation:

URL worked successfully.
Access remained valid while IAM permissions were valid.

5. Add Explicit Deny Policy to IAM User

Added inline deny policy named denyS3 to the IAM user:

Example policy:
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyS3",
      "Effect": "Deny",
      "Action": "s3:*",
      "Resource": "*"
    }
  ]
}
```

6. Test Existing Presigned URL Again

## Observation:

Previously generated presigned URL stopped working immediately.
Access Denied error occurred.

<img width="957" height="182" alt="test5 unknown object presigned url" src="https://github.com/user-attachments/assets/0a7aaf2b-cd46-4a39-8b61-4c2587d1c522" />



## Important concept:

Presigned URLs depend on the CURRENT permissions of the identity that created them.

Even though, URL expiration time had NOT expired, the URL still failed because:

Explicit Deny overrides Allow permissions

7. Generate Presigned URL Despite Deny Policy on iamuser
```
aws s3 presign s3://demofruitsbucket100/image.jpg --expires-in 3600
```

## Observation:

URL generation still succeeded.
Opening the URL failed with Access Denied.

## Important concept:

AWS CLI does NOT validate object access while generating the URL.
Actual authorization happens only when the URL is used.

8. Generate Presigned URL for Non-Existent Object
```
aws s3 presign s3://demofruitsbucket100/doesnotexist.jpg --expires-in 3600
```

## Observation:

URL was generated successfully.
Accessing the URL failed because the object did not exist.

## Important concept:

AWS does not verify object existence during presigned URL generation.

## Key Learnings
1. Presigned URLs
2. Provide temporary access to private S3 objects
3. Use permissions of creator identity
4. Support downloads and uploads
5. Permission Dependency
6. Presigned URLs work only if:
    > creator credentials remain valid
    > creator permissions still allow access

7. Explicit Deny always overrides:
    > IAM Allows
    > Bucket Policies
    > Presigned URL access
    > Expiration Behavior

8. Effective lifetime of a presigned URL is:
    > MIN(URL expiry time, credential validity)

## Examples and Scenario	Result:

URL valid 1 hour, credentials valid 12 hours	URL works 1 hour
URL valid 7 days, role expires in 6 hours	URL works 6 hours
Explicit deny added after URL creation	URL stops immediately

## Cleanup

1. Delete bucket and objects:
```
  aws s3 rb s3://my-presign-demo-bucket --force
```

2. Remove inline deny policy from IAM user after testing.
