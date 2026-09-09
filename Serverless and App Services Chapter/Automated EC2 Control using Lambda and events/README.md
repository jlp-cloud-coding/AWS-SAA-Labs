# Automated EC2 Control using Lambda and Events - a simple Event driven architecture

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

This was an important implementation/debugging discovery in this project. With older console there was no explicit execution role needed for the EventBridge rule to automatically trigger the lambda function. The initial mistake was I was giving the EC2StartStopLambda role which had a LambdaStartandStop.json policy that was created for the lambda function explicitly in the EventBridge rule. It never worked because this role allows only Lambda function to assume it and allows EC2 Start/Stop + CloudWatch logs. So EventBridge rule was unable to invoke the ProtectEC2 Lambda function and I had to create a new execution role explictly for the EventBridge rule to invoke the lambda function.

# 5A. Scheduled EventBridge Rule — Automatic EC2 Stop

In addition to the **event-pattern EventBridge rule** used for self-healing, this project also includes a **scheduled EventBridge rule**.

The scheduled rule is configured in the AWS console with a specific **UTC time**. At that scheduled time, EventBridge automatically invokes the `Lambda_instance_stop.py` function.

> **Note:** This lab uses the scheduled-time configuration available in the AWS console. The schedule is based on **UTC**, rather than manually entering a cron expression.

### Scheduled automation flow

```text
Scheduled EventBridge Rule
          │
          │ At configured UTC time
          ▼
Lambda_instance_stop.py
          │
          │ Reads EC2_INSTANCES
          ▼
    EC2 Instances
          │
          │ Stop
          ▼
EC2 Instance → Stopped
          │
          │ EC2 state-change event
          ▼
EventBridge Event-Pattern Rule
          │
          │ Detects protected instance
          ▼
ProtectEC2 Lambda
          │
          │ start_instances()
          ▼
Protected EC2 Instance
          │
          ▼
       Running
```

### How the two EventBridge rules work together

There are **two different EventBridge rules** in this project:

| EventBridge Rule                | Trigger                                        | Purpose                                                            |
| ------------------------------- | ---------------------------------------------- | ------------------------------------------------------------------ |
| **Scheduled EventBridge Rule**  | Configured UTC time                            | Automatically invokes `Lambda_instance_stop.py`                    |
| **EC2 State-Change Event Rule** | EC2 `stopped` event matching the event pattern | Detects when the protected instance stops and invokes `ProtectEC2` |

The scheduled rule handles the **automatic stop**, while the event-pattern rule handles the **self-healing restart**.

### Complete end-to-end workflow

At the configured UTC time:

```text
1. Scheduled EventBridge rule triggers
              ↓
2. Lambda_instance_stop.py executes
              ↓
3. Lambda reads EC2_INSTANCES environment variable
              ↓
4. Lambda calls ec2.stop_instances()
              ↓
5. EC2 instances transition to Stopped
              ↓
6. EC2 generates an Instance State-change Notification
              ↓
7. Event-pattern EventBridge rule evaluates the event
              ↓
8. Protected instance matches:
       state = stopped
       AND
       instance-id = protected instance
              ↓
9. ProtectEC2 Lambda is invoked
              ↓
10. ProtectEC2 calls ec2.start_instances()
              ↓
11. Protected instance transitions:
       Stopped → Pending → Running
```

This demonstrates how a **time-based EventBridge trigger** and an **event-driven EventBridge trigger** can work together to automate and protect an EC2 instance.

### Key concept

The two rules have different responsibilities:

```text
                    EventBridge
                       │
          ┌────────────┴────────────┐
          │                         │
          ▼                         ▼
   Scheduled Rule             Event Pattern Rule
   (UTC time)                 (EC2 state change)
          │                         │
          ▼                         ▼
  Stop EC2 instances        Detect protected EC2
                                    │
                                    ▼
                              Start protected
                                    │
                                    ▼
                                  Running
```


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

1. Lambda environment variable configuration for `EC2_INSTANCES`. (below image is for StartEC2 lambda function and is same as in StopEC2)

<img width="957" height="370" alt="startec2_envvariables" src="https://github.com/user-attachments/assets/a04004d4-3412-4988-beee-2d8045698f79" />

2. Manual Lambda invocation and EC2 stop/start result.
 
<img width="941" height="375" alt="startEC2" src="https://github.com/user-attachments/assets/e40deceb-9704-4973-9dec-dba0587b9c43" />

<img width="955" height="374" alt="startec2_0" src="https://github.com/user-attachments/assets/54e8fd4a-c281-4358-996c-358ed99003fb" />

<img width="959" height="355" alt="startec2_2" src="https://github.com/user-attachments/assets/609551c2-0d06-4de8-b6db-12cde1aa9dd0" />

<img width="956" height="377" alt="startec2_3" src="https://github.com/user-attachments/assets/3a0a2234-4739-4d94-8afc-36aafa9a7fa3" />

<img width="957" height="370" alt="startec2_envvariables" src="https://github.com/user-attachments/assets/7c769523-51a0-4864-b374-af712a0f4260" />

<img width="956" height="355" alt="started_EC2" src="https://github.com/user-attachments/assets/4156b783-ee02-4c40-8e85-e7b65cf6d820" />

## StopEC2
Same as above screens create a different lambda function to manually invoke stopping of EC2 instances with a different python script for stopping the EC2 instances

<img width="956" height="353" alt="stopped_ec2" src="https://github.com/user-attachments/assets/8f0c4fe4-c3cf-45e8-a9b3-a4cb83162791" />
 
3. EventBridge rule configuration and Event pattern filtering for the protected instance

<img width="957" height="374" alt="EBSchedule" src="https://github.com/user-attachments/assets/b44d9155-c548-4165-abd7-d5991cb564d1" />

<img width="959" height="374" alt="EB1" src="https://github.com/user-attachments/assets/19af0f63-d7bc-45ca-9600-acd3ceb692db" />

<img width="956" height="383" alt="Screenshot 2026-09-06 170323" src="https://github.com/user-attachments/assets/6eb7393a-bae2-499b-aae6-670417e9033b" />

<img width="958" height="370" alt="Screenshot 2026-09-06 170424" src="https://github.com/user-attachments/assets/e2e06c03-3697-4a97-8a97-24bdcc6b962c" />

<img width="959" height="380" alt="Screenshot 2026-09-06 170446" src="https://github.com/user-attachments/assets/b861cdec-ba01-49b2-803f-739ddd353ba7" />

<img width="959" height="379" alt="Screenshot 2026-09-06 170524" src="https://github.com/user-attachments/assets/9f4ac5d5-9b84-47e2-8bbb-caf3047d58d1" />

<img width="956" height="374" alt="EBTarget1" src="https://github.com/user-attachments/assets/a59ed572-a0bd-4d1c-ab40-3430512c34c1" />

<img width="954" height="374" alt="EBTarget2" src="https://github.com/user-attachments/assets/8d01cfbf-b718-4515-8923-6d6f28c95aac" />

4. Lambda execution role

<img width="956" height="373" alt="iamRole1" src="https://github.com/user-attachments/assets/d15575d9-853a-4687-85c3-a7a572e75fa4" />

<img width="956" height="373" alt="iamRole2" src="https://github.com/user-attachments/assets/cfb1ca8f-9e22-46db-8d25-05dff7de43de" />

<img width="957" height="375" alt="iamRole3" src="https://github.com/user-attachments/assets/273a5594-bb7a-4143-9411-e875846cddd1" />

5. EventBridge execution role:

   No screenshot for this role is available so pasting in the json instead that was used by this role:

   ``` json
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

7. CloudWatch/Lambda logs showing the self-healing invocation.

<img width="959" height="371" alt="Validation2" src="https://github.com/user-attachments/assets/429dd0e1-257d-40dd-8223-ab876f43b8e1" />

<img width="959" height="349" alt="validation2a" src="https://github.com/user-attachments/assets/6d43601d-dbce-4757-92c8-b0674f844aba" />


