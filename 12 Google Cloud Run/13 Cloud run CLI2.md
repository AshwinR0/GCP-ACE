# Cloud Run CLI Operations & Service Management — Consolidation Notes

### 1. Overview
This module consolidates foundational and advanced `gcloud` operations for **Google Cloud Run**, transitioning from initial environment configurations to managing deployment lifecycles, revisions, traffic splitting, and scaling properties. Cloud Run abstracts away infrastructure management, allowing engineers to focus purely on containerized application delivery via automation-friendly CLI commands.

---

### 2. ACE Exam Focus & Common Exam Traps

- **High-Priority Concepts:**
  - Configuring default project and region properties via `gcloud config`.
  - Understanding the relationship between a **Cloud Run Service** and its immutable **Revisions**.
  - Traffic routing mechanics (migrating 100% traffic or splitting percentages between revisions).
- **Exam Importance:** HIGH
- **Key Commands & Properties:**
  - `gcloud config set run/region <REGION>`
  - `gcloud run deploy <SERVICE_NAME> --image=<IMAGE_URL> --revision-suffix=<SUFFIX>`
  - `--allow-unauthenticated`
- **Common Exam Traps:**
  - Assuming old revisions are deleted when a new revision is deployed. (Exam reality: Old revisions remain intact for rolling updates and rollbacks unless explicitly deleted).

---

### 3. Detailed Concept Explanation

#### Revision Lifecycle & Traffic Splitting
When a service is updated via `gcloud run deploy`, Cloud Run builds a new **revision**. 
- **Immutability:** Revisions cannot be modified after creation. Any configuration change (environment variables, CPU, memory, or container image) generates a new revision.
- **Traffic Migration:** By default, a new revision immediately receives 100% of the service's traffic. Using `gcloud run services update-traffic`, administrators can split traffic dynamically across multiple active revisions (e.g., for canary deployments or blue/green testing).

#### Scaling & Resource Boundaries
- **Concurrency (`--concurrency`):** Controls how many concurrent requests a single container instance handles.
- **Instance Limits (`--min-instances` / `--max-instances`):** Manages cost versus performance, keeping instances warm (`min > 0`) or preventing runaway scaling (`max = N`).

---

### 4. Practical Setup & Implementation Guide

*(Consolidated from previous command line operations)*

1. **Configure Environment Defaults:**
   ```bash
   gcloud config set project my-cloud-run-project
   gcloud config set run/region us-central1
   ```
2. **Deploy Service Revision v1:**
   ```bash
   gcloud run deploy hello-world \
     --image=docker.io/library/hello-world:latest \
     --revision-suffix=v1 \
     --allow-unauthenticated
   ```
3. **Deploy Subsequent Revision v2:**
   ```bash
   gcloud run deploy hello-world \
     --image=docker.io/library/hello-world:v2 \
     --revision-suffix=v2 \
     --allow-unauthenticated
   ```

---

### 5. Visual Representations

```
Incoming HTTPS Request
         │
         ▼
┌────────────────────────────────────────┐
│ Cloud Run Global HTTPS Load Balancer   │
└────────────────────────────────────────┘
         │
         ▼
┌────────────────────────────────────────┐
│ Cloud Run Service: "hello-world"       │
│                                        │
│  [ Revision: hello-world-v1 ] (0%)     │
│  [ Revision: hello-world-v2 ] (100%)   │
└────────────────────────────────────────┘
         │
         ▼
  Container Instance (v2)
```

---

### 6. Mental Models & Analogies

- **Revisions as Software Build Artifacts:** Think of a Cloud Run Service as an app package in an app store, and **Revisions** as specific app versions (`v1.0`, `v2.0`). Users automatically download the newest version, but older versions remain available on the server if you need to downgrade due to a bug.

---

### 7. Step-by-Step Workflows / Comparison Tables

#### Comparison Table: Core Cloud Run Management Commands

| Action | `gcloud` Command Syntax | Purpose |
| :--- | :--- | :--- |
| **Deploy / Update Service** | `gcloud run deploy SERVICE --image=IMAGE` | Creates a service or updates it by deploying a new revision. |
| **Describe Service** | `gcloud run services describe SERVICE` | Views current configuration, URLs, and status metrics. |
| **List Revisions** | `gcloud run revisions list` | Lists all immutable revisions created under the project/region. |
| **Delete Service** | `gcloud run services delete SERVICE` | Deletes the service along with all associated revisions and traffic mappings. |

---

### 8. Common Mistakes & Exam Traps

- **Trap:** Expecting environment variable modifications to apply to an existing revision.
- **Correction:** Environment variables, resource limits, and container images are bound to a *revision*. Changing them always spawns a *new* revision.

---

### 9. Certification Practice Questions

#### Question 1
An administrator updates a Cloud Run service configuration (such as memory limits) using the `gcloud run deploy` command. What happens to the existing active revision?
* A. It is overwritten in place with the new memory configuration.
* B. It remains active or archived as an immutable revision, while a new revision is created with the updated configuration.
* C. It is automatically deleted to free up project quotas.
* D. The deployment fails because configurations cannot be changed after initial creation.

**Correct Answer:** B  
**Explanation:** Revisions are immutable. Updating any configuration element creates a brand-new revision.  
**Exam Takeaway:** Revisions are immutable snapshots; updates always generate new revisions.

---

#### Question 2
You want to inspect the details, URL, and current traffic allocation of an existing Cloud Run service named `order-processor` from the command line. Which command should you use?
* A. `gcloud run services describe order-processor`
* B. `gcloud run revisions show order-processor`
* C. `gcloud compute instances describe order-processor`
* D. `gcloud run details order-processor`

**Correct Answer:** A  
**Explanation:** `gcloud run services describe` is the correct command to view service metadata, status, URLs, and traffic targets.  
**Exam Takeaway:** Use `gcloud run services describe` for service-level metadata inspection.

---

#### Question 3
If you deploy a container image to Cloud Run and completely omit the `--region` flag, where does `gcloud` determine which region to deploy the service to?
* A. It defaults to `us-central1` globally across all GCP accounts.
* B. It checks the local `gcloud config` property `run/region`.
* C. It fails and requires the explicit flag every time.
* D. It deploys the service to all available GCP regions simultaneously.

**Correct Answer:** B  
**Explanation:** `gcloud` checks local user configurations (`gcloud config get-value run/region`) if the explicit CLI flag is missing.  
*Exam Takeaway:** Default CLI settings prevent repetitive flag entries.

---

#### Question 4
Which of the following components forms the basic building block of a Cloud Run deployment history?
* A. Pods
* B. Revisions
* C. Snapshots
* D. Replica Sets

**Correct Answer:** B  
**Explanation:** Cloud Run manages deployments using immutable units called revisions.  
*Exam Takeaway:** Revisions track version history in Cloud Run.

---

#### Question 5
Can a single Cloud Run service have multiple revisions active and serving traffic simultaneously?
* A. No, only one revision can exist per service at any time.
* B. Yes, traffic can be split across multiple revisions using traffic-splitting rules.
* C. Yes, but only if they use different container registries.
* D. No, old revisions are automatically destroyed the moment a new one is deployed.

**Correct Answer:** B  
*Explanation:* Cloud Run supports multi-revision traffic splitting, useful for canary releases.  
*Exam Takeaway:* Multiple revisions can coexist under a single service for traffic splitting.

---

#### Question 6
When cleaning up a test environment containing a Cloud Run service, what happens to underlying revision objects when you delete the parent service?
* A. They remain orphaned in the project and must be deleted individually.
* B. They are deleted automatically along with the service.
* C. They are moved to a cold-storage archive bucket.
* D. They are converted into standalone Compute Engine instances.

**Correct Answer:** B  
*Explanation:* Deleting a Cloud Run service removes all child revisions and associated traffic mappings.  
*Exam Takeaway:* Service deletion cascades to child revisions.

---

#### Question 7
Which of the following CLI approaches is correct for setting a minimum instance count of `2` on a service named `payment-service` during deployment?
* A. `gcloud run deploy payment-service --min-instances=2`
* B. `gcloud run instances set payment-service --count=2`
* C. `gcloud run services update payment-service --instances=min:2`
* D. `gcloud config set run/min-instances 2`

**Correct Answer:** A  
*Explanation:* `--min-instances` is passed directly to `gcloud run deploy` (or `gcloud run services update`).  
*Exam Takeaway:** Use `--min-instances` to set instance minimum floors.

---

#### Question 8
What type of infrastructure abstraction does Cloud Run provide to developers?
* A. Infrastructure-as-a-Service (IaaS) virtual machines
* B. Fully-managed Serverless container platform
* C. Kubernetes cluster node pools
* D. Managed relational database instances

**Correct Answer:** B  
*Explanation:* Cloud Run is a fully-managed serverless platform for containerized applications.  
*Exam Takeaway:* Cloud Run = fully managed serverless containers.

---

#### Question 9
If a developer wants to run a container image stored in Artifact Registry instead of Docker Hub using `gcloud run deploy`, how is the image specified?
* A. By prefixing it with the full Artifact Registry repository path (e.g., `LOCATION-docker.pkg.dev/PROJECT/REPO/IMAGE`)
* B. By adding `--registry=artifact-registry`
* C. Artifact Registry cannot be used with Cloud Run.
* D. By modifying local Docker daemon configurations.

**Correct Answer:** A  
*Explanation:* Full registry paths (such as Artifact Registry host patterns) override the Docker Hub default behavior.  
*Exam Takeaway:* Specify full artifact paths to pull from Artifact Registry.

---

#### Question 10
Why might an engineer configure a custom `--revision-suffix` during a Cloud Run deployment?
* A. To encrypt the container image contents
* B. To make the immutable revision name easily identifiable (e.g., matching a Git commit or release tag version)
* C. To bypass IAM authentication checks
* D. To increase maximum CPU limits per instance

**Correct Answer:** B  
*Explanation:* Revision suffixes add readability to auto-generated revision hash strings.  
*Exam Takeaway:** Revision suffixes customize revision naming visibility.