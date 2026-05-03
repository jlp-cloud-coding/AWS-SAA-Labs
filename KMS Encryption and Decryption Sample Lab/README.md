# AWS KMS Encryption & Decryption (CloudShell Demo)

This project demonstrates how to use AWS KMS (Key Management Service) to encrypt and decrypt data using **AWS CloudShell**.

---

## Overview

This lab shows how to:

* Create a plaintext file
* Encrypt it using a KMS key alias
* Decrypt it back to plaintext
* Verify results

This uses **direct KMS encryption (< 4KB data)**.

---

## Why CloudShell?

Using AWS CloudShell:

* No AWS CLI setup required
* Pre-authenticated environment
* Runs directly in browser

---

## 📂 Files Used

| File                   | Description        |
| ---------------------- | ------------------ |
| `message.txt`          | Original plaintext |
| `not_a_message.enc`    | Encrypted file     |
| `decryptedmessage.txt` | Decrypted output   |

---

## Prerequisites

* AWS account
* A KMS Key created
* You can provide the key_id or Alias created (e.g., `alias/secret`)

Configure Key: 
<img width="953" height="376" alt="configurekey" src="https://github.com/user-attachments/assets/30faceb8-045a-4b6e-85b0-9afca32ddce1" />

Alias:
<img width="957" height="374" alt="alias-step2" src="https://github.com/user-attachments/assets/53c6a7f7-da60-4586-a571-9abd7eb0b0c7" />

<img width="947" height="374" alt="step4" src="https://github.com/user-attachments/assets/70f5662d-52a7-4dd1-8e0b-bf0a39a1f750" />

<img width="959" height="380" alt="step3" src="https://github.com/user-attachments/assets/30d63ace-a870-4a81-9a3b-a99b2fc86d06" />

<img width="959" height="335" alt="key_created" src="https://github.com/user-attachments/assets/8ee4523e-3184-4f93-8c74-84d102c14ca6" />

---
# Navigate to CloudShell command prompt via AWS Management Console

## Step 1: Create a File

```bash
echo "this is a top secret message for the ruler" > message.txt
```

---

## 🔐 Step 2: Encrypt

```bash
aws kms encrypt \
  --key-id alias/secret \
  --plaintext fileb://message.txt \
  --output text \
  --query CiphertextBlob \
  | base64 --decode > not_a_message.enc
```

---

## View Encrypted Data

```bash
cat not_a_message.enc
```
<img width="943" height="383" alt="encrypted_Cipher_text" src="https://github.com/user-attachments/assets/7ff75e31-b392-470d-9664-0d906760bc9c" />

Output will look like random/unreadable format

---

## 🔓 Step 3: Decrypt

```bash
aws kms decrypt \
  --ciphertext-blob fileb://not_a_message.enc \
  --output text \
  --query Plaintext \
  | base64 --decode > decryptedmessage.txt
```

---

## ✅ Step 4: Verify

```bash
cat decryptedmessage.txt
```

### ✔️ Output:

```
this is a top secret message for the ruler
```
<img width="950" height="377" alt="decrypted_plain_text" src="https://github.com/user-attachments/assets/9fc5948e-e1c0-4f5f-85dd-185c85f362f4" />
---

## Key Concepts

* **Direct KMS Encryption**

  * Used for small data (< 4KB)
  * No DEK involved

* **KMS Key (KEK/Key Encryption Key)**

  * Used directly to encrypt/decrypt data

---

## Notes

* Data is encrypted using KMS key
* KMS does not store your file
* Only ciphertext is stored locally in CloudShell

---

## Cost Awareness

* KMS key costs ~ $1/month
* Usage cost is minimal, covered if account has free AWS credits

---

## Cleanup

1. Go to KMS Console
2. Select your key
3. Click **Schedule Key Deletion**
4. Minimum waiting period: 7 days

---

## Takeaway

* Understand how KMS works in practice
* Learn encryption/decryption flow
