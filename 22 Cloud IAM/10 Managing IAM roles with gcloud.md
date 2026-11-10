# GCP Associate Cloud Engineer (ACE) Study Notes

## Chapter: Managing IAM and Projects via `gcloud` CLI

---

### 1. Overview
This chapter explores managing Google Cloud **Identity and Access Management (IAM)** and project configurations programmatically using the **`gcloud` CLI**. It covers project identification, authentication routines, inspecting and updating IAM policies and bindings, managing projects, and working with IAM roles (`describe`, `create`, and `copy`).

---

### 2. ACE Exam Focus & Common Exam Traps
- **Exam Importance:** **HIGH**. The ACE exam features multiple questions requiring correct `gcloud` syntax for managing identity states, authenticating, and updating project-level IAM bindings.
- **High-Priority Commands:**
  - **Project Configuration:** `gcloud config set project [PROJECT_ID]`
  - **Project Info:** `gcloud compute project-info describe`
  - **Authentication:** `gcloud auth login`, `gcloud auth list`, `gcloud auth revoke`
  - **IAM Policies:** `gcloud projects get-iam-policy [PROJECT_ID]`
  - **IAM Bindings:** 
    - `gcloud projects add-iam-policy-binding [PROJECT_ID] --member="..." --role="..."`
    - `gcloud projects remove-iam-policy-binding [PROJECT_ID] --member="..." --role="..."`
  - **IAM Roles:** `gcloud iam roles describe`, `gcloud iam roles create`, `gcloud iam roles copy`
- **Common Exam Traps:**
  - Forgetting to pass the `--member` or `--role` flag when running `add-iam-policy-binding`, which results in a syntax error.
  - Confusing project-level policy additions with bucket-level or resource-level IAM commands.

---

### 3. Detailed Concept Explanation

#### A. Project Configuration & Info Commands
- **Setting Current Project:** Before executing resource commands, ensure your active configuration points to the correct project:
  ```bash
  gcloud config set project [PROJECT_ID]
  ```
- **Describing Project Metadata:** To view quotas, default network tiers, and project details:
  ```bash
  gcloud compute project-info describe
  ```

#### B. Authentication Management (`gcloud auth`)
- **`gcloud auth login`:** Initiates browser-based user credential authentication for local environments. (Unnecessary in Cloud Shell, where authentication is automatic).
- **`gcloud auth list`:** Lists all currently active accounts logged into the CLI session.
- **`gcloud auth revoke`:** Logs out the active user account and removes local credentials.

#### C. Authorization & IAM Policy Mechanics
- **IAM Policy Structure:** A project-level IAM policy consists of a set of **IAM bindings**. Each binding maps a group of **members** (users, service accounts, or groups) to a specific **role**.
- **Inspecting Policies:** 
  ```bash
  gcloud projects get-iam-policy [PROJECT_ID]
  ```
- **Adding Policy Bindings:** Appends or merges a member into a role binding:
  ```bash
  gcloud projects add-iam-policy-binding [PROJECT_ID] \
      --member="user:in28minutes@gmail.com" \
      --role="roles/storage.objectAdmin"
  ```
- **Removing Policy Bindings:** Removes a member from a role binding:
  ```bash
  gcloud projects remove-iam-policy-binding [PROJECT_ID] \
      --member="user:in28minutes@gmail.com" \
      --role="roles/storage.objectAdmin"
  ```

#### D. IAM Role Management (`gcloud iam roles`)
- **Describing Roles:** Views permissions bundled inside a specific role:
  ```bash
  gcloud iam roles describe roles/storage.objectAdmin
  ```
- **Creating Roles:** Provision custom roles from scratch using fine-grained permissions. (Newly created custom roles start in the `alpha` stage).
- **Copying Roles:** Duplicate an existing predefined role to serve as a base for a custom role:
  ```bash
  gcloud iam roles copy \
      --source=roles/storage.objectAdmin \
      --destination=my.custom.role \
      --destination-project=my-project-id
  ```

---

### 4. Practical Setup & Implementation Guide

#### Step-by-Step Command Workflow in Cloud Shell
1. **Set the Active Project:**
   ```bash
   gcloud config set project my-first-project-id
   ```
   *Expected Outcome:* Updates core project configuration properties.
2. **Inspect Project Details:**
   ```bash
   gcloud compute project-info describe
   ```
   *Expected Outcome:* Outputs metadata, default service accounts, and resource quotas.
3. **Verify Active Accounts:**
   ```bash
   gcloud auth list
   ```
   *Expected Outcome:* Displays currently authenticated user emails.
4. **View Project IAM Policy:**
   ```bash
   gcloud projects get-iam-policy my-first-project-id
   ```
   *Expected Outcome:* Outputs current JSON/YAML formatted IAM bindings.
5. **Add an IAM Policy Binding:**
   ```bash
   gcloud projects add-iam-policy-binding my-first-project-id \
       --member="user:in28minutes@gmail.com" \
       --role="roles/storage.objectAdmin"
   ```
   *Expected Outcome:* Adds the user to the role binding and prints the updated policy.
6. **Remove the IAM Policy Binding:**
   ```bash
   gcloud projects remove-iam-policy-binding my-first-project-id \
       --member="user:in28minutes@gmail.com" \
       --role="roles/storage.objectAdmin"
   ```
   *Expected Outcome:* Removes the user from the binding and prints the updated policy.
7. **Copy a Predefined Role to a Custom Role:**
   ```bash
   gcloud iam roles copy \
       --source=roles/storage.objectAdmin \
       --destination=myCustomRole \
       --destination-project=my-first-project-id
   ```
   *Expected Outcome:* Creates a new user-managed custom role in `alpha` stage based on `storage.objectAdmin`.

---

### 5. Visual Representations

#### IAM Policy Structure Hierarchy
```text
  +-------------------------------------------------------------+
  |                        IAM POLICY                           |
  |                                                             |
  |   +-----------------------------------------------------+   |
  |   |                     BINDING 1                       |   |
  |   |   - Role: roles/owner                               |   |
  |   |   - Members: [user:in28minutes.demos@gmail.com]     |   |
  |   +-----------------------------------------------------+   |
  |                                                             |
  |   +-----------------------------------------------------+   |
  |   |                     BINDING 2                       |   |
  |   |   - Role: roles/storage.objectAdmin                 |   |
  |   |   - Members: [serviceAccount:xyz, user:abc]         |   |
  |   +-----------------------------------------------------+   |
  +-------------------------------------------------------------+
```

---

### 6. Mental Models & Analogies

#### The Club Guestlist Analogy
- **`gcloud auth login` / `list` / `revoke`:** Handing your driver's license to the club doorman to check your ID, verifying who is currently inside, or signing out when leaving.
- **IAM Policy & Bindings:** The master guestbook clipboard at the front desk. The clipboard represents the **IAM Policy**, and each row grouping names under a specific VIP badge level represents an **IAM Binding**.

---

### 7. Step-by-Step Workflows / Comparison Tables

#### Summary Table: Essential `gcloud` IAM and Project Commands
| Task | `gcloud` Command | Description |
| :--- | :--- | :--- |
| **Set Project** | `gcloud config set project [ID]` | Updates active working project ID. |
| **Get Project Info** | `gcloud compute project-info describe` | Inspects metadata, quotas, and service accounts. |
| **Check Accounts** | `gcloud auth list` | Lists active authenticated user identities. |
| **View IAM Policy** | `gcloud projects get-iam-policy [ID]` | Displays all policy bindings for a project. |
| **Add Binding** | `gcloud projects add-iam-policy-binding` | Adds a member to a role binding. |
| **Remove Binding** | `gcloud projects remove-iam-policy-binding` | Removes a member from a role binding. |
| **Describe Role** | `gcloud iam roles describe [ROLE]` | Lists fine-grained permissions inside a role. |
| **Copy Role** | `gcloud iam roles copy` | Creates a custom role by copying an existing role. |

---

### 8. Common Mistakes & Exam Traps
- **Trap:** Forgetting the prefix type (like `user:`, `serviceAccount:`, or `group:`) when supplying the `--member` flag in IAM binding commands.
  - *Correction:* Always include the correct member type prefix (e.g., `--member="user:email@gmail.com"`).
- **Trap:** Confusing `gcloud projects get-iam-policy` with bucket-level policy retrieval commands.
  - *Exam Tip:* Project-level policies use `gcloud projects`, whereas storage bucket operations use `gcloud storage buckets` or `gsutil`.

---

### 9. Certification Practice Questions

#### Question 1
You want to view all active IAM policy bindings currently configured on your Google Cloud project using the command line. Which command should you execute?
- A. `gcloud compute project-info describe`
- B. `gcloud projects get-iam-policy [PROJECT_ID]`
- C. `gcloud iam roles describe`
- D. `gcloud auth list`

**Correct Answer:** **B**  
**Explanation:** `gcloud projects get-iam-policy` retrieves the complete IAM policy containing all role bindings for the specified project.  
**Incorrect Answers:** A outputs project metadata/quotas. C describes specific role permissions. D lists active authenticated user sessions.  
**Exam Takeaway:** Use `gcloud projects get-iam-policy` to inspect project-level IAM bindings.

---

#### Question 2
Which command adds a user as a member to a specific role binding at the project level?
- A. `gcloud projects set-iam-policy`
- B. `gcloud projects add-iam-policy-binding`
- C. `gcloud iam roles create`
- D. `gcloud compute instances add-access`

**Correct Answer:** **B**  
**Explanation:** `gcloud projects add-iam-policy-binding` is the correct command to append a member to an IAM role binding within a project.  
- **Incorrect Answers:** A overwrites the entire policy. C creates a custom role. D is invalid.  
- **Exam Takeaway:** Use `add-iam-policy-binding` to assign members to roles.

---

#### Question 3
What prefix must be included before an email address when specifying a human user in the `--member` flag during an IAM policy binding command?
- A. `serviceAccount:`
- B. `user:`
- C. `group:`
- D. `domain:`

**Correct Answer:** **B**  
**Explanation:** Human user accounts require the `user:` prefix (e.g., `--member="user:name@gmail.com"`).  
**Incorrect Answers:** A is for service accounts. C is for Google Groups. D is for Workspace domains.  
**Exam Takeaway:** Prefix individual users with `user:` in IAM binding commands.

---

#### Question 4
You need to inspect the granular permissions included inside the predefined `roles/storage.objectAdmin` role using the `gcloud` CLI. Which command should you run?
- A. `gcloud projects get-iam-policy`
- B. `gcloud iam roles describe roles/storage.objectAdmin`
- C. `gcloud storage buckets describe`
- D. `gcloud auth list`

**Correct Answer:** **B**  
**Explanation:** `gcloud iam roles describe` displays the title, stage, and individual permissions associated with a specific role.  
**Incorrect Answers:** A retrieves project policies. C describes storage buckets. D lists user logins.  
**Exam Takeaway:** Use `gcloud iam roles describe` to view the permissions inside a role.

---

#### Question 5
How do you check which user accounts are currently authenticated in your local CLI terminal session?
- A. `gcloud auth list`
- B. `gcloud config set project`
- C. `gcloud projects list-users`
- D. `gcloud iam users describe`

**Correct Answer:** **A**  
**Explanation:** `gcloud auth list` displays all active accounts authenticated in the local CLI session.  
**Incorrect Answers:** B sets projects. C and D are invalid commands.  
**Exam Takeaway:** Use `gcloud auth list` to verify active CLI user credentials.

---

#### Question 6
You want to duplicate an existing predefined role (`roles/storage.objectAdmin`) to create a new custom role within your project via the CLI. Which command should you use?
- A. `gcloud iam roles create`
- B. `gcloud iam roles copy`
- C. `gcloud projects duplicate-role`
- D. `gcloud storage buckets clone`

**Correct Answer:** **B**  
-- **Explanation:** `gcloud iam roles copy` is used to duplicate existing roles into custom user-managed roles.  
-- **Incorrect Answers:** A creates blank custom roles from scratch. C and D are invalid commands.  
-- **Exam Takeaway:** Use `gcloud iam roles copy` to duplicate existing roles.

---

#### Question 7
What command is used to log out or revoke active user credentials from the `gcloud` CLI?
- A. `gcloud auth logout`
- B. `gcloud auth revoke`
- C. `gcloud config clear-auth`
- D. `gcloud iam revoke-access`

**Correct Answer:** **B**  
**Explanation:** `gcloud auth revoke` logs out the active user and deletes local credentials.  
**Incorrect Answers:** A, C, and D are incorrect command syntax.  
**Exam Takeaway:** Use `gcloud auth revoke` to log out of the CLI.

---

#### Question 8
What is the default launch stage assigned to a newly created custom IAM role when initialized via the `gcloud` CLI?
- A. General Availability (GA)
- B. Beta
- C. Alpha
- D. Deprecated

**Correct Answer:** **C**  
**Explanation:** Newly created custom roles default to the `alpha` launch stage for initial testing.  
**Incorrect Answers:** A, B, and D represent other lifecycle stages.  
**Exam Takeaway:** Custom roles start in the `alpha` stage by default.

---

#### Question 9
Which command inspects project metadata, default service accounts, and resource quotas?
- A. `gcloud compute project-info describe`
- B. `gcloud projects describe-quota`
- C. `gcloud config list-metadata`
- D. `gcloud storage project-meta`

**Correct Answer:** **A**  
**Explanation:** `gcloud compute project-info describe` outputs project-level metadata, default network settings, and quotas.  
**Incorrect Answers:** B, C, and D are incorrect or invalid commands.  
**Exam Takeaway:** Use `gcloud compute project-info describe` for project metadata inspection.

---

#### Question 10
If you want to remove a specific user from an IAM policy binding on a project without deleting the entire policy, which command should you run?
- A. `gcloud projects delete-iam-policy`
- B. `gcloud projects remove-iam-policy-binding`
- C. `gcloud iam users delete`
- D. `gcloud config unset member`

**Correct Answer:** **B**  
**Explanation:** `gcloud projects remove-iam-policy-binding` removes the specified member from the target role binding.  
**Incorrect Answers:** A destroys the entire policy. C and D are invalid commands.  
**Exam Takeaway:** Use `remove-iam-policy-binding` to revoke specific member role assignments.