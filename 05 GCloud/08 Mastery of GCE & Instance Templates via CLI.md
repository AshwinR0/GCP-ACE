# GCP Associate Cloud Engineer - Mastery of GCE & Instance Templates via CLI

## Overview
This section focuses on the operational management of **Google Compute Engine (GCE)** using the `gcloud` CLI. We cover the full lifecycle of VM instances—from basic state changes (start/stop) to complex zonal moves—and the management of **Instance Templates**. 

For the ACE exam, understanding the command-line syntax for automating infrastructure is critical. You will be tested on specific flags that control disk persistence and how to bridge the gap between a "Template" (the blueprint) and an "Instance" (the live resource).

---

## 🎯 ACE Exam Focus

### Frequently Tested Areas
*   **Bulk Operations:** Running a single command on multiple instances by separating names with spaces.
*   **Disk Lifecycle:** Controlling which disks are deleted or kept when an instance is removed using `--delete-disks` and `--keep-disks`.
*   **Zonal Migration:** Using the `gcloud compute instances move` command.
*   **Template Creation:** Understanding that `instance-templates` support the same flags as `instances create` (e.g., `--preemptible`, `--metadata-from-file`).
*   **Provisioning from Template:** The specific syntax: `gcloud compute instances create [NAME] --source-instance-template=[TEMPLATE_NAME]`.

### Common Exam Trap
❌ **Confusing `instance-templates` with `instances`.**
*   **Template:** A global blueprint. It does not cost money until used.
*   **Instance:** A zonal resource that consumes billing immediately.
*   **Trap:** Trying to "start" a template. (You can only *create* an instance *from* a template).

---

## 1. Compute Instance Lifecycle Commands

The `gcloud compute instances` group is used for day-to-day VM management.

### ✅ Basic State Management
| Action | Command Example | Result |
| :--- | :--- | :--- |
| **Start** | `gcloud compute instances start vm-1` | Powers on a stopped VM. |
| **Stop** | `gcloud compute instances stop vm-1 vm-2` | Shuts down multiple VMs (space-separated). |
| **Reset** | `gcloud compute instances reset vm-1` | Hard reboot (clears RAM state). |
| **Delete** | `gcloud compute instances delete vm-1` | Permanently removes the VM resource. |

### ✅ Advanced Deletion Control
When you delete a VM, you must decide the fate of attached storage.

*   **`--delete-disks`**:
    *   `all`: Deletes boot and all data disks.
    *   `boot`: Deletes only the OS disk.
    *   `data`: Deletes only additional attached disks.
*   **`--keep-disks`**: Explicitly preserves specific disks even if the VM is deleted.

### ✅ Zonal Migration (`move`)
If you need to move a VM to a different zone within the same region:
```bash
gcloud compute instances move my-vm \
    --destination-zone us-central1-f \
    --zone us-central1-b
```
*   **Note:** This handles the backup, move, and restoration process in one command.

---

## 2. Instance Templates: The CLI Blueprint

Instance Templates allow you to define a configuration once and reuse it.

### ✅ Core Management
*   **List Templates:** `gcloud compute instance-templates list`
*   **Describe Details:** `gcloud compute instance-templates describe [TEMPLATE_NAME]`
*   **Delete Template:** `gcloud compute instance-templates delete [TEMPLATE_NAME]`

### ✅ Creating Templates
You can create a template from scratch or from an existing instance.

**From Scratch (Default):**
```bash
gcloud compute instance-templates create template-from-cli
```
*   *Default Machine Type:* `n1-standard-1`.

**From a Source Instance:**
```bash
gcloud compute instance-templates create my-template \
    --source-instance=my-live-vm \
    --source-instance-zone=us-central1-a
```

### ✅ Shared Configuration Flags
Instance templates support almost all flags used in standard VM creation:
*   **Cost:** `--preemptible`
*   **Automation:** `--metadata-from-file startup-script=script.sh`
*   **Maintenance:** `--maintenance-policy=MIGRATE`
*   **Identity:** `--service-account=[EMAIL]`

---

## 3. Launching Instances from Templates

This is the most common real-world workflow for scaling.

**Syntax:**
```bash
gcloud compute instances create new-vm-from-template \
    --source-instance-template=my-web-template \
    --zone=us-east1-b
```

---

## 4. Visual Representations

### Command Hierarchy
```text
          [ gcloud ]
               |
      [ compute ]
      /         \
 [ instances ]   [ instance-templates ]
      |                  |
   - list             - list
   - start/stop       - create (from scratch or instance)
   - delete           - describe
   - move             - delete
```

### Relationship Map
```text
[ Source VM ] --(create template)--> [ Instance Template ] --(create instance)--> [ New VM ]
                                            |
                                            |----(Machine Type)
                                            |----(Boot Disk)
                                            |----(Startup Script)
```

---

## 5. Mental Models & Analogies

### Analogy: The Cookie Cutter
*   **Instance Template:** The **Cookie Cutter**. It is a shape (blueprint). You can't eat the cookie cutter itself, and it doesn't "run."
*   **Compute Instance:** The **Cookie**. You use the cutter to make the cookie. You can have 100 cookies made from one cutter. The cookies are what actually "exist" and get eaten (billable resources).

---

## 6. Important Notes / Observations

*   **⚠️ Bulk Cleanup:** You can delete multiple templates or instances in one line by listing their names: `gcloud compute instances delete vm1 vm2 vm3`.
*   **⚠️ Metadata Persistence:** If you define a startup script in an instance template, every VM launched from that template will execute that script on boot.
*   **⚠️ Zone Prompt:** If you don't specify a `--zone` when creating an instance from a template, `gcloud` will prompt you to select one.

---

## 7. Common Mistakes & Exam Traps

### ❌ Mistake 1: Disk Loss
**Scenario:** Deleting a VM and losing data.
*   ✔ **Correct:** If you want to keep the data disk but delete the VM, use `--keep-disks=data`.

### ❌ Mistake 2: Forgetting the Source Zone
**Scenario:** Creating a template from an instance without specifying where that instance lives.
*   ✔ **Correct:** You **must** provide `--source-instance-zone` when creating a template from an existing VM.

---

## 🎯 10 Certification Practice Questions

**Q1. You need to stop three VM instances named `web-1`, `web-2`, and `web-3` simultaneously using the CLI. What is the correct syntax?**
A. `gcloud compute instances stop web-1, web-2, web-3`
B. `gcloud compute instances stop web-1 web-2 web-3`
C. `gcloud compute instances stop --all web`
D. `gcloud compute instances stop web-[1-3]`
*   **✅ Answer: B** (Space-separated names).

**Q2. You are deleting a VM but want to ensure that all attached data disks are also removed to save costs. Which flag should you use?**
A. `--remove-all`
B. `--delete-disks=all`
C. `--force-delete`
D. `--purge-disks`
*   **✅ Answer: B**

**Q3. You want to move a running VM from `us-central1-b` to `us-central1-f`. Which CLI command is designed for this specific task?**
A. `gcloud compute instances migrate`
B. `gcloud compute instances move`
C. `gcloud compute instances update-zone`
D. `gcloud compute instances transfer`
*   **✅ Answer: B**

**Q4. What is the correct command to see the full configuration details of an instance template named `prod-template`?**
A. `gcloud compute instance-templates list prod-template`
B. `gcloud compute instance-templates describe prod-template`
C. `gcloud compute instance-templates show prod-template`
D. `gcloud config configurations describe prod-template`
*   **✅ Answer: B**

**Q5. You want to create an instance template based on the configuration of an existing, perfectly configured VM named `gold-standard`. What must you provide in the CLI?**
A. Only the name of the template.
B. The name of the template and the `--source-instance` and `--source-instance-zone`.
C. The name of the template and the `--copy-from` flag.
D. You can only create templates from Custom Images, not instances.
*   **✅ Answer: B**

**Q6. Which flag would you add to a `gcloud compute instance-templates create` command to ensure all instances launched from it are Preemptible?**
A. `--preemptible`
/B. `--spot`
C. `--provisioning-model=preemptible`
D. `--lifecycle=temporary`
*   **✅ Answer: A**

**Q7. You have an instance template named `nginx-template`. What is the correct command to launch a new VM named `web-server` from this template?**
A. `gcloud compute instances create web-server --template=nginx-template`
B. `gcloud compute instances launch web-server --source-template=nginx-template`
C. `gcloud compute instances create web-server --source-instance-template=nginx-template`
D. `gcloud compute instance-templates launch nginx-template --name=web-server`
*   **✅ Answer: C**

**Q8. True or False: Instance Templates can be used to specify whether a VM should have deletion protection enabled.**
A. True
B. False
*   **✅ Answer: A** (Flags like `--deletion-protection` are supported).

**Q9. If you run `gcloud compute instance-templates create my-template` without any other flags, what machine type will be assigned?**
A. f1-micro
B. e2-medium
C. n1-standard-1
D. It will prompt you to choose one.
*   **✅ Answer: C** (n1-standard-1 is the CLI default for templates).

**Q10. You need to prevent a specific boot disk from being deleted when its VM is deleted via the CLI. Which flag allows this?**
A. `--no-boot-disk-auto-delete`
B. `--keep-disks=boot`
C. Both A and B are valid approaches depending on the command used.
D. `--lock-disk`
*   **✅ Answer: C** (`--no-boot-disk-auto-delete` is used during *create*; `--keep-disks` can be used during *delete*).

---

## ⚡ 30-Second Revision
*   **Start/Stop/Reset/Delete:** Basic lifecycle.
*   **Move:** Change zones within a region easily.
*   **Template:** A global blueprint.
*   **`--source-instance`**: Create a template from a live VM.
*   **`--source-instance-template`**: Create a VM from a template.
*   **Disk Control:** Use `--delete-disks` or `--keep-disks` to manage state after deletion. ☁️