# Horizontal Pod Autoscaler (HPA): Implementation, YAML Configuration & Metrics - Study Notes

---

### 1. Overview
This section builds upon the conceptual foundations of the Horizontal Pod Autoscaler (HPA) by demonstrating practical ways to create and configure it using both imperative (`kubectl`) and declarative (YAML manifest) approaches. It highlights how HPA dynamically adjusts pod replica counts based on metrics like CPU utilization, and emphasizes the crucial prerequisite of having the Kubernetes metrics server installed.

---

### 2. ACE Exam Focus & Common Exam Traps
* **Exam Importance:** **HIGH**
* **High-Priority Topics:**
  * Syntax and configuration parameters for HPA YAML manifests (`minReplicas`, `maxReplicas`, `targetAverageUtilization`).
  * Imperative command-line creation using `kubectl autoscale`.
  * Core prerequisites: The absolute necessity of the **metrics-server** add-on for HPA to successfully fetch resource utilization metrics.
* **Important Configurations & Defaults:**
  * Target targets CPU utilization (e.g., target percentage like 70%).
  * Scale target reference must match an existing scale-supporting resource type (such as a `Deployment` or `ReplicaSet`) and provide its name.
* **Exam Keywords:** *"kubectl autoscale"*, *"metrics server"*, *"minReplicas"*, *"maxReplicas"*, *"targetAverageUtilization"*.

---

### 3. Detailed Concept Explanation

#### A. Creating HPA via Imperative Commands
* **Command Syntax:** 
  ```bash
  kubectl autoscale deployment <DEPLOYMENT_NAME> --min=<MIN_PODS> --max=<MAX_PODS> --cpu-percent=<PERCENTAGE>
  ```
* **How it Works:** Quickly generates an HPA resource targeting an existing deployment without needing to author a complete YAML file from scratch.

#### B. Creating HPA via Declarative YAML Configuration
* **Structure & Components:**
  * `kind: HorizontalPodAutoscaler`
  * `scaleTargetRef`: Defines the target resource block (e.g., `kind: Deployment`, `name: hello-world-rest-api`).
  * `minReplicas` and `maxReplicas`: Establishes the safe boundary limits for scaling.
  * `metrics`: Specifies the metric type (Resource) and target specifications (e.g., CPU utilization at 70%).

#### C. Essential Prerequisite: Metrics Server
* **Purpose:** HPA cannot function without metric data. The **Metrics Server** collects resource metrics (CPU and memory) from Kubelets across the cluster and exposes them through the Kubernetes API server via the `metrics.k8s.io` API.
* **Exam Relevance:** If an HPA is configured but fails to scale or shows `<unknown>` for current CPU metrics, missing or misconfigured metrics server components are a prime troubleshooting suspect.

---

### 4. Practical Setup & Implementation Guide

#### Method 1: Creating HPA using Imperative Command (`kubectl`)
* **Prerequisites:** An active Kubernetes cluster with a deployed application and the metrics-server installed.
* **Command:**
  ```bash
  kubectl autoscale deployment hello-world-rest-api --min=2 --max=10 --cpu-percent=70
  ```
* **Expected Outcome:** A horizontal pod autoscaler object is created, targeting the `hello-world-rest-api` deployment, restricting replica bounds between 2 and 10, with a target CPU utilization of 70%.
* **Verification Command:**
  ```bash
  kubectl get hpa
  ```

#### Method 2: Creating HPA using a YAML Manifest
* **YAML Manifest (`hpa.yaml`):**
  ```yaml
  apiVersion: autoscaling/v2
  kind: HorizontalPodAutoscaler
  metadata:
    name: hello-world-rest-api-hpa
  spec:
    scaleTargetRef:
      apiVersion: apps/v1
      kind: Deployment
      name: hello-world-rest-api
    minReplicas: 2
    maxReplicas: 10
    metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
  ```
* **Step-by-Step Execution:**
  1. Save the configuration above into a file named `hpa.yaml`.
  2. Apply the configuration using `kubectl`:
     ```bash
     kubectl apply -f hpa.yaml
     ```
* **Expected Outcome:** The declarative HPA configuration is applied to the cluster.
* **Verification Command:**
  ```bash
  kubectl describe hpa hello-world-rest-api-hpa
  ```

---

### 5. Visual Representations

#### HPA Request and Metrics Flow Architecture
```text
 +--------------------+       Reads Metrics       +------------------+
 |                    | <------------------------ |                  |
 |  Horizontal Pod    |                           |  Metrics Server  |
 |  Autoscaler (HPA)  |                           |  (Add-on)        |
 |                    |                           |                  |
 +---------+----------+                           +--------^---------+
           |                                               |
           | Evaluates (e.g., > 70% CPU)                   | Collects
           v                                               | stats from
 +--------------------+                                    | Kubelets
 | Scale Decision     | -----------------------------------+
 | (Add/Remove Pods)  |
 +---------+----------+
           |
           v
 +--------------------+
 | Target Deployment  | ---> [Pod 1] [Pod 2] ... [Pod N]
 | (hello-world-...)  |
 +--------------------+
```

---

### 6. Mental Models & Analogies

* **The Thermostat Analogy:** An HPA configured with a CPU target (like 70%) acts just like an automated HVAC thermostat. If the temperature (CPU load) rises past the target threshold, the system triggers cooling elements (scales up pods) until the environment stabilizes. If the load drops, it scales back down.

---

### 7. Step-by-Step Workflows / Comparison Tables

#### HPA Creation Methods Comparison Table

| Parameter / Feature | Imperative (`kubectl autoscale`) | Declarative (`kubectl apply -f hpa.yaml`) |
| :--- | :--- | :--- |
| **Speed** | Extremely fast for quick adjustments | Requires file creation/management |
| **Version Control Friendliness** | Low (changes live only in cluster runtime) | High (manifests can be tracked in Git) |
| **Customization Depth** | Limited to flags available in the command | Highly customizable (supports multiple metrics, advanced scaling policies, behavior configs) |
| **Best Used For** | Quick testing, labs, or ad-hoc adjustments | Production environments, GitOps pipelines |

---

### 8. Common Mistakes & Exam Traps

* **Trap:** Forgetting that `metrics-server` is required for HPA. 
  * *Scenario:* An exam question states that an HPA was successfully deployed using YAML, but running `kubectl get hpa` shows `<unknown>` under the `TARGETS` column and the pods never scale.
  * *Correct Answer/Reasoning:* The metrics server add-on is missing or not running in the cluster, meaning HPA cannot read CPU/memory metrics.
* **Trap:** Targeting a non-existent deployment name in the HPA manifest scaleTargetRef.
  * *Correction:* Ensure the target resource kind and name precisely match an active deployment name in the cluster namespace.

---

### 9. Certification Practice Questions

#### Question 1
An administrator creates an HPA resource using a YAML file to scale a deployment named `inventory-api`. After deployment, running `kubectl get hpa` displays `<unknown>` for the current CPU utilization metric, and scaling does not occur. What is the most likely cause of this issue?
* A) The `maxReplicas` value is set lower than `minReplicas`.
* B) The metrics-server add-on is not installed or running in the cluster.
* C) The target deployment uses a RollingUpdate strategy instead of Recreate.
* D) HPA version 2 is not compatible with GKE Standard clusters.

#### Answer
**B**

#### Explanation
When HPA outputs `<unknown>` for metrics, it indicates that the metrics pipeline cannot fetch utilization values, typically because the `metrics-server` is missing or disabled in the cluster.

---

#### Question 2
Which imperative command correctly creates a Horizontal Pod Autoscaler for a deployment named `frontend-app` with a minimum of 2 pods, a maximum of 8 pods, and a target CPU utilization of 60%?
* A) `kubectl scale deployment frontend-app --min=2 --max=8 --cpu-percent=60`
* B) `kubectl autoscale deployment frontend-app --min=2 --max=8 --cpu-percent=60`
* C) `kubectl create hpa frontend-app --min-replicas=2 --max-replicas=8 --cpu=60%`
* D) `kubectl update deployment frontend-app --autoscale=true --target-cpu=60`

#### Answer
**B**

#### Explanation
The correct imperative command structure to automatically scale a deployment based on CPU percentage parameters is `kubectl autoscale deployment <name> --min=<num> --max=<num> --cpu-percent=<percentage>`.

---

#### Question 3
In a declarative HPA YAML configuration file, under which field path is the target resource (such as a Deployment) specified?
* A) `spec.scaleTargetRef`
* B) `metadata.targetDeployment`
* C) `spec.metrics.resource.target`
* D) `status.replicas`

#### Answer
**A**

#### Explanation
The `scaleTargetRef` block inside `spec` explicitly points HPA to the target resource type, API version, and name (e.g., `kind: Deployment`, `name: hello-world-rest-api`).

---

#### Question 4
What happens if an HPA is configured with `minReplicas: 3` and `maxReplicas: 10`, but the current load on the deployment drops to near 0% CPU utilization?
* A) The HPA scales the deployment down to 0 replicas.
* B) The HPA scales the deployment down to 3 replicas and stops there.
* C) The HPA deletes the deployment object entirely to save resources.
* D) The HPA returns an error because `minReplicas` cannot be respected during low load.

#### Answer
**B**

#### Explanation
The HPA respects the `minReplicas` boundary constraint. Even under zero utilization, it will scale down no further than the specified minimum of 3 replicas.

---

#### Question 5
Which API version block is standard and modern for writing an HPA configuration manifest in current Kubernetes versions?
* A) `autoscaling/v1`
* B) `autoscaling/v2`
* C) `apps/v1`
* D) `v1`

#### Answer
**B**

#### Explanation
`autoscaling/v2` is the standard API version supporting complex metrics configurations (like multi-metric scaling and behavior configurations), superseding older legacy versions.

---

#### Question 6
An engineer wants to inspect the detailed status events and scaling history of an HPA object named `payment-service-hpa`. Which command should they execute?
* A) `kubectl log hpa/payment-service-hpa`
* B) `kubectl describe hpa payment-service-hpa`
* C) `kubectl inspect hpa payment-service-hpa`
* D) `kubectl get deployment payment-service-hpa -o yaml`

#### Answer
**B**

#### Explanation
`kubectl describe` provides comprehensive details on resource configurations, current/target metrics values, and recent event history for autoscalers.

---

#### Question 7
True or False: A Horizontal Pod Autoscaler can scale down a deployment below the `minReplicas` limit if system-wide memory pressure is detected.
* A) True
* B) False

#### Answer
**B**

#### Explanation
False. `minReplicas` acts as a hard lower limit constraint for the deployment replicas managed by the HPA, regardless of system pressure.

---

#### Question 8
Which component of the Kubernetes control plane or cluster architecture directly provides the raw CPU metrics utilized by the HPA controller loop?
* A) Kube-DNS
* B) Metrics Server
* C) Persistent Volume Controller
* D) Ingress Controller

#### Answer
**B**

#### Explanation
The Metrics Server aggregates resource usage data from Kubelets and exposes it to the Kubernetes API, allowing HPA to read CPU and memory utilization metrics.

---

#### Question 9
If an HPA configuration sets a target average CPU utilization of 50%, and the current pods in the deployment have a combined average usage of 85%, how will the HPA controller respond?
* A) Scale in (reduce the number of pod replicas)
* B) Scale out (increase the number of pod replicas)
* C) Do nothing until utilization reaches 100%
* D) Automatically upgrade the node instance type

#### Answer
**B**

#### Explanation
Since current utilization (85%) exceeds the target threshold (50%), the HPA controller will scale out (add pod replicas) to bring average utilization back down toward the 50% target.

---

#### Question 10
Can a single Horizontal Pod Autoscaler target multiple independent deployments simultaneously?
* A) Yes, by listing them under a comma-separated string in `scaleTargetRef`.
* B) Yes, by utilizing wildcard patterns in the deployment name.
* C) No, a single HPA instance targets exactly one scalable resource (such as one Deployment or ReplicaSet).
* D) Yes, provided they share the same container image repository.

#### Answer
**C**

#### Explanation
An HPA resource is bound to a single scale target reference defined in its spec (`scaleTargetRef`). To autoscale multiple deployments, you must configure separate HPA objects for each deployment.