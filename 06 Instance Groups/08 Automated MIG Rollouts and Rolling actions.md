# GCP Associate Cloud Engineer - Automated MIG Rollouts & Rolling Actions

## Overview

In the previous steps, we learned how to change the "blueprint" of a Managed Instance Group (MIG) using `set-instance-template`. However, simply changing the template does **not** automatically update the running virtual machines. 

This section covers the "Day 2" operations of a Cloud Engineer: **Rolling Actions**. We explore how to take that new template and apply it to the existing fleet in a controlled, automated, and zero-downtime fashion using commands like `restart`, `replace`, and `start-update`.

---

## 🎯 ACE Exam Focus

### Frequently Tested Areas
*   **The Update Gap:** Understanding that `set-instance-template` updates the *group configuration* but not the *live instances*.
*   **Rolling Action Commands:** Distinguishing between `restart`, `replace`, and `start-update`.
*   **Capacity Logic:** Mastering `max-surge` (extra VMs) and `max-unavailable` (downed VMs).
*   **Replacement Methods:** `recreate` (keeps instance names) vs. `substitute` (new instance names).
*   **Canary Deployments:** Updating a small percentage (target size) of the fleet to test a new version.

### Common Exam Trap
❌ **Assuming `rolling-action restart` installs a new template.**
*   ✔ **Correct:** `restart` only power-cycles the VM. To get a new template/OS change, you usually need `replace` or `start-update`.

---

## 1. The Update Gap: Config vs. Reality

When you run `gcloud compute instance-groups managed set-instance-template`, the following happens:
1.  **Group Config:** Updated immediately.
2.  **New Instances:** Any VM added by the autoscaler *from now on* will use the new template.
3.  **Existing Instances:** **Nothing happens.** They continue running the old version until you trigger a rollout.

---

## 2. Controlled Rollouts: Rolling Actions

To update existing VMs, you use `rolling-action` commands. These allow you to manage releases from V1 to V2 without taking the entire service offline.

### A. `rolling-action restart`
*   **Behavior:** Performs a standard stop-and-start on the VMs.
*   **Usage:** Best for clearing memory leaks or refreshing an app that doesn't require an OS/Template change.
*   **Downtime:** Instances go down briefly, but the group stays up if you limit the number updated at once.

### B. `rolling-action replace`
*   **Behavior:** Deletes the existing instance and creates a brand-new one.
*   **Usage:** **Mandatory** when you have a new Instance Template (e.g., new OS, new code image).

### C. `rolling-action start-update`
*   **Behavior:** The most comprehensive command. It allows you to specify the new version and deployment parameters in one go.
*   **Usage:** Professional standard for CI/CD pipelines.

---

## 3. Capacity Controls: Max Surge & Max Unavailable

These flags determine the speed and safety of your rollout.

| Flag | Purpose | Logic |
| :--- | :--- | :--- |
| **`--max-surge`** | Controls "Over-provisioning" | Creates **extra** VMs (V2) before deleting old ones (V1). Ensures 100% capacity is maintained. |
| **`--max-unavailable`** | Controls "Under-provisioning" | Deletes old VMs (V1) before the new ones (V2) are ready. Reduces temporary capacity. |

**Example:** If you have 100 instances and set `--max-surge=5`, GCP will update 5 at a time, momentarily running 105 instances.

---

## 4. Replacement Methods: Name Persistence

When using `replace`, you must decide if the instance names should stay the same.

*   **`recreate` (Default for many stateful needs):**
    *   Deletes the VM but **keeps the same name**.
    *   *Analogy:* Renaming a file but keeping the filename.
*   **`substitute`:**
    *   Deletes the VM and creates a new one with a **completely new name** (new random suffix).
    *   *Analogy:* Deleting a file and creating a brand-new one.

---

## 5. Canary Deployments (Advanced Rollout)

A **Canary Deployment** allows you to test a new template on a small subset of your users.

### The Workflow:
1.  Target a small size (e.g., 10%).
2.  GCP updates only those instances to the "Canary Template" (V2).
3.  The other 90% stay on the "Standard Template" (V1).
4.  If the 10% perform well, you update the entire group to V2.

```bash
gcloud compute instance-groups managed rolling-action start-update [MIG_NAME] \
    --version template=V2-template \
    --canary-version template=V2-template,target-size=10%
```

---

## 6. Visual Representations

### Max Surge Rollout Logic
```text
Target Size: 4 VMs
Max Surge: 1
Max Unavailable: 0

[ V1 ] [ V1 ] [ V1 ] [ V1 ]          (Start)
[ V1 ] [ V1 ] [ V1 ] [ V1 ] + [ V2 ] (Surge 1)
[ V2 ] [ V1 ] [ V1 ] [ V1 ]          (Delete 1 old)
... repeats until ...
[ V2 ] [ V2 ] [ V2 ] [ V2 ]          (Complete)
```

### Canary Deployment Map
```text
            [ Load Balancer ]
             /            \
      [ 90% Traffic ]    [ 10% Traffic ]
             |                  |
      [ V1 Instances ]   [ V2 Instances ]
        (Stable)           (Canary/Test)
```

---

## 7. Mental Models & Analogies

### Analogy: The Apartment Complex (Replacement Methods)
*   **`recreate`**: You evict a tenant, gut the apartment, and remodel it. The address (Name) remains "Apt 4B."
*   **`substitute`**: You tear down the building and build a new one next door. The new building has a completely different address (Name).

### Analogy: The Relay Race (Max Surge)
*   **Max Surge**: The next runner starts running *before* the first runner hands off the baton. For a moment, two people are running for one spot.
*   **Max Unavailable**: The first runner stops, sits down, and waits for the second runner to wake up and start. For a moment, no one is running.

---

## 8. Common Mistakes & Exam Traps

### ❌ Mistake 1: Expecting `restart` to update code
*   **Problem:** You update the template and run `rolling-action restart`.
*   **Result:** The VMs reboot but still use the old disk/code.
*   **Fix:** Use `rolling-action replace` to trigger the use of the new template.

### ❌ Mistake 2: Setting Max Unavailable to 100%
*   **Problem:** Setting `--max-unavailable` equal to your group size.
*   **Result:** Complete downtime. Every VM is deleted at once.

---

## 🎯 10 Certification Practice Questions

**Q1. You have updated the instance template for a MIG, but the existing instances are still running the old version. Which command will automate the update to the new version across the fleet in a controlled way?**
A. `gcloud compute instances update`
B. `gcloud compute instance-groups managed rolling-action start-update`
C. `gcloud compute instance-groups managed set-instance-template`
D. `gcloud compute instances restart --all`
*   **✅ Answer: B**

**Q2. During a rolling update, you want to ensure that your application's total capacity never drops below 100%. Which parameter should you configure?**
A. `max-unavailable = 1`
B. `max-unavailable = 0`
C. `max-surge = 0`
D. `replacement-method = substitute`
*   **✅ Answer: B** (Setting unavailable to 0 means no VM can be down; surge must be used to provide new capacity).

**Q3. What is the difference between the `recreate` and `substitute` replacement methods?**
A. `recreate` is faster.
B. `substitute` preserves the instance names.
C. `recreate` preserves the instance names.
D. `substitute` does not delete the old VM.
*   **✅ Answer: C**

**Q4. You want to test a new version of your application on exactly 5% of your instances before rolling it out to the whole group. What is this called in GCP?**
A. Rolling Restart
B. Blue-Green Deployment
C. Canary Version
D. Opportunistic Update
*   **✅ Answer: C**

**Q5. You run a `rolling-action restart` on a MIG. What happens to the instances?**
A. They are deleted and recreated from the template.
B. They are stopped and started.
C. They are moved to a different zone.
D. Their machine type is upgraded.
*   **✅ Answer: B**

**Q6. You have 10 instances. You set `max-surge=2`. What is the maximum number of instances that might be running at any point during the update?**
A. 8
B. 10
C. 12
D. 2
*   **✅ Answer: C**

**Q7. Which command allows you to change the template and trigger the update process simultaneously?**
A. `rolling-action replace`
B. `rolling-action start-update`
C. `set-instance-template`
D. `resize`
*   **✅ Answer: B**

**Q8. True or False: If you use the `substitute` replacement method, the random 4-character suffix of your instance names will change.**
A. True
B. False
*   **✅ Answer: A**

**Q9. Why would a company use `max-unavailable` instead of `max-surge`?**
A. To avoid the temporary cost of running extra instances.
B. To make the update faster.
C. Because `max-surge` only works for stateless groups.
D. To ensure high availability.
*   **✅ Answer: A**

**Q10. You have performed a `set-instance-template` to V2. You want to update the instances manually, one by one, at your own pace. Which commands can you use? (Select two)**
A. `recreate-instances`
B. `update-instances`
C. `rolling-action start-update`
D. `gcloud config set`
*   **✅ Answer: A and B** (These target specific instances manually).

---

## ⚡ 30-Second Revision
*   **Rolling Action:** The bridge between a template change and a live update.
*   **`restart`**: Reboot only.
*   **`replace`**: New VM, new template.
*   **`max-surge`**: Extra VMs (safe but costs more).
*   **`max-unavailable`**: Missing VMs (risky but cheaper).
*   **Canary**: Test on a small `%` (target-size).
*   **`recreate`**: Keep names. **`substitute`**: Get new names. ☁️