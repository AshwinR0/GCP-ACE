### 1. Overview
This topic introduces managing **Google Cloud Run** directly from the command line using the `gcloud` CLI. Cloud Run is a fully-managed serverless container platform that automatically scales containerized applications. This section covers configuring the default project and region, deploying a container image from a public registry (such as Docker Hub), managing multiple revisions of a service under the same service name, and configuring deployment options like instance counts, concurrency, CPU, and memory limits.

---

### 2. ACE Exam Focus & Common Exam Traps

- **High-Priority Concepts:**
  - Setting default `gcloud` configurations (`gcloud config set project` and `gcloud config set run/region`).
  - Deploying services using `gcloud run deploy`.
  - Understanding how **revisions** work: deploying a new image under the same service name automatically creates a new revision and shifts 100% of the traffic to it by default.
  - Managing scaling properties (`--min-instances`, `--max-instances`, `--concurrency`, `--scaling`).
- **Exam Importance:** HIGH
- **Key Command Flags:**
  - `--region`: Specifies the target region (can also be defaulted via config).
  - `--revision-suffix`: Appends a suffix to the auto-generated revision name (e.g., `-v1`, `-v2`).
  - `--max-instances` / `--min-instances`: Controls container scaling boundaries.
  - `--concurrency`: Sets the maximum number of simultaneous requests a single container instance can handle.
- **Common Exam Traps:**
  - Confusing service-level changes with revision creation: Running `gcloud run deploy` with an existing service name creates a *new revision* for that service rather than overwriting the existing running container directly.
  - Forgetting that unauthenticated access must be explicitly allowed (`--allow-unauthenticated`) if public endpoints are expected; otherwise, it defaults to requiring authentication (IAM).

---

### 3. Detailed Concept Explanation

#### Project & Region Configuration
Before interacting with Cloud Run via `gcloud`, the environment requires a target project ID and a default compute/run region.
- **Project Configuration:** `gcloud config set project <PROJECT_ID>` directs all subsequent commands to the specified project.
- **Region Configuration:** `gcloud config set run/region <REGION>` sets the default region for Cloud Run services (e.g., `us-central1`), avoiding the need to pass `--region` with every command.

#### Deploying Services & Revisions
- **`gcloud run deploy`**: The core command used to build source code or deploy an existing container image to Cloud Run.
- **Container Registry Source:** If no full container registry path (like Artifact Registry or Container Registry) is prefixed, `gcloud` defaults to pulling public container images from **Docker Hub**.
- **Revisions:** Cloud Run creates an immutable snapshot called a **revision** every time a service is deployed or updated. 
  - The `--revision-suffix` flag allows developers to assign a readable identifier (e.g., `v1`, `v2`) to the revision name (resulting in names like `hello-world-v1`).
  - By default, a newly deployed revision immediately receives **100% of the traffic**.

#### Advanced Deployment Flags
Cloud Run allows fine-tuning runtime behavior using flags during deployment:
- `--min-instances`: Keeps a specified number of container instances warm to eliminate cold starts (note: incurs cost when idle).
- `--max-instances`: Prevents runaway scaling and protects downstream resources by capping the maximum number of concurrent container instances.
- `--concurrency`: Defines how many requests a single container instance can process simultaneously (default varies by language/runtime).
- `--cpu` and `--memory`: Allocates specific hardware resource limits to each container instance (e.g., `--memory 512Mi`, `--cpu 1`).

---

### 4. Practical Setup & Implementation Guide

#### Prerequisites & Dependencies
- Access to a Google Cloud account and project.
- The Google Cloud SDK (`gcloud` CLI) installed, or using Cloud Shell.

#### Step-by-Step Implementation Instructions

1. **Set the Default Project:**
   ```bash
   gcloud config set project my-cloud-run-project
   ```
   *Expected Outcome:* Confirms the active project is set to `my-cloud-run-project`.

2. **Set the Default Cloud Run Region:**
   ```bash
   gcloud config set run/region us-central1
   ```
   *Expected Outcome:* Configures `us-central1` as the default location for future Cloud Run deployments.

3. **Deploy the First Revision (v1):**
   ```bash
   gcloud run deploy hello-world \
     --image=docker.io/username/hello-world:0.0.1.RELEASE \
     --revision-suffix=v1 \
     --allow-unauthenticated
   ```
   *Note:* If prompted to enable the required API (`run.googleapis.com`), type `y` to proceed. When asked about unauthenticated invocations, select `y` to make the service publicly accessible.
   *Expected Outcome:* Provisions the service `hello-world`, creates revision `hello-world-v1`, routes 100% of traffic to it, and outputs a service URL.

4. **Verify the Deployment:**
   Open the returned service URL in a browser or via `curl` with the appropriate path (e.g., appending `/hello-world`).
   *Expected Outcome:* Returns the response from revision `v1` (e.g., `Hello World V1 LOCAL`).

5. **Deploy the Second Revision (v2):**
   ```bash
   gcloud run deploy hello-world \
     --image=docker.io/username/hello-world:0.0.2.RELEASE \
     --revision-suffix=v2 \
     --allow-unauthenticated
   ```
   *Expected Outcome:* Recognizes the existing service name `hello-world`, provisions a new revision named `hello-world-v2`, automatically migrates 100% of traffic to `v2`, and outputs the updated response when hitting the endpoint.

6. **Cleanup (Optional):**
   To delete the entire project and all associated resources:
   ```bash
   gcloud projects delete my-cloud-run-project
   ```

---

### 5. Visual Representations

```
[ Developer / CLI ]
       │
       │ gcloud run deploy hello-world (Image v1, Suffix: v1)
       ▼
┌────────────────────────────────────────────────────────┐
│ Cloud Run Service: hello-world                         │
│                                                        │
│  ┌──────────────────────────────────────────────────┐  │
│  │ Revision: hello-world-v1 (100% Traffic)          │  │
│  │ Container Image: hello-world:0.0.1.RELEASE       │  │
│  └──────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────┘
       │
       │ Later: gcloud run deploy hello-world (Image v2, Suffix: v2)
       ▼
┌────────────────────────────────────────────────────────┐
│ Cloud Run Service: hello-world                         │
│                                                        │
│  ┌──────────────────────────────────────────────────┐  │
│  │ Revision: hello-world-v1 (0% Traffic)            │  │
│  └──────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────┐  │
│  │ Revision: hello-world-v2 (100% Traffic)          │  │
│  │ Container Image: hello-world:0.0.2.RELEASE       │  │
│  └──────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────┘
```

---

### 6. Mental Models & Analogies

- **Cloud Run Service vs. Revision:** Think of a **Cloud Run Service** as a book title, and **Revisions** as printed editions (`v1`, `v2`). When you release updates, you don't change the book title; you print a brand-new edition. Readers (traffic) automatically get directed to the newest edition by default, but older editions remain archived on the shelf and can be rolled back to if needed.

---

### 7. Step-by-Step Workflows / Comparison Tables

#### Workflow: Deploying a New Update to an Existing Cloud Run Service
1. **Prerequisite:** Ensure container image is built and uploaded to a registry (Docker Hub, Artifact Registry).
2. **Execute Deploy Command:** Run `gcloud run deploy <SERVICE_NAME> --image=<IMAGE_PATH> --revision-suffix=<SUFFIX>`.
3. **Automatic Actions by GCP:** 
   - Generates a new immutable revision configuration.
   - Spins up container instances for the new revision.
   - Automatically switches 100% of incoming router traffic from the old revision to the new revision.
4. **Verification:** Test the service endpoint to verify the updated behavior.

#### Comparison Table: Scaling Configurations (`--min-instances` vs `--max-instances` vs `--concurrency`)

| Flag / Setting | What it Controls | Default Behavior | Use Case |
| :--- | :--- | :--- | :--- |
| `--min-instances` | Minimum warm containers kept running. | `0` (scales to zero when idle) | Eliminating cold starts for latency-sensitive applications. |
| `--max-instances` | Upper boundary limit for concurrent scaling containers. | `100` (or region/project limits) | Protecting backend databases or controlling budget spikes. |
| `--concurrency` | Maximum simultaneous requests handled by *a single* container instance. | Automatically tuned based on CPU allocation | Adjusting how much load a single container handles before spinning up a duplicate. |

---

### 8. Common Mistakes & Exam Traps

- **Mistake:** Assuming running `gcloud run deploy` on an existing service overwrites the current running container code seamlessly in-place without creating a new resource version.
  - **Correction:** Every deployment creates a brand-new *revision*. This allows easy rollbacks if the new version fails.
- **Trap:** Believing that changing `--min-instances` to `1` eliminates all cost during periods of zero traffic.
  - **Correction:** Setting `--min-instances=1` means at least one container is *always* running, meaning you pay for idle compute time 24/7.

---

### 9. Certification Practice Questions

#### Question 1
You are deploying a new version of a containerized web application to an existing Cloud Run service named `inventory-api` using the `gcloud` CLI. What happens to the traffic distribution when the deployment completes successfully?

* A. Traffic is split evenly (50/50) between the previous revision and the new revision.
* B. 100% of the incoming traffic is automatically routed to the newly created revision.
* C. Traffic remains entirely on the old revision until you manually split the traffic using a tag.
* D. The deployment fails because an existing service name cannot accept new revisions without being deleted first.

**Correct Answer:** B  
**Explanation:** By default, when you deploy a new image or configuration to an existing Cloud Run service, Cloud Run automatically creates a new immutable revision and shifts 100% of incoming traffic to it.  
**Incorrect Answers:**  
A is incorrect because traffic is not split 50/50 by default.  
C is incorrect because traffic automatically migrates to the new revision without requiring manual traffic splitting steps.  
D is incorrect because Cloud Run services are specifically designed to accept multiple revisions over time.  
**Exam Takeaway:** New revisions receive 100% of traffic by default upon a successful deployment.

---

#### Question 2
Your development team wants to deploy a public-facing microservice to Cloud Run via the command line. When executing `gcloud run deploy`, the CLI prompts you with: *"Do you want to allow unauthenticated invocations?"* What does selecting "Yes" accomplish?

* A. It configures the service to allow public, unauthenticated HTTP requests without requiring an IAM identity token.
* B. It disables HTTPS and forces the service to accept insecure HTTP connections.
* C. It deletes the requirement for a Docker Hub container repository.
* D. It automatically generates a static internal IP address for the service within a VPC network.

**Correct Answer:** A  
**Explanation:** Allowing unauthenticated invocations binds the `allUsers` principal with the `roles/run.invoker` role on the Cloud Run service, making it publicly accessible to anyone on the internet without needing a Google Cloud IAM token.  
*Incorrect Answers:*  
B is incorrect because Cloud Run enforces secure HTTPS connections by default.  
C is incorrect; authentication settings have no relation to where the container image is stored.  
D is incorrect; Cloud Run services use public HTTPS URLs and integration with VPC is configured separately via VPC connectors.  
**Exam Takeaway:** `--allow-unauthenticated` controls public accessibility via the `allUsers` invoker role.

---

#### Question 3
You are using the `gcloud` CLI to manage multiple services and want to ensure that all subsequent `gcloud run` commands default to the `us-central1` region without needing to supply the `--region` flag every time. Which command should you execute?

* A. `gcloud config set compute/region us-central1`
* B. `gcloud run services update --region us-central1`
* C. `gcloud config set run/region us-central1`
* D. `gcloud defaults set region us-central1`

**Correct Answer:** C  
**Explanation:** The correct configuration property for setting the default region specifically for Cloud Run commands in `gcloud` is `run/region`.  
*Incorrect Answers:*  
A sets the default region for Compute Engine resources, not Cloud Run.  
B is an invalid syntax/command structure for setting global CLI defaults.  
D is not a valid `gcloud config` property path.  
**Exam Takeaway:** Know your `gcloud config` properties: `compute/region` for Compute Engine and `run/region` for Cloud Run.

---

#### Question 4
Your application experiences intermittent performance issues due to "cold starts" whenever traffic spikes after a long period of inactivity. Which configuration flag should you add to your `gcloud run deploy` command to mitigate this?

* A. `--max-instances=0`
* B. `--min-instances=1`
* C. `--concurrency=1`
* D. `--scaling=manual`

**Correct Answer:** B  
**Explanation:** Setting `--min-instances=1` (or higher) ensures that at least one container instance is kept warm and ready to serve requests, thereby eliminating cold starts.  
*Incorrect Answers:*  
A would restrict instances to zero, preventing the application from running.  
C configures maximum concurrent requests per instance, which does not prevent cold starts.  
D is not a valid option value format for basic instance floor controls in this context.  
**Exam Takeaway:** Use `--min-instances > 0` to keep instances warm and avoid cold starts at the expense of idle cost.

---

#### Question 5
When deploying an application to Cloud Run via `gcloud run deploy` without specifying a full container registry URL (such as Artifact Registry), where does Google Cloud look by default to pull the container image?

* A. Google Container Registry (GCR) storage buckets in your project
* B. Docker Hub
* C. GitHub Container Registry (GHCR)
* D. A local temporary directory on your local machine

**Correct Answer:** B  
**Explanation:** If no domain prefix or alternative container repository path is provided in the image parameter, Google Cloud defaults to pulling the public image from Docker Hub.  
*Incorrect Answers:*  
A, C, and D are incorrect because Docker Hub is the default fallback registry when a fully qualified path to Artifact Registry or Container Registry is omitted.  
**Exam Takeaway:** Omitting registry paths in `gcloud run deploy` defaults to Docker Hub.

---

#### Question 6
You deployed a web application revision named `hello-world-v1` to Cloud Run. You now want to deploy an updated package and append a clean tag identifier to the revision name. Which flag handles this requirement?

* A. `--tag`
* B. `--revision-suffix`
* C. `--version-name`
* D. `--service-suffix`

**Correct Answer:** B  
**Explanation:** The `--revision-suffix` flag appends custom text to the auto-generated base revision name (e.g., specifying `-v1` results in revision names like `service-name-v1`).  
*Incorrect Answers:*  
A is used for creating traffic-routable service URLs/tags, not naming the immutable base revision suffix directly.  
C and D are invalid flags for this operation.  
**Exam Takeaway:** Use `--revision-suffix` to explicitly name or version-tag revisions during deployment.

---

#### Question 7
An administrator wants to ensure that a Cloud Run service does not scale past 10 container instances simultaneously, regardless of how high incoming traffic scales. Which configuration flag enforces this restriction?

* A. `--max-instances=10`
* B. `--min-instances=10`
* C. `--concurrency=10`
* D. `--capacity-limit=10`

**Correct Answer:** A  
**Explanation:** The `--max-instances` flag sets an upper bound on the number of container instances that can run concurrently for a service or revision.  
*Incorrect Answers:*  
B sets a minimum instance floor.  
C limits concurrent requests *per container instance*, not total instances.  
D is a distractor and invalid flag.  
**Exam Takeaway:** `--max-instances` protects systems by capping container scale-out behavior.

---

#### Question 8
You have multiple revisions of a service running in Cloud Run. You want to adjust how many simultaneous requests a single container instance can process before Cloud Run spins up an extra container instance. Which flag modifies this setting?

* A. `--max-requests`
* B. `--concurrency`
* C. `--instance-threshold`
* D. `--parallelism`

**Correct Answer:** B  
*Explanation:* The `--concurrency` flag specifies the maximum number of requests that a single container instance can process concurrently.  
*Incorrect Answers:*  
A, C, and D are incorrect flags.  
**Exam Takeaway:** Concurrency controls per-container request capacity.

---

#### Question 9
Which of the following statements is true regarding Cloud Run revisions?

* A. Revisions are mutable; you can modify code inside a running revision directly.
* B. Revisions are immutable snapshots of container code, environment variables, and configurations.
* C. Deleting a Cloud Run service automatically preserves all associated historical revisions independently.
* D. You cannot run more than two revisions under the same Cloud Run service at any given time.

**Correct Answer:** B  
**Explanation:** Cloud Run revisions are immutable snapshots created upon deployment containing code configurations and settings. Once created, they cannot be changed.  
*Incorrect Answers:*  
A is false because revisions are immutable.  
C is false because deleting a service deletes all its associated revisions.  
D is false; multiple revisions can coexist under a single service to support rolling updates and traffic splitting.  
**Exam Takeaway:** Revisions are immutable snapshots.

---

#### Question 10
You are preparing to run a suite of deployment scripts in Cloud Shell. You want to verify which GCP project is currently set as the active target for your `gcloud` command-line session. Which command should you run?

* A. `gcloud project info`
* B. `gcloud config get-value project`
* C. `gcloud status`
* D. `gcloud services list-active`

**Correct Answer:** B  
**Explanation:** `gcloud config get-value project` queries and prints the currently active project configuration property.  
*Incorrect Answers:*  
A, C, and D do not query the active project configuration property in this manner.  
**Exam Takeaway:** Use `gcloud config get-value project` to verify the active CLI project.