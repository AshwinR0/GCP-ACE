# GCP Associate Cloud Engineer (ACE) Study Notes

## Chapter: IAM Identity and Access Management (Part 4 - Deep Dive into Roles & Console Navigation)

---

### 1. Overview
This section continues the exploration of Google Cloud **Identity and Access Management (IAM)** with a specific focus on navigating the GCP console to manage **Roles**. It details how Google-managed roles (both primitive and predefined) are structured, how service-specific storage roles differ in scope, and how administrators can build and transition user-defined **Custom Roles** using launch stages.

---

### 2. ACE Exam Focus & Common Exam Traps
- **Exam Importance:** **HIGH**. The ACE exam heavily tests your understanding of role granularity and the principle of least privilege.
- **High-Priority Topics:**
  - **Primitive vs. Predefined Location:** In the GCP Console, both basic/primitive roles (`viewer`, `editor`, `owner`) and predefined roles are grouped under the **Predefined tab** and managed entirely by Google.
  - **Storage Scope Distinctions:**
    - `roles/storage.admin`: Full control of *both* buckets and objects.
    - `roles/storage.objectAdmin`: Full control over *objects only* (cannot create or delete buckets).
  - **Custom Role Launch Stages:** Understand the progression lifecycle (`Alpha` -> `Beta` -> `General Availability (GA)` -> `Disabled`).
- **Common Exam Traps:**
  - Assuming custom roles are managed by Google. (Trap! Custom roles are user-created and user-managed).
  - Confusing bucket management capabilities with object management roles.

---

### 3. Detailed Concept Explanation

#### A. Role Categories in the GCP Console
1. **Predefined Tab (Google-Managed):**
   - Contains both **Basic/Primitive roles** (`roles/viewer`, `roles/editor`, `roles/owner`) and **Predefined roles** (service-specific bundles).
   - Maintained automatically by Google; when new features or permissions are introduced, Google updates these roles automatically.
2. **Custom Roles Tab (User-Managed):**
   - Created, modified, and maintained entirely by your organization.

#### B. Deep Dive into Specific Roles
- **Viewer Role (`roles/viewer`):** 
  - Contains read-only actions (`get` and `list` permissions across components like `compute.images` and `compute.instances`).
- **Editor Role (`roles/editor`):** 
  - Extends viewing capabilities to include resource creation, modification, and deletion (e.g., attaching disks, creating/deleting VM instances).
- **Owner Role (`roles/owner`):** 
  - Provides full administrative access across nearly all Google Cloud resources and IAM settings.
- **Cloud Storage Predefined Roles:**
  - **Storage Admin (`roles/storage.admin`):** Full control over buckets (`storage.buckets`, `storage.folders`) and objects (`storage.objects`).
  - **Storage Object Admin (`roles/storage.objectAdmin`):** Full control over objects only, omitting bucket-level creation or structural changes.

#### C. Custom Roles & Launch Stages
When defining custom roles, you configure:
- **Title and ID:** A readable title and a unique identifier handle.
- **Launch Stages:**
  - `Alpha`: Used during initial creation and iterative testing.
  - `Beta`: Used for broader internal validation.
  - `General Availability (GA)`: Production-ready.
  - `Disabled`: Temporarily turns off the role without deleting it.
- **Adding Permissions:** You can add permissions explicitly by searching individual actions (e.g., `compute.instances.delete`) or by cloning/filtering permissions from existing predefined roles (like `storage.objectAdmin`).

---

### 4. Practical Setup & Implementation Guide

#### Navigating and Creating Custom Roles via Console
1. **Navigation Path:** Open the GCP Console, search for **IAM & Admin**, and click **Roles**.
2. **Reviewing Google-Managed Roles:**
   - Look under the default view to see Google-managed primitive and predefined roles.
   - Inspect specific permissions by clicking on roles like `roles/viewer`, `roles/editor`, `roles/owner`, or storage roles.
3. **Creating a Custom Role:**
   - Click **+ CREATE ROLE**.
   - Enter a **Title** (e.g., `My First Custom Role`) and a unique **Role ID** (e.g., `MyFirstCustomRole`).
   - Select the initial **Launch stage** (e.g., `Alpha`).
   - Choose a method to add permissions:
     - Click **+ ADD PERMISSIONS** to search and select individual fine-grained permissions (e.g., `compute.instances.delete`).
     - Alternatively, filter by an existing reference role (e.g., `storage object admin`) to pull a subset of its permissions.
   - Click **Create** to save your custom role.

---

### 5. Visual Representations

#### Custom Role Lifecycle & Launch Stages
```text
  +------------------+     Promote      +------------------+     Promote      +------------------+
  |      ALPHA       | -------------> |       BETA       | -------------> |       GA         |
  | (Initial Testing)|                | (Internal Review)|                | (Production Use) |
  +------------------+                +------------------+                +------------------+
           |                                                                       |
           +--------------------------+--------------------------------------------+
                                      |
                                      v
                             +------------------+
                             |     DISABLED     | (Temporarily Inactive)
                             +------------------+
```

---

### 6. Mental Models & Analogies

#### The Software Development Lifecycle Analogy
- **Launch Stages of a Custom Role:** Just like software moves from an internal prototype branch (`Alpha`) to a staging environment (`Beta`) and finally to production release (`General Availability`), custom roles move through these exact phases to ensure security stability before live rollout. If a bug is found, it can be toggled to `Disabled` just like rolling back a feature flag.

---

### 7. Step-by-Step Workflows / Comparison Tables

#### Comparison: Google-Managed vs. User-Managed Roles
| Feature | Google-Managed (Basic & Predefined) | User-Managed (Custom Roles) |
| :--- | :--- | :--- |
| **Creator** | Google | You / Your Organization |
| **Updates** | Automatically updated by Google | Manually maintained and updated by you |
| **Console Location** | Predefined tab | Custom Roles tab |
| **Flexibility** | Fixed permission bundles | Fully customizable permission sets |

---

### 8. Common Mistakes & Exam Traps
- **Trap:** Assuming basic roles are listed under a separate tab from predefined roles in the console.
  - *Correction:* Both basic/primitive roles and predefined roles live under the same Google-managed **Predefined** tab.
- **Trap:** Believing that changing a custom role's launch stage to `Disabled` deletes its configurations.
  - *Correction:* Disabling a custom role only suspends its use; the role configuration and its assigned permissions remain intact for future re-activation.

---

### 9. Certification Practice Questions

#### Question 1
Where are basic (primitive) roles located within the IAM Roles section of the Google Cloud Console?
- A. Under a separate tab named "Basic Roles".
- B. Under the "Predefined" tab alongside Google-managed service-specific roles.
- C. Under the "Custom Roles" tab.
- D. In a standalone billing management portal.

**Correct Answer:** **B**  
**Explanation:** In the GCP Console, both primitive roles (`viewer`, `editor`, `owner`) and predefined roles are grouped together under the Google-managed **Predefined** tab.  
**Incorrect Answers:** A, C, and D reference non-existent console tab structures for basic roles.  
**Exam Takeaway:** Primitive and predefined roles share the Google-managed Predefined tab in the GCP Console.

---

#### Question 2
When creating a custom role, what is the recommended launch stage to select when you are initially building, testing, and iterating on the role's permissions?
- A. General Availability (GA)
- B. Alpha
- C. Production Locked
- D. Stable

**Correct Answer:** **B**  
**Explanation:** `Alpha` is the designated launch stage for early-stage testing and iterative adjustments of custom roles.  
**Incorrect Answers:** A is for stable production releases. C and D are not standard GCP custom role launch stages.  
**Exam Takeaway:** Use `Alpha` for new custom roles during initial test cycles.

---

#### Question 3
An application requires the ability to create new objects inside a Cloud Storage bucket, but administrators want to ensure the application cannot delete any objects or manage the storage buckets themselves. Which predefined role should you assign?
- A. `roles/storage.admin`
- B. `roles/storage.objectAdmin`
- C. `roles/storage.objectCreator`
- D. `roles/storage.objectViewer`

**Correct Answer:** **C**  
**Explanation:** `roles/storage.objectCreator` allows creating new objects (`storage.objects.create`) without granting permissions to delete objects or manage buckets.  
**Incorrect Answers:** A and B permit object deletion. D is read-only.  
**Exam Takeaway:** For write-only/upload access without deletion rights, select `storage.objectCreator`.

---

#### Question 4
What occurs when an administrator changes the launch stage of a custom role from `Alpha` to `Disabled`?
- A. The custom role is permanently wiped from the project.
- B. The role is temporarily deactivated so it cannot be used in new or active policy bindings, but its configuration is preserved.
- C. The role automatically converts into a primitive Viewer role.
- D. All users assigned to this role are downgraded to basic viewers.

**Correct Answer:** **B**  
**Explanation:** The `Disabled` launch stage temporarily suspends a custom role's usability without erasing its configuration settings.  
**Incorrect Answers:** A, C, and D are incorrect descriptions of the disabled launch stage.  
**Exam Takeaway:** Disabling a custom role pauses its use without permanent deletion.

---

#### Question 5
Which primitive role grants permissions to view, create, update, and delete most Google Cloud resources, but lacks permissions to manage billing accounts or IAM policies?
- A. `roles/viewer`
- B. `roles/editor`
- C. `roles/owner`
- D. `roles/storage.admin`

**Correct Answer:** **B**  
**Explanation:** The primitive `roles/editor` role allows viewing, creating, updating, and deleting project resources, but does not grant IAM or billing management rights (which require Owner).  
**Incorrect Answers:** A is read-only. C grants full IAM and billing control. D is service-specific.  
**Exam Takeaway:** Editor = full resource mutation rights without IAM/billing management.

---

#### Question 6
When creating a custom role, how can you efficiently assign permissions without manually searching for every single action?
- A. By importing permissions from an existing reference role (such as `storage.objectAdmin`) or filtering by service.
- B. By uploading a local binary file directly into the IAM policy engine.
- C. By writing a custom Python script that bypasses the console interface.
- D. By assigning the Project Owner role first and letting GCP auto-filter it.

**Correct Answer:** **A**  
**Explanation:** The GCP Console allows you to filter and copy permission sets from existing predefined roles (like `storage.objectAdmin`) or search by service when building custom roles.  
**Incorrect Answers:** B, C, and D do not describe standard console custom role creation workflows.  
**Exam Takeaway:** You can streamline custom role creation by filtering and selecting permissions from existing Google-managed roles.

---

#### Question 7
Which statement is true regarding Google-managed predefined roles?
- A. Users can edit the internal permission list of a predefined role directly in the console.
- B. Google automatically updates predefined roles when new features or services are introduced.
- C. Predefined roles expire automatically every 365 days.
- D. Predefined roles can only be applied at the organization folder level.

**Correct Answer:** **B**  
- **Explanation:** Google maintains predefined roles, automatically updating their permission layouts as new cloud features and services roll out.  
- **Incorrect Answers:** A is false (predefined roles are read-only; use custom roles if changes are needed). C and D are false.  
- **Exam Takeaway:** Google manages and automatically updates Predefined roles.

---

#### Question 8
A user has been assigned `roles/viewer` on a Google Cloud project. Which of the following actions can they successfully perform?
- A. Delete a Compute Engine virtual machine instance.
- B. Create a new Cloud Storage bucket.
- C. List and inspect the configuration details of existing virtual machine instances.
- D. Assign IAM roles to other team members.

**Correct Answer:** **C**  
**Explanation:** The Viewer role provides read-only access (`get` and `list` permissions) across project resources like virtual machines.  
**Incorrect Answers:** A and B require editor/admin rights. D requires owner or security admin rights.  
**Exam Takeaway:** Viewer enables read-only inspection (`get`/`list`) across project resources.

---

#### Question 9
What is the primary difference between `roles/storage.admin` and `roles/storage.objectAdmin`?
- A. `storage.admin` only allows reading objects, while `storage.objectAdmin` allows full bucket control.
- B. `storage.admin` grants full control of both buckets and objects, whereas `storage.objectAdmin` grants full control over objects only without bucket management capabilities.
- C. `storage.admin` is a custom role, while `storage.objectAdmin` is a basic primitive role.
- D. There is no functional difference between the two roles.

**Correct Answer:** **B**  
**Explanation:** `storage.admin` covers buckets and objects, while `storage.objectAdmin` is scoped strictly to objects.  
**Incorrect Answers:** A, C, and D are factually incorrect.  
**Exam Takeaway:** Check storage role scopes carefully: Admin = buckets + objects; Object Admin = objects only.

---

#### Question 10
If an organization requires a role with a precise combination of permissions that does not exist among Google's predefined offerings, what is the appropriate solution?
- A. Use the primitive Owner role.
- B. Create a User-Managed Custom Role.
- C. Share root credentials across the engineering team.
- D. Contact Google Support to rewrite a predefined role.

**Correct Answer:** **B**  
**Explanation:** When predefined roles fail to meet exact organizational requirements, you build a custom role tailored to your specific permission criteria.  
**Incorrect Answers:** A violates least privilege. C is a critical security violation. D is unnecessary since custom roles are user-managed.  
**Exam Takeaway:** Use Custom Roles when predefined options do not meet specific access requirements.