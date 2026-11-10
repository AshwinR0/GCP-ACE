# Comprehensive Study Notes: Managing GCP Compute Images via Gcloud

---

### 1. Overview
This module explores the management of **Compute Engine Images** in Google Cloud Platform using the Google Cloud CLI (`gcloud`). It covers listing images (with filtering flags like `--no-standard-images` and `--show-deprecated`), creating images from various sources (disks, snapshots, other images, or image families), deprecating and viewing deprecated items, deleting images, and exporting images to Cloud Storage buckets as virtual disks.

---

### 2. ACE Exam Focus & Common Exam Traps

**Exam Importance:** `HIGH`

- **High-Priority Concepts & Commands:**
  - **Listing Custom Images Only:** `gcloud compute images list --no-standard-images` hides the massive catalog of public Google-provided images and lists only user/custom images.
  - **Deprecation Behavior:** When an image is deprecated (`--state=deprecated`), a standard `gcloud compute images list --no-standard-images` will **hide** it by default. To view it, you must append the `--show-deprecated` flag.
  - **Image Creation Sources:** You can instantiate a new image from multiple sources:
    - `--source-disk` & `--source-disk-zone`
    - `--source-snapshot`
    - `--source-image` & `--source-image-project`
    - `--source-image-family` & `--source-image-project`
  - **Exporting Images:** `gcloud compute images export` enables exporting an image to a virtual disk file stored inside a Cloud Storage (GCS) bucket.
- **Keywords for Scenario Questions:** "List custom images only", "Hide/show deprecated images", "Create image from snapshot or disk", "Export image to Cloud Storage virtual disk".

---

### 3. Detailed Concept Explanation

#### A. Listing Images (`gcloud compute images list`)
* **Default Behavior:** Running `gcloud compute images list` outputs every public image provided by Google alongside your project's custom images, which can result in a massive and slow-loading dataset.
* **Filtering Custom Images:** Use the `--no-standard-images` flag to restrict output strictly to custom images created within your project.
* **Deprecation Visibility:** By default, standard listings only show active, non-deprecated images. To view deprecated images, you must explicitly use the `--show-deprecated` flag.

#### B. Creating Images (`gcloud compute images create`)
Images serve as templates for booting virtual machine instances. When creating them via `gcloud`, you can pull from multiple sources:
1. **From an Existing Disk:** `--source-image` or `--source-disk` along with `--source-disk-zone`.
2. **From a Snapshot:** `--source-snapshot=[SNAPSHOT_NAME]`.
3. **From Another Image / Project:** `--source-image=[NAME]` and `--source-image-project=[PROJECT_ID]`.
4. **From an Image Family:** `--source-image-family=[FAMILY_NAME]` to automatically fetch the latest non-deprecated image in that family.

#### C. Deprecating and Deleting Images
* **Deprecation:** Marking an image as deprecated (`gcloud compute images deprecate [IMAGE_NAME] --state=deprecated`) signals that it should no longer be used for new deployments without immediately breaking existing environments.
* **Deletion:** `gcloud compute images delete [IMAGE_NAME]` permanently removes the image template from the project (prompting a confirmation check).

#### D. Exporting Images
* **Command:** `gcloud compute images export`
* **Purpose:** Takes a GCP custom image and exports it as a virtual disk file (such as a `.vmdk` or raw disk file) into a designated Google Cloud Storage bucket path.

---

### 4. Practical Setup & Implementation Guide

#### A. Listing Custom Images
1. Open Cloud Shell and list only custom images in your project:
   ```bash
   gcloud compute images list --no-standard-images
   ```
2. To include deprecated images in your list output:
   ```bash
   gcloud compute images list --no-standard-images --show-deprecated
   ```

#### B. Creating an Image from a Source Snapshot
1. Create a new custom image from an existing point-in-time snapshot:
   ```bash
   gcloud compute images create my-custom-image-v2 \
       --source-snapshot=my-test-snapshot-v1
   ```

#### C. Deprecating an Image
1. Mark a custom image as deprecated:
   ```bash
   gcloud compute images deprecate my-custom-image-v2 \
       --state=deprecated
   ```
2. Verify that it disappears from a standard custom image list:
   ```bash
   gcloud compute images list --no-standard-images
   ```
3. View it again by adding the show-deprecated flag:
   ```bash
   gcloud compute images list --no-standard-images --show-deprecated
   ```

#### D. Exporting an Image to Cloud Storage
1. Export a custom image into a virtual disk file within a Cloud Storage bucket:
   ```bash
   gcloud compute images export \
       --destination-uri=gs://my-bucket-exports/my-image-disk.vmdk \
       --image=my-custom-image-v2
   ```

#### E. Deleting an Image
1. Permanently delete a custom image:
   ```bash
   gcloud compute images delete my-custom-image-v2
   ```

---

### 5. Visual Representations

#### Gcloud Compute Images Command Hierarchy & Flow
```text
gcloud compute images
  ├── list [--no-standard-images] [--show-deprecated]
  ├── create [NAME] 
  │     ├── --source-disk & --source-disk-zone
  │     ├── --source-snapshot
  │     ├── --source-image & --source-image-project
  │     └── --source-image-family & --source-image-project
  ├── deprecate [NAME] --state=deprecated
  ├── export --destination-uri=gs://... --image=...
  └── delete [NAME]
```

---

### 6. Mental Models & Analogies

- **Deprecating an Image (`--state=deprecated`):** Think of deprecating an image like putting a "Discontinued / Do Not Use for New Construction" stamp on an architectural blueprint. The blueprint is still sitting in the filing cabinet so old buildings built with it can reference it, but the foremen are warned not to use it for breaking ground on any new projects.

---

### 7. Step-by-Step Workflows / Comparison Tables

#### Image Management Gcloud Command Reference Table

| Task | Gcloud Command Syntax | Key Flags / Parameters |
| :--- | :--- | :--- |
| **List Custom Images Only** | `gcloud compute images list` | `--no-standard-images` |
| **List Including Deprecated** | `gcloud compute images list` | `--no-standard-images --show-deprecated` |
| **Create Image from Snapshot** | `gcloud compute images create [NAME]` | `--source-snapshot=[SNAPSHOT]` |
| **Create Image from Disk** | `gcloud compute images create [NAME]` | `--source-disk=[DISK] --source-disk-zone=[ZONE]` |
| **Deprecate Image** | `gcloud compute images deprecate [NAME]` | `--state=deprecated` |
| **Export Image to GCS** | `gcloud compute images export` | `--destination-uri=gs://... --image=[NAME]` |
| **Delete Image** | `gcloud compute images delete [NAME]` | None (prompts confirmation) |

---

### 8. Common Mistakes & Exam Traps

- **Trap 1:** Expecting `gcloud compute images list --no-standard-images` to display an image that has been marked as deprecated.
  * *Correction:* Deprecated images are hidden by default from standard listings. You must append `--show-deprecated` to view them.
- **Trap 2:** Confusing source flags when creating images from snapshots versus disks.
  * *Correction:* Use `--source-snapshot` for snapshots, but use `--source-disk` *and* `--source-disk-zone` when building an image from an active persistent disk.
- **Trap 3:** Assuming exporting an image copies it to another zone instead of a Cloud Storage bucket.
  * *Correction:* `gcloud compute images export` targets a Cloud Storage (`gs://`) destination URI to save a virtual disk file archive.

---

### 9. Certification Practice Questions

#### Question 1
An administrator executes `gcloud compute images list --no-standard-images` in their project, but a custom image they know exists does not appear in the output. What is the most likely reason?
- A. The image has been deleted permanently.
- B. The image has been deprecated, and deprecated images are hidden from standard listings by default.
- C. The user forgot to specify the zone flag.
- D. The Cloud Storage API is disabled.

* **Correct Answer:** B
* **Explanation:** Deprecated images are hidden by default from standard `gcloud compute images list` outputs. To view them, you must include the `--show-deprecated` flag.
* **Incorrect Answers:** 
  - A: If deleted, it would not exist at all, but deprecation is a common state filter trap.
  - C: Images are global resources; zone flags are not required for listing images.
  - D: Images are governed by Compute Engine APIs, not Cloud Storage API enablement for listings.
* **Exam Takeaway:** Deprecated images are hidden from standard list commands unless `--show-deprecated` is used.

#### Question 2
Which flag should be added to `gcloud compute images list --no-standard-images` if you need to view custom images that have been marked as deprecated?
- A. `--show-deprecated`
- B. `--include-deprecated=true`
- C. `--all-states`
- D. `--show-hidden-templates`

* **Correct Answer:** A
* **Explanation:** The correct flag to display deprecated images in `gcloud` image listings is `--show-deprecated`.
* **Incorrect Answers:** 
  - B, C, & D: Invalid flag syntax.
* **Exam Takeaway:** Use `--show-deprecated` to list deprecated images.

#### Question 3
You want to create a new custom image named `app-image-v1` using an existing point-in-time snapshot called `db-snap-01`. Which `gcloud` command should you execute?
- A. `gcloud compute images create app-image-v1 --source-snapshot=db-snap-01`
- B. `gcloud compute snapshots create app-image-v1 --source-image=db-snap-01`
- C. `gcloud compute disks create app-image-v1 --snapshot=db-snap-01`
- D. `gcloud compute images export app-image-v1 --source=db-snap-01`

* **Correct Answer:** A
* **Explanation:** Creating an image from a snapshot is performed via `gcloud compute images create [NAME] --source-snapshot=[SNAPSHOT_NAME]`.
* **Incorrect Answers:** 
  - B, C, & D: Incorrect syntax or command groupings.
* **Exam Takeaway:** Creating an image from a snapshot uses `--source-snapshot`.

#### Question 4
An engineer needs to export an existing custom image named `prod-golden-image` as a virtual disk file into a Google Cloud Storage bucket path (`gs://my-backup-bucket/image.vmdk`). Which command structure is correct?
- A. `gcloud compute images export --destination-uri=gs://my-backup-bucket/image.vmdk --image=prod-golden-image`
- B. `gcloud compute snapshots backup --bucket=my-backup-bucket --image=prod-golden-image`
- C. `gcloud compute disks export gs://my-backup-bucket/image.vmdk`
- D. `gcloud storage cp prod-golden-image gs://my-backup-bucket/`

* **Correct Answer:** A
* **Explanation:** The `gcloud compute images export` command uses the `--destination-uri` flag pointing to a GCS bucket alongside the `--image` flag.
* **Incorrect Answers:** 
  - B, C, & D: Incorrect syntax paths.
* **Exam Takeaway:** Image exporting uses `gcloud compute images export` with `--destination-uri`.

#### Question 5
How can an administrator mark an existing custom image named `legacy-image` as deprecated using the Google Cloud CLI?
- A. `gcloud compute images delete legacy-image --deprecated`
- B. `gcloud compute images deprecate legacy-image --state=deprecated`
- C. `gcloud compute images update legacy-image --status=inactive`
- D. `gcloud compute snapshots deprecate legacy-image`

* **Correct Answer:** B
* **Explanation:** The correct syntax to deprecate an image is `gcloud compute images deprecate [IMAGE_NAME] --state=deprecated`.
* **Incorrect Answers:** 
  - A: Deletes the image entirely instead of deprecating it.
  - C & D: Invalid command syntax.
* **Exam Takeaway:** Deprecate an image using `gcloud compute images deprecate [NAME] --state=deprecated`.

#### Question 6
When creating a custom image from an active persistent disk using `gcloud compute images create`, which additional parameters are required alongside the disk name?
- A. `--source-disk` and `--source-disk-zone`
- B. `--source-bucket` and `--region`
- C. `--source-snapshot-id` only
- D. `--local-ssd-path`

* **Correct Answer:** A
* **Explanation:** Creating an image from an active disk requires specifying both the disk name (`--source-disk`) and its location zone (`--source-disk-zone`).
* **Incorrect Answers:** 
  - B, C, & D: Incorrect flag configurations.
* **Exam Takeaway:** Building an image from a disk requires `--source-disk` and `--source-disk-zone`.

#### Question 7
True or False: Standard Compute Engine images provided publicly by Google appear by default when running `gcloud compute images list` unless filtered out.
- A. True
- B. False

* **Correct Answer:** A
* **Explanation:** True. Running `gcloud compute images list` outputs all public Google-provided images along with custom images, which requires filtering via `--no-standard-images` to isolate custom templates.
* **Incorrect Answers:** 
  - B: False; public images appear by default.
* **Exam Takeaway:** Unfiltered `gcloud compute images list` displays public Google images.

#### Question 8
If an image family is specified when creating a new VM instance or image, what does GCP automatically select?
- A. The oldest deprecated image in the family.
- B. The latest non-deprecated image belonging to that image family.
- C. A random image from any public project.
- D. A local SSD scratch partition.

* **Correct Answer:** B
* **Explanation:** Image families point automatically to the latest non-deprecated image within that specific family group.
* **Incorrect Answers:** 
  - A: Deprecated images are ignored.
  - C & D: Incorrect behaviors.
* **Exam Takeaway:** Image families resolve to the latest non-deprecated image.

#### Question 9
What happens when you execute `gcloud compute images delete my-image` via Cloud Shell?
- A. The image is deprecated automatically without confirmation.
- B. The command prompts for confirmation, and upon agreement, permanently deletes the image template.
- C. The image is exported to Cloud Storage automatically.
- D. The image is converted into a zonal persistent disk.

* **Correct Answer:** B
* **Explanation:** Deleting an image prompts for a confirmation check, and upon confirmation, permanently removes the image template.
* **Incorrect Answers:** 
  - A, C, & D: Incorrect outcomes; deletion is explicit and permanent upon confirmation.
* **Exam Takeaway:** Image deletion is permanent upon confirmation.

#### Question 10
Which `gcloud` command group manages Compute Engine images?
- A. `gcloud compute disks`
- B. `gcloud compute images`
- C. `gcloud compute snapshots`
- D. `gcloud storage buckets`

* **Correct Answer:** B
* **Explanation:** All image management operations are handled under the `gcloud compute images` command group.
* **Incorrect Answers:** 
  - A, C, & D: Manage disks, snapshots, and object storage buckets respectively.
* **Exam Takeaway:** Image commands fall under `gcloud compute images`.