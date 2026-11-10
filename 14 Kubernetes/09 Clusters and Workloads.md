# Google Kubernetes Engine (GKE) - Console UI Navigation & Workloads Study Notes

---

### 1. Overview
This module explores the Google Kubernetes Engine (GKE) console dashboard, focusing on the **Clusters** and **Workloads** management tabs. It breaks down how to monitor resource metrics (CPU and memory utilization), interpret cost-optimization recommendations, manage GKE release channels, review cluster versions, inspect deployed workloads, and understand how pod resource requests map to infrastructure billing in GKE Autopilot.

---

### 2. ACE Exam Focus & Common Exam Traps
- **Exam Importance:** **HIGH**
- **Core Concepts:**
  - **GKE Release Channels:** Rapid, Regular, and Stable [5].
  - **Resource Requests & Limits:** Controlling container compute allocation and how it directly affects costs in Autopilot mode [5].
  - **Kubernetes Declarative Configuration:** Managing workloads using YAML definition manifests rather than imperative CLI commands [5].
- **Key Distinctions:**
  - **Release Channels:**
    - *Rapid:* Cutting-edge features immediately upon release (less stable) [5].
    - *Regular:* Balanced feature set and standard stability (default channel) [5].
    - *Stable:* Prioritizes absolute stability for production environments over new features [5].
- **Exam Keywords to Watch For:**
  - *"Prioritize stability for production environments over newest features"* $\rightarrow$ **Stable release channel** [5].
  - *"Get the newest features immediately"* $\rightarrow$ **Rapid release channel** [5].
  - *"Right-sizing pods to lower cost in GKE Autopilot"* $\rightarrow$ Adjusting CPU and memory **resource requests** [5].

---

### 3. Detailed Concept Explanation

#### GKE Cluster UI Tabs and Telemetry
- **Overview Tab:** Summarizes health metrics, cluster version, regional configuration, and basic CPU/memory allocation.
- **Utilization & Observability Tabs:** Displays real-time and historical graphs tracking CPU utilization, memory usage, and network traffic across the cluster.
- **Cost Optimization Tab:** Highlights how efficiently provisioned compute resources are being used and provides cost-saving recommendations.
- **Storage Tab:** Tracks persistent storage volumes (Persistent Disks, Filestore) attached to the cluster.

#### GKE Release Channels
Google offers automated upgrade management through three distinct release channels:
1. **Rapid Channel:** Delivers new Kubernetes versions and features as soon as they are tested and released by Google. Best for development environments and bleeding-edge testing.
2. **Regular Channel:** The default channel. Balances new feature availability with solid version stability.
3. **Stable Channel:** Focuses heavily on version reliability and stability. Upgrades occur less frequently and are heavily tested, making it ideal for core production environments [5].
*Note:* Clusters can also be manually upgraded to a target version if automated channels are not preferred [5].

#### Workloads Tab & Pod Management
- **Workloads Overview:** Displays active deployments, status, and the number of running pods associated with each deployment.
- **Pods:** The smallest deployable units of computing in Kubernetes. A deployment can manage one or multiple pod instances (replicas) [5].
- **Resource Requests and Cost Optimization (Right-Sizing):** 
  - In GKE Autopilot, billing is strictly derived from the CPU and memory **resource requests** specified in the workload configuration, not just the raw idle consumption [5].
  - If a deployment requests 2 GB of memory but only uses 0.2 GB, you are still billed for the requested 2 GB [5]. Tuning these values is known as **right-sizing** [5].
- **Declarative YAML Configuration:** While workloads can be created via imperative commands (`kubectl create deployment`), Kubernetes operates declaratively using **YAML configuration files**, which can be inspected directly inside the UI's YAML tab [5].

---

### 4. Practical Setup & Implementation Guide
*Note: This module focuses on console telemetry and UI navigation rather than new infrastructure setup steps. To review cluster creation, refer to the previous practical guides.*

#### Inspecting Workloads and Pod Metrics via Console
1. Navigate to **Kubernetes Engine** > **Workloads** in the Google Cloud Console [5].
2. Click on the deployment name (`hello-world-rest-api`) to view detailed metrics [5]:
   - Review CPU and memory utilization graphs.
   - Inspect active revisions and the number of running pod replicas [5].
3. Check the **YAML** tab to view the declarative manifest configuration underlying the deployment [5].
4. Access the service endpoint through the provided LoadBalancer IP link or append `/hello-world` to verify application response [5].

---

### 5. Visual Representations

#### GKE Release Channels Hierarchy
```text
[ GKE Release Channels ]
  |
  +---> Rapid Channel ---> Earliest access to new features (Dev / Testing) [5]
  |
  +---> Regular Channel -> Balanced features and stability (Default) [5]
  |
  +---> Stable Channel --> Maximum stability and reliability (Production) [5]
```

#### Pod Resource Request vs. Billing Relationship in Autopilot
```text
+-------------------------------------------------------------+
|               Kubernetes Pod Manifest Spec                  |
|     resources:                                              |
|       requests:                                             |
|         cpu: "500m"      <--- Drives GKE Autopilot Billing  |
|         memory: "2Gi"    <--- Drives GKE Autopilot Billing  |
+-------------------------------------------------------------+
                              |
                              v
          [ Billed for Requested Units (2 Gi / 0.5 CPU) ]
           (Even if actual container usage is only 0.2 Gi) [5]
```

---

### 6. Mental Models & Analogies

- **Release Channels are like software update tracks on a smartphone:** 
  - **Rapid** is the developer beta channel where you get new features immediately, but bugs might occur.
  - **Regular** is the standard official OS update channel used by most consumers.
  - **Stable** is the enterprise-locked long-term support version where security patches are applied, but UI changes are avoided to prevent workflow disruptions.
- **Resource Requests are like booking hotel conference rooms:** You pay for the room size you book (request), not how many people actually show up inside it [5]. If you book a 100-person room for a 5-person meeting, you still pay for the 100-person capacity unless you right-size your reservation.

---

### 7. Step-by-Step Workflows / Comparison Tables

#### GKE Release Channels Comparison Table

| Release Channel | Target Environment | Update Frequency | Stability Level |
| :--- | :--- | :--- | :--- |
| **Rapid** | Development / Non-Production [5] | Highest (Immediate) | Lowest |
| **Regular** | General Purpose / Staging [5] | Moderate (Default) | Standard |
| **Stable** | Core Production Workloads [5] | Lowest (Prioritized) | Highest |

---

### 8. Common Mistakes & Exam Traps

- **Trap 1:** Assuming billing in GKE Autopilot is based purely on active, real-time CPU/memory utilization spikes.
  - *Correction:* Billing is driven by the configured **resource requests** specified in the pod YAML manifests. If a pod requests massive amounts of memory that it doesn't use, you are still billed for those requested amounts [5].
- **Trap 2:** Choosing the *Rapid* release channel for a mission-critical financial database running in production.
  - *Correction:* Production workloads requiring maximum stability should always use the **Stable** release channel [5].

---

### 9. Certification Practice Questions

#### Q1: An enterprise financial institution is deploying a mission-critical payment processing application onto GKE. The security and operations teams mandate that the cluster must prioritize platform stability over cutting-edge Kubernetes features. Which GKE release channel should be selected?
- A. Rapid release channel
- B. Regular release channel
- C. Stable release channel
- D. Experimental release channel
- **Correct Answer:** C
- **Explanation:** The Stable release channel prioritizes reliability and version stability over new features, making it ideal for core production environments [5].
- **Incorrect Options:** A (Rapid delivers bleeding-edge features quickly but carries higher stability risks). B (Regular is the default channel). D is a fictitious channel name.
- **Exam Takeaway:** Production environments requiring high stability should use the Stable release channel [5].

#### Q2: A development team wants to test upcoming GKE features and Kubernetes updates as soon as they become available, accepting a higher risk of bugs or breaking changes. Which GKE release channel fits this use case?
- A. Stable release channel
- B. Regular release channel
- C. Rapid release channel
- D. Offline release channel
- **Correct Answer:** C
- **Explanation:** The Rapid release channel provides early access to new GKE features and Kubernetes versions as soon as they are released [5].
- **Incorrect Options:** A and B prioritize higher stability and slower rollout schedules. D is invalid.
- **Exam Takeaway:** Early feature adoption points directly to the Rapid release channel [5].

#### Q3: How is pricing calculated for workloads running inside a GKE Autopilot cluster when applications specify high CPU and memory resource requests compared to their actual low average consumption?
- A. Billing is calculated based on actual real-time CPU and memory utilization metrics.
- B. Billing is calculated based on the CPU and memory resource **requests** declared in the workload configuration [5].
- C. Billing is waived if the container uses less than 10% of provisioned resources.
- D. Billing is based strictly on the number of namespaces created in the cluster.
- **Correct Answer:** B
- **Explanation:** In GKE Autopilot, billing is based on the resource requests (CPU and memory) defined in your workload specs, regardless of low average runtime utilization [5].
- **Incorrect Options:** A, C, and D misrepresent the Autopilot resource-request billing mechanism.
- **Exam Takeaway:** Autopilot billing is governed by pod resource requests [5].

#### Q4: An administrator notices that a GKE Autopilot deployment is requesting significantly more memory than it actually consumes during normal operations. What optimization practice should they perform to reduce cluster costs?
- A. Switch the cluster from Autopilot to Standard mode.
- B. Right-size the deployment by updating the resource requests and limits in the pod configuration [5].
- C. Delete all persistent storage disks attached to the cluster.
- D. Enable the Rapid release channel.
- **Correct Answer:** B
- **Explanation:** Right-sizing involves setting optimal CPU and memory resource requests and limits to avoid over-provisioning and lowering Autopilot costs [5].
- **Incorrect Options:** A is an overly complex architectural change. C and D do not address pod memory over-provisioning.
- **Exam Takeaway:** Right-sizing resource requests helps control and reduce costs in GKE Autopilot [5].

#### Q5: Which GKE console tab provides observability metrics regarding CPU utilization, memory usage, and network traffic across the cluster?
- A. Storage tab
- B. Observability tab (or Utilization tab) [5]
- C. IAM & Admin tab
- D. Billing export tab
- **Correct Answer:** B
- **Explanation:** The Observability and Utilization tabs track performance metrics like CPU, memory, and network utilization [5].
- **Incorrect Options:** A tracks persistent disks and filestore volumes. C handles permissions. D manages overall cloud billing configuration.
- **Exam Takeaway:** Cluster resource telemetry and performance metrics are found under Observability and Utilization tabs [5].

#### Q6: When reviewing a deployment inside the GKE Workloads console tab, what is each individual running instance of the application container called?
- A. A node pool
- B. A pod [5]
- C. A service mesh
- D. A namespace
- **Correct Answer:** B
- **Explanation:** In Kubernetes, application instances run inside pods, which are the smallest deployable units managed within a deployment [5].
- **Incorrect Options:** A represents a grouping of worker nodes. C refers to service communication layers. D is a virtual cluster isolation boundary.
- **Exam Takeaway:** Individual running application instances inside a deployment are called pods [5].

#### Q7: True or False: Kubernetes is declarative, meaning you can inspect and manage workloads using configuration manifest files written in YAML format.
- A. True [5]
- B. False
- **Correct Answer:** A
- **Explanation:** Kubernetes uses declarative configuration principles where desired states are defined via YAML files (inspectable in the console's YAML tab) [5].
- **Exam Takeaway:** Kubernetes supports declarative configuration via YAML manifests [5].

#### Q8: What is the default release channel assigned to a standard GKE cluster if no alternative option is explicitly selected during creation?
- A. Rapid
- B. Regular [5]
- C. Stable
- D. Custom
- **Correct Answer:** B
- **Explanation:** Regular is the default release channel assigned when creating GKE clusters [5].
- **Incorrect Options:** A and C are selectable choices but are not the default. D is invalid.
- **Exam Takeaway:** The Regular release channel is the default setting for GKE clusters [5].

#### Q9: If an administrator wants to manually upgrade a GKE cluster instead of relying on an automated release channel, what mechanism is available in the console cluster details page?
- A. A manual version upgrade button [5]
- B. Running an automated shell script via Cloud Storage
- C. Deleting the project and recreating it with a new version flag
- D. Modifying the project IAM policy bindings
- **Correct Answer:** A
- **Explanation:** The console provides a manual upgrade option allowing administrators to select a target Kubernetes version directly from the cluster details page [5].
- **Incorrect Options:** B, C, and D are incorrect or overly disruptive mechanisms.
- **Exam Takeaway:** GKE clusters support both automated release channels and manual version upgrades [5].

#### Q10: What purpose does the "Cost Optimization" tab serve in the GKE console interface?
- A. It automatically deletes inactive GCP projects.
- B. It displays how efficiently compute resources are utilized and offers recommendations for optimizing cluster costs [5].
- C. It alters the organization's primary billing credit card.
- D. It restricts network traffic to lower egress fees.
- **Correct Answer:** B
- **Explanation:** The Cost Optimization tab evaluates compute resource efficiency and provides recommendations to help reduce infrastructure expenses [5].
- **Incorrect Options:** A, C, and D are unrelated to GKE cost optimization insights.
- **Exam Takeaway:** Cost optimization tabs provide recommendations for improving cluster resource efficiency [5].