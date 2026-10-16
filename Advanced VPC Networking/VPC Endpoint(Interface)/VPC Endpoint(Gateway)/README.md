# VPC Gateway Endpoint for Amazon S3

## Architecture Overview

In this lab, I created an **S3 Gateway Endpoint** in my private VPC (`a4l-secretvpc`) to allow my private EC2 instance to access Amazon S3 without using a NAT Gateway or routing S3 traffic through the public internet.

<img width="923" height="496" alt="Screenshot 2026-10-14 162445" src="https://github.com/user-attachments/assets/32c8a1c5-493f-47a5-a9e8-77b4f894e8dd" />


## 🛠️ Implementation Steps

### 1. Create the S3 Bucket

- Created an S3 bucket named `a4lsecretvpc-bucket-xtq6elrv28wy`.
- Uploaded a file named `supersecret.txt` containing the text `Cats are the best`.

### 2. Test Initial Connectivity

- Connected to the private EC2 instance `A4L-PrivateEC2` using the EC2 Instance Connect Endpoint from Lab 1.
- Ran `ping 1.1.1.1` to test internet connectivity. The request resulted in 100% packet loss.
- Ran `aws s3 ls`, but the command timed out because the instance could not reach S3 through its public service endpoint.

### 3. Create the S3 Gateway Endpoint

Navigated to **VPC Console → Endpoints → Create endpoint** and configured the following:

| Setting | Value |
|---|---|
| Name | `PrivateVPCS3` |
| Service category | AWS services |
| Service name | `com.amazonaws.us-east-1.s3` |
| Endpoint type | Gateway |
| VPC | `a4l-secretvpc` |
| Route table | `rtb-05b47a1de1cccb1c0` |

### 4. Verify the Route Table

- Checked the main VPC route table and confirmed that AWS automatically added a route for S3.
- The route uses the AWS-managed prefix list `pl-63a5400a` as the destination and the Gateway Endpoint `vpce-05fc4de15bcc3db5f` as the target.
- Checked the prefix list entries to view the IP ranges associated with S3 in `us-east-1`.

### 5. Test S3 Access and Download the File

- Ran `aws s3 ls` from the EC2 instance and confirmed that the bucket `a4lsecretvpc-bucket-xtq6elrv28wy` was listed.
- Downloaded `supersecret.txt` using the following command:

  ```bash
  aws s3 cp s3://a4lsecretvpc-bucket-xtq6elrv28wy/supersecret.txt supersecret.txt
  ```

- Ran `ls -la` to verify that `supersecret.txt` was present in the local working directory.
- Ran `cat supersecret.txt` and confirmed that the output was `Cats are the best`.

## Key Observation

Unlike Interface Endpoints, which use Elastic Network Interfaces (ENIs) with private IP addresses, Gateway Endpoints add routes to the VPC route table using an AWS-managed prefix list.

Traffic destined for S3 is routed through the Gateway Endpoint over the AWS network, without requiring a NAT Gateway or an internet gateway route for that traffic. **S3 Gateway Endpoints have no additional endpoint charges.**

## Screenshots

<img width="947" height="307" alt="Screenshot 2026-10-15 161852" src="https://github.com/user-attachments/assets/1c80c764-8610-4bd6-aa2e-9cccce06e094" />

<img width="950" height="353" alt="Screenshot 2026-10-15 162020" src="https://github.com/user-attachments/assets/82a28ebd-9d2a-45a0-8f46-5a4fe7659fa1" />

<img width="958" height="351" alt="Screenshot 2026-10-15 162244" src="https://github.com/user-attachments/assets/f078d0cc-e2c3-42f3-a785-914ead222635" />

<img width="957" height="355" alt="Screenshot 2026-10-15 162300" src="https://github.com/user-attachments/assets/3bae013b-2b31-4501-8269-b250c251e52f" />

<img width="956" height="353" alt="Screenshot 2026-10-15 162525" src="https://github.com/user-attachments/assets/b246e325-1886-4cdd-a120-71d8ca8bb522" />

<img width="959" height="360" alt="Screenshot 2026-10-15 162615" src="https://github.com/user-attachments/assets/c64a0a15-079b-4a87-ab8d-55fdb844dbe7" />

<img width="959" height="374" alt="Screenshot 2026-10-15 162754" src="https://github.com/user-attachments/assets/f1a4e929-aa36-4cb0-81e0-97283ec69bfe" />

<img width="932" height="349" alt="Screenshot 2026-10-15 162813" src="https://github.com/user-attachments/assets/790800bc-6d8c-4112-885e-e626e869fd4a" />

<img width="731" height="379" alt="Screenshot 2026-10-15 163239" src="https://github.com/user-attachments/assets/fd399cb8-0880-46a0-ad42-44e794cf32ad" />
