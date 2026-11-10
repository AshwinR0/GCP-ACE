# Google Kubernetes Engine (GKE) - Console Walkthrough Study Notes

---

### 1. Overview
This module transitions from theoretical concepts to practical execution within the Google Cloud Console [3]. It demonstrates the step-by-step process of setting up a new GCP project, enabling the required Kubernetes APIs, exploring both **Standard** and **Autopilot** cluster creation interfaces, and finally initiating the creation of a production-ready **GKE Autopilot** cluster named `my-cluster` [3].

---

### 2. ACE Exam Focus & Common Exam Traps
- **Exam Importance:** **HIGH**
- **Core Concepts:** 
  - Prerequisite steps before launching a GKE cluster (Project creation, Billing account assignment, and enabling the `Kubernetes Engine API`).
  - Distinguishing the UI configuration options available when toggling between Standard and Autopilot modes [3].
- **Key Distinctions:**
  - **Standard Cluster Setup UI:** Exposes detailed infrastructure choices such as Zonal vs. Regional selection, node pool sizing, explicit node counts (defaulting to 3), provisioning models (Standard vs. Spot VMs), and machine families/sizes (E-series vs. C-series) [3].
  - **Autopilot Cluster Setup UI:** Streamlines deployment by stripping away node configuration complexity, requiring only a cluster name and region selection [3].
- **Exam Keywords to Watch For:**
  - *"Before you can make use of the Kubernetes engine service"* $\rightarrow$ **Enable the Kubernetes Engine API** [3].
  - *"Cluster creation duration"* $\rightarrow$ Can take anywhere from **5 to 15 minutes** to successfully provision [3].

---

### 3. Detailed Concept Explanation

#### Project Setup & Initialization Workflow
- **Prerequisites:** Every resource in Google Cloud resides inside a project. Before launching any GKE cluster, a project must be linked to a valid billing account [3].
- **API Activation:** GCP services are modular. The **Kubernetes Engine API** must be explicitly enabled before the control plane and worker infrastructure can be provisioned via the console or CLI [3].

#### Interface Differences (Standard vs. Autopilot in Console)
1. **Default Presentation:** Upon clicking *Create* under Kubernetes Clusters, the console defaults to **Autopilot** mode because Google recommends it for most production deployments due to its minimized overhead [3].
2. **Standard Customization Panels:** Toggling to *Standard Mode* exposes deep architectural controls:
   - **Location Type:** Choosing between a *Zonal cluster* (single zone, lower fault tolerance) or a *Regional cluster* (multi-zone, highly available control plane and worker nodes).
   - **Node Pools:** Defining the initial number of worker nodes (e.g., default starting count of 3) [3].
   - **Provisioning Models:** Selecting standard VMs or cost-efficient Spot VMs [3].
   - **Machine Series & Sizes:** Choosing optimized hardware families like **E-series** for cost-effective daily computing or **C-series** for high CPU demand, along with machine sizes (micro, small, medium) [3].

---

### 4. Practical Setup & Implementation Guide

#### Complete Walkthrough: Creating a GKE Autopilot Cluster via Console

1. **Create a Dedicated Project:**
   - Navigate to the top navigation bar in the Google Cloud Console and click the project dropdown.
   - Click **New Project**.
   - **Project Name:** `my Kubernetes project` [3].
   - Select an active **Billing Account** [3].
   - Click **Create** [3]. Once created, click **Select Project** [3].

2. **Enable the Kubernetes Engine API:**
   - In the top search bar, type `Kubernetes` [3].
   - Select **Kubernetes Engine** from the product results.
   - If prompted, click **Enable** to activate the Kubernetes Engine API. *(Note: This process takes 1–2 minutes)* [3].

3. **Navigate to Cluster Management:**
   - In the left-hand navigation menu, go to **Kubernetes Engine** > **Clusters** [3].
   - Click **Create** [3].

4. **Explore Standard Mode (Optional Interface Inspection):**
   - Click **Switch to standard cluster** (observe the warning indicating that Google recommends Autopilot for most production workloads) [3].
   - Review the available options: Location type (Zonal/Regional), default node pools (default count: 3), provisioning models (Spot VMs), and node machine families (E-series for low cost, C-series for high CPU) [3].

5. **Configure and Launch the Autopilot Cluster:**
   - Navigate back to **Kubernetes Engine** > **Clusters** and click **Create** again to return to the default **Autopilot** mode [3].
   - Enter **Cluster Name:** `my-cluster` [3].
   - Select the suggested default **Region** [3].
   - Click **Create** [3].
   - Monitor the progress banner noting that cluster creation takes 5 to 15 minutes [3].

#### Equivalent `gcloud` Commands
```bash
# 1. Create a project and set it as default (CLI alternative)
gcloud projects create my-kubernetes-project --name="my Kubernetes project"
gcloud config set project my-kubernetes-project

# 2. Enable the Kubernetes Engine API
gcloud services enable container.googleapis.com

# 3. Create an Autopilot cluster via CLI
gcloud container clusters create-auto my-cluster \
    --region=us-central1
```

---

### 5. Visual Representations

#### Console Navigation Hierarchy for GKE Creation
```text
Google Cloud Console Root
  |
  +---> Project Selector ---> Create Project ("my Kubernetes project") + Billing
  |
  +---> Search bar ---> "Kubernetes Engine" ---> Enable API
  |
  +---> Kubernetes Engine Menu ---> Clusters ---> Click "Create"
          |
          +---> [Default] Autopilot Mode selected
          |       - Name: my-cluster
          |       - Region: default region
          |       - Action: Click Create (Provisioning takes 5-15 mins)
          |
          +---> [Alternative] Switch to Standard Mode
                  - Location Type (Zonal / Regional)
                  - Default Node Pools (Node count, e.g., 3)
                  - Provisioning Model (Standard / Spot VMs)
                  - Node Configuration (E-series, C-series, sizing)
```

---

### 6. Mental Models & Analogies

- **Enabling the API is like turning on the main circuit breaker in a new house:** Before you can plug in appliances (deploy pods or create clusters), you have to flip the utility switch (enable the API) so the infrastructure has power and authorization to function.

---

### 7. Step-by-Step Workflows / Comparison Tables

#### GKE Console Provisioning Workflow

| Step Number | Operational Action | Console Navigation Path / Details |
| :--- | :--- | :--- |
| **1** | Project Provisioning | Top Bar Dropdown $\rightarrow$ *New Project* $\rightarrow$ Name project $\rightarrow$ Link billing [3]. |
| **2** | Service Activation | Top Search Bar $\rightarrow$ *Kubernetes Engine* $\rightarrow$ Click *Enable API* [3]. |
| **3** | Cluster Menu Access | Left Navigation $\rightarrow$ *Kubernetes Engine* $\rightarrow$ *Clusters* $\rightarrow$ *Create* [3]. |
| **4** | Mode Selection | Choose between *Autopilot* (Default/Recommended) or *Standard* [3]. |
| **5** | Parameter Input | Enter Cluster Name (`my-cluster`), select Region, and click *Create* [3]. |

---

### 8. Common Mistakes & Exam Traps

- **Trap 1:** Forgetting to enable the `container.googleapis.com` API before running scripts or trying to configure clusters via the console.
  - *Correction:* Always ensure the service API is enabled; otherwise, cluster deployment commands will fail with API authorization or service-not-found errors.
- **Trap 2:** Assuming GKE clusters finish building instantaneously.
  - *Correction:* Cluster provisioning takes between 5 to 15 minutes. Attempting to deploy applications immediately before the control plane and nodes report a healthy status will result in connection timeouts [3].

---

### 9. Certification Practice Questions

#### Q1: You are setting up a fresh GCP project to deploy a containerized application on GKE using the Google Cloud Console. After creating and selecting the project, you navigate to Kubernetes Engine, but the system prompts you that the service is unavailable. What step was likely missed?
- A. The Compute Engine instance limit was reached.
- B. The Kubernetes Engine API was not enabled for the project.
- C. The user account lacks the Project Owner IAM role.
- D. A regional network VPC firewall rule is blocking access.
- **Correct Answer:** B
- **Explanation:** Before any service like GKE can be used in a new project, its corresponding API must be explicitly enabled [3].
- **Incorrect Options:** A, C, and D are unrelated to the initial API activation requirement for utilizing a GCP service dashboard.
- **Exam Takeaway:** Always verify that service APIs are enabled before attempting resource deployment.

#### Q2: When creating a GKE cluster via the Google Cloud Console, which operational mode is selected by default to align with Google's recommended production practices?
- A. GKE Standard mode
- B. GKE Autopilot mode
- C. Managed Instance Group container mode
- D. Anthos hybrid container mode
- **Correct Answer:** B
- **Explanation:** Autopilot is the default cluster creation mode in the console because Google recommends it for most production workloads due to its automated best practices and minimal operational overhead [3].
- **Incorrect Options:** A is an advanced alternative mode. C and D refer to different architectural options.
- **Exam Takeaway:** Autopilot is the default selection in the GKE creation console.

#### Q3: During the configuration of a GKE Standard cluster in the console, you examine the default node pools section. What is the default initial number of worker nodes suggested by the interface?
- A. 1 node
- B. 3 nodes
- C. 5 nodes
- D. 10 nodes
- **Correct Answer:** B
- **Explanation:** The console interface defaults the initial worker node count to 3 for standard cluster deployments [3].
- **Incorrect Options:** A, C, and D do not match the default pre-populated node count shown in the standard cluster creation screen.
- **Exam Takeaway:** Standard GKE console creation defaults to an initial pool count of 3 nodes [3].

#### Q4: While creating a GKE Standard cluster, you review the node machine configuration choices. Which of the following machine series options does the console explicitly highlight for low-cost, day-to-day computing?
- A. C-series
- B. N-series
- C. E-series
- D. M-series
- **Correct Answer:** C
- **Explanation:** The E-series machine family is highlighted for low-cost, day-to-day general-purpose computing workloads [3].
- **Incorrect Options:** A (C-series is optimized for high compute power). B and D represent alternative heavy-duty or memory-optimized families.
- **Exam Takeaway:** E-series instances are targeted for low-cost, routine computing tasks [3].

#### Q5: You have just clicked "Create" on a new GKE Autopilot cluster named `my-cluster` in the Google Cloud Console. What should you expect immediately following submission?
- A. The cluster is ready instantly, and kubectl pods can be scheduled.
- B. A status message indicating that cluster creation can take 5 minutes or more (sometimes up to 15 minutes).
- C. An immediate authentication prompt requiring an SSH key upload.
- D. A mandatory redirection to configure a Cloud NAT gateway.
- **Correct Answer:** B
- **Explanation:** GKE cluster creation takes time to provision the underlying control plane and infrastructure, typically displaying a status notice that it takes 5 to 15 minutes [3].
- **Incorrect Options:** A is incorrect because clusters are not ready instantly. C and D are not automatic prerequisites for basic autopilot creation.
- **Exam Takeaway:** Expect a waiting period of 5 to 15 minutes when provisioning new GKE clusters [3].

#### Q6: Which of the following parameters is mandatory when creating a GKE Autopilot cluster via the console?
- A. Explicit machine size configuration (e.g., e2-medium)
- B. Cluster name and region selection
- C. Manual worker node count specification
- D. Custom SSH private key upload
- **Correct Answer:** B
- **Explanation:** Autopilot simplifies setup; users only need to provide a cluster name and select the target region [3].
- **Incorrect Options:** A and C apply to Standard mode node configuration, not Autopilot. D is unnecessary.
- **Exam Takeaway:** Autopilot cluster creation requires minimal inputs like a cluster name and region [3].

#### Q7: If an engineer wants to use Spot VMs to save costs on a GKE deployment, which cluster mode must they select during the setup phase?
- A. GKE Autopilot mode
- B. GKE Standard mode
- C. Cloud Run container mode
- D. App Engine standard mode
- **Correct Answer:** B
- **Explanation:** Spot VMs and custom provisioning models are configured under the node pool settings of GKE Standard mode [3].
- **Incorrect Options:** A (Autopilot abstracts node configuration entirely, disallowing manual Spot VM node selection). C and D are serverless platforms, not GKE modes.
- **Exam Takeaway:** Spot VMs require GKE Standard mode selection.

#### Q8: When reviewing the warning message when switching from Autopilot to Standard mode in the console, what does Google emphasize regarding control plane management?
- A. Google manages the control plane in Standard mode, but you manage more configuration settings in your cluster and nodes.
- B. Google relinquishes all control plane management to the user in Standard mode.
- C. The control plane is disabled entirely in Standard mode.
- D. Control plane billing is waived in Standard mode.
- **Correct Answer:** A
- **Explanation:** The warning states that even in Standard mode, Google manages your control plane, but you gain control over cluster and node configurations [3].
- **Incorrect Options:** B, C, and D misrepresent the shared responsibility model where Google always manages the Kubernetes control plane in GKE.
- **Exam Takeaway:** Google manages the GKE control plane across both operational modes.

#### Q9: What interface action do you perform to inspect advanced node configuration settings like Zonal vs. Regional layout during cluster creation?
- A. Click "Switch to standard cluster"
- B. Click "Enable advanced IAM"
- C. Navigate to Cloud Storage settings
- D. Select "Serverless containers"
- **Correct Answer:** A
- **Explanation:** Switching to a standard cluster opens up granular infrastructure choices including location types (Zonal/Regional) and node pool details [3].
- **Incorrect Options:** B, C, and D are invalid paths for revealing GKE node options.
- **Exam Takeaway:** Advanced infrastructure controls are accessed by switching to GKE Standard mode in the console [3].

#### Q10: Why does the Google Cloud Console recommend Autopilot mode as the default option for new users building Kubernetes clusters?
- A. It provides root SSH access to every physical machine rack.
- B. It optimizes for most production workloads and implements automatic best practices with minimal operational effort.
- C. It is the only mode that supports zero billing.
- D. It allows infinite CPU allocation without service quotas.
- **Correct Answer:** B
- **Explanation:** Autopilot is recommended because it is optimized for most production workloads, automates best practices, and minimizes operational overhead [3].
- **Incorrect Options:** A, C, and D are factually incorrect.
- **Exam Takeaway:** Autopilot is recommended for production because it enforces best practices and reduces operational friction.