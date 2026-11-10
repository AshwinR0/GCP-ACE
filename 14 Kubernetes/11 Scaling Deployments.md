# Google Kubernetes Engine (GKE) - Scaling, Pods, Load Balancing, and Self-Healing Study Notes

---

### 1. Overview
This module explores core Kubernetes operational behaviors in GKE, focusing on scaling deployments, inspecting pod instances, load balancing via services, and testing Kubernetes' built-in **self-healing** capabilities [4]. By scaling the `hello-world-rest-api` deployment from 1 to 2 replicas, this module demonstrates how Kubernetes automatically provisions new pods, how services distribute incoming traffic across active replicas, and how controllers replace deleted or unhealthy pods to maintain the desired system state [4].

---

### 2. ACE Exam Focus & Common Exam Traps
- **Exam Importance:** **HIGH**
- **Core Concepts:**
  - Scaling a deployment using `--replicas` [4].
  - Inspecting workloads and pods via `kubectl get pods` and `kubectl get deployment` [4].
  - Kubernetes self-healing: automatic recreation of pods when deleted or terminated [4].
  - Automatic load balancing across multiple pod replicas managed by a single Service [4].
- **Key Distinctions:**
  - **Manual Scaling vs. Autoscaling:** Manual scaling uses explicit commands (`kubectl scale`), whereas autoscaling dynamically adjusts replicas based on metric thresholds (like CPU utilization) [4].
  - **Pod Identifiers vs. Services:** Pods have ephemeral names/IDs, while Services provide stable front-end endpoints that automatically load-balance traffic to active pods [4].
- **Exam Keywords to Watch For:**
  - *"Automatically replace a failed or deleted pod to match the desired state"* $\rightarrow$ **Self-healing / Deployment Controller** [4].
  - *"Distribute traffic across multiple pod instances automatically"* $\rightarrow$ Kubernetes **Service** load balancing [4].

---

### 3. Detailed Concept Explanation

#### 1. Scaling Deployments
- **Command:** `kubectl scale deployment <deployment-name> --replicas=<count>` [4]
- **Behavioral Note:** While the initial pod in a deployment spins up quickly, scaling up to additional replicas can occasionally take time (sometimes several minutes depending on node provisioning) [4]. Always verify status using `kubectl get deployment` and check if `READY` counts match desired instances [4].

#### 2. Pod Inspection (`kubectl get pods`)
- Pods are the smallest deployable units [6]. Each pod receives a unique, random string identifier appended to its name (e.g., `hello-world-rest-api-xxxx-yyyy`) [4].
- In the sample application, the app extracts the last characters of its pod name and returns them in HTTP responses, making it easy to observe load balancing in action [4].

#### 3. Automatic Load Balancing via Services
- When a Service of type `LoadBalancer` is mapped to a deployment with multiple replicas, incoming requests to the external IP and port are automatically distributed across all healthy, active pods [4]. Successive `curl` requests will hit different pod IDs interchangeably [4].

#### 4. Kubernetes Self-Healing
- **Mechanism:** Kubernetes operates on a **reconciliation loop** (desired state vs. current state). 
- If a pod is manually deleted using `kubectl delete pod <pod-name>`, the deployment controller notices that the active pod count has dropped below the desired replica count (e.g., 1 out of 2) [4].
- **Result:** Kubernetes automatically provisions a brand-new replacement pod with a unique identifier to restore the desired state [4].

---

### 4. Practical Setup & Implementation Guide

#### Step 1: Scale the Deployment to 2 Replicas
1. Open Cloud Shell [4].
2. Execute the scale command [4]:
   ```bash
   kubectl scale deployment hello-world-rest-api --replicas=2
   ```
   *Expected Outcome:* `deployment.apps/hello-world-rest-api scaled` [4].

#### Step 2: Verify Deployment and Pod Status
1. Check deployment readiness [4]:
   ```bash
   kubectl get deployment
   ```
   *(Note: Look at the `READY` column. If it shows `1/2`, wait a few moments and re-run until it displays `2/2`)* [4].
2. List individual pods [4]:
   ```bash
   kubectl get pods
   ```
   *Expected Outcome:* Two distinct running pods with unique suffixes [4].

#### Step 3: Test Load Balancing via `curl`
1. Retrieve the external IP of your service [4]:
   ```bash
   kubectl get services
   ```
2. Run repeated `curl` requests to the external IP and port 8080 [4]:
   ```bash
   curl http://<EXTERNAL-IP>:8080/hello-world
   ```
   *Expected Outcome:* Responses alternate between different pod identifiers (e.g., displaying different random suffixes), confirming automatic load balancing across the two active pods [4].

#### Step 4: Test Self-Healing by Deleting a Pod
1. Copy the name of one running pod from `kubectl get pods`.
2. Delete that specific pod [4]:
   ```bash
   kubectl delete pod <pod-name>
   ```
3. Immediately inspect the pods again [4]:
   ```bash
   kubectl get pods
   ```
   *Expected Outcome:* The deleted pod disappears, but a new pod with a fresh creation timestamp and unique identifier automatically spawns to maintain the desired replica count of 2 [4].

---

### 5. Visual Representations

#### Self-Healing and Load Balancing Lifecycle Flow
```text
[ Client Requests ] ---> (External IP:8080) ---> [ Kubernetes Service (LoadBalancer) ]
                                                            |
                        +-----------------------------------+-----------------------------------+
                        |                                                                       |
                        v                                                                       v
         [ Pod A (ID: 8h6f5) - Active ]                                          [ Pod B (ID: pcx16) - Active ]
                        |                                                                       |
      (Manual Deletion: kubectl delete pod)                                                             |
                        v                                                                       |
         [ Pod A Terminated ]                                                                   |
                        |                                                                       |
         [ Deployment Controller Detects Drift ]                                                |
                        |                                                                       |
                        v                                                                       |
         [ Pod C (ID: fresh-new-id) Automatically Provisioned ]                                 |
```

---

### 6. Mental Models & Analogies

- **Self-Healing in Kubernetes is like a restaurant kitchen staffing policy:** The general manager (Deployment controller) has a rule: *"We must always have exactly 2 chefs working the grill."* If a chef cuts their hand and leaves early (Pod deleted/crashes), the manager immediately hires and places a new chef on the line to keep the total count at 2, without human intervention from upper management.

---

### 7. Step-by-Step Workflows / Comparison Tables

#### Kubernetes Scaling & Verification Workflow

| Step | Action / Command | Expected Behavior / Purpose |
| :--- | :--- | :--- |
| **1. Scale** | `kubectl scale deployment <name> --replicas=2` [4] | Increases desired pod replica count from 1 to 2 [4]. |
| **2. Verify Deployment** | `kubectl get deployment` [4] | Confirms readiness ratio (e.g., `2/2` ready) [4]. |
| **3. Verify Pods** | `kubectl get pods` [4] | Lists individual running pod instances and unique IDs [4]. |
| **4. Test Routing** | `curl http://<EXTERNAL-IP>:8080/hello-world` [4] | Demonstrates automatic round-robin style load balancing across pods [4]. |
| **5. Test Self-Healing** | `kubectl delete pod <pod-name>` [4] | Forces pod termination to watch Kubernetes automatically recreate a replacement [4]. |

---

### 8. Common Mistakes & Exam Traps

- **Trap 1:** Assuming scaling a deployment happens instantly across all environments.
  - *Correction:* While scaling is usually fast, node provisioning or image pulling can cause delays. Always check readiness using `kubectl get deployment` or `kubectl get pods` rather than assuming immediate availability [4].
- **Trap 2:** Believing that deleting a pod permanently reduces the replica count in a managed deployment.
  - *Correction:* Because deployments enforce a desired state, deleting a pod triggers self-healing, causing Kubernetes to immediately spin up a replacement pod [4].

---

### 9. Certification Practice Questions

#### Q1: An administrator scales a GKE deployment from 1 replica to 3 replicas using `kubectl scale deployment my-app --replicas=3`. Shortly after, one of the pods crashes due to an unhandled exception. What action does Kubernetes take?
- A. The deployment permanently scales down to 2 replicas.
- B. The deployment controller detects the missing pod and automatically creates a replacement pod to restore the desired count of 3 [4].
- C. The entire GKE cluster shuts down to prevent data corruption.
- D. The administrator must manually execute `kubectl restart` to fix the issue.
- **Correct Answer:** B
- **Explanation:** Kubernetes self-healing reconciles actual state with desired state, automatically replacing crashed or deleted pods [4].
- **Incorrect Options:** A, C, and D violate core Kubernetes self-healing and controller reconciliation behavior.
- **Exam Takeaway:** Kubernetes automatically replaces failed or deleted pods to match desired replica counts [4].

#### Q2: You have exposed a GKE deployment with 2 replicas using a Kubernetes Service of type `LoadBalancer`. When sending multiple sequential `curl` requests to the external IP, what behavior do you observe regarding response handling?
- A. All requests are routed exclusively to the pod with the oldest creation timestamp.
- B. Requests fail with a 503 error because multiple pods conflict on the same port.
- C. Incoming requests are automatically load-balanced across both active pod instances [4].
- D. Requests are redirected outside the cluster to Google Cloud Storage.
- **Correct Answer:** C
- **Explanation:** A Kubernetes Service automatically load-balances incoming traffic across all healthy active pods belonging to the target deployment [4].
- **Incorrect Options:** A, B, and D misrepresent standard Service load-balancing mechanics.
- **Exam Takeaway:** Services distribute traffic across all active pod replicas [4].

#### Q3: Which command is used to check the current status and readiness of pods running inside a Kubernetes cluster?
- A. `gcloud container instances list`
- B. `kubectl get pods` [4]
- C. `kubectl describe cluster-health`
- D. `gcloud compute status`
- **Correct Answer:** B
- **Explanation:** `kubectl get pods` lists all active pods, their statuses, restart counts, and unique identifiers [4].
- **Incorrect Options:** A, C, and D are invalid or incorrect command structures for checking pod states.
- **Exam Takeaway:** Use `kubectl get pods` to review running pod statuses [4].

#### Q4: What happens if an administrator manually deletes a specific running pod using `kubectl delete pod <pod-name>` within a deployment configured with 2 replicas?
- A. The pod is deleted, and a new replacement pod is automatically created to maintain the desired count of 2 [4].
- B. The entire deployment is deleted from the cluster.
- C. The deployment scale permanently drops to 1 replica until manually scaled back up.
- D. The cluster locks up and rejects future `kubectl` commands.
- **Correct Answer:** A
- **Explanation:** Deleting a pod triggers reconciliation; the deployment controller detects the drop in active replicas and spawns a new pod [4].
- **Incorrect Options:** B, C, and D are incorrect descriptions of self-healing behavior.
- **Exam Takeaway:** Manually deleting pods in a deployment results in automatic recreation via self-healing [4].

#### Q5: When scaling a deployment using `kubectl scale`, which flag specifies the target number of running instances?
- A. `--instances`
- B. `--replicas` [4]
- C. `--count`
- D. `--scale-factor`
- **Correct Answer:** B
- **Explanation:** The `--replicas=<number>` flag specifies the desired number of pod instances for a deployment [4].
- **Incorrect Options:** A, C, and/or D are incorrect flag names for `kubectl scale`.
- **Exam Takeaway:** Use `--replicas` to scale deployments [4].

#### Q6: Why might scaling a deployment from 1 replica to 2 replicas take a few minutes before both pods report a `READY` status?
- A. Kubernetes requires manual administrator approval for every additional replica.
- B. Additional worker node capacity or container image pulling may take time to provision and initialize [4].
- C. Google Cloud limits scaling operations to exactly one pod per hour.
- D. Services block traffic until explicit firewall rules are added.
- **Correct Answer:** B
- **Explanation:** Initialization delays during scaling can occur due to node autoscaling, container image downloads, or application start-up duration [4].
- **Incorrect Options:** A, C, and D are false.
- **Exam Takeaway:** Scaling delays are typically caused by infrastructure provisioning or container initialization time [4].

#### Q7: True or False: Individual pods have unique identifiers appended to their names to distinguish them within a deployment.
- A. True [4]
- B. False
- **Correct Answer:** A
- **Explanation:** Kubernetes assigns unique random alphanumeric suffixes to pod names generated under a deployment controller [4].
- **Exam Takeaway:** Pod names include unique generated identifiers [4].

#### Q8: What core architectural principle enables Kubernetes to automatically recover from pod crashes or deletions without manual operator intervention?
- A. Immutable infrastructure locking
- B. The reconciliation control loop (Desired state vs. Actual state) [4]
- C. Static load balancer routing tables
- D. Manual snapshot recovery
- **Correct Answer:** B
- **Explanation:** Kubernetes controllers continuously monitor cluster state and reconcile differences between desired specifications and current runtime state (self-healing) [4].
- **Incorrect Options:** A, C, and D do not explain automated recovery mechanics.
- **Exam Takeaway:** Controller reconciliation drives Kubernetes self-healing [4].

#### Q9: If a custom application returns its unique pod identifier in API responses, what benefit does this provide during testing?
- A. It allows you to verify that incoming requests are successfully load-balanced across multiple pods [4].
- B. It encrypts database passwords inside the container memory.
- C. It automatically scales worker nodes up and down.
- D. It deletes old log files from Cloud Storage.
- **Correct Answer:** A
- **Explanation:** Returning pod IDs allows developers to verify that a LoadBalancer service is distributing traffic across multiple active replicas [4].
- **Incorrect Options:** B, C, and D are unrelated to application-level response tagging.
- **Exam Takeaway:** Tagging responses with pod IDs helps verify load balancing behavior [4].

#### Q10: Which Kubernetes component abstracts away pod IP volatility and provides a stable network endpoint for client traffic?
- A. A Deployment
- B. A Service [4]
- C. A Persistent Volume
- D. A Node Pool
- **Correct Answer:** B
- **Explanation:** Services provide stable networking abstracts (IPs and DNS) that route traffic to dynamic, ephemeral pod endpoints [4].
- **Incorrect Options:** A manages replicas. C handles storage. D manages worker VMs.
- **Exam Takeaway:** Services provide stable networking in front of ephemeral pods.