# GCP Associate Cloud Engineer - Compute Engine (GCE) Fundamentals

## Overview
Google Compute Engine (GCE) is the **Infrastructure as a Service (IaaS)** heart of GCP. It allows you to provision and manage Virtual Machines (VMs) on Google’s global infrastructure. Instead of buying physical hardware, you "rent" virtualized CPUs, RAM, and storage.

For the ACE exam, Compute Engine is a primary topic. You must understand how to create, configure, and manage these instances, as well as how to optimize costs and choose the right hardware for specific workloads.

---

## 🎯 ACE Exam Focus

### Frequently Tested Areas
*   **Machine Configurations:** Choosing between General Purpose, Compute-Optimized, and Memory-Optimized.
*   **VM Lifecycle:** Understanding the difference between Stop, Suspend, Reset, and Delete.
*   **Images:** Using Public images vs. creating Custom images for faster scaling.
*   **Networking Basics:** Enabling HTTP/HTTPS traffic via firewall checkboxes during creation.
*   **SSH Access:** How to connect to a Linux VM via the Cloud Console.
*   **Labels:** Using metadata to organize and filter resources for billing and management.

### Common Exam Trap
❌ **Billing Misconception:** Thinking you aren't charged for a "Stopped" VM. 
*   ✔ **Reality:** You stop paying for **CPU and RAM**, but you still pay for the **Persistent Disk (Storage)** and any reserved **Static IPs**.

---

## 1. What is Compute Engine?
Compute Engine is GCP’s service for running Virtual Machines. 

### Key Capabilities:
*   **Lifecycle Management:** Create, start, stop, and delete instances.
*   **Scalability:** Use Managed Instance Groups (MIGs) to scale based on traffic (Auto-scaling).
*   **Storage:** Attach Persistent Disks (HDD/SSD) or Local SSDs.
*   **Networking:** Manage internal/external IPs and firewall rules.

---

## 2. Machine Configuration: The "Hardware"

When creating a VM, you must choose a **Machine Family** and a **Machine Type**.

### Machine Families
| Family | Purpose | Example Use Case |
| :--- | :--- | :--- |
| **General Purpose (E2, N2)** | Best price-performance ratio. | Web servers, dev environments, small databases. |
| **Compute-Optimized (C2)** | High performance per core. | Gaming, high-performance computing (HPC). |
| **Memory-Optimized (M2)** | Massive RAM capacity. | Large In-memory databases (SAP HANA), Analytics. |

### Understanding Machine Type Names (e.g., `e2-standard-2`)
*   **e2:** The Family (General purpose).
*   **standard:** The workload profile (Balanced CPU/RAM).
*   **2:** The number of vCPUs (Virtual CPUs).

---

## 3. Boot Disk & Images: The "Software"

The **Image** is the blueprint for your VM’s operating system.

1.  **Public Images:** Maintained by Google or vendors (e.g., Debian, Ubuntu, Windows Server, Red Hat).
2.  **Custom Images:** Created by you. You take a snapshot of a configured VM and save it to create identical copies later.

---

## 4. VM Lifecycle Operations

| Operation | What it does | Billing Note |
| :--- | :--- | :--- |
| **Start** | Boots the VM from the disk. | Full billing starts. |
| **Stop** | Shuts down the OS. | No CPU/RAM charge; **Disk charge continues.** |
| **Suspend** | "Pauses" the VM (saves RAM state to disk). | Small charge for saved state + Disk charge. |
| **Reset** | Hard reboot (clears RAM). | Billing continues. |
| **Delete/Terminate**| Permanently removes the VM. | All billing stops (unless disks are kept). |

---

## 5. VM Creation Workflow (Step-by-Step)

1.  **Navigation:** Go to `Compute Engine` > `VM Instances` > `Create`.
2.  **Basics:** Name the VM (e.g., `my-first-vm`) and add **Labels** (e.g., `env:dev`).
3.  **Location:** Select **Region** and **Zone** (Default: `us-central1-a`).
4.  **Hardware:** Choose **Machine Family** (General Purpose) and **Type**.
5.  **Software:** Choose **Boot Disk/Image** (Default: Debian).
6.  **Firewall:** Check **"Allow HTTP traffic"** to permit web access on port 80.
7.  **Launch:** Click **Create**.

---

## 6. Visual Representations

### VM Hardware Hierarchy
```text
[ VM Instance ]
      |
      |--- [ Machine Family: General Purpose (E2) ]
      |           |
      |           |--- [ Machine Type: e2-standard-2 ]
      |                     |-- 2 vCPUs
      |                     |-- 8 GB RAM
      |
      |--- [ Boot Disk: Debian Image ]
      |
      |--- [ Firewall: Allow HTTP (Port 80) ]
```

### SSH Connection Flow
```text
[ Browser / Console ] --(Encrypted Tunnel)--> [ GCP Project ] --(Internal Network)--> [ VM Instance ]
```

---

## 7. Mental Models & Analogies

### The "Rented Laptop" Analogy
Imagine you are renting a laptop from a store:
*   **Machine Type:** Choosing if you want a basic laptop (General), a gaming rig (Compute-optimized), or a workstation for 4K video (Memory-optimized).
*   **Image:** Choosing if you want the laptop to come with Windows, Linux, or a pre-installed set of tools you requested.
*   **Stop vs. Delete:** **Stopping** is like closing the lid and putting it in a locker (you still pay for the locker/storage). **Deleting** is returning it to the store permanently.

---

## 8. Important Notes & Best Practices

*   **⚠️ Cost Management:** Always **Stop** or **Delete** instances when finished with a lab to preserve your $300 free credit.
*   **Labels:** Essential for production. Use them to track which department owns which VM for billing reports.
*   **SSH:** If the SSH popup is blocked, check your browser settings to allow popups from `google.com`.

---

## 9. Common Mistakes & Exam Traps

### Mistake 1:
Trying to connect to a web server on a VM without checking the "Allow HTTP traffic" box.
*   ✔ **Correct:** You must explicitly open Port 80 in the firewall settings or via the checkbox.

### Mistake 2:
Confusing vCPU with physical CPU.
*   ✔ **Correct:** vCPUs are virtualized threads. A machine with `2 vCPUs` might be sharing a physical core with other users.

---

## 🎯 10 Certification Practice Questions

**Q1. You need to run a high-performance database that requires 512GB of RAM. Which machine family should you choose?**
A. General Purpose
B. Compute-Optimized
C. Memory-Optimized
D. Shared-Core
*   **✅ Answer: C** (Memory-optimized is designed for large RAM workloads).

**Q2. You have finished your work for the day on a development VM. You want to ensure you are not billed for CPU resources but want to keep your files. What should you do?**
A. Delete the instance
B. Reset the instance
C. Stop the instance
D. Suspend the instance
*   **✅ Answer: C** (Stopping stops CPU/RAM billing but preserves the disk).

**Q3. Which of the following is an example of a Public Image?**
A. A snapshot you took of your production server.
B. A Debian 11 OS provided by Google.
C. A disk image shared by your teammate.
D. A backup of a Windows VM.
*   **✅ Answer: B.**

**Q4. What does the "2" represent in the machine type `e2-standard-2`?**
A. 2 GB of RAM
B. 2 vCPUs
C. 2 Network Interfaces
D. 2 Local SSDs
*   **✅ Answer: B.**

**Q5. You are unable to access your web application via its external IP address. You confirmed the app is running. What is the most likely cause?**
A. The VM doesn't have an Internal IP.
B. The Firewall is blocking HTTP traffic.
C. You didn't add a Label to the VM.
D. The machine type is too small.
*   **✅ Answer: B.**

**Q6. You want to organize your VMs by department (Sales, Finance) for billing purposes. Which feature should you use?**
A. Regions
B. Machine Types
C. Labels
D. SSH keys
*   **✅ Answer: C.**

**Q7. What is the main difference between a Reset and a Stop/Start?**
A. Reset is free; Stop/Start costs money.
B. Reset clears the memory but doesn't shut down the VM entirely; Stop/Start involves a full power cycle.
C. Reset deletes the boot disk.
D. There is no difference.
*   **✅ Answer: B.**

**Q8. You need to run a web server with a balanced amount of CPU and RAM for a medium-traffic site. Which family is most appropriate?**
A. C2
B. M2
C. E2
D. A2 (GPU)
*   **✅ Answer: C.**

**Q9. Which tool is most commonly used to run command-line instructions inside a Linux VM directly from the GCP Console?**
A. RDP
B. FTP
C. SSH
D. Telnet
*   **✅ Answer: C.**

**Q10. True or False: If you delete a VM, the internal and external IP addresses associated with it are always permanently reserved for you.**
A. True
B. False
*   **✅ Answer: B** (IPs are released unless you specifically reserve them as Static IPs).

---

## ⚡ 30-Second Revision
*   **Compute Engine:** Virtual Machines (IaaS).
*   **Machine Families:** E2 (General), C2 (Compute), M2 (Memory).
*   **Images:** OS blueprints (Public or Custom).
*   **Lifecycle:** Stop = No CPU pay, Yes Disk pay. Delete = No pay.
*   **Networking:** Check "Allow HTTP" for web servers.
*   **Labels:** Use for organization and billing. ☁️