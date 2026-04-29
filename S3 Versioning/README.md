# AWS S3 Versioning + Static Website Hosting Lab

## Overview

This lab demonstrates how **Amazon S3 Versioning** works along with **Static Website Hosting**, including how object updates, deletions, and recovery behave in a version-enabled bucket.

---

##  Services Used

* Amazon S3
* S3 Static Website Hosting

---

## Steps:

### 1. Create S3 Bucket

* Bucket Name: `mys3-versioning-example-bucket`
* Enabled **Versioning**
* Disabled **Block Public Access** (for demo purposes)

<img width="958" height="353" alt="create-bucket-1" src="https://github.com/user-attachments/assets/cf145a3f-60f4-46c2-aefb-79724bda77fd" />


---

### 2. Enable Static Website Hosting

* Enabled static website hosting
* Configured:

  * **Index document** → `index.html`
  * (Optional) Error document → `error.html`

---

### 3. Configure Bucket Policy (Public Read Access)

```json
{
  "Version":"2012-10-17",
  "Statement":[
    {
      "Sid":"PublicRead",
      "Effect":"Allow",
      "Principal": "*",
      "Action":["s3:GetObject"],
      "Resource":["arn:aws:s3:::mys3-versioning-example-bucket/*"]
    }
  ]
}
```

---

### 4. Upload Objects

* Uploaded:

  * `index.html`
  * `img/` folder
  * `img/catname.jpg`
 
 <img width="931" height="373" alt="hide-versioning-2" src="https://github.com/user-attachments/assets/301db675-7a31-467d-8f32-b9648ee9fbbd" />


---

### 5. Observe Versioning Behavior

#### 🔹 Initial Upload (show versioning toggle on)

* `catname.jpg` → Version ID: `v1`
 <img width="950" height="374" alt="show-versions-3" src="https://github.com/user-attachments/assets/48e4b106-961c-4909-bbd0-77a54df19299" />

#### 🔹 Initial Upload (show versioning toggle off)

<img width="950" height="373" alt="hide-version-4-samefileappearsoverwriiten" src="https://github.com/user-attachments/assets/fc9b27fb-06bb-4100-9305-bd38e2421d7f" />


#### 🔹 Re-upload Same File Name

* New version created → Version ID: `v2`
* Old version preserved

<img width="946" height="370" alt="after-versioning-versionidsforsamefiles-5" src="https://github.com/user-attachments/assets/5c64a6fe-0dbe-47dd-bfdb-b99360675d8e" />


---

### 6. Delete Object

* Toggle show versions off
* Deleted `catname.jpg`
* S3 created a **Delete Marker**
<img width="934" height="371" alt="delete marker added - 7" src="https://github.com/user-attachments/assets/15d5792b-a745-4310-bac1-7dd218e0e618" />


#### Result:

* Object appears deleted (when show version toggle is off)
* Previous versions still exist
* Website returns **404**

<img width="943" height="347" alt="delete-hideversions-6" src="https://github.com/user-attachments/assets/d37a1587-a6f4-4094-a904-ebdb6f2d26ff" />

---

### 7. Recover Object

#### Option 1: Delete Delete Marker

* Restores latest version

<img width="959" height="377" alt="delete-show-versions-7" src="https://github.com/user-attachments/assets/242c5db6-6857-4e0c-bc7f-6713568f9817" />


#### Option 2: Delete Latest Version

* Previous version becomes current

---

## 🧠 Key Concepts Learned

### ✅ Versioning Behavior

* Each upload creates a new version
* Same object key(image name) → multiple versions

---

### ✅ Delete Marker

* Deleting an object does NOT remove it
* Adds a delete marker (hides object)

---

### ✅ Recovery

* Delete marker removal restores object
* Deleting current version rolls back to previous version
* Deleting both delete market and all versions actually deletes everything inside bucket

---

### ✅ Static Website Behavior

* Only **current version** is served
* If delete marker exists → **404 error**

---

## ⚠️ Important Notes

* Versioning cannot be disabled once enabled (only suspended)
* Permanent deletion requires removing **all versions and Delete marker**
* Block Public Access should remain **enabled in production**
* Use **CloudFront** for secure (HTTPS) delivery

---

## 🎯 Key Takeaways

* Versioning protects against accidental deletion and overwrites
* Delete ≠ permanent removal (when versioning is enabled)
* Static websites rely only on the latest object version
* S3 can act as a lightweight hosting solution for static content

---

## Author

* Hands-on practice based on Cantrill’s course
