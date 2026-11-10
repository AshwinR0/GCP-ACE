# Google Kubernetes Engine (GKE) - Horizontal Pod Autoscaling (HPA) and Node Scaling Study Notes

---

### 1. Overview
This module explores advanced scaling concepts in GKE, divided into two distinct dimensions: **Pod-level scaling** (Horizontal Pod Autoscaling using `kubectl autoscale`) and **Node-level scaling** (managing and autoscaling worker nodes in **GKE Standard mode** vs. **GKE Autopilot mode**) [7]. While Autopilot abstracts node management entirely, running clusters in Standard mode requires administrators to actively manage worker node capacity—either via manual resizing commands (`gcloud container clusters resize`) or node autoscaling updates (`gcloud container clusters update`)—to ensure pods have sufficient compute resources to run [7].

---

### 2. ACE Exam Focus & Common Exam Traps
- **Exam Importance:** **HIGH**
- **Core Concepts:**
  - Horizontal Pod Autoscaler (HPA) concept and command syntax (`kubectl autoscale deployment`) [7].
  - Inspecting HPA metrics using `kubectl get hpa` [7].
  - Node pool sizing and resizing in GKE Standard mode using `gcloud container clusters resize` [7].
  - Enabling node autoscaling in GKE Standard using `gcloud container clusters update` [7].
- **Key Distinctions:**
  - **HPA (Pod Autoscaling):** Scales the number of pod instances based on resource utilization metrics (e.g., CPU target percentage) [7]. Managed via `kubectl` [7].
  - **Node Autoscaling / Resizing:** Scales the underlying Compute Engine VM instances in a cluster. Managed via `gcloud` [7].
- **Exam Keywords to Watch For:**
  - *"Automatically increase or decrease pod replicas based on CPU utilization"* $\rightarrow$ **Horizontal Pod Autoscaler (HPA) / `kubectl autoscale`** [7].
  - *"Manually change the number of worker nodes in a GKE Standard node pool"* $\rightarrow$ **`gcloud container clusters resize`** [7].
  - *"Enable node autoscaling on a GKE Standard cluster"* $\rightarrow$ **`gcloud container clusters update`** [7].

---

### 3. Detailed Concept Explanation

#### 1. Horizontal Pod Autoscaling (HPA)
- **Purpose:** Automatically scales the number of pod replicas up or down based on observed resource utilization (such as average CPU utilization) [7].
- **Command Syntax:**
  ```bash
  kubectl autoscale deployment <deployment-name> \
      --max=<max-replicas> \
      --cpu-percent=<target-percentage>
  ```
  *Example from transcript:* 
  ```bash
  kubectl autoscale deployment hello-world-rest-api --max=4 --cpu-percent=70
  ```
  This command creates a Horizontal Pod Autoscaler ensuring that if CPU utilization exceeds 70%, pods scale up automatically up to a maximum of 4 replicas [7].
- **Inspection Command:**
  ```bash
  kubectl get hpa
  ```
  *Details Provided:* Displays the target resource utilization, current CPU utilization percentage, minimum pods, maximum pods, and current active replica counts [7].

#### 2. Pod Scaling vs. Node Scaling
- **Pod Scaling (Microservice Level):** Focuses entirely on how many copies of your containerized application are running [7]. Controlled using `kubectl` [7].
- **Node Scaling (Infrastructure Level):** Focuses on the underlying virtual machines (nodes) that house those pods [7]. 
  - In **GKE Autopilot**, node scaling is entirely automated by Google [7].
  - In **GKE Standard**, if you scale up your pod deployments without available underlying node capacity, pods will remain in a pending state. Therefore, administrators must scale the worker nodes manually or via autoscaling [7].

#### 3. Managing GKE Standard Worker Nodes via `gcloud`
- **Manual Node Resizing:**
  ```bash
  gcloud container clusters resize <cluster-name> \
      --node-pool=<node-pool-name> \
      --num-nodes=5 \
      --zone=<zone>
  ```
  (Used to manually scale a specific node pool to a set number of instances, e.g., 5 nodes) [7].
- **Enabling Node Autoscaling:**
  ```bash
  gcloud container clusters update <cluster-name> \
      --enable-autoscaling \
      --min-nodes=1 \
      --max-nodes=10 \
      --zone=<zone>
  ```
  (Configures the cluster to automatically add or remove worker nodes dynamically between 1 and 10 nodes based on workload demand) [7].

---

### 4. Practical Setup & Implementation Guide

#### Step 1: Set Up Horizontal Pod Autoscaling (HPA)
1. Open Cloud Shell [4].
2. Configure autoscaling for the existing deployment [7]:
   ```bash
   kubectl autoscale deployment hello-world-rest-api \
       --max=4 \
       --cpu-percent=70
   ```
   *Expected Outcome:* `horizontalpodautoscaler.autoscaling/hello-world-rest-api autoscaled` [7].

#### Step 2: Inspect HPA Status
1. Run the inspection command [7]:
   ```bash
   kubectl get hpa
   ```
   *Expected Outcome:* Displays a table showing the target metric (70% CPU), current utilization (0%), minimum pods, maximum pods (4), and active replica counts [7].

#### Step 3: Managing GKE Standard Nodes (Reference Implementation)
*Note: The following commands apply to GKE Standard clusters where infrastructure management is the user's responsibility [7].*
1. **Manual Resize Command:**
   ```bash
   gcloud container clusters resize my-standard-cluster \
       --node-pool=default-pool \
       --num-nodes=5 \
       --zone=us-central1-a
   ```
2. **Enable Node Autoscaling Command:**
   ```bash
   gcloud container clusters update my-standard-cluster \
       --enable-autoscaling \
       --min-nodes=1 \
       --max-nodes=10 \
       --zone=us-central1-a
   ```

---

### 5. Visual Representations

#### Scaling Hierarchy: Pods vs. Nodes
```text
[ GKE Cluster ]
  |
  +---> Pod Level (Microservice Apps)
  |       - Managed via: kubectl scale / kubectl autoscale (HPA) [7]
  |       - Scales application replicas based on CPU/Memory metrics [7].
  |
  +---> Node Level (Underlying VMs - GKE Standard Mode Only)
          - Managed via: gcloud container clusters resize / update [7]
          - Scales physical/virtual worker machines to supply cluster capacity [7].
```

---

### 6. Mental Models & Analogies

- **Pod Autoscaling vs. Node Autoscaling:** 
  - **Pod Autoscaling (HPA)** is like hiring more individual cashiers at a grocery store when checkout lines get long (increasing application workers).
  - **Node Autoscaling** is like building a larger store building with more checkout lanes and parking spaces when the overall property runs out of floor space to hold the cashiers and customers.

---

### 7. Step-by-Step Workflows / Comparison Tables

#### Pod vs. Node Scaling Comparison Table

| Feature / Dimension | Horizontal Pod Autoscaler (HPA) | GKE Standard Node Autoscaling / Resizing |
| :--- | :--- | :--- |
| **Target Object** | Application Pod Replicas [7] | Underlying Compute Engine Worker Nodes [7] |
| **Primary Metric** | CPU / Memory Utilization Percentage [7] | Cluster Node Resource Pressure / Pending Pods |
| **Management Tool** | `kubectl` [7] | `gcloud` [7] |
| **Autopilot Behavior** | Fully supported | Automated entirely by Google (No manual node scaling needed) [7] |

---

### 8. Common Mistakes & Exam Traps

- **Trap 1:** Trying to manage GKE Standard worker nodes using `kubectl` commands.
  - *Correction:* `kubectl` is restricted to Kubernetes cluster objects (pods, deployments, services, HPA). Infrastructure node management (`resize`, `update`) requires the `gcloud` CLI [7].
- **Trap 2:** Assuming node scaling is required when deploying workloads to a GKE Autopilot cluster.
  - *Correction:* GKE Autopilot manages underlying node capacity automatically based on pod resource requests; manual or automated node scaling via `gcloud` only applies to GKE Standard mode [7].

---

### 9. Certification Practice Questions

#### Q1: An administrator wants to ensure that a GKE deployment automatically increases its pod replica count when average CPU utilization exceeds 70%, up to a maximum of 4 replicas. Which command should they run?
- A. `gcloud container clusters resize my-cluster --num-nodes=4`
- B. `kubectl autoscale deployment hello-world-rest-api --max=4 --cpu-percent=70` [7]
- C. `kubectl scale deployment hello-world-rest-api --replicas=4`
- D. `gcloud container clusters update my-cluster --enable-autoscaling`
- **Correct Answer:** B
- **Explanation:** `kubectl autoscale deployment` creates a Horizontal Pod Autoscaler (HPA) that adjusts pod counts dynamically based on CPU utilization thresholds [7].
- **Incorrect Options:** A manages nodes via gcloud. C performs static manual scaling, not autoscaling. D enables cluster node autoscaling, not pod autoscaling.
- **Exam Takeaway:** Use `kubectl autoscale` to configure Horizontal Pod Autoscaling based on metrics like CPU utilization [7].

#### Q2: Which command is used to inspect the active configuration, target metrics, and current utilization of a Horizontal Pod Autoscaler in a Kubernetes cluster?
- A. `kubectl get pods`
- B. `kubectl get hpa` [7]
- C. `gcloud container clusters list`
- D. `kubectl describe nodes`
- **Correct Answer:** B
- **Explanation:** `kubectl get hpa` displays Horizontal Pod Autoscaler details, including target CPU percentages, current utilization, and replica bounds [7].
- **Incorrect Options:** A lists pods. C lists gcloud clusters. D describes node hardware.
- **Exam Takeaway:** Use `kubectl get hpa` to review autoscaler status [7].

#### Q3: You have deployed workloads onto a GKE Standard cluster and notice that newly scheduled pods remain in a pending state due to insufficient cluster compute capacity. What tool and command must you use to manually scale the number of worker nodes in the cluster?
- A. `kubectl scale nodes --replicas=5`
- B. `gcloud container clusters resize` [7]
- C. `kubectl autoscale cluster`
- D. `gcloud container clusters update-nodes`
- **Correct Answer:** B
- **Explanation:** Managing worker nodes in GKE Standard mode requires using the `gcloud` CLI, specifically `gcloud container clusters resize` [7].
- **Incorrect Options:** A is invalid syntax. C is incorrect. D uses an inaccurate command name.
- **Exam Takeaway:** Use `gcloud container clusters resize` to manually scale worker nodes in GKE Standard [7].

#### Q4: When managing a GKE cluster in **Autopilot mode**, how must an administrator scale the underlying worker node infrastructure?
- A. By running `gcloud container clusters resize` manually every week.
- B. Node scaling is fully managed by Google Cloud automatically; no manual node intervention is required [7].
- C. By editing the cluster node YAML file using `kubectl edit nodes`.
- D. By purchasing dedicated sole-tenant hardware instances via the billing console.
- **Correct Answer:** B
- **Explanation:** In GKE Autopilot, Google automatically handles node provisioning, scaling, and maintenance, requiring zero node-level management from the user [7].
- **Incorrect Options:** A, C, and D describe manual tasks or incorrect assumptions associated with Standard mode or raw infrastructure.
- **Exam Takeaway:** GKE Autopilot completely automates underlying node infrastructure management [7].

#### Q5: Which command enables node autoscaling on a GKE Standard cluster with a specified minimum and maximum node range?
- A. `kubectl autoscale cluster --min=1 --max=10`
- B. `gcloud container clusters update --enable-autoscaling --min-nodes=1 --max-nodes=10` [7]
- C. `gcloud container clusters resize --auto-scale`
- D. `kubectl update deployment --autoscaling=true`
- **Correct Answer:** B
- **Explanation:** Node autoscaling for GKE Standard clusters is configured using the `gcloud container clusters update` command with autoscaling flags [7].
- **Incorrect Options:** A uses incorrect `kubectl` syntax for node configuration. C uses an invalid resize flag. D targets deployments, not cluster nodes.
- **Exam Takeaway:** Use `gcloud container clusters update` with autoscaling flags to configure node autoscaling on Standard clusters [7].

#### Q6: What primary metric is configured in the example HPA command (`kubectl autoscale deployment hello-world-rest-api --max=4 --cpu-percent=70`) to drive scaling decisions?
- A. Available hard disk space
- B. Average CPU utilization percentage [7]
- C. Inbound network packet latency
- D. Total active user sessions
- **Correct Answer:** B
- **Explanation:** The `--cpu-percent` flag sets CPU utilization as the primary metric for triggering horizontal pod scaling [7].
- **Incorrect Options:** A, C, and D are not the metric specified by that flag.
- **Exam Takeaway:** CPU utilization percentage drives the default HPA metric configuration shown [7].

#### Q7: True or False: If you are running your workloads on a GKE Standard cluster, you must ensure sufficient worker node capacity exists before scaling your application pod deployments.
- A. True [7]
- B. False
- **Correct Answer:** A
- **Explanation:** In GKE Standard mode, pod scheduling depends on available node capacity; if nodes are fully utilized, new pods will remain pending until nodes are scaled up [7].
- **Exam Takeaway:** GKE Standard worker nodes must be scaled to provide capacity for growing application deployments [7].

#### Q8: Which tool (`gcloud` or `kubectl`) is required to interact with and modify cluster-level worker node pools?
- A. `kubectl`
- B. `gcloud` [7]
- C. `docker`
- D. `bq`
- **Correct Answer:** B
- **Explanation:** Infrastructure-level resources like GKE clusters and node pools are managed using the `gcloud` CLI [7].
- **Incorrect Options:** A manages internal Kubernetes objects. C builds images. D queries BigQuery.
- **Exam Takeaway:** Use `gcloud` for GKE cluster and node infrastructure management [7].

#### Q9: What happens to pod replica counts when CPU utilization drops significantly below the configured HPA threshold percentage?
- A. The HPA controller automatically terminates excess pod replicas to optimize resource usage [7].
- B. The cluster shuts down entirely.
- C. Pods are migrated to a different GCP region.
- D. The deployment locks its current scale permanently.
- **Correct Answer:** A
- **Explanation:** When resource utilization falls below target thresholds, horizontal pod autoscalers scale down active replica instances [7].
- **Incorrect Options:** B, C, and D are false.
- **Exam Takeaway:** HPA scales down pod instances when load decreases below target thresholds [7].

#### Q10: Why does the transcript emphasize that node scaling is unnecessary to worry about when using GKE Autopilot mode?
- A. Because Autopilot runs containers without any underlying nodes.
- B. Because GKE Autopilot automatically manages and provisions the required node infrastructure based on pod demands [7].
- C. Because Autopilot clusters are restricted to exactly one single node.
- D. Because GKE Standard is deprecated and no longer supported.
- **Correct Answer:** B
- **Explanation:** Autopilot provides a serverless Kubernetes experience where Google automatically handles node scaling behind the scenes [7].
- **Incorrect Options:** A, C, and D contain factually incorrect statements.
- **Exam Takeaway:** Autopilot eliminates node scaling responsibilities by handling infrastructure automatically [7].