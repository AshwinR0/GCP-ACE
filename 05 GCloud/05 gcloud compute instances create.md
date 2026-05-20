# GCP Associate Cloud Engineer - Deep Dive: `gcloud compute instances create`

## Overview
The `gcloud compute instances create` command is one of the most powerful and frequently used commands in the Google Cloud SDK. While a simple `gcloud compute instances create [NAME]` works using defaults, professional cloud engineers must master the specific **flags (options)** that define the VM's hardware, software, and operational behavior.

This section covers how to customize everything from CPU/RAM to startup scripts and deletion protection via the CLI.

---

## 🎯 ACE Exam Focus

### Frequently Tested Areas
*   **Custom Machine Types:** How to specify exact CPU and Memory values.
*   **Availability Policies:** Difference between `MIGRATE` and `TERMINATE` during maintenance.
*   **Storage Behavior:** Managing the `auto-delete` property of boot disks.
*   **Bootstrapping:** Using `--metadata-from-file` for startup and shutdown scripts.
*   **Preemptibility:** Launching low-cost, short-lived instances via the CLI.
*   **Deletion Protection:** Preventing accidental VM removal.

### Common Exam Trap
❌ **Confusing `--startup-script` with `--metadata-from-file`.**
*   ✔ **`--metadata startup-script="..."`**: Used for short, inline scripts.
*   ✔ **`--metadata-from-file startup-script=path/to/file`**: Best practice for long, complex scripts stored in a local file.

---

## 1. Machine Configuration: Predefined vs. Custom

### Predefined Machine Types
If you know the name of a standard machine type (e.g., `n1-standard-1`), you use a single flag:
```bash
--machine-type=n1-standard-96
```

### Custom Machine Types
If standard shapes don't fit, you can "build your own" using these three flags:
*   `--custom-cpu`: Number of cores.
*   `--custom-memory`: Amount of RAM (e.g., `3072MB` or `3GB`).
*   `--custom-vm-type`: The family (e.g., `n1` or `n2`).

**Example:**
```bash
gcloud compute instances create my-custom-vm \
    --custom-cpu=6 \
    --custom-memory=3072MB \
    --custom-vm-type=n2
```

---

## 2. Source Selection (Where to Boot From)
You don't always boot from a plain OS image. You have five main options:

| Source Type | Flag Example |
| :--- | :--- |
| **Image** | `--image=[IMAGE_NAME]` |
| **Image Family** | `--image-family=[FAMILY_NAME]` (Always pulls the latest version) |
| **Snapshot** | `--source-snapshot=[SNAPSHOT_NAME]` |
| **Instance Template** | `--source-instance-template=[TEMPLATE_NAME]` |
| **Machine Image** | `--source-machine-image=[MACHINE_IMAGE_NAME]` |

---

## 3. Availability & Maintenance Policies

These flags define how the VM behaves when Google performs hardware maintenance.

*   **`--maintenance-policy`**: 
    *   `MIGRATE` (Default): VM moves to a new host while running.
    *   `TERMINATE`: VM is powered off.
*   **`--restart-on-failure`**:
    *   `--restart-on-failure` (Default): Restarts VM if it crashes.
    *   `--no-restart-on-failure`: Stays off after a crash.
*   **`--preemptible`**: Includes this flag to create a short-lived, low-cost instance.

---

## 4. Storage & Deletion Protection

### Boot Disk Management
By default, the boot disk is deleted when the VM is deleted. You can change this behavior:
*   `--boot-disk-size`: Specify size (e.g., `20GB`).
*   `--boot-disk-type`: Specify type (e.g., `pd-ssd` or `pd-standard`).
*   `--no-boot-disk-auto-delete`: **CRITICAL.** Keeps the disk and its data even if the VM is destroyed.

### Deletion Protection
To prevent someone from running `gcloud compute instances delete` by accident:
*   `--deletion-protection`
*   `--no-deletion-protection` (Default)

---

## 5. Automation: Startup & Shutdown Scripts

You can inject code into the VM to run at specific lifecycle events.

| Type | Metadata Key | Description |
| :--- | :--- | :--- |
| **Startup** | `startup-script` | Runs every time the VM boots up. |
| **Shutdown** | `shutdown-script` | Runs when the VM is signaled to stop. |

**Example using a local file:**
```bash
--metadata-from-file startup-script=./web-config.sh
```

---

## 6. Visual Representations

### Command Flag Hierarchy
```text
gcloud compute instances create [NAME]
   |
   ├── [Hardware] --------> --machine-type OR --custom-cpu/memory
   ├── [Software] --------> --image OR --source-snapshot
   ├── [Storage] ---------> --boot-disk-size / --no-boot-disk-auto-delete
   ├── [Availability] ----> --preemptible / --maintenance-policy
   ├── [Automation] ------> --metadata-from-file startup-script=...
   └── [Networking] ------> --tags / --network / --subnet
```

### CLI Hierarchy Navigation
```text
gcloud
  └── compute (Group)
        └── instances (Subgroup)
              ├── create
              ├── delete
              ├── describe
              ├── start/stop
              └── reset
```

---

## 7. Mental Models & Analogies

### Analogy: Ordering a Custom Pizza
*   **Standard Machine Type:** Ordering a "Number 4" (Pepperoni). It's a pre-set combination.
*   **Custom Machine Type:** Picking exactly how many slices of pepperoni (CPU) and how much cheese (RAM) you want.
*   **Auto-delete Disk:** Deciding if you want to throw away the box (Disk) when you finish the pizza (Delete VM). If you want to keep the box for leftovers, use `--no-boot-disk-auto-delete`.

---

## 8. Common Mistakes & Exam Traps

### ❌ Mistake 1: Forgetting the "No" Prefix
Many GCP flags are "Boolean" (True/False). If the default is True, you must use `--no-[FLAGNAME]` to turn it off.
*   *Example:* `--no-boot-disk-auto-delete` or `--no-restart-on-failure`.

### ❌ Mistake 2: Preemptible Maintenance Conflict
*   **Trap:** Trying to set `--maintenance-policy=MIGRATE` on a `--preemptible` VM.
*   **Fact:** Preemptible VMs **cannot** migrate. They must be terminated.

---

## 🎯 10 Certification Practice Questions

**Q1. You need to create a VM with exactly 4 vCPUs and 8GB of RAM using the N2 family. Which CLI flags should you use?**
A. `--machine-type=n2-standard-4`
B. `--custom-cpu=4 --custom-memory=8GB --custom-vm-type=n2`
C. `--cpu=4 --ram=8 --type=n2`
D. `--n2-custom-cpu=4 --n2-custom-memory=8`
*   **✅ Answer: B**

**Q2. Which flag ensures that the boot disk is NOT deleted when the VM instance is terminated?**
A. `--keep-boot-disk`
B. `--no-boot-disk-auto-delete`
C. `--boot-disk-persistent`
D. `--no-delete-storage`
*   **✅ Answer: B**

**Q3. You want to run a script to unregister a server from a database when the VM is being shut down. Which flag should you use?**
A. `--startup-script`
B. `--shutdown-script`
C. `--metadata-from-file shutdown-script=[PATH]`
D. `--on-stop-script`
*   **✅ Answer: C**

**Q4. What is the default maintenance policy for a standard Compute Engine instance?**
A. TERMINATE
B. SUSPEND
C. MIGRATE
D. RESTART
*   **✅ Answer: C**

**Q5. You want to launch a VM from a specific point-in-time backup of another VM's disk. Which flag is required?**
A. `--image`
B. `--source-snapshot`
C. `--source-instance-template`
D. `--disk-backup`
*   **✅ Answer: B**

**Q6. You want to prevent a Junior Engineer from accidentally deleting a production VM via the CLI. Which flag should you include during creation?**
A. `--force-protection`
B. `--deletion-protection`
C. `--no-delete`
D. `--lock-instance`
*   **✅ Answer: B**

**Q7. You are writing a script to create 100 web servers. You want to make sure they all have the exact same tags for firewall rules. Which flag should you use?**
A. `--network-tags`
B. `--tags`
C. `--label`
D. `--firewall-group`
*   **✅ Answer: B**

**Q8. A company wants to use a specialized Service Account for a VM to access Cloud Storage. Which flag allows this?**
A. `--identity`
B. `--service-account`
C. `--iam-user`
D. `--sa-link`
*   **✅ Answer: B**

**Q9. If you don't want a VM to automatically restart after a physical host failure, which flag is correct?**
A. `--restart-on-failure=false`
B. `--no-restart-on-failure`
C. `--stop-on-crash`
D. `--manual-restart-only`
*   **✅ Answer: B**

**Q10. You need to find the documentation for all flags available for creating an instance. Which CLI command provides this help?**
A. `gcloud help create`
B. `gcloud compute instances create --help`
C. `gcloud info compute`
D. `gcloud instances describe --help`
*   **✅ Answer: B**

---

## ⚡ 30-Second Revision
*   **Customization:** Use `--custom-cpu` and `--custom-memory`.
*   **Sources:** Can be Image, Snapshot, Template, or Machine Image.
*   **Scripts:** Startup and Shutdown scripts are managed via `--metadata`.
*   **Maintenance:** Default is `MIGRATE`.
*   **Protection:** Use `--deletion-protection` for prod VMs.
*   **Persistence:** Use `--no-boot-disk-auto-delete` to keep data after VM deletion. ☁️