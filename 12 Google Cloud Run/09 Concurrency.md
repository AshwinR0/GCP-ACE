# GCP Associate Cloud Engineer (ACE) Study Notes: Chapter - Google Cloud Run Concurrency

## 1. Overview
This topic covers **concurrency** in Google Cloud Run. Concurrency defines the maximum number of simultaneous, concurrent requests that a **single container instance** can handle at any given time. Adjusting concurrency directly impacts how many container instances Cloud Run provisions, influencing overall performance, scaling speed, and cost efficiency.

---

## 2. ACE Exam Focus & Common Exam Traps
*   **Exam Importance**: **HIGH**. The default concurrency number and how concurrency affects instance provisioning and cost are frequent exam topics.
*   **Key Values & Flags to Memorize**:
    *   *Default Concurrency*: **80** requests per instance.
    *   *Single-threading constraint*: If an application is not thread-safe and cannot handle parallel requests, concurrency must be explicitly set to **1**.
    *   *CLI Flag*: `--concurrency` (e.g., `--concurrency=60` or `--concurrency=1`).
*   **Core Logic**:
    *   *Higher Concurrency* $\rightarrow$ Fewer instances needed, faster scaling, and lower costs for high-traffic apps.
    *   *Concurrency = 1* $\rightarrow$ Every single incoming request gets its own dedicated instance (useful for legacy or non-thread-safe code, but leads to high instance counts and slower scaling under heavy loads).
*   **Keywords for Scenario Questions**:
    *   *“Default concurrent requests per instance in Cloud Run”* $\rightarrow$ **80**.
    *   *“App cannot handle parallel requests / legacy single-threaded app”* $\rightarrow$ Set concurrency to **1**.
    *   *“Reduce the number of provisioned instances and optimize scaling cost under heavy incoming request load”* $\rightarrow$ Increase **concurrency**.

---

## 3. Detailed Concept Explanation

### A. What is Concurrency?
*   Concurrency refers to how many requests a single container instance processes *simultaneously*.
*   *Example*: If 800 requests arrive at the same time:
    *   With **concurrency = 1**, Cloud Run must spin up **800 distinct container instances** (one per request).
    *   With **concurrency = 80** (default), Cloud Run only needs about **10 container instances** (800 / 80).

### B. Default Concurrency and Configuration
*   **Default Setting**: Cloud Run sets the default maximum concurrent requests per instance to **80**.
*   **Customization**: You can adjust this value during initial deployment or update it on an existing service using the `--concurrency` flag or via the Google Cloud Console.
*   **When to use Concurrency = 1**: If your application architecture uses global mutable state, legacy libraries, or language runtimes that cannot process thread-parallel requests safely, you must configure concurrency to `1`.

---

## 4. Practical Setup & Implementation Guide

### Configuring Concurrency via `gcloud`

#### 1. Setting Concurrency During Initial Deployment
```bash
gcloud run deploy my-service \
    --image=us-central1-docker.pkg.dev/my-project/my-repo/my-app:v1 \
    --concurrency=60 \
    --region=us-central1 \
    --allow-unauthenticated
```

#### 2. Updating Concurrency on an Existing Service
```bash
gcloud run services update my-service \
    --concurrency=40 \
    --region=us-central1
```

#### 3. Forcing Single-Threaded Processing (Concurrency = 1)
```bash
gcloud run services update my-service \
    --concurrency=1 \
    --region=us-central1
```

---

## 5. Visual Representations

### Concurrency Impact on Instance Provisioning (800 Incoming Requests)
```
SCENARIO A: Concurrency = 1
[800 Requests] ---> [Instance 1] [Instance 2] ... [Instance 800] (800 Total Instances needed)

SCENARIO B: Concurrency = 80 (Default)
[800 Requests] ---> [Instance 1 (Handles 80)] ... [Instance 10 (Handles 80)] (Only 10 Total Instances needed)
```

---

## 6. Mental Models & Analogies

*   **Concurrency is like a checkout lane clerk at a supermarket**: 
    *   With **concurrency = 1**, the supermarket opens a separate cash register and hires a brand-new clerk for *every single customer* who walks through the door (requiring 800 clerks for 800 customers).
    *   With **concurrency = 80** (default), a single efficient clerk handles a fast-moving queue of up to 80 customers concurrently using digital scan-and-go lines, requiring only a fraction of the total staff.

---

## 7. Step-by-Step Workflows / Comparison Tables

### Concurrency Setting Comparison Table

| Concurrency Setting | Provisioned Instances for 800 Requests | Suitability / Use Case |
| :--- | :--- | :--- |
| **Concurrency = 1** | 800 instances | Non-thread-safe applications, legacy codebases that break under parallel execution. |
| **Concurrency = 80 (Default)** | 10 instances | Standard web apps, microservices, and APIs built to handle multi-threaded parallel requests. |
| **Custom Concurrency (e.g., 40 / 100)** | Scaled dynamically (Requests / Concurrency) | Tuned based on application memory boundaries and CPU thread limits. |

---

## 8. Common Mistakes & Exam Traps

*   *Trap 1*: Assuming higher concurrency always improves application performance.
    *   *Correction*: If an application is CPU-bound or memory-bound, stuffing too many concurrent requests into a single container instance can exhaust resource limits and cause crashes. Concurrency should match application capabilities.
*   *Trap 2*: Forgetting the default Cloud Run concurrency value.
    *   *Correction*: The default concurrency for Cloud Run is **80** requests per instance.
*   *Trap 3*: Thinking setting concurrency to 1 means the service handles only 1 request ever.
    *   *Correction*: Concurrency = 1 means a *single instance* handles only 1 request *at a time*. Additional requests will spawn additional instances (up to max-instances) rather than being processed concurrently on the same instance.

---

## 9. Certification Practice Questions

#### Question 1
What is the default maximum number of concurrent requests that a single container instance can handle in a Google Cloud Run service?
*   A) 10
*   B) 50
*   C) 80
*   D) 1000
*   **Correct Answer**: C
*   **Explanation**: The default concurrency setting for Cloud Run services is 80 requests per instance.
*   **Exam Takeaway**: Memorize the default Cloud Run concurrency value of **80**.

#### Question 2
An application deployed on Cloud Run is a legacy monolithic service that uses global mutable variables and cannot process parallel requests safely. What concurrency setting must be configured for this service?
*   A) Concurrency = 80
*   B) Concurrency = 1
*   C) Concurrency = 500
*   D) Concurrency = auto
*   **Correct Answer**: B
*   **Explanation**: If an application cannot process parallel requests safely, concurrency must be set to 1 so that each instance handles only one request at a time.
*   **Exam Takeaway**: Non-thread-safe apps requiring single execution per instance map to concurrency = 1.

#### Question 3
If 400 requests arrive simultaneously at a Cloud Run service configured with the default concurrency setting of 80, approximately how many container instances will Cloud Run initially need to provision?
*   A) 1 instance
*   B) 5 instances
*   C) 80 instances
*   D) 400 instances
*   **Correct Answer**: B
*   **Explanation**: 400 requests divided by a concurrency limit of 80 requests per instance results in 5 container instances (`400 / 80 = 5`).
*   **Exam Takeaway**: Instance count = Total Requests / Concurrency Limit.

#### Question 4
Which `gcloud` flag is used to configure the maximum concurrent requests per instance when deploying or updating a Cloud Run service?
*   A) `--max-requests`
*   B) `--concurrency`
*   C) `--parallelism-limit`
*   D) `--threads`
*   **Correct Answer**: B
*   **Explanation**: The CLI flag to control instance concurrency in Cloud Run is `--concurrency`.
*   **Exam Takeaway**: Use `--concurrency` to specify maximum concurrent requests per instance.

#### Question 5
What is the primary operational advantage of configuring a *higher* concurrency limit on an application that supports parallel request processing?
*   A) It increases the security perimeter against SQL injection attacks.
*   B) It reduces the total number of container instances required under heavy load, leading to lower costs and faster scaling.
*   C) It forces the application to run exclusively on App Engine Standard.
*   D) It eliminates all cold starts permanently without requiring resident instances.
*   **Correct Answer**: B
*   **Explanation**: Higher concurrency means fewer instances are needed to process a given volume of requests, which optimizes scaling efficiency and lowers costs.
*   **Exam Takeaway**: Higher concurrency reduces instance counts, improves scaling, and lowers costs.

#### Question 6
An administrator notices that their Cloud Run service is spinning up hundreds of container instances even during moderate traffic volumes. Upon checking, they find that concurrency was manually set to 1. What is the impact of keeping concurrency at 1?
*   A) Every single request is routed to its own dedicated container instance, causing high instance proliferation.
*   B) Requests are blocked entirely with a 504 Gateway Timeout error.
*   C) The service scales down to zero instantly.
*   D) Billing charges are completely waived by Google Cloud.
*   **Correct Answer**: A
*   **Explanation**: Setting concurrency to 1 means each instance can only process one request at a time, forcing Cloud Run to provision a new instance for every concurrent request.
*   **Exam Takeaway**: Concurrency = 1 leads to high instance counts because each request requires its own instance.

#### Question 7
Can an administrator update the concurrency value on an existing, running Cloud Run service without having to recreate the service from scratch?
*   A) No, concurrency is immutable once a Cloud Run service is initialized.
*   B) Yes, using the `gcloud run services update` command with the `--concurrency` flag.
*   C) Yes, but only by modifying the underlying source code's `Dockerfile`.
*   D) No, updates require migrating the service to Cloud Run jobs.
*   **Correct Answer**: B
*   **Explanation**: Concurrency values can be modified dynamically on existing services using `gcloud run services update` or via the Google Cloud Console.
*   **Exam Takeaway**: Concurrency settings can be updated on existing services dynamically.

#### Question 8
If an application is memory-intensive and prone to crashing if more than 10 requests hit a single container at the same time, what concurrency value should the administrator configure?
*   A) Concurrency = 80 (default)
*   B) Concurrency = 10
*   C) Concurrency = 1000
*   D) Concurrency = 0
*   **Correct Answer**: B
*   **Explanation**: Concurrency should be tuned to match application resource limits. Setting it to 10 prevents any single instance from receiving more than 10 simultaneous requests.
*   **Exam Takeaway**: Tune concurrency limits to match application memory and CPU boundaries.

#### Question 9
Which of the following statements accurately describes how Cloud Run handles requests that exceed the concurrency limit of an active instance?
*   A) Excess requests are dropped immediately with a 403 Forbidden error.
*   B) Cloud Run automatically provisions new container instances (up to the max-instances ceiling) to absorb the excess request load.
*   C) Requests are queued indefinitely inside a Cloud SQL database.
*   D) The service restarts automatically.
*   **Correct Answer**: B
*   **Explanation**: When incoming request volume exceeds the concurrency capacity of active instances, Cloud Run automatically scales out by provisioning new container instances (subject to `max-instances`).
*   **Exam Takeaway**: Cloud Run scales out by spinning up new instances when concurrency limits on existing instances are reached.

#### Question 10
Does a Cloud Run function share the same default concurrency limit concept as a Cloud Run service?
*   A) No, functions run strictly single-threaded with concurrency locked permanently at 1.
*   B) Yes, Cloud Run functions and services both manage execution concurrency, though default limits and configurations align with serverless execution models.
*   C) Functions do not process requests; they process Pub/Sub messages only.
*   D) Functions require manual Kubernetes pod tuning.
*   **Correct Answer**: B
*   **Explanation**: Cloud Run functions also manage concurrency (often defaulting to process single or multiple concurrent requests depending on function settings and runtimes), sharing the underlying Cloud Run container execution model.
*   **Exam Takeaway**: Cloud Run functions share the underlying container concurrency model.