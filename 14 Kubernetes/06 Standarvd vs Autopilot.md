# Google Kubernetes Engine (GKE) - Complete Chapter Study Notes

---

### 1. Overview
**Google Kubernetes Engine (GKE)** is Google Cloud's managed, production-grade environment for running containerized applications using Kubernetes. It streamlines the deployment, scaling, and operational management of container workloads by leveraging Google's internal infrastructure orchestration experience. 

GKE offers two operational modes [2]:
- **Standard Mode:** Provides granular control over worker node configuration, machine types, pools, and scaling [1, 2]. You manage the worker infrastructure and pay for the underlying virtual machines regardless of pod usage [1].
- **Autopilot Mode:** A hands-off, serverless-like experience where Google manages, provisions, sizes, secures, and repairs nodes automatically [1]. You deploy your applications and pay strictly based on the CPU and memory resources requested by your pods [1].

---

### 2. ACE Exam Focus & Common Exam Traps
- **Exam Importance:** **HIGH**
- **Core Concepts:** 
  - Selecting between Standard and Autopilot based on business or technical requirements.
  - Understanding GKE pricing differences (Node-based billing for Standard vs. Pod-request resource billing for Autopilot) [1].
- **Key Distinctions:**
  - **Autopilot Mode:** Recommended by Google for most workloads [2]; reduces operational effort to zero [1]; automated node provisioning and security best practices enforced by default [1].
  - **Standard Mode:** Required when specialized hardware (GPUs/TPUs) is needed, when node-level access (SSH) or custom software installation is mandatory, or when utilizing Spot VMs for cost optimization [2].
- **Exam Keywords to Watch For:**
  - *"Zero operational overhead, automated node patching, pay per requested CPU/memory"* $\rightarrow$ **GKE Autopilot** [1]
  - *"Attach GPUs/TPUs, SSH into worker nodes, custom kernel configurations, Spot VMs"* $\rightarrow$ **GKE Standard** [2]

---

### 3. Detailed Concept Explanation

#### GKE Standard Mode Architecture & Cost Model
- **Configuration Control:** Administrators configure region/zones, the exact number of worker nodes, node pools, and machine series (e.g., E-series for general-purpose, C-series for compute-optimized) [2].
- **Billing:** Billed directly for the provisioned Compute Engine VM instances making up the cluster nodes, whether or not pods are actively running on them [1].
- **Use Cases:** 
  - High-performance computing (HPC) or Machine Learning workloads requiring specialized hardware such as GPUs or TPUs [2].
  - Scenarios requiring direct node-level access (SSH) to install custom system software or daemon agents [2].
  - Cost-saving architectures leveraging **Spot VMs** for fault-tolerant, non-critical batch processing [2].
  - Integration within complex hybrid or multi-cloud topologies requiring central configuration alignment [2].

#### GKE Autopilot Mode Architecture & Cost Model
- **Zero Node Management:** Google handles cluster node provisioning, scaling, patching, security isolation, and health remediation automatically [1].
- **Billing:** Billed strictly based on the resource requests (CPU and memory) declared in the deployment manifest files of the running pods [1].
- **Advantages:**
  - Enhanced security defaults out of the box [1].
  - Automated safe rolling upgrades for nodes [1].
  - Optimized resource efficiency without idle VM overhead [1].

---

### 4. Practical Setup & Implementation Guide

#### Step 1: Project Setup and API Enablement
1. Open the Google Cloud Console.
2. Create a new project named `my Kubernetes project` (or an equivalent project identifier) and assign an active billing account.
3. Select the newly created project.
4. Navigate to the search bar, type `Kubernetes Engine`, and select **Kubernetes Engine API**.
5. Click **Enable** to initialize the service API within the project.

#### Step 2: Exploring Cluster Creation Modes (Standard vs. Autopilot)
1. Go to **Kubernetes Engine** > **Clusters** and click **Create**.
2. **Autopilot Comparison (Default):** Notice that Autopilot is selected by default as the recommended production approach [2]. It requires only a cluster name and region selection [2].
3. **Standard Mode Inspection:** Click **Switch to standard cluster** to review advanced configuration features [2]:
   - Location type choice: Zonal or Regional clusters [2].
   - Node Pool customization (e.g., default pool sizing, changing node counts) [2].
   - Provisioning models (Standard VMs vs. **Spot VMs**) [2].
   - Hardware machine families (E-series for standard balance, C-series for high CPU performance) and sizing options (medium, micro, small) [2].

#### Step 3: Deploying a GKE Autopilot Cluster
1. Return to **Kubernetes Engine** > **Clusters** and click **Create** [2].
2. Keep the default **Autopilot** mode selected [2].
3. Configure the cluster parameters:
   - **Cluster Name:** `my-cluster` [2]
   - **Region:** Select the default or preferred GCP region [2].
4. Click **Create**. 
   - *Note:* Cluster creation for GKE can take anywhere from 5 to 15 minutes to complete initialization.

#### Equivalent `gcloud` Command Line Reference
To create a standard or autopilot cluster via the Google Cloud CLI:
```bash
# Create an Autopilot cluster (default behavior or explicit flag)
gcloud container clusters create-auto my-cluster \
    --region=us-central1

# Create a Standard cluster with specific node counts and machine types
gcloud container clusters create my-standard-cluster \
    --zone=us-central1-a \
    --num-nodes=3 \
    --machine-type=e2-medium
```

---

### 5. Visual Representations

#### GKE Cluster Creation Decision Flow
```text
Start: Do you need custom nodes, GPUs/TPUs, or node-level SSH access?
               |
               +---> YES ---> Choose GKE STANDARD MODE
               |                - Configure machine types, sizes, and pools.
               |                - Pay for underlying Compute Engine VMs.
               |
               +---> NO ----> Choose GKE AUTOPILOT MODE (Recommended)
                                - Zero node management.
                                - Pay per pod CPU/memory requests.
```

---

### 6. Mental Models & Analogies

- **GKE Autopilot vs. Standard:** Think of **Autopilot** as a fully managed cloud storage drive where you only pay for the gigabytes you write, without worrying about buying physical hard disks or replacing broken circuit boards. **Standard** is like renting an empty server rack in a datacenter: you choose the exact hardware model, plug it in yourself, manage its cooling, and pay rent for the physical box regardless of how much data you store on it.

---

### 7. Step-by-Step Workflows / Comparison Tables

#### GKE Operational Modes Workflow Comparison

| Operational Phase | GKE Standard Workflow | GKE Autopilot Workflow |
| :--- | :--- | :--- |
| **1. Initialization** | Choose region/zone, define node counts, machine types, and storage parameters [2]. | Select region and specify cluster name [2]. |
| **2. Infrastructure Maintenance** | Manual or scheduled node image upgrades, patching, and OS maintenance [1]. | Completely automated and safe background updates managed by Google [1]. |
| **3. Scaling Operations** | Manual or autoscaled node pool modification [1]. | Automatic pod-driven provisioning of underlying capacity [1]. |
| **4. Cost Allocation** | Billed based on active VM instance hours [1]. | Billed based on requested pod CPU and memory units [1]. |

---

### 8. Common Mistakes & Exam Traps

- **Trap 1:** Believing that GKE Autopilot charges you for the total cluster capacity allocated by Google.
  - *Correction:* You are billed based on the **resource requests (CPU and memory) defined in your workload specs**, not the actual total underlying node size [1].
- **Trap 2:** Choosing Autopilot when the scenario explicitly requires connecting specialized hardware devices like GPUs.
  - *Correction:* Specialized hardware attachments (GPUs/TPUs) and custom kernel flags require **GKE Standard** [2].

---

### 9. Certification Practice Questions

#### Q1: Your company is deploying a new containerized microservices application on GKE and wants to minimize operational overhead. The team should not have to manage worker nodes, handle upgrades, or perform capacity sizing. Which GKE mode should you choose?
- A. GKE Standard mode with cluster autoscaler enabled
- B. GKE Autopilot mode
- C. Compute Engine with managed instance groups and Docker container deployment
- D. GKE Standard mode with preemptible VMs
- **Correct Answer:** B
- **Explanation:** GKE Autopilot mode is designed to reduce operational effort to zero by managing node creation, scaling, patching, and security automatically.
- **Incorrect Options:** A and D use GKE Standard, which still places the burden of node configuration and management on the administrator. C uses raw Compute Engine rather than managed Kubernetes.
- **Exam Takeaway:** Minimal operational overhead and hands-off node maintenance point directly to GKE Autopilot.

#### Q2: A machine learning team requires a GKE cluster to run training workloads that depend on dedicated NVIDIA GPUs. Which configuration approach should you select?
- A. GKE Autopilot cluster with automatic resource allocation
- B. GKE Standard cluster with node pools configured for GPU machine types
- C. Google App Engine flexible environment
- D. Cloud Run with GPU scaling flags enabled
- **Correct Answer:** B
- **Explanation:** GKE Standard allows full control over node configurations, including attaching specialized hardware like GPUs or TPUs.
- **Incorrect Options:** A (Autopilot does not offer the same level of granular custom hardware configurations like specialized node pool instance selection). C and D do not support direct GPU infrastructure attachment in the context of standard ML workloads running on Kubernetes.
- **Exam Takeaway:** Specialized hardware (GPUs/TPUs) requires GKE Standard.

#### Q3: How is billing calculated when running workloads on a GKE Autopilot cluster?
- A. By the total number and machine size of underlying worker nodes provisioned by Google
- B. Based on the amount of CPU and memory requested by your deployed pods
- C. Flat monthly rate per active namespace inside the cluster
- D. Per inbound network packet processed by the cluster load balancer
- **Correct Answer:** B
- **Explanation:** In GKE Autopilot mode, customers are billed based on the resource requests (CPU and memory) specified in their pod configurations.
- **Incorrect Options:** A describes the billing mechanism for GKE Standard mode. C and D are incorrect pricing structures.
- **Exam Takeaway:** Autopilot bills by pod resource requests; Standard bills by provisioned node virtual machines.

#### Q4: An administrator needs to troubleshoot a low-level network issue inside a GKE cluster node by securely logging in via SSH and inspecting kernel modules. Which GKE cluster architecture supports this requirement?
- A. GKE Autopilot cluster
- B. GKE Standard cluster
- C. Cloud Run service with debug mode enabled
- D. Google Kubernetes Engine serverless pods
- **Correct Answer:** B
- **Explanation:** GKE Standard provides node-level access, allowing administrators to SSH into nodes and perform system-level troubleshooting or custom software installation.
- **Incorrect Options:** A, C, and D abstract away the underlying infrastructure, denying direct node-level access.
- **Exam Takeaway:** Node-level access (SSH) is exclusive to GKE Standard.

#### Q5: You want to optimize costs for a batch processing workload running on GKE that is fault-tolerant and can be interrupted at any time. Which GKE Standard configuration option should you use?
- A. Sole-tenant nodes
- B. Spot VMs
- C. Compute Engine regional persistent disks
- D. Extended memory machine types
- **Correct Answer:** B
- **Explanation:** Spot VMs are heavily discounted, short-lived virtual machine instances ideal for fault-tolerant, non-immediate batch workloads.
- **Incorrect Options:** A (Sole-tenant nodes dedicate physical hardware to a single project, increasing costs). C and D do not address node cost optimization for batch tasks.
- **Exam Takeaway:** Cost-optimized batch workloads on GKE Standard frequently use Spot VMs.

#### Q6: Which entity is fully managed by Google Cloud in *both* GKE Standard and GKE Autopilot deployment modes?
- A. Worker nodes
- B. Node auto-repair policies
- C. The Kubernetes control plane (API server, etcd)
- D. Application container code
- **Correct Answer:** C
- **Explanation:** In both GKE Standard and Autopilot, Google manages the control plane components (such as the API server and etcd database).
- **Incorrect Options:** A and B are managed by Google only in Autopilot (the user manages nodes in Standard). D is always managed by the developer.
- **Exam Takeaway:** The control plane is always managed by Google in GKE, regardless of whether you choose Standard or Autopilot.

#### Q7: Your engineering team prefers a managed Kubernetes environment but wants to define custom node machine types and install custom daemon software on every node. Which choice satisfies these constraints?
- A. GKE Autopilot
- B. GKE Standard
- C. Google Kubernetes Engine serverless add-on
- D. Cloud Functions
- **Correct Answer:** B
- **Explanation:** GKE Standard allows custom node selection and software installation via node-level configurations.
- **Incorrect Options:** A, C, and D do not allow custom node-level software installation or custom machine type tuning.
- **Exam Takeaway:** Custom node configurations require GKE Standard.

#### Q8: What is the primary goal of GKE Autopilot mode?
- A. To provide the cheapest possible compute rates regardless of configuration
- B. To reduce Kubernetes operational effort to zero by automating node management
- C. To allow direct access to physical infrastructure datacenters
- D. To eliminate the need for writing Kubernetes YAML manifests
- **Correct Answer:** B
- **Explanation:** Autopilot aims to eliminate infrastructure and node management tasks, lowering operational effort to zero.
- **Incorrect Options:** A is false (pricing is resource-based, not universally the lowest). C and D are inaccurate statements about Autopilot.
- **Exam Takeaway:** Autopilot's core design goal is eliminating operational effort through automation.

#### Q9: When creating a GKE Standard cluster using the Google Cloud Console, which of the following location options can you select?
- A. Global or Continental only
- B. Zonal or Regional
- C. Local edge zones exclusively
- D. Standalone project regions only
- **Correct Answer:** B
- **Explanation:** GKE Standard clusters can be deployed as either single-zone (zonal) or multi-zone (regional) configurations.
- **Incorrect Options:** A, C, and D use incorrect terminology for GKE location structures.
- **Exam Takeaway:** GKE clusters can be configured as Zonal or Regional.

#### Q10: How does GKE Autopilot handle cluster scaling when a new resource-heavy application deployment is submitted?
- A. It requires the administrator to manually add more worker nodes via the console.
- B. It automatically provisions and scales the necessary underlying node capacity based on the application's pod resource requests.
- C. It rejects the deployment if it exceeds the initial cluster node limit.
- D. It pauses the deployment until a GKE Standard migration is performed.
- **Correct Answer:** B
- **Explanation:** Autopilot dynamically provisions and adjusts node capacity automatically based on incoming pod resource requirements.
- **Incorrect Options:** A describes manual management found in Standard. C and D are false behavior models.
- **Exam Takeaway:** Autopilot handles node scaling automatically based on pod demands.