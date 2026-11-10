# Google Kubernetes Engine (GKE) - Connecting and Deploying Applications Study Notes

---

### 1. Overview
This module covers connecting to a newly provisioned GKE cluster using Google Cloud Shell and deploying a containerized sample application (*Hello, World REST API*) from Docker Hub [4]. It outlines how to fetch cluster credentials via `gcloud`, schedule container workloads using `kubectl create deployment`, expose services externally using a Kubernetes `LoadBalancer`, and verify application accessibility via an external IP address and port [4].

---

### 2. ACE Exam Focus & Common Exam Traps
- **Exam Importance:** **HIGH**
- **Core Concepts:**
  - The distinct roles of `gcloud` vs. `kubectl` [4].
  - Populating local configuration (`kubeconfig`) using `gcloud container clusters get-credentials` [4].
  - Exposing internal pod deployments to external traffic using a Kubernetes service of type `LoadBalancer` [4].
- **Key Distinctions:**
  - **`gcloud` CLI:** Used for managing Google Cloud resources (creating clusters, updating IAM, fetching cluster connection parameters) [4].
  - **`kubectl` CLI:** The standard Kubernetes command-line tool used to interact with the Kubernetes API server inside the cluster to manage deployments, pods, and services [4].
- **Exam Keywords to Watch For:**
  - *"Configure kubeconfig file / download authorization data for a GKE cluster"* $\rightarrow$ `gcloud container clusters get-credentials` [4].
  - *"Expose a deployment to external internet traffic via an external IP address"* $\rightarrow$ Service type `LoadBalancer` [4].

---

### 3. Detailed Concept Explanation

#### Google Cloud Shell Environment
- **What is Cloud Shell?** A free, browser-based administrative terminal pre-loaded with essential developer tools, including the Google Cloud SDK (`gcloud`), Kubernetes CLI (`kubectl`), and code editors (Cloud Code) [4].
- **Convenience:** Eliminates local tool installation dependencies and automatically manages user credentials for API authorization [4].

#### Connecting to GKE Clusters (`gcloud`)
- To interact with a Kubernetes cluster using `kubectl`, the local environment must know the cluster's endpoint and possess valid security certificates [4].
- The `gcloud container clusters get-credentials` command downloads these security details and automatically updates your local `kubeconfig` file [4].

#### Deploying Workloads and Exposing Services (`kubectl`)
1. **Deployments (`kubectl create deployment`):** Manages the desired state of pods running container images pulled from registries (e.g., Docker Hub) [4].
2. **Services (`kubectl expose deployment`):** Pods have internal, ephemeral IP addresses. To access a pod from outside the cluster, a Kubernetes **Service** of type `LoadBalancer` must be created, which provisions an external TCP/IP load balancer [4].

---

### 4. Practical Setup & Implementation Guide

#### Step 1: Launch and Configure Cloud Shell
1. Open the Google Cloud Console (`console.cloud.google.com`) [4].
2. Click the **Activate Cloud Shell** icon in the top right header bar [4].
3. Click **Continue** on the introductory prompt and **Authorize** credentials when prompted [4].
4. *(Optional)* Click the **Open in new window** icon for an isolated terminal tab and adjust terminal settings (text size, color theme) if needed [4].

#### Step 2: Fetch Cluster Credentials
1. Navigate to **Kubernetes Engine** > **Clusters** in the Cloud Console [4].
2. Locate your cluster (`my-cluster`), click the three dots or click directly on the cluster, and click **Connect** [4].
3. Copy the provided `gcloud container clusters get-credentials` command [4].
4. Paste and run the command in Cloud Shell [4]:
   ```bash
   gcloud container clusters get-credentials my-cluster --region=<your-region> --project=<your-project-id>
   ```
   *Expected Outcome:* Response confirms that the cluster endpoint and authorization data were successfully fetched and added to the `kubeconfig` file [4].

#### Step 3: Deploy the Containerized Application
1. Run the `kubectl` command to create a deployment using a pre-built public image from Docker Hub [4]:
   ```bash
   kubectl create deployment hello-world-rest-api \
       --image=in28min/hello-world-rest-api:0.0.1.RELEASE
   ```
   *Expected Outcome:* Confirmation that the deployment has been created.

#### Step 4: Expose the Deployment via a Load Balancer Service
1. Expose the deployment externally on port `8080` using a service of type `LoadBalancer` [4]:
   ```bash
   kubectl expose deployment hello-world-rest-api \
       --type=LoadBalancer \
       --port=8080
   ```
   *Expected Outcome:* Confirmation that the service has been exposed.

#### Step 5: Verify External Access
1. Retrieve the external IP address of the service [4]:
   ```bash
   kubectl get services
   ```
   *(Note: If the `EXTERNAL-IP` column displays `<pending>`, wait a couple of minutes or use `kubectl get services --watch` until an external IP is assigned) [4].*
2. Copy the external IP address.
3. Open a new browser tab and navigate to:
   `http://<EXTERNAL-IP>:8080` (should display `healthy:true`) [4]
   `http://<EXTERNAL-IP>:8080/hello-world` (should return the "Hello World" application response) [4].

---

### 5. Visual Representations

#### Request and Deployment Flow Architecture
```text
[ Developer in Cloud Shell ]
       |
       +---> 1. Runs: gcloud container clusters get-credentials (updates kubeconfig)
       |
       +---> 2. Runs: kubectl create deployment (sends request to GKE Control Plane)
       |
       +---> 3. Runs: kubectl expose deployment --type=LoadBalancer
               |
               v
     +-------------------------------------------------------+
     |                  GKE Cluster                          |
     |                                                       |
     |   [ GCP External Load Balancer (External IP:Port) ]   |
     |                          |                            |
     |                          v                            |
     |            [ Kubernetes Service (Port 8080) ]         |
     |                          |                            |
     |                          v                            |
     |            [ Pod: hello-world-rest-api ]              |
     +-------------------------------------------------------+
```

---

### 6. Mental Models & Analogies

- **`gcloud` vs. `kubectl`:** Think of **`gcloud`** as the general building contractor that builds the physical office building and sets up the master building security badges (`get-credentials`), while **`kubectl`** is the internal facility manager that decides which employees sit in which rooms, moves furniture around, and organizes inner office procedures (`kubectl create deployment`, `expose`).

---

### 7. Step-by-Step Workflows / Comparison Tables

#### GKE Deployment and Exposure Workflow

| Phase | Command / Action | Purpose |
| :--- | :--- | :--- |
| **1. Authentication** | `gcloud container clusters get-credentials` | Downloads endpoint certs to link local terminal to cluster [4]. |
| **2. Workload Creation**| `kubectl create deployment <name> --image=<path>` | Schedules container pods on GKE worker nodes [4]. |
| **3. Network Exposure** | `kubectl expose deployment <name> --type=LoadBalancer --port=8080` | Provisions an external cloud load balancer to route public traffic to internal pods [4]. |
| **4. Verification** | `kubectl get services` | Retrieves the assigned `EXTERNAL-IP` for browser testing [4]. |

---

### 8. Common Mistakes & Exam Traps

- **Trap 1:** Confusing `gcloud` and `kubectl` responsibilities in exam scenarios.
  - *Correction:* Use `gcloud` for GCP project and cluster infrastructure-level operations; use `kubectl` exclusively for managing Kubernetes objects (pods, deployments, services, namespaces) inside the cluster.
- **Trap 2:** Expecting an `EXTERNAL-IP` immediately after running `kubectl expose`.
  - *Correction:* Provisioning an external cloud load balancer takes time. If the IP shows as `<pending>`, wait a moment or use the `--watch` flag [4].

---

### 9. Certification Practice Questions

#### Q1: An administrator has successfully created a new GKE cluster named `my-cluster`. They open Cloud Shell and want to configure their local environment to communicate with this specific cluster. Which command should they execute?
- A. `gcloud container clusters create-auto my-cluster`
- B. `gcloud container clusters get-credentials my-cluster`
- C. `kubectl config use-context my-cluster`
- D. `gcloud compute instances set-metadata my-cluster`
- **Correct Answer:** B
- **Explanation:** The `gcloud container clusters get-credentials` command fetches cluster endpoint data and security certificates, configuring the local `kubeconfig` file [4].
- **Incorrect Options:** A creates a new cluster. C assumes the context already exists. D targets Compute Engine metadata, not GKE credentials.
- **Exam Takeaway:** Use `get-credentials` to prepare a local environment for interacting with a GKE cluster via `kubectl`.

#### Q2: You have deployed a web application container inside a GKE cluster using `kubectl create deployment`. By default, pods are isolated within the cluster's internal network. What Kubernetes service type must you configure to expose this deployment to external public traffic via a cloud load balancer?
- A. ClusterIP
- B. NodePort
- C. LoadBalancer
- D. ExternalName
- **Correct Answer:** C
- **Explanation:** A service of type `LoadBalancer` provisions an external cloud load balancer integrated with the underlying cloud provider network to route public traffic to your pods [4].
- **Incorrect Options:** A (`ClusterIP`) exposes the service only internally within the cluster. B (`NodePort`) exposes it on a static port across each node's IP. D maps external names to DNS entries.
- **Exam Takeaway:** Exposing applications externally via a cloud provider load balancer requires a service of type `LoadBalancer` [4].

#### Q3: While executing `kubectl get services` immediately after creating a LoadBalancer service for your GKE deployment, the `EXTERNAL-IP` column displays `<pending>`. What is the most appropriate action?
- A. Delete the cluster and recreate it with standard mode.
- B. Run `gcloud auth login` again.
- C. Wait a couple of minutes for the cloud provider to provision the external load balancer, or check back using `--watch`.
- D. Restart the Cloud Shell virtual machine.
- **Correct Answer:** C
- **Explanation:** Provisioning a cloud load balancer takes time. An initial `<pending>` status is normal while GCP sets up the forwarding rules and external IPs [4].
- **Incorrect Options:** A, B, and D are unnecessary steps since asynchronous infrastructure provisioning is expected behavior.
- **Exam Takeaway:** A pending external IP on a Kubernetes LoadBalancer service simply requires time to provision.

#### Q4: What is the primary tool used to deploy, scale, and manage containerized applications directly on a Kubernetes cluster?
- A. `gcloud`
- B. `kubectl`
- C. `gsutil`
- D. `bq`
- **Correct Answer:** B
- **Explanation:** `kubectl` is the standard command-line interface for running commands against Kubernetes clusters [4].
- **Incorrect Options:** A (`gcloud`) manages Google Cloud resources. C (`gsutil`) manages Cloud Storage. D (`bq`) manages BigQuery.
- **Exam Takeaway:** `kubectl` is used for Kubernetes cluster application management.

#### Q5: You are using Google Cloud Shell to manage your GKE environment. Which of the following statements about Cloud Shell is true?
- A. It requires a paid subscription fee for enterprise users.
- B. It comes pre-installed with developer tools like `gcloud` and `kubectl`, and is free for users [4].
- C. It requires local installation of the Google Cloud SDK on your laptop.
- D. It only runs inside local Windows PowerShell windows.
- **Correct Answer:** B
- **Explanation:** Cloud Shell is a free, browser-based administrative environment pre-configured with essential tools like `gcloud` and `kubectl` [4].
- **Incorrect Options:** A, C, and D contain false statements regarding Cloud Shell's cost, installation requirements, and execution platform.
- **Exam Takeaway:** Cloud Shell provides a free, pre-configured environment with `gcloud` and `kubectl` out of the box [4].

#### Q6: When running a containerized REST API inside a GKE pod that listens on port 8080, how do you specify this port when exposing the deployment via `kubectl expose`?
- A. `--target-port=80`
- B. `--port=8080`
- C. `--internal-port=8080`
- D. `--container-port=80`
- **Correct Answer:** B
- **Explanation:** The `--port` flag configures the port that the service exposes (matching the application port, e.g., 8080) [4].
- **Incorrect Options:** A, C, and D use non-standard or incorrect flag names for the basic `kubectl expose` command syntax.
- **Exam Takeaway:** Use the `--port` flag matching your application port when exposing services with `kubectl` [4].

#### Q7: Which command should an administrator run to continuously monitor the status of services and watch for the assignment of an external IP address?
- A. `kubectl get services --watch`
- B. `kubectl describe cluster`
- C. `gcloud container operations list`
- D. `kubectl watch pods`
- **Correct Answer:** A
- **Explanation:** Appending `--watch` to `kubectl get services` allows continuous real-time updates of resource states, including IP assignments [4].
- **Incorrect Options:** B, C, and D do not provide real-time service IP assignment tracking.
- **Exam Takeaway:** Use `--watch` to monitor live updates on Kubernetes resources like services.

#### Q8: A developer has packaged their container application and pushed it to Docker Hub. When writing the `kubectl create deployment` command, how is the container image specified?
- A. Using the `--registry` flag pointing to Google Artifact Registry
- B. Using the `--image` flag followed by the repository image name and version tag [4]
- C. Using the `--source-code` flag pointing to a local folder path
- D. Using the `--dockerfile` flag
- **Correct Answer:** B
- **Explanation:** Container images are specified using the `--image=<image-name>:<tag>` flag in `kubectl create deployment` [4].
- **Incorrect Options:** A, C, and D use incorrect flag configurations for direct image deployment.
- **Exam Takeaway:** The `--image` flag specifies the container image and tag for deployments [4].

#### Q9: What happens if you execute `kubectl create deployment` without configuring a corresponding service?
- A. The pods will fail to start and crash-loop.
- B. The deployment runs successfully, but pods are only accessible internally within the cluster network and cannot be reached from the public internet.
- C. Google Cloud automatically assigns an external public IP address to every individual pod.
- D. The deployment command is rejected by the API server.
- **Correct Answer:** B
- **Explanation:** Deployments run successfully without a service, but they lack external ingress routing, meaning they can only communicate internally (ClusterIP scope) until explicitly exposed via a service [4].
- **Incorrect Options:** A, C, and D are incorrect; pods start fine internally without external services.
- **Exam Takeaway:** Services are required to provide network routing (such as LoadBalancers) to access deployments externally.

#### Q10: Why must you select your specific Google Cloud project and region when executing `gcloud container clusters get-credentials`?
- A. To ensure billing is routed to the correct cost center.
- B. Because multiple clusters with the same name can exist across different regions or projects, requiring specific identifiers to fetch the correct endpoint [4].
- C. To automatically delete old container logs.
- D. To grant temporary root administrator privileges to the user machine.
- **Correct Answer:** B
- **Explanation:** Cluster names can overlap across different projects and regions; specifying the project, region, and cluster name guarantees the correct cluster context is targeted [4].
- **Incorrect Options:** A, C, and D are unrelated to fetching cluster credentials.
- **Exam Takeaway:** Region, project, and cluster name uniquely identify a GKE cluster when retrieving credentials.