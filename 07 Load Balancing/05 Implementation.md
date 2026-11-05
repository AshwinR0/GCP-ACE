# GCP Study Notes: Google Cloud Load Balancing (Application Load Balancer & Instance Groups)

## 1. Overview
Load balancing in Google Cloud Platform (GCP) allows you to distribute incoming network or application traffic across multiple backend instances (such as Virtual Machines, Managed Instance Groups, containers, or serverless services). This ensures high availability, fault tolerance, and optimal resource utilization.

In this module, the focus is on creating a **Global External Application Load Balancer** backed by an **Instance Group** containing multiple web server VMs. The load balancer acts as a single point of entry for internet traffic, routing requests dynamically to healthy backend instances based on established rules, protocols, and health checks.

---

## 2. ACE Exam Focus & Common Exam Traps
* **Exam Importance:** **HIGH**. Load balancing is a core GCP networking topic frequently tested via real-world scenarios.
* **Key Concepts:**
  * **Frontend vs. Backend:** Frontend manages how users connect (IP, port, protocol, SSL certs), while Backend defines where the traffic goes (Instance Groups, GKE, Serverless).
  * **Global vs. Regional:** Global load balancers can route traffic to backends across multiple regions; regional load balancers stay within a single region.
  * **Health Checks:** Crucial for determining if a backend instance can receive traffic. Pay attention to terms like *check interval*, *timeout*, and *healthy/unhealthy thresholds*.
* **Frequently Confused Services:**
  * **External Application Load Balancer:** Layer 7 (HTTP/HTTPS), proxy-based, ideal for web applications.
  * **External Network Load Balancer:** Layer 4 (TCP/UDP/SSL), passthrough-based, used for high-performance non-HTTP workloads.
* **Exam Keywords:** *"Distribute incoming web traffic globally"*, *"HTTP/HTTPS workload"*, *"SSL termination"*, *"Path-based routing"*.

---

## 3. Detailed Concept Explanation

### A. Managed / Unmanaged Instance Groups for Load Balancing
* **Definition:** A collection of VM instances that can be managed as a single entity.
* **Purpose:** Acts as the backend target pool for the load balancer.
* **Prerequisites:** Requires an **Instance Template** (which defines machine type, OS image, startup scripts like Apache, etc.) to stamp out VMs consistently.

### B. Load Balancer Architecture Components
* **Frontend:**
  * Consists of an IP address (ephemeral or static), port (e.g., port 80 for HTTP, 443 for HTTPS), and protocol.
  * Handles client connections and optionally terminates SSL/TLS.
* **Backend Service:**
  * Defines how the load balancer distributes traffic to backends.
  * Supports various backend types: **Instance Groups**, **Google Kubernetes Engine (GKE)**, **Cloud Run**, **App Engine**, and **Cloud Storage buckets**.
  * Configures **backend balancing modes** such as *CPU utilization*, *request rate*, or *in-flight requests*.
* **Health Checks:**
  * Probes backend instances periodically to verify application responsiveness.
  * Parameters include check interval, timeout, and consecutive success/failure thresholds.
* **Routing Rules:**
  * Determines where to forward traffic based on hostnames or URL paths (e.g., routing `/images/*` to one backend and `/video/*` to another).

### C. Cloud CDN (Content Delivery Network)
* Integration option available with external HTTP(S) load balancing.
* Caches static content closer to users at edge locations globally to reduce latency and backend load.

---

## 4. Practical Setup & Implementation Guide

### Prerequisites
* Active GCP account with permissions to create Compute Engine resources and Load Balancers.
* An existing **Instance Template** configured with a web server (e.g., Apache HTTP server).

### Step-by-Step Implementation

#### Step 1: Create an Instance Group
1. Navigate to **Compute Engine** > **Instance groups** in the GCP Console.
2. Click **Create instance group**.
3. Name the instance group: `my-instance-group-for-lb`.
4. Select the pre-created **Instance template**.
5. Set the number of instances to **2**.
6. Accept remaining defaults and click **Create**.

#### Step 2: Create a Global External Application Load Balancer
1. Navigate to **Network services** > **Load balancing**.
2. Click **Create load balancer**.
3. Choose **Application Load Balancer (HTTP/HTTPS)** and select **Public facing (external)**, then click **Next**.
4. Choose **Global** deployment and select the latest generation **Global external application load balancer**, then click **Configure**.

#### Step 3: Configure Frontend
1. Name the frontend configuration: `my-global-http-lb-frontend`.
2. Protocol: **HTTP** (Port `80`).
3. IP version: **IPv4** (Ephemeral).
4. Click **Done**.

#### Step 4: Configure Backend
1. Click **Backend configuration** > **Backend services & backend buckets** > **Create a backend service**.
2. Name the backend service: `my-global-http-lb-backend`.
3. Backend type: **Instance group**.
4. Protocol: **HTTP**, Named port: `http`.
5. Under **Backends**, select the instance group (`my-instance-group-for-lb`) and set port mapping to `80`.
6. Select **Balancing mode** (e.g., **CPU utilization** at target `80%`).

#### Step 5: Configure Health Check
1. In the backend service creation panel, create a new **Health check**.
2. Name: `my-global-http-lb-health-check`.
3. Keep default check intervals, timeouts, and thresholds, then click **Create**.
4. Save the backend service.

#### Step 6: Configure Routing Rules & Review
1. Set simple host and path rules to direct all traffic to the default backend service.
2. Review configuration details and click **Create**.
3. *Note:* Load balancer creation can take 10 to 15 minutes to fully provision globally.

### Verification & Cleanup
* **Verification:** Retrieve the frontend IP address from the load balancer details page, open a browser using `http://<IP_Address>`, and verify that responses alternate between backend instances.
* **Cleanup:** 
  1. Delete the load balancer, associated backend services, and health checks.
  2. Navigate to Instance Groups and delete `my-instance-group-for-lb` to remove underlying VMs and prevent recurring charges.

---

## 5. Visual Representations

### Load Balancer Request Flow Architecture
```text
[ Internet Clients ]
        |
        v  (Sends request to Frontend IP:Port 80)
+-------------------------------------------------------+
|           Global External Application LB              |
|  +-------------------------------------------------+  |
|  | Frontend: HTTP / IPv4 / Port 80                 |  |
|  +-------------------------------------------------+  |
|  | Routing Rules: Default Route                    |  |
|  +-------------------------------------------------+  |
|  | Backend Service (Instance Group Target)         |  |
|  +-------------------------------------------------+  |
+-------------------------------------------------------+
        |                    |
        | (Distributes)      | (Distributes)
        v                    v
+------------------+  +------------------+
|  VM Instance 1   |  |  VM Instance 2   |
| (Apache Server)  |  | (Apache Server)  |
+------------------+  +------------------+
```

---

## 6. Mental Models & Analogies

* **The Load Balancer is like a Restaurant Host:** 
  * The **Frontend** is the front desk reception where guests (traffic) arrive.
  * The **Routing Rules & Backend Service** are the seating charts deciding which table (VM instance in the Instance Group) gets the next group of guests based on availability (CPU utilization / Health checks).

---

## 7. Step-by-Step Workflows / Comparison Tables

### Comparison Table: External Application vs. External Network Load Balancer

| Feature | External Application Load Balancer | External Network Load Balancer |
| :--- | :--- | :--- |
| **OSI Layer** | Layer 7 (HTTP/HTTPS) | Layer 4 (TCP/UDP/SSL) |
| **Proxy Architecture** | Reverse proxy (terminates connections) | Passthrough (preserves client source IP easily) |
| **Use Cases** | Web applications, URL path-based routing, SSL offloading | High-throughput gaming, non-HTTP/S TCP/UDP streams |
| **Backend Support** | Instance Groups, GKE, Cloud Run, Buckets | Instance Groups, GKE, Target Pools |

---

## 8. Common Mistakes & Exam Traps

* **Mistake:** Testing the load balancer immediately after creation and concluding it is broken because of HTTP 502/504 errors.
  * **Correction:** Global load balancers take 10–15 minutes to provision and propagate configurations across global edge points. Patience is required.
* **Exam Trap:** Choosing an Application Load Balancer for a custom non-HTTP TCP-based database replication protocol.
  * **Correction:** Layer 7 Application Load Balancers handle HTTP/HTTPS only. For raw TCP/UDP traffic, a Network Load Balancer must be selected.
* **Mistake:** Forgetting to configure a "Named Port" on the Instance Group matching the port defined in the backend service.
  * **Correction:** The backend service expects a named port to know which port the application listens to on the instance group VMs.

---

## 9. Certification Practice Questions

### Question 1
An e-commerce company is hosting a web application on Google Cloud using a managed instance group of Compute Engine VMs. They require a load balancing solution that can inspect incoming uniform resource identifiers (URIs) and route traffic to different backend pools based on whether the user is accessing images or video content. Which load balancing service should they choose?

* A. External TCP/UDP Network Load Balancer
* B. External Application Load Balancer
* C. Internal TCP/UDP Load Balancer
* D. Internal Application Load Balancer

**Correct Answer:** B  
**Explanation:** Path-based routing (inspecting URIs/paths like `/images` or `/video`) is a Layer 7 capability exclusive to Application Load Balancers. Since users are coming from the internet, an External Application Load Balancer is required.  
**Incorrect Options:**  
* *A & C* are Layer 4 network load balancers which operate on IP protocols and ports, not HTTP paths/URIs.  
* *D* is an internal load balancer meant for private, intra-VPC traffic rather than public internet traffic.  
* **Exam Takeaway:** Keywords like *"URIs"*, *"paths"*, and *"HTTP/HTTPS"* point directly to Application Load Balancers.

---

### Question 2
Your team has configured a new Global External Application Load Balancer pointing to an Instance Group backend. Immediately after clicking "Create", you test the frontend ephemeral IP address in your browser and receive an error message. What is the most likely reason, and what action should you take?

* A. The firewall blocks port 80; you must delete the load balancer and create it again with HTTPS.
* B. The load balancer configuration is invalid; you must immediately reconfigure the backend service timeout.
* C. Global load balancers take several minutes to fully provision and deploy rules globally; you should wait 10-15 minutes and re-test.
* D. Instance groups cannot serve as backends for global application load balancers; you must migrate to Cloud Storage buckets.

**Correct Answer:** C  
**Explanation:** Global HTTP(S) load balancers take roughly 10 to 15 minutes to fully deploy and propagate infrastructure changes across Google's edge network.  
* **Incorrect Options:** A, B, and D describe false conditions or unnecessary actions. Instance groups are fully supported backends.  
* **Exam Takeaway:** Expect provisioning delays with global network/load balancing infrastructure in real-world scenarios and exam items.

---

### Question 3
An application running on Compute Engine instances behind an External Application Load Balancer is experiencing intermittent downtime. You need to ensure that the load balancer stops routing traffic to instances that are unresponsive or throwing application errors. What component should you configure?

* A. Cloud CDN caching rules
* B. A custom VPC firewall rule for port 22
* C. A health check
* D. An SSL certificate policy

**Correct Answer:** C  
**Explanation:** Health checks actively probe backend instances at regular intervals to determine if they are healthy and able to accept traffic. Unhealthy instances are automatically removed from rotation.  
* **Incorrect Options:**  
* *A* optimizes static asset delivery, not instance health.  
* *B* deals with SSH administrative access.  
* *D* handles security encryption settings.  
* **Exam Takeaway:** For traffic steering based on instance availability, health checks are mandatory components.

---

### Question 4
You are designing a high-performance gaming application backend on GCP that uses raw UDP packets on port 5000. Which load balancer configuration fits this requirement?

* A. External Application Load Balancer
* B. Internal Application Load Balancer
* C. External Network Load Balancer
* D. Serverless Network Endpoint Group

**Correct Answer:** C  
* **Explanation:** Raw UDP traffic operates at Layer 4. The External Network Load Balancer natively supports TCP and UDP traffic pass-through.  
* **Incorrect Options:** Application Load Balancers (A and B) handle HTTP/HTTPS traffic only.  
* **Exam Takeaway:** Match Layer 4 protocols (TCP/UDP) with Network Load Balancers, and Layer 7 protocols (HTTP/HTTPS) with Application Load Balancers.

---

### Question 5
Which component of an Application Load Balancer is responsible for receiving incoming client requests on a designated IP address and port?

* A. Backend service
* B. Frontend configuration
* C. Instance template
* D. Health check probe

**Correct Answer:** B  
* **Explanation:** The frontend holds the IP address, port, and protocol specification where clients connect to the load balancer.  
* **Incorrect Options:** Backends handle distribution; templates configure VMs; health checks monitor health.  
* **Exam Takeaway:** Remember the split: Frontend = client entry point; Backend = workload destination.

---

### Question 6
Your architecture requires web traffic to remain entirely internal to your Virtual Private Cloud (VPC) network, serving internal microservices without exposure to the public internet. Which type of load balancer should you deploy?

* A. External Application Load Balancer
* B. External Network Load Balancer
* C. Internal Application Load Balancer
* D. Global SSL Proxy Load Balancer

**Correct Answer:** C  
* **Explanation:** Internal Application Load Balancers are designed for private, internal-only HTTP(S) traffic within a VPC.  
* **Incorrect Options:** Options A and B are public-facing/external.  
* **Exam Takeaway:** Look for words like *"internal only"*, *"private IP"*, or *"intra-VPC"* to choose internal load balancing variations.

---

### Question 7
When configuring a backend service for an Instance Group using an Application Load Balancer, which of the following balancing modes can be specified to optimize traffic distribution? (Select the most appropriate option discussed)

* A. CPU utilization
* B. Hard disk capacity
* C. RAM allocation size
* D. Billing cost optimization

**Correct Answer:** A  
* **Explanation:** Backend balancing modes can leverage metrics such as CPU utilization, request rate, or in-flight requests.  
* **Incorrect Options:** Disk capacity, RAM, and billing are not native load balancer balancing metrics.  
* **Exam Takeaway:** CPU utilization is a standard metric used for managing backend load distribution in GCP.

---

### Question 8
You want to improve application response times for static images distributed globally via your HTTP(S) load balancer by serving content from edge locations closest to your users. What feature should you enable?

* A. Cloud Logging
* B. Cloud CDN
* C. VPC Flow Logs
* D. Identity-Aware Proxy (IAP)

**Correct Answer:** B  
* **Explanation:** Cloud CDN caches content at Google's network edge locations, speeding up delivery and reducing load on backend instances.  
* **Incorrect Options:** Logging services diagnose issues; IAP manages access security.  
* **Exam Takeaway:** Edge caching and global content delivery translate to **Cloud CDN**.

---

### Question 9
What is the primary function of an Instance Template when setting up a backend Instance Group for a load balancer?

* A. It defines the routing paths for incoming HTTP requests.
* B. It acts as a blueprint defining machine type, boot disk, and startup scripts for the VMs in the group.
* C. It acts as a firewall rule to block unauthorized external IPs.
* D. It monitors backend health parameters like timeout and check intervals.

**Correct Answer:** B  
* **Explanation:** An instance template is an immutable blueprint used to create virtual machine instances consistently within instance groups.  
* **Incorrect Options:** Routing rules handle paths; health checks handle instance status monitoring.  
* **Exam Takeaway:** Instance templates are blueprints for VM creation.

---

### Question 10
After setting up your Global External Application Load Balancer and backend instance group, you notice that all instances are marked as "Unhealthy" by the health checker. What should you verify first?

* A. Check if firewall rules allow ingress traffic from GCP health checker IP ranges to the backend instances.
* B. Check if the project billing account is frozen.
* C. Verify that Cloud Storage logging is enabled.
* D. Verify that the frontend IP address is static instead of ephemeral.

**Correct Answer:** A  
* **Explanation:** GCP health checks probe instances from specific internal IP ranges. If VPC firewall rules block these ranges, instances will appear unhealthy.  
* **Incorrect Options:** Billing, logging, and IP ephemerality do not directly cause health check failures on operational web servers.  
* **Exam Takeaway:** Firewall rules blocking GCP health check IP probes are a primary root cause for unexpected "unhealthy" backend states.