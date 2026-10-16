## Architecture Overview

In this lab, I created an **Egress-Only Internet Gateway (EIGW)** for my VPC (`a4l-secretvpc`) to enable outbound IPv6 internet access from my private EC2 instance while preventing the internet from initiating inbound connections.

<img width="947" height="497" alt="Screenshot 2026-10-14 163521" src="https://github.com/user-attachments/assets/4dde30ad-a82c-4f44-ba4a-371e0be09866" />

## 🛠️ Implementation Steps

### 1. Tested Initial IPv6 Connectivity

- Connected to my private EC2 instance (`A4L-PrivateEC2`) using the EC2 Instance Connect Endpoint.
- Ran the following command to test IPv6 connectivity:

```bash
ping -6 ipv6.google.com
```

- The ping failed with 100% packet loss because there was no route for outbound IPv6 traffic.

### 2. Created the Egress-Only Internet Gateway

- Navigated to **VPC → Egress-Only Internet Gateways → Create Egress-Only Internet Gateway**.
- Created the gateway with the following configuration:

| Setting | Value |
|---|---|
| Name | `A4LIPV6` |
| VPC | `a4l-secretVPC` |
| VPC ID | `vpc-0ccd2f1a442017280` |
| Egress-Only Internet Gateway ID | `eigw-0cb9fa69c9b90f436` |

### 3. Updated the Route Table

- Opened the route table `rtb-05b47a1de1cccb1c0`.
- Added the following route:

| Setting | Value |
|---|---|
| Destination | `::/0` |
| Target | `eigw-0cb9fa69c9b90f436` |

- Saved the changes.

### 4. Verified IPv6 Connectivity

- Ran the ping command again from the EC2 instance:

```bash
ping -6 ipv6.google.com
```

- Received successful ping responses with 0% packet loss, confirming outbound IPv6 connectivity.

## ✅ Verification & Testing

- **Before configuration:** IPv6 ping failed because no outbound IPv6 route was configured.
- **After configuration:** IPv6 ping succeeded after adding the `::/0` route to the Egress-Only Internet Gateway.
  
## Key Learnings & Troubleshooting / FailuresCrucial Missing Step: 

- Simply creating an Egress-Only Internet Gateway and attaching it to the VPC will not enable internet connectivity by itself.
- Failure Point: If you forget to update the subnet's route table, traffic will still fail (100% packet loss) because the VPC router doesn't know where to send IPv6 traffic destined for ::/0.
- The Fix: You must explicitly configure the route table by adding ::/0 pointed to the newly created eigw-xxxx object as the target. 

## Key Observation

An Egress-Only Internet Gateway supports **outbound IPv6 traffic only**. It allows instances to initiate connections to the internet and receive responses to those connections, but prevents the internet from initiating connections to the instances. Unlike a NAT Gateway, it does not perform IP address translation.

## Screenshots:

<img width="954" height="377" alt="Screenshot 2026-10-15 163512" src="https://github.com/user-attachments/assets/93d5e7e8-9a4d-424b-b9d1-fa43a3ce491c" />

<img width="959" height="371" alt="Screenshot 2026-10-15 163616" src="https://github.com/user-attachments/assets/670a320a-c0db-4dc8-8b54-58dfcc373186" />

<img width="953" height="372" alt="Screenshot 2026-10-15 163700" src="https://github.com/user-attachments/assets/8ccabccb-d713-4155-a552-ed99336a7e84" />

<img width="959" height="355" alt="Screenshot 2026-10-15 163729" src="https://github.com/user-attachments/assets/d494e058-fbfe-4090-ab5a-d94bf1558499" />

