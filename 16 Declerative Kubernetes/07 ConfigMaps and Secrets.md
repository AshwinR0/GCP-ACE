### 1. Overview
Kubernetes **ConfigMaps** and **Secrets** are built-in API objects used to decouple configuration artifacts and sensitive data from container image content. 

- **ConfigMaps** store non-confidential configuration parameters (such as database hosts, database ports, names, or non-secret URLs) in plain text.
- **Secrets** are designed to store sensitive, confidential information (such as passwords, API keys, OAuth tokens, and TLS certificates). While Kubernetes Secrets use **Base64 encoding** by default, they are distinct from plain text ConfigMaps and can be integrated with encryption-at-rest features.

Both ConfigMaps and Secrets can be easily injected into application containers at runtime as **environment variables** using `configMapKeyRef` and `secretKeyRef`.

---

### 2. ACE Exam Focus & Common Exam Traps

*   **Exam Importance:** **HIGH**. The ACE exam frequently tests the differences between ConfigMaps and Secrets, security best practices, and how to inject their values into containers using environment variables.
*   **High-Priority Topics:**
    *   **ConfigMap Use Case:** Non-sensitive configuration data stored in plain text (e.g., ports, hostnames, configuration flags).
    *   **Secret Use Case:** Sensitive data like passwords, API keys, tokens, and certificates.
    *   **Encoding vs. Encryption:** Base64 encoding is *not* strong encryption; it is simply an encoding scheme. (Note: GKE supports application-layer secret encryption-at-rest using Cloud KMS keys).
    *   **Environment Variable References:** 
        *   ConfigMaps use `configMapKeyRef`.
        *   Secrets use `secretKeyRef`.
*   **Common Exam Traps:**
    *   Believing that Base64 encoding makes a Kubernetes Secret fully secure/encrypted by default. (Base64 can be easily decoded by anyone with read access to the cluster object).
    *   Using ConfigMaps to store database passwords or API keys instead of Secrets.

---

### 3. Detailed Concept Explanation

#### A. ConfigMaps
*   **Purpose:** Store non-confidential key-value pairs of configuration data.
*   **Format:** Stored as human-readable plain text.
*   **Examples:** `db-host=my-db-service`, `db-port=5432`, `db-name=inventory`.

#### B. Secrets
*   **Purpose:** Store sensitive, confidential data.
*   **Format:** Uses **Base64 encoding** for value obfuscation in manifest definitions.
*   **Examples:** Database passwords, private TLS keys, API authentication tokens.

#### C. Injecting ConfigMaps and Secrets as Environment Variables
Containers can read properties from ConfigMaps and Secrets dynamically at runtime by referencing them inside the container specification block:
1.  **ConfigMap Integration:** Uses `configMapKeyRef` to point to a specific ConfigMap name and its designated key.
2.  **Secret Integration:** Uses `secretKeyRef` to point to a specific Secret name and its designated confidential key.

---

### 4. Practical Setup & Implementation Guide

#### Step 1: Creating a ConfigMap and a Secret Manifest
Create a file named `config-and-secrets.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: db-config
data:
  db-host: "my-db-service"
  db-port: "5432"
  db-name: "inventory"
  db-user: "admin"
---
apiVersion: v1
kind: Secret
metadata:
  name: my-app-secret
type: Opaque
data:
  # Note: Values must be Base64 encoded (e.g., 'P@ssword123' encoded is 'UEBzc3dvcmQxMjM=')
  my-app-db-password: "UEBzc3dvcmQxMjM="
```

#### Step 2: Referencing ConfigMap and Secret Values in a Deployment Manifest
Create a file named `deployment-with-env.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-rest-api-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-world-rest-api
  template:
    metadata:
      labels:
        app: hello-world-rest-api
    spec:
      containers:
      - name: hello-world-rest-api
        image: gcr.io/google-samples/hello-app:1.0
        ports:
        - containerPort: 8080
        env:
        # Injecting values from ConfigMap using configMapKeyRef
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: db-config
              key: db-host
        - name: DB_PORT
          valueFrom:
            configMapKeyRef:
              name: db-config
              key: db-port
        # Injecting sensitive values from Secret using secretKeyRef
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: my-app-secret
              key: my-app-db-password
```

#### Step 3: Applying Configurations
Deploy the ConfigMap, Secret, and Deployment to your cluster:
```bash
kubectl apply -f config-and-secrets.yaml
kubectl apply -f deployment-with-env.yaml
```

#### Step 4: Verification
Verify that environment variables are correctly injected into the running container:
```bash
kubectl exec -it <pod-name> -- env
```

#### Step 5: Cleanup
Delete the deployed resources:
```bash
kubectl delete -f deployment-with-env.yaml
kubectl delete -f config-and-secrets.yaml
```

---

### 5. Visual Representations

#### ConfigMap & Secret Injection Architecture
```text
+-----------------------+         +-----------------------+
|  ConfigMap: db-config |         | Secret: my-app-secret |
|  - db-host: db-svc    |         |  - password: [Base64] |
+-----------------------+         +-----------------------+
            │                                 │
     (configMapKeyRef)                 (secretKeyRef)
            │                                 │
            v                                 v
+---------------------------------------------------------+
|                  Container Environment                  |
|          DB_HOST = my-db-service                        |
|          DB_PASSWORD = *********                        |
+---------------------------------------------------------+
```
*Explanation:* ConfigMaps and Secrets store configuration items separately and inject them securely into container environments via explicit key references (`configMapKeyRef` and `secretKeyRef`).

---

### 6. Mental Models & Analogies

*   **The Analogy:** **A Public Bulletin Board vs. A Locked Safe**
*   **The Mapping:**
    *   *ConfigMap* = A public office bulletin board listing general guidelines, office location details, and break room schedules in plain text for everyone to see.
    *   *Secret* = A private combination safe in the manager's office used to store sensitive keys, door codes, and master passwords (scrambled or encoded so they aren't written raw on paper).

---

### 7. Step-by-Step Workflows / Comparison Tables

#### Comparison Table: ConfigMaps vs. Secrets

| Feature | ConfigMaps (`ConfigMap`) | Secrets (`Secret`) |
| :--- | :--- | :--- |
| **Primary Purpose** | Storing non-sensitive configuration parameters. | Storing sensitive confidential data (passwords, tokens, keys). |
| **Data Format** | Plain text string format. | **Base64 encoded** string format. |
| **Security Level** | Low (readable by anyone with object read access). | Medium/High (Base64 encoded by default; supports encryption-at-rest in GKE). |
| **Injection Reference** | `configMapKeyRef` | `secretKeyRef` |
| **Common Examples** | Hostnames, ports, environment flags (`prod`/`dev`). | Database passwords, API authentication keys, TLS certificates. |

---

### 8. Common Mistakes & Exam Traps

*   *Trap:* Assuming that Base64 encoding in Kubernetes Secrets provides robust encryption.
    *   *Correction:* Base64 is an *encoding* format, not encryption. Anyone who can read the Kubernetes Secret object can decode Base64 instantly. True data protection requires enabling GKE secret encryption-at-rest with Cloud KMS keys or utilizing proper RBAC restrictions.
*   *Trap:* Using `configMapKeyRef` to pull passwords or confidential API keys.
    *   *Correction:* Always use `secretKeyRef` for sensitive credentials.

---

### 9. Certification Practice Questions

#### Question 1
You need to store a non-sensitive database port number and a database hostname for your application configuration. Which Kubernetes object should you use?
*   A) Secret
*   B) ConfigMap
*   C) PersistentVolume
*   D) ServiceAccount

**Correct Answer:** **B**
*Explanation:* ConfigMaps are specifically designed to store non-confidential configuration settings like hostnames and ports in plain text.
*Incorrect Options:* A is meant for sensitive credentials. C handles storage volumes. D handles cluster identity permissions.
*Exam Takeaway:* Use **ConfigMaps** for non-sensitive data and **Secrets** for sensitive credentials.

---

#### Question 2
How are values stored inside a standard Kubernetes Secret object by default in its YAML manifest definition?
*   A) Encrypted using AES-256 military-grade encryption
*   B) Base64 encoded
*   C) Saved as raw unencoded plain text
*   D) Hashed using SHA-512 algorithms with a salt

**Correct Answer:** **B**
*Explanation:* Kubernetes Secrets store values encoded using Base64 by default (which obfuscates plain text view but is easily decoded).
*Incorrect Options:* A requires explicit cluster-level KMS configuration. C describes ConfigMaps. D is incorrect.
*Exam Takeaway:* Kubernetes Secrets use **Base64 encoding** by default.

---

#### Question 3
An application container needs to read a database password stored inside a Kubernetes Secret object named `my-app-secret` as an environment variable. Which configuration reference syntax must be used in the deployment spec?
*   A) `configMapKeyRef`
*   B) `secretKeyRef`
*   C) `envFromConfig`
*   D) `volumeMountSecret`

**Correct Answer:** **B**
*Explanation:* To inject values from a Secret into an environment variable, you use `secretKeyRef` specifying the secret name and key.
*Incorrect Options:* A is used for ConfigMaps. C and D are invalid or alternative structures.
*Exam Takeaway:* Use `secretKeyRef` for Secrets and `configMapKeyRef` for ConfigMaps.

---

#### Question 4
True or False: Base64 encoding provides robust cryptographic encryption for Kubernetes Secrets out-of-the-box.
*   A) True
*   B) False

**Correct Answer:** **B**
*Explanation:* False. Base64 is an encoding scheme, not encryption. Anyone with read access to the cluster API can decode Base64 values instantly.
*Exam Takeaway:* Base64 encoding is not encryption; rely on GKE encryption-at-rest and RBAC to secure secrets.

---

#### Question 5
Which of the following data points is best suited to be stored inside a Kubernetes ConfigMap rather than a Secret?
*   A) TLS private certificate key
*   B) OAuth client secret token
*   C) Application log level (`INFO` or `DEBUG`)
*   D) Production database root password

**Correct Answer:** **C**
*Explanation:* Log levels are non-sensitive configuration settings, making them ideal for a ConfigMap.
*Incorrect Options:* A, B, and D contain sensitive credentials that must be stored in Secrets.
*Exam Takeaway:* Non-sensitive operational flags belong in ConfigMaps.

---

#### Question 6
When configuring a container environment variable to pull a host URL from a ConfigMap named `db-config`, which reference parameter matches standard syntax?
*   A) `configMapKeyRef`
*   B) `secretKeyRef`
*   C) `configRef`
*   D) `mapValueRef`

**Correct Answer:** **A**
*Explanation:* `configMapKeyRef` is the correct reference block used to pull keys out of a ConfigMap into environment variables.
*Incorrect Options:* B is for secrets. C and D are invalid keys.
*Exam Takeaway:* ConfigMap environment injection uses `configMapKeyRef`.

---

#### Question 7
An auditor finds that plain text passwords stored in a ConfigMap are visible to anyone with basic read permissions to the namespace. What architectural mistake was made?
*   A) The ConfigMap was not encrypted with a KMS key.
*   B) Sensitive confidential data was stored in a ConfigMap instead of a Secret.
*   C) The container image was built incorrectly.
*   D) The service account lacked RBAC roles.

**Correct Answer:** **B**
*Explanation:* ConfigMaps store configuration data in unencrypted plain text and should never be used to store passwords or credentials. Secrets should be used instead.
*Incorrect Options:* A, C, and D do not address the primary design flaw of storing credentials in a ConfigMap.
*Exam Takeaway:* Never store passwords or tokens in ConfigMaps.

---

#### Question 8
Which GKE feature can be configured to protect Kubernetes Secret data at rest in the cluster's etcd database using customer-managed encryption keys?
*   A) Cloud KMS (Key Management Service) integration
*   B) Cloud NAT gateways
*   C) VPC Service Controls
*   D) Google Cloud Load Balancing

**Correct Answer:** **A**
*Explanation:* GKE supports application-layer secret encryption using Cloud KMS keys to encrypt secrets at rest in etcd.
*Incorrect Options:* B, C, and D handle networking, security perimeters, and traffic routing, not secret encryption at rest.
*Exam Takeaway:* Use Cloud KMS with GKE for enhanced secret encryption at rest.

---

#### Question 9
If an application container environment variable references a key in a Secret object, but that key does not exist inside the target Secret, what happens when the pod tries to start?
*   A) The pod starts successfully, and the environment variable is left blank.
*   B) The pod fails to start and remains in a `CreateContainerConfigError` or `CrashLoopBackOff` state.
*   C) Kubernetes automatically generates a random placeholder value for the missing key.
*   D) The cluster downgrades the Secret to a ConfigMap.

**Correct Answer:** **B**
*Explanation:* Missing configuration keys referenced by `configMapKeyRef` or `secretKeyRef` cause container creation errors (`CreateContainerConfigError`).
*Incorrect Options:* A, C, and D are false; Kubernetes does not guess missing values or bypass missing references.
*Exam Takeaway:* Missing configuration keys cause container configuration errors on startup.

---

#### Question 10
Can a single container read environment variables from *both* a ConfigMap and a Secret simultaneously?
*   A) No, a container can only reference configuration sources from one object type at a time.
*   B) Yes, a container's `env` block can combine multiple entries referencing both ConfigMaps (`configMapKeyRef`) and Secrets (`secretKeyRef`).

**Correct Answer:** **B**
*Explanation:* Containers can mix and match multiple environment variable sources freely within their container spec definitions.
*Exam Takeaway:* Containers can consume environment variables from multiple ConfigMaps and Secrets concurrently.