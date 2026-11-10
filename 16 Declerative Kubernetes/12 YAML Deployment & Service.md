# GCP Associate Cloud Engineer (ACE) Study Notes: Kubernetes YAML Deployment & Service Practical Implementation

---

### 1. Overview
This section demonstrates how to use a declarative Kubernetes YAML manifest (`deployment-v1.yaml`) to simultaneously create a **Namespace**, **Deployment**, and a **LoadBalancer Service** in a Google Kubernetes Engine (GKE) cluster. 

Instead of relying on imperative `kubectl run` or `kubectl create` commands, declarative manifests allow you to define the **desired state** of your infrastructure. Kubernetes then reconciles this desired state with the actual state in the cluster. This approach ensures reproducibility, consistency, and ease of multi-environment management (e.g., deploying the exact same file across development, staging, and production environments).

---

### 2. ACE Exam Focus & Common Exam Traps

*   **Exam Importance:** **HIGH**. The ACE exam heavily tests declarative object management (`kubectl apply -f`), namespace organization, multi-object manifests separated by `---`, and mapping Services to Deployments using selectors.
*   **High-Priority Topics:**
    *   **Imperative vs. Declarative Management:** Knowing why `kubectl apply -f` is preferred in production over ad-hoc imperative commands.
    *   **Multi-Object Manifests:** Understanding that a single YAML file can contain multiple Kubernetes resources separated by three hyphens (`---`).
    *   **Namespace Scoping:** How to create resources inside a specific namespace and how to query them using the namespace flag (`-n` or `--namespace`).
    *   **Service-to-Deployment Binding:** Ensuring a Service's `spec.selector` matches the Pod labels defined inside the Deployment's template (`spec.template.metadata.labels`).
*   **Common Exam Traps:**
    *   Forgetting to specify the namespace when checking resources (e.g., running `kubectl get pods` when the pod was deployed to a custom namespace, which results in an empty list because `kubectl` defaults to the `default` namespace).
    *   Confusing container-level resource limits with service-level port mappings.

---

### 3. Detailed Concept Explanation

#### A. Declarative Management via `kubectl apply -f`
*   **Concept:** You write text configurations (`.yaml` files) declaring what resources you want (`Deployment`, `Service`, `Namespace`).
*   **Command:** `kubectl apply -f <filename>.yaml`
*   **How it Works:** Kubernetes reads the file, parses the desired state, and applies changes (creating new objects or updating existing ones). If you modify the YAML file later and run `kubectl apply -f` again, Kubernetes performs an incremental update (rolling change) to match the new configuration.

#### B. Multi-Object Manifest Structure (`---`)
A single YAML configuration file can house multiple Kubernetes resources stacked back-to-back. They are separated by a document separator line consisting of three hyphens: `---`. In this module's context, a single file contains:
1.  A `ConfigMap` or setup definitions (or direct Deployment specifications).
2.  A `Deployment` object (`apps/v1`).
3.  A `Service` object (`v1` of type `LoadBalancer`).

#### C. Connecting Services to Pods via Selectors
*   **The Deployment Pod Template:** Inside the Deployment, `spec.template.metadata.labels` applies labels (e.g., `app: hello-world-rest-api`) to every pod instance spawned.
*   **The Service Selector:** The Service object contains `spec.selector`. It uses this selector query to look across the cluster and target *any* pod matching that label, routing external cloud load balancer traffic directly to those pods.

---

### 4. Practical Setup & Implementation Guide

#### Step 1: Open Cloud Shell Editor & Upload Manifest
1. Launch **Google Cloud Shell** from the GCP Console.
2. Click the **Open Editor** icon to launch the integrated Cloud Shell Code Editor.
3. Open the folder corresponding to your course materials (e.g., `in28minutes_demos` or the directory containing `deployment-v1.yaml`).
4. Drag and drop the `deployment-v1.yaml` file from your local machine into the Cloud Shell file explorer window.

#### Step 2: Clean Up Pre-existing Resources (Optional)
If you created objects in prior steps with conflicting names, delete them first:
```bash
kubectl delete service hello-world-rest-api
kubectl delete deployment hello-world-rest-api
```

#### Step 3: Create a Dedicated Namespace
Logical isolation is best practice. Create a dedicated namespace for your application:
```bash
kubectl create namespace hello-world-rest-api-namespace
```
*Expected Outcome:* Namespace is created successfully.

#### Step 4: Apply the Multi-Object YAML Manifest
Deploy both the Deployment and Service defined in the file to your custom namespace:
```bash
kubectl apply -f deployment-v1.yaml
```
*(Note: If your YAML file includes the namespace metadata directly inside the objects, they will deploy directly to that namespace).*

#### Step 5: Verify Deployment Status and Pods within the Namespace
Because resources were deployed to a custom namespace, you must explicitly pass the namespace flag (`-n` or `--namespace`):
```bash
kubectl get pods --namespace=hello-world-rest-api-namespace
```
*Expected Outcome:* Shows the running pod(s) with a status of `Running`.

#### Step 6: Check Service and External IP Assignment
Query the Service to retrieve its provisioned external load balancer IP address:
```bash
kubectl get service --namespace=hello-world-rest-api-namespace
```
*(Note: Provisioning a cloud `LoadBalancer` type service and assigning an external public IP can take a few minutes).*

#### Step 7: Test the Endpoint via `curl`
Once the external IP is ready, test connectivity:
```bash
curl http://<EXTERNAL-IP>:8080/hello-world
```
*Expected Output:* Returns the application response (e.g., `hello world v3`).

---

### 5. Visual Representations

#### Multi-Object Declarative Deployment Architecture
```text
+-----------------------------------------------------------------+
|                    deployment-v1.yaml File                      |
|                                                                 |
|   +-----------------------+         +-----------------------+   |
|   |   Deployment Object   |   ---   |     Service Object    |   |
|   |   (apps/v1)           |         |     (v1 / LoadBalancer|   |
|   +-----------------------+         +-----------------------+   |
+-----------------------------------------------------------------+
                                 │
                     `kubectl apply -f`
                                 │
                                 ▼
+-----------------------------------------------------------------+
| GKE Cluster (Namespace: hello-world-rest-api-namespace)         |
|                                                                 |
|  [ LoadBalancer Service ] ──(Selector match)──> [ Pod (v3) ]    |
+-----------------------------------------------------------------+
```
*Explanation:* Running `kubectl apply -f` processes the multi-document YAML file, deploying both the Deployment controller and the LoadBalancer service into the targeted GKE namespace simultaneously.

---

### 6. Mental Models & Analogies

*   The Analogy: **An Interior Design Blueprint Package**
*   The Mapping:
    *   *YAML Manifest* = A comprehensive blueprint package handed to a builder detailing where walls go (Deployments), where electrical sockets are placed (Services), and which floor the office is on (Namespaces).
    *   *`kubectl apply -f`* = Handing the blueprint package to the construction foreman so they can build or remodel the space to match the paper instructions precisely.

---

### 7. Step-by-Step Workflows / Comparison Tables

#### Comparison Table: Imperative Commands vs. Declarative YAML Files

| Feature | Imperative Commands (`kubectl run` / `create`) | Declarative YAML Manifests (`kubectl apply -f`) |
| :--- | :--- | :--- |
| **Reproducibility** | Low; hard to replicate identical setups manually. | **High; fully reproducible infrastructure-as-code.** |
| **Version Tracking** | Difficult to track changes over time. | **Easy; files can be stored in Git repositories (GitOps).** |
| **Complexity** | Quick for simple, ad-hoc testing and debugging. | Best for complex multi-object architectures and production systems. |
| **Reconciliation** | One-time execution command. | Continuously tracks and updates the **desired state**. |

---

### 8. Common Mistakes & Exam Traps

*   *Trap:* Running `kubectl get pods` and wondering why your newly created pods are missing, forgetting that they were deployed to a custom namespace (`hello-world-rest-api-namespace`) instead of `default`.
    *   *Correction:* Always specify the namespace flag (`-n <namespace-name>`) when querying resources outside the `default` namespace.
*   *Trap:* Assuming an external `LoadBalancer` IP is ready to answer HTTP requests instantly the exact second the IP address string appears in `kubectl get service`.
    *   *Correction:* Cloud provider load balancers take a few minutes to fully provision health checks and wire up backend target pools; if a `curl` command fails initially, wait 5–10 minutes and try again.

---

### 9. Certification Practice Questions

#### Question 1
An administrator runs `kubectl apply -f deployment-v1.yaml`. The YAML file contains both a `Deployment` and a `Service` object separated by three hyphens (`---`). How does Kubernetes process this file?
*   A) It throws a syntax error because only one resource type is allowed per YAML file.
*   B) It deploys only the first resource object and ignores everything after the hyphens.
*   C) It successfully parses and creates both the Deployment and the Service objects in the cluster.
*   D) It requires you to run two separate commands specifying individual object flags.

**Correct Answer:** **C**
*Explanation:* Kubernetes supports multi-object YAML manifests separated by `---`, allowing you to create multiple resources in a single apply command.
*Incorrect Options:* A, B, and D are false.
*Exam Takeaway:* Multiple Kubernetes objects can be grouped into a single YAML file using `---` separators.

---

#### Question 2
After successfully executing `kubectl apply -f deployment-v1.yaml` specifying a custom namespace `my-app-ns`, an engineer runs `kubectl get pods` and sees no pods listed. What is the most likely reason?
*   A) The pods failed to start and were deleted by the cluster.
*   B) The engineer forgot to specify the namespace flag (`-n my-app-ns`), and `kubectl` defaulted to checking the `default` namespace.
*   C) The container image download failed.
*   D) The YAML file syntax was rejected by the API server.

**Correct Answer:** **B**
*Explanation:* `kubectl` commands default to querying the `default` namespace unless explicitly told otherwise using `-n` or `--namespace`.
*Incorrect Options:* A, C, and D are incorrect assumptions.
*Exam Takeaway:* Always specify the namespace flag when checking resources outside `default`.

---

#### Question 3
What is the primary operational advantage of using declarative YAML manifests over imperative commands in production GKE environments?
*   A) YAML files run code directly on bare-metal CPU hardware without kernel virtualization.
*   B) YAML files enable version tracking, GitOps integration, code reviews, and reproducible infrastructure deployments.
*   C) Imperative commands are completely blocked by GKE security firewalls.
*   D) YAML files eliminate the need for container images.

**Correct Answer:** **B**
*Explanation:* Declarative files act as infrastructure-as-code, allowing teams to store configs in version control, audit changes, and roll back cleanly.
*Incorrect Options:* A, C, and D are false.
*Exam Takeaway:* Declarative configurations provide reproducibility and version control tracking.

---

#### Question 4
In a declarative Kubernetes manifest containing both a Service and a Deployment, how does the Service discover and route traffic to the Deployment's pods?
*   A) By referencing the deployment's random pod hash names directly.
*   B) By using a label selector (`spec.selector`) that matches the labels defined in the pod template.
*   C) By scanning the worker node IP tables manually via SSH.
*   D) By matching the container image version tag.

**Correct Answer:** **B**
*Explanation:* Services use label selectors (`spec.selector`) to dynamically discover and route traffic to any pod matching those labels.
*Incorrect Options:* A, C, and D do not describe Service-to-Pod routing mechanics.
*Exam Takeaway:* Services route traffic to pods using matching label selectors.

---

#### Question 5
Which `kubectl` command syntax correctly retrieves the status of all Pods inside a namespace named `hello-world-rest-api-namespace`?
*   A) `kubectl get pods --namespace=hello-world-rest-api-namespace`
*   B) `kubectl describe cluster --all`
*   C) `kubectl search pods -default`
*   D) `kubectl get all --cluster`

**Correct Answer:** **A**
*Explanation:* The `--namespace` (or `-n`) flag specifies the target namespace when querying resources with `kubectl get`.
*Incorrect Options:* B, C, and D use invalid or non-existent syntax.
*Exam Takeaway:* Use `kubectl get <resource> -n <namespace>` to query namespaced resources.

---

#### Question 6
When you define the desired state of a deployment in a YAML file and apply it, what does the Kubernetes control plane do if the actual cluster state differs from your desired state?
*   A) It terminates the entire cluster and alerts Google Cloud support.
*   B) It actively reconciles the actual state to match your declared desired state.
*   C) It updates your local YAML file to match the broken cluster state.
*   D) It prompts the administrator via a command-line interactive questionnaire.

**Correct Answer:** **B**
*Explanation:* Kubernetes controllers continuously run a reconciliation loop to ensure the actual state of the cluster matches the desired state declared in manifests.
*Incorrect Options:* A, C, and D are false.
*Exam Takeaway:* Kubernetes controllers continuously reconcile actual states with declared desired states.

---

#### Question 7
If a Service is created with `type: LoadBalancer` in GKE, what component is automatically provisioned in Google Cloud to route external traffic to your pods?
*   A) A Google Cloud External HTTP(S) or TCP/UDP Load Balancer
*   B) A Cloud NAT Gateway
*   C) A Virtual Private Cloud (VPC) Firewall rule blocking all traffic
*   D) An Artifact Registry repository

**Correct Answer:** **A**
*Explanation:* Creating a Kubernetes Service of type `LoadBalancer` in GKE automatically provisions a corresponding external cloud load balancer in your GCP project.
*Incorrect Options:* B, C, and D handle networking gateways, firewall blocking, and image storage, not load balancing.
*Exam Takeaway:* GKE LoadBalancer services automatically provision external GCP cloud load balancers.

---

#### Question 8
True or False: A single YAML manifest file can contain definitions for different resource kinds, such as a Namespace, a Deployment, and a Service, deployed together.
*   A) True
*   B) False

**Correct Answer:** **A**
*Explanation:* True. Multi-object YAML manifests can combine any number of distinct resource kinds in a single file by separating them with `---`.
*Exam Takeaway:* Multi-object manifests can include mixed resource kinds.

---

#### Question 9
Why might a freshly provisioned `LoadBalancer` service return connection errors or take a few minutes before successfully responding to an external `curl` request?
*   A) The cloud load balancer needs time to provision backend health checks and route traffic to node ports.
*   B) Kubernetes automatically shuts down pods after the first curl request.
*   C) External IP addresses require manual DNS registration before traffic passes.
*   D) The container image version is automatically downgraded.

**Correct Answer:** **A**
*Explanation:* Cloud load balancers take a brief period to spin up, configure forwarding rules, and establish healthy target pools with cluster nodes.
*Incorrect Options:* B, C, and D are incorrect.
*Exam Takeaway:* Allow a few minutes for external cloud load balancers to initialize health checks after provisioning.

---

#### Question 10
Which command allows you to change your active default namespace context in `kubectl` so you don't have to append `-n <namespace>` to every subsequent command?
*   A) `kubectl config set-context --current --namespace=<namespace-name>`
*   B) `kubectl set namespace-default <namespace-name>`
*   C) `kubectl switch-context <namespace-name>`
*   D) `kubectl namespace active <namespace-name>`

**Correct Answer:** **A**
*Explanation:* `kubectl config set-context --current --namespace=<name>` updates your active CLI context to target that namespace by default.
*Incorrect Options:* B, C, and D are invalid commands.
*Exam Takeaway:* Use `kubectl config set-context --current --namespace=<name>` to set your default namespace context.