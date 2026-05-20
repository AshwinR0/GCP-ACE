# GCP Associate Cloud Engineer - Managing Multiple `gcloud` Configurations

## Overview
As a Cloud Engineer, you rarely work on just one project. You might have a **Development** project, a **QA** project, and a **Production** project. Manually changing the Project ID, Region, and Zone every time you switch tasks is inefficient. 

Google Cloud allows you to create **Named Configurations** (profiles). Each configuration stores its own set of properties (Account, Project, Zone, etc.). Switching between environments becomes a simple one-line command.

---

## 🎯 ACE Exam Focus

### Frequently Tested Areas
*   **The Activation Behavior:** Knowing that `gcloud config configurations create [NAME]` both creates **and** automatically activates the new configuration.
*   **Listing:** Using `gcloud config configurations list` to see which profile is currently `ACTIVE` (marked with `True`).
*   **Switching Profiles:** Using the `activate` command to swap between environments.
*   **Inspecting:** Using `describe` to see settings of a configuration without having to activate it first.
*   **Cleanup:** Knowing you cannot delete the currently active configuration.

### Common Exam Trap
❌ **Trying to "edit" the name of a configuration.**
*   ✔ **Correct:** Configuration names are immutable. If you want a different name, you must create a new one and delete the old one.

---

## 1. The Configuration Management Commands

All commands follow the pattern: `gcloud config configurations [ACTION] [NAME]`

| Command | Purpose | Exam Detail |
| :--- | :--- | :--- |
| **`list`** | Shows all stored profiles. | Displays `NAME`, `IS_ACTIVE`, `ACCOUNT`, `PROJECT`, `DEFAULT_ZONE`. |
| **`create`** | Adds a new named profile. | **Automatically activates** the new profile upon creation. |
| **`activate`** | Switches the active profile. | This changes your Project/Zone context immediately. |
| **`describe`** | Shows settings for one profile. | Allows you to peek at a profile's settings without switching to it. |
| **`delete`** | Removes a profile. | You **cannot** delete the profile you are currently using. |

---

## 2. Professional Workflow: Switching Environments

### Step 1: List existing profiles
```bash
gcloud config configurations list
```
*Look for the `True` in the `IS_ACTIVE` column.*

### Step 2: Create a new environment (e.g., for QA)
```bash
gcloud config configurations create qa-env
```
*Note: You are now automatically "inside" the `qa-env` profile, but it has no project set yet.*

### Step 3: Configure the new environment
```bash
gcloud config set project my-qa-project-id
gcloud config set compute/region us-east1
```

### Step 4: Switch back to Production
```bash
gcloud config configurations activate prod-env
```

---

## 3. Visual Representations

### The Configuration Context Map
```text
[ gcloud SDK ]
      |
      |--- [ Configuration: dev-env ] (ACTIVE: False)
      |         |-- Project: dev-555
      |         |-- Zone: us-west1-a
      |
      |--- [ Configuration: prod-env ] (ACTIVE: True) <--- Commands apply here
                |-- Project: prod-999
                |-- Zone: us-central1-c
```

### Workflow Diagram
```text
[ Create ] ----> [ Set Settings ] ----> [ Work ]
                                          |
                                          V
[ List ] <---- [ Activate another ] <---- [ Finish Task ]
```

---

## 4. Mental Models & Analogies

### Analogy: Browser Profiles
Think of `gcloud` configurations like **Chrome or Edge Profiles**. 
*   **Work Profile:** Logged into work email, has work bookmarks (Project/Zone).
*   **Personal Profile:** Logged into personal email, has different settings.
*   **The Switch:** When you switch profiles, the whole context of what you "see" and "do" changes instantly, but the other profile's settings stay saved in the background.

---

## 5. Important Notes & Observations

*   **Initial Setup:** When you first run `gcloud init`, it usually creates a configuration named `default`.
*   **Automation:** If you are writing a script that needs to touch two projects, it is often safer to use the `--project` flag in the command itself rather than switching configurations in the script.
*   **Credential Sharing:** Multiple configurations can share the same `account` (email), or they can use different ones.

---

## 6. Common Mistakes & Exam Traps

### ❌ Mistake 1: Forgetting `create` activates
*   **Scenario:** You run `gcloud config configurations create test-env` and then immediately run `gcloud compute instances list` expecting to see your production VMs.
*   **Result:** You see nothing (or an error). 
*   **Reason:** You are now in `test-env`, which is empty.

### ❌ Mistake 2: Deleting the Active Config
*   **Scenario:** You are using `prod-env` and try to run `gcloud config configurations delete prod-env`.
*   **Result:** Error. You must `activate` a different configuration first.

---

## 🎯 10 Certification Practice Questions

**Q1. You are working on two different GCP projects. What is the most efficient way to switch between them using the gcloud CLI?**
A. Run `gcloud init` every time you switch.
B. Use `gcloud config set project` every time you switch.
C. Create two named configurations and use `gcloud config configurations activate` to swap.
D. Open two different Cloud Shell windows.
*   **✅ Answer: C**

**Q2. After running `gcloud config configurations create new-project-config`, which configuration is active?**
A. The `default` configuration.
B. No configuration is active until you run `activate`.
C. The `new-project-config` configuration.
D. The last configuration that was active before the command.
*   **✅ Answer: C**

**Q3. You want to see the details of a configuration named `stage-env` without making it the active configuration. Which command do you use?**
A. `gcloud config configurations list stage-env`
B. `gcloud config configurations describe stage-env`
C. `gcloud config list stage-env`
D. `gcloud config get stage-env`
*   **✅ Answer: B**

**Q4. You try to delete a configuration named `dev-config` but receive an error. What is the most likely reason?**
A. `dev-config` is the default configuration.
B. `dev-config` is currently active.
C. You don't have IAM permissions to delete configurations.
D. The configuration contains a project with a lien on it.
*   **✅ Answer: B**

**Q5. In the output of `gcloud config configurations list`, which column indicates which profile is currently in use?**
A. `NAME`
B. `IS_ACTIVE`
C. `STATUS`
D. `ACCOUNT`
*   **✅ Answer: B**

**Q6. True or False: Every configuration in gcloud MUST have a different Google Account (email) associated with it.**
A. True
B. False
*   **✅ Answer: B** (They can share the same account but point to different projects).

**Q7. You have 3 configurations: `dev`, `qa`, and `prod`. You want to move from `qa` to `dev`. What command is required?**
A. `gcloud config set dev`
/B. `gcloud config configurations activate dev`
C. `gcloud init dev`
D. `gcloud configurations switch dev`
*   **✅ Answer: B**

**Q8. What properties are typically stored within a gcloud configuration? (Select all that apply)**
A. Project ID
B. Default Zone
C. Account (Email)
D. User's Local Password
*   **✅ Answer: A, B, and C**

**Q9. Which command would you use to see a list of all your named configurations?**
A. `gcloud config list`
B. `gcloud config configurations list`
C. `gcloud configurations show`
D. `gcloud projects list`
*   **✅ Answer: B**

**Q10. You want to create a new configuration named `security-audit`. What is the correct syntax?**
A. `gcloud config set configurations security-audit`
B. `gcloud config configurations add security-audit`
C. `gcloud config configurations create security-audit`
D. `gcloud init --name security-audit`
*   **✅ Answer: C**

---

## ⚡ 30-Second Revision
*   **Multiple Environments?** Use Configurations.
*   **List:** `gcloud config configurations list`
*   **Create:** `gcloud config configurations create [name]` (Also activates it!).
*   **Switch:** `gcloud config configurations activate [name]`.
*   **Describe:** `gcloud config configurations describe [name]` (Peek at settings).
*   **Restriction:** Can't delete the active one. ☁️