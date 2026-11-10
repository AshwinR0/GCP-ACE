### 1. Overview
As part of Kubernetes Deployment YAML manifests, you can configure runtime **Environment Variables**, **Resource Requests**, and **Resource Limits**. 

- **Environment Variables** allow you to pass configuration values (such as database hostnames or API keys) into your container at runtime without hardcoding them into your application source code.
- **Resource Requests and Limits** allow you to manage and govern CPU and memory usage. **Requests** guarantee a minimum amount of resources for a container, while **Limits** cap the maximum amount of resources a container is allowed to consume, preventing a single runaway container from starving others on the node.

---

### 2. ACE Exam Focus & Common Exam Traps

*   **Exam Importance:** **HIGH**. The ACE exam frequently tests resource management concepts (Requests vs. Limits) and how containers consume cluster resources.
*   **High-Priority Topics:**
    *   **Resource Requests (`resources.requests`):** Minimum guaranteed resources for a container. Used by the Kubernetes scheduler to place pods onto nodes with enough free capacity.
    *   **Resource Limits (`resources.limits`):** Maximum ceiling of resources a container can consume. If a container exceeds its memory limit, it is terminated (OOMKilled). If it exceeds its CPU limit, it is throttled.
    *   **Environment Variables (`env`):** Key-value pairs passed directly into the container's runtime environment.
*   **Common Exam Traps:**
    *   Confusing Requests with Limits: **Requests** are used for scheduling and guarantees; **Limits** act as hard boundaries and security caps.
    *   Misunderstanding CPU units: `250m` means 250 millicores (or 0.25 of a single CPU core), and `0.5` or `500m` represents half of a CPU core.
    *   Thinking exceeding a memory limit throttles the application (Memory limits result in an **OOMKilled** error and container crash, whereas CPU limits result in CPU **throttling/slowdown**).

---

### 3. Detailed Concept Explanation

#### A. Environment Variables (`env`)
*   **Purpose:** Inject configuration data dynamically into containers at runtime.
*   **Implementation:** Configured under `spec.template.spec.containers.env` as a list of name-value pairs.
*   **Use Case:** Passing non-secret configuration values like database hosts, operational environments (`prod`, `staging`), or feature flags.

#### B. Resource Requests (`resources.requests`)
*   **Definition:** The minimum amount of CPU and memory guaranteed to a container.
*   **Scheduler Role:** When Kubernetes schedules a pod, it checks if the target node has enough *unallocated* resource requests to satisfy the pod's requirements. If a node lacks capacity, the pod remains in a `Pending` state.
*   **Units:**
    *   **CPU:** Measured in CPU cores or **millicores** (`m`). E.g., `250m` = 0.25 of a core; `1` = 1 full CPU core.
    *   **Memory:** Measured in bytes using standard binary or decimal suffixes. E.g., `128Mi` (Mebibytes) or `1Gi` (Gibibytes).

#### C. Resource Limits (`resources.limits`)
*   **Definition:** The maximum ceiling or boundary of CPU and memory a container is permitted to consume.
*   **Enforcement:**
    *   **Memory Limit Exceeded:** If a container tries to use more memory than its limit, Kubernetes terminates the container process with an **Out Of Memory (OOMKilled)** error.
    *   **CPU Limit Exceeded:** CPU is a compressible resource. If a container hits its CPU limit, Kubernetes throttles its processing speed rather than crashing it.

---

### 4. Practical Setup & Implementation Guide

#### Step 1: Writing a Deployment Manifest with Environment Variables and Resources
Create a file named `app-config-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: configured-app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: configured-app
  template:
    metadata:
      labels:
        app: configured-app
    spec:
      containers:
      - name: main-app
        image: gcr.io/google-samples/hello-app:1.0
        ports:
        - containerPort: 8080
        
        # 1. Environment Variables Configuration
        env:
        - name: DATABASE_HOST
          value: "my-db-service"
        - name: ENVIRONMENT
          value: "production"

        # 2. Resource Requests and Limits Configuration
        resources:
          requests:
            memory: "128Mi"
            cpu: "250m"         # 0.25 of a CPU core
          limits:
            memory: "512Mi"
            cpu: "500m"         # 0.5 of a CPU core
```

#### Step 2: Applying the Manifest
Deploy the resource to your GKE cluster:
```bash
kubectl apply -f app-config-deployment.yaml
```

#### Step 3: Verifying the Setup
*   Check the deployment rollout status:
    ```bash
    kubectl rollout status deployment/configured-app-deployment
    ```
*   Inspect the running pod to verify environment variables and resource allocations:
    ```bash
    kubectl describe pod <pod-name>
    ```

#### Step 4: Cleanup
Delete the deployment:
```bash
kubectl delete -f app-config-deployment.yaml
```

---

### 5. Visual Representations

#### Requests vs. Limits and Node Scheduling Flow
```text
[ Pod Manifest Request: 250m CPU, 128Mi Memory ]
                       │
                       ▼
         +───────────────────────────+
         |     Kubernetes Scheduler  |
         +───────────────────────────+
                       │
         Evaluates Node Free Capacity
                       │
       ┌───────────────┴───────────────┐
       ▼                               ▼
[ Node A: Has Capacity ]      [ Node B: Insufficient Capacity ]
       │                               │
       ▼                               ▼
( Pod Scheduled Successfully )    ( Pod remains PENDING )
```
*Explanation:* The scheduler relies on `resources.requests` to determine whether a node has enough available headroom to host a new pod.

---

### 6. Mental Models & Analogies

*   **The Analogy:** **A Shared Office Workspace and Power Contracts**
*   **The Mapping:**
    *   *Resource Request* = A guaranteed baseline desk space and minimum electrical power supply your office lease promises you so you can guarantee your computers turn on.
    *   *Resource Limits* = A circuit breaker fuse box that caps your maximum power consumption. If you try to plug in too many heavy appliances and pull more power than your limit, the breaker trips (analogous to an **OOMKilled** memory termination).

---

### 7. Step-by-Step Workflows / Comparison Tables

#### Comparison Table: Resource Requests vs. Resource Limits

| Feature | Resource Requests (`resources.requests`) | Resource Limits (`resources.limits`) |
| :--- | :--- | :--- |
| **Definition** | The minimum amount of resources guaranteed for a container. | The maximum ceiling of resources a container can consume. |
| **Scheduler Impact** | **Used by the scheduler** to find a node with enough free capacity. | Not used for initial pod placement scheduling. |
| **Exceeding Action** | N/A (Guaranteed baseline). | **Memory:** Container is terminated (`OOMKilled`).<br>**CPU:** Container is throttled (slowed down). |
| **Best Practice** | Set close to the average expected usage of the application. | Set based on peak load bounds to protect cluster nodes from resource starvation. |

---

### 8. Common Mistakes & Exam Traps

*   *Trap:* Assuming that a container exceeding its CPU limit will crash with an error.
    *   *Correction:* CPU is compressible; exceeding a CPU limit results in **throttling** (latency/slowdown), whereas exceeding a memory limit results in an immediate crash via **OOMKilled**.
*   *Trap:* Confusing where environment variables are declared in a YAML file.
    *   *Correction:** Environment variables (`env`) are defined directly under the container specification block (`spec.template.spec.containers.env`), alongside image configurations and resource blocks.

---

### 9. Certification Practice Questions

#### Question 1
An administrator is configuring a deployment manifest in GKE and wants to ensure that a container is allocated a minimum baseline of 250 millicores of CPU and 128Mi of memory. Which section of the container specification should be used?
*   A) `resources.limits`
*   B) `resources.requests`
*   C) `env.minimums`
*   D) `scheduling.guarantees`

**Correct Answer:** **B**
*Explanation:* `resources.requests` specifies the minimum guaranteed resources required by a container, which the Kubernetes scheduler uses for placement.
*Incorrect Options:* A sets maximum limits. C and D are invalid configuration paths.
*Exam Takeaway:* Minimum guaranteed resources are always configured under `resources.requests`.

---

#### Question 2
What action does Kubernetes take when a container exceeds its configured **memory limit** (`resources.limits.memory`)?
*   A) It throttles the container's processing speed until memory usage drops.
*   B) It automatically migrates the pod to a larger GKE node instance.
*   C) It terminates the container with an Out Of Memory (`OOMKilled`) error.
*   D) It borrows extra memory dynamically from neighboring system pods.

**Correct Answer:** **C**
*Explanation:* Memory is an incompressible resource in Kubernetes. If a container exceeds its memory limit, Kubernetes terminates it with an `OOMKilled` exit status.
*Incorrect Options:* A describes CPU throttling behavior. B and D do not happen automatically based solely on hitting a memory limit.
*Exam Takeaway:* Memory limit violations result in container termination (`OOMKilled`).

---

#### Question 3
How are CPU resources typically measured and declared within a Kubernetes resource request or limit configuration?
*   A) In gigabytes (GB) and megabytes (MB) only
*   B) In CPU cores or millicores (`m`)
*   C) In network packets per second (pps)
*   D) In disk input/output operations per second (IOPS)

**Correct Answer:** **B**
*Explanation:* CPU resources in Kubernetes are measured in CPU units (e.g., `1` for a full core) or millicores (e.g., `250m` for a quarter of a core).
*Incorrect Options:* A measures memory. C and D measure network and storage performance, not CPU.
*Exam Takeaway:* CPU requests and limits use cores or millicores (`m`).

---

#### Question 4
An application container in your GKE cluster attempts to use more CPU cycles than specified in its `resources.limits.cpu` parameter. What happens to the container?
*   A) It crashes immediately with a CPU exhaustion exit code.
*   B) It is throttled (its execution speed is slowed down by the kernel).
*   C) It automatically borrows unused CPU capacity from other nodes across the region.
*   D) It restarts via a liveness probe failure.

**Correct Answer:** **B**
*Explanation:* CPU is a compressible resource. When a container hits its CPU limit, Kubernetes throttles its CPU allocation rather than killing it.
*Incorrect Options:* A describes memory limit behavior (`OOMKilled`). C and D are incorrect.
*Exam Takeaway:* CPU limit violations lead to **throttling**, not termination.

---

#### Question 5
You need to pass a non-secret configuration parameter named `DATABASE_HOST` with a value of `my-db` into your containerized application via the deployment YAML file. Where should this be placed?
*   A) Under `spec.template.spec.containers.env`
*   B) Under `resources.requests.database`
*   C) Under `metadata.annotations`
*   D) Under `spec.strategy.rollingUpdate`

**Correct Answer:** **A**
*Explanation:* Runtime environment variables are declared as a list under the `env` field inside the container specification (`spec.template.spec.containers.env`).
*Incorrect Options:* B is for resource allocations. C is for descriptive notes. D is for rolling update rules.
*Exam Takeaway:* Environment variables are configured under the container's `env` block.

---

#### Question 6
If a pod's `resources.requests` requirements exceed the remaining unallocated capacity on every single node inside your GKE cluster, what state will the pod remain in?
*   A) `Running`
*   B) `Pending`
*   C) `Failed`
*   D) `Completed`

**Correct Answer:** **B**
*Explanation:* When the scheduler cannot find a node with enough available resources to satisfy a pod's requests, the pod gets stuck in the `Pending` state.
*Incorrect Options:* A requires successful scheduling. C and D occur after execution attempts or termination.
*Exam Takeaway:* Unschedulable pods due to lack of resource requests remain in the `Pending` state.

---

#### Question 7
Which of the following values represents 0.5 of a single CPU core in a Kubernetes resource specification?
*   A) `50m`
*   B) `500m`
*   C) `5000m`
*   D) `0.05`

**Correct Answer:** **B**
*Explanation:* `500m` stands for 500 millicores, which equals half (`0.5`) of a full CPU core (`1000m` = 1 core).
*Incorrect Options:* A is 0.05 cores. C is 5 cores. D is 0.05 cores.
*Exam Takeaway:* Millicores (`m`) represent fractions of a CPU core where `1000m` equals 1 full core.

---

#### Question 8
What is the primary role of **Resource Requests** during the Kubernetes scheduling process?
*   A) They dictate the maximum memory a pod can use before crashing.
*   B) They allow the scheduler to match a pod to a node that has enough free resources to guarantee those minimum requirements.
*   C) They encrypt environment variables passed into the container.
*   D) They configure external load balancer port bindings automatically.

**Correct Answer:** **B**
*Explanation:* The scheduler uses resource requests to ensure that node capacity is not overcommitted beyond safe operating thresholds when placing new pods.
*Incorrect Options:* A describes resource limits. C and D are unrelated.
*Exam Takeaway:* Resource requests are used by the scheduler for node placement and capacity planning.

---

#### Question 9
True or False: Setting resource limits is mandatory for every container deployed in a Kubernetes cluster.
*   A) True, Kubernetes rejects all pods lacking resource limits.
*   B) False, resource requests and limits are optional, though highly recommended as a best practice.

**Correct Answer:** **B**
*Explanation:* While configuring requests and limits is an essential production best practice, Kubernetes does not strictly mandate them unless enforced via cluster-level LimitRanges or Validating Webhooks.
*Exam Takeaway:* Resource requests and limits are optional in default Kubernetes configurations, but recommended best practices.

---

#### Question 10
If you omit resource limits entirely on a container specification, what happens regarding its resource consumption on the GKE node?
*   A) The container can consume all available CPU and memory resources on the node, potentially starving other pods.
*   B) The container is automatically assigned a hard limit of 10 megabytes of memory.
*   C) The container fails to start due to a validation error.
*   D) The container runs with zero CPU access.

**Correction Answer:** **A**
*Explanation:* Without limits, a container can consume as much node resource as is available, which can lead to "noisy neighbor" issues where one runaway container crashes other workloads on the same node.
*Incorrect Options:* B, C, and D are incorrect; Kubernetes does not enforce arbitrary low caps or prevent startup unless configured otherwise.
*Exam Takeaway:* Always define resource limits to prevent runaway containers from exhausting node resources.