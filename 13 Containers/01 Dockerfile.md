# Creating Docker Images: Dockerfile & Build Process

## 1. Overview
This topic covers the fundamentals of creating custom container images using a **Dockerfile** for different runtime environments (specifically Python and Node.js). A Dockerfile acts as a text-based blueprint containing step-by-step instructions required to package an application, its runtime, and its dependencies into a lightweight, portable, and reproducible container image. Understanding how to write a Dockerfile and build images via the Docker CLI is a foundational skill for deploying containerized workloads to Google Cloud services like **Cloud Run**, **Google Kubernetes Engine (GKE)**, and **Compute Engine**.

---

## 2. ACE Exam Focus & Common Exam Traps

- **Exam Importance:** **HIGH** (Containerization is central to GCP modern application development; questions frequently test deployment architectures involving Cloud Run and GKE which rely on valid container images).
- **High-Priority Topics:**
  - Purpose of a `Dockerfile` and basic instructions (`FROM`, `WORKDIR`, `COPY`, `RUN`, `EXPOSE`, `CMD`).
  - Use of **Alpine** base images for minimizing image size and optimizing security attack surfaces.
  - The `docker build` command syntax and tagging conventions (`-t image_name:tag`).
- **Common Exam Traps:**
  - Confusing `EXPOSE` with network port publishing (remember: `EXPOSE` is purely documentation/metadata; actual port mapping happens during runtime execution flags like `-p`).
  - Assuming base images must be heavy operating system distributions instead of lightweight distros like Alpine Linux.

---

## 3. Detailed Concept Explanation

### What is a Dockerfile?
A Dockerfile is a plaintext document containing sequential commands that Docker executes automatically to build an image. Each instruction creates a distinct layer in the final container image.

### Key Dockerfile Instructions
1. **`FROM`**: Specifies the base image to build upon (e.g., lightweight Linux distributions like Alpine).
2. **`WORKDIR`**: Sets the default working directory for subsequent instructions (`RUN`, `COPY`, `CMD`) inside the container filesystem.
3. **`COPY`**: Copies local files and directories from the host machine into the container's filesystem.
4. **`RUN`**: Executes build-time commands (such as downloading dependencies) inside the image layer.
5. **`EXPOSE`**: Documents which network ports the container listens on at runtime.
6. **`CMD`**: Defines the default command that executes when a container starts up from the image.

---

## 4. Practical Setup & Implementation Guide

### Prerequisites
- Docker engine installed locally.
- Application source code and a configuration file for dependencies (`requirements.txt` for Python, `package.json` for Node.js).

### Step 1: Writing a Python Dockerfile (`Dockerfile`)
Create a file named `Dockerfile` in your project root directory:

```dockerfile
# Use a lightweight Python Alpine base image
FROM python:3.9-alpine

# Set the working directory inside the container
WORKDIR /app

# Copy application files from host root to container working directory
COPY . /app

# Install Python dependencies defined in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Document the port the application runs on
EXPOSE 5000

# Specify the default command to run the application
CMD ["python", "app.py"]
```

### Step 2: Writing a Node.js Dockerfile (`Dockerfile`)
For a Node.js application, the structure remains consistent but utilizes Node-specific runtime commands:

```dockerfile
# Use a lightweight Node Alpine base image
FROM node:18-alpine

# Set the working directory inside the container
WORKDIR /app

# Copy package files and source code
COPY . /app

# Install Node.js dependencies cleanly
RUN npm ci

# Document the application port
EXPOSE 3000

# Specify the startup command
CMD ["node", "app.js"]
```

### Step 3: Building the Docker Image
Execute the `docker build` command in the directory containing your `Dockerfile`:

```bash
docker build -t my-python-app:1.0.0 .
```
* **`-t my-python-app:1.0.0`**: Tags the image with a custom name (`my-python-app`) and version version tag (`1.0.0`).
* **`.`**: Specifies the build context (current directory containing the `Dockerfile`).

### Step 4: Verification
Verify that the image has been successfully created locally:
```bash
docker images
```

---

## 5. Visual Representations

### Docker Build Workflow Architecture
```text
+-------------------+      +-------------------+      +------------------------+
| Local Directory   |      | Docker Daemon     |      | Final Container Image  |
|                   |      |                   |      |                        |
| - Dockerfile      | ---> | 1. Read FROM      | ---> | Layer 1: Base (Alpine) |
| - app.py          |      | 2. Run WORKDIR    |      | Layer 2: App Code      |
| - requirements.txt|      | 3. Copy Files     |      | Layer 3: Dependencies  |
|                   |      | 4. Run PIP Install|      |                        |
+-------------------+      +-------------------+      +------------------------+
```

---

## 6. Mental Models & Analogies

* **The Recipe & Cookbook Analogy:** Think of a `Dockerfile` as a cooking recipe written on a piece of paper, and the `docker build` command as the process of actually baking the cake. The final container image is the baked cake, which you can box up, ship anywhere (to GCP Cloud Run or GKE), and eat (run) reliably without needing to re-gather raw ingredients or rewrite instructions.
* **Alpine Linux Analogy:** Using an Alpine base image is like packing light for a trip using a compact travel backpack instead of a massive wardrobe trunk—it contains everything you need to function, but gets rid of all unnecessary weight and bulk.

---

## 7. Step-by-Step Workflows / Comparison Tables

### Comparison: Python vs. Node.js Container Configuration

| Feature / Step | Python Application | Node.js Application |
| :--- | :--- | :--- |
| **Base Image** | `python:X.X-alpine` | `node:XX-alpine` |
| **Dependency File** | `requirements.txt` | `package.json` / `package-lock.json` |
| **Install Command** | `pip install -r requirements.txt` | `npm ci` |
| **Startup Command** | `["python", "app.py"]` | `["node", "app.js"]` |

---

## 8. Common Mistakes & Exam Traps

- **Trap 1:** Believing `EXPOSE 5000` automatically opens network ports to public traffic or other containers. 
  * *Correction:* `EXPOSE` is purely informational metadata. You must map or expose ports explicitly at runtime using platform controls or container runtime flags.
- **Trap 2:** Forgetting the trailing dot (`.`) in the `docker build` command.
  * *Correction:* The dot tells Docker where to locate the build context (context path); omitting it results in a build syntax error.

---

## 9. Certification Practice Questions

### Question 1
An Associate Cloud Engineer is containerizing a Python microservice to deploy on Google Cloud Run. To ensure fast container startup times and a minimal security footprint, which type of base image should the engineer select in the Dockerfile?
- A. A full Windows Server Core image
- B. A lightweight Linux distribution like Alpine
- C. A heavy enterprise Ubuntu desktop image
- D. An uncompressed virtual machine disk snapshot

**Correct Answer:** B
**Explanation:** Alpine Linux provides a minimal, lightweight environment (often under 5 MB) which reduces image size, accelerates network transmission/pull times, and reduces the overall security vulnerability footprint.
**Incorrect Options:** A and C introduce massive unnecessary operating system binaries and bloat. D is irrelevant to container image builds.
**Exam Takeaway:** Always prefer lightweight Linux distros (like Alpine or slim variants) for container workloads on GCP.

### Question 2
Which Dockerfile instruction is used exclusively for documentation to declare the network port on which the container listens during runtime?
- A. `RUN`
- B. `WORKDIR`
- C. `EXPOSE`
- D. `CMD`

**Correct Answer:** C
**Explanation:** The `EXPOSE` instruction acts as a metadata documentation mechanism letting users or container orchestrators know which port the application listens on.
**Incorrect Options:** A (`RUN`) executes build commands. B (`WORKDIR`) sets paths. D (`CMD`) defines startup runtime operations.
**Exam Takeaway:** Know the exact purpose of individual Dockerfile instructions for configuration accuracy.

### Question 3
You have written a Dockerfile for a Node.js application in your current working directory. Which command should you execute to build a Docker image named `node-app` with the version tag `2.0`?
- A. `docker run -t node-app:2.0 .`
- B. `docker build -t node-app:2.0 .`
- C. `docker create -f node-app:2.0 .`
- D. `docker push node-app:2.0`

**Correct Answer:** B
**Explanation:** The `docker build` command compiles a Dockerfile into an image, and `-t` assigns a repository name and tag, followed by the build context path (`.`).
**Incorrect Options:** A (`docker run`) starts a container from an existing image, it does not build one. C is invalid syntax. D (`docker push`) uploads an already built image to a registry.
**Exam Takeaway:** Memorize basic Docker CLI syntax patterns for building (`docker build -t name:tag .`).

### Question 4
In a Python Dockerfile, dependencies are typically listed in a file called `requirements.txt`. What instruction is commonly used in the Dockerfile to install these dependencies into the container image?
- A. `COPY requirements.txt`
- B. `RUN pip install -r requirements.txt`
- C. `CMD pip install`
- D. `EXPOSE requirements.txt`

**Correct Answer:** B
**Explanation:** The `RUN` instruction executes build-time package manager commands like `pip install` to install required dependencies into the image layers.
**Incorrect Options:** A only copies the file; it does not execute installation. C executes operations at container startup rather than build time. D is an invalid usage of network documentation syntax.
**Exam Takeaway:** Use `RUN` for build-time operations like package installations.

### Question 5
What is the primary role of the `WORKDIR` instruction inside a Dockerfile?
- A. It downloads remote code repositories from GitHub.
- B. It defines the public port mapping for host routing.
- C. It sets the working directory for any subsequent `RUN`, `COPY`, and `CMD` instructions.
- D. It creates a persistent storage volume mapped to Google Cloud Storage.

**Correct Answer:** C
**Explanation:** `WORKDIR` dictates the active directory path for all subsequent instructions in the Dockerfile script.
**Incorrect Options:** A, B, and D describe completely separate behaviors unrelated to `WORKDIR`.
**Exam Takeaway:** `WORKDIR` handles path organization inside the container build context.

### Question 6
An engineer wants to build a Node.js container image. Instead of `npm install`, the Dockerfile uses `npm ci`. What is the primary operational characteristic of `npm ci`?
- A. It installs dependencies cleanly and rapidly using a `package-lock.json` file, making it ideal for automated builds.
- B. It upgrades all application packages to their absolute latest major version.
- C. It compiles raw Java source code inside a Node container.
- D. It publishes the container directly to Google Artifact Registry.

**Correct Answer:** A
**Explanation:** `npm ci` (Node Package Manager Clean Install) installs dependencies directly from `package-lock.json`, ensuring repeatable and reliable builds in CI/CD pipelines and Dockerfiles.
**Incorrect Options:** B contradicts the goal of locked versions. C and D describe unrelated deployment actions.
**Exam Takeaway:** `npm ci` is standard practice for reliable, reproducible container builds in Node.js apps.

### Question 7
When executing the command `docker build -t my-app:1.0 .`, what does the final dot (`.`) represent?
- A. The termination of the command string.
- B. The root administrative password for the container.
- C. The build context path indicating the current directory where the Dockerfile resides.
- D. The public IP address of the container registry.

**Correct Answer:** C
**Explanation:** The dot (`.`) specifies the build context—the path on the host filesystem where Docker looks for the Dockerfile and application assets.
**Incorrect Options:** A, B, and D are factually incorrect definitions.
**Exam Takeaway:** Never forget the build context argument (`.`) when invoking `docker build`.

### Question 8
If a Dockerfile contains multiple `RUN` instructions, how does Docker treat them structurally during the build process?
- A. They are combined into a single volatile file that disappears after building.
- B. Each `RUN` instruction creates an independent filesystem layer in the final image.
- C. They are completely ignored unless prefixed with a `CMD` instruction.
- D. They cause an immediate syntax error due to concurrency rules.

**Correct Answer:** B
**Explanation:** Every instruction (`FROM`, `RUN`, `COPY`) in a Dockerfile creates a new read-only layer in the resulting container image stack.
**Incorrect Options:** A, C, and D misrepresent how Docker handles layer architecture.
**Exam Takeaway:** Understanding image layers helps optimize Dockerfile caching and build speeds.

### Question 9
Why are lightweight base distributions like Alpine preferred over standard OS builds when deploying microservices to Google Cloud Run?
- A. They automatically scale CPU allocations based on traffic spikes.
- B. They decrease image payload sizes, improving deployment and cold-start speeds.
- C. They provide built-in native support for Google Cloud SQL proxies.
- D. They bypass security vulnerability compliance scans automatically.

**Correct Answer:** B
**Explanation:** Smaller image sizes reduce download/transfer overhead, which directly decreases container cold-start latency on serverless platforms like Cloud Run.
**Incorrect Options:** A, C, and D are handled by platform architecture or external mechanisms, not base image selection.
**Exam Takeaway:** Image size minimization directly affects performance and container cold starts on GCP serverless services.

### Question 10
Which instruction dictates the command that runs by default when a container is instantiated from a built image?
- A. `FROM`
- B. `COPY`
- C. `CMD`
- D. `WORKDIR`

**Correct Answer:** C
**Explanation:** The `CMD` instruction sets the default command executed when running a container instance from the image.
**Incorrect Options:** A sets base images, B copies files, and C sets paths.
**Exam Takeaway:** Use `CMD` to supply the default execution payload for your container instances.