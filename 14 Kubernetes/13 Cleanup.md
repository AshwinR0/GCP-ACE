# Google Kubernetes Engine (GKE) - Cleanup and Resource Deletion Study Notes

---

### 1. Overview
This module covers the cleanup phase of the GKE project journey [8]. It focuses on removing Kubernetes application objects (`kubectl delete service`, `kubectl delete deployment`) and shutting down the underlying cluster infrastructure (`gcloud container clusters delete` or via the Google Cloud Console) to prevent unnecessary cloud resource consumption and avoid unexpected billing charges [8].

---

### 2. ACE Exam Focus & Common Exam Traps
- **Exam Importance:** **HIGH**
- **Core Concepts:**
  - Deleting Kubernetes cluster objects using `kubectl delete` (Service first, then Deployment) [8].
  - Deleting GKE cluster infrastructure using the `gcloud` CLI (`gcloud container clusters delete`) or the Google Cloud Console [8].
- **Key Distinctions:**
  - **Deletion Order:** It is best practice to delete the `Service` (which destroys the external cloud LoadBalancer first) before deleting the `Deployment` to ensure proper network resource teardown [8].
  - **Tooling Rule:** `kubectl` is used to delete internal Kubernetes components (services, deployments, pods), whereas `gcloud` is used to delete the overarching GKE cluster infrastructure [8].
- **Exam Keywords to Watch For:**
  - *"Tear down external load balancers and services before removing app workloads"* $\rightarrow$ `kubectl delete service` followed by `kubectl delete deployment` [8].
  - *"Permanently remove a GKE cluster via command line"* $\rightarrow$ `gcloud container clusters delete` [8].

---

### 3. Detailed Concept Explanation

#### 1. Kubernetes Object Deletion Sequence
- **Step 1: Delete the Service (`kubectl delete service`)**
  - **Why first?** Deleting the service destroys the external cloud **LoadBalancer** provisioned by Google Cloud, freeing up networking endpoints [8]. Deleting this first avoids dangling external load-balancing rules [8].
  - **Command Syntax:**
    ```bash
    kubectl delete service hello-world-rest-api
    ```
- **Step 2: Delete the Deployment (`kubectl delete deployment`)**
  - Once the external network routing layer is removed, the application pods managed by the deployment can be safely terminated [8].
  - **Command Syntax:**
    ```bash
    kubectl delete deployment hello-world-rest-api
    ```

#### 2. Cluster Infrastructure Deletion (`gcloud` or Console)
- Once application workloads and services are cleared, the cluster itself can be deleted to stop infrastructure billing [8].
- **CLI Command:**
  ```bash
  gcloud container clusters delete <cluster-name> --region=<region>
  ```
- **Console Workflow:** Navigate to **Kubernetes Engine** > **Clusters**, locate the target cluster (`my-cluster`), click the context menu or select the cluster, click **Delete**, and confirm by typing the cluster name [8].

---

### 4. Practical Setup & Implementation Guide

#### Step 1: Inspect Current Kubernetes Objects
1. Open Cloud Shell [4].
2. List active deployments and services to verify target object names [8]:
   ```bash
   kubectl get deployment
   kubectl get service
   ```
   *Expected Outcome:* Lists `hello-world-rest-api` for both deployments and services [8].

#### Step 2: Delete the Service
1. Execute the deletion command for the service [8]:
   ```bash
   kubectl delete service hello-world-rest-api
   ```
   *Expected Outcome:* Deletion of the service may take a short moment while the external LoadBalancer resource is fully cleaned up by GCP [8].

#### Step 3: Delete the Deployment
1. Execute the deletion command for the deployment [8]:
   ```bash
   kubectl delete deployment hello-world-rest-api
   ```
   *Expected Outcome:* The deployment and its associated pods are removed almost instantly [8]. Verify with `kubectl get deployment` [8].

#### Step 4: Delete the GKE Cluster (Optional / Cost Management)
1. **Via CLI:**
   ```bash
   gcloud container clusters delete my-cluster --region=us-central1
   ```
2. **Via Console:**
   - Go to **Kubernetes Engine** > **Clusters** [8].
   - Select `my-cluster` [8].
   - Click **Delete**, type the cluster name to confirm, and click **Delete** [8].

---

### 5. Visual Representations

#### Teardown Sequence Flow
```text
[ Step 1: Delete Service ] 
       |
       v (Destroys external GCP LoadBalancer & frees public IP)
[ Step 2: Delete Deployment ] 
       |
       v (Terminates all running application pods instantly)
[ Step 3: Delete GKE Cluster (Optional) ]
       |
       v (Stops all control plane and node infrastructure billing)
```

---

### 6. Mental Models & Analogies

- **Teardown Sequence is like moving out of a rented retail storefront:** 
  - First, you take down your external business sign and disconnect the public utility meters (**Deleting the Service/LoadBalancer**). 
  - Next, you pack up your inventory and clear out the employees (**Deleting the Deployment/Pods**). 
  - Finally, you hand back the master building lease keys to the landlord (**Deleting the Cluster Infrastructure**).

---

### 7. Step-by-Step Workflows / Comparison Tables

#### GKE Resource Cleanup Workflow

| Order | Action | Command / Method | Purpose |
| :--- | :--- | :--- | :--- |
| **1** | Inspect Objects | `kubectl get deployment` / `kubectl get service` [8] | Confirms active object names before deletion [8]. |
| **2** | Delete Service | `kubectl delete service <name>` [8] | Destroys external LoadBalancer and routing rules [8]. |
| **3** | Delete Deployment | `kubectl delete deployment <name>` [8] | Terminates application workload pods [8]. |
| **4** | Delete Cluster | `gcloud container clusters delete` or Console [8] | Shuts down master control plane and worker nodes to stop billing [8]. |

---

### 8. Common Mistakes & Exam Traps

- **Trap 1:** Trying to delete cluster infrastructure using `kubectl delete cluster`.
  - `kubectl` is restricted to cluster-internal objects; it cannot delete the overarching GKE cloud cluster resource. Cluster deletion requires `gcloud` or the Cloud Console [8].
- **Trap 2:** Leaving external LoadBalancer services running after deleting workloads.
  - Failing to delete services tied to external load balancers can result in dangling cloud networking charges even after application pods are removed [8].

---

### 9. Certification Practice Questions

#### Q1: An administrator wants to clean up a GKE environment by deleting an application deployment and its external exposure service. Which command correctly removes the Kubernetes service named `hello-world-rest-api`?
- A. `gcloud container services delete hello-world-rest-api`
- B. `kubectl delete service hello-world-rest-api` [8]
- C. `kubectl remove service hello-world-rest-api`
- D. `gcloud compute firewall-rules delete hello-world-rest-api`
- **Correct Answer:** B
- **Explanation:** Kubernetes objects are managed via `kubectl`, and the delete command syntax is `kubectl delete service <name>` [8].
- **Incorrect Options:** A uses an invalid gcloud structure. C uses `remove` instead of `delete`. D targets VPC firewall rules.
- **Exam Takeaway:** Use `kubectl delete service` to remove Kubernetes services [8].

#### Q2: What underlying GCP cloud resource is typically destroyed when you execute `kubectl delete service` on a service configured with type `LoadBalancer`?
- A. A Cloud Storage bucket
- B. An external cloud load balancer and its associated forwarding rules [8]
- C. The GKE control plane master nodes
- D. The entire VPC network subnet
- **Correct Answer:** B
- **Explanation:** Deleting a `LoadBalancer` service triggers the cleanup and removal of the external cloud load balancer provisioned by Google Cloud [8].
- **Incorrect Options:** A, C, and D are unaffected by deleting a Kubernetes service.
- **Exam Takeaway:** Deleting a LoadBalancer service removes the corresponding external cloud load balancer [8].

#### Q3: Which command is used to completely delete a GKE cluster named `my-cluster` from the command line?
- A. `kubectl delete cluster my-cluster`
- B. `gcloud container clusters delete my-cluster` [8]
- C. `gcloud compute instances delete my-cluster`
- D. `kubectl destroy cluster my-cluster`
- **Correct Answer:** B
- **Explanation:** GKE cluster infrastructure is managed via `gcloud`, using the command `gcloud container clusters delete` [8].
- **Incorrect Options:** A uses invalid kubectl syntax. C targets raw compute instances, not GKE container clusters. D uses `destroy`.
- **Exam Takeaway:** Use `gcloud container clusters delete` to remove GKE clusters [8].

#### Q4: Why is it recommended to delete the Kubernetes service before deleting the deployment during application cleanup?
- A. To ensure that external load balancers and networking rules are properly torn down before pods disappear [8].
- B. Because Kubernetes prevents you from deleting deployments if a service exists.
- C. To prevent the Google Cloud billing account from being deleted.
- D. Because deployments cannot be deleted while pods are running.
- **Correct Answer:** A
- **Explanation:** Deleting the service first ensures clean teardown of external cloud load-balancing infrastructure tied to the app [8].
- **Incorrect Options:** B, C, and D state false operational constraints.
- **Exam Takeaway:** Clean up services and load balancers before removing backing deployments [8].

#### Q5: After running `kubectl delete deployment hello-world-rest-api`, how long does it typically take for the deployment and its associated pods to be removed?
- A. Several hours
- B. It requires a manual server reboot.
- C. Almost instantly [8]
- D. Exactly 24 hours due to cache locks
- **Correct Answer:** C
- **Explanation:** Deployment and pod deletion operations execute almost instantly in Kubernetes [8].
- **Incorrect Options:** A, B, and D are incorrect.
- **Exam Takeaway:** Pod and deployment deletions happen very quickly [8].

#### Q6: If an administrator wants to verify that all deployments have been successfully removed from a cluster, which command should they run?
- A. `kubectl get deployment` [8]
- B. `gcloud compute deployments list`
- C. `kubectl describe cluster`
- D. `gsutil ls`
- **Correct Answer:** A
- **Explanation:** `kubectl get deployment` lists active deployments, confirming if any remain after a cleanup [8].
- **Incorrect Options:** B, C, and D are incorrect command paths.
- **Exam Takeaway:** Use `kubectl get deployment` to verify deployment states [8].

#### Q7: True or False: You can delete a GKE cluster directly from the Google Cloud Console by navigating to Kubernetes Engine, selecting the cluster, and clicking Delete [8].
- A. True [8]
- B. False
- **Correct Answer:** A
- **Explanation:** Clusters can be deleted through both the `gcloud` CLI and the Google Cloud Console interface [8].
- **Exam Takeaway:** Clusters can be deleted via the Cloud Console [8].

#### Q8: What happens to your ongoing GKE Autopilot cluster billing if you delete all deployments and services but leave the cluster running?
- A. Billing drops to zero immediately.
- B. You continue to be billed for cluster management fees and any active underlying resource allocations [8].
- C. Google automatically refunds all previous charges.
- D. The cluster converts into a free-tier permanent instance.
- **Correct Answer:** B
- **Explanation:** While Autopilot bills based on pod resource requests (which become zero when deployments are deleted), control plane management fees or baseline infrastructure provisions may still apply unless the cluster itself is deleted.
- **Incorrect Options:** A, C, and D misrepresent cloud billing rules.
- **Exam Takeaway:** Deleting workloads stops pod resource billing, but deleting the cluster stops all cluster-associated infrastructure costs.

#### Q9: Which tool (`gcloud` or `kubectl`) must be used to delete a GKE cluster resource?
- A. `kubectl`
- B. `gcloud` [8]
- C. `docker`
- D. `terraform` only
- **Correct Answer:** B
- **Explanation:** `gcloud` is the correct CLI tool for managing GKE cluster lifecycles [8].
- **Incorrect Options:** A manages internal cluster objects. C builds images. D is an IaC tool (though valid, gcloud is the primary CLI).
- **Exam Takeaway:** Use `gcloud` for GKE cluster lifecycle management [8].

#### Q10: What confirmation step does the Google Cloud Console require when you click "Delete" on an active GKE cluster?
- A. Entering your personal credit card security pin code.
- B. Typing the exact name of the cluster to confirm the destructive action.
- C. Uploading an SSL certificate.
- D. Waiting for 48 hours for administrative approval.
- **Correct Answer:** B
- **Explanation:** GCP consoles require typing the exact resource name to prevent accidental deletion of production clusters.
- **Incorrect Options:** A, C, and D are false.
- **Exam Takeaway:** Deleting resources in the Cloud Console requires typing the resource name for confirmation.