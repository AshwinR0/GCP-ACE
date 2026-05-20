# GCP Associate Cloud Engineer - Managing MIGs via CLI (`gcloud`)

## Overview
While the Google Cloud Console provides a user-friendly interface for managing **Managed Instance Groups (MIGs)**, the `gcloud` CLI is essential for automation, scripting, and professional operations. This section covers the command hierarchy, creation workflows, and the configuration of scaling and healing policies using the command line.

For the ACE exam, you must be able to recognize the specific command paths and flags for managing compute resources, as well as the prerequisite steps like setting the active project.

---

## 🎯 ACE Exam Focus

### Frequently Tested Areas
*   **Command Path:** `gcloud compute instance-groups managed [ACTION]`.
*   **Prerequisites:** Using `gcloud config set project` before executing resource commands.
*   **Autoscaling Modes:** Distinguishing between `on`, `off`, and `only-scale-out`.
*   **Cooldown Period:** Understanding that this is the wait time between scaling actions.
*   **Auto-healing Policies:** The combination of `initial-delay` and `health-check`.
*   **Resource Identity:** Providing the `--zone` or `--region` flag for non-global resources.

### Common Exam Trap
❌ **Forgetting the "managed" keyword.**
*   ✔ **Correct:** `gcloud compute instance-groups managed list`. 
*   **Trap:** `gcloud compute instance-groups list` might show the groups, but many automated actions (like autoscaling) require the `managed` subgroup.

---

## 1. Prerequisites: Setting the Context

Before managing MIGs, you must ensure your CLI is pointing to the correct project.

### Step 1: Authorization
When opening a new Cloud Shell or terminal:
```bash
# Authorizes the SDK to access GCP
gcloud auth login 
```

### Step 2: Set the Project (Critical)
```bash
gcloud config set project [PROJECT_ID]
```
*   This updates the `core/project` property in your active configuration.

---

## 2. The Core Management Commands

The base path for all these commands is: `gcloud compute instance-groups managed`

| Action | Command | Purpose |
| :--- | :--- | :--- |
| **List** | `list` | Summary of name, location, template, and size. |
| **Create** | `create [NAME]` | Launch a new MIG from a template. |
| **Delete** | `delete [NAME]` | Permanently remove the group and its instances. |
| **Describe** | `describe [NAME]` | View full YAML metadata of the MIG. |

---

## 3. Creating a MIG via CLI

To create a MIG, you must at least specify a **name**, a **zone** (or region), and an **Instance Template**.

**Example (Zonal MIG):**
```bash
gcloud compute instance-groups managed create my-mig \
    --zone=us-central1-a \
    --template=my-instance-template-v1 \
    --size=1
```

**Key Flags:**
*   `--template`: The blueprint for the VMs.
*   `--size`: Initial target size.
*   `--region`: Use this instead of `--zone` for Regional MIGs.

---

## 4. Managing Autoscaling

Autoscaling is a sub-command of the managed instance group group.

### A. Setting Autoscaling
```bash
gcloud compute instance-groups managed set-autoscaling my-mig \
    --max-replicas=2 \
    --min-replicas=1 \
    --target-cpu-utilization=0.60 \
    --cool-down-period=60 \
    --zone=us-central1-a
```

### B. Autoscaling Flags & Definitions
*   **`--max-replicas`**: (Mandatory) The ceiling for scaling.
*   **`--cool-down-period`**: How long the autoscaler waits after a scaling action before looking at metrics again (Default: 60s).
*   **`--mode`**:
    *   `on`: Normal scaling (Up and Down).
    *   `off`: No scaling.
    *   `only-scale-out`: Will add VMs during spikes but **never delete them** when load drops.

### C. Stopping Autoscaling
```bash
gcloud compute instance-groups managed stop-autoscaling my-mig --zone=us-central1-a
```

---

## 5. Managing Auto-healing Policies

Auto-healing policies (Health Checks + Initial Delay) are modified using the `update` command.

```bash
gcloud compute instance-groups managed update my-mig \
    --initial-delay=300 \
    --health-check=my-http-check \
    --zone=us-central1-a
```

*   **Initial Delay:** The time given to an instance to finish booting and starting the app before the first health check is performed.

---

## 6. Visual Representations

### CLI Hierarchy Map
```text
          [ gcloud ]
               |
          [ compute ]
               |
      [ instance-groups ]
               |
          [ managed ]
               |
   ---------------------------------
   |         |          |          |
[list]    [create]   [describe] [delete]
             |
      -----------------------
      |                     |
[set-autoscaling]   [stop-autoscaling]
```

### The Workflow Loop
```text
[ gcloud init ] --> [ config set project ]
                          |
                          V
[ create MIG ] <--- [ create Template ]
      |
      V
[ set-autoscaling ] --> [ list / describe ]
```

---

## 7. Mental Models & Analogies

### Analogy: The Thermostat (Autoscaling)
*   **`set-autoscaling`**: You are setting the target temperature (60% CPU) and the range (1 to 2 VMs).
*   **Cooldown Period:** After the AC kicks in, the thermostat waits 60 seconds for the air to circulate before checking if it needs to work harder.
*   **`stop-autoscaling`**: You are turning off the thermostat; the house stays at whatever state it's in.

---

## 8. Common Mistakes & Exam Traps

### ❌ Mistake 1: Missing the Zone Flag
*   **Problem:** Running `gcloud compute instance-groups managed list` works globally, but `set-autoscaling` requires a `--zone` or `--region` because the resource is location-specific.

### ❌ Mistake 2: Confusing "Update" with "Create"
*   **Trap:** Trying to add a health check during the `create` phase.
*   **Fact:** While some CLI versions allow it, the standard operational pattern is to use `update` to modify auto-healing properties of an existing group.

---

## 🎯 10 Certification Practice Questions

**Q1. What is the correct gcloud command path to see all managed instance groups in your project?**
A. `gcloud compute managed-groups list`
B. `gcloud compute instance-groups managed list`
C. `gcloud compute instances managed list`
D. `gcloud instance-groups list --managed`
*   **✅ Answer: B**

**Q2. You attempt to run a GCE command in Cloud Shell but receive an error: "The project property is not set." Which command fixes this?**
A. `gcloud config set project [PROJECT_ID]`
B. `gcloud auth login`
C. `gcloud projects enable [PROJECT_ID]`
D. `gcloud init --project`
*   **✅ Answer: A**

**Q3. You want to enable autoscaling for a MIG named `web-mig`. Which flag is MANDATORY?**
A. `--min-replicas`
B. `--target-cpu-utilization`
C. `--max-replicas`
D. `--cool-down-period`
*   **✅ Answer: C**

**Q4. You want to increase the capacity of your group during a marketing campaign but ensure that instances are never automatically removed to avoid performance drops. Which mode should you use?**
A. `on`
B. `off`
C. `only-scale-out`
D. `static`
*   **✅ Answer: C**

**Q5. What does the `--cool-down-period` flag define in an autoscaling policy?**
A. The time it takes for a VM to shut down.
B. The time to wait after a scaling action before evaluating metrics again.
C. The delay before the first health check.
D. The time between a VM crash and a replacement.
*   **✅ Answer: B**

**Q6. Which command is used to remove an autoscaling policy from a managed instance group?**
A. `gcloud compute instance-groups managed delete-autoscaling`
B. `gcloud compute instance-groups managed stop-autoscaling`
C. `gcloud config unset autoscaling`
D. `gcloud compute autoscaler delete`
*   **✅ Answer: B**

**Q7. You need to update the time the MIG waits for a VM to start its application before performing a health check. Which flag do you use with the `update` command?**
A. `--cool-down-period`
B. `--boot-time`
C. `--initial-delay`
D. `--health-check-delay`
*   **✅ Answer: C**

**Q8. True or False: When you create a MIG via CLI, the current size defaults to the target size immediately.**
A. True
B. False
*   **✅ Answer: B** (The target size is set, but current size begins at 0 and increments as VMs provision).

**Q9. Which command allows you to see the full metadata, including the current target size and status, of a single MIG named `app-mig`?**
A. `gcloud compute instance-groups managed list app-mig`
B. `gcloud compute instance-groups managed describe app-mig`
C. `gcloud compute instances describe app-mig`
D. `gcloud compute instance-groups info app-mig`
*   **✅ Answer: B**

**Q10. How do you specify that a MIG should be created across an entire region rather than a single zone using the CLI?**
A. Use the `--region` flag instead of the `--zone` flag.
B. Use the `--multi-zonal` flag.
C. Use the `--global` flag.
D. You cannot create regional MIGs via CLI.
*   **✅ Answer: A**

---

## ⚡ 30-Second Revision
*   **Hierarchy:** `gcloud compute instance-groups managed`.
*   **`create`**: Needs `--template` and `--zone`.
*   **`set-autoscaling`**: Needs `--max-replicas`.
*   **`only-scale-out`**: Adds VMs, never deletes them.
*   **`cool-down`**: Wait after scaling to check metrics.
*   **`initial-delay`**: Wait after boot to check health.
*   **`stop-autoscaling`**: Turns off the auto-scaling logic. ☁️