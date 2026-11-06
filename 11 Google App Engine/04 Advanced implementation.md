### 1. Overview
This topic covers exploring App Engine dashboard statistics, deploying a new version (`v2`) of an existing service without downtime using the `--version` flag, and managing concurrent versions via **traffic splitting** (such as splitting traffic equally by IP address, cookie, or random distribution). It emphasizes zero-downtime deployments and version management in production systems.

---

### 2. ACE Exam Focus & Common Exam Traps
*   **Exam Importance**: **HIGH**. Version management, zero-downtime updates, and traffic-splitting behavior are heavily tested configuration concepts.
*   **Key Commands & Concepts to Memorize**:
    *   *Deploying a specific version*: `gcloud app deploy --version=v2` (or `-v v2`).
    *   *Default promotion behavior*: By default, `gcloud app deploy` automatically promotes the new version to receive 100% of the production traffic unless configured otherwise.
    *   *Traffic Splitting Options*: Traffic can be split across versions by **IP address**, **Cookie**, or **Random allocation**.
    *   *Zero Downtime*: While a new version is deploying, the older active version continues to serve incoming client requests seamlessly.
*   **Keywords for Scenario Questions**:
    *   *“Deploy a new version explicitly labeled v2 / canary test”* $\rightarrow$ `gcloud app deploy --version=v2`
    *   *“Zero-downtime release / update code without interrupting users”* $\rightarrow$ App Engine version deployment.
    *   *“Split traffic between versions by user cookie or IP address”* $\rightarrow$ App Engine Traffic Splitting console/CLI configuration.

---

### 3. Detailed Concept Explanation

#### A. Version Deployment Workflow
*   When you modify code (e.g., changing `v1` to `v2` in `main.py`) and deploy it using `gcloud app deploy --version=v2`, App Engine creates a parallel isolated environment for version `v2`.
*   **Zero-Downtime Guarantee**: While `v2` is building and initializing, the existing `v1` version remains fully live and serves production requests without interruption.
*   **Automatic Promotion**: By default, once `v2` deployment completes successfully, App Engine shifts 100% of the live production traffic from `v1` to `v2`.

#### B. Traffic Splitting
*   App Engine allows multiple versions of the same service to run concurrently.
*   **Traffic Allocation Settings**: Administrators can define granular percentage splits (e.g., 50% to `v1` and 50% to `v2`) through the GCP Console or CLI.
*   **Allocation Algorithms**:
    *   *IP Address*: Ensures users coming from the same IP address consistently hit the same version.
    *   *Cookie*: Uses a browser cookie to pin a user session to a specific version.
    *   *Random*: Distributes incoming requests randomly across versions per request.

---

### 4. Practical Setup & Implementation Guide

#### Prerequisites & Dependencies
*   An active Google Cloud project with App Engine initialized and a working initial version (`v1`) deployed.

#### Step-by-Step Implementation Instructions

##### Step 1: Modify Application Code for Version 2
1.  Open Cloud Shell and open the code editor.
2.  Open your application script (`main.py`).
3.  Modify the text or return string from `v1` to `v2` (e.g., change `My Default Service - V1` to `My Default Service - V2`).
4.  Save the file (`Ctrl + S` or `Cmd + S`).

##### Step 2: Deploy Version 2 via CLI
1.  Navigate to your project code directory in Cloud Shell:
    ```bash
    cd default-service
    ```
2.  Run the deployment command specifying the explicit version label `--version`:
    ```bash
    gcloud app deploy --version=v2
    ```
3.  Type `Y` (Yes) when prompted to confirm the deployment.
4.  *Observation*: While the deployment builds in the background, refresh your app URL in the browser to confirm that `v1` continues serving traffic with zero downtime.

##### Step 3: Access Specific Version URLs
1.  Open the Google Cloud Console and navigate to **App Engine $\rightarrow$ Services / Versions**.
2.  Observe that both `v1` and `v2` are listed under the default service.
3.  Click the individual version-specific URL link for `v1` to test the older code version directly, independent of the active traffic split.

##### Step 4: Configure Traffic Splitting
1.  In the App Engine dashboard, click the **Split Traffic** button.
2.  Choose your splitting criteria method: **IP address**, **Cookie**, or **Random**.
3.  Add version `v1` back into the traffic distribution list.
4.  Adjust the percentages so that both `v1` and `v2` receive **50%** each.
5.  Click **Save**.
6.  *Verification*: Refresh your main application URL multiple times; you will alternate between seeing the `v1` response and the `v2` response.

---

### 5. Visual Representations

#### App Engine Version Traffic Splitting Architecture
```
                     +----------------------------+
                     | Incoming User Traffic      |
                     +----------------------------+
                                   |
                                   v
                     +----------------------------+
                     | App Engine Load Balancer   |
                     | (Split by IP / Cookie /    |
                     |  Random Algorithm)         |
                     +----------------------------+
                        /                      \
         50% Traffic   /                        \  50% Traffic
                      v                          v
      +-----------------------------+  +-----------------------------+
      | Version 1 (v1)              |  | Version 2 (v2)              |
      | - Previous code deployment  |  | - Newly deployed code       |
      | - Active instance running   |  | - Active instance running   |
      +-----------------------------+  +-----------------------------+
```

---

### 6. Mental Models & Analogies

*   **Deploying a new version with traffic splitting is like a restaurant opening a secondary serving window while the primary kitchen is still running**: The kitchen cooks up a newly formulated recipe (`v2`), but instead of slamming the main door shut, management keeps the original window (`v1`) fully operational to feed current customers. Once the new menu is ready, management directs half the queue to the new window (`v2`) and half to the old window (`v1`) to compare customer reactions side-by-side.

---

### 7. Step-by-Step Workflows / Comparison Tables

#### Version Deployment & Traffic Management Workflow

| Workflow Phase | Action / Command | Expected System Behavior |
| :--- | :--- | :--- |
| **1. Code Modification** | Edit `main.py` locally and save. | Prepares new code changes (`v2`) while keeping `v1` untouched. |
| **2. Explicit Deploy** | `gcloud app deploy --version=v2` | Uploads code and builds artifacts for `v2` while `v1` stays live (zero downtime). |
| **3. Verification** | Access version-specific URLs | Allows direct testing of isolated versions via unique links. |
| **4. Traffic Splitting** | Configure split via Console (e.g., 50/50) | Routes production traffic proportionally across active versions based on IP/Cookie/Random. |

---

### 8. Common Mistakes & Exam Traps

*   **Trap 1**: Assuming that deploying version `v2` automatically deletes or overwrites version `v1`.
    *   *Correction*: App Engine preserves older versions unless you manually delete them. Multiple versions can run simultaneously.
*   **Trap 2**: Believing that traffic can only be split randomly.
    *   *Correction*: App Engine supports splitting traffic by **IP address**, **Cookie**, or **Random** distribution.
*   **Trap 3**: Expecting application downtime during a version deployment.
    *   *Correction*: App Engine guarantees zero-downtime releases; the old version continues handling live requests while the new version builds and deploys in the background.

---

### 9. Certification Practice Questions

#### Question 1
An administrator deploys version `v2` of a service using the command `gcloud app deploy --version=v2`. While the deployment build is actively running in the background, what happens to user requests hitting the application URL?
*   A) Requests fail with a 503 Service Unavailable error until the build completes.
*   B) Requests are automatically queued until the new version takes over entirely.
*   C) Requests continue to be served successfully by version `v1` with zero downtime.
*   D) Requests are split 50/50 immediately between `v1` and the building `v2` instance.
*   **Correct Answer**: C
*   **Explanation**: App Engine executes deployments safely in the background, ensuring that the existing active version (`v1`) continues serving traffic without downtime.
*   **Exam Takeaway**: App Engine supports zero-downtime deployments; older versions remain live while new versions build.

#### Question 2
You have both version `v1` and version `v2` of an application deployed and running under the default service in App Engine. You want to route 50% of production traffic to `v1` and 50% to `v2`. Which criteria methods are natively supported by App Engine when configuring traffic splitting? (Choose the best combination)
*   A) CPU utilization, memory thresholds, and disk I/O.
*   B) IP address, Cookie, or Random distribution.
*   C) Geographic country code, browser language header, and operating system.
*   D) User IAM role credentials and project permissions.
*   **Correct Answer**: B
*   **Explanation**: App Engine supports splitting traffic across active versions using **IP address**, **Cookie**, or **Random** allocation rules.
*   **Exam Takeaway**: Traffic splitting methods supported by App Engine: IP address, Cookie, and Random.

#### Question 3
What command-line flag is used with `gcloud app deploy` to explicitly specify a custom version identifier string like `v2` instead of letting Google Cloud generate an automatic timestamp-based version ID?
*   A) `--label=v2`
*   B) `--version=v2`
*   C) `--tag=v2`
*   D) `--name=v2`
*   **Correct Answer**: B
*   **Explanation**: The correct flag to assign a explicit version label during deployment is `--version=v2`.
*   **Exam Takeaway**: Use `--version=<VERSION_ID>` to assign explicit version tags during deployment.

#### Question 4
After successfully deploying version `v2` of an App Engine application without any extra arguments, what is the default traffic routing behavior for that newly deployed version?
*   A) It receives 0% of traffic until manual traffic splitting rules are saved.
*   B) It is automatically promoted to receive 100% of the live production traffic.
*   C) It shares traffic equally (50/50) with the previous version.
*   D) It is placed in a paused state until manually started.
*   **Correct Answer**: B
*   **Explanation**: By default, running `gcloud app deploy` automatically promotes the new deployment to receive 100% of live traffic.
*   **Exam Takeaway**: Default behavior promotes newly deployed versions to 100% traffic unless overridden.

#### Question 5
An engineer wants to test version `v2` of an application directly without affecting the production users who are currently visiting the main application URL served by version `v1`. How can this be accomplished?
*   A) Delete version `v1` temporarily, test `v2`, and then recreate `v1`.
*   B) Access the version-specific URL provided for `v2` in the Google Cloud Console.
*   C) Stop all instances of version `v1` via the command line.
*   D) Change the project region to isolate the test environment.
*   **Correct Answer**: B
*   **Explanation**: App Engine generates unique version-specific URLs for every deployed version, allowing administrators to access and test specific versions directly without impacting production traffic.
*   **Exam Takeaway**: Version-specific URLs allow direct isolated testing of non-default versions.

#### Question 6
If you configure traffic splitting by **Cookie** in App Engine, what is the resulting behavior for returning users?
*   A) Their requests are routed randomly across all active versions on every single page refresh.
*   B) A browser cookie ensures that subsequent requests from the same user are pinned to the same version they originally hit.
*   C) Users are blocked from accessing the site if their cookies are disabled.
*   D) Traffic is split based strictly on the user's public internet service provider IP address block.
*   **Correct Answer**: B
*   **Explanation**: Cookie-based traffic splitting sets a cookie in the user's browser, ensuring consistent routing to the same application version across multiple requests.
*   **Exam Takeaway**: Cookie splitting ensures user session consistency across requests to the same version.

#### Question 7
What happens to version `v1` of an App Engine service when version `v2` is deployed and promoted to receive 100% of the traffic?
*   A) Version `v1` is automatically deleted from the GCP project.
*   B) Version `v1` remains active in the background and can be rolled back to or split traffic with at any time.
*   C) Version `v1` is moved to App Engine Flexible automatically.
*   D) Version `v1` is permanently archived to Coldline Cloud Storage.
*   **Correct Answer**: B
*   **Explanation**: Deploying a new version does not delete older versions. Version `v1` remains preserved, allowing you to rollback traffic or re-allocate split percentages whenever needed.
*   **Exam Takeaway**: Older versions are retained after new deployments, allowing instant rollbacks.

#### Question 8
If an organization runs multiple independent microservices (e.g., `frontend` and `payments`) within a single App Engine application, how are these microservices organized in the console?
*   A) As separate versions under the same service.
*   B) As separate services (formerly modules) under the application.
*   C) As separate App Engine applications in distinct GCP projects.
*   D) As separate persistent disk volumes attached to version one.
*   **Correct Answer**: B
*   **Explanation**: Microservices within an App Engine application are organized as distinct **services**. Each service can independently have multiple versions and traffic splits.
*   **Exam Takeaway**: Microservices in App Engine are structured as separate services.

#### Question 9
When configuring traffic splitting using the **IP address** method, how does App Engine handle requests from users behind a corporate proxy who share the same external NAT IP address?
*   A) All users behind that proxy will consistently be routed to the same version.
*   B) Traffic from that IP address is blocked for security reasons.
*   C) Requests from that IP address are split randomly per click because proxy IPs are ignored.
*   D) The proxy IP address triggers an automatic rollback to version one.
*   **Correct Answer**: A
*   **Explanation**: IP address-based traffic splitting hashes the incoming client IP address. Users sharing the same public IP address (such as a corporate NAT gateway) will consistently land on the same version.
*   **Exam Takeaway**: IP splitting binds users sharing an IP address (like a proxy/NAT gateway) to the same version.

#### Question 10
Which of the following describes the correct relationship hierarchy from top to lowest level in App Engine?
*   A) Version $\rightarrow$ Service $\rightarrow$ Instance $\rightarrow$ Application
*   B) Application $\rightarrow$ Service $\rightarrow$ Version $\rightarrow$ Instance
*   C) Instance $\rightarrow$ Version $\rightarrow$ Service $\rightarrow$ Application
*   D) Service $\rightarrow$ Application $\rightarrow$ Instance $\rightarrow$ Version
*   **Correct Answer**: B
*   **Explanation**: The proper hierarchy flows downward from Application (top container) to Services (microservices), Versions (deployments), and finally Instances (running runtimes).
*   **Exam Takeaway**: Memorize the hierarchy: Application $\rightarrow$ Service $\rightarrow$ Version $\rightarrow$ Instance.