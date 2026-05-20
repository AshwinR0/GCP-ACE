# GCP Associate Cloud Engineer - Updating Managed Instance Groups (MIGs)

## Overview

In a production environment, applications are never static. You will frequently need to deploy new versions of code, update security patches, or simply refresh instances. In Google Cloud, **Managed Instance Groups (MIGs)** provide sophisticated ways to update thousands of Virtual Machines with zero downtime.

This section covers the core update strategies: **Rolling Updates**, **Canary Deployments**, and **Rolling Restarts/Replacements**.

---

## 🎯 ACE Exam Focus

### Frequently Tested Areas
*   **Rolling Update:** The standard way to transition from an old Instance Template to a new one.
*   **Canary Testing:** Running two versions of an application simultaneously within the same group to test a small subset of traffic.
*   **Proactive vs. Opportunistic:** Knowing that "Proactive" (Automatic) is for immediate changes and "Opportunistic" is for low-priority updates.
*   **Max Surge & Max Unavailable:** Critical settings that control capacity during an update.
*   **Restart vs. Replace:** The difference between a simple OS reboot (Restart) and a full VM recreation (Replace).

### Common Exam Trap
❌ **Confusing Max Surge with Max Unavailable.**
*   ✔ **Max Surge:** Extra VMs created *above* the target size (increases cost temporarily).
*   ✔ **Max Unavailable:** VMs taken *offline* (reduces capacity temporarily).

---

## 1. Update Types: Rolling Update vs. Restart/Replace

| Feature | Rolling Update | Rolling Restart/Replace |
| :--- | :--- | :--- |
| **Change Template?** | **YES.** Moves to a new v2 template. | **NO.** Uses the existing template. |
| **Use Case** | Deploying new application code. | Applying OS patches or clearing memory leaks. |
| **Downtime** | None (if configured correctly). | None (if configured correctly). |
| **Options** | Canary, Proactive, Opportunistic. | Max Surge (Replace only), Max Unavailable. |

---

## 2. The "When": Proactive vs. Opportunistic

This defines the **Update Policy**.

*   **Proactive (Automatic):** The MIG actively starts replacing VMs as soon as you save the update. This is the standard for deployments.
*   **Opportunistic:** The MIG **waits.** It will only apply the new template when an instance is naturally replaced (e.g., if the VM crashes or if the autoscaler adds a new VM).

---

## 3. The "How Much": Max Surge & Max Unavailable

These controls ensure your application stays online during the update.

### ✅ Max Surge
*   **Definition:** The maximum number of *extra* instances that can be created during the update.
*   **Logic:** Create the "New" VM before deleting the "Old" one.
*   **Pro:** Zero capacity loss.
*   **Con:** Higher temporary cost (billing for extra VMs).

### ✅ Max Unavailable
*   **Definition:** The maximum number of instances that can be *offline* during the update.
*   **Logic:** Delete the "Old" VM before creating the "New" one.
*   **Pro:** No extra cost.
*   **Con:** Temporary drop in performance/capacity.

---

## 4. Advanced Update Strategies

### A. Canary Deployment
*   **What it is:** Testing a new version (v2) on a small percentage of instances (e.g., 10%) while the rest stay on v1.
*   **Process:** 
    1. Specify the new template.
    2. Set the "Target size" for the canary (e.g., 1 instance).
    3. Verify logs/performance.
    4. If okay, roll out to 100%.

### B. Rolling Restart/Replace
Sometimes you just need to "turn it off and on again."
*   **Restart:** Shuts down and starts up the same VM (Preserves local disk).
*   **Replace:** Deletes the VM and creates a brand new one from the template (Fresh state).

---

## 5. Visual Representations

### Update Flow: Max Surge = 2 (Target Size = 10)
```text
[ Current State ] : 10 VMs (v1)
          |
[ Step 1 ] : Create 2 NEW VMs (v2) ---> Total = 12 VMs
          |
[ Step 2 ] : Delete 2 OLD VMs (v1) ---> Total = 10 VMs
          |
[ Step 3 ] : Repeat until 100% v2
```

### Update Flow: Max Unavailable = 2 (Target Size = 10)
```text
[ Current State ] : 10 VMs (v1)
          |
[ Step 1 ] : Delete 2 OLD VMs (v1) ---> Total = 8 VMs
          |
[ Step 2 ] : Create 2 NEW VMs (v2) ---> Total = 10 VMs
          |
[ Step 3 ] : Repeat until 100% v2
```

---

## 6. Mental Models & Analogies

### Analogy: The Restaurant Staff
*   **Rolling Update:** Hiring 2 new chefs trained on a new menu (v2) and retiring 2 old chefs (v1) at a time.
*   **Canary:** Putting 1 new chef in the kitchen to see if customers like the new food before firing everyone else.
*   **Max Surge:** Paying for 2 extra temporary chefs so the kitchen stays full while the transition happens.
*   **Max Unavailable:** Telling 2 chefs to stay home while their replacements get ready (the kitchen is short-staffed for a bit).

---

## 7. Step-by-Step Workflow: Update VMs

1.  Navigate to **Compute Engine > Instance Groups**.
2.  Select your MIG.
3.  Click **Update VMs**.
4.  Choose a **New Instance Template**.
5.  (Optional) Toggle **Canary testing** and set the number of instances to test.
6.  Open **Update Configuration**:
    *   Set **Proactive** for immediate rollout.
    *   Set **Max Surge** (e.g., 1) to avoid capacity loss.
    *   Set **Max Unavailable** (e.g., 0) if you want zero downtime.
7.  Click **Update**.

---

## 8. Common Mistakes & Exam Traps

### ❌ Mistake 1: Setting Max Unavailable = 100%
*   **Problem:** You have 10 instances and set Max Unavailable to 10.
*   **Result:** The MIG deletes every instance at once. Your website goes **OFFLINE**.

### ❌ Mistake 2: Opportunistic Updates for Critical Fixes
*   **Problem:** You have a security patch and use "Opportunistic."
*   **Result:** The update might not happen for weeks unless the VMs crash or you manually kill them. Use **Proactive** for critical updates.

---

## 🎯 10 Certification Practice Questions

**Q1. You want to update your MIG to a new version of your application. You want to ensure that the group never has fewer than the target number of instances during the update. Which setting should you use?**
A. Max Unavailable = 1
B. Max Surge = 1 (and Max Unavailable = 0)
C. Opportunistic update
D. Rolling Restart
*   **✅ Answer: B**

**Q2. Which update policy should you choose if you want the MIG to update instances only when they are being replaced or restarted for other reasons?**
A. Proactive
B. Automatic
C. Opportunistic
D. Scalable
*   **✅ Answer: C**

**Q3. You want to run two different instance templates in a single MIG to test a small percentage of traffic on a new version. What is this called?**
A. Blue-Green Deployment
B. Rolling Update
C. Canary Testing
D. Split-horizon scaling
*   **✅ Answer: C**

**Q4. What is the main difference between a Rolling Update and a Rolling Replace?**
A. Rolling Update changes the template; Rolling Replace uses the same template.
B. Rolling Update costs more.
C. Rolling Replace is always opportunistic.
D. There is no difference.
*   **✅ Answer: A**

**Q5. You have 100 instances. You set Max Surge to 10. How many instances might be running at the peak of the update process?**
A. 90
B. 100
C. 110
D. 10
*   **✅ Answer: C**

**Q6. You are performing a Rolling Restart on a MIG. Which setting controls how many instances can be rebooting at the same time?**
A. Max Surge
B. Max Unavailable
C. Initial Delay
D. Cool-down period
*   **✅ Answer: B**

**Q7. True or False: In a Canary Deployment, you can specify exactly how many instances should use the new template.**
A. True
B. False
*   **✅ Answer: A**

**Q8. Why would an engineer choose a Rolling Replace over a Rolling Restart?**
A. To clear local disk data and ensure a "fresh" start from the template.
B. Because restarts are not allowed on GCP.
C. Because replacing is faster than restarting.
D. To change the Instance Template.
*   **✅ Answer: A**

**Q9. Which parameter allows you to specify that instance names should not change during an update?**
A. Sticky Names
B. Retain Instance Names
C. Permanent Hostnames
D. Static IDs
*   **✅ Answer: B**

**Q10. You set Max Unavailable to 2 on a MIG with 5 instances. How many instances are guaranteed to be online and serving traffic during the update?**
A. 5
B. 2
C. 3
D. 0
*   **✅ Answer: C**

---

## ⚡ 30-Second Revision
*   **Rolling Update:** Gradually shift to a new template.
*   **Canary:** v1 and v2 run together for testing.
*   **Proactive:** Start update NOW.
*   **Opportunistic:** Update when convenient.
*   **Max Surge:** Temporary *extra* capacity.
*   **Max Unavailable:** Temporary *reduced* capacity.
*   **Rolling Replace:** Trash the VM and build a new one from the same template. ☁️