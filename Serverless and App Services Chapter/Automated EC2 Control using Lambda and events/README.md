# Self-Healing & Scheduled EC2 Automation using Amazon EventBridge and AWS Lambda

## 📌 Project Overview

This project demonstrates how to automate **Amazon EC2 instance management** using **AWS Lambda** and **Amazon EventBridge**.

The project has two main capabilities:

1. **Manual EC2 Start/Stop**

   * Invoke a Lambda function manually.
   * Pass EC2 instance IDs through an environment variable.
   * Start or stop one or multiple EC2 instances without hardcoding the instance IDs in the Lambda code.

2. **Self-Healing EC2 Protection**

   * Amazon EventBridge detects when a protected EC2 instance enters the `stopped` state.
   * EventBridge automatically invokes a Lambda function.
   * The Lambda function starts the protected EC2 instance again.

This demonstrates an event-driven AWS architecture where **EventBridge detects an EC2 state change and Lambda automatically responds to it**.

---

## 🏗️ Architecture

```text
                         ┌─────────────────────────────┐
                         │       EC2 Instances         │
                         │                             │
                         │  Instance A                 │
                         │  Instance B                 │
                         │  Protected Instance        │
                         └──────────────┬──────────────┘
                                        │
                                        │ EC2 state change
                                        │ stopped
                                        ▼
                         ┌─────────────────────────────┐
                         │       Amazon EventBridge    │
                         │                             │
                         │ Event Pattern:              │
                         │ EC2 state = stopped         │
                         │ Protected Instance ID       │
                         └──────────────┬──────────────┘
                                        │
                                        │ Invoke Lambda
                                        ▼
                         ┌─────────────────────────────┐
                         │   ProtectEC2 Lambda         │
                         │                             │
                         │ lambda_instance_protect.py │
                         │                             │
                         │ Starts protected instance   │
                         └─────────────────────────────┘


Manual EC2 Management
────────────────────────────────────────────────────────

      Manual Lambda Invocation
                │
                │ EC2_INSTANCES
                │ environment variable
                ▼
      ┌─────────────────────────┐
      │ EC2 Management Lambda   │
      │                         │
      │ Start / Stop instances  │
      └────────────┬────────────┘
                   │
                   ▼
              EC2 Instances
```

### Example self-healing flow

```text
EC2 Running
    │
    │ Manual stop
    ▼
EC2 Stopping
    │
    ▼
EC2 Stopped
    │
    │ EventBridge detects state change
    ▼
ProtectEC2 Lambda invoked
    │
    │ ec2.start_instances()
    ▼
EC2 Pending
    │
    ▼
EC2 Running
```

---

# 1. Manual EC2 Start/Stop using Lambda Environment Variables

The manual Lambda uses an environment variable called:

```text
EC2_INSTANCES
```

The variable contains one or more EC2 instance IDs separated by commas.

### Example

```text
EC2_INSTANCES=i-0123456789abcdef0,i-0987654321abcdef0
```

This makes the Lambda reusable without modifying the Python code whenever the target instances change.

---

## Manual Stop Lambda

### `Lambda_instance_stop.py`

```python
import boto3
import os
import json

region = 'us-east-1'
ec2 = boto3.client('ec2', region_name=region)

def lambda_handler(event, context):
    instances = os.environ['EC2_INSTANCES'].split(",")

    ec2.stop_instances(InstanceIds=instances)

    print('stopped instances: ' + str(instances))
```

### How it works

```text
EC2_INSTANCES
      │
      │ "i-123,i-456"
      ▼
os.environ['EC2_INSTANCES']
      │
      ▼
.split(",")
      │
      ▼
["i-123", "i-456"]
      │
      ▼
ec2.stop_instances()
```

### Why use an environment variable?

Instead of writing:

```python
ec2.stop_instances(
    InstanceIds=[
        "i-1234567890abcdef0",
        "i-0987654321abcdef0"
    ]
)
```

the instance IDs can be configured directly in the Lambda configuration.

This means the same Lambda can be reused for different EC2 instances.

---

## Manual Start Lambda

The same environment-variable approach can be used to manually start EC2 instances.

### `Lambda_instance_start.py`

```python
import boto3
import os
import json

region = 'us-east-1'
ec2 = boto3.client('ec2', region_name=region)

def lambda_handler(event, context):
    instances = os.environ['EC2_INSTANCES'].split(",")

    ec2.start_instances(InstanceIds=instances)

    print('started instances: ' + str(instances))
```

The same environment variable can therefore be used for both operations:

```text
EC2_INSTANCES=i-0123456789abcdef0,i-0987654321abcdef0
```

The **Start Lambda** calls:

```python
ec2.start_instances()
```

while the **Stop Lambda** calls:

```python
ec2.stop_instances()
```

---

# 2. Automated EC2 Self-Healing with EventBridge

The second part of the project protects a specific EC2 instance.

The protected instance used in this project is:

```text
i-07ffba5b3ebdab6c4
```

Amazon EventBridge monitors EC2 state-change events.

The rule is configured to match:

```text
source:
aws.ec2

detail-type:
EC2 Instance State-change Notification

detail:
state:
stopped

instance-id:
i-07ffba5b3ebdab6c4
```

When the protected instance enters the `stopped` state, EventBridge invokes the `ProtectEC2` Lambda.

---

# 3. Self-Healing Lambda

### `lambda_instance_protect.py`

```python
import boto3
import os
import json

region = 'us-east-1'
ec2 = boto3.client('ec2', region_name=region)

def lambda_handler(event, context):
    print("Received event: " + json.dumps(event))

    instances = [event['detail']['instance-id']]

    ec2.start_instances(InstanceIds=instances)

    print(
        'Protected instance stopped - starting up instance: '
        + str(instances)
    )
```

Unlike the manual Lambda, this function does **not** need an environment variable containing the instance ID.

The instance ID comes directly from the EventBridge event:

```python
event['detail']['instance-id']
```

### EventBridge event → Lambda

Conceptually:

```text
EventBridge Event
       │
       ▼
{
  "detail": {
    "instance-id": "i-07ffba5b3ebdab6c4",
    "state": "stopped"
  }
}
       │
       ▼
event['detail']['instance-id']
       │
       ▼
["i-07ffba5b3ebdab6c4"]
       │
       ▼
ec2.start_instances()
```

This allows the Lambda to react to the instance that generated the event.

---

# 4. IAM Roles and Permissions

There are **two separate IAM roles** involved in this architecture.

This distinction was also an important debugging lesson during implementation.

## Lambda Execution Role

The Lambda functions use the Lambda execution role:

### `policies/lambdarole.json`

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ec2:Start*",
        "ec2:Stop*"
      ],
      "Resource": "*"
    }
  ]
}
```

This role gives Lambda permission to:

* Write logs to Amazon CloudWatch Logs.
* Start EC2 instances.
* Stop EC2 instances.

The Lambda execution role trusts:

```text
lambda.amazonaws.com
```

---

## EventBridge Target Execution Role

EventBridge uses a **separate execution role** to invoke the `ProtectEC2` Lambda.

Example policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "lambda:InvokeFunction"
      ],
      "Resource": [
        "arn:aws:lambda:us-east-1:YOUR_ACCOUNT_ID:function:ProtectEC2"
      ]
    }
  ]
}
```

The EventBridge role trusts:

```text
events.amazonaws.com
```

### Important distinction

```text
Lambda
  │
  │ assumes
  ▼
Lambda Execution Role
  │
  └── EC2 Start/Stop + CloudWatch Logs


EventBridge
  │
  │ assumes
  ▼
EventBridge Execution Role
  │
  └── lambda:InvokeFunction
             │
             ▼
        ProtectEC2 Lambda
```

These roles have **different purposes and different trust relationships**.

---

# 5. EventBridge Configuration

The EventBridge rule listens for:

```text
EC2 Instance State-change Notification
```

and matches the protected instance when it becomes:

```text
stopped
```

### Rule concept

```json
{
  "source": [
    "aws.ec2"
  ],
  "detail-type": [
    "EC2 Instance State-change Notification"
  ],
  "detail": {
    "state": [
      "stopped"
    ],
    "instance-id": [
      "i-0123456789abcde"
    ]
  }
}
```

### Target

```text
EventBridge Rule
       │
       ▼
ProtectEC2 Lambda
```

The EventBridge target is configured with the **EventBridge execution role** that has permission to invoke the Lambda.

---

# 6. Debugging & Key Lessons

## 🔐 1. EventBridge requires its own execution role

One of the main debugging challenges was that EventBridge initially could not invoke the `ProtectEC2` Lambda.

The important distinction was:

> The Lambda execution role and EventBridge target execution role are not interchangeable.

The Lambda role trusts:

```text
lambda.amazonaws.com
```

while the EventBridge role must trust:

```text
events.amazonaws.com
```

The EventBridge role also needs:

```text
lambda:InvokeFunction
```

permission for the target Lambda.

---

## 🧩 2. EventBridge can be configured using an event pattern

The rule can match specific EC2 events rather than reacting to every event.

In this project, the important filters are:

```text
EC2 state = stopped
+
specific protected instance ID
```

This prevents the self-healing Lambda from restarting unrelated EC2 instances.

---

## 🔄 3. The EC2 state transition is asynchronous

When the instance is stopped and then automatically restarted, the observed sequence is:

```text
Stopping
   ↓
Stopped
   ↓
Pending
   ↓
Running
```

The important trigger is the `stopped` event.

EventBridge detects that event and invokes the protection Lambda.

---

## 🧪 4. Manual testing validates the entire workflow

A useful test was:

```text
Manually stop EC2 instances
        ↓
Protected instance reaches Stopped
        ↓
EventBridge detects event
        ↓
ProtectEC2 Lambda executes
        ↓
Lambda calls start_instances()
        ↓
Protected EC2 returns to Running
```

This validates the complete event-driven workflow rather than testing each AWS service independently.

---

## 🛠️ 5. EventBridge target configuration can be easy to overlook

In newer AWS accounts/configuration experiences, the EventBridge target setup includes an execution role for invoking the Lambda.

This was an important implementation/debugging discovery in this project.

---

# 7. Project Structure

```text
.
├── README.md
│
├── policies/
│   └── lambdarole.json
│
└── lambda/
    ├── Lambda_instance_stop.py
    ├── Lambda_instance_start.py
    └── lambda_instance_protect.py
```

### Lambda responsibilities

| Lambda                       | Invocation  | Purpose                                          |
| ---------------------------- | ----------- | ------------------------------------------------ |
| `Lambda_instance_stop.py`    | Manual      | Stop EC2 instances from `EC2_INSTANCES`          |
| `Lambda_instance_start.py`   | Manual      | Start EC2 instances from `EC2_INSTANCES`         |
| `lambda_instance_protect.py` | EventBridge | Automatically restart the protected EC2 instance |

---

# 🎯 Key Takeaways

* **AWS Lambda** can automate EC2 Start/Stop operations.
* **Lambda environment variables** make manual EC2 management reusable without hardcoding instance IDs.
* **Amazon EventBridge** can react to EC2 state-change events.
* EventBridge can trigger a Lambda automatically when a protected EC2 instance stops.
* The self-healing Lambda obtains the instance ID directly from the EventBridge event.
* **Lambda execution roles and EventBridge execution roles are separate IAM roles.**
* Lambda trusts `lambda.amazonaws.com`.
* EventBridge trusts `events.amazonaws.com`.
* EventBridge needs `lambda:InvokeFunction` permission to invoke the Lambda target.
* The resulting architecture demonstrates a practical **event-driven and self-healing AWS automation pattern**.
* The project also demonstrates both **manual automation** and **event-driven automation** using AWS managed services.

---

## 📸 Screenshots

Add screenshots here showing:

1. Lambda environment variable configuration for `EC2_INSTANCES`.
2. Manual Lambda invocation and EC2 stop/start result.
3. EventBridge rule configuration.
4. Event pattern filtering for the protected instance.
5. EventBridge target configuration.
6. EventBridge execution role.
7. Lambda execution role.
8. CloudWatch/Lambda logs showing the self-healing invocation.
9. EC2 state transition from `Stopping → Stopped → Pending → Running`.

These screenshots provide visual evidence of the configuration and end-to-end workflow.
