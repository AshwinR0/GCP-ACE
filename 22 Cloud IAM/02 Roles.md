# GCP Associate Cloud Engineer (ACE) Study Notes

## Chapter: IAM Identity and Access Management (Part 2 - Deep Dive into Roles)

---

### 1. Overview
Building upon the core IAM concepts, this section explores **Roles** in depth. A role in Google Cloud is a collection of permissions that dictate what actions a member can perform on specific resources. 

GCP provides three distinct types of roles to accommodate various security and administrative needs:
1. **Basic / Primitive Roles:** Legacy, broad-scoped roles (Owner, Editor, Viewer).
2. **Predefined Roles:** Granular, Google-managed roles tailored to specific services and job functions.
3. **Custom Roles:** User-defined roles created when predefined options do not meet specific organizational requirements.

---

### 2. ACE Exam Focus & Common Exam Traps
- **Exam Importance:** **HIGH**. Scenario questions frequently test your ability to select the correct role based on the **Principle of Least Privilege**.
- **High-Priority Topics:**
  - Knowing when *not* to use basic roles (avoid in production due to overly broad scope).
  - Understanding the hierarchy and permissions difference within service-specific predefined roles (e.g., `Storage Admin` vs. `Storage Object Admin` vs. `Storage Object Creator` vs. `Storage Object Viewer`).
  - Recognizing the purpose and constraints of Custom Roles.
- **Common Exam Traps:** 
  - Choosing `Owner` or `Editor` when a user only needs to view or manage a single service (violates least privilege).
  - Confusing `Storage Admin` (can manage both buckets and objects) with `Storage Object Admin` (can manage objects only, not buckets).

---

### 3. Detailed Concept Explanation

#### A. Basic / Primitive Roles
Basic roles predate Cloud IAM and apply broadly across an entire GCP project. 
- **Viewer (`roles/viewer`):** 
  - Provides read-only access (`get` and `list` actions) across *all* resources in a project (approx. 1,600+ permissions). 
  - *Use case:* General code/resource inspection or compliance auditing.
- **Editor (`roles/editor`):** 
  - Includes all Viewer permissions plus actions that modify state or create/delete resources across *all* services in a project.
  - *Limitation:* Cannot manage IAM policies or manage billing accounts.
- **Owner (`roles/owner`):** 
  - Includes all Editor permissions plus the ability to manage IAM policies (assign roles, create custom roles), manage billing accounts, and delete the project itself (approx. 3,800+ permissions).
- **Production Warning:** Basic roles are **not recommended for production environments** because they violate the Principle of Least Privilege by granting project-wide access.

#### B. Predefined Roles
Predefined roles are fine-grained roles managed and updated automatically by Google. They provide access tailored to specific service actions.
- **Cloud Storage Examples:**
  - **Storage Admin (`roles/storage.admin`):** Full control over buckets and objects (create, delete, update, get, list, set/get IAM policies).
  - **Storage Object Admin (`roles/storage.objectAdmin`):** Full control over *objects* only (cannot create, delete, or manage buckets themselves).
  - **Storage Object Creator (`roles/storage.objectCreator`):** Can upload/create new objects, but cannot delete or modify existing ones.
  - **Storage Object Viewer (`roles/storage.objectViewer`):** Read-only access to objects (`get` and `list`).
- *Note:* Most Google-managed roles include baseline project-level visibility permissions like `resourcemanager.projects.get` and `resourcemanager.projects.list`.

#### C. Custom Roles
When predefined roles do not provide the exact combination of permissions needed, you can create **Custom Roles**.
- User-managed and user-defined.
- Can bundle specific permissions across multiple services or isolate precise actions.
- Include **Launch Stages** to track role maturity: `Alpha`, `Beta`, `General Availability (GA)`, and `Disabled`.
- *Best Practice:* Always prefer Google-managed Predefined roles over Custom roles when possible, as Predefined roles are automatically maintained and updated by Google when new features roll out.

---

### 4. Practical Setup & Implementation Guide

#### Navigating and Creating Custom Roles via GCP Console
1. **Navigation Path:** Open the GCP Console search bar, type **Roles**, and select **IAM & Admin > Roles**.
2. **Exploring Existing Roles:**
   - Use the filter box to search by name (e.g., `roles/viewer`, `roles/owner`, `roles/storage.admin`).
   - Click on any role to inspect its granular permissions (e.g., viewing the ~1,661 permissions of a Viewer or filtering by specific services like App Engine).
3. **Creating a Custom Role:**
   - Click **+ CREATE ROLE**.
   - Provide a descriptive **Title** and a unique **Role ID** (used for binding policies via CLI/API).
   - Set the **Launch stage** (e.g., `Alpha` for testing new configurations, progressing to `GA` for production use).
   - Click **+ ADD PERMISSIONS**.
   - Filter permissions by service (e.g., App Engine Admin) or specific permission string.
   - Select the target permissions and click **Add**.
   - Click **Create** to save the custom role.

---

### 5. Visual Representations

#### Role Hierarchy & Scope Comparison
```text
  +-----------------------------------------------------------+
  |                   BASIC / PRIMITIVE ROLES                 |
  |   (Owner / Editor / Viewer - Entire Project Scope)        |
  +-----------------------------------------------------------+
                                |
                                v
  +-----------------------------------------------------------+
  |                     PREDEFINED ROLES                      |
  |     (Service-Specific, e.g., Storage Admin / BigQuery)    |
  +-----------------------------------------------------------+
                                |
                                v
  +-----------------------------------------------------------+
  |                       CUSTOM ROLES                        |
  |       (User-Defined Bundles for Granular Use Cases)       |
  +-----------------------------------------------------------+
```

---

### 6. Mental Models & Analogies

#### The Building Access Analogy
- **Basic Roles:** Giving someone the Master Key to the entire corporate building (Owner/Editor) vs. a universal viewing badge that opens every single room's glass window to look inside, but touches nothing (Viewer).
- **Predefined Roles:** Giving someone a specific department keycard—for example, the "Archive Room Manager" keycard allows them to handle all boxes in the archive room, while the "Archive Intern" keycard only lets them drop off new boxes without moving or deleting old ones.
- **Custom Roles:** Customizing a unique keychain that takes specific keys from three different department lockers because standard keychains didn't fit a multi-department project requirement.

---

### 7. Step-by-Step Workflows / Comparison Tables

#### Comparison of Role Types
| Feature | Basic (Primitive) Roles | Predefined Roles | Custom Roles |
| :--- | :--- | :--- | :--- |
| **Scope** | Project-wide | Service-specific | User-defined |
| **Granularity** | Low (Very broad access) | High (Fine-grained) | Fully customizable |
| **Management** | Google-managed (Legacy) | Google-managed | User-managed |
| **Production Use** | Not recommended (except rare audit views) | Highly recommended | Used only when predefined roles fail |
| **Updates** | Static | Automatically updated by Google | Manually updated by user |

---

### 8. Common Mistakes & Exam Traps
- **Trap:** Assuming `Storage Object Admin` can create new storage buckets.
  - *Correction:* `Storage Object Admin` only manages *objects* inside buckets. To create or delete the buckets themselves, the user needs `Storage Admin` or project-level editing rights.
- **Trap:** Using custom roles as a default choice.
  - *Exam Tip:* If an exam question asks how to give a developer restricted access, always look for an existing **Predefined Role** first before selecting a Custom Role option. Google prefers predefined roles because they require zero maintenance overhead.

---

### 9. Certification Practice Questions

#### Question 1
A junior database administrator needs permission to create, delete, and modify Cloud Storage buckets as well as manage all the objects inside them. Which predefined role should you assign to satisfy the Principle of Least Privilege?
- A. `roles/storage.objectAdmin`
- B. `roles/storage.admin`
- C. `roles/storage.objectViewer`
- D. `roles/editor`

**Correct Answer:** **B**  
**Explanation:** `roles/storage.admin` provides full control over both Cloud Storage buckets and the objects within them.  
**Incorrect Answers:**  
A (`storage.objectAdmin`) only manages objects, not buckets.  
C (`storage.objectViewer`) is read-only.  
D (`roles/editor` or basic editor) grants project-wide write permissions to all services, violating least privilege.  
**Exam Takeaway:** To manage both buckets and objects in Cloud Storage, look for `Storage Admin`.

---

#### Question 2
Why are basic (primitive) roles like Viewer, Editor, and Owner generally discouraged for daily use in production environments?
- A. They automatically expire after 90 days for security compliance.
- B. They grant permissions across the entire project rather than being restricted to specific services, violating the Principle of Least Privilege.
- C. They can only be assigned to service accounts and cannot be bound to human users.
- D. They do not support IAM conditions.

**Correct Answer:** **B**  
**Explanation:** Basic roles have a project-wide scope that grants immense power over every single service in the project, which breaches security best practices (Principle of Least Privilege).  
**Incorrect Answers:** A, C, and D are factually incorrect statements regarding basic roles.  
**Exam Takeaway:** Never use basic roles in production unless there is an explicit need for broad, project-wide auditing.

---

#### Question 3
An auditor requires read-only visibility into all resources across an entire Google Cloud project. Which role fits this requirement best?
- A. `roles/viewer`
- B. `roles/storage.objectViewer`
- C. `roles/editor`
- D. `roles/owner`

**Correct Answer:** **A**  
**Explanation:** The primitive `roles/viewer` role grants read-only (`get` and `list`) access across all resources present in the GCP project.  
**Incorrect Answers:**  
B is restricted only to storage objects.  
C and D grant high-level write and administrative privileges.  
**Exam Takeaway:** For project-wide read-only inspection, `roles/viewer` is the appropriate choice.

---

#### Question 4
You have created a new custom role to bundle specific permissions for a specialized internal application. What is the purpose of setting a role's "Launch Stage" to `Alpha`?
- A. It forces the role to execute tasks faster using accelerated compute instances.
- B. It flags the role stage for testing and evaluation before promoting it to General Availability (GA).
- C. It automatically grants the role public access via `allUsers`.
- D. It restricts the role so it can only be applied to App Engine.

**Correct Answer:** **B**  
**Explanation:** Launch stages (`Alpha`, `Beta`, `GA`, `Disabled`) help track the lifecycle and maturity of user-defined custom roles during creation and deployment.  
**Incorrect Answers:** A, C, and D are unrelated to custom role launch stages.  
**Exam Takeaway:** Custom role launch stages represent maturity levels (Alpha -> Beta -> GA).

---

#### Question 5
A member assigned the `roles/storage.objectCreator` role attempts to delete an outdated object from a production storage bucket and receives a permission denied error. Why did this happen?
- A. The user's basic role overrides the predefined role.
- B. The `storage.objectCreator` role only provides permissions to create new objects, not to delete or modify existing ones.
- C. Object deletion requires project owner privileges.
- D. The bucket has IAM Conditions enabled blocking all deletions.

**Correct Answer:** **B**  
**Explanation:** `roles/storage.objectCreator` only permits object creation (`storage.objects.create`). It lacks the deletion permission (`storage.objects.delete`), which belongs to roles like `storage.objectAdmin` or `storage.admin`.  
**Incorrect Answers:** A, C, and D introduce scenarios not supported by the role's actual permission layout.  
**Exam Takeaway:** Understand the exact capability limits of fine-grained storage roles (Creator = create only; Viewer = read only; Object Admin = manage objects; Admin = buckets + objects).

---

#### Question 6
Which of the following responsibilities is unique to the `Owner` primitive role compared to the `Editor` role?
- A. Creating and managing Compute Engine virtual machines.
- B. Deploying code to App Engine applications.
- C. Managing IAM policies and billing accounts.
- D. Reading data from Cloud Storage buckets.

**Correct Answer:** **C**  
**Explanation:** The Owner role contains all Editor permissions plus the capability to manage IAM policies (assign roles, build custom roles) and manage project billing accounts.  
**Incorrect Answers:** A, B, and D can all be performed by the Editor role as well.  
**Exam Takeaway:** Owner = Editor + IAM Management + Billing Management.

---

#### Question 7
When should you choose to create a **Custom Role** instead of using a **Predefined Role**?
- A. Whenever you want Google to automatically update the role's permissions as new services launch.
- B. When existing predefined roles do not fulfill your precise permission requirements and you need a custom bundle of specific permissions.
- C. To bypass project-level security policies entirely.
- D. Custom roles should always be preferred over predefined roles as a default standard.

**Correct Answer:** **B**  
**Explanation:** Custom roles are built specifically for situations where pre-packaged Google-managed predefined roles do not align with your exact permission matrix.  
**Incorrect Answers:** A describes predefined roles. C is false. D is a bad practice (predefined roles are preferred over custom roles).  
**Exam Takeaway:** Use predefined roles first; build custom roles only when predefined options don't fit.

---

#### Question 8
Almost every predefined and custom GCP role includes baseline permissions to perform which of the following actions?
- A. Delete the entire GCP project.
- B. Modify the master billing profile.
- C. View and list projects (`resourcemanager.projects.get` and `resourcemanager.projects.list`).
- D. Create new organization nodes.

**Correct Answer:** **C**  
**Explanation:** Most roles include basic project visibility permissions (`resourcemanager.projects.get` and `resourcemanager.projects.list`) so users can see the projects they have access to within the console.  
**Incorrect Answers:** A, B, and D require high administrative permissions (Owner or Organization Admin).  
**Exam Takeaway:** Project list/get permissions are standard baseline components across most roles for console usability.

---

#### Question 9
An administrator wants to give a contractor permission to view and modify objects within a specific Cloud Storage bucket, but explicitly wants to prevent them from creating or deleting whole buckets. Which role combination or single role is ideal?
- A. `roles/owner`
- B. `roles/storage.admin`
- C. `roles/storage.objectAdmin`
- D. `roles/viewer`

**Correct Answer:** **C**  
**Explanation:** `roles/storage.objectAdmin` allows full management of objects inside a bucket without granting permission to create or delete the buckets themselves.  
**Incorrect Answers:**  
A gives total project ownership.  
B (`storage.admin`) allows bucket creation/deletion.  
D (`viewer`) is read-only.  
**Exam Takeaway:** Match object-level administration requirements directly to `storage.objectAdmin`.

---

#### Question 10
Which entity is responsible for creating, updating, and maintaining **Predefined Roles** in Google Cloud?
- A. The individual project owner.
- B. Google.
- C. The organization security lead via a deployment script.
- D. Community contributors through open-source templates.

**Correct Answer:** **B**  
**Explanation:** Predefined roles are curated, managed, and updated directly by Google as new cloud capabilities and permissions are introduced.  
**Incorrect Answers:** A and C refer to custom or project-level configuration actions.  
**Exam Takeaway:** Google manages Predefined roles; you manage Custom roles.