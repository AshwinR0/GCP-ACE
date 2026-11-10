# Google Kubernetes Engine (GKE) - Core Kubernetes Concepts: Deployments, Pods, and Services Study Notes

---

### 1. Overview
This module provides an in-depth review of three fundamental, tightly integrated Kubernetes logical concepts: **Deployments**, **Pods**, and **Services** [6]. Understanding how these components interact is vital for deploying, scaling, and exposing containerized applications on GKE. A **Deployment** manages application lifecycles and scaling rules [6], **Pods** are the smallest deployable execution units that house your containers [6], and **Services** provide internal stability or external network routing (load balancing) to reach those pods [6].

---

### 2. ACE Exam Focus & Common Exam Traps
- **Exam Importance:** **HIGH**
- **Core Concepts:**
  - The relationship between Deployments and Pod replicas (`--replicas`) [6].
  - The internal nature of Pods (ephemeral and isolated behind cluster networks) [6].
  - The role of a Service (specifically type `LoadBalancer`) in exposing deployments externally [6].
- **Key Distinctions:**
  - **Pods vs. Deployments:** Deployments are high-level management objects that define the desired state (e.g., container image, replica count), whereas Pods are the actual running instances spawned to satisfy that state [6].
  - **Pods vs. Services:** Pods handle compute execution and contain your app code; Services act as stable networking proxies/load balancers routing traffic to those pods [6].
- **Exam Keywords to Watch For:**
  - *"Specify the number of instances for a workload"* $\rightarrow$ Deployment `--replicas` parameter [6].
  - *"Expose internal pods to the outside world / external traffic"* $\rightarrow$ Kubernetes **Service** of type `LoadBalancer` [6].

---

### 3. Detailed Concept Explanation

#### 1. Deployments (`Deployment`)
- **Definition & Purpose:** A Deployment is a high-level Kubernetes resource object used to manage the desired state of your application workloads [6].
- **Configuration Details:** 
  - Specifies the container image representing your application (e.g., pulled from Docker Hub or Artifact Registry) [6].
  - Specifies the **replica count** using options like `--replicas` to dictate how many concurrent instances of the application should run simultaneously [6].
- **Example Usage:** 
  ```bash
  kubectl create deployment hello-world-rest-api \
      --image=in28min/hello-world-rest-api:0.0.1.RELEASE \
      --replicas=2
  ```
  This command instructs Kubernetes to maintain 2 active running instances of the specified container image [6].

#### 2. Pods (`Pod`)
- **Definition & Purpose:** A Pod is the smallest, core deployable unit in Kubernetes [6]. 
- **Relationship to Deployments:** When you create a deployment with multiple replicas, Kubernetes automatically creates matching independent **pod instances** [6]. If you specify 2 replicas, 2 pods are created [6].
- **Important Characteristics:** 
  - Pods are strictly **internal to the cluster** [6]. 
  - They receive dynamic, ephemeral internal IP addresses.
  - **Crucial Rule:** Nobody outside the cluster can access individual pods directly without an intermediary routing layer (a Service) [6].

#### 3. Services (`Service`)
- **Definition & Purpose:** A Kubernetes Service is an abstraction layer that defines a logical set of pods and a policy by which to access them [6].
- **External Exposure:** Because pods come and go and are isolated internally, a Service is required to expose them to the outside world [6].
- **Load Balancing:** When you create a service of type `LoadBalancer` (via `kubectl expose deployment`), Kubernetes provisions a cloud provider load balancer (e.g., a GCP external TCP/IP load balancer) [6]. 
- **Traffic Routing:** Incoming requests sent to the load balancer/service are automatically load-balanced and distributed across all active, healthy pod instances belonging to that deployment [6].

---

### 4. Practical Setup & Implementation Guide
*Note: This module reinforces core conceptual architecture rather than introducing new setup configurations. For implementation commands, refer to the previous GKE connection guides.*

#### Scaling a Deployment with Replicas (Additional Context)
To adjust the number of pod instances via `kubectl`:
```bash
# Scale an existing deployment to 3 replicas
kubectl scale deployment hello-world-rest-api --replicas=3
```
*Verification Command:*
```bash
kubectl get pods
```
*Expected Outcome:* Lists the active pod instances matching the requested replica count.

---

### 5. Visual Representations

#### Architecture Flow: Deployment, Pods, and Service Interaction
```text
[ External User / Web Browser ]
              |
              v (Sends request to External IP)
   +---------------------------------------+
   |   GCP Load Balancer (Service Layer)   |
   +---------------------------------------+
              |
              +---------------+---------------+ (Load balances traffic)
              |                               |
              v                               v
    +-------------------+           +-------------------+
    |    Pod Instance 1 |           |    Pod Instance 2 |
    |  (hello-world)    |           |  (hello-world)    |
    +-------------------+           +-------------------+
    ^                                         ^
    +-----------------------------------------+
                    Managed by
             [ Deployment Controller ]
```

---

### 6. Mental Models & Analogies

- **Deployments, Pods, and Services as a Restaurant Business:**
  - **Deployment** is the franchise rulebook and recipe book that states: *"We must always have 2 active line cooks preparing meals using Recipe X."*
  - **Pods** are the individual active line cooks working in the kitchen executing the recipes. If one cook goes on break or gets sick, the manager replaces them to match the rulebook count.
  - **Service (LoadBalancer)** is the front-of-house host/waiter standing at the entrance. Customers walk up to the host (Service external IP), and the host routes incoming customer orders evenly to whichever active line cook (Pod) is ready to take the next ticket.

---

### 7. Step-by-Step Workflows / Comparison Tables

#### Kubernetes Core Concept Comparison

| Concept | Role / Description | Scope / Accessibility | Key Command / Action |
| :--- | :--- | :--- | :--- |
| **Deployment** | Manages application state, image versions, and replica scaling [6]. | Cluster-wide control object. | `kubectl create deployment` [6] |
| **Pod** | Smallest compute unit hosting one or more containers [6]. | Internal to cluster network; isolated [6]. | Automatically created by deployments [6]. |
| **Service** | Exposes pods externally and distributes incoming traffic [6]. | Internal (`ClusterIP`) or External (`LoadBalancer`) [6]. | `kubectl expose deployment` [6] |

---

### 8. Common Mistakes & Exam Traps

- **Trap 1:** Assuming clients on the public internet can directly query individual pod IP addresses inside a GKE cluster.
  - *Correction:* Pods are strictly internal to the cluster network. External traffic must pass through a Kubernetes **Service** (such as a `LoadBalancer`) to reach target pods [6].
- **Trap 2:** Confusing the role of a Deployment with a Pod.
  - *Correction:* A Pod is the actual running container instance, whereas a Deployment is the management blueprint that dictates how many pod replicas should exist and which container image they should run [6].

---

### 9. Certification Practice Questions

#### Q1: An administrator wants to deploy an application on GKE and ensure that exactly 3 instances of the container run concurrently. Where is this instance count specified?
- A. Inside the Kubernetes Service definition using the `--ports` flag
- B. Within the Deployment configuration using the replica specification (e.g., `--replicas=3`) [6]
- C. In the global Google Cloud project metadata
- D. Automatically handled by the cluster node BIOS firmware
- **Correct Answer:** B
- **Explanation:** The number of running application instances (replicas) is specified when creating or scaling a deployment [6].
- **Incorrect Options:** A handles network ports, not replica counts. C and D are completely unrelated to container instance sizing.
- **Exam Takeaway:** Deployment configurations control the number of running pod instances via replica parameters [6].

#### Q2: Why are Kubernetes Pods considered insufficient on their own for serving public web traffic coming from the internet?
- A. Pods can only run Windows-based container images.
- B. Pods are internal to the cluster network, and clients outside the cluster cannot access them directly without a service [6].
- C. Pods do not support CPU resource limits.
- D. Pods are permanent infrastructure objects that cannot be scaled up or down.
- **Correct Answer:** B
- **Explanation:** Pods are internal components of a cluster and lack native public routing, requiring a Kubernetes Service to expose them to external traffic [6].
- **Incorrect Options:** A, C, and D contain incorrect statements about pod capabilities and limitations.
- **Exam Takeaway:** Pods are internal-only entities and require a Service for external exposure [6].

#### Q3: When a Kubernetes Service of type `LoadBalancer` is created to expose a deployment with multiple pod replicas, how does traffic behave when external requests arrive?
- A. All incoming traffic is routed exclusively to the first pod created, ignoring the others.
- B. The load balancer automatically distributes incoming requests among the active pods in the deployment [6].
- C. The service rejects requests unless every pod has identical internal IP addresses.
- D. Traffic is blocked until a manual proxy script is executed on the control plane.
- **Correct Answer:** B
- **Explanation:** A service acting as a load balancer distributes incoming requests across the active pod instances belonging to the target deployment [6].
- **Incorrect Options:** A, C, and D misrepresent the load-balancing and traffic-distribution behavior of Kubernetes services.
- **Exam Takeaway:** Services distribute incoming traffic across active pods [6].

#### Q4: Which Kubernetes object represents the smallest deployable unit of computing that can be created and managed in Kubernetes?
- A. Deployment
- B. Service
- C. Pod [6]
- D. Node Pool
- **Correct Answer:** C
- **Explanation:** Pods are the fundamental and smallest deployable computing units in the Kubernetes object model [6].
- **Incorrect Options:** A and B are higher-level management and networking abstractions. D represents physical/virtual worker machines.
- **Exam Takeaway:** Pods are the smallest deployable units in Kubernetes [6].

#### Q5: A user executes `kubectl expose deployment hello-world-rest-api --type=LoadBalancer --port=8080`. What underlying cloud component is typically provisioned as a result of this command in GKE?
- A. A Cloud VPN tunnel
- B. An external cloud load balancer assigned an external public IP address [6]
- C. A Virtual Private Cloud (VPC) firewall blocking all ports
- D. A Cloud Storage bucket for log collection
- **Correct Answer:** B
- **Explanation:** Exposing a deployment as a `LoadBalancer` provisions an external cloud load balancer equipped with a public IP to route traffic into the cluster [6].
- **Incorrect Options:** A, C, and D are unrelated network and storage resources.
- **Exam Takeaway:** A `LoadBalancer` service provisions an external cloud load balancer [6].

#### Q6: What is the primary function of a Kubernetes Deployment object?
- A. To manage the desired state, image versions, and scaling of application instances (pods) [6]
- B. To provide physical server rack cooling in Google datacenters
- C. To allocate static public IP addresses to individual container files
- D. To encrypt hard drives attached to worker nodes
- **Correct Answer:** A
- **Explanation:** Deployments provide declarative updates and management for pods, ensuring the specified image and replica counts are maintained [6].
- **Incorrect Options:** B, C, and D describe infrastructure hardware, IP assignment, and storage encryption tasks outside deployment scope.
- **Exam Takeaway:** Deployments manage application state, container images, and replica scaling [6].

#### Q7: If a deployment has 2 active replicas running, how many corresponding Pods exist in the cluster for that deployment?
- A. 0 pods
- B. 1 pod shared by both replicas
- C. Exactly 2 pods [6]
- D. An infinite number of auto-multiplying pods
- **Correct Answer:** C
- **Explanation:** Each replica specified in a deployment corresponds to a running pod instance; 2 replicas equal 2 pods [6].
- **Incorrect Options:** A, B, and D violate the replica-to-pod mapping rule.
- **Exam Takeaway:** The number of replica instances matches the number of pods created [6].

#### Q8: True or False: Pod IP addresses are ephemeral, meaning they can change when pods are rescheduled, restarted, or scaled.
- A. True
- B. False
- **Correct Answer:** A
- **Explanation:** Pod IPs are dynamic and ephemeral; services provide a stable networking endpoint masking underlying pod IP volatility.
- **Exam Takeaway:** Pod IP addresses are ephemeral, making stable Services essential for reliable communication.

#### Q9: Which command-line tool is used to interact with Kubernetes clusters to create deployments and expose services?
- A. `gcloud`
- B. `kubectl` [6]
- C. `gsutil`
- D. `docker`
- **Correct Answer:** B
- **Explanation:** `kubectl` is the primary command-line tool for executing commands against Kubernetes clusters [6].
- **Incorrect Options:** A manages GCP infrastructure. C manages Cloud Storage. D builds container images locally.
- **Exam Takeaway:** Use `kubectl` for Kubernetes resource management operations [6].

#### Q10: What happens if all pod replicas associated with a deployment crash or fail simultaneously?
- A. The deployment controller detects the failure and automatically attempts to recreate new pods to restore the desired replica state.
- B. The deployment object is permanently deleted from the cluster.
- C. The cluster automatically downgrades to GKE Standard mode.
- D. External load balancers permanently route traffic to error static pages without recovery.
- **Correct Answer:** A
- **Explanation:** Deployments maintain self-healing capabilities; if pods fail, the deployment controller automatically provisions new pods to match the target replica count.
- **Incorrect Options:** B, C, and D are false descriptions of Kubernetes self-healing mechanics.
- **Exam Takeaway:** Deployments provide self-healing by automatically replacing failed pods to match desired replica states.