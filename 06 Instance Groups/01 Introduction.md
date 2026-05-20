# GCP Associate Cloud Engineer - Instance Groups (MIGs & Unmanaged)

## Overview
This section covers **Instance Groups**, the primary mechanism in Google Cloud for managing clusters of Virtual Machines as a single entity. Instead of managing individual VMs, instance groups allow you to apply configurations, scaling rules, and updates to a whole fleet simultaneously.

For the ACE exam, understanding the difference between **Managed Instance Groups (MIGs)** and **Unmanaged Instance Groups** is critical for answering questions about High Availability (HA), disaster recovery, and cost-effective scaling.

---

## 🎯 ACE Exam Focus

### Frequently Tested Areas
*   **MIG vs. Unmanaged:** Knowing that MIGs are for identical VMs and Unmanaged are for heterogeneous VMs.
*   **Instance Templates:** The prerequisite for creating any Managed Instance Group.
*   **Auto-healing:** How health checks trigger the replacement of failed instances.
*   **Auto-scaling:** Adding/removing VMs based on CPU, load, or other metrics.
*   **Regional vs. Zonal:** Choosing Regional MIGs for the highest availability.
*   **Update Strategies:** Distinguishing between Rolling Updates and Canary Deployments.

### Common Exam Trap
❌ **Using Unmanaged Instance Groups for Auto-scaling.**
*   ✔ **Correct:** Unmanaged groups **CANNOT** auto-scale or auto-heal. If a question mentions "different kinds of VMs" and "auto-scaling," it is a trick—you can't have both in one group.

---

## 1. What is an Instance Group?

An Instance Group is a collection of virtual machine instances managed as a **single entity**. Instead of taking actions on VM-1, VM-2, and VM-3 individually, you take actions on "Web-Server-Group."

### Why use them?
*   **Simplicity:** Manage 100 VMs as easily as one.
*   **Lifecycle Sync:** All VMs in the group share the same lifecycle goals.

---

## 2. Managed Instance Groups (MIGs)

MIGs are the "bread and butter" of GCP compute scaling. They use an **Instance Template** to create a set of **identical** virtual machines.

### Key Characteristics:
| Feature | Description |
| :--- | :--- |
| **Identicality** | Every VM has the same machine type, image, and configuration. |
| **Template-Based** | You cannot create a MIG without an Instance Template. |
| **Auto-healing** | Uses Health Checks to detect and replace crashed/unresponsive VMs. |
| **Auto-scaling** | Automatically grows or shrinks the fleet based on user load. |
| **Managed Releases** | Support for zero-downtime updates (Rolling/Canary). |
| **High Availability** | Can be **Regional**, spreading VMs across multiple zones to survive a zone outage. |

---

## 3. Unmanaged Instance Groups

These are groups of "misfit" VMs that do not share the same configuration.

### Key Characteristics:
*   **Diverse Configs:** VMs can have different hardware, different OS images, and different settings.
*   **No Automation:** You do **NOT** get auto-scaling, auto-healing, or managed updates.
*   **Use Case:** Only used when you have existing, differently-configured VMs that you need to group together (e.g., for a legacy load balancer).
*   **Recommendation:** **Not recommended** for modern cloud-native architectures.

---

## 4. Managed Instance Group Features (Deep Dive)

### A. Self-Healing (Auto-healing)
1.  **Health Check:** A "ping" or request sent to the application (e.g., on Port 80).
2.  **Detection:** If the app stops responding, the MIG marks the instance as unhealthy.
3.  **Recreation:** The MIG deletes the bad VM and launches a fresh one from the template.

### B. Scalability & Availability
*   **Auto-scaling:** Dynamically adjust the number of instances (e.g., 2 during the night, 20 during the day).
*   **Regional MIGs:** By selecting "Regional," the MIG places instances in multiple zones (e.g., `us-east1-a`, `us-east1-b`, `us-east1-c`). If one zone fails, the others take over.

### C. Update Strategies
MIGs allow you to update your application code/OS without taking the whole site offline.

| Strategy | Behavior |
| :--- | :--- |
| **Rolling Update** | Updates instances one-by-one or in small batches (e.g., 2 at a time). |
| **Canary Deployment** | Updates a tiny percentage (e.g., 10%) to test the new version. If it works, you roll it out to the rest. |

---

## 5. Visual Representations

### MIG Architecture Map
```text
      [ Instance Template ]
               |
               V
   [ Managed Instance Group ]
   /           |            \
[ VM-1 ]    [ VM-2 ]     [ VM-3 ]
 (Identical) (Identical) (Identical)
```

### Regional High Availability (HA)
```text
      [ Regional MIG: us-central1 ]
       /           |            \
   [ Zone A ]   [ Zone B ]   [ Zone C ]
      |            |            |
    [ VM ]       [ VM ]       [ VM ]
```

---

## 6. Mental Models & Analogies

### MIG: The Army of Clones
Think of a MIG like an army of identical clone soldiers. If one falls in battle (crashes), a new identical one is sent to replace it (auto-healing). If the battle gets harder, more clones are produced (auto-scaling). They all look and act exactly the same.

### Unmanaged: The Misfit Toys
Think of an Unmanaged group like a collection of different toys (a teddy bear, a toy car, and a puzzle). They are in the same box (the group), but you can't "clone" the teddy bear to make more, and if the car breaks, the puzzle doesn't automatically fix it.

---

## 7. Service Comparison Table

| Category | Managed Instance Group (MIG) | Unmanaged Instance Group |
| :--- | :--- | :--- |
| **Purpose** | Scalable, highly available clusters | Grouping existing, diverse VMs |
| **Managed Level** | Fully Managed | Partially Managed |
| **Scalability** | Automatic | **Manual only** |
| **Self-Healing** | Yes (via Health Checks) | No |
| **Update Support** | Rolling / Canary | None |
| **ACE Relevance** | **High Priority** | Low Priority |

---

## 8. Common Mistakes & Exam Traps

### ❌ Mistake 1: Not using a Health Check
*   **Problem:** You enable auto-healing but don't configure a health check.
*   **Result:** The MIG can only see if the VM is "ON." It can't see if your web server software has crashed. Always attach a Health Check.

### ❌ Mistake 2: Zonal vs. Regional
*   **Trap:** Choosing a Zonal MIG for a mission-critical app.
*   **Correct:** Always use **Regional** MIGs for production to ensure one zone failure doesn't kill the whole app.

---

## 9. Real-World Use Case
**E-commerce Website (Black Friday):**
The company uses a **Regional MIG**. 
1.  **Normal Days:** 3 VMs running.
2.  **Black Friday:** Auto-scaling detects high CPU and adds 50 more VMs.
3.  **App Crash:** One VM's code freezes. Auto-healing detects the failure and replaces it.
4.  **Updates:** On Saturday, they do a **Rolling Update** to put a "Thank You" banner on the site without taking it down.

---

## 🎯 10 Certification Practice Questions

**Q1. You need to create a cluster of identical VMs that can automatically increase in number when CPU usage exceeds 70%. Which should you use?**
A. Unmanaged Instance Group
B. Managed Instance Group
C. Single VM with high specs
D. Placement Group
*   **✅ Answer: B**

**Q2. Which resource is mandatory to create before you can launch a Managed Instance Group?**
A. Snapshot
B. Custom Image
C. Instance Template
D. Firewall Rule
*   **✅ Answer: C**

**Q3. You have a Managed Instance Group. One of the VMs is running, but the web server software inside it has crashed. What feature should you use to fix this automatically?**
A. Auto-scaling
B. Auto-healing with a Health Check
C. Rolling Updates
D. Live Migration
*   **✅ Answer: B**

**Q4. To achieve the highest availability for your application in the `us-central1` region, what type of MIG should you create?**
A. Zonal MIG
B. Global MIG
C. Regional MIG
D. Shared-core MIG
*   **✅ Answer: C**

**Q5. You want to test a new version of your application on only 2 out of your 10 instances in a MIG. What is this strategy called?**
A. Rolling Update
B. Canary Deployment
C. Blue-Green Deployment
D. Hot Swap
*   **✅ Answer: B**

**Q6. True or False: Unmanaged Instance Groups support auto-scaling.**
A. True
B. False
*   **✅ Answer: B**

**Q7. What happens if an instance in a MIG with auto-healing enabled fails its health check?**
A. It is restarted.
B. It is suspended.
C. It is deleted and a new instance is created from the template.
D. An alert is sent to the administrator but no action is taken.
*   **✅ Answer: C**

**Q8. You have three legacy VMs, each with different OS versions and hardware requirements. You need to group them to be used with a Load Balancer. Which group should you use?**
A. Managed Instance Group
B. Unmanaged Instance Group
C. Instance Template
D. Tenant Group
*   **✅ Answer: B** (Key: different configurations).

**Q9. Which update method in a MIG allows you to update a specific percentage of instances at a time?**
A. All-at-once
B. Rolling Update
C. Manual Update
D. Snapshot Restore
*   **✅ Answer: B**

**Q10. Why are Regional MIGs preferred over Zonal MIGs for production workloads?**
A. They are cheaper.
B. They allow for faster CPU performance.
C. They protect the application against a single zone failure.
D. They do not require an Instance Template.
*   **✅ Answer: C**

---

## ⚡ 30-Second Revision
*   **MIG:** Identical VMs + Template + Auto-scaling + Auto-healing.
*   **Unmanaged:** Different VMs + No Automation.
*   **Auto-healing:** Needs a **Health Check**.
*   **Regional MIG:** Best for **High Availability**.
*   **Canary:** Test new code on a small group first.
*   **Rolling Update:** Update the fleet in steps. ☁️