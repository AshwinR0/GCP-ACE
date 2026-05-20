# GCP Associate Cloud Engineer - Compute Engine Cost Optimization

## Overview
In the cloud, cost management is as important as technical architecture. Google Cloud Platform (GCP) provides several built-in mechanisms to reduce the cost of Virtual Machines (VMs). This section covers the four primary cost-saving strategies for Compute Engine (GCE) and Google Kubernetes Engine (GKE): **Sustained Use Discounts**, **Committed Use Discounts**, **Preemptible VMs**, and **Spot VMs**.

For the ACE exam, you must be able to recommend the right billing model based on workload characteristics (e.g., Is it a 24/7 web server or a flexible batch job?).

---

## 🎯 ACE Exam Focus

### Frequently Tested Areas
*   **Sustained Use Discounts (SUD):** Knowing they are automatic and require no configuration.
*   **Committed Use Discounts (CUD):** Choosing between 1-year or 3-year terms for predictable workloads.
*   **Preemptible vs. Spot:** Understanding that Spot VMs are the modern version without the 24-hour limit.
*   **Use Cases:** Identifying that batch processing is ideal for Preemptible/Spot, while production databases need CUD.
*   **Exclusions:** Remembering that App Engine Flexible and Dataflow do **not** receive SUD/CUD.

### Common Exam Trap
❌ **Trying to "Convert" a Preemptible VM into a Regular VM.**
*   ✔ **Correct:** You cannot convert them. You must delete the preemptible instance and recreate it as a regular instance (though you can reuse the disk).

---

## 1. Sustained Use Discounts (SUD) - "The Automatic Reward"

SUDs are automatic discounts applied when you run a VM for a significant portion of a billing month.

*   **How it works:** If a VM runs for more than 25% of a month, GCP starts applying a discount (20% to 50%) on every incremental minute.
*   **Effort:** **Zero.** No manual setup or "opting-in" is required.
*   **Applicability:** GCE and GKE instances.
*   **⚠️ Exclusions (High Exam Relevance):**
    *   Machine Types: **E2** and **A2** families do not qualify.
    *   Services: **App Engine Flexible** and **Dataflow** do not qualify.

---

## 2. Committed Use Discounts (CUD) - "The Contract"

CUDs are for workloads with **predictable** resource needs. You commit to using a specific amount of CPU and RAM for a fixed term.

*   **Terms:** 1 year or 3 years.
*   **Discount:** Up to **70%** (Higher than SUD).
*   **Commitment Types:**
    1.  **Hardware:** Committing to vCPUs, RAM, GPUs, or Local SSDs.
    2.  **Software:** Committing to licenses (e.g., SLES, RHEL).
*   **Workflow:** Go to `Compute Engine` > `Committed Use Discounts` > `Purchase Commitment`.
*   **Best For:** 24/7 Production servers, primary databases, and stable web apps.

---

## 3. Preemptible & Spot VMs - "The Budget Choice"

These are "spare capacity" VMs that Google sells at a massive discount because they reserve the right to take them back at any time.

### Preemptible VMs (Legacy)
*   **Price:** Very cheap (~80% discount).
*   **Lifespan:** **Max 24 hours.** Google will kill the VM after 24 hours, even if they don't need the capacity.
*   **Notice:** 30-second termination warning.

### Spot VMs (The Modern Version)
*   **Price:** Highly dynamic (60% to 91% discount).
*   **Lifespan:** **No maximum runtime.** If Google doesn't need the capacity, it can run for days.
*   **Notice:** 30-second termination warning.

### Comparison: Preemptible vs. Spot
| Feature | Preemptible VMs | Spot VMs |
| :--- | :--- | :--- |
| **Max Runtime** | 24 Hours | **None** |
| **Discount** | Fixed (~80%) | Dynamic (60-91%) |
| **Termination** | 30-sec warning | 30-sec warning |
| **Use Case** | Fault-tolerant Batch Jobs | Fault-tolerant Batch Jobs |

---

## 4. Visual Representations

### The "Which One Should I Use?" Flowchart
```text
Is the workload predictable and 24/7?
      |
      |-- (YES) --> Committed Use Discount (CUD) [Max Savings]
      |
      |-- (NO)  --> Is the workload fault-tolerant (can be interrupted)?
                        |
                        |-- (YES) --> Spot VMs [Massive Savings]
                        |
                        |-- (NO)  --> Sustained Use Discount (SUD) [Automatic]
```

### Savings Scale
```text
[ $$$ ] Standard On-Demand (Short-term, unpredictable)
   |
[ $$  ] Sustained Use Discount (Automatic, mid-term)
   |
[ $   ] Committed Use Discount (1-3 Year Contract)
   |
[ ¢   ] Spot VMs (Highly interruptible, spare capacity)
```

---

## 5. Mental Models & Analogies

### The Airline Ticket Analogy
*   **On-Demand:** Buying a ticket at the gate 5 minutes before takeoff. Expensive but flexible.
*   **Sustained Use:** Frequent flyer miles. The more you fly this month, the more Google gives you back automatically.
*   **Committed Use:** Buying a non-refundable 1-year pass. You save 70%, but you pay even if you don't fly.
*   **Spot/Preemptible:** **"Standby" tickets.** You get a tiny price, but if a full-paying passenger shows up, the airline kicks you off the plane with 30 seconds' notice.

---

## 6. Service Comparison Table

| Feature | SUD | CUD | Spot / Preemptible |
| :--- | :--- | :--- | :--- |
| **Action Required** | None (Automatic) | Purchase Commitment | Set "Spot" to ON |
| **Best Use Case** | General usage | Steady-state Production | Batch jobs/Testing |
| **Savings** | 20-50% | Up to 70% | 60-91% |
| **Availability** | Guaranteed | Guaranteed | **Not Guaranteed** |
| **SLA** | Yes | Yes | No |

---

## 7. Common Mistakes & Exam Traps

### ❌ Mistake 1: Using Free Tier for Spot VMs
*   **Trap:** Trying to use your $300 free credits to test Spot VMs.
*   **Fact:** **Free tier credits are NOT applicable** for launching Preemptible or Spot VMs. You will be billed real money.

### ❌ Mistake 2: SUD on E2 instances
*   **Trap:** An exam question asks why your E2-medium VM didn't get a Sustained Use Discount.
*   **Fact:** E2, A2, and certain newer machine types are excluded from SUD.

### ❌ Mistake 3: Spot VMs for Web Frontends
*   **Trap:** Recommending Spot VMs for a customer-facing website to save money.
*   **Fact:** Bad idea. Spot VMs can be killed with 30 seconds' notice, which would crash the site for users.

---

## 🎯 10 Certification Practice Questions

**Q1. You are running a production database 24/7 for the next year. Which billing model offers the HIGHEST discount?**
A. Sustained Use Discount
B. Spot VMs
C. Committed Use Discount (3-year)
D. On-demand
*   **✅ Answer: C** (Highest guaranteed discount for predictable workloads).

**Q2. Which discount type is applied automatically by GCP without any user intervention?**
A. Committed Use Discount
B. Sustained Use Discount
C. Spot Discount
D. Flat-rate billing
*   **✅ Answer: B**

**Q3. A company has a batch processing job that takes 10 hours but isn't urgent. They want the lowest possible cost. What should they use?**
A. E2 Standard VM
B. Spot VM
C. Committed Use Discount
D. App Engine Flexible
*   **✅ Answer: B** (Lowest cost, and the job can handle interruptions).

**Q4. What is the key difference between a Spot VM and a legacy Preemptible VM?**
A. Spot VMs have a 24-hour limit.
B. Preemptible VMs have no maximum runtime.
C. Spot VMs have no maximum runtime.
D. Spot VMs do not give a termination notice.
*   **✅ Answer: C**

**Q5. How much notice does Google provide before reclaiming a Spot VM?**
A. 10 minutes
B. 5 minutes
C. 30 seconds
D. No notice is given
*   **✅ Answer: C**

**Q6. You are using an E2-small instance for 100% of the month. Why are you NOT seeing a Sustained Use Discount on your bill?**
A. E2 instances are not eligible for SUD.
B. You forgot to click "Enable SUD" in the console.
C. SUD only applies after 2 years.
D. You must use at least 2 vCPUs to qualify.
*   **✅ Answer: A**

**Q7. Which service does NOT support Sustained Use Discounts?**
A. Compute Engine
B. Google Kubernetes Engine
C. Dataflow
D. N1 Machine Types
*   **✅ Answer: C**

**Q8. You purchased a 1-year Hardware Commitment for the `us-central1` region. Can you apply this discount to a VM in `asia-south1`?**
A. Yes, commitments are global.
B. No, commitments are regional.
*   **✅ Answer: B** (Commitments are tied to a specific region).

**Q9. Can you migrate a Preemptible VM to a regular VM while it is running?**
A. Yes, via the "Edit" menu.
B. No.
*   **✅ Answer: B**

**Q10. Why should you avoid using Spot VMs for an immediate, user-facing application?**
A. They are too expensive.
B. They might not be available when needed and can be terminated abruptly.
C. They only run Linux.
D. They cannot be connected to a Load Balancer.
*   **✅ Answer: B**

---

## ⚡ 30-Second Revision
*   **SUD:** Automatic. >25% usage. No E2/A2.
*   **CUD:** 1 or 3-year contract. Predictable workloads. Best for Prod.
*   **Preemptible:** Cheap. Max 24 hours.
*   **Spot:** Cheaper. **No max runtime.**
*   **The "Batch" Rule:** If it can be interrupted, use Spot. If not, use CUD/SUD.
*   **The "Warning" Rule:** Always 30 seconds to clean up. ☁️