# GCP Associate Cloud Engineer - GCE Billing & High Availability Policies

## Overview
This section covers the financial and operational mechanics of Google Compute Engine (GCE). We explore how Google bills you for resources, how to protect your wallet using **Budget Alerts**, and how Google ensures your applications stay online during physical hardware maintenance via **Live Migration**.

For the ACE exam, you must distinguish between what you pay for when a VM is "Stopped" versus "Running," and you must understand the "Availability Policy" settings that govern VM behavior during infrastructure updates.

---

## 🎯 ACE Exam Focus

### Frequently Tested Areas
*   **Billing Granularity:** Per-second billing with a **1-minute minimum**.
*   **Stopped VM Costs:** You stop paying for CPU, but **Storage (Persistent Disks) and Static IPs** still incur costs.
*   **Budget Alerts:** Thresholds (e.g., 50%, 90%, 100%) and notification methods.
*   **Live Migration:** The default behavior of moving VMs to new hardware without downtime.
*   **Exclusions:** Knowing that **Preemptible/Spot VMs** and **GPUs** cannot be live-migrated.
*   **Availability Policies:** Configuring "On-host maintenance" (Migrate vs. Terminate).

### Common Exam Trap
❌ **Believing Live Migration moves a VM to a different region.**
*   ✔ **Correct:** Live Migration only moves a VM to a different host **within the same zone**.

---

## 1. Compute Engine Billing Mechanics

GCP uses a "Pay-as-you-go" model with high precision, but there are specific rules about the first few seconds of usage.

| Feature | Rule |
| :--- | :--- |
| **Minimum Charge** | **1 Minute.** If you run a VM for 10 seconds and delete it, you pay for 60 seconds. |
| **Incremental Charge**| **Per-Second.** After the first minute, you are billed for exactly how many seconds the VM runs. |
| **Stopped VM Status** | **Compute (vCPU/RAM) = $0.** Persistent Disk (Storage) = **Full Price.** |
| **Suspended VM Status**| No Compute charge, but you pay for the storage of the RAM state + Persistent Disk. |

---

## 2. Cost Monitoring & Control

### Budget Alerts
A "Budget" in GCP is a monitoring tool, not a hard limit that shuts down your services.
*   **Thresholds:** You define percentages of your budget (e.g., 50%, 90%, 100%) to trigger alerts.
*   **Notifications:** Sends emails to Billing Admins and Users.
*   **Scope:** Can be applied to specific projects or the entire billing account.

### Billing Export
For deep analysis, you can export billing data to:
1.  **Cloud Storage:** As CSV or JSON files.
2.  **BigQuery:** (Recommended) Allows you to run SQL queries to find "hidden" costs or trends.

---

## 3. High Availability: Live Migration

Google occasionally needs to patch the physical "host" (the actual metal server) that your VM lives on. **Live Migration** prevents your VM from going down during this time.

*   **Process:** Google moves your running VM instance to another host in the same zone.
*   **Downtime:** Zero. The application keeps running, and the state is preserved.
*   **Attributes:** IP addresses, metadata, and properties remain unchanged.
*   **Supported:** Works even for VMs with **Local SSDs**.
*   **NOT Supported:**
    1.  **Preemptible / Spot Instances.**
    2.  **GPUs** (Graphics Processing Units).

---

## 4. Availability Policies

These settings determine how the VM behaves during maintenance or crashes. They are found under **Management > Availability Policy**.

### A. On-host Maintenance
| Option | Behavior (The Default is "Migrate") |
| :--- | :--- |
| **Migrate** | Moves the VM to a healthy host. Application stays UP. |
| **Terminate** | Shuts down the VM. Application goes DOWN. |

### B. Automatic Restart
*   **Function:** If the VM crashes due to a hardware failure or software bug (non-user initiated), GCP will automatically try to start it back up.
*   **Default:** **On.**

---

## 5. Visual Representations

### Live Migration Workflow
```text
      [ Physical Host A ]               [ Physical Host B ]
             |                                 |
      [ Running VM ] ---------------------> [ Running VM ]
             |        (Live Migration)         |
             |                                 |
      [ Needs Update ]                  [ Healthy & New ]
```

### The Billing Timeline
```text
Start VM (0s)      60s Mark            95s Mark         Stop VM
     |----------------|-------------------|----------------|
     [  1-Min Min.  ] [  Billed by Sec  ] [ Billing Stops* ]
                                          *Disk billing continues
```

---

## 6. Mental Models & Analogies

### Analogy: The Fish Bowl (Live Migration)
Imagine your VM is a fish in a bowl. Google needs to clean the glass (Maintenance). 
*   **Migrate:** Google places a second bowl next to the first and gently pours the water and the fish into the new bowl. The fish never leaves the water and never stops swimming.
*   **Terminate:** Google takes the fish out, cleans the bowl, and puts the fish back later. The fish (your app) stops breathing for a while.

---

## 7. Common Mistakes & Exam Traps

### ❌ Mistake 1: Expecting Alerts to Stop VMs
*   **Problem:** Thinking a $10 budget alert will shut down your VM when you hit $10.
*   **Fact:** Budget alerts are **informational only**. You must use Pub/Sub and Cloud Functions if you want to automatically kill VMs based on cost.

### ❌ Mistake 2: Missing the "GPU" Exception
*   **Problem:** Assuming a high-end Machine Learning VM with a GPU will live-migrate.
*   **Fact:** **GPUs cannot be live-migrated.** They must be set to "Terminate" during maintenance.

---

## 🎯 10 Certification Practice Questions

**Q1. You start a VM, run it for 45 seconds, and then delete it. How long are you billed for?**
A. 45 seconds
B. 1 minute
C. 2 minutes
D. 0 seconds
*   **✅ Answer: B** (GCE has a 1-minute minimum).

**Q2. You have a VM that is currently in the "Stopped" state. Which of the following are you still being charged for?**
A. vCPU and RAM
B. GPU processing
C. Persistent Disk storage
D. Network egress
*   **✅ Answer: C**

**Q3. You want to receive an email when your GCP project spending reaches 90% of your $500 monthly limit. What should you configure?**
A. Billing Export to BigQuery
B. Quotas
C. Budget Alerts
D. Stackdriver Logging
*   **✅ Answer: C**

**Q4. Where can you perform complex SQL queries on your historical billing data?**
A. Cloud Storage
B. BigQuery (after setting up Billing Export)
C. Compute Engine Console
D. Identity and Access Management (IAM)
*   **✅ Answer: B**

**Q5. During a scheduled hardware upgrade, Google needs to move your VM. You want the VM to remain online without any interruption. Which setting should you use?**
A. On-host maintenance: Terminate
B. On-host maintenance: Migrate
C. Preemptibility: On
D. Automatic Restart: Off
*   **✅ Answer: B**

**Q6. Which of these resources CANNOT take advantage of Live Migration? (Select two)**
A. VMs with Local SSDs
B. Preemptible VMs
C. VMs with GPUs
D. Standard E2 Instances
*   **✅ Answer: B and C**

**Q7. A VM instance crashes because of a physical hardware failure in the Google Data Center. You want the VM to come back online automatically without manual intervention. What should you enable?**
A. Live Migration
B. Automatic Restart
C. Site Reliability Engineering (SRE)
D. Cloud Monitoring
*   **✅ Answer: B**

**Q8. Where do you find the configuration for Live Migration and Automatic Restart?**
A. VPC Network settings
B. Availability Policy (under Management)
C. IAM Roles
D. Metadata tags
*   **✅ Answer: B**

**Q9. True or False: Live Migration can move a VM from `us-central1-a` to `us-east1-b` to avoid a regional storm.**
A. True
B. False
*   **✅ Answer: B** (Live migration is within the same zone).

**Q10. What is the recommended best practice to avoid "billing surprises" at the end of the month?**
A. Never use Persistent Disks.
B. Create budget alerts at 50%, 90%, and 100%.
C. Only use Preemptible VMs.
D. Delete your project every Friday.
*   **✅ Answer: B**

---

## ⚡ 30-Second Revision
*   **Billing:** 1-min minimum, then per-second.
*   **Stopped VM:** Pay for Disk, not CPU.
*   **Budget Alerts:** Notifications only, does not stop services.
*   **BigQuery:** Go-to for billing analysis.
*   **Live Migration:** Same zone, zero downtime, default behavior.
*   **Exclusions:** No Live Migration for GPUs or Preemptible VMs.
*   **Availability Policy:** Migrate vs. Terminate + Automatic Restart. ☁️