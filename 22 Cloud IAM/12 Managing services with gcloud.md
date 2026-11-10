# GCP Associate Cloud Engineer (ACE) Study Notes

## Chapter: Managing Services via `gcloud` (Enabling, Disabling, and Listing APIs)

---

### 1. Overview
In Google Cloud, individual APIs and services (such as Cloud Spanner, Cloud SQL, Compute Engine) must be explicitly enabled before you can create or use resources tied to them within a project. 

The `gcloud services` command group allows cloud engineers to programmatically manage these APIs [USER TEXT]. It provides functionality to audit enabled services, explore all available platform services, and turn specific APIs on or off as requirements change.

---

### 2. ACE Exam Focus & Common Exam Traps
- **Exam Importance:** **HIGH**. Scenario-based questions frequently check troubleshooting workflows when a user encounters a "Permission Denied" or "Service Not Enabled" error when attempting to deploy a resource (e.g., creating a VM or a database instance).
- **High-Priority Commands:**
  - **List Enabled Services:** `gcloud services list --enabled --project=[PROJECT_ID]` [USER TEXT]
  - **List Available Services:** `gcloud services list --available --project=[PROJECT_ID]` [USER TEXT]
  - **Enable a Service:** `gcloud services enable [SERVICE_NAME] --project=[PROJECT_ID]` [USER TEXT]
  - **Disable a Service:** `gcloud services disable [SERVICE_NAME] --project=[PROJECT_ID]` [USER TEXT]
- **Common Exam Traps:**
  - Confusing `gcloud services list` (which outputs both enabled and available services by default) with filtered flags like `--enabled` [USER TEXT].
  - Forgetting that running `gcloud services list --available` outputs a massive list of every GCP service available across the entire platform, which can take a long time to render [USER TEXT].

---

### 3. Detailed Concept Explanation

#### A. The `gcloud services` Command Group
The `gcloud services` command structure governs API management inside your active Google Cloud project. Every service is identified by a unique service name ending in `.googleapis.com` (e.g., `compute.googleapis.com` for Compute Engine) [USER TEXT].

#### B. Listing Services (`gcloud services list`)
- **Default Behavior:** Running `gcloud services list` without filters returns a comprehensive list of both currently enabled services and all available services [USER TEXT].
- **Filtering Enabled Services Only:** Use the `--enabled` flag to narrow down output strictly to APIs currently active and running in the project [USER TEXT]:
  ```bash
  gcloud services list --enabled --project=[PROJECT_ID]
  ```
- **Auditing Available Services:** Use the `--available` flag to query every service that *can* potentially be enabled within the project environment [USER TEXT]:
  ```bash
  gcloud services list --available --project=[PROJECT_ID]
  ```
  *(Note: Because Google Cloud offers hundreds of services, this command generates a massive output list and takes longer to execute) [USER TEXT].*

#### C. Enabling and Disabling Services (`gcloud services enable` / `disable`)
- **Enabling an API:** When deploying a new infrastructure component, you often need to turn on its underlying service first [USER TEXT]:
  ```bash
  gcloud services enable compute.googleapis.com --project=[PROJECT_ID]
  ```
- **Disabling an API:** To shut down an unused service and strip API accessibility from a project, use the disable command [USER TEXT]:
  ```bash
  gcloud services disable sqladmin.googleapis.com --project=[PROJECT_ID]
  ```

---

### 4. Practical Setup & Implementation Guide

#### Step-by-Step API Management via Cloud Shell
1. **List currently enabled services for a project:**
   ```bash
   gcloud services list --enabled --project=my-sample-project-id
   ```
   *Expected Outcome:* Returns a concise list showing active APIs like Spanner, Cloud SQL, and Telemetry [USER TEXT].
2. **Explore all available services:**
   ```bash
   gcloud services list --available --project=my-sample-project-id
   ```
   *Expected Outcome:* Streams a large catalog of every Google Cloud service available for activation [USER TEXT]. *(Press `Ctrl + C` or `Cmd + C` to abort execution if needed) [USER TEXT].*
3. **Enable a specific service (e.g., Compute Engine):**
   ```bash
   gcloud services enable compute.googleapis.com --project=my-sample-project-id
   ```
   *Expected Outcome:* Provisions and activates the Compute Engine API for the target project [USER TEXT].
4. **Disable a specific service:**
   ```bash
   gcloud services disable compute.googleapis.com --project=my-sample-project-id
   ```
   *Expected Outcome:* Deactivates the API endpoint for the project [USER TEXT].

---

### 5. Visual Representations

#### Service State Management Flow
```text
  +-------------------------------------------------------------+
  |                   `gcloud services list`                    |
  +-------------------------------------------------------------+
         |                                             |
         v                                             v
  [ --enabled flag ]                            [ --available flag ]
  Returns only active APIs                      Returns ALL platform APIs
  (e.g., Spanner, SQL)                          (Massive catalog list)
         |                                             |
         +--------------------------+------------------+
                                    |
                    +---------------+---------------+
                    |                               |
                    v                               v
         [ `gcloud services enable` ]    [ `gcloud services disable` ]
           Turns API ON for project        Turns API OFF for project
```

---

### 6. Mental Models & Analogies

#### The Utility Meter Analogy
- **Available Services (`--available`):** Like the utility company's master catalog listing every possible service hookup available in the city (gas, water, high-speed fiber, electricity variants).
- **Enabled Services (`--enabled`):** Like checking your house meter panel to see which specific utilities are currently turned on and active inside your home.
- **Enable / Disable Commands:** Like calling the utility provider to flip the circuit breaker or water valve on (`enable`) or off (`disable`) for a specific service line.

---

### 7. Step-by-Step Workflows / Comparison Tables

#### Summary Table: `gcloud services` Commands
| Task | `gcloud` Command | Description |
| :--- | :--- | :--- |
| **List Enabled** | `gcloud services list --enabled` | Displays only currently active APIs in the project [USER TEXT]. |
| **List Available** | `gcloud services list --available` | Outputs all services available across Google Cloud [USER TEXT]. |
| **Enable Service** | `gcloud services enable [SERVICE_ID]` | Turns on a specific API endpoint for the project [USER TEXT]. |
| **Disable Service** | `gcloud services disable [SERVICE_ID]` | Turns off and deactivates a specific API [USER TEXT]. |

---

### 8. Common Mistakes & Exam Traps
- **Trap:** Forgetting that `gcloud services list` without any flag returns *both* enabled and available services [USER TEXT].
- **Trap:** Assuming that creating a project automatically enables every single GCP service out of the box.
  *Exam Tip:* If an exam scenario describes a user getting an error when trying to provision a resource like a Cloud SQL database or a Kubernetes cluster, the root cause is often that the corresponding API (`sqladmin.googleapis.com` or `container.googleapis.com`) has not been enabled yet.

---

### 9. Certification Practice Questions

#### Question 1
An engineer attempts to provision a new Compute Engine virtual machine inside a newly created Google Cloud project via the command line and receives an error stating that the Compute Engine service is not enabled. Which command should they run to fix this issue?
- A. `gcloud compute instances start`
- B. `gcloud services enable compute.googleapis.com` [USER TEXT]
- C. `gcloud projects activate-service`
- D. `gcloud config set compute/enabled true`

**Correct Answer:** **B**  
**Explanation:** To resolve service-disabled errors, you must use `gcloud services enable` followed by the specific service endpoint name (e.g., `compute.googleapis.com`) [USER TEXT].  
**Incorrect Answers:** A attempts to start an instance that cannot be created yet. C and D are invalid command structures.  
**Exam Takeaway:** Use `gcloud services enable` to turn on inactive APIs.

---

#### Question 2
What is the primary difference in output when running `gcloud services list --enabled` versus `gcloud services list --available`?
- A. `--enabled` shows active APIs in your project [USER TEXT], while `--available` outputs a massive catalog of every GCP service that can potentially be enabled [USER TEXT].
- B. `--enabled` lists human users, while `--available` lists service accounts.
- C. `--available` is faster to execute than `--enabled`.
- D. There is no difference; both commands return identical JSON data.

**Correct Answer:** **A**  
**Explanation:** `--enabled` filters the results strictly to APIs currently active in the project [USER TEXT], whereas `--available` queries the entire platform catalog of eligible services [USER TEXT].  
**Incorrect Answers:** B, C, and D are false statements.  
**Exam Takeaway:** Understand the filtering scope difference between `--enabled` and `--available` [USER TEXT].

---

#### Question 3
If you execute the command `gcloud services list` without specifying any filter flags, what does Google Cloud return?
- A. Only disabled services.
- B. Both enabled services and available services [USER TEXT].
- C. An error requiring a flag selection.
- D. Active billing ledgers.

**Correct Answer:** **B**  
**Explanation:** Running `gcloud services list` with no flags lists both enabled services and available services by default [USER TEXT].  
**Incorrect Answers:** A, C, and D are incorrect.  
**Exam Takeaway:** Default list execution includes both enabled and available services [USER TEXT].

---

#### Question 4
Which command should you execute if you want to deactivate and turn off the Cloud SQL Admin API for a specific project?
- A. `gcloud services disable sqladmin.googleapis.com` [USER TEXT]
- B. `gcloud sql database delete`
- C. `gcloud services remove sql-admin`
- D. `gcloud projects block-api`

**Correct Answer:** **A**  
**Explanation:** `gcloud services disable` followed by the service name shuts down the specified API endpoint [USER TEXT].  
**Incorrect Answers:** B deletes databases, not the API service itself. C and D are invalid syntax.  
**Exam Takeaway:** Use `gcloud services disable` to turn off an active API [USER TEXT].

---

#### Question 5
What suffix is universally appended to Google Cloud service identifiers used in `gcloud services` commands?
- A. `.googlecloud.com`
- B. `.googleapis.com`
- C. `.gserviceaccount.com`
- D. `.internal.net`

**Correct Answer:** **B**  
**Explanation:** GCP API service names use the `.googleapis.com` suffix (e.g., `compute.googleapis.com`) [USER TEXT].  
**Incorrect Answers:** A, C, and D represent other domains (like service accounts).  
**Exam Takeaway:** GCP service endpoints end with `.googleapis.com`.

---

#### Question 6
Why might the execution of `gcloud services list --available` take a noticeably long time to render output in your terminal?
- A. Because it runs deep cryptographic checks on all database backups.
- B. Because there are a large number of available services across Google Cloud that must be compiled into a massive list [USER TEXT].
- C. Because it requires manual password authentication for every available service.
- D. Because it checks billing account balances across all regions.

**Correct Answer:** **B**  
**Explanation:** Querying all available platform services compiles a very large list, which causes execution delays [USER TEXT].  
**Incorrect Answers:** A, C, and D are incorrect.  
**Exam Takeaway:** Expect execution delays when listing all available services due to the sheer volume of offerings [USER TEXT].

---

#### Question 7
How can you gracefully abort or stop the execution of a long-running `gcloud` command like `gcloud services list --available` in your terminal session?
- A. Press `Ctrl + C` or `Command + C` [USER TEXT]
- B. Close your browser window without saving.
- C. Run `gcloud abort-process`.
- D. Type `exit` and restart the computer.

**Correct Answer:** **A**  
**Explanation:** Standard keyboard interrupt shortcuts (`Ctrl + C` or `Command + C`) stop active CLI process execution [USER TEXT].  
**Incorrect Answers:** B, C, and D are either overly disruptive or invalid.  
**Exam Takeaway:** Use `Ctrl + C` / `Command + C` to interrupt running CLI processes [USER TEXT].

---

#### Question 8
If an administrator wants to verify active service APIs specifically for a project whose ID is different from their current default configuration, which flag should they add to the command?
- A. `--project=[PROJECT_ID]` [USER TEXT]
- B. `--target-id=[PROJECT_ID]`
- C. `--scope=[PROJECT_ID]`
- D. `--billing-project=[PROJECT_ID]`

**Correct Answer:** **A**  
**Explanation:** The `--project` flag overrides the default configuration and targets a specific project ID [USER TEXT].  
**Incorrect Answers:** B, C, and D are incorrect flag names.  
**Exam Takeaway:** Use `--project=[PROJECT_ID]` to specify target projects across `gcloud` commands [USER TEXT].

---

#### Question 9
True or False: Once a Google Cloud service has been disabled using `gcloud services disable`, all existing data associated with that service's resources inside the project is instantly and permanently deleted.
- A. True
- B. False

**Correct Answer:** **B**  
**Explanation:** False. Disabling an API stops programmatic API calls and management endpoints, but it does not automatically delete stored database or disk data immediately (though billing and access rules shift).  
**Exam Takeaway:** Disabling an API disables management endpoints, but data persistence behavior depends on the specific service.

---

#### Question 10
Which of the following tasks can be accomplished using the `gcloud services` command group?
- A. Listing enabled services, listing available services, enabling services, and disabling services [USER TEXT].
- B. Managing IAM user passwords and multi-factor authentication devices.
- C. Provisioning physical networking cables in Google data centers.
- D. Configuring organization billing credit cards.

**Correct Answer:** **A**  
**Explanation:** The `gcloud services` group manages service enablement, listing, and disabling [USER TEXT].  
**Incorrect Answers:** B, C, and D fall under IAM, physical hardware management, and Billing respectively.  
**Exam Takeaway:** `gcloud services` controls API enablement and auditing.