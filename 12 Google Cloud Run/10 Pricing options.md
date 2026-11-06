# GCP Associate Cloud Engineer (ACE) Study Notes: Chapter - Google Cloud Run Pricing Options

## 1. Overview
This topic covers the **two core pricing models** available in Google Cloud Run: **Request-Based Billing** (the default) and **Instance-Based Billing**. It details how billing is calculated under each model, their ideal traffic use cases, and specific structural configurations (such as GPUs, minimum instances, or background threads) where instance-based billing becomes mandatory.

---

## 2. ACE Exam Focus & Common Exam Traps
*   **Exam Importance**: **HIGH**. Understanding when to choose request-based vs. instance-based billing, along with identifying mandatory triggers for instance-based billing, frequently appears in cost-optimization and architecture scenario questions.
*   **Key Concepts to Memorize**:
    *   *Request-Based Billing (Default)*: Billed per request **plus** a per-second rate for vCPU and memory **consumed strictly during request processing**. Ideal for spiky, event-driven, or highly variable traffic.
    *   *Instance-Based Billing*: Billed for the **entire lifetime/duration of a container instance** (no per-request fee). Ideal for stable, continuously available applications.
    *   *Mandatory Instance-Based Billing Scenarios*:
        1.  Attaching a **GPU** to your Cloud Run service.
        2.  Maintaining a minimum number of active instances all the time (`min-instances > 0`).
        3.  Running **asynchronous background threads** outside of direct request processing.
*   **Keywords for Scenario Questions**:
    *   *“Spiky / event-driven / variable traffic / minimize idle costs”* $\rightarrow$ **Request-based billing**.
    *   *“Stable / continuous traffic / instance-based billing”* $\rightarrow$ **Instance-based billing**.
    *   *“Cloud Run service requires a GPU or asynchronous background threads”* $\rightarrow$ **Instance-based billing is mandatory**.

---

## 3. Detailed Concept Explanation

### A. Request-Based Billing (Default)
*   **How it Works**: You pay a fee per request, combined with a fine-grained per-second rate for the exact vCPU and memory resources consumed **while processing requests**.
*   **Idle Behavior**: When no requests are being processed, resource allocation drops to zero (if scaling down to zero), resulting in $0 cost.
*   **Ideal Use Case**: Event-driven architectures, sporadic tasks, and spiky or highly variable web traffic.

### B. Instance-Based Billing
*   **How it Works**: You are charged for the total uptime/lifetime of a container instance from the moment it starts until it shuts down, regardless of whether it is actively processing an incoming request every second. There is no separate per-request fee.
*   **Ideal Use Case**: Stable, predictable traffic patterns where instances remain warm and active continuously.

### C. Mandatory Instance-Based Billing Scenarios
Cloud Run automatically forces instance-based billing under certain structural configurations:
1.  **Attaching a GPU**: Hardware acceleration requires continuous resource reservation.
2.  **Resident Instances (`min-instances > 0`)**: If you configure warm minimum instances to run continuously to eliminate cold starts, those instances must be billed under instance-based pricing because they live outside direct request-triggered processing windows.
3.  **Asynchronous Background Threads**: If your application spawns background processes or threads that run *after* or *independent* of an HTTP request response cycle, the container must stay alive, triggering instance-based billing.

---

## 4. Practical Setup & Implementation Guide
*Note: Pricing model selections are configured during service deployment or updates via the Google Cloud Console billing/pricing panels or CLI flags.*

---

## 5. Visual Representations

### Request-Based vs. Instance-Based Billing Comparison
```
1. REQUEST-BASED BILLING (Default):
[Request Arrives] ===> [Process Request: Billed for vCPU/Memory + Request Fee] ===> [Idle: $0 Cost]

2. INSTANCE-BASED BILLING:
[Instance Starts Up] ==========================================================> [Instance Shuts Down]
|<----------------------- Billed for Entire Lifetime / Uptime ------------------>|
(Mandatory when: min-instances > 0, GPU attached, or background threads active)
```

---

## 6. Mental Models & Analogies

*   **Request-Based Billing is like a pay-per-mile taxi ride**: You only pay for the exact distance traveled while you are inside the cab moving toward your destination. When you get out and walk away, the meter stops and you pay nothing more.
*   **Instance-Based Billing is like renting a car by the flat hourly rate**: You pay for every single hour the keys are in your hand, whether you are actively driving down the highway or sitting parked in a driveway with the engine idling.

---

## 7. Step-by-Step Workflows / Comparison Tables

### Cloud Run Pricing Models Comparison Table

| Pricing Model | Billing Basis | Ideal Traffic Pattern | Mandatory Triggers / Exceptions |
| :--- | :--- | :--- | :--- |
| **Request-Based (Default)** | Per-request + per-second vCPU/RAM **during request execution** | Spiky, event-driven, highly variable traffic | None (Default choice for standard serverless apps). |
| **Instance-Based** | Billed for the **entire lifetime/uptime** of the container instance | Stable, continuous traffic | **GPUs attached**, **`min-instances > 0`**, or **asynchronous background threads**. |

---

## 8. Common Mistakes & Exam Traps

*   *Trap 1*: Assuming request-based billing charges you for CPU and memory usage even when the container is sitting idle between requests.
    *   *Correction*: Under request-based billing, vCPU and memory charges apply strictly *during request processing*. When idle, if instances scale to zero, costs drop to zero.
*   *Trap 2*: Believing you can choose request-based billing when maintaining resident instances (`min-instances > 0`).
    *   *Correction*: Maintaining minimum active instances forces Cloud Run to switch to **instance-based billing** because those instances run continuously outside of direct request triggers.
*   *Trap 3*: Overlooking background threads as a pricing driver.
    *   *Correction*: If an application runs asynchronous background processing threads, instance-based billing becomes mandatory because the container cannot shut down immediately after returning an HTTP response.

---

## 9. Certification Practice Questions

#### Question 1
What is the default pricing model applied to a newly created Google Cloud Run service?
*   A) Instance-based billing
*   B) Request-based billing
*   C) Flat monthly reservation tier
*   D) Spot container preemption pricing
*   **Correct Answer**: B
*   **Explanation**: Request-based billing is the default pricing model for Cloud Run.
*   **Exam Takeaway**: Cloud Run defaults to **request-based billing**.

#### Question 2
An application on Cloud Run experiences highly spiky, unpredictable web traffic with long idle windows. Which pricing option is recommended to minimize costs during low-traffic periods?
*   A) Instance-based billing
*   B) Request-based billing
*   C) Dedicated core-hour reservation billing
*   D) Permanent manual scaling billing
*   **Correct Answer**: B
*   **Explanation**: Request-based billing charges only when requests are actively processed, making it ideal for spiky or variable traffic.
*   **Exam Takeaway**: Spiky, variable traffic maps to **request-based billing**.

#### Question 3
Under which of the following structural configurations is **instance-based billing mandatory** in Cloud Run? (Choose the best combination)
*   A) Attaching a GPU, setting minimum instances greater than zero (`min-instances > 0`), or running asynchronous background threads.
*   B) Using the default concurrency setting of 80.
*   C) Deploying a Python application using standard Buildpacks.
*   D) Allowing public unauthenticated access to the HTTPS URL.
*   **Correct Answer**: A
*   **Explanation**: Instance-based billing is mandatory when GPUs are attached, when resident instances are maintained (`min-instances > 0`), or when asynchronous background threads run.
*   **Exam Takeaway**: Memorize the mandatory instance-based triggers: GPUs, `min-instances > 0`, and background threads.

#### Question 4
How are vCPU and memory resources billed under the default **request-based billing** model in Cloud Run?
*   A) Charged at a flat hourly rate for every virtual machine node in the region.
*   B) Charged at a per-second rate strictly for the resources consumed during request processing.
*   C) Charged based on the total physical storage size of the image in Artifact Registry.
*   D) Completely free of charge regardless of traffic volume.
*   **Correct Answer**: B
*   **Explanation**: Under request-based billing, vCPU and memory are billed at a per-second rate during active request execution.
*   **Exam Takeaway**: Request-based billing bills vCPU/memory per second during request processing.

#### Question 5
What is the primary characteristic of **instance-based billing** regarding container instance lifetimes?
*   A) You are billed for the entire lifetime/uptime of the instance, with no per-request fee.
*   B) You are billed only when a client clicks a web link.
*   C) You are billed exclusively for inbound data transfer gigabytes.
*   D) Instances are billed only when they scale down to zero.
*   **Correct Answer**: A
*   **Explanation**: Instance-based billing charges for the continuous uptime and lifetime of the container instance rather than per request.
*   **Exam Takeaway**: Instance-based billing charges for total container uptime.

#### Question 6
An enterprise application requires continuous 24/7 availability with stable traffic patterns and maintains resident warm instances (`min-instances = 5`) to ensure zero cold starts. Which pricing model will Cloud Run apply?
*   A) Request-based billing automatically.
*   B) Instance-based billing, because maintaining minimum instances forces instance-based rules.
*   C) Free tier zero-cost billing.
*   D) Spot preemption batch billing.
*   **Correct Answer**: B
*   **Explanation**: Configuring minimum instances greater than zero requires continuous uptime, making instance-based billing mandatory.
*   **Exam Takeaway**: Setting `min-instances > 0` forces instance-based billing.

#### Question 7
If a Cloud Run service executes asynchronous background threads that continue running after an HTTP response has been sent back to the client, why is instance-based billing required?
*   A) Because background threads cannot run on serverless platforms.
*   B) Because the container instance must remain alive outside of direct request processing windows to finish the background work.
*   C) Because Artifact Registry requires continuous storage sync.
*   D) Because concurrency must be locked permanently to 1.
*   **Correct Answer**: B
*   **Explanation**: Background threads operate outside of direct request execution windows, requiring continuous container uptime that forces instance-based billing.
*   **Exam Takeaway**: Asynchronous background threads require continuous uptime, triggering instance-based billing.

#### Question 8
When is instance-based billing *not* required?
*   A) When attaching hardware GPUs to accelerate machine learning models.
*   B) When running a standard web API with request-based auto-scaling and `min-instances = 0`.
*   C) When running background threads that execute after HTTP requests conclude.
*   D) When maintaining resident warm instances (`min-instances = 2`).
*   **Correct Answer**: B
*   **Explanation**: A standard web API scaling down to zero (`min-instances = 0`) operates entirely under default request-based billing.
*   **Exam Takeaway**: Standard request-driven apps with zero minimum instances use default request-based billing.

#### Question 9
What financial impact does scaling down to zero (`min-instances = 0`) have under request-based billing when a service receives zero requests overnight?
*   A) Compute costs drop to zero because no instances are running and no requests are processed.
*   B) You continue paying for container instance uptime.
*   C) You are billed a flat standby reservation fee.
*   D) The service incurs daily storage snapshot penalties.
*   **Correct Answer**: A
*   **Explanation**: Scaling down to zero under request-based billing shuts down all active instances, resulting in zero compute charges during idle periods.
*   **Exam Takeaway**: Request-based billing with zero instances results in zero idle compute cost.

#### Question 10
If an administrator attaches a hardware **GPU** to a Cloud Run service to handle intense graphical rendering, which pricing model is automatically enforced?
*   A) Request-based billing
*   B) Instance-based billing
*   C) Free tier promotional billing
*   D) Spot instance preemptible billing
*   **Correct Answer**: B
*   **Explanation**: Attaching specialized hardware like GPUs requires continuous reservation of physical compute resources, making instance-based billing mandatory.
*   **Exam Takeaway**: Attaching GPUs forces instance-based billing.