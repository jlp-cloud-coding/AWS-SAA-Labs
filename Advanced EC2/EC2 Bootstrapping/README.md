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
[CFNTemplate_UserData](cloudformation_with_UserData.txt)

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
[CFNTemplate_cfn-init][cloudformation_with_cfn-init.txt]

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

# Screenshots

## Approach 1

### EC2 Instance Connect

Shows successful execution of the User Data script and the custom **cowsay** MOTD.

*Insert Screenshot*

---

### WordPress Verification

Accessing the EC2 Public IPv4 address confirms:

- Apache running
- PHP configured
- MariaDB initialized
- WordPress installation page displayed

*Insert Screenshot*

---

# Key Takeaways

This project demonstrates the evolution of EC2 bootstrapping:

1. **Approach 1** — Manual bootstrapping using EC2 User Data.
2. **Approach 2** — Automated infrastructure deployment using CloudFormation with plain User Data.
3. **Approach 3** — Production-grade orchestration using CloudFormation Metadata, `cfn-init`, `cfn-signal`, and `CreationPolicy`.

The final approach provides better maintainability, deployment visibility, and automatic rollback, making it the preferred pattern for production environments.
