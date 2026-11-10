# Google Cloud Storage (GCS) - CLI Operations (`ls`, `rm`, `cp`, `sign-url`) Study Notes

---

### 1. Overview
Building upon general GCS command structures, this module covers four essential operational subgroups under the modern **`gcloud storage`** command-line interface: **`ls`** (listing buckets and objects with advanced filtering), **`rm`** (removing files, directories, or entire buckets recursively), **`cp`** (copying files between local machines, GCS buckets, and even external cloud storage providers like AWS S3), and **`sign-url`** (generating time-bound, cryptographically signed URLs to grant temporary, controlled download or upload access to protected objects) [1].

---

### 2. ACE Exam Focus & Common Exam Traps

- **Exam Importance:** **HIGH**
- **Core Concepts:**
  - **`gcloud storage ls`**: 
    - Running it alone lists bucket URLs in the active project [1].
    - Passing a bucket URL lists root-level objects [1].
    - Using wildcards like `/**` lists items recursively across subdirectories [1], and `/**/*.png` filters by file extensions.
  - **`gcloud storage rm`**: 
    - Using `/*` deletes items only in the immediate directory (non-recursive).
    - Using `/**` deletes objects and subdirectories recursively, but **only affects live versions** (historical non-current versions remain untouched).
    - Running `rm --recursive` on a bucket URL deletes all versions of all resources *and* deletes the bucket itself.
  - **`gcloud storage cp`**: 
    - Handles local-to-cloud uploads, cloud-to-local downloads, and cross-cloud transfers (e.g., Google Cloud Storage to AWS S3).
  - **`gcloud storage sign-url`**: 
    - Generates temporary access links for protected objects.
    - Can configure expiration windows (e.g., `--duration=1h`) and HTTP verbs (`--http-method=PUT` for uploads or default `GET` for downloads), and requires a service account for impersonation.
- **High-Priority Examination Topics:**
  - Recognizing the functional difference between single-star (`/*`) and double-star (`/**`) wildcard patterns in delete and list commands.
  - Understanding that signed URLs can grant temporary upload capabilities using the `PUT` method, not just downloads.

---

### 3. Detailed Concept Explanation

#### A. The `ls` Subgroup (Listing & Filtering)
* **Project-Level Listing:** Running `gcloud storage ls` without arguments lists all GCS buckets within the active project context (requires a properly configured project or `--project` flag) [1].
* **Root-Level Listing:** Running `gcloud storage ls gs://my-bucket` lists files and folder paths residing strictly at the root level of that specific container [1].
* **Recursive Wildcard Filtering:**
  * `gs://my-bucket/**` $\rightarrow$ Lists all files recursively across all nested subdirectories [1].
  * `gs://my-bucket/**/*.png` $\rightarrow$ Filters output to list only files ending with the `.png` extension across all directories.

#### B. The `rm` Subgroup (Removal & Cleanup)
* **Single-Star vs. Double-Star Scope:**
  * `gs://my-bucket/my-dir/*` $\rightarrow$ Deletes objects directly inside `my-dir`, but **ignores subdirectories** [1].
  * `gs://my-bucket/my-dir/**` $\rightarrow$ Recursively deletes all objects and subdirectories inside `my-dir` [1].
* **Version Limitation Warning:** Standard `rm` commands targeting live files **only affect live versions** [1]. Historical non-current versions remain preserved unless explicitly targeted or purged.
* **Recursive Bucket Deletion:** Running `gcloud storage rm --recursive gs://my-bucket-name` deletes **all versions of all resources** inside the bucket and then **deletes the bucket container itself** [1].

#### C. The `cp` Subgroup (Copying & Cross-Cloud Transfers)
* **Local to Cloud Uploads:** `gcloud storage cp *.txt gs://my-bucket` copies local text files into the specified GCS container [1].
* **Cloud to Local Downloads:** `gcloud storage cp gs://my-bucket/*.txt .` pulls text files from the bucket down to your current local machine directory [1].
* **Cross-Cloud Transfers (GCS to AWS S3):** `gcloud storage cp` natively supports copying data between GCP storage and external cloud storage environments (e.g., copying a GCS object URL directly to an AWS S3 bucket destination path) [1].

#### D. The `sign-url` Subgroup (Temporary Access Control)
* **Purpose:** Grants time-bound, secure access to private, protected objects for users who lack direct IAM permissions on the bucket [1].
* **Download Signed URLs (Default `GET`):** Generates a temporary link allowing an external user to download a specific file for a limited duration (e.g., 1 hour) by impersonating a designated service account [1].
* **Upload Signed URLs (`PUT`):** Allows an end-user to *upload* a file to a precise path using the HTTP `PUT` method, often restricted by content type constraints (e.g., permitting only text files for a 1-hour window) [1].

---

### 4. Practical Setup & Implementation Guide

#### Executing CLI Operations in Cloud Shell
1. **Prerequisites:** Active Google Cloud CLI session authenticated with appropriate permissions (`Storage Object Viewer`, `Storage Admin`, or Service Account Token Creator for signed URLs).
2. **Step 1: Listing Project Buckets**
   ```bash
   gcloud storage ls
   ```
   *Expected outcome:* Returns the `gs://` URLs of all buckets in your active project [1].
3. **Step 2: Recursive Text File Listing**
   ```bash
   gcloud storage ls gs://my-bucket-name/**/*.txt
   ```
   *Expected outcome:* Recursively traverses all folders and lists only `.txt` files [1].
4. **Step 3: Copying Local Files to Cloud**
   ```bash
   gcloud storage cp *.txt gs://my-bucket-name/uploads/
   ```
   *Expected outcome:* Uploads all local text files into the specified cloud directory path [1].
5. **Step 4: Generating a Temporary Signed Download URL**
   ```bash
   gcloud storage sign-url gs://my-bucket-name/report.pdf --duration=1h --impersonate-service-account=my-sa@my-project.iam.gserviceaccount.com
   ```
   *Expected outcome:* Generates a secure, time-limited URL valid for 1 hour that allows downloading `report.pdf` via service account impersonation [1].
6. **Step 5: Generating a Temporary Signed Upload URL (`PUT`)**
   ```bash
   gcloud storage sign-url gs://my-bucket-name/uploads/incoming.txt --http-method=PUT --duration=30m --impersonate-service-account=my-sa@my-project.iam.gserviceaccount.com
   ```
   *Expected outcome:* Returns a secure URL enabling an external client to upload a file using HTTP `PUT` for a 30-minute window [1].

---

### 5. Visual Representations

#### Wildcard Scope Comparison (`*` vs `/**`)
```text
[Bucket: my-bucket]
   └── my-dir/
         ├── file1.txt (Direct child)
         ├── file2.png (Direct child)
         └── sub-folder/
               └── file3.txt (Nested child)

 Command A: gcloud storage rm gs://my-bucket/my-dir/*
   └─> Affects: file1.txt and file2.png ONLY. (Subdirectories and file3.txt are skipped)

 Command B: gcloud storage rm gs://my-bucket/my-dir/**
   └─> Affects: file1.txt, file2.png, sub-folder/, and file3.txt recursively.
```

#### Signed URL Authorization Flow
```text
[External End User] ──(Requests Signed Link)──> [GCP Admin / CLI (`sign-url`)]
                                                       │
                                        (Generates Cryptographic Token bound
                                         to Service Account + Duration limit)
                                                       │
[External End User] <──(Receives Signed URL)───────────┘
   │
   └──(Accesses GCS Object directly via URL within time window [1])
         │
         ▼
   [Google Cloud Storage (GCS)]
```

---

### 6. Mental Models & Analogies

* **The VIP Backstage Pass (`sign-url`):**
  * **Analogy:** Your concert venue (GCS bucket) is locked down with strict security guards (IAM policies). A fan wants to get inside for just one specific song. Instead of giving them a permanent employee badge, management prints a special VIP wristband that expires automatically in exactly 60 minutes and only grants access to stage door B. 
  * **Mapping to GCP:** The VIP wristband represents a signed URL generated via `gcloud storage sign-url`, granting temporary, scoped access without altering permanent bucket IAM roles [1].

---

### 7. Step-by-Step Workflows / Comparison Tables

#### GCS CLI Subgroup Operations Summary Table

| Subgroup Tool | Core Purpose / Action | Key Flag / Wildcard Syntax | Important Operational Rule |
| :--- | :--- | :--- | :--- |
| **`ls`** | List buckets or contents [1] | `/**` (recursive), `/**/*.png` (filter) | Requires explicit `gs://` target paths for content listing. |
| **`rm`** | Delete objects or buckets [1] | `/*` (surface), `/**` (recursive), `--recursive` | Only affects **live versions** unless explicitly purged; `--recursive` deletes bucket containers. |
| **`cp`** | Copy files across environments [1] | Source & Destination paths (local, GCS, AWS S3) | Supports local uploads, cloud downloads, and cross-cloud transfers [1]. |
| **`sign-url`** | Generate temporary access links [1] | `--duration`, `--http-method=PUT`, `--impersonate-service-account` | Binds access to a service account for a limited time window [1]. |

---

### 8. Common Mistakes & Exam Traps

* **Mistake:** Assuming that running `gcloud storage rm gs://bucket/dir/**` will delete all historical non-current versions of files in version-enabled buckets.
  * **Correction:** Standard delete commands (`rm`) target and affect **live versions only** [1]. Non-current historical versions persist unless explicitly targeted or cleared via lifecycle rules.
* **Exam Trap:** Confusing `/*` and `/**` scope in deletion commands.
  * **Correction:** A single star (`/*`) only deletes items in the immediate directory level, leaving subdirectories intact. A double star (`/**`) deletes items **recursively** across all nested subdirectories [1].
* **Mistake:** Assuming signed URLs are strictly for downloading files.
  * **Correction:** Signed URLs can also be configured using `--http-method=PUT` to grant external users temporary, secure permissions to **upload** files to specific paths [1].

---

### 9. Certification Practice Questions

#### Q1: An administrator executes the command `gcloud storage rm gs://my-bucket/folder/*` (with a single star). What is the impact on files inside `my-folder/sub-folder/`?
- A. All files and subdirectories are recursively deleted.
- B. Only files directly inside `folder/` are deleted; files inside `sub-folder/` are left untouched [1].
- C. The command fails because single stars are illegal in GCS paths.
- D. The entire bucket container is deleted.
- **Correct Answer:** B
- **Explanation:** A single star (`/*`) limits deletion to the immediate surface directory level and does not traverse subdirectories [1].
- **Incorrect Answers:** A describes double-star (`/**`) behavior. C is false. D requires the `--recursive` flag on the bucket root.
- **Exam Takeaway:** Single star (`/*`) targets surface files only, whereas double star (`/**`) targets recursively.

#### Q2: You need to delete all objects, nested subdirectories, *and* the bucket container itself named `my-obsolete-bucket` in a single command. Which command should you run?
- A. `gcloud storage rm gs://my-obsolete-bucket`
- B. `gcloud storage rm --recursive gs://my-obsolete-bucket` [1]
- C. `gcloud storage buckets purge gs://my-obsolete-bucket`
- D. `gcloud storage delete-all gs://my-obsolete-bucket`
- **Correct Answer:** B
- **Explanation:** Passing the `--recursive` flag to a bucket URL with `gcloud storage rm` deletes all versions of all resources inside the bucket and then removes the bucket itself [1].
- **Incorrect Answers:** A fails if objects still exist. C and D are invalid command structures.
- **Exam Takeaway:** Recursive bucket deletion uses `gcloud storage rm --recursive`.

#### Q3: An enterprise user wants to copy data directly from a Google Cloud Storage bucket into an AWS S3 bucket container destination. Is this supported by the `gcloud storage cp` utility?
- A. No, `cp` is strictly restricted to local machine transfers.
- B. Yes, `gcloud storage cp` natively supports cross-cloud transfers between GCS and external object storage services like AWS S3 [1].
- C. Yes, but only if an intermediary Compute Engine Linux instance acts as a proxy bridge.
- D. No, cross-cloud copying requires BigQuery data transfer services.
- **Correct Answer:** B
- **Explanation:** The `cp` subgroup supports cross-cloud transfers, allowing direct copy operations from Google Cloud Storage to AWS S3 [1].
- **Incorrect Answers:** A, C, and D are incorrect limitations.
- **Exam Takeaway:** `gcloud storage cp` supports cross-cloud object transfers [1].

#### Q4: A client needs to download a secure, private GCS object for a limited duration of 30 minutes without being granted permanent IAM access to the bucket. What tool/command should you provide?
- A. `gcloud storage sign-url` configured with a 30m duration [1]
- B. `gcloud storage buckets update --public`
- C. `gcloud storage acl grant-temporary`
- D. `gsutil temporary-access`
- **Correct Answer:** A
- **Explanation:** The `sign-url` subgroup generates time-bound, cryptographically signed URLs allowing temporary access to protected objects [1].
- **Incorrect Answers:** B exposes the bucket permanently. C and D are incorrect command syntax.
- **Exam Takeaway:** Temporary access to private objects uses **`gcloud storage sign-url`** [1].

#### Q5: Can a signed URL generated via `gcloud storage sign-url` be configured to allow an external user to *upload* a file to GCS rather than download it?
- A. No, signed URLs are strictly read-only for downloads.
- B. Yes, by specifying the HTTP method option (e.g., `--http-method=PUT`) [1].
- C. Yes, by changing the bucket storage class to Archive.
- D. No, uploads always require full IAM service account authentication keys.
- **Correct Answer:** B
- **Explanation:** Signed URLs can be configured for write operations by setting the HTTP method to `PUT`, enabling temporary file uploads [1].
- **Incorrect Answers:** A, C, and D are false.
- **Exam Takeaway:** Signed URLs support temporary **uploads** using `--http-method=PUT` [1].

#### Q6: When object versioning is enabled on a bucket, what happens to historical non-current versions when you run a standard `gcloud storage rm` command on a live file?
- A. All non-current versions are purged alongside the live file.
- B. The non-current versions remain untouched in the bucket; standard `rm` commands only affect live versions [1].
- C. The non-current versions automatically convert back into live files.
- D. The delete command fails with a version conflict error.
- **Correct Answer:** B
- **Explanation:** Standard `rm` execution targets and affects live versions only; older historical non-current versions persist [1].
- **Incorrect Answers:** A, C, and D are incorrect behavior descriptions.
- **Exam Takeaway:** Standard delete operations affect **live versions only** [1].

#### Q7: A developer runs `gcloud storage ls gs://my-bucket/**/*.png`. What will the output return?
- A. A list of all files in the bucket root directory.
- B. A recursive listing of all `.png` files across all nested subdirectories in the bucket [1].
- C. An error because double wildcards cannot precede file extensions.
- D. The total byte size count of all PNG graphics.
- **Correct Answer:** B
- **Explanation:** The `/**/` pattern combined with an extension filter recursively evaluates all subdirectories for matching `.png` files [1].
- **Incorrect Answers:** A ignores subdirectories. C is false. D is a sizing query, not a listing query.
- **Exam Takeaway:** Recursive extension filtering uses `/**/` wildcards [1].

#### Q8: What mandatory parameter must be supplied when generating a signed URL using `gcloud storage sign-url` to define the security context under which the operation executes?
- A. `--project-owner-email`
- B. `--impersonate-service-account` [1]
- C. `--kms-crypto-key`
- D. `--iam-role-override`
- **Correct Answer:** B
- **Explanation:** Generating signed URLs requires specifying the service account that the accessor will impersonate [1].
- **Incorrect Answers:** A, C, and D are incorrect flags.
- **Exam Takeaway:** Signed URL generation requires a service account for impersonation [1].

#### Q9: What happens if you run `gcloud storage ls` without passing any target bucket URL arguments?
- A. It lists all objects in the default project bucket.
- B. It lists the bucket URLs of all buckets present in the active GCP project [1].
- C. It returns a syntax error requiring an explicit bucket path.
- D. It lists all available cloud storage regions globally.
- **Correct Answer:** B
- **Explanation:** Running `gcloud storage ls` without arguments lists all bucket container URLs in the active project [1].
- **Incorrect Answers:** A, C, and D misstate the default behavior of project-level `ls`.
- **Exam Takeaway:** Running `gcloud storage ls` alone lists all project buckets [1].

#### Q10: Which subgroup under `gcloud storage` should an administrator invoke if they need to copy a file from a local directory into a cloud bucket?
- A. `ls`
- B. `rm`
- C. `cp` [1]
- D. `sign-url`
- **Correct Answer:** C
- **Explanation:** The `cp` subgroup manages file copying operations between local environments, cloud buckets, and external providers [1].
- **Incorrect Answers:** A lists items. B deletes items. D generates signed URLs.
- **Exam Takeaway:** File copying uses the **`cp`** subgroup [1].