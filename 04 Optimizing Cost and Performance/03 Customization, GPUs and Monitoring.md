# GCP Associate Cloud Engineer - Advanced GCE: Customization, GPUs, and Monitoring

## Overview
This section dives into the advanced configuration of Google Compute Engine (GCE). We move beyond standard "predefined" machine shapes to explore **Custom Machine Types** for precise resource allocation and **GPUs** for high-performance computing. Additionally, we cover critical operational knowledge: how to manage VM lifecycles (editing vs. stopping), how resources are organized globally vs. zonally, and the limitations of default monitoring.

---

## 🎯 ACE Exam Focus

### Frequently Tested Areas
*   **Custom Machine Types:** Knowing which families support them (E2, N2, N1).
*   **GPU Restrictions:** No Live Migration; maintenance policy must be set to **Terminate**.
*   **VM Lifecycle:** You **MUST stop** a VM to change its machine type (vCPU/RAM).
*   **Resource Scopes:** Distinguishing that VMs are **Zonal**, while Images and Instance Templates are **Global**.
*   **Monitoring Agent:** Knowing that **Memory** and **Disk Space** metrics require the Cloud Monitoring Agent.

### Common Exam Trap
❌ **Trying to see "Memory Usage" in the basic GCP Console charts.**
*   ✔ **Reality:** Default monitoring only shows CPU, Disk IO, and Network. For Memory/Disk space, you must install the **Cloud Monitoring Agent**.

---

## 1. Custom Machine Types

When predefined VM shapes (e.g., `e2-standard-4`) don't fit your specific workload, you can create a **Custom Machine Type**.

### Key Characteristics
*   **Flexibility:** Independently choose the number of vCPUs and the amount of GB of memory.
*   **Supported Families:** Primarily available for **E2, N2, and N1** series.
*   **Billing:** You are charged per vCPU and per GB of RAM provisioned.
*   **OS Support:** Supports all major OSs (CentOS, Debian, RHEL, Ubuntu, Windows).

### 🧠 Analogy: The Tailored Suit
*   **Predefined:** Buying a "Large" shirt off the rack. It's okay, but maybe the sleeves are too long.
*   **Custom:** A tailor-made suit. You specify the exact chest size and arm length so you don't pay for extra fabric (resources) you don't need.

---

## 2. GPUs (Graphics Processing Units)

GPUs are used to accelerate math-intensive and graphics-heavy workloads, such as AI, Machine Learning, and Video Rendering.

### Implementation Methods
1.  **GPU Machine Family:** Choosing a dedicated family like **A2** (Nvidia A100) or **G2**.
2.  **Adding GPUs to Normal Families:** Adding a GPU to an existing family (e.g., **N1**). 
    *   *Note:* You **cannot** add GPUs to E2 (Shared Core) or Memory-Optimized families.

### ⚠️ Critical Restrictions for the Exam
| Feature | GPU Instance Requirement |
| :--- | :--- |
| **Live Migration** | **NOT SUPPORTED.** |
| **Maintenance Policy**| Must be set to **Terminate VM Instance**. |
| **Availability Policy**| **Automatic Restart** should be **ON** to recover after maintenance. |
| **Software** | You must use images with **GPU Libraries** (CUDA/Drivers) pre-installed. |

---

## 3. VM Operational Management

### Changing Machine Types
*   **Rule:** You cannot change vCPU or RAM on a running VM.
*   **Workflow:** 
    1.  Stop VM.
    2.  Click Edit.
    3.  Change Machine Type.
    4.  Start VM.

### Organization & Filtering
*   **Projects:** Every VM is owned by a Project. 
*   **Labels:** Best practice for large-scale management. You can filter 100+ VMs by labels (e.g., `env:prod`, `dept:finance`).
*   **Filters:** In the console, you can filter by Name, Zone, Status, IPs, and Network.

---

## 4. Resource Scope: Global vs. Zonal

Understanding where a resource "lives" is vital for disaster recovery planning.

| Resource | Scope | Exam Insight |
| :--- | :--- | :--- |
| **VM Instance** | **Zonal** | Tied to a specific data center area. |
| **Image** | **Global** | Can be used to launch a VM in ANY region/zone. |
| **Instance Template** | **Global** | Reusable blueprint across the whole project. |

```text
[ Global Scope ]
      |
      |--- Images
      |--- Instance Templates
      |
      [ Regional Scope ]
            |
            |--- Subnets
            |
            [ Zonal Scope ]
                  |
                  |--- VM Instances
                  |--- Persistent Disks
```

---

## 5. Monitoring: Basic vs. Advanced

GCP provides "Basic Monitoring" by default, but it has blind spots.

| Metric Type | Basic (Default) | Cloud Monitoring Agent (Required) |
| :--- | :--- | :--- |
| **CPU Utilization** | ✅ Yes | ✅ Yes |
| **Network (In/Out)** | ✅ Yes | ✅ Yes |
| **Disk IOPS / Throughput**| ✅ Yes | ✅ Yes |
| **Memory Utilization** | ❌ No | ✅ Yes |
| **Disk Space (Internal)** | ❌ No | ✅ Yes |

---

## 🎯 10 Certification Practice Questions

**Q1. A data science team needs a VM with exactly 6 vCPUs and 22 GB of RAM. Predefined types don't match. What should you do?**
A. Create a custom machine type using the E2 family.
B. Use an N2-standard-8 and let the extra resources sit idle.
C. Use a shared-core instance.
D. You cannot customize RAM and CPU independently in GCP.
*   **✅ Answer: A**

**Q2. You are running a VM with an attached GPU. Google needs to perform maintenance on the physical host. What will happen to your VM?**
A. It will live-migrate to a new host with no downtime.
B. It will be terminated and automatically restarted if configured.
C. It will be suspended and resumed automatically.
D. Google will delay maintenance indefinitely.
*   **✅ Answer: B** (GPUs do not support live migration).

**Q3. You need to increase the RAM of a production VM that is currently running. What is the correct sequence of actions?**
A. Edit the VM settings, change the RAM, and click Save.
B. Take a snapshot, delete the VM, and recreate it with more RAM.
C. Stop the VM, edit the machine type, save, and start the VM.
D. Live-migrate the VM to a larger machine type.
*   **✅ Answer: C**

**Q4. Which of these metrics is NOT available in the GCP Console by default without installing an agent?**
A. CPU Utilization
B. Disk Write Bytes
C. Memory Utilization
D. Network Packets
*   **✅ Answer: C**

**Q5. You want to launch identical web servers in Mumbai (`asia-south1`) and London (`europe-west2`) using the same custom boot disk. What resource allows this?**
A. A Zonal Disk
B. A Global Image
C. A Regional Subnet
D. A Local SSD
*   **✅ Answer: B**

**Q6. You want to quickly identify all VMs belonging to the "Marketing" department for billing analysis. What is the best practice?**
A. Use a naming convention like `mkt-vm-1`.
B. Assign a `dept:marketing` label to each VM and filter by label.
C. Put all Marketing VMs in a separate region.
D. Use different machine types for each department.
*   **✅ Answer: B**

**Q7. On which machine family is the "Add GPU" button disabled in the GCP Console?**
A. N1
B. E2
C. A2
D. N2
*   **✅ Answer: B** (E2 and shared-core do not support GPUs).

**Q8. True or False: Instance Templates are global resources by default.**
A. True
B. False
*   **✅ Answer: A**

**Q9. To ensure your GPU-attached VM comes back online after a host maintenance event, which availability policy should be set to "On"?**
A. Live Migration
B. Preemptibility
C. Automatic Restart
D. Provisioning Model: Spot
*   **✅ Answer: C**

**Q10. Why would a Cloud Engineer install the "Cloud Monitoring Agent" on a GCE instance?**
A. To enable SSH access.
B. To see internal disk space and memory usage metrics.
C. To reduce the cost of the VM.
D. To enable live migration for GPUs.
*   **✅ Answer: B**

---

## ⚡ 30-Second Revision
*   **Custom Machine Types:** Mix/match CPU/RAM on E2, N2, N1.
*   **GPUs:** AI/ML power. **NO Live Migration.** Set to **Terminate**.
*   **Edit VM:** Always **STOP** first.
*   **Labels:** The best way to filter/organize.
*   **Scope:** Image = Global; VM = Zonal.
*   **Monitoring Agent:** Required for Memory and Disk space usage. ☁️