# GCP Associate Cloud Engineer (ACE) Study Notes

## Chapter: IAM Identity and Access Management (Part 3 - Hands-On Exploration of Roles)

---

### 1. Overview
This section covers a practical, hands-on exploration of Google Cloud IAM roles using the GCP Console. It investigates the internal composition of basic roles (`viewer`, `owner`), service-specific predefined roles (`storage.admin`, `storage.objectAdmin`, `storage.objectViewer`), and demonstrates how to create a user-defined **Custom Role** with specific permissions and launch stages.

---

### 2. ACE Exam Focus & Common Exam Traps
- **Exam Importance:** **HIGH**
- **Key Concepts:**
  - **Viewer Role Characteristics:** Contains thousands of permissions, restricted strictly to `get` and `list` actions, making it read-only across the project.
  - **Owner Role Characteristics:** Contains significantly more permissions (~3,800+) than viewer or editor, covering full administrative capabilities including billing and IAM policies.
  - **Storage Role Nuances:**
    - `storage.admin`: Full control over **both** buckets and objects.
    - `storage.objectAdmin`: Full control over **objects only** (cannot modify or create buckets).
    - `storage.objectCreator`: Can only create/upload new objects.
    - `storage.objectViewer`: Read-only access (`get` and `list`) to objects.
  - **Baseline Permissions:** Almost every role includes `resourcemanager.projects.get` and `resourcemanager.projects.list` to permit project visibility in the console.
  - **Custom Role Launch Stages:** `Alpha`, `Beta`, `General Availability (GA)`, and `Disabled`.
- **Common Exam Traps:**
  - Expecting `storage.objectAdmin` to allow bucket creation. (Trap! It only manages objects).
  - Forgetting that custom role IDs are immutable handles used directly inside policy bindings.

---

### 3. Detailed Concept Explanation

#### A. Inspecting Role Permissions in the Console
When exploring roles via **IAM & Admin > Roles**:
- **Viewer Role (`roles/viewer`):** Comprises roughly 1,661 permissions. Every permission mapped inside this role is restricted to non-mutating read actions (`get` and `list`).
- **Owner Role (`roles/owner`):** Comprises roughly 3,801 permissions, incorporating resource creation, modification, deletion, IAM bindings, and billing management.
- **Service-Specific Predefined Storage Roles Breakdown:**
  - `roles/storage.admin`: Permissions include `storage.buckets.*` (create, delete, get, list, update, set/get IAM policy) and `storage.objects.*`.
  - `roles/storage.objectAdmin`: Permissions include `storage.objects.*` (create, delete, get, list, update), but **excludes** bucket management operations.
  - `roles/storage.objectCreator`: Permissions restricted strictly to creating objects (`storage.objects.create`).
  - `roles/storage.objectViewer`: Read-only permissions restricted to `storage.objects.get` and `storage.objects.list`.

#### B. Custom Role Lifecycle and Creation Parameters
When creating a custom role:
- **Role ID / Title:** A unique system identifier used when binding policies programmatically or via CLI.
- **Launch Stages:**
  - `Alpha`: Ideal when initially testing and iterating on custom permissions.
  - `Beta`: Ready for broader internal evaluation.
  - `General Availability (GA)`: Fully ready for production workloads.
  - `Disabled`: Temporarily turns off the role without deleting it.
- **Permission Aggregation:** Custom roles let you cherry-pick individual permissions across multiple GCP services (e.g., combining App Engine permissions with Cloud Storage permissions) into a single logical role unit.

---

### 4. Practical Setup & Implementation Guide

#### Step-by-Step Walkthrough: Exploring and Creating Roles in GCP Console
1. **Navigate to Roles:**
   - Open the GCP Console navigation menu or search bar, type **Roles**, and click on **IAM & Admin > Roles**.
2. **Inspect Predefined Roles:**
   - Filter by name: Type `roles/viewer` to review read-only project permissions.
   - Filter by name: Type `roles/owner` to view high-level project permissions.
   - Search for `storage admin`, `storage object admin`, and `storage object viewer` to evaluate how their permissions differ between buckets and objects.
3. **Create a Custom Role:**
   - Click **+ CREATE ROLE** at the top of the Roles page.
   - Fill in the form fields:
     - **Title:** `My Custom Role`
     - **ID:** `myCustomRole` (Note: Ensure the ID is properly formatted for programmatic reference).
     - **Launch stage:** Set to **Alpha** (since it's an initial test version).
   - Click **+ ADD PERMISSIONS**.
   - Filter permissions by service (e.g., choose **App Engine Admin** or filter by specific strings).
   - Select a subset of desired permissions (e.g., 5 specific permissions) and click **Add**.
   - Click **Create** to finalize and provision the custom role.

---

### 5. Visual Representations

#### Storage Roles Permission Boundary Comparison
```text
  +-------------------------------------------------------------+
  |                   STORAGE ADMIN (`storage.admin`)           |
  |   - Manages Buckets (Create, Delete, Update, Get, List)     |
  |   - Manages Objects (Create, Delete, Update, Get, List)     |
  +-------------------------------------------------------------+
                                |
                                v
  +-------------------------------------------------------------+
  |              STORAGE OBJECT ADMIN (`storage.objectAdmin`)   |
  |   - NO Bucket Management Access                             |
  |   - Manages Objects (Create, Delete, Update, Get, List)     |
  +-------------------------------------------------------------+
                                |
                                v
  +-------------------------------------------------------------+
  |          STORAGE OBJECT CREATOR (`storage.objectCreator`)   |
  |   - NO Bucket Management Access                             |
  |   - Restricted strictly to Creating/Uploading Objects       |
  +-------------------------------------------------------------+
                                |
                                v
  +-------------------------------------------------------------+
  |           STORAGE OBJECT VIEWER (`storage.objectViewer`)    |
  |   - NO Bucket Management Access                             |
  |   - Restricted strictly to Get and List (Read-Only Objects) |
  +-------------------------------------------------------------+
```

---

### 6. Mental Models & Analogies

#### The Restaurant Kitchen Analogy
- **Viewer Role:** A food critic sitting in the dining room with a clipboard who can look at everything happening (`get`/`list`), but cannot cook, clean, or move any equipment.
- **Storage Admin:** The Executive Head Chef who owns the entire kitchen space, can build/demolish prep stations (buckets), and cook or discard any ingredients (objects).
- **Storage Object Admin:** A line cook who can prep, modify, and discard ingredients at any workstation, but lacks the authority to remodel the kitchen layout or install new ovens (buckets).
- **Storage Object Creator:** A delivery assistant whose sole job is to bring boxes of ingredients inside and drop them off on the counter, but cannot open, move, or throw away existing items.

---

### 7. Step-by-Step Workflows / Comparison Tables

#### Storage Roles vs. Capabilities Reference Table
| Role Name | Bucket Creation/Deletion | Bucket IAM Management | Object Creation | Object Deletion | Object Reading (Get/List) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`storage.admin`** | Yes | Yes | Yes | Yes | Yes |
| **`storage.objectAdmin`** | No | No | Yes | Yes | Yes |
| **`storage.objectCreator`** | No | No | Yes | No | No |
| **`storage.objectViewer`** | No | No | No | No | Yes |

---

### 8. Common Mistakes & Exam Traps
- **Trap:** Assuming that a custom role created in `Alpha` stage cannot be updated.
  - *Correction:* Custom roles can be edited, updated, and promoted through launch stages (`Alpha` -> `Beta` -> `GA`) at any time after creation.
- **Trap:** Believing `storage.objectCreator` can delete old logs or data files to free up bucket space.
  - *Correction:* `storage.objectCreator` can *only* write/create objects. Any attempt to delete or overwrite will fail due to lack of `storage.objects.delete` permission.

---

### 9. Certification Practice Questions

#### Question 1
An application running on GCP needs permission to upload temporary build artifacts into a specific Cloud Storage bucket, but security guidelines dictate that the application must *never* be allowed to delete or overwrite any existing objects or buckets. Which predefined role should you assign?
- A. `roles/storage.admin`
- B. `roles/storage.objectAdmin`
- C. `roles/storage.objectCreator`
- D. `roles/storage.objectViewer`

**Correct Answer:** **C**  
**Explanation:** `roles/storage.objectCreator` grants permissions exclusively to create new objects (`storage.objects.create`), preventing deletion or bucket-level modifications.  
**Incorrect Answers:**  
A and B allow object deletion.  
D is read-only.  
**Exam Takeaway:** When a workload only needs to write or upload files without deletion privileges, use `storage.objectCreator`.

---

#### Question 2
When inspecting the internal permission list of almost any predefined or custom GCP role in the Console, which two baseline permissions are almost universally present?
- A. `resourcemanager.projects.get` and `resourcemanager.projects.list`
- B. `compute.instances.create` and `compute.instances.delete`
- C. `storage.buckets.create` and `storage.buckets.delete`
- D. `iam.roles.create` and `iam.roles.delete`

**Correct Answer:** **A**  
**Explanation:** Project metadata visibility permissions (`resourcemanager.projects.get` and `resourcemanager.projects.list`) are bundled into almost all roles so users can view project listings in the console.  
**Incorrect Answers:** B, C, and D grant high-level service write/admin capabilities that are restricted to specific administrative roles.  
**Exam Takeaway:** Project list and get permissions are standard baseline elements for console navigation.

---

#### Question 3
You are designing a custom role for a specialized testing framework. Which launch stage should you select while you are actively experimenting with and modifying the role's permissions?
- A. General Availability (GA)
- B. Alpha
- C. Deprecated
- D. Production Locked

**Correct Answer:** **B**  
**Explanation:** The `Alpha` stage is intended for early testing, experimentation, and iterative adjustments on custom roles.  
**Incorrect Answers:**  
A (`GA`) is for stable, production-ready roles.  
C (`Deprecated`) flags roles for future removal.  
D is not a standard GCP launch stage.  
**Exam Takeaway:** Use `Alpha` for new custom roles during the initial testing phase.

---

#### Question 4
A user assigned the `roles/storage.objectAdmin` role tries to delete an empty Cloud Storage bucket and receives an error message stating they lack permissions. What is the cause of this error?
- A. `storage.objectAdmin` only controls objects within buckets, not the buckets themselves.
- B. Deleting an empty bucket requires the `roles/viewer` role.
- C. Cloud Storage buckets cannot be deleted by anyone except the Billing Account Administrator.
- D. Buckets can only be deleted via the `gcloud` command-line interface, not the Console.

**Correct Answer:** **A**  
**Explanation:** `roles/storage.objectAdmin` provides full administrative control over objects, but contains zero permissions for bucket management (which requires `storage.admin` or project-level editing rights).  
**Incorrect Answers:** B, C, and D are factually incorrect.  
**Exam Takeaway:** Bucket-level creation and deletion require `storage.admin` or higher, whereas object admin roles are scoped strictly to objects.

---

#### Question 5
Which role contains the highest number of overall permissions across a Google Cloud project, encompassing thousands of resource actions as well as billing and IAM policy management?
- A. `roles/viewer`
- B. `roles/editor`
- C. `roles/owner`
- D. `roles/storage.admin`

**Correct Answer:** **C**  
**Explanation:** The `roles/owner` role contains the complete set of permissions for a project (approx. 3,800+), including billing management and IAM policy configuration.  
**Incorrect Answers:**  
A and B have fewer permissions and lack IAM/billing administration rights.  
D is scoped strictly to Cloud Storage.  
**Exam Takeaway:** Owner is the most powerful basic role available at the project level.

---

#### Question 6
When creating a custom role, why is it important to assign a clean, well-formatted Role ID?
- A. Because the Role ID determines how fast the role executes API calls.
- B. Because the Role ID is the programmatic handle used to bind the role to members in IAM policies.
- C. Because GCP automatically bills you based on the length of the Role ID string.
- D. Because custom role IDs cannot contain alphanumeric characters.

**Correct Answer:** **B**  
**Explanation:** The Role ID serves as the unique identifier required when referencing and assigning the custom role inside IAM policy bindings via CLI, scripts, or APIs.  
**Incorrect Answers:** A, C, and D are false statements.  
**Exam Takeaway:** Role IDs function as programmatic handles for custom role policy bindings.

---

#### Question 7
An internal security review reveals that a junior operator has `roles/viewer` assigned to their account. What level of access does this grant them over a newly created Compute Engine virtual machine instance in the project?
- A. Full administrative control (they can start, stop, and delete the VM).
- B. Read-only visibility (they can inspect the VM configuration, status, and metadata, but cannot modify or control it).
- C. Zero visibility (Viewer roles only apply to Cloud Storage, not Compute Engine).
- D. Root shell access via SSH.

**Correct Answer:** **B**  
**Explanation:** The primitive `roles/viewer` role grants read-only access (`get` and `list` actions) across all resources in the project, including Compute Engine instances.  
**Incorrect Answers:** A and D require write or administrative roles (`editor` or `owner`). C is false because viewer applies project-wide.  
**Exam Takeaway:** Viewer = read-only visibility across all project services.

---

#### Question 8
If you need to grant a user permission to view objects inside a Cloud Storage bucket without allowing them to modify, upload, or delete any data, which role should you choose?
- A. `roles/storage.objectViewer`
- B. `roles/storage.objectCreator`
- C. `roles/storage.objectAdmin`
- D. `roles/editor`

**Correct Answer:** **A**  
**Explanation:** `roles/storage.objectViewer` grants read-only access (`get` and `list`) strictly to objects.  
**Incorrect Answers:** B allows creation, C allows full object management, and D grants project-wide editing rights.  
**Exam Takeaway:** For read-only object inspection, use `storage.objectViewer`.

---

#### Question 9
Which of the following actions can a user with the `roles/editor` role perform that a user with the `roles/viewer` role *cannot* perform?
- A. List the names of virtual machine instances in the project.
- B. View the configuration settings of a BigQuery dataset.
- C. Create, modify, and delete Compute Engine virtual machine instances.
- D. View project logs in Cloud Logging.

**Correct Answer:** **C**  
**Explanation:** The Editor role permits resource creation, modification, and deletion across project services, whereas the Viewer role is restricted strictly to read-only inspection.  
**Incorrect Answers:** A, B, and D can all be performed by a Viewer role.  
**Exam Takeaway:** Editor adds mutation and creation capabilities on top of viewer permissions.

---

#### Question 10
What happens to a custom role if its launch stage is explicitly set to `Disabled`?
- A. The role is permanently deleted from the Google Cloud project along with its permission history.
- B. The role is temporarily turned off, preventing its use in new or active policy bindings without being permanently erased.
- C. The role automatically converts into a primitive Viewer role.
- D. All users assigned to the role are instantly promoted to Project Owners.

**Correct Answer:** **B**  
**Explanation:** Setting a custom role launch stage to `Disabled` deactivates the role without deleting it, allowing administrators to suspend use temporarily.  
**Incorrect Answers:** A, C, and D are incorrect descriptions of the disabled launch stage.  
**Exam Takeaway:** Disabled launch stage pauses role functionality without deleting the custom role configuration.