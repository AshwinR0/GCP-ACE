# GCP Associate Cloud Engineer - Advanced MIG Updates via CLI

## Overview
Once a **Managed Instance Group (MIG)** is operational, a Cloud Engineer must perform ongoing maintenance tasks such as scaling, troubleshooting specific instances, and deploying updated configurations. This section focuses on the command-line operations used to modify existing MIGs without deleting the entire group.

For the **ACE exam**, you must distinguish between actions that preserve the VM (Refresh/Restart) and actions that destroy/recreate the VM (Replace/Recreate).

---

## 🎯 ACE Exam Focus

### Frequently Tested Commands
*   **`resize`**: Changing the target number of instances.
*   **`recreate-instances`**: Forcing a specific VM to be deleted and rebuilt from the template.
*   **`update-instances`**: Fine-grained control over how updates are applied to specific VMs.
*   **`set-instance-template`**: Swapping the blueprint (template) used by the MIG.

### Common Exam Trap
❌ **Confusing `recreate-instances` with `update-instances`.**
*   ✔ **`recreate-instances`**: ALWAYS deletes and recreates the VM (most disruptive).
*   ✔ **`update-instances`**: Can perform less disruptive actions like a simple **Restart** or **Refresh** based on the flags you provide.

---

## 1. Resizing a MIG (`resize`)

The `resize` command is used to manually scale the group up or down.

### CLI Workflow
1.  **Check current size:**
    ```bash
    gcloud compute instance-groups managed list
    ```
    *Note the `TARGET_SIZE` column.*

2.  **Change the size:**
    ```bash
    gcloud compute instance-groups managed resize [MIG_NAME] \
        --size [NEW_SIZE] \
        --zone [ZONE]
    ```

3.  **Verify:** Run the `list` command again to see the updated `TARGET_SIZE`.

---

## 2. Recreating Instances (`recreate-instances`)

Used when a specific VM is acting "flaky" or has a corrupt OS, and you want a fresh start based on the original Instance Template.

### How it works:
*   It deletes the existing VM.
*   It creates a brand-new VM with the same name (if it's a stateful MIG) or a new name (if stateless) based on the template.

### CLI Example:
```bash
gcloud compute instance-groups managed recreate-instances MyMIG \
    --instances MyMIG-85fb \
    --zone us-central1-a
```

---

## 3. Fine-Grained Updates (`update-instances`)

This command is more flexible than `recreate-instances`. It allows you to specify the level of disruption you are willing to tolerate.

### Key Logic: Action Levels
| Action | Disruption Level | Description |
| :--- | :--- | :--- |
| **`REFRESH`** | Low | Updates metadata/labels without stopping the VM. |
| **`RESTART`** | Medium | Performs a standard OS reboot. No deletion occurs. |
| **`REPLACE`** | High | Deletes and recreates the instance (Default for `recreate`). |

### Important Flags
*   **`--minimal-action`**: The least disruptive action the MIG should take to apply the change.
*   **`--most-disruptive-allowed-action`**: The ceiling of disruption. If an update requires a "Replace" but you set this to "Restart," the update will fail rather than delete the VM.

### CLI Example:
```bash
gcloud compute instance-groups managed update-instances MyMIG \
    --instances MyMIG-85fb \
    --minimal-action restart \
    --zone us-central1-a
```

---

## 4. Updating the Blueprint (`set-instance-template`)

When you create a "v2" of your application, you create a new Instance Template. You then need to tell the MIG to start using that new template for all future instances.

### CLI Example:
```bash
gcloud compute instance-groups managed set-instance-template MyMIG \
    --template my-new-v2-template \
    --zone us-central1-a
```
*   **Note:** Simply running this command **does not** automatically update existing VMs. It only sets the template for *new* VMs. To update existing ones, you would follow up with a Rolling Update or `update-instances`.

---

## 5. Visual Representations

### Disruption Spectrum
```text
[ REFRESH ] <---------- [ RESTART ] <---------- [ REPLACE / RECREATE ]
(Least Disruptive)                          (Most Disruptive)
Keep VM running         Power cycle VM          Delete & Start Over
```

### Request Flow: Recreate Instance
```text
Admin Command
     |
Managed Instance Group
     |
1. Signal VM for Deletion
     |
2. Delete Persistent State (if stateless)
     |
3. Launch New VM from Template
```

---

## 6. Mental Models & Analogies

### Analogy: Fixing a Computer
*   **Resize:** Buying more computers for the office.
*   **Refresh:** Changing the screensaver on the computer while the employee is still typing.
*   **Restart:** Turning the computer off and on again to clear a minor bug.
*   **Recreate / Replace:** Throwing the computer in the trash and pulling a brand-new one out of the storage room.

---

## 7. Service Comparison Table

| Feature | `resize` | `recreate-instances` | `update-instances` |
| :--- | :--- | :--- | :--- |
| **Main Goal** | Scaling quantity | Fixing 1 broken VM | Changing 1 VM's config |
| **Scope** | Entire Group | Specific Instances | Specific Instances |
| **Max Disruption** | Variable | Always **Replace** | Configurable (Restart/Refresh) |
| **ACE Relevance** | High | High | High |

---

## 8. Common Mistakes & Exam Traps

### ❌ Common Exam Mistakes

### Mistake 1:
Using `recreate-instances` when you only need to update a metadata label.
✔ **Correct:** Use `update-instances --minimal-action refresh` to avoid unnecessary downtime.

### Mistake 2:
Thinking `set-instance-template` immediately replaces all running VMs.
✔ **Correct:** This command only updates the "blueprint." Existing VMs keep running on the old template until they are manually updated or replaced by the autoscaler.

---

## 🎯 10 Certification Practice Questions

**Q1. You need to reduce the number of instances in a Managed Instance Group from 5 to 3. Which command should you use?**
A. `update-instances`
B. `set-instance-template`
C. `resize`
D. `recreate-instances`
*   **✅ Answer: C**

**Q2. An instance in your MIG is unresponsive. You want to completely delete it and have the MIG provision a fresh copy from the template. Which command is best?**
A. `update-instances --minimal-action refresh`
B. `recreate-instances`
C. `resize`
D. `gcloud compute instances stop`
*   **✅ Answer: B**

**Q3. You want to apply a small metadata change to a specific instance in a MIG but ensure that the VM is NOT deleted during the process. What flag should you use with `update-instances`?**
A. `--minimal-action replace`
B. `--minimal-action restart`
C. `--most-disruptive-allowed-action restart`
D. `--force-refresh`
*   **✅ Answer: C** (Setting the "Most Disruptive" to restart prevents deletion/replacement).

**Q4. You have created `template-v2` with a new version of your app. You want all *future* instances added by the autoscaler to use this template. What command do you run?**
A. `resize`
B. `update-instances`
C. `set-instance-template`
D. `gcloud config set template`
*   **✅ Answer: C**

**Q5. How do you see the names of specific instances inside a MIG so you can recreate them?**
A. `gcloud compute instance-groups managed list`
B. `gcloud compute instances list`
C. `gcloud config list`
D. `gcloud compute instance-templates list`
*   **✅ Answer: B**

**Q6. Which command allows you to restart a VM without deleting its local persistent disk?**
A. `recreate-instances`
B. `update-instances --minimal-action restart`
C. `resize --size 0`
D. `delete-instances`
*   **✅ Answer: B**

**Q7. True or False: The `resize` command requires a `--zone` flag if the MIG is zonal.**
A. True
B. False
*   **✅ Answer: A**

**Q8. What is the default action for `recreate-instances`?**
A. Restart
B. Refresh
C. Replace (Delete and Recreate)
D. Suspend
*   **✅ Answer: C**

**Q9. You want to ensure that an update to a specific VM never causes more disruption than a simple reboot. Which flag configuration is correct?**
A. `--minimal-action restart`
B. `--most-disruptive-allowed-action restart`
C. `--force-restart`
D. `--no-replace`
*   **✅ Answer: B**

**Q10. In the output of `gcloud compute instance-groups managed list`, what column tells you the current number of instances the group is trying to maintain?**
A. `CURRENT_SIZE`
B. `HEALTH_STATE`
C. `TARGET_SIZE`
D. `TEMPLATE_NAME`
*   **✅ Answer: C**

---

## ⚡ 30-Second Revision

*   **Resize:** Changing the count.
*   **Recreate:** Deleting and rebuilding a specific VM (Replace).
*   **Update-Instances:** Choose your level of pain (Refresh vs. Restart vs. Replace).
*   **Set-Instance-Template:** Updating the "Master Plan" for the group.
*   **Exam Strategy:** If a question asks to fix a specific VM, look for `recreate-instances`. If it asks to change the group size, look for `resize`. ☁️