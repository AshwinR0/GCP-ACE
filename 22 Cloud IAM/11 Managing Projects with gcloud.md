# GCP Associate Cloud Engineer (ACE) Study Notes

## Chapter: Managing Projects and IAM via `gcloud` (Part 2)

---

###  1. Overview
This chapter expands on managing Google Cloud **Projects** and their associated **IAM policies** using the `gcloud` command-line interface. It covers project lifecycle operations (listing, describing, creating, deleting, and undeleting), project migration between organizations, and managing project-level IAM bindings.

---

### 2. ACE Exam Focus & Common Exam Traps
- **Exam Importance:** **HIGH**. The ACE exam frequently tests project lifecycle operations, deletion grace periods, recovery commands, and project-level IAM management.
- **High-Priority Commands:**
  - **List Projects:** `gcloud projects list`
  - **Describe Project:** `gcloud projects describe [PROJECT_ID]`
  - **Create Project:** `gcloud projects create [PROJECT_ID]`
  - **Delete Project:** `gcloud projects delete [PROJECT_ID]`
  - **Undelete Project:** `gcloud projects undelete [PROJECT_ID]`
  - **Move Project:** `gcloud projects move [PROJECT_ID] --organization [ORG_ID]`
  - **Get IAM Policy:** `gcloud projects get-iam-policy [PROJECT_ID]`
  - **Add/Remove IAM Bindings:** `gcloud projects add-iam-policy-binding` / `gcloud projects remove-iam-policy-binding`
- **Common Exam Traps:**
  - Believing that executing `gcloud projects delete` instantly and permanently wipes the project data. (Trap! Google provides a temporary grace period during which deleted projects can be recovered using `undelete`).
  - Confusing project ID requirements when querying IAM policies or project descriptions.

---

### 3. Detailed Concept Explanation

#### A. Project Lifecycle Management (`gcloud projects`)
- **Listing Projects:** To see all projects you have access to, along with their project IDs, names, and project numbers:
  ```bash
  gcloud projects list
  ```
- **Describing Projects:** To inspect detailed metadata (status, organization linkage, project name, project number, project ID):
  ```bash
  gcloud projects describe [PROJECT_ID]
  ```
- **Creating and Deleting Projects:**
  - Creation: `gcloud projects create [PROJECT_ID]`
  - Deletion: `gcloud projects delete [PROJECT_ID]`
- **The Deletion Grace Period & Undelete:** 
  - When you delete a project, it is not erased immediately. It enters a temporary shutdown grace period (typically 30 days).
  - During this grace period, an administrator can restore the project using:
    ```bash
    gcloud projects undelete [PROJECT_ID]
    ```

#### B. Project Organization Migration
- If your corporate structure changes, you can migrate an existing project from one organization node to another using:
  ```bash
  gcloud projects move [PROJECT_ID] --organization=[ORG_ID]
  ```

#### C. Reviewing and Modifying Project IAM Policies
- **Get Policy:** Retrieves all role bindings associated with a project:
  ```bash
  gcloud projects get-iam-policy [PROJECT_ID]
  ```
- **Modify Bindings:** Add or remove member-to-role mappings dynamically:
  - Add: `gcloud projects add-iam-policy-binding [PROJECT_ID] --member="..." --role="..."`
  - Remove: `gcloud projects remove-iam-policy-binding [PROJECT_ID] --member="..." --role="..."`

---

### 4. Practical Setup & Implementation Guide

#### Step-by-Step Project Management via CLI
1. **List all available projects:**
   ```bash
   gcloud projects list
   ```
   *Expected Outcome:* Displays a table showing Project ID, Name, and Project Number.
2. **Describe a specific project:**
   ```bash
   gcloud projects describe my-sample-project-id
   ```
   *Expected Outcome:* Outputs configuration details, lifecycle state (`ACTIVE`), and parent organization identifiers.
3. **Delete a project (initiating the grace period):**
   ```bash
   gcloud projects delete my-sample-project-id
   ```
   *Expected Outcome:* Prompts for confirmation and places the project into a pending deletion state.
4. **Undelete/Restore the project within the grace period:**
   ```bash
   gcloud projects undelete my-sample-project-id
   ```
   *Expected Outcome:* Restores the project back to `ACTIVE` status.
5. **Move a project to a new organization:**
   ```bash
   gcloud projects move my-sample-project-id --organization=123456789012
   ```
   *Expected Outcome:* Reassociates the project under the target organization node.

---

### 5. Visual Representations

#### Project Deletion and Recovery Lifecycle
```text
  +------------------+          `gcloud projects delete`         +-------------------+
  |      ACTIVE      | ----------------------------------------> | PENDING DELETION  |
  | (Normal State)   |                                           | (Grace Period)    |
  +------------------+                                           +-------------------+
           ^                                                               |
           |                    `gcloud projects undelete`                 |
           +---------------------------------------------------------------+
           | 
      (Restored within timeframe)
```

---

### 6. Mental Models & Analogies

#### The Recycling Bin Analogy
- **Project Deletion (`gcloud projects delete`):** Like dragging a folder into your computer's operating system Recycle Bin or Trash. It's out of active view and disabled, but not permanently shredded yet.
- **Project Undelete (`gcloud projects undelete`):** Like clicking "Restore from Trash" within the grace period window to pull the folder back onto your desktop safe and sound.

---

### 7. Step-by-Step Workflows / Comparison Tables

#### Summary Table: Project Management Commands
| Action | `gcloud` Command | Description |
| :--- | :--- | :--- |
| **List** | `gcloud projects list` | Lists all projects accessible to the user. |
| **Describe** | `gcloud projects describe [ID]` | Outputs detailed configuration metadata for a project. |
| **Create** | `gcloud projects create [ID]` | Provisions a new project container. |
| **Delete** | `gcloud projects delete [ID]` | Shuts down and schedules a project for deletion. |
| **Undelete** | `gcloud projects undelete [ID]` | Restores a deleted project during the grace period. |
| **Move** | `gcloud projects move [ID] --organization=[ORG_ID]` | Migrates a project to a different organization node. |

---

### 8. Common Mistakes & Exam Traps
- **Trap:** Assuming `gcloud projects delete` instantly wipes all resources permanently with no recovery option.
  - *Correction:* Deleted projects enter a temporary grace period during which they can be recovered using `gcloud projects undelete`.
- **Trap:** Forgetting that project move operations require appropriate permissions on both the source and target organization hierarchies.

---

### 9. Certification Practice Questions

#### Question 1
An administrator accidentally deletes a critical production project using the `gcloud projects delete` command. Within the allowed grace period, how can they restore the project?
- A. Run `gcloud projects undelete [PROJECT_ID]`
- B. Run `gcloud projects restore [PROJECT_ID]`
- C. Contact Google Cloud Support to recover it; CLI recovery is impossible.
- D. Run `gcloud projects recreate-deleted [PROJECT_ID]`

**Correct Answer:** **A**  
**Explanation:** Projects scheduled for deletion can be recovered during the grace period using `gcloud projects undelete`.  
**Incorrect Answers:** B, C, and D are incorrect command names or false assumptions.  
**Exam Takeaway:** Use `gcloud projects undelete` to recover a deleted project within its grace period.

---

#### Question 2
Which `gcloud` command should you execute to display a list of all projects you have access to, including their project IDs and names?
- A. `gcloud compute projects show-all`
- B. `gcloud projects list`
- C. `gcloud config list-projects`
- D. `gcloud iam projects query`

**Correct Answer:** **B**  
**Explanation:** `gcloud projects list` outputs all projects accessible to the current user profile.  
**Incorrect Answers:** A, C, and D are invalid commands.  
**Exam Takeaway:** Use `gcloud projects list` to view accessible projects.

---

#### Question 3
What command allows you to migrate an existing project from one organization node to a different organization node?
- A. `gcloud projects move [PROJECT_ID] --organization=[ORG_ID]`
- B. `gcloud projects transfer --target-org=[ORG_ID]`
- C. `gcloud organization migrate-project`
- D. Projects cannot be moved between organizations once created.

**Correct Answer:** **A**  
**Explanation:** `gcloud projects move` combined with the `--organization` flag allows migrating projects between organization hierarchies.  
**Incorrect Answers:** B and C are incorrect syntax. D is false.  
**Exam Takeaway:** Use `gcloud projects move --organization=` to shift projects between organizations.

---

#### Question 4
What information is returned when you execute the `gcloud projects describe [PROJECT_ID]` command?
- A. The complete audit log history of the project for the last 30 days.
- B. Detailed configuration metadata, including lifecycle status, project name, project number, and parent organization.
- C. The exact monthly billing charges incurred to date.
- D. A list of all virtual machines currently running inside the project.

**Correct Answer:** **B**  
**Explanation:** `gcloud projects describe` outputs metadata concerning the project's configuration state, IDs, and organization hierarchy.  
**Incorrect Answers:** A, C, and D require Logging, Billing, or Compute-specific commands respectively.  
**Exam Takeaway:** Use `gcloud projects describe` for project metadata inspection.

---

#### Question 5
True or False: When you run `gcloud projects delete`, Google Cloud deletes the project and its resources instantly and permanently without any possibility of recovery.
- A. True
- B. False

**Correct Answer:** **B**  
**Explanation:** False. Deletion initiates a temporary shutdown grace period, allowing the project to be recovered using `undelete`.  
**Exam Takeaway:** Project deletion is not instantaneous and permanent; it includes a recovery grace period.

---

#### Question 6
Which command retrieves the entire IAM policy and all role bindings for a specific project?
- A. `gcloud projects get-iam-policy [PROJECT_ID]`
- B. `gcloud iam policies list-all`
- C. `gcloud compute project-info describe`
- D. `gcloud projects show-bindings`

**Correct Answer:** **A**  
**Explanation:** `gcloud projects get-iam-policy` outputs the full JSON/YAML policy structure of project IAM bindings.  
**Incorrect Answers:** B, C, and D are incorrect commands.  
**Exam Takeaway:** Use `get-iam-policy` to inspect project-level IAM bindings.

---

#### Question 7
If an administrator wants to add a user to a specific role on a project via the CLI, which command should they use?
- A. `gcloud projects update-iam-policy`
- B. `gcloud projects add-iam-policy-binding`
- C. `gcloud iam users assign`
- D. `gcloud projects bind-role`

**Correct Answer:** **B**  
-- **Explanation:** `gcloud projects add-iam-policy-binding` appends a member-to-role mapping to the project's IAM policy.  
-- **Incorrect Answers:** A, C, and D are incorrect syntax.  
-- **Exam Takeaway:** Use `add-iam-policy-binding` to assign roles to members.

---

#### Question 8
Which command removes a member from a project-level IAM role binding?
- A. `gcloud projects remove-iam-policy-binding`
- B. `gcloud projects delete-binding`
- C. `gcloud iam users revoke`
- D. `gcloud config clear-member`

**Correct Answer:** **A**  
**Explanation:** `gcloud projects remove-iam-policy-binding` removes the specified member from a role binding.  
**Incorrect Answers:** B, C, and D are invalid commands.  
**Exam Takeaway:** Use `remove-iam-policy-binding` to strip member access from role bindings.

---

#### Question 9
What unique numerical identifier is automatically assigned to every Google Cloud project alongside its user-defined Project ID?
- A. Project Number
- B. Billing UUID
- C. Organization Hash
- D. Compute Serial Key

**Correct Answer:** **A**  
**Explanation:** GCP projects possess both a user-chosen Project ID and a system-generated Project Number.  
**Incorrect Answers:** B, C, and D are distractor terms.  
**Exam Takeaway:** Projects feature both a human-readable Project ID and a system-generated Project Number.

---

#### Question 10
Which of the following lifecycle states must a project be in before it can be successfully undeleted?
- A. ACTIVE
- B. PENDING_DELETION (within the grace period window)
- C. FROZEN
- D. ARCHIVED

**Correct Answer:** **B**  
**Explanation:** Undeletion applies specifically to projects currently in the pending deletion grace period state.  
**Incorrect Answers:** A projects are already active. C and D are not standard GCP project states.  
**Exam Takeaway:** Undeletion recovers projects sitting in the pending deletion state.