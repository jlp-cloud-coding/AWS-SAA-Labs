## Architecture Overview

In this lab, I created an **EC2 Instance Connect Endpoint** in the private VPC `a4l-secretvpc`. This allows me to connect to a private EC2 instance using SSH without assigning it a public IPv4 address or setting up a bastion host.

<img width="937" height="497" alt="Screenshot 2026-10-13 180647" src="https://github.com/user-attachments/assets/62ecd0fb-b82a-4fbd-8bb4-4777c499a0b6" />

## 🛠️ Implementation Steps

### 1. Identify the Private EC2 Instance

- Located the EC2 instance `A4L-PrivateEC2` (`i-06074bb29b010b429`) in the private subnet in `us-east-1a`.
- Verified that the instance has no public IPv4 address. It uses a private IPv4 address (`10.16.37.196`) and also has an IPv6 address.

### 2. Create the EC2 Instance Connect Endpoint

Navigated to **VPC Console → Endpoints → Create endpoint** and configured the following:

| Setting | Value |
|---|---|
| Name | `PrivateVPCEC2InstanceConnect` |
| Service category | EC2 Instance Connect Endpoint |
| VPC | `a4l-secretvpc` (`vpc-0963f942d141ae08d`) |
| Security group | `sg-0c0e95f0ef81cfb4f` |
| Subnet | `sn-app-A` (`subnet-0778141dc6138934a`) |
| IP address type | Dualstack |

### 3. Verify the Endpoint

- Confirmed that the endpoint status changed to **Available**.
- Endpoint ID: `eice-0570bc56d58057eba`.

## ✅ Verification and Testing

- Opened the EC2 Console and initiated a browser-based connection to `A4L-PrivateEC2` using the new endpoint.
- Successfully authenticated and established an interactive SSH session with the private Amazon Linux 2023 instance.
- Key Observation: When checking the SSH login logs on the private instance, the incoming connection IP matched the IP address of the endpoint's Elastic Network Interface (ENI) rather than my local public IP. This happens because the Preserve Client IP option was left unchecked during endpoint creation. The terminal displayed the session as `ec2-user@ip-10-16-36-216`, confirming that the connection was established.
   - Note on Preserve Client IP
    ---------------------------:
     If this box was checked, AWS would forward my local computer's public IP address (which it gets from my web    browser session when I log into the AWS console) straight through to the instance, making my actual home/office IP show up in the SSH logs instead of the ENI IP.

<img width="956" height="389" alt="Screenshot 2026-10-15 161544" src="https://github.com/user-attachments/assets/cfe7f933-086e-4e00-9382-46e4f59c3c00" />

<img width="959" height="371" alt="Screenshot 2026-10-15 161623" src="https://github.com/user-attachments/assets/c82859d3-686c-4635-a2b6-580f702f8cdc" />

- ## Screenshots
  
<img width="957" height="350" alt="Screenshot 2026-10-14 165158" src="https://github.com/user-attachments/assets/c94f6854-f179-4883-94ea-c7b75789c77c" />

<img width="956" height="360" alt="Screenshot 2026-10-14 165419" src="https://github.com/user-attachments/assets/8291fdee-daf1-45ea-9c27-4c5ac4534dae" />

<img width="959" height="356" alt="Screenshot 2026-10-14 165512" src="https://github.com/user-attachments/assets/80d3e633-e62c-4bbe-b93b-562fff72878d" />

<img width="958" height="357" alt="Screenshot 2026-10-14 165537" src="https://github.com/user-attachments/assets/5012521e-c6e6-4bc4-b166-574affb15a7a" />

<img width="956" height="355" alt="Screenshot 2026-10-14 165757" src="https://github.com/user-attachments/assets/fc175d05-add6-4c8b-bd4a-10c0210381db" />

<img width="959" height="359" alt="Screenshot 2026-10-14 170501" src="https://github.com/user-attachments/assets/b9959421-8196-4252-ad7c-d0ee1c09cace" />

<img width="959" height="362" alt="Screenshot 2026-10-14 170537" src="https://github.com/user-attachments/assets/23e44cdf-6eb8-4678-aa9c-5f1e4f89072d" />
