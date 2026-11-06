# GCP Associate Cloud Engineer (ACE) Study Notes: Chapter - Google Cloud Run Services Implementation & Management

## 1. Overview
This topic covers the practical setup, deployment, and management of **Google Cloud Run services** through the Google Cloud Console. It details configuring container images, setting authentication rules, choosing billing models (request-based vs. instance-based), adjusting auto-scaling boundaries (including scaling down to zero), allocating vCPUs and memory resources, examining built-in observability metrics, and exploring networking, custom domains, and security controls such as **Binary Authorization**.

---

## 2. ACE Exam Focus & Common Exam Traps
*   **Exam Importance**: **HIGH**. Configuration options for Cloud Run services, billing options, scaling parameters, and security features like Binary Authorization frequently appear in scenario-based ACE exam questions.
*   **Key Configurations & Behaviors to Memorize**:
    *   *API Dependency*: Creating any Cloud Run resource automatically triggers the enablement of the **Cloud Run Admin API**.
    *   *Billing Options*: 
        *   **Request-based**: Charged only when instances are actively processing requests.
        *   *Instance-based*: Charged for the entire lifecycle of the container instances.
    *   *Scaling Configuration*: You can configure manual scaling (fixed instance count) or auto-scaling (specifying minimum and maximum instances, including setting `min instances = 0` to scale down to zero when idle).
    *   *Binary Authorization*: A security control used to ensure that only trusted, verified container images are deployed to Cloud Run.
    *   *Triggers*: Cloud Run services/functions can be driven by HTTPS URLs or event sources like **Pub/Sub**, **Cloud Storage**, **Firestore**, or **Eventarc**.
*   **Keywords for Scenario Questions**:
    *   *“Ensure only trusted container images with no security vulnerabilities are deployed to Cloud Run”* $\rightarrow$ **Binary Authorization**.
    *   *“Pay only when requests are actively being processed”* $\rightarrow$ **Request-based billing**.
    *   *“Map a personalized URL (e.g., example.com) to a Cloud Run endpoint”* $\rightarrow$ **Custom domain mapping**.

---

## 3. Detailed Concept Explanation

### A. Core Cloud Run Configurations
*   **Services vs. Jobs vs. Worker Pools**:
    *   *Services*: Expose a unique HTTPS endpoint for web applications, REST APIs, WebSockets, or gRPC.
    *   *Jobs*: Execute batch processing or run-to-completion tasks.
    *   *Worker Pools*: Provide compute for non-HTTP pull-based queue workloads.
*   **Container Source Options**: You can either deploy an existing pre-built container image from **Artifact Registry** or **Docker Hub**, or build from source code repositories (GitHub, GitLab, Bitbucket).
*   **Authentication**: Options include allowing unauthenticated **public access** or locking down the service using **IAM** or **Identity-Aware Proxy (IAP)**.

### B. Resource Sourcing and Performance Controls
*   **Resource Sizing**: Fine-tune allocation per instance, including memory (e.g., 512 MB or higher), vCPUs, non-persistent local storage, GPUs, request timeouts, and **max concurrent requests** per instance.
*   **Observability Dashboard**: Out-of-the-box monitoring tracking request counts, end-to-end request latencies, active container instance counts, and CPU/memory utilization, alongside direct access to system logs and runtime error reporting.

### C. Security and Networking
*   **Ingress Control**: Restrict incoming traffic to all public internet sources or limit access to internal VPC networks.
*   **Custom Domains**: Map human-readable custom domain names (e.g., `www.example.com`) to the underlying Cloud Run service endpoint.
*   **Binary Authorization**: Enforces policy-based security checks to block untrusted or vulnerable container images from deploying.
*   **Threat Detection**: Continuously monitors workloads for runtime security attacks and structural configuration changes.

---

## 4. Practical Setup & Implementation Guide

### Prerequisites & Dependencies
*   An active Google Cloud project with billing enabled.

### Step-by-Step Implementation Instructions

#### Step 1: Navigate and Initialize Cloud Run
1.  Type **Cloud Run** in the top search bar of the Google Cloud Console and select **Cloud Run**. *(Note: This automatically enables the Cloud Run Admin API).*
2.  Click **Create Service**.

#### Step 2: Configure Container Deployment & Service Identity
1.  Select **Deploy one revision from an existing container image**.
2.  Click **Select** and choose **Test with sample container** (a pre-built Google test container image).
3.  Name the service `hello`.
4.  Choose your target **Region** (e.g., `us-central1`).

#### Step 3: Configure Authentication, Billing, and Scaling
1.  Under **Authentication**, select **Allow public access** (to permit unauthenticated testing).
2.  Under **Pricing / Billing**, select **Request-based billing** (pay only when processing requests).
3.  Under **Scaling**, select **Automatic scaling**:
    *   Set **Minimum number of instances** to `0` (enables scale-to-zero to minimize cost when idle).
    *   Set **Maximum number of instances** to a desired ceiling (e.g., `10`).

#### Step 4: Configure Resources, Networking, and Security
1.  Expand **Containers, networking, security**:
    *   Under **Resources**, verify memory allocation (e.g., `512 MiB`) and vCPU settings.
    *   Under **Networking**, ensure **Ingress** allows all traffic from the internet.
    *   Review security settings like Binary Authorization options.
2.  Click **Create**. Wait a couple of minutes for deployment completion until the status turns ready.

#### Step 5: Test the Service Endpoint
1.  Copy the generated **URL** from the service details page (e.g., `https://hello-xxxx-uc.a.run.app`).
2.  Open a new browser tab, paste the URL, and verify the success message ("*Congratulations. You have successfully deployed a container image to Cloud Run*").

#### Step 6: Explore Tabs (Revisions, Metrics, Logs, Triggers, Networking, Security)
1.  **Revisions**: Review active revisions and use the Actions menu to manage traffic splits.
2.  **Observability**: Check **Metrics** (request counts, CPU/memory utilization), **Logs**, and **Errors**.
3.  **Triggers**: Review event triggers such as Pub/Sub, Cloud Storage, Firestore, or Eventarc.
4.  **Networking**: Explore custom domain mapping options.
5.  **Security**: Review Binary Authorization and threat detection controls.

#### Step 7: Cleanup Resources
1.  To delete the service and avoid lingering charges, select the `hello` service checkbox in the Cloud Run dashboard and click **Delete**. Confirm deletion.

---

## 5. Visual Representations

### Cloud Run Service Configuration Architecture
```
+-----------------------------------------------------------------+
|                      CLOUD RUN SERVICE (hello)                  |
+-----------------------------------------------------------------+
  |
  ├── Container Source: Pre-built Sample Container (Artifact Registry)
  ├── Authentication: Allow Public Access
  ├── Billing Model: Request-Based (Pay per request/processing time)
  ├── Auto-Scaling: Min = 0 (Scale to Zero), Max = 10
  ├── Resources: 512 MiB Memory, vCPU allocation
  └── Security: Binary Authorization & Threat Detection Enabled
```

---

## 6. Mental Models & Analogies

*   **Binary Authorization is like an airport security customs checkpoint for luggage**: Before cargo (container images) is allowed onto the airfield (Cloud Run runtime), airport security agents scan every bag against strict safety manifests. If a bag contains prohibited or unverified items (vulnerabilities), it is rejected at the gate, ensuring only trusted cargo boards the plane.

---

## 7. Step-by-Step Workflows / Comparison Tables

### Cloud Run Service Management Tabs Reference

| Console Tab Menu | Primary Management Purpose |
| :--- | :--- |
| **Revisions** | Track code/config snapshots, manage canary rollouts, and configure traffic splitting. |
| **Observability (Metrics/Logs/Errors)** | Monitor request rates, latencies, CPU/memory utilization, and system error tracing. |
| **Triggers** | Configure event sources like Cloud Storage, Pub/Sub, Firestore, or Eventarc. |
| **Networking** | Control ingress traffic filtering and map custom domains (e.g., `example.com`). |
| **Security** | Configure authentication methods (IAM, IAP), Binary Authorization, and threat detection. |

---

## 8. Common Mistakes & Exam Traps

*   **Trap 1**: Assuming Cloud Run creation requires manual pre-activation of the Cloud Run API before deploying.
    *   *Correction*: Creating a Cloud Run service, job, or worker pool automatically triggers the enablement of the **Cloud Run Admin API** behind the scenes.
*   **Trap 2**: Confusing request-based billing with instance-based billing regarding idle periods.
    *   *Correction*: Under request-based billing combined with scale-to-zero (`min instances = 0`), you incur **zero cost** when idle. Instance-based billing charges for the full lifecycle of running instances regardless of active request processing.
*   **Trap 3**: Believing custom domain mapping handles DNS registration automatically.
    *   *Correction*: While Cloud Run handles custom domain routing mappings, you must still configure your domain registrar's DNS records (such as CNAME or A records) to point to Google Cloud.

---

## 9. Certification Practice Questions

#### Question 1
An administrator wants to secure a Cloud Run service deployment pipeline so that only verified container images meeting strict organizational security policies can be deployed, preventing images with known vulnerabilities. Which Cloud Run security feature should be enabled?
*   A) Identity-Aware Proxy (IAP)
*   B) Binary Authorization
*   C) Cloud Run Threat Detection
*   D) VPC Service Controls perimeter lockdown
*   **Correct Answer**: B
*   **Explanation**: Binary Authorization is a security control that enforces policy-based checks to ensure only trusted container images are deployed to Cloud Run.
*   **Exam Takeaway**: Enforcing trusted container deployments maps to **Binary Authorization**.

#### Question 2
When creating a new Cloud Run service for the first time in a fresh Google Cloud project without prior manual setup, what happens regarding API dependencies?
*   A) Deployment fails until the administrator runs `gcloud services enable run.googleapis.com` manually.
*   B) The Cloud Run Admin API is automatically enabled as part of the resource creation process.
*   C) The service defaults automatically to an App Engine Standard runtime sandbox.
*   D) The project is restricted to local testing mode for 24 hours.
*   **Correct Answer**: B
*   **Explanation**: Creating any Cloud Run resource (service, job, or worker pool) automatically enables the Cloud Run Admin API for you.
*   **Exam Takeaway**: Cloud Run API dependencies are enabled automatically when creating resources.

#### Question 3
An enterprise wants to minimize idle compute costs for a low-traffic web API hosted on Cloud Run. Which combination of billing and scaling configurations achieves zero cost when no requests are being processed?
*   A) Instance-based billing with a minimum instance count of 2.
*   B) Request-based billing with a minimum instance count of 0.
*   C) Instance-based billing with manual scaling fixed at 10 instances.
*   D) Request-based billing with continuous CPU allocation enabled.
*   **Correct Answer**: B
*   **Explanation**: Request-based billing combined with a minimum instance count of 0 allows the service to scale down to zero instances when idle, resulting in zero compute charges.
*   **Exam Takeaway**: Request-based billing + `min instances = 0` achieves zero cost during idle periods.

#### Question 4
An administrator has mapped a custom domain (`www.example.com`) to a Cloud Run service. Beyond configuring the mapping inside the Cloud Run console, what external action is mandatory for domain resolution to succeed?
*   A) Upgrading the GCP project to Enterprise Support tier.
*   B) Updating DNS records (such as CNAME or A records) at the domain registrar to point to Google Cloud.
*   C) Installing an SSL certificate manually on a Compute Engine proxy VM.
*   D) Configuring an external HTTP(S) Load Balancer across multiple regions.
*   **Correct Answer**: B
*   **Explanation**: Mapping a custom domain in Cloud Run requires updating your external domain registrar's DNS records to point traffic to Google Cloud's endpoints.
*   **Exam Takeaway**: Custom domain mapping requires updating DNS records at your domain registrar.

#### Question 5
Which metrics can be tracked out-of-the-box inside the Cloud Run Observability dashboard? (Choose the best combination)
*   A) Request count, end-to-end request latency, active container instance counts, and CPU/memory utilization.
*   B) Physical datacenter room temperature and power grid consumption.
*   C) Local workstation disk read/write throughput of the developer.
*   D) Project credit card expiration dates and billing tier limits.
*   **Correct Answer**: A
*   **Explanation**: The Cloud Run metrics dashboard tracks request metrics, latencies, instance counts, and CPU/memory resource utilization out-of-the-box.
*   **Exam Takeaway**: Cloud Run observability tracks request counts, latencies, instance counts, and resource utilization.

#### Question 6
If an application requires triggers from multiple event sources, which of the following event brokers or storage systems can natively trigger Cloud Run services or functions?
*   A) Cloud Pub/Sub, Cloud Storage, Firestore, and Eventarc.
*   B) BigQuery batch export logs exclusively.
*   C) Cloud SQL direct binary log streams only.
*   D) Cloud Source Repositories commit hooks.
*   **Correct Answer**: A
*   **Explanation**: Cloud Run services and functions can be triggered by events from Cloud Pub/Sub, Cloud Storage, Firestore, and Eventarc.
*   **Exam Takeaway**: Supported event triggers for Cloud Run include Pub/Sub, Cloud Storage, Firestore, and Eventarc.

#### Question 7
What is the operational difference between request-based billing and instance-based billing on Cloud Run?
*   A) Request-based billing charges only when processing requests, whereas instance-based billing charges for the entire lifecycle of the instances.
*   B) Instance-based billing is free, whereas request-based billing requires a monthly subscription.
*   C) Request-based billing requires manual instance scaling, whereas instance-based billing uses serverless auto-scaling.
*   D) Instance-based billing only applies to App Engine Flexible environments.
*   **Correct Answer**: A
*   **Explanation**: Request-based billing charges for actual request processing time, whereas instance-based billing charges for the full operational lifecycle of the container instances.
*   **Exam Takeaway**: Request-based bills per request processing; instance-based bills for the full instance lifecycle.

#### Question 8
Where can administrators configure traffic splitting percentages across multiple active revisions of a Cloud Run service?
*   A) In the `app.yaml` dispatch file exclusively.
*   B) Under the **Revisions** tab using the **Manage Traffic** option in the Cloud Run console.
*   C) Inside the Artifact Registry image tag metadata settings.
*   D) By modifying the VPC network firewall rules.
*   **Correct Answer**: B
*   **Explanation**: Multi-revision traffic splitting is managed directly in the Cloud Run console under the Revisions tab using the Manage Traffic tool.
*   **Exam Takeaway**: Cloud Run revision traffic splitting is managed via the Revisions tab in the console.

#### Question 9
Which security feature in Cloud Run continuously monitors workloads for runtime security attacks and structural configuration modifications?
*   A) Cloud Run threat detection
*   B) Binary Authorization enforcement
*   C) Identity-Aware Proxy (IAP) tunneling
*   D) VPC service perimeter firewalls
*   **Correct Answer**: A
*   **Explanation**: Cloud Run threat detection continuously scans and monitors workloads for runtime attacks and configuration shifts.
*   **Exam Takeaway**: Runtime security monitoring and attack detection are handled by Cloud Run threat detection.

#### Question 10
If a developer wants to restrict incoming traffic to a Cloud Run service so that it only accepts requests from an internal VPC network rather than the public internet, what setting should be modified?
*   A) Authentication settings
*   B) Ingress traffic controls
*   C) Binary Authorization policies
*   D) Custom domain mapping bindings
*   **Correct Answer**: B
*   **Explanation**: Ingress settings allow you to control where traffic originates from, restricting access from the public internet to internal networks only.
*   **Exam Takeaway**: Network accessibility restrictions are controlled via **Ingress** settings.