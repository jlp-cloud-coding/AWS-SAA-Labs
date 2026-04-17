## AWS CloudFormation: EC2 Instance Deployment
## Project Overview
This lab demonstrates the use of Infrastructure as Code (IaC) to provision a virtual server within the AWS Cloud. Using a YAML-based CloudFormation template, I automated the creation of an EC2 instance to ensure repeatable and consistent deployments.

## Architecture Components
Compute: Amazon EC2 (t2.micro - Free Tier eligible)
Infrastructure as Code: AWS CloudFormation (YAML)
Network: Deployed within the Default VPC
Security: Configured with a Security Group to allow specific inbound traffic.
## Key Takeaways
Stack Lifecycle: Practiced the full lifecycle of a stack (Create -> Execute -> Delete).
Declarative Configuration: Using YAML to define the "desired state" of infrastructure.
## Resource Cleanup: 
Ensuring all associated resources (EBS volumes, etc.) are terminated upon stack deletion to optimize costs.
## How to Use
Upload the ec2instance.yaml file to the AWS CloudFormation console.
Specify the required parameters (e.g., LatestAmiId, SSHandWebLocation).
Create the stack and monitor the events tab for resource creation.
