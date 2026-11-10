# GCP Associate Cloud Engineer (ACE) Study Notes

## Chapter: Hosting a Static Website on Google Cloud Storage

---

### 1. Overview
Google Cloud Storage can be used to host **static websites** (comprising HTML, CSS, client-side JavaScript, and images) directly without provisioning or managing virtual machines, web servers (like Nginx/Apache), or backend scaling infrastructure.

#### Why Use Cloud Storage for Static Web Hosting?
- **Serverless & Fully Managed:** No infrastructure provisioning, OS patching, or maintenance required.
- **High Availability & Scalability:** Automatically scales up or down based on traffic load without manual intervention.
- **Cost-Effective:** Pay only for storage consumed and network egress utilized, with zero idle server costs.

---

### 2. ACE Exam Focus & Common Exam Traps
- **Exam Importance:** **HIGH**. Scenario-based questions frequently test how to expose storage objects publicly and configure static website elements.
- **High-Priority Topics:**
  - **Public Access Prevention (PAP):** PAP is enabled by default on all new Cloud Storage buckets. To make a website public, you must **disable Public Access Prevention** first.
  - **Public Principal:** To grant public read access, use the **`allUsers`** identifier (representing anyone on the public internet, anonymous or authenticated).
  - **Required Role:** Assign the **Storage Object Viewer** role (`roles/storage.objectViewer`) to the `allUsers` principal.
  - **Domain Verification (Optional Custom Domain):** If hosting on a custom domain (e.g., `www.in28minutes.com`), the bucket name must match the domain name exactly, and domain ownership must be verified.
- **Common Exam Traps:**
  - Selecting `allAuthenticatedUsers` when a website needs to be completely public to anonymous visitors. (`allAuthenticatedUsers` requires any valid Google/external account login; `allUsers` allows truly public unauthenticated browsing).
  - Forgetting to disable Public Access Prevention before attempting to add public IAM bindings.

---

### 3. Detailed Concept Explanation

#### A. Architecture and Core Requirements
1. **Bucket Creation:** Files are uploaded into a Cloud Storage bucket. For custom domain setups, the bucket name must match the domain name precisely.
2. **Core Content Files:** Typically includes an entry point homepage (`index.html`) and an optional error page (`404.html` or similar).
3. **Overcoming Public Access Prevention (PAP):** 
   - PAP is a security guardrail enforced across Google Cloud to block accidental public data exposure. 
   - For public-facing static websites, PAP must be explicitly turned off at the bucket level.
4. **IAM Policy Configuration for Public Access:**
   - **Principal:** `allUsers`
   - **Role:** `Storage Object Viewer` (`roles/storage.objectViewer`)
   - **Warning Prompt:** The console issues a public access warning popup requiring explicit confirmation ("Allow public access") before saving the policy.

---

### 4. Practical Setup & Implementation Guide

#### Step-by-Step Implementation via GCP Console
1. **Navigate to Cloud Storage:**
   - Open the GCP Console, search for **Cloud Storage**, and select **Buckets**.
2. **Prepare or Select a Bucket:**
   - Use an existing bucket or create a new one (e.g., `my-first-bucket-in-28minutes`). Upload your static website files (`index.html`, CSS, JS, images) into the bucket.
3. **Disable Public Access Prevention (PAP):**
   - Click on your bucket name and go to the **Permissions** tab.
   - Locate the setting for **Public Access Prevention** and change it from *Enforced* to **Not enforced** (Disabled). Confirm the action when prompted.
4. **Grant Public Read Access:**
   - Refresh the permissions view if necessary, then click **Grant Access** (or **Add Principal**).
   - In the **New principals** field, type `allUsers` and select it.
   - In the **Select a role** dropdown, search for and choose **Storage Object Viewer** (`roles/storage.objectViewer`).
   - Click **Save**.
   - When the public access warning prompt appears ("Are you sure you want to make this resource public?"), click **Allow public access**.
5. **Testing and Verification:**
   - Navigate back to the **Objects** tab inside your bucket.
   - Click on your homepage file (e.g., `index.html`).
   - Copy its generated **Public URL** (or Object URL).
   - Paste the URL into a web browser tab to confirm that the static website renders correctly.

---

### 5. Visual Representations

#### Static Website Public Access Architecture Flow
```text
  +-------------------------------------------------------------+
  |                   PUBLIC INTERNET USER                      |
  +-------------------------------------------------------------+
                                 |
                        Requests index.html via Public URL
                                 |
                                 v
  +-------------------------------------------------------------+
  |              GOOGLE CLOUD STORAGE BUCKET                    |
  |                                                             |
  |   - Public Access Prevention: DISABLED                      |
  |   - IAM Policy Binding: Principal [allUsers]                |
  |     assigned Role [Storage Object Viewer]                   |
  |                                                             |
  |   +-----------------------------------------------------+   |
  |   |                   index.html                        |   |
  |   +-----------------------------------------------------+   |
  +-------------------------------------------------------------+
```

---

### 6. Mental Models & Analogies

#### The Billboard Analogy
- **Standard Private Storage Bucket:** Like filing cabinet drawers locked inside a secure corporate office basement—only authorized internal employees can view them.
- **Disabling PAP & Adding `allUsers`:** Like taking a project poster out of the secure file drawer and mounting it on an outdoor public billboard facing the public street so anyone walking past can look at it freely.

---

### 7. Step-by-Step Workflows / Comparison Tables

#### Comparison: Traditional Hosting vs. Cloud Storage Static Hosting
| Feature | Traditional Server Hosting (e.g., Nginx on Compute Engine) | Cloud Storage Static Hosting |
| :--- | :--- | :--- |
| **Infrastructure Management** | High (OS updates, patches, scaling groups) | **None** (Fully managed serverless storage) |
| **Uptime & Availability** | Requires manual high-availability configuration | Built-in high availability provided by Google |
| **Scaling** | Requires managed instance groups and auto-scaling rules | **Infinite automatic scaling** with traffic fluctuations |
| **Cost Model** | Pay for running instances 24/7 (even idle) | Pay only for storage size and data egress |

---

### 8. Common Mistakes & Exam Traps
- **Trap:** Forgetting that Public Access Prevention (PAP) is enabled by default.
  - *Correction:* If an exam question states that users receive a "Permission Denied" or 403 error when trying to access files in a public static website bucket, verify whether Public Access Prevention needs to be disabled or if the `allUsers` principal is missing the Storage Object Viewer role.
- **Trap:** Using `allAuthenticatedUsers` instead of `allUsers` when building a completely open public website.
  - *Exam Tip:* `allAuthenticatedUsers` requires visitors to have a Google account. If the website must be viewable anonymously by anyone on the internet, you must use **`allUsers`**.

---

### 9. Certification Practice Questions

#### Question 1
You have uploaded a static website portfolio built with HTML and CSS into a Google Cloud Storage bucket. You want to make the website accessible anonymously to anyone on the public internet. Which two steps must you perform?
- A. Enable Public Access Prevention and assign the Project Owner role to `allUsers`.
- B. Disable Public Access Prevention and assign the `Storage Object Viewer` role to the `allUsers` principal.
- C. Format all files to XML and assign the Compute Admin role to `allAuthenticatedUsers`.
- D. Create a user-managed service account key and hardcode it into the website's index.html file.
- **Correct Answer:** **B**  
- **Explanation:** To make a bucket public, you must disable Public Access Prevention and bind the `Storage Object Viewer` role to the `allUsers` principal.  
- **Incorrect Answers:** A keeps PAP enabled (blocking access). C uses the wrong role and principal type. D is a security violation.  
- **Exam Takeaway:** Public website hosting requires disabling PAP and assigning `Storage Object Viewer` to `allUsers`.

---

#### Question 2
What is the primary operational advantage of hosting a static website directly out of Google Cloud Storage compared to deploying it on a traditional unmanaged Compute Engine virtual machine?
- A. Cloud Storage provides absolute control over the Linux kernel configuration.
- B. Cloud Storage is fully managed, highly available, and scales automatically without requiring server maintenance or OS patching.
- C. Cloud Storage eliminates all network data egress charges completely.
- D. Cloud Storage allows direct SSH root shell access for debugging web scripts.
- **Correct Answer:** **B**  
- **Explanation:** Cloud Storage is serverless and fully managed, freeing administrators from infrastructure upkeep, patching, and scaling configuration.  
- **Incorrect Answers:** A and D describe VMs, not storage buckets. C is false (egress costs still apply).  
- **Exam Takeaway:** Cloud Storage static hosting eliminates server management and scales automatically.

---

#### Question 3
What security setting is enabled by default on all newly created Google Cloud Storage buckets to protect against accidental public data exposure?
- A. Public Access Prevention (PAP)
- B. VPC Service Controls
- C. Automatic Signed URL rotation
- D. Object-level encryption bypass
- **Correct Answer:** **A**  
- **Explanation:** Public Access Prevention (PAP) is enabled by default on all new buckets to prevent accidental public sharing.  
- **Incorrect Answers:** B, C, and D are separate security features or incorrect terms.  
- **Exam Takeaway:** Public Access Prevention (PAP) is enabled by default on new buckets.

---

#### Question 4
If you want to configure a custom domain name (e.g., `www.my-company-website.com`) for a static website hosted on Cloud Storage, what is the first requirement regarding the bucket naming convention?
- A. The bucket name can be anything, as long as it starts with "www".
- B. The bucket name must match the domain name exactly.
- C. The bucket name must be an auto-generated cryptographic UUID string.
- D. The bucket name must match the project ID.
- **Correct Answer:** **B**  
- **Explanation:** When mapping a custom domain name to a Cloud Storage static website, Google requires the storage bucket name to match the custom domain name precisely.  
- **Incorrect Answers:** A, C, and D do not satisfy custom domain bucket naming rules.  
- **Exam Takeaway:** Custom domain buckets in GCP must match the domain name exactly.

---

#### Question 5
An administrator attempts to add the `allUsers` principal with the `Storage Object Viewer` role to a bucket, but the GCP Console blocks the change and reports that Public Access Prevention is enforced. What must the administrator do first?
- A. Delete the storage bucket and create a new project.
- B. Turn off or disable Public Access Prevention on the bucket.
- C. Upgrade the project billing account to Enterprise tier.
- D. Rotate the project owner service account keys.
- **Correct Answer:** **B**  
- **Explanation:** Public Access Prevention must be disabled before public IAM bindings (`allUsers`) can be applied to a bucket.  
- **Incorrect Answers:** A, C, and D are unnecessary steps.  
- **Exam Takeaway:** Disable Public Access Prevention before making a bucket public.

---

#### Question 6
What is the functional difference between the `allUsers` identifier and the `allAuthenticatedUsers` identifier when configuring bucket permissions?
- A. `allUsers` includes anyone on the public internet (including anonymous visitors), whereas `allAuthenticatedUsers` requires the user to have any valid Google or external cloud identity.
- B. `allUsers` is restricted to internal employees, while `allAuthenticatedUsers` is public.
- C. There is no functional difference; they are interchangeable aliases.
- D. `allUsers` requires multi-factor authentication, while `allAuthenticatedUsers` does not.
- **Correct Answer:** **A**  
- **Explanation:** `allUsers` represents everyone on the internet anonymously, while `allAuthenticatedUsers` represents anyone logged in with a Google or federated identity.  
- **Incorrect Answers:** B, C, and D are incorrect definitions.  
- **Exam Takeaway:** Know the distinction: `allUsers` = anonymous public internet; `allAuthenticatedUsers` = any logged-in identity.

---

#### Question 7
Which IAM role should be assigned to the `allUsers` principal to allow visitors to view and render static website assets like HTML and images?
- A. `roles/storage.admin`
- B. `roles/storage.objectAdmin`
- C. `roles/storage.objectViewer`
- D. `roles/editor`
- **Correct Answer:** **C**  
- **Explanation:** `roles/storage.objectViewer` grants read-only access (`get` and `list`) to view objects and their metadata without modification rights.  
- **Incorrect Answers:** A and B grant administrative/write privileges. D grants project-wide editor rights.  
- **Exam Takeaway:** Use `storage.objectViewer` for public website read access.

---

#### Question 8
True or False: Hosting a static website in Cloud Storage requires configuring backend server-side scripts like PHP or Node.js runtime engines inside the bucket.
- A. True
- B. False
- **Correct Answer:** **B**  
- **Explanation:** False. Cloud Storage static hosting only serves static assets (HTML, CSS, client-side JS, images); it cannot execute server-side runtime scripts like PHP or Python natively.  
- **Exam Takeaway:** Cloud Storage static hosting is limited strictly to static files (HTML/CSS/JS/images); it does not run server-side code.

---

#### Question 9
When you make a bucket public by assigning `allUsers` the `Storage Object Viewer` role, what security warning message does the GCP Console display?
- A. A warning prompting you to confirm that adding all users will make the resource publicly accessible to anyone on the internet.
- B. An error demanding that you enter a credit card authorization code.
- C. A notification stating that the bucket will be deleted in 24 hours.
- D. A prompt requiring you to enable Cloud SQL database backups.
- **Correct Answer:** **A**  
- **Explanation:** The console explicitly warns administrators before applying public IAM policies to confirm that data exposure is intentional.  
- **Incorrect Answers:** B, C, and D are unrelated warnings.  
- **Exam Takeaway:** Expect explicit public access warning confirmations when making storage buckets public in the console.

---

#### Question 10
If a static website hosted on Cloud Storage experiences a sudden 100x spike in visitor traffic due to a viral marketing campaign, how does Google Cloud Storage respond?
- A. The bucket crashes due to CPU overload unless you manually scale out virtual machine instances.
- B. Cloud Storage scales automatically and indefinitely to adapt to the traffic increase without manual intervention.
- C. The website is automatically throttled and returns HTTP 503 errors until billing limits are increased.
- D. Visitors are redirected to a maintenance page managed by Google Support.
- **Correct Answer:** **B**  
- **Explanation:** Cloud Storage scales automatically and indefinitely to handle fluctuating traffic loads.  
- **Incorrect Answers:** A, C, and D describe server-based bottlenecks that do not apply to serverless storage hosting.  
- **Exam Takeaway:** Cloud Storage scales automatically without manual server scaling adjustments.