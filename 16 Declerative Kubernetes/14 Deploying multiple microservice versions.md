### 1. Overview
This transcript covers the practical demonstration of deploying multiple microservice versions (`v1` and `v2`) using a multi-object Kubernetes manifest (`deployment-v2.yaml`), and using **label selectors** to dynamically control and shift traffic routing. 

It demonstrates how cleaning up resources using declarative files (`kubectl delete -f`) works seamlessly, how GKE's external `LoadBalancer` distributes traffic across multiple versions when selectors match broadly, and how narrowing a selector's key-value pairs (e.g., adding `version: v1` or `version: v2`) instantly shifts traffic routing to a single targeted application version with zero downtime.

---

### 2. ACE Exam Focus & Common Exam Traps

*   **Exam Importance:** **HIGH**. The ACE exam heavily tests declarative file operations (`kubectl apply -f` and `kubectl delete -f`) and how altering a Service’s label selector dynamically changes its routing endpoints without modifying the underlying deployments.
*   **High-Priority Topics:**
    *   **Declarative Deletion:** Using `kubectl delete -f <filename>.yaml` to clean up all resources defined within that specific manifest file at once.
    *   **Incremental Application Updates:** Recognizing that `kubectl apply -f` intelligently detects which specific object definitions have changed (e.g., only updating the Service selector while leaving unchanged Deployments untouched).
    *   **Dynamic Label Routing:** Shifting traffic between versions live by updating the `spec.selector` block inside a Service definition.
*   **Common Exam Traps:**
    *   Thinking you need to delete and recreate pods or deployments to switch traffic versions. (You simply update the Service's label selector configuration and re-apply).
    *   Forgetting that changing a Service selector instantly cuts off or routes traffic to pods matching the new filter criteria.

---

### 3. Detailed Concept Explanation

#### A. Declarative Cleanup and Multi-Resource Management
*   **Bulk Deletion:** Instead of deleting deployments, pods, and services manually one by one, running `kubectl delete -f deployment-v1.yaml` reads the file and systematically removes all declared resources from the specified namespace.
*   **Smart Change Detection (`kubectl apply`):** When you modify a single resource within a multi-object manifest file (such as updating only the Service's selector from broad to specific) and run `kubectl apply -f`, Kubernetes compares the desired state with the live state and updates *only* the modified object, leaving stable workloads untouched.

#### B. Dynamic Traffic Shifting via Selectors
*   **Default State (Broad Matching):** If a Service selector matches only `app: hello-world-rest-api`, it selects both `v1` and `v2` pods, resulting in round-robin load balancing between both application versions.
*   **Targeted Isolation (Specific Matching):** Adding a second label constraint like `version: v1` or `version: v2` to the Service selector acts as a logical **AND** filter, routing user traffic exclusively to that specific version.

---

### 4. Practical Setup & Implementation Guide

#### Step 1: Cleaning Up Previous Resources
Delete all resources defined in the older manifest file:
```bash
kubectl delete -f deployment-v1.yaml
```
*Expected Outcome:* The deployment and service associated with `deployment-v1.yaml` are deleted from your namespace.

#### Step 2: Deploying Multi-Version Architecture (`deployment-v2.yaml`)
Apply the new multi-object manifest containing V1, V2, and a broad Service:
```bash
kubectl apply -f deployment-v2.yaml
```
*Expected Outcome:* Two deployments (`hello-world-rest-api-v1` and `hello-world-rest-api-v2`) and one load balancer service are created.

#### Step 3: Verifying Pod Readiness and Load Balancing
1. Check pod status in your custom namespace:
   ```bash
   kubectl get pods --namespace=hello-world-rest-api-ns
   ```
2. Retrieve the external LoadBalancer IP:
   ```bash
   kubectl get service --namespace=hello-world-rest-api-ns
   ```
3. Test endpoint responses via `curl`:
   ```bash
   curl http://<EXTERNAL-IP>:8080/hello-world
   ```
   *Observation:* Repeated requests alternate responses between `v1` and `v2` because the broad service selector matches both pod sets.

#### Step 4: Restricting Traffic to Version 1 Only
1. Open `deployment-v2.yaml` and update the Service selector to include `version: v1`:
   ```yaml
   spec:
     selector:
       app: hello-world-rest-api
       version: v1
   ```
2. Re-apply the manifest:
   ```bash
   kubectl apply -f deployment-v2.yaml
   ```
   *Observation:* Subsequent `curl` requests return responses **exclusively from v1**.

#### Step 5: Shifting Traffic to Version 2 Only
1. Update the Service selector in `deployment-v2.yaml` to specify `version: v2`:
   ```yaml
   spec:
     selector:
       app: hello-world-rest-api
       version: v2
   ```
2. Re-apply the manifest:
   ```bash
   kubectl apply -f deployment-v2.yaml
   ```
   *Observation:* Subsequent `curl` requests return responses **exclusively from v2**.

#### Step 6: Full Resource Cleanup
Delete all resources defined in `deployment-v2.yaml` when finished:
```bash
kubectl delete -f deployment-v2.yaml
```

---

### 5. Visual Representations

#### Dynamic Selector Traffic Shifting Workflow
```text
[ deployment-v2.yaml ] 
         │
         ├──> Deployment V1 (Pods: app=rest-api, version=v1)
         ├──> Deployment V2 (Pods: app=rest-api, version=v2)
         └──> Service (Selector matches labels)
                       │
         ┌─────────────┴─────────────┐
         ▼                           ▼
  Selector: {app}             Selector: {app, version:v2}
         │                           │
         ▼                           ▼
[ Balances V1 & V2 ]       [ Routes to V2 Only ]
```
*Explanation:* By modifying the Service's selector inside the YAML file and running `kubectl apply`, traffic routing shifts dynamically between joint load balancing and strict version isolation.

---

### 6. Mental Models & Analogies

*   **The Analogy:** **A Radio Station Frequency Tuner**
*   **The Mapping:**
    *   *Deployments* = Multiple broadcast towers transmitting different music channels simultaneously (`v1` and `v2`).
    *   *Service Selector* = The frequency dial on your radio receiver. When tuned broadly, you hear a mix of both stations; when fine-tuned to a specific frequency sub-tag, you isolate and listen to just one channel clearly.

---

### 7. Step-by-Step Workflows / Comparison Tables

#### Common CLI Operations Reference Table

| Task Description | Command Syntax |
| :--- | :--- |
| **Delete all resources in a YAML file** | `kubectl delete -f <filename>.yaml` |
| **Apply/Update resources in a YAML file** | `kubectl apply -f <filename>.yaml` |
| **Check pods in a specific namespace** | `kubectl get pods --namespace=<namespace-name>` |
| **Check service status and external IPs** | `kubectl get service --namespace=<namespace-name>` |

---

### 8. Common Mistakes & Exam Traps

*   *Trap:* Assuming that running `kubectl delete -f` requires you to specify each resource name individually.
    *   *Correction:* The `-f` flag tells `kubectl` to read the manifest file and automatically delete *all* objects defined within it in the correct order.
*   *Trap:* Thinking that changing a Service selector requires restarting the pods or deployments.
    *   *Correction:* Service selectors are independent network rules; applying an updated selector updates the routing table instantly without touching container workloads.

---

### 9. Certification Practice Questions

#### Question 1
An administrator needs to delete all resources (deployments, services, etc.) that were previously created using a manifest file named `deployment-v1.yaml`. Which single command should be executed?
*   A) `kubectl destroy --all`
*   B) `kubectl delete -f deployment-v1.yaml`
*   C) `kubectl remove deployment --file=deployment-v1.yaml`
*   D) `kubectl purge -all-resources`

**Correct Answer:** **B**
*Explanation:* `kubectl delete -f <filename>.yaml` parses the configuration file and removes all resources declared within it.
*Incorrect Options:* A, C, and D use invalid or non-existent syntax.
*Exam Takeaway:* Use `kubectl delete -f <filename>` for clean, file-based resource deletion.

---

#### Question 2
When you run `kubectl apply -f deployment-v2.yaml` after modifying only the Service selector section within a multi-object manifest file, how does Kubernetes process the update?
*   A) It deletes and recreates every single object defined in the file from scratch.
*   B) It intelligently recognizes that only the Service object definition changed and updates only that specific object.
*   C) It rejects the file because partial updates are not allowed.
*   D) It restarts all cluster worker nodes.

**Correct Answer:** **B**
*Explanation:* `kubectl apply` performs intelligent differential tracking, updating only the resources that have actual configuration changes.
*Incorrect Options:* A, C, and D are false.
*Exam Takeaway:* `kubectl apply` updates only modified resources in a manifest file.

---

#### Question 3
Your cluster runs two deployments: V1 and V2, both carrying the label `app: rest-api`. Initially, your Service routes traffic to both versions. You update the Service selector to include `version: v2`. What is the immediate effect on incoming client requests?
*   A) Requests fail with a 503 service unavailable error.
*   B) Requests are routed exclusively to version 2 pods.
*   C) Requests continue balancing across both V1 and V2 pods.
*   D) Version 1 pods are automatically deleted from the cluster.

**Correct Answer:** **B**
*Explanation:* Narrowing the Service selector to include `version: v2` filters out V1 pods, routing user traffic strictly to V2 instances.
*Incorrect Options:* A, C, and D are incorrect.
*Exam Takeaway:* Updating service selectors immediately shifts traffic routing to match the new filter criteria.

---

#### Question 4
True or False: Deleting a Service object automatically deletes the underlying Deployment and Pods that were handling its traffic.
*   A) True, services act as parent controllers over deployments.
*   B) False, services and deployments are independent objects; deleting a service only stops external traffic routing, leaving the deployment and pods running.

**Correct Answer:** **B**
*Explanation:* Services and Deployments are decoupled. Deleting a service does not delete the pods or deployments it was routing traffic to.
*Exam Takeaway:* Services and Deployments operate independently; deleting one does not delete the other.

---

#### Question 5
If a Service selector matches multiple versions of an application deployment that share the same app label, how does the Service handle incoming requests?
*   A) It drops all requests to prevent version conflicts.
*   B) It load balances requests across all active matching pods from both deployments.
*   C) It forwards 100% of traffic to the newest pod and keeps older pods idle.
*   D) It requires manual round-robin proxy configurations.

**Correct Answer:** **B**
*Explanation:* Kubernetes services act as internal load balancers, distributing incoming traffic across all healthy pods matching the service's selector.
*Incorrect Options:* A, C, and D are false.
*Exam Takeaway:* Services load balance across all pods matching their selectors.

---

#### Question 6
Which command verifies the external IP address assigned to a Kubernetes LoadBalancer service?
*   A) `kubectl get service`
*   B) `kubectl describe nodes`
*   C) `kubectl cluster-info external`
*   D) `kubectl ip-lookup`

**Correct Answer:** **A**
*Explanation:* `kubectl get service` (or `kubectl get svc`) lists services along with their assigned cluster IPs and external load balancer IP addresses.
*Incorrect Options:* B, C, and D are incorrect or invalid commands.
*Exam Takeaway:* Use `kubectl get service` to check external IPs for LoadBalancer services.

---

#### Question 7
Why might an external `curl` request to a newly provisioned GKE LoadBalancer service fail immediately after creation, requiring a short wait time?
*   A) The cluster control plane is restarting.
*   B) The cloud provider load balancer needs time to provision infrastructure, wire up routing rules, and establish healthy target pools.
*   C) Kubernetes requires explicit password authentication for all curl commands.
*   D) Container ports must be manually bound via SSH.

**Correct Answer:** **B**
*Explanation:* External cloud load balancers take a few minutes to spin up and pass initial health checks before accepting live traffic.
*Incorrect Options:* A, C, and D are false.
*Exam Takeaway:* Allow time for cloud load balancers to initialize health checks post-deployment.

---

#### Question 8
What type of logic is applied when multiple key-value pairs are defined inside a Service's `spec.selector` block?
*   A) OR logic (pods matching any single key are selected)
*   B) AND logic (pods must match all specified key-value pairs)
*   C) Random selection logic
*   D) Exclusive negation logic

**Correct Answer:** **B**
*Explanation:* Multiple selector fields enforce logical **AND** constraints, requiring a pod to possess all specified labels to be selected.
*Incorrect Options:* A, C, and D misrepresent selector matching logic.
*Exam Takeaway:* Multiple selector keys enforce logical **AND** matching.

---

#### Question 9
If you wish to remove all deployed resources associated with `deployment-v2.yaml` from your cluster, what command should you run?
*   A) `kubectl delete -f deployment-v2.yaml`
*   B) `kubectl remove app --file=deployment-v2.yaml`
*   C) `kubectl clean deployment-v2.yaml`
*   D) `kubectl terminate -f deployment-v2.yaml`

**Correct Answer:** **A**
*Explanation:* `kubectl delete -f <filename>` reads the manifest file and deletes all associated resources.
*Incorrect Options:* B, C, and D are invalid syntax.
*Exam Takeaway:* Use `kubectl delete -f <filename>` to clean up multi-object manifests.

---

#### Question 10
True or False: You must restart your GKE cluster nodes every time you update a Service's label selector.
*   A) True, node reboots are mandatory for networking changes.
*   B) False, Service selector changes take effect dynamically via the Kubernetes API without requiring node reboots.

**Correct Answer:** **B**
*Explanation:* Kubernetes API objects update dynamically. Changing a selector in a YAML file and running `kubectl apply` takes effect instantly without restarting nodes or workloads.
*Exam Takeaway:* Kubernetes configuration changes apply dynamically through the API without requiring node reboots.