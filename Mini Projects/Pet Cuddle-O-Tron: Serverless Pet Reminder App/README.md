# 🐾 Pet Cuddle-O-Tron: Serverless Pet Reminder App

A simple web application built on AWS that lets users schedule email reminders for pet cuddles. Instead of running a server 24/7, this app uses **AWS serverless services** to run code only when a user requests a reminder.

I built this project to practice hands-on AWS architecture for the **AWS Certified Solutions Architect – Associate (SAA-C03)** exam.

---

## 🏗️ How It Works

Here is how data flows through the app from the browser to your inbox:

```text
[ S3 Static Website ] ──(POST Request)──► [ API Gateway ]
                                               │
                                               ▼
[ Amazon SES ] ◄── [ Step Functions ] ◄── [ API Lambda ]
  (Sends Email)       (Wait Timer)      (Starts Workflow)
```

### Request Flow

1. **Frontend**: A static HTML/JS web page hosted on an **S3 bucket**.

2. **API Endpoint**: **API Gateway** accepts the form data (email, message, wait time) from the web page.

3. **Backend Trigger**: A Python **Lambda function (`api_lambda`)** takes that data and starts a **Step Functions** workflow.

4. **Timer**: **Step Functions** waits for the specified delay time (e.g., 60 seconds).

5. **Email Sender**: Once the timer runs out, Step Functions triggers a second **Lambda function (`email_reminder_lambda`)**, which uses **Amazon SES** to send the email.

---

## 🛠️ AWS Services Used

* **S3**: Web hosting for the HTML, CSS, and JavaScript frontend.
* **API Gateway**: REST API endpoint configured with CORS to accept browser requests.
* **AWS Lambda**: Runs Python scripts without managing servers.
* **AWS Step Functions**: Manages the wait timer and workflow logic.
* **Amazon SES**: Handles email verification and sending.
* **AWS IAM**: Grants least-privilege permissions between services.

---

## 💡 What I Learned & Issues I Fixed

Building this helped me understand how serverless services connect and how to debug real-world integration errors.

Here are the exact errors encountered during implementation and how they were resolved.

### 1. CORS Policy Blocking Request — `TypeError: Failed to fetch`

#### ❌ Error Snippet

```text
Access to fetch at 'https://<api-id>.execute-api.us-east-1.amazonaws.com/prod/petcuddleotron'
from origin 'http://demobucket-petcuddleotron.s3-website-us-east-1.amazonaws.com'
has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.

serverless.js:26 POST https://<api-id>.execute-api.us-east-1.amazonaws.com/prod/petcuddleotron net::ERR_FAILED 200 (OK)
serverless.js:46 TypeError: Failed to fetch
    at sendData (serverless.js:26:5)
    at HTMLButtonElement.<anonymous> (serverless.js:21:80)
```

**Root Cause:**
CORS was previously enabled on the root path `/` instead of the explicit child resource `/petcuddleotron`. Therefore, API Gateway did not return the required `Access-Control-Allow-Origin` header for incoming browser POST requests.

**Resolution:**

1. Highlighted `/petcuddleotron` specifically in the API Gateway resources tree.
2. Clicked **Enable CORS** for both `POST` and `OPTIONS` methods.
3. Saved the configuration.
4. Redeployed the API to the `prod` stage.

<img width="957" height="373" alt="Screenshot 2026-09-29 225342" src="https://github.com/user-attachments/assets/86497c13-ebf5-4b14-a5c3-6d13b2c3d67e" />

<img width="948" height="371" alt="Screenshot 2026-09-29 225405" src="https://github.com/user-attachments/assets/da446ab1-b74d-450e-a842-c032a04b315a" />

<img width="959" height="414" alt="Screenshot 2026-09-29 225549" src="https://github.com/user-attachments/assets/fb6d27c8-8e6b-4fcc-a44f-ea327d63a4f0" />

---

### 2. Python String Syntax Error in Lambda — `UserCodeSyntaxError`

#### ❌ Error Snippet

```json
{
  "errorMessage": "Syntax error in module 'lambda_function': invalid syntax (lambda_function.py, line 36)",
  "errorType": "Runtime.UserCodeSyntaxError",
  "requestId": "",
  "stackTrace": [
    " File \"/var/task/lambda_function.py\" Line 36\n sm.start_execution( stateMachineArn=arn:aws:states:us-east-1:<account-id>:stateMachine:PetCuddleOTron, input=json.dumps(data, cls=DecimalEncoder) )\n"
  ]
}
```

**Root Cause:**
The Step Functions State Machine ARN parameter was pasted as unquoted raw text rather than a valid Python string inside `lambda_function.py`.

**Resolution:**
Wrapped the ARN value inside single quotes:

```python
stateMachineArn='arn:aws:states:us-east-1:<account-id>:stateMachine:PetCuddleOTron'
```

---

### 3. Missing Payload Key Error — `KeyError: 'body'`

#### ❌ Error Snippet

```json
{
  "errorMessage": "'body'",
  "errorType": "KeyError",
  "requestId": "78725593-9459-47be-8206-5d99e59cc0d8",
  "stackTrace": [
    " File \"/var/task/lambda_function.py\", line 17, in lambda_handler\n data = json.loads(event['body'])\n"
  ]
}
```

**Root Cause:**
The lab explicitly used standard **Lambda Integration (non-proxy)** rather than Lambda Proxy Integration.

In non-proxy mode, API Gateway passes the unwrapped payload directly as the top-level `event` dictionary rather than wrapping it inside `event['body']`.

**Resolution:**
Updated `lambda_function.py` to reference `event` directly:

```python
# Old (Proxy mode assumption):
# data = json.loads(event['body'])

# New (Non-proxy mode fix):
data = event
```

---

## 📸 Screenshots & Proof of Delivery

*(Add your stage screenshots here)*

* **SES Verification**: Both sender and recipient emails verified in the SES Sandbox.

<img width="951" height="371" alt="p1-ses-step1" src="https://github.com/user-attachments/assets/5ceb9ec4-b13c-4e1a-99d0-99726e9e62ea" />

<img width="959" height="373" alt="p1-ses-step2" src="https://github.com/user-attachments/assets/0a95e63d-0198-4e9d-938a-bef11d208c5d" />

<img width="959" height="377" alt="p1-ses-step3" src="https://github.com/user-attachments/assets/3abe7019-4b7a-443a-8472-f7f9e548917b" />

<img width="959" height="378" alt="p1-ses-step4" src="https://github.com/user-attachments/assets/ab1935ef-7979-4620-bc29-f9047c3d9809" />


* **SES, IAM Execution roles & Lambda**: Add a email lambda function to use SES to send emails for the serverless application.

<img width="959" height="377" alt="s1" src="https://github.com/user-attachments/assets/ea8ac56f-9fc2-450f-a20e-9609b6527dbf" />

<img width="959" height="379" alt="s2" src="https://github.com/user-attachments/assets/055e4fc2-092b-4742-9d1f-297fb7d8cea7" />

<img width="959" height="377" alt="s3" src="https://github.com/user-attachments/assets/6ae3925d-554a-46ae-a5ba-c5f452cc7ea7" />

<img width="959" height="380" alt="s4" src="https://github.com/user-attachments/assets/54d1cfb7-a9b5-485b-b6fc-239da4c15c7f" />

<img width="959" height="372" alt="s5" src="https://github.com/user-attachments/assets/7bd4df4d-f459-4db9-b3ec-f929de92c30d" />

* **Step Functions Execution**: Visual workflow running and waiting.

<img width="959" height="360" alt="s1" src="https://github.com/user-attachments/assets/bd2a1e44-c710-4595-a5b5-93f157dc4567" />

<img width="959" height="354" alt="s2" src="https://github.com/user-attachments/assets/902f9617-d49e-4d6d-8af6-c2bc339920e9" />

<img width="959" height="376" alt="s3" src="https://github.com/user-attachments/assets/3ffaad69-e40c-4a42-be41-b3f5624c2132" />

<img width="959" height="374" alt="s4" src="https://github.com/user-attachments/assets/307665ae-26c1-4bfd-80f4-5411c9a008d9" />

<img width="854" height="389" alt="s5" src="https://github.com/user-attachments/assets/16df1d78-b6c9-41e4-b920-d4f1d95ba467" />

<img width="959" height="352" alt="s6" src="https://github.com/user-attachments/assets/0d091f0f-cb6a-4136-9634-946b1519f6af" />

<img width="959" height="355" alt="s7" src="https://github.com/user-attachments/assets/0f4f6dfe-efd1-4be4-af1d-eaf59dd96cb0" />

<img width="959" height="350" alt="s8" src="https://github.com/user-attachments/assets/9451f321-4b76-4f54-887d-18eff6008775" />

* **API & Lambda Test**: API Gateway successfully passing requests to Lambda.

<img width="947" height="376" alt="Screenshot 2026-09-29 215717" src="https://github.com/user-attachments/assets/cf313b5e-42d3-4f1a-be71-2a8ee681ba1e" />

<img width="957" height="377" alt="Screenshot 2026-09-29 215745" src="https://github.com/user-attachments/assets/bee4ecba-32f1-4bc8-8b68-108f228b57b2" />

<img width="956" height="382" alt="Screenshot 2026-09-29 220052" src="https://github.com/user-attachments/assets/2a05a7b3-f4ba-4188-9134-fc15d78ec3a2" />

<img width="943" height="375" alt="Screenshot 2026-09-29 220146" src="https://github.com/user-attachments/assets/b0f73980-6c18-417d-909b-2600925c1b93" />

<img width="959" height="380" alt="Screenshot 2026-09-29 220228" src="https://github.com/user-attachments/assets/ed9737ee-eb61-4061-86eb-f32986874668" />

<img width="959" height="379" alt="Screenshot 2026-09-29 220314" src="https://github.com/user-attachments/assets/405f28e5-13b1-4e8d-829b-031f657fde62" />

<img width="950" height="383" alt="Screenshot 2026-09-29 220420" src="https://github.com/user-attachments/assets/afdd25db-b6e2-41fd-8553-6d7b080bf1b5" />

<img width="945" height="355" alt="Screenshot 2026-09-29 220546" src="https://github.com/user-attachments/assets/2fc040c8-7264-44e4-82f4-c64b639ceebe" />

<img width="959" height="377" alt="Screenshot 2026-09-29 220626" src="https://github.com/user-attachments/assets/af97fcf4-6381-49d0-b0ea-d43d0b22c547" />

<img width="951" height="356" alt="Screenshot 2026-09-29 220748" src="https://github.com/user-attachments/assets/453f2665-3919-4a99-a746-8f76a4aa5c55" />

<img width="688" height="389" alt="Screenshot 2026-09-29 220848" src="https://github.com/user-attachments/assets/f560b583-38af-4027-bc1b-be735d3e7b61" />

* **S3 Static Website Hosting**: Static website hosting(front end files) using S3 bucket.

<img width="951" height="358" alt="Screenshot 2026-09-29 221040" src="https://github.com/user-attachments/assets/8d73a909-44a6-4c09-ad71-18a38422c66e" />

<img width="954" height="382" alt="Screenshot 2026-09-29 221128" src="https://github.com/user-attachments/assets/c722863f-733a-46b0-8445-9e7e02c62207" />

<img width="900" height="353" alt="Screenshot 2026-09-29 221341" src="https://github.com/user-attachments/assets/7c77a735-65f9-4822-b2d7-3d6715801b2e" />

<img width="956" height="377" alt="Screenshot 2026-09-29 221424" src="https://github.com/user-attachments/assets/ecccea51-f256-4c83-b959-f5e89a9cfea1" />

<img width="902" height="460" alt="Screenshot 2026-09-29 222146" src="https://github.com/user-attachments/assets/8de47964-c19d-4f54-81e5-c09cf62112d7" />

* **Validation and Verification**: Email Received - Proof of the reminder arriving in the inbox.

<img width="743" height="467" alt="Screenshot 2026-09-29 230351" src="https://github.com/user-attachments/assets/9634e41b-43aa-46ff-9e2f-b062a8561de3" />

<img width="953" height="373" alt="Screenshot 2026-09-29 230453" src="https://github.com/user-attachments/assets/745765e3-bd9f-43bb-9a05-c4fde34d29bc" />

<img width="957" height="378" alt="Screenshot 2026-09-29 230530" src="https://github.com/user-attachments/assets/7c01d3b2-2b91-47f9-8f0c-9039ed055c6a" />

<img width="950" height="386" alt="Screenshot 2026-09-29 230547" src="https://github.com/user-attachments/assets/8edec7a5-1a3b-409a-9dd1-1c88a51d18cf" />

<img width="875" height="321" alt="Screenshot 2026-09-29 230848" src="https://github.com/user-attachments/assets/26a78384-fd52-406c-9c82-2cd9419d8240" />

---

## 🧹 Resource Cleanup

To follow AWS security and cost best practices, all resources were tested and completely deleted after project completion.

Resources included:

* S3 bucket
* API Gateway endpoint
* Lambda functions
* Step Functions state machine
* IAM roles
