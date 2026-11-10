# Google Cloud Storage (GCS) - Object Versioning Study Notes

---

### 1. Overview
Object Versioning in Google Cloud Storage (GCS) is a feature designed to protect data from accidental overwrites or accidental deletions [1]. When object versioning is enabled on a bucket, GCS automatically preserves older iterations of a file whenever it is modified, replaced, or deleted [1]. 

Because GCS objects are immutable (meaning you cannot edit parts of a file directly) [2], any change or deletion normally replaces or removes the entire object. Versioning introduces a history track, allowing administrators to restore previous states, retrieve past iterations, and mitigate human error [1].

---

### 2. ACE Exam Focus & Common Exam Traps

- **Exam Importance:** **HIGH**
- **Core Concepts:**
  - Object versioning is enabled and disabled at the **bucket level** [1].
  - When versioning is active, the most current active file is known as the **live version** [1].
  - Deleting a live version does not permanently erase the file; instead, it turns that version into a **non-current object version** and makes the previous historical version (if any) live again [1].
  - Deleting a **non-current version** permanently and irreversibly removes it from Cloud Storage [1].
- **High-Priority Examination Topics:**
  - Identification mechanics: Every object version is uniquely tracked using its **object key plus a generation number** [1].
  - Default behavior: Standard read requests targeting just the object key always point to the **live version** [1]. To access older versions, you must reference the specific generation number [1].
  - Cost implications: Storing multiple versions increases storage consumption, driving up storage costs [1]. Lifecycle rules should be used to purge old non-current versions automatically [1].

---

### 3. Detailed Concept Explanation

#### A. How Object Versioning Works
1. **Enabling Versioning:** Configured at the **bucket level** [1]. It can be toggled on or off at any time [1].
2. **Live Version vs. Non-Current Version:**
   * **Live Version:** The current, active iteration of an object that applications reference by default via its standard object key [1].
   * **Non-Current Version:** An older historical iteration of an object created when a live version is overwritten or deleted [1].
3. **Deletion Behavior:**
   * Deleting a *live version* converts it into a *non-current version* [1]. (If an older version existed prior, it can become live again; if no previous version existed, the object enters a "soft-deleted" placeholder state known as a delete marker).
   * Deleting a *non-current version* **permanently deletes** it from GCS, making it unrecoverable [1].

#### B. Identification: Object Key + Generation Number
* Older versions cannot be retrieved simply by calling the base file name, because default requests target the live version [1].
* GCS assigns a unique **generation number** to every version created [1].
* Syntax structure for referencing history: `[Object Key] + [Generation Number]` (e.g., file v1, v2, v3) [1].

#### C. Cost Management & Storage Implications
* Because multiple versions of the same file occupy physical space inside your bucket, your storage consumption increases, leading to higher billing costs [1].
* **Mitigation:** To control costs, administrators can configure **Object Lifecycle Management** rules to automatically purge non-current versions older than a specific threshold (e.g., non-current versions older than 15 days) [1].

---

### 4. Practical Setup & Implementation Guide

*No command-line code execution block was provided in this specific transcript segment. However, versioning can be toggled via `gsutil` or the Google Cloud Console:*

* **CLI Example (Enabling Versioning on a Bucket):**
  ```bash
  gsutil versioning set on gs://my-bucket-name
  ```
* **CLI Example (Checking Versioning Status):**
  ```bash
  gsutil versioning get gs://my-bucket-name
  ```
* **CLI Example (Listing all versions of an object including non-current ones):**
  ```bash
  gsutil ls -a gs://my-bucket-name/path/to/object
  ```

---

### 5. Visual Representations

#### Object Version Lifecycle Flow
```text
[Bucket: Versioning Enabled]
   │
   ├── Upload File.txt (Version 1) ──────> Generation ID: 101 (Becomes Live Version)
   │
   ├── Overwrite / Update File.txt ──────> Generation ID: 102 (Becomes New Live Version)
   │                                     └─> Generation ID: 101 shifts to Non-Current Version
   │
   └── User Deletes Live File ───────────> Generation ID: 102 shifts to Non-Current Version
         │
         ├── Option A: Restore Generation ID 101 or 102 (Recoverable)
         └── Option B: Delete Non-Current Version explicitly ──> PERMANENTLY DELETED FROM GCS [1]
```

---

### 6. Mental Models & Analogies

* **The Document Revision History (Google Docs / Microsoft Word):**
  * **Analogy:** When you type a document and hit save repeatedly, modern editors keep a version history. If you accidentally delete a paragraph or overwrite a whole section, you don't lose your work forever; you simply open the Version History menu and restore yesterday's draft. However, if you manually go into the history log and hit "Delete permanently" on an old draft, it is gone for good.
  * **Mapping to GCP:** The document is the GCS object, the version history list represents generation numbers, and hitting permanent delete on a non-current version removes it entirely.

---

### 7. Step-by-Step Workflows / Comparison Tables

#### Live Version vs. Non-Current Version Comparison

| Metric / Action | Live Version | Non-Current Version |
| :--- | :--- | :--- |
| **Status** | Current, active version of the file [1]. | Historical, older iteration of the file [1]. |
| **Default Access Path** | Accessed via standard object key path alone [1]. | Accessed via object key **plus generation number** [1]. |
| **Result of Deletion** | Converts into a non-current version (or creates a delete marker) [1]. | **Permanently and irrevocably deleted** from GCS [1]. |
| **Cost Impact** | Standard storage pricing. | Adds to total storage footprint; requires lifecycle rules to purge and save costs [1]. |

---

### 8. Common Mistakes & Exam Traps

* **Mistake:** Assuming that deleting a file when versioning is enabled completely erases it immediately and forever.
  * **Correction:** Deleting a live version turns it into a *non-current version*, protecting it from immediate data loss. It is only permanently deleted if you delete the *non-current version* itself [1].
* **Exam Trap:** Trying to retrieve an old version of a file by requesting its standard file name/key.
  * **Correction:** Standard requests always fetch the **live version**. To get an older file version, you must specify its unique **generation number** [1].
* **Mistake:** Assuming versioning is configured at the individual object level.
  * **Correction:** Object versioning is toggled strictly at the **bucket level** and applies to all objects inside that bucket [1].

---

### 9. Certification Practice Questions

*(Note: These 10 practice questions integrate concepts across all shared GCS transcripts up to this point, including storage classes, uploads, structural constraints, and versioning).*

#### Q1: An administrator enables object versioning on a GCS bucket. A user accidentally deletes an active file ("live version"). What happens to that file inside the bucket?
- A. The file is permanently deleted immediately and cannot be recovered.
- B. The file becomes a non-current version, and can be restored or accessed using its generation number.
- C. The file automatically moves to the Archive storage class.
- D. The bucket disables versioning automatically due to a conflict.
- **Correct Answer:** B
- **Explanation:** Deleting a live version turns it into a non-current version rather than destroying it instantly, allowing recovery [1].
- **Incorrect Answers:** A describes behavior when versioning is disabled or when non-current versions are deleted. C and D are unrelated automation behaviors.
- **Exam Takeaway:** Deleting a live version with versioning enabled turns it into a non-current version [1].

#### Q2: How can a user programmatically or via CLI access an older, non-current version of a GCS object rather than the active live version?
- A. By appending the project ID to the file name.
- B. By specifying the object key combined with its unique generation number.
- C. By changing the bucket storage class to Coldline.
- D. By clearing the deletion marker flag via standard HTTP GET.
- **Correct Answer:** B
- **Explanation:** Older versions are tracked and accessed using the object key plus their specific generation number [1].
- **Incorrect Answers:** A, C, and D do not target historical object generation pointers.
- **Exam Takeaway:** Generation numbers identify non-current object versions [1].

#### Q3: What happens when a user explicitly deletes a *non-current* object version from a version-enabled GCS bucket?
- A. It converts back into a live version.
- B. It gets archived for 365 days.
- C. It is permanently and completely deleted from Cloud Storage [1].
- D. It triggers an automatic multi-region backup snapshot.
- **Correct Answer:** C
- **Explanation:** Deleting non-current versions purges them permanently from storage [1].
- **Incorrect Answers:** A, B, and D are incorrect recovery or class behaviors.
- **Exam Takeaway:** Deleting non-current versions results in permanent deletion [1].

#### Q4: At what level is object versioning configured in Google Cloud Storage?
- A. Project level
- B. Bucket level [1]
- C. Individual object/file level
- D. Organization policy root level only
- **Correct Answer:** B
- **Explanation:** Object versioning is enabled and disabled at the bucket level [1].
- **Incorrect Answers:** A, C, and D misstate the configuration scope.
- **Exam Takeaway:** Versioning is controlled at the bucket level [1].

#### Q5: A company wants to keep historical versions of files for audit recovery, but wants to automatically purge non-current versions older than 30 days to reduce storage costs. What feature should they use?
- A. Object Lifecycle Management rules [1]
- B. Public Access Prevention
- C. Parallel Composite Upload throttling
- D. Autoclass regional locking
- **Correct Answer:** A
- **Explanation:** Lifecycle rules allow automated management of non-current versions, such as deleting items older than a certain age to lower costs [1].
- **Incorrect Answers:** B handles public exposure. C handles upload streams. D optimizes storage classes, not version retention cleanup directly.
- **Exam Takeaway:** Use lifecycle rules to purge old non-current versions and control costs [1].

#### Q6: Which storage class should be chosen for data that is accessed less than once a year and has a 365-day minimum storage duration requirement?
- A. Standard
- B. Nearline
- C. Coldline
- D. Archive
- **Correct Answer:** D
- **Explanation:** Archive storage is designed for deep long-term digital preservation and data accessed less than once a year.
- **Incorrect Answers:** A, B, and C have shorter minimum durations and higher storage tier costs for yearly access profiles.
- **Exam Takeaway:** Less than once a year access maps to Archive storage.

#### Q7: An application needs to upload a large multi-gigabyte file over a network connection that frequently drops packets and disconnects. Which upload method protects the transfer from starting over from 0%?
- A. Simple Upload
- B. Resumable Upload
- C. Multipart Form Upload without headers
- D. Parallel Single-Threaded Stream
- **Correct Answer:** B
- **Explanation:** Resumable uploads allow transfers to resume from the exact byte where a network failure occurred.
- **Incorrect Answers:** A forces a complete restart. C and D lack resume checkpoints.
- **Exam Takeaway:** Network instability on large uploads requires **Resumable Uploads**.

#### Q8: What is the maximum size limit for a single object stored in Google Cloud Storage?
- A. 5 Gigabytes
- B. 100 Gigabytes
- C. 5 Terabytes
- D. Unlimited file size
- **Correct Answer:** C
- **Explanation:** GCS permits a maximum size of 5 TB for any single object.
- **Incorrect Answers:** A and B are too low. D is incorrect because individual object size is capped at 5 TB, even though total bucket capacity is infinite.
- **Exam Takeaway:** 5 TB is the single object size limit.

#### Q9: Two separate project teams try to create new GCS buckets with the exact same name, `shared-log-bucket`. What will happen?
- A. Both succeed because they belong to different projects.
- B. Only the first team succeeds because bucket names share a single global namespace.
- C. GCP automatically appends a random string hash to the second bucket.
- D. The buckets merge into a multi-project container.
- **Correct Answer:** B
- **Explanation:** Bucket names must be globally unique across all of Google Cloud, meaning no two projects can use the same name.
- **Incorrect Answers:** A, C, and D are false.
- **Exam Takeaway:** Global uniqueness is required for all GCS bucket names.

#### Q10: When a bucket has versioning enabled, what term describes the current active file that applications read by default using the standard object key?
- A. The non-current version
- B. The delete marker
- C. The live version [1]
- D. The snapshot clone
- **Correct Answer:** C
- **Explanation:** The live version is the active, latest iteration of an object referenced by standard requests [1].
- **Incorrect Answers:** A represents historical states. B represents deletion placeholders. D is a term used in persistent disks, not GCS objects.
- **Exam Takeaway:** The latest active version is called the **live version** [1].