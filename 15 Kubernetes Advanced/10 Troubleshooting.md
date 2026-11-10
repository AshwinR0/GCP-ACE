### 1. Overview
This topic explores **Kubernetes troubleshooting techniques and common deployment errors** encountered when working with GKE clusters. 
* **The Problem It Solves:** Helps administrators diagnose why applications fail to deploy, run, or scale up properly inside a cluster.
* **Primary Use Cases:** Resolving image pulling errors (`ImagePullBackOff`), debugging crashing application runtimes (`CrashLoopBackOff`), and resolving scheduling bottlenecks (`PodUnschedulable`).

---

### 2. ACE Exam Focus & Common Exam Traps
* **Exam Importance:** **HIGH**
* **High-Priority Troubleshooting Commands & Errors:**
  * **`ImagePullBackOff` / `ErrImagePull`:** Occurs when Kubernetes cannot pull the container image. 
    * *Fix/Check:* Verify the image name, registry path, and tag. Ensure the node's service account has proper permissions to pull from the registry (e.g., Artifact Registry / Docker Hub).
  * **`CrashLoopBackOff`:** Occurs when a container starts up, crashes immediately due to an internal error, and Kubernetes repeatedly restarts it.
    * *Fix/Check:* Inspect container logs using `kubectl logs <pod-name>` or execute commands inside the pod using `kubectl exec -it <pod-name> -- <command>`.
  * **`PodUnschedulable`:** Occurs when the scheduler cannot find a node with enough resources to host the pod.
    * *Fix/Check:* Check cluster resource availability, node scaling, or adjust pod CPU/memory requests and limits.
  * **`kubectl describe <object> <name>`:** The golden troubleshooting command that reveals detailed resource states and recent **events**.
* **Keywords for Scenario Questions:** *"ImagePullBackOff"*, *"CrashLoopBackOff"*, *"PodUnschedulable"*, *"Check container logs"*, *"Describe pod for events"*, *"Insufficient resources"*, *"Node service account permissions"*.
* **Common Exam Traps:**
  * Using `kubectl logs` for image pulling errors (logs won't help if the container never started because the image couldn't be pulled; use `kubectl describe pod` instead to see image-pull failure events).
  * Forgetting that GKE Autopilot clusters can take time (e.g., up to 10 minutes) to provision new underlying nodes during initial scale-up.

---

### 3. Detailed Concept Explanation

#### A. ImagePullBackOff / ErrImagePull
* **Cause:** The container runtime cannot pull the requested container image from the registry. This is usually due to a misspelled image tag, a non-existent version (e.g., `0.0.10.RELEASE` when only `0.0.1` exists), or permission restrictions.
* **Troubleshooting Steps:**
  1. Run `kubectl get pods` to see the status.
  2. Run `kubectl describe pod <pod-name>` to check the specific error events (e.g., repository unauthorized or manifest unknown).
  3. Verify the image name, tag, and verify that the worker node's service account has permissions to access the registry (such as Google Artifact Registry).

#### B. CrashLoopBackOff
* **Cause:** The container starts successfully, but the application throws an unhandled exception, crashes immediately, or exits with a non-zero exit code. Kubernetes attempts self-healing by restarting it repeatedly, entering a back-off loop.
* **Troubleshooting Steps:**
  1. Check application logs: `kubectl logs <pod-name>`.
  2. Execute interactive commands inside the container for debugging: `kubectl exec -it <pod-name> -- <command>`.

#### C. PodUnschedulable
* **Cause:** The Kubernetes scheduler cannot place a pod onto any existing worker node because of strict hardware constraints or lack of available compute resources (CPU/Memory).
* **Troubleshooting Steps:**
  1. Check cluster resource availability and node capacity.
  2. Review pod resource requests and limits; lower excessive memory or CPU requests if necessary, or add more worker nodes/node pools.

#### D. The Golden Troubleshooting Command: `kubectl describe`
* Whenever an object (Pod, Deployment, ReplicaSet, Service) behaves unexpectedly, running `kubectl describe <object-type> <object-name>` prints out metadata, current state details, and a chronological history of **Events**. The Events section is typically where the exact root cause of a failure is logged.

---

### 4. Practical Setup & Implementation Guide

#### Prerequisites
* A running Kubernetes cluster (Standard or Autopilot) and access via Google Cloud Shell with `kubectl` configured.

#### Step-by-Step Troubleshooting Workflow

1. **Simulate an Image Pull Error:**
   ```bash
   kubectl create deployment hello-world-rest-api --image=in28min/hello-world-rest-api:0.0.10.RELEASE
   ```
   * *Expected Outcome:* The deployment is created, but pods fail to launch due to a non-existent image tag.

2. **Identify the Failing Pod Status:**
   ```bash
   kubectl get pods
   ```
   * *Expected Outcome:* Pod status displays **`ImagePullBackOff`** or `ErrImagePull`. *(Note: On Autopilot clusters, initial node provisioning can take several minutes if scaling up from zero).*

3. **Inspect Detailed Pod Events:**
   ```bash
   kubectl describe pod <pod-name>
   ```
   * *Expected Outcome:* Output reveals detailed configuration specs and an **Events** section showing errors like `Failed to pull image "..."`.

4. **Troubleshoot a Crashing Pod (`CrashLoopBackOff`):**
   ```bash
   # View logs of a crashing pod
   kubectl logs <pod-name>
   
   # Execute a command inside the container to investigate
   kubectl exec -it <pod-name> -- /bin/sh
   ```

5. **Verify Cluster Resource Bottlenecks (`PodUnschedulable`):**
   * If pods show unschedulable states, review node capacity or describe the pod to see scheduler warnings regarding insufficient CPU or memory.

---

### 5. Visual Representations

#### Troubleshooting Error Decision Flow
```text
[ Pod Fails to Start ]
       |
       +---> Status: ImagePullBackOff / ErrImagePull?
       |        |---> Cause: Wrong image tag or registry permission issue.
       |        |---> Action: Check tag name & node service account permissions via `kubectl describe pod`.
       |
       +---> Status: CrashLoopBackOff?
       |        |---> Cause: Application crashes repeatedly upon startup.
       |        |---> Action: Check logs via `kubectl logs <pod-name>` or execute commands via `kubectl exec`.
       |
       +---> Status: PodUnschedulable?
                |---> Cause: Insufficient CPU/Memory or cluster resource constraints.
                |---> Action: Check cluster node capacity and adjust pod resource requests/limits.
```

---

### 6. Mental Models & Analogies

* **The Delivery Package Analogy:**
  * **ImagePullBackOff:** Ordering a custom part from a catalog, but giving the shipping company a catalog item number that was never printed (`0.0.10`). The delivery driver returns to the warehouse empty-handed.
  * **CrashLoopBackOff:** The package arrives, you open it and plug the appliance in, but it immediately sparks and blows a fuse. You unplug it, reset it, plug it back in, and it blows the fuse again in an endless loop.
  * **PodUnschedulable:** A massive delivery truck arrives at a small driveway, but the garage is completely full, so the truck cannot park anywhere.

---

### 7. Step-by-Step Workflows / Comparison Tables

#### Kubernetes Deployment Error Quick Reference

| Error Status | Primary Meaning | Recommended First Diagnostic Command |
| :--- | :--- | :--- |
| **ImagePullBackOff** | Container image tag doesn't exist or registry permissions failed. | `kubectl describe pod <pod-name>` (look at Events). |
| **CrashLoopBackOff** | Application starts, throws an unhandled error, crashes, and restarts in a loop. | `kubectl logs <pod-name>` |
| **PodUnschedulable** | Scheduler cannot find a node with enough capacity or matching constraints. | `kubectl describe pod <pod-name>` (look for scheduler warnings). |

---

### 8. Common Mistakes & Exam Traps
* **Trap:** Running `kubectl logs` when troubleshooting an `ImagePullBackOff` error.
  * *Correction:* `kubectl logs` reads output from running application containers. If the container hasn't started because the image can't be pulled, logs won't exist. You must use `kubectl describe pod` to view image pull error events.
* **Trap:** Assuming Autopilot clusters scale up instantly within seconds.
  * *Correction:* Scaling up an Autopilot cluster when new nodes need to be provisioned from scratch can take several minutes (up to 10 minutes in some cases).

---

### 9. Certification Practice Questions

#### Question 1
An administrator deploys an application to GKE, but after a few minutes, `kubectl get pods` shows a status of `ImagePullBackOff`. What is the most likely root cause of this error?
* A) The application code contains a NullPointerException inside the main Java class.
* B) The container image tag specified in the deployment configuration does not exist in the registry.
* C) The worker node has run out of physical RAM capacity.
* D) The Kubernetes Service object has an incorrect port configuration.

> **Correct Answer:** B
> **Explanation:** `ImagePullBackOff` occurs when the container runtime cannot find or pull the specified image, typically due to a typo in the image name or a non-existent tag.
> **Incorrect Options:** Option A leads to a `CrashLoopBackOff`. Option C leads to `PodUnschedulable` or `OOMKilled`. Option D breaks networking, not image pulling.
> **Exam Takeaway:** `ImagePullBackOff` = Image tag or registry access problem.

#### Question 2
When troubleshooting a pod that fails to pull its container image, which command provides the most useful details and chronological error events?
* A) `kubectl logs <pod-name>`
* B) `kubectl describe pod <pod-name>`
* C) `kubectl top pod <pod-name>`
* D) `kubectl exec -it <pod-name> -- /bin/sh`

> **Correct Answer:** B
> **Explanation:** `kubectl describe pod` displays full configuration metadata and a detailed **Events** log showing why an image pull failed.
> **Incorrect Options:** Option A fails because logs are generated by running containers. Option C displays resource usage. Option D fails because you cannot exec into a container that has never successfully started.
> **Exam Takeaway:** Use `kubectl describe pod` to review failure events.

#### Question 3
An application container starts up successfully but immediately encounters an unhandled runtime exception and exits with a non-zero code. Kubernetes attempts to restart it repeatedly. What pod status will you observe?
* A) ImagePullBackOff
* B) CrashLoopBackOff
* C) PodUnschedulable
* D) Succeeded

> **Correct Answer:** B
> **Explanation:** `CrashLoopBackOff` indicates that a container starts, crashes, and restarts in an endless back-off loop due to application startup errors.
> **Incorrect Options:** Option A is for image pull errors. Option C is for scheduling bottlenecks. Option D is for successful batch completion.
> **Exam Takeaway:** Crashing application loops result in `CrashLoopBackOff`.

#### Question 4
Which command should you execute to inspect the standard output and error logs of a running or crashing container inside a pod?
* A) `kubectl describe pod <pod-name>`
* B) `kubectl logs <pod-name>`
* C) `kubectl get pods -o wide`
* D) `kubectl get events`

> **Correct Answer:** B
> **Explanation:** `kubectl logs` retrieves the stdout/stderr logs produced by container processes inside a pod.
> **Incorrect Options:** Option A shows metadata and events. Option C shows nodes and image details. Option D shows cluster-wide events, not specific container output.
> **Exam Takeaway:** Use `kubectl logs` to view container application logs.

#### Question 5
A newly created pod remains in the `Pending` state, and `kubectl describe pod` reveals that the scheduler cannot find a node with sufficient CPU resources. What error or condition does this represent?
* A) ImagePullBackOff
* B) CrashLoopBackOff
* C) PodUnschedulable
* D) ErrImagePull

> **Correct Answer:** C
> **Explanation:** `PodUnschedulable` occurs when cluster nodes lack the necessary resources (CPU/Memory) or constraints to host the pending pod.
> **Incorrect Options:** Options A, B, and D relate to image pulling and container crashes.
> **Exam Takeaway:** Resource bottlenecks result in unschedulable pods.

#### Question 6
If a container image is stored in a private Google Artifact Registry, what is another crucial thing to verify when troubleshooting an `ImagePullBackOff` error, aside from the image tag name?
* A) Whether the local developer's laptop has an active internet connection.
* B) Whether the worker node's service account has proper permissions to pull images from Artifact Registry.
* C) Whether the Kubernetes API server has sufficient disk space.
* D) Whether the cluster has an external HTTP load balancer configured.

> **Correct Answer:** B
> **Explanation:** Worker nodes require proper IAM service account permissions to pull images from private registries like Google Artifact Registry.
> **Incorrect Options:** Options A, C, and D do not directly cause private registry image pull failures.
> **Exam Takeaway:** Private registry image pulls require correct node service account permissions.

#### Question 7
True or False: You can run interactive commands inside a running containerized application pod using the `kubectl exec` command for diagnostic purposes.
* A) True
* B) False

> **Correct Answer:** A
> **Explanation:** True. `kubectl exec -it <pod-name> -- <command>` allows administrators to run interactive shell commands directly inside a running container.
> **Incorrect Options:** Option B is incorrect because interactive debugging via `kubectl exec` is a standard Kubernetes troubleshooting technique.
> **Exam Takeaway:** Use `kubectl exec` to run interactive commands inside running pods.

#### Question 8
What is the primary role of the **Events** section found at the bottom of a `kubectl describe` command output?
* A) It stores historical billing data for Google Cloud compute resources.
* B) It lists chronological warnings and status transitions (such as failed scheduling or image pull errors) for that resource.
* C) It tracks every user password change made in the Google Cloud Console.
* D) It compiles container source code into binary executables.

> **Correct Answer:** B
> **Explanation:** The Events section records a chronological history of significant occurrences, status changes, and warnings for a Kubernetes object.
> **Incorrect Options:** Options A, C, and D are unrelated to Kubernetes object events.
> **Exam Takeaway:** Always check the Events section in `kubectl describe` outputs during troubleshooting.

#### Question 9
During an initial deployment scale-up on a GKE Autopilot cluster, you notice that pod creation takes a noticeable amount of time (e.g., several minutes). What is the typical reason for this delay?
* A) The Google Cloud billing console is processing payment authorization.
* B) The cluster may need to provision new underlying worker nodes from scratch to accommodate the workload.
* C) Kubernetes automatically deletes the deployment every 5 minutes for security audits.
* D) The API server is compressing logs.

> **Correct Answer:** B
> **Explanation:** Autopilot and standard clusters may take time to provision brand-new worker nodes if existing node capacity is exhausted, causing initial pod creation delays.
> **Incorrect Options:** Options A, C, and D are false.
> **Exam Takeaway:** Node provisioning time can introduce delays during initial cluster scale-up.

#### Question 10
If an application container repeatedly crashes because it requests more memory than the node's physical limit allows, what error state might you encounter?
* A) ImagePullBackOff
* B) OOMKilled (Out Of Memory Killed) / CrashLoopBackOff
* C) Succeeded
* D) Running normally without interruption

> **Correct Answer:** B
> **Explanation:** Exceeding memory limits triggers the kernel to terminate the container (`OOMKilled`), which subsequently results in a `CrashLoopBackOff` as Kubernetes tries to restart it.
> **Incorrect Options:** Options A, C, and D do not match memory limit exhaustion behavior.
> **Exam Takeaway:** Memory limit breaches lead to `OOMKilled` and `CrashLoopBackOff` loops.