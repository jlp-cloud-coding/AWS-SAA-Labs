# CloudFormation Template - Non-portable templates and failure scenarios

A hands-on lab demonstrating why static, hardcoded values in **AWS CloudFormation** templates destroy template portability, leading to deployment failures across multiple stacks and regions.

---

## Architectural Concept

In CloudFormation, templates define **Logical Resources** (the "WHAT") which map to **Physical Resources** (the "HOW") when a stack is created. However, writing templates with hardcoded identifiers introduces critical architectural bottlenecks:
1. **Global Uniqueness Violations:** Resources like Amazon S3 buckets require globally unique names across all AWS accounts and regions. Hardcoding a static bucket name guarantees collision failures when deploying a stack a second time or if another account owns that name.
2. **Regional Binding Failures:** Resources like EC2 Amazon Machine Images (**AMIs**) are restricted to the specific region where they were created. Hardcoding an AMI ID from `us-east-1` and deploying the template in `us-west-2` (Oregon) causes immediate stack creation failures.

---

## Steps

### Scenario 1: Global S3 Name Collision (`nonportable1` & `nonportable2`)
* **Action:** Created a stack using a YAML template containing a hardcoded S3 bucket name (`accatpics13333337`).
* **Observation:** The initial stack creation failed because the S3 bucket name already existed globally (or within the account scope). Even after deleting the stack, attempting to recreate it resulted in name conflicts until the static identifier was altered.

### Scenario 2: Regional AMI Mismatch (`Oregon Region Deployment`)
* **Action:** Switched the AWS Management Console region from `us-east-1` to `us-west-2` (Oregon) and attempted to deploy the same template.
* **Observation:** The stack creation failed because the hardcoded `ImageId` (`ami-090fa75af13c156b4`) from `us-east-1` does not exist in the Oregon region.

---

## 🛠️ Sample Non-Portable Template Structure

```yaml
Resources:
  Bucket:
    Type: 'AWS::S3::Bucket'
    Properties:
      BucketName: 'accatpics13333337'  # Causes global naming collisions
  Instance:
    Type: 'AWS::EC2::Instance'
    Properties:
      InstanceType: "t3.micro"
      ImageId: 'ami-090fa75af13c156b4' # Causes regional deployment failures
```

**Official AWS Documentation & References**
AWS CloudFormation Resource Reference: [AWS::S3::Bucket Resource Type](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-s3-bucket.html)

**Key Takeaways & Best Practices**
Avoid Hardcoding: Never hardcode resource names or region-specific IDs in production CloudFormation templates.

Leverage Parameters & Mappings: Use CloudFormation Parameters for dynamic input values, Mappings for region-specific AMI selection, and let AWS auto-generate unique physical names using features like BucketName omission or Fn::Sub.
