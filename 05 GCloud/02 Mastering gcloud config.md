# GCP Associate Cloud Engineer - Mastering `gcloud config` Commands

## Overview
The `gcloud config` command group is the "Settings Menu" of your Google Cloud SDK. It allows you to view and modify the properties that govern how the `gcloud` tool behaves. Without a proper configuration, every command you run would require you to manually type your Project ID, Region, and Zone—a tedious and error-prone process.

For the ACE exam, you must master the syntax for **listing**, **setting**, and **unsetting** these properties, and understand the relationship between **Sections** and **Properties**.

---

## 🎯 ACE Exam Focus

### Frequently Tested Areas
*   **The Default Section:** Understanding that `core` is the default section (you can omit `core/` in commands).
*   **Property Syntax:** The `section/property` format (e.g., `compute/zone`).
*   **Setting the Project:** The most frequent CLI question involves `gcloud config set project [PROJECT_ID]`.
*   **Verbosity:** Using `core/verbosity` to enable debug mode for troubleshooting.
*   **Help Flag:** Using `--help` to discover available properties.

### Common Exam Trap
❌ **Trying to set a region without specifying the section.**
*   ✔ **Correct:** `gcloud config set region [VALUE]` will fail if you don't use `compute/region`, because `region` is not in the `core` section.

---

## 1. Understanding Sections and Properties

GCP organizes configuration settings into **Sections**. Every **Property** lives inside a specific Section.

| Section | Common Properties | Description |
| :--- | :--- | :--- |
| **`core`** (Default) | `project`, `account`, `verbosity` | Core settings for authentication and project identification. |
| **`compute`** | `region`, `zone` | Default locations for Compute Engine resources. |
| **`container`** | `cluster`, `location` | Settings for Google Kubernetes Engine (GKE). |

---

## 2. Reading Configuration: `gcloud config list`

This command shows you what is currently configured.

### Commands & Filtering
*   **List Everything:** `gcloud config list`
*   **Filter by Core Property:** `gcloud config list account` (Works because `core` is default).
*   **Filter by Compute Property:** `gcloud config list compute/region` (You **must** specify the `compute/` section).
*   **Explicit Core Syntax:** `gcloud config list core/account` (Same as just `account`).

---

## 3. Writing Configuration: `gcloud config set`

Use this to change your active settings.

### Essential Examples
```bash
# 1. Set the Active Project (Most Important for Exam)
gcloud config set project my-production-project-123

# 2. Set the Default Zone
gcloud config set compute/zone us-central1-a

# 3. Set the Default Region
gcloud config set compute/region us-central1

# 4. Enable Debugging (Verbosity)
gcloud config set core/verbosity debug
```

### 🧠 Operational Best Practice
Before running a destructive command (like deleting a VM or database), **always** run `gcloud config list` first to ensure you are in the correct project and account.

---

## 4. Removing Configuration: `gcloud config unset`

If you want to clear a setting (e.g., you no longer want a default zone), use `unset`.

*   **Remove Project:** `gcloud config unset project`
*   **Remove Zone:** `gcloud config unset compute/zone`

---

## 5. Getting Help: The `--help` Flag

If you forget a property name or syntax, GCP has a built-in manual.

*   **General Config Help:** `gcloud config --help`
*   **Specific Set Help:** `gcloud config set --help`
*   *Note:* Use the `j/k` keys to scroll and `q` to quit the help screen.

---

## 6. Visual Representations

### The Configuration Mental Model
```text
[ gcloud SDK ]
      |
      |--- [ Active Configuration ]
                |
                |--- [ Section: core ]
                |         |-- account: bob@gmail.com
                |         |-- project: prod-123
                |
                |--- [ Section: compute ]
                          |-- region: europe-west1
                          |-- zone: europe-west1-b
```

### Command Syntax Logic
```text
gcloud config [ACTION] [SECTION]/[PROPERTY] [VALUE]

Example:
gcloud config   set      compute /   zone     us-east1-a
```

---

## 7. Mental Models & Analogies

### Analogy: The Car's Preset Buttons
Think of `gcloud config` like the **Memory Preset buttons** in a car.
*   **Property:** Seat Position, Radio Station, Mirror Angle.
*   **Set:** You adjust everything and hit "Save."
*   **List:** You look at the screen to see which driver profile is active.
*   **Switching Configs:** Swapping from "Driver A" to "Driver B." Everything (Project, Region, Account) changes at once.

---

## 8. Common Mistakes & Exam Traps

### ❌ Mistake 1: Forgetting the `compute/` prefix
*   **Command:** `gcloud config set zone us-central1-a`
*   **Result:** Error. 
*   **Reason:** `zone` is not in the `core` section. You must use `compute/zone`.

### ❌ Mistake 2: Confusing `list` and `set`
*   **Trap:** The exam might ask for a command to *check* the project but provide a `set` command in the options.
*   **Fix:** `list` is for reading; `set` is for writing.

---

## 🎯 10 Certification Practice Questions

**Q1. Which section is considered the "default" in gcloud, allowing you to omit the section name in commands?**
A. compute
B. core
C. general
D. account
*   **✅ Answer: B**

**Q2. What is the correct command to set the default region for Compute Engine resources?**
A. `gcloud config set region us-east1`
B. `gcloud config set core/region us-east1`
C. `gcloud config set compute/region us-east1`
D. `gcloud config update region us-east1`
*   **✅ Answer: C**

**Q3. You want to see only the account currently configured in your active profile. Which command works? (Select two)**
A. `gcloud config list account`
B. `gcloud config list core/account`
C. `gcloud config show account`
D. `gcloud config get account`
*   **✅ Answer: A and B**

**Q4. You need to troubleshoot a failing gcloud command and want to see the detailed API requests being made. What should you do?**
A. `gcloud config set compute/verbosity debug`
B. `gcloud config set core/verbosity debug`
C. `gcloud config list --debug`
D. `gcloud log show --all`
*   **✅ Answer: B**

**Q5. What happens when you run `gcloud config unset project`?**
A. The GCP project is deleted from the cloud.
B. The project ID is removed from the active local configuration.
C. You are logged out of gcloud.
D. The project is moved to the "archive" section.
*   **✅ Answer: B**

**Q6. You are in the middle of a command and forget which properties are available under the `core` section. What flag should you use?**
A. `-h`
B. `--help`
C. `man gcloud`
D. `--info`
*   **✅ Answer: B**

**Q7. Which command allows you to change the active project to `my-test-project`?**
A. `gcloud project set my-test-project`
B. `gcloud config set project my-test-project`
C. `gcloud config list project my-test-project`
D. `gcloud use project my-test-project`
*   **✅ Answer: B**

**Q8. True or False: `gcloud config list compute/zone` will only show the zone property, even if a project is also configured.**
A. True
B. False
*   **✅ Answer: A**

**Q9. Which property is NOT part of the `core` section?**
A. `account`
B. `project`
C. `zone`
D. `verbosity`
*   **✅ Answer: C** (It's under `compute`).

**Q10. What is the standard way to exit the help documentation screen in Cloud Shell?**
A. `Ctrl + C`
/B. `Esc`
C. `q`
D. `exit`
*   **✅ Answer: C**

---

## ⚡ 30-Second Revision
*   **List:** `gcloud config list [section/property]`
*   **Set:** `gcloud config set [section/property] [value]`
*   **Unset:** `gcloud config unset [section/property]`
*   **Section Rule:** `core` is optional; `compute` (and others) is mandatory.
*   **Top 3 Properties:** `project`, `compute/region`, `compute/zone`. ☁️