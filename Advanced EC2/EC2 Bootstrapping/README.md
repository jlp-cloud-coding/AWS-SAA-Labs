# AWS EC2 Bootstrapping Evolution

Automating application deployment on Amazon EC2 using progressively better bootstrapping techniques.

This project demonstrates how EC2 initialization evolves from a simple User Data script into a production-grade CloudFormation deployment using **cfn-init** and **cfn-signal**.

The demo application installs:

- WordPress
- Apache
- MariaDB
- PHP
- Custom `cowsay` MOTD banner

---

# Architecture Evolution

```
Approach 1
Manual EC2 Launch
        │
        ▼
Plain User Data Script
        │
        ▼
Approach 2
CloudFormation
        │
        ▼
Same User Data Script
        │
        ▼
Approach 3
CloudFormation + Metadata + cfn-init + cfn-signal
```

---

# Approach 1 – EC2 Console + Plain User Data

## Overview

Launch an EC2 instance manually through the AWS Console and paste the bootstrap script into the **User Data** field.

During the first boot, **cloud-init** executes the script as the `root` user.

## UserData
View the UserData script below:
[ViewScript](userdata.txt)


## Advantages

- Very easy to learn
- Great for testing
- Quick proof of concept

## Limitations

- Manual deployment
- Not reusable
- Difficult to maintain
- Doesn't scale

---

# Approach 2 – CloudFormation + Plain User Data

## Overview

Instead of pasting the script manually, the exact same script is embedded inside an AWS CloudFormation template.

CloudFormation provisions the EC2 instance and passes the script to **cloud-init** automatically.

```yaml
UserData:
  Fn::Base64: !Sub |
    #!/bin/bash -xe

    dnf install ...
```
View entire CFN template file below:
[CFNTemplate](cloudformation_with_UserData.txt)

## Advantages

- Infrastructure becomes reproducible
- Fully automated deployments
- Version controlled
- Same template can deploy multiple environments

## Limitation

CloudFormation **does not wait** for the User Data script to finish.

If the EC2 instance launches successfully but the installation fails five minutes later:

- Stack still becomes **CREATE_COMPLETE**
- Deployment appears successful
- Failure is hidden

This is known as a **fire-and-forget** deployment.

---

# Approach 3 – CloudFormation + cfn-init + cfn-signal

## Overview

Instead of embedding a huge bash script inside User Data, the configuration is moved into the CloudFormation **Metadata** section.

The User Data becomes only a small bootstrap wrapper.

```yaml
UserData:
  Fn::Base64: !Sub |
    #!/bin/bash -xe

    /opt/aws/bin/cfn-init ...

    /opt/aws/bin/cfn-signal ...
```
View entire CFN template file below:
[CFNTemplate_cfn-init]cloudformation_with_cfn-init.txt

The actual installation steps now live inside the template Metadata as structured configuration.

## What cfn-init Does

`cfn-init` reads the Metadata section and performs tasks such as:

- Installing packages
- Creating files
- Running commands
- Managing services
- Applying permissions

Instead of a long sequential bash script, the deployment becomes declarative and easier to maintain.

## What cfn-signal Does

`cfn-signal` reports the final status of the initialization back to CloudFormation.

Combined with a **CreationPolicy**, CloudFormation waits until initialization finishes.

### Success

```
CREATE_IN_PROGRESS

↓

EC2 bootstraps successfully

↓

cfn-signal

↓

CREATE_COMPLETE
```

### Failure

```
CREATE_IN_PROGRESS

↓

Bootstrap fails

↓

No success signal received

↓

Stack rollback

↓

DELETE_COMPLETE
```

This prevents partially configured infrastructure from remaining deployed.

---

# Code Comparison

## Approach 1 & 2

Large imperative User Data script

```yaml
UserData:
  Fn::Base64: !Sub |
    #!/bin/bash -xe

    dnf install ...

    systemctl enable httpd

    systemctl start httpd

    ...

    mysql ...

    wget wordpress ...
```

---

## Approach 3

Small bootstrap wrapper

```yaml
UserData:
  Fn::Base64: !Sub |
    #!/bin/bash -xe

    /opt/aws/bin/cfn-init \
      --stack ${AWS::StackId} \
      --resource EC2Instance \
      --configsets wordpress_install \
      --region ${AWS::Region}

    /opt/aws/bin/cfn-signal \
      -e $? \
      --stack ${AWS::StackId} \
      --resource EC2Instance \
      --region ${AWS::Region}
```

The deployment logic is now stored in CloudFormation Metadata rather than inside a large bash script.

---

# Comparison

| Feature | Approach 1 | Approach 2 | Approach 3 |
|----------|------------|------------|------------|
| Launch Method | EC2 Console | CloudFormation | CloudFormation |
| Infrastructure as Code | ❌ | ✅ | ✅ |
| User Data | Plain Bash | Plain Bash | cfn-init Wrapper |
| Configuration Stored In | User Data | User Data | Metadata |
| Reusable | ❌ | ✅ | ✅ |
| Scalable | ❌ | ✅ | ✅ |
| CloudFormation Waits for Bootstrapping | N/A | ❌ | ✅ |
| Automatic Rollback | ❌ | ❌ | ✅ |
| Production Ready | ❌ | ⚠️ | ✅ |

---

# Inspecting User Data with IMDSv2

After the EC2 instance launches, the User Data script can be retrieved from inside the instance using the **EC2 Instance Metadata Service Version 2 (IMDSv2)**.

This demonstrates that the bootstrap script is delivered to the instance during launch and is available through the metadata service.

## Step 1 — Request an IMDSv2 Session Token

```bash
TOKEN=$(curl -X PUT \
"http://169.254.169.254/latest/api/token" \
-H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
```

The token is valid for **6 hours (21,600 seconds)** and must be included in subsequent metadata requests.

---

## Step 2 — View Available Metadata Categories

```bash
curl \
-H "X-aws-ec2-metadata-token: $TOKEN" \
http://169.254.169.254/latest/meta-data/
```

Example output:

```
ami-id
instance-id
instance-type
hostname
local-ipv4
public-ipv4
security-groups
...
```

These values describe the running EC2 instance.

---

## Step 3 — View the User Data

```bash
curl \
-H "X-aws-ec2-metadata-token: $TOKEN" \
http://169.254.169.254/latest/user-data/
```

This command returns the exact User Data that was supplied when the EC2 instance was launched.

For:

- **Approach 1**, it returns the script entered manually in the EC2 Console.
- **Approach 2**, it returns the script embedded inside the CloudFormation template.
- **Approach 3**, it returns only the lightweight wrapper that invokes `cfn-init` and `cfn-signal`.

---

## Why IMDSv2?

AWS introduced **Instance Metadata Service Version 2 (IMDSv2)** to improve security over IMDSv1.

Benefits include:

- Session-oriented authentication using temporary tokens
- Protection against Server-Side Request Forgery (SSRF) attacks
- Better security for applications running on EC2 instances

## Flow:
CloudFormation / EC2 Console
            │
            ▼
      User Data
            │
            ▼
       cloud-init
            │
            ▼
   EC2 Instance Boots
            │
            ▼
Stored by Instance Metadata Service
            │
            ▼
curl http://169.254.169.254/latest/user-data/

# Screenshots

## Approach 1

### EC2 Instance Connect

Shows successful execution of the User Data script and the custom **cowsay** MOTD along with the cloud_init_output.log and cloud_init.log files

<img width="959" height="355" alt="Screenshot 2026-07-11 160814" src="https://github.com/user-attachments/assets/c7fe8ebf-dca8-45ec-95fc-45712c9057cb" />

<img width="958" height="373" alt="Screenshot 2026-07-11 160843" src="https://github.com/user-attachments/assets/10532d66-165f-4c0d-948f-d80cc134c336" />

<img width="959" height="377" alt="Screenshot 2026-07-11 160936" src="https://github.com/user-attachments/assets/ec214208-31a7-4d1f-b3a7-78fdb5a89bad" />

<img width="953" height="355" alt="Screenshot 2026-07-11 161525" src="https://github.com/user-attachments/assets/cead1106-a575-488e-a5a6-7af6ab796c23" />

<img width="959" height="386" alt="Screenshot 2026-07-11 162304" src="https://github.com/user-attachments/assets/74f19c38-79c9-462b-9b9f-75175eb45a1f" />

---

### WordPress Verification

Accessing the EC2 Public IPv4 address confirms:

- Apache running
- PHP configured
- MariaDB initialized
- WordPress installation page displayed
  
<img width="950" height="474" alt="Screenshot 2026-07-11 162111" src="https://github.com/user-attachments/assets/4b4397a7-77f3-4d9f-99b8-22a6720774dc" />

---

# Key Takeaways

This project demonstrates the evolution of EC2 bootstrapping:

1. **Approach 1** — Manual bootstrapping using EC2 User Data.
2. **Approach 2** — Automated infrastructure deployment using CloudFormation with plain User Data.
3. **Approach 3** — Production-grade orchestration using CloudFormation Metadata, `cfn-init`, `cfn-signal`, and `CreationPolicy`.

The final approach provides better maintainability, deployment visibility, and automatic rollback, making it the preferred pattern for production environments.
