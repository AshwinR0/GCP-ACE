# Google Cloud Storage (GCS) - Object Lifecycle Management Study Notes

---

### 1. Overview
Object Lifecycle Management in Google Cloud Storage (GCS) allows you to automate the cost optimization and cleanup of your stored objects [1]. Because data access frequency naturally drops over time, keeping everything in the Standard storage class indefinitely can become expensive. 

Lifecycle management lets you define rules containing **conditions** and **actions** at the bucket level [1]. When objects meet your specified conditions, GCS automatically applies the configured actions—such as transitioning data to cheaper storage classes (e.g., Nearline, Coldline, Archive) or permanently deleting unneeded files—without requiring manual intervention [1].

---

### 2. ACE Exam Focus & Common Exam Traps

- **Exam Importance:** **HIGH**
- **Core Concepts:**
  - Lifecycle rules consist of **Conditions** (e.g., age, creation date, storage class, live vs. non-current status, newer versions count) and **Actions** (`SetStorageClass` or `Delete`) [1].
  - **Strict Transition Rules / Pathways:** You can only transition storage classes *downward* along a specific progression path [1]. You cannot skip tiers arbitrarily or transition "upwards" (e.g., you cannot transition from Archive back to Standard via lifecycle rules).
- **High-Priority Examination Topics:**
  - Allowed transition paths:
    - **Standard / Multi-Regional / Regional** $\rightarrow$ Nearline, Coldline, or Archive [1].
    - **Nearline** $\rightarrow$ Coldline or Archive [1].
    - **Coldline** $\rightarrow$ Archive [1].
  - Multiple conditions logic: When multiple conditions are specified within a single rule, **all conditions must be satisfied** for the action to trigger [1].
  - Minimum storage duration interaction: Transitioning or deleting data via lifecycle rules still respects minimum storage duration charges (e.g., Nearline requires 30 days, Coldline 90 days, Archive 365 days); moving data before these windows expire can incur pro-rated early-deletion fees.

---

### 3. Detailed Concept Explanation

#### A. Components of a Lifecycle Rule
A lifecycle rule is structured using two main blocks configured at the **bucket level** [1]:
1. **Conditions (When to apply):** Criteria that an object must meet. Multiple conditions can be combined, and **all conditions must evaluate to true** for the action to run [1]. Common conditions include:
   * **Age:** Number of days since the object was created (defaults to days) [1].
   * **Created Before:** A specific calendar date/time boundary.
   * **IsLive:** Boolean check (true if it's the current live version; false if it's a non-current version) [1].
   * **MatchesStorageClass:** Filters objects currently residing in a specific storage class (e.g., `STANDARD`) [1].
   * **NumNewerVersions:** Counts how many newer iterations exist (useful for version-enabled buckets) [1].
   * **DaysSinceNoncurrent:** Age tracking for non-current versions.
2. **Actions (What to do):** The automated operation executed when conditions match [1]:
   * **`SetStorageClass`:** Moves the object to a more cost-effective, colder storage class (e.g., Standard $\rightarrow$ Archive) [1].
   * **`Delete`:** Permanently removes the object or non-current version from the bucket [1].

#### B. Allowed Storage Class Transition Matrix
GCS enforces strict rules on the direction of storage class transitions. You can only move data downwards to cooler tiers:
* From **Standard / Multi-Regional / Regional** $\rightarrow$ Can move to **Nearline**, **Coldline**, or **Archive** [1].
* From **Nearline** $\rightarrow$ Can move to **Coldline** or **Archive** [1].
* From **Coldline** $\rightarrow$ Can move only to **Archive** [1].
* *Illegal Transitions:* Moving data "upwards" (e.g., Archive $\rightarrow$ Standard) or skipping downward rules via lifecycle automation is not permitted.

---

### 4. Practical Setup & Implementation Guide

#### Configuring Lifecycle Rules via the Google Cloud Console
1. **Prerequisites:** An active GCS bucket and appropriate IAM roles (Storage Admin).
2. **Console Navigation:**
   * Open the **Google Cloud Console** $\rightarrow$ **Cloud Storage** $\rightarrow$ **Buckets**.
   * Click on your target bucket (e.g., `my-first-bucket-in-28-minutes`).
3. **Adding a Rule:**
   * Select the **Lifecycle** tab at the top of the bucket details page [1].
   * Click **Add a rule** [1].
   * **Step 1: Select rule action:** Choose whether you want to *Set storage class* or *Delete objects*, then click **Continue** [1].
   * **Step 2: Select object conditions:** Define your filters (e.g., set **Age** to `30 days`, or specify storage class constraints like `Standard`), then click **Continue** [1].
   * **Step 3: Review and save:** Confirm your configuration parameters and click **Create** to activate the rule.
4. **Expected Outcome:** The rule immediately begins evaluating bucket objects automatically in the background. As objects hit the specified criteria (e.g., age > 30 days), GCS executes the action automatically without manual intervention [1].

---

### 5. Visual Representations

#### Lifecycle Transition Flow & Allowed Paths
```text
[Standard / Regional / Multi-Regional Storage] ──────────┐
   │                                                     │
   │ (Allowed via Lifecycle Rule)                        │ (Allowed)
   ▼                                                     ▼
[Nearline Storage (30-day min)] ─────────┐         [Archive Storage (365-day min)]
   │                                     │                ▲
   │ (Allowed)                           │                │
   ▼                                     │ (Allowed)      │
[Coldline Storage (90-day min)] ─────────┴────────────────┘

 *Rule: Transitions can only move DOWNWARDS to colder tiers. Upward transitions or skipping tiers directly is prohibited.*
```

#### Multiple Condition Logic Flow
```text
[Object Evaluation Check]
   │
   ├── Condition 1: Age > 30 Days? ──────────┐
   ├── Condition 2: IsLive == True? ─────────┼──> ALL conditions must be TRUE ──> [Trigger Action: Delete / SetStorageClass]
   └── Condition 3: StorageClass == Standard?┘    (If any condition fails ──────> Action skipped)
```

---

### 6. Mental Models & Analogies

* **The Corporate Records Retention Schedule (Lifecycle Management):**
  * **Analogy:** In a law firm, active case files sit on a desk for the first month (**Standard**). After 30 days, files are automatically boxed up and moved to an on-site filing room (**Nearline**). After 90 days, they move to a regional secure archive box storage locker (**Coldline**). After 1 year, unopened legacy files are shredded entirely (**Delete**). Nobody manually tracks calendar dates for every single file; a master company policy runs automatically in the background based on predefined rules.
  * **Mapping to GCP:** The office rules represent GCS Object Lifecycle Management policies executing automated transitions and cleanups based on age conditions.

---

### 7. Step-by-Step Workflows / Comparison Tables

#### Action Types in GCS Lifecycle Management

| Action Type | Parameter Requirements | Primary Objective | Example Use Case |
| :--- | :--- | :--- | :--- |
| **`SetStorageClass`** | Target colder storage tier (Nearline, Coldline, Archive) [1] | Optimize costs by moving aging data to cheaper at-rest tiers [1] | Move data older than 365 days from Standard to Archive [1]. |
| **`Delete`** | Age, non-current status, or date boundaries [1] | Automatically purge redundant or expired data to save space [1] | Delete temporary staging logs older than 30 days [1]. |

---

### 8. Common Mistakes & Exam Traps

* **Mistake:** Attempting to write a lifecycle rule that automatically moves data *upwards* from Archive back to Standard storage.
  * **Correction:** Lifecycle rules only support *downward* transitions toward colder storage classes [1]. You cannot use lifecycle rules to promote data back to Standard.
* **Exam Trap:** Expecting an action to fire when only *some* of the multiple conditions defined in a rule are met.
  * **Correction:** When multiple conditions are combined in a single rule, **all conditions must be satisfied** for the action to execute [1].
* **Mistake:** Forgetting about minimum storage duration penalties when setting up automated lifecycle deletions or transitions.
  * **Correction:** If a lifecycle rule moves or deletes data before its minimum duration window expires (e.g., moving Standard data to Archive and deleting it at day 40), pro-rated early-deletion fees apply for the balance of the minimum commitment period.

---

### 9. Certification Practice Questions

#### Q1: An administrator wants to configure an object lifecycle management rule to automatically move files from Standard storage to Archive storage after they exceed 1 year of age [1]. Which action type should be selected?
- A. `Delete`
- B. `SetStorageClass` [1]
- C. `UpgradeClass`
- D. `MakeImmutable`
- **Correct Answer:** B
- **Explanation:** The `SetStorageClass` action type is used to transition objects to a colder storage class (such as Archive) based on conditions like age [1].
- **Incorrect Answers:** A is used for data removal, not class transitions. C and D are incorrect action identifiers that do not exist in GCS lifecycle configurations.
- **Exam Takeaway:** Changing storage tiers via automation uses the `SetStorageClass` action [1].

#### Q2: A lifecycle rule is configured with multiple conditions: `Age > 30 days`, `IsLive = true`, and `StorageClass = Standard`. How does GCS evaluate these conditions when determining whether to execute the action?
- A. Any single condition matching will trigger the action.
- B. All conditions must be satisfied simultaneously for the action to trigger [1].
- C. Conditions are evaluated using a round-robin weighted priority system.
- D. The rule fails validation because multiple conditions cannot be combined.
- **Correct Answer:** B
- **Explanation:** When multiple conditions are set in a lifecycle rule, every condition must evaluate to true for the action to happen [1].
- **Incorrect Answers:** A, C, and D misstate GCS rule evaluation logic.
- **Exam Takeaway:** Multiple conditions require **all conditions to match** [1].

#### Q3: Which of the following storage class transition paths is **allowed** under GCS Object Lifecycle Management rules?
- A. Archive storage moving back up to Standard storage automatically.
- B. Coldline storage moving directly up to Nearline storage.
- C. Standard storage moving down to Archive storage [1].
- D. Nearline storage moving up to Regional Standard storage.
- **Correct Answer:** C
- **Explanation:** Transitions must move downward toward colder tiers. Standard to Archive is a valid downward transition [1].
- **Incorrect Answers:** A, B, and D describe upward transitions or tier-skipping promotions, which are prohibited by lifecycle rule policies.
- **Exam Takeaway:** Lifecycle transitions only move **downwards** to colder classes [1].

#### Q4: A company wants to automatically delete temporary log objects 60 days after they are uploaded to a bucket, without manual administrator intervention. What is the most efficient native GCP mechanism to achieve this?
- A. Write a custom Cron job on a Compute Engine instance to loop `gsutil rm` daily.
- B. Configure an Object Lifecycle Management rule with a `Delete` action and an age condition of 60 days [1].
- C. Set the bucket storage class default to Archive immediately.
- D. Enable object versioning and manually purge non-current files.
- **Correct Answer:** B
- **Explanation:** Lifecycle management handles automated object deletion natively based on age conditions without requiring external scripts [1].
- **Incorrect Answers:** A introduces unnecessary infrastructure overhead. C changes cost tiers, but doesn't delete files. D requires manual cleanup instead of full automation.
- **Exam Takeaway:** Automated deletion based on age is a core use case for GCS **Lifecycle Management** [1].

#### Q5: Can an object stored in the Nearline storage class be transitioned directly to Coldline storage using an automated GCS lifecycle rule?
- A. No, Nearline can only transition to Archive.
- B. Yes, Nearline can transition to Coldline or Archive [1].
- C. Yes, Nearline can transition directly back to Standard.
- D. No, Nearline objects cannot participate in lifecycle management rules.
- **Correct Answer:** B
- **Explanation:** Nearline storage is allowed to transition downward to either Coldline or Archive storage [1].
- **Incorrect Answers:** A is too restrictive. C is an upward transition (prohibited). D is false.
- **Exam Takeaway:** Nearline can transition to **Coldline or Archive** [1].

#### Q6: What happens if an object is moved from Standard storage to Archive storage via a lifecycle rule after only 10 days, and is then immediately deleted on day 15?
- A. No additional charges apply because the object lived less than a month.
- B. Early-deletion penalties apply for the remainder of the Archive class's 365-day minimum duration requirement.
- C. The deletion request is blocked by GCS until 365 days pass.
- D. The file converts back to Standard storage automatically to avoid fees.
- **Correct Answer:** B
- **Explanation:** Deleting data before its minimum storage duration period elapses incurs pro-rated early deletion charges for the remaining balance.
- **Incorrect Answers:** A ignores minimum duration rules. C is false; you can delete files anytime, but you pay for it. D is false.
- **Exam Takeaway:** Minimum storage duration commitments apply even when objects are managed by automated lifecycle rules.

#### Q7: Which condition checks whether an object is the current active iteration or an older historical version in a version-enabled bucket?
- A. `MatchesStorageClass`
- B. `IsLive` [1]
- C. `Age`
- D. `CreatedBefore`
- **Correct Answer:** B
- **Explanation:** `IsLive` checks whether an object is the current active version (true) or a non-current version (false) [1].
- **Incorrect Answers:** A checks class tiers. C and D track chronological time boundaries.
- **Exam Takeaway:** Use `IsLive` to filter current versus historical object versions in lifecycle rules [1].

#### Q8: An administrator configures a lifecycle rule with the action `SetStorageClass` to Coldline, but omits specifying any conditions. How will GCS process this rule?
- A. It will immediately apply Coldline status to every object in the bucket.
- B. It will fail validation because lifecycle rules require conditions to define when actions apply.
- C. It will default to applying the action exclusively to new uploads older than 30 days.
- D. It will convert the bucket container into a cold storage zone.
- **Correct Answer:** B
- **Explanation:** Lifecycle rules require valid conditions (such as age or storage class filters) to determine target applicability; rule creation without conditions is invalid.
- **Incorrect Answers:** A, C, and D describe invalid or unconfigured behaviors.
- **Exam Takeaway:** Lifecycle rules require both valid conditions and actions.

#### Q9: Which of the following conditions evaluates how many newer iterations of an object exist when versioning is enabled on a bucket?
- A. `Age`
- B. `NumNewerVersions` [1]
- C. `DaysSinceNoncurrent`
- D. `MaxVersionLimit`
- **Correct Answer:** B
- **Explanation:** `NumNewerVersions` counts how many newer versions exist, which is useful for cleaning up older redundant history trees [1].
- **Incorrect Answers:** A tracks time. C tracks age since non-current transition. D is not a standard GCS lifecycle condition keyword.
- **Exam Takeaway:** `NumNewerVersions` targets historical version counts in version-enabled buckets [1].

#### Q10: What is the primary benefit of utilizing Object Lifecycle Management in GCS?
- A. It guarantees 100% uptime SLA for Archive data classes.
- B. It automates cost optimization and data cleanup without manual overhead [1].
- C. It speeds up multi-part file parallel upload throughput.
- D. It prevents accidental public data exposures.
- **Correct Answer:** B
- **Explanation:** The primary goal of lifecycle management is automating storage tier shifts and cleanups to keep cloud storage costs low without manual user overhead [1].
- **Incorrect Answers:** A is false (Archive has no SLA). C relates to parallel composite uploads. D relates to Public Access Prevention.
- **Exam Takeaway:** Lifecycle management = automated cost reduction and cleanup [1].