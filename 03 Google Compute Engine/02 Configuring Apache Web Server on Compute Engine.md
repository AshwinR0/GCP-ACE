# GCP Associate Cloud Engineer - Configuring Apache Web Server on Compute Engine

## Overview
This section demonstrates the practical "Day 1" operations of a Cloud Engineer: logging into a VM, installing software, and serving web traffic. We cover how to transform a raw Compute Engine instance into a functional **Apache HTTP Server**, verify connectivity via the External IP, and use Linux shell commands to customize web content dynamically.

For the ACE exam, this illustrates the **IaaS (Infrastructure as a Service)** model where you are responsible for the OS configuration, software installation, and runtime environment.

---

## 🎯 ACE Exam Focus

### Frequently Tested Areas
*   **Firewall Rules:** Understanding that Port 80 must be open to see the Apache page.
*   **IP Addresses:** Distinguishing between the **External IP** (used by the browser) and the **Internal IP** (displayed via command line).
*   **Superuser Access:** Using `sudo` to perform administrative tasks.
*   **Persistence:** Knowing that manual changes to a VM (like installing Apache) will be lost if the VM is deleted, unless a **Custom Image** or **Startup Script** is used.

### Common Exam Trap
❌ **Trying to reach the web server via Internal IP from your home computer.**
*   ✔ **Correct:** Internal IPs are only routable within the VPC. You must use the **External IP** to access the server from the public internet.

---

## 1. Step-by-Step Workflow: Installing Apache

### Phase 1: Elevating Privileges
To install software, you need administrative (root) permissions.
```bash
sudo su
```
*   `sudo`: "SuperUser Do"
*   `su`: "Switch User" (defaults to root)

### Phase 2: Package Management
GCP's default Debian image uses `apt` (Advanced Package Tool).
1.  **Update the Index:** Synchronizes the local list of available packages with the online repositories.
    ```bash
    apt update
    ```
2.  **Install Apache:**
    ```bash
    apt install apache2 -y
    ```
    *   `apache2` is the package name.
    *   `-y` (optional) automatically answers "Yes" to the disk space prompt.

### Phase 3: Verification
1.  Go to the **GCP Console** > **Compute Engine** > **VM Instances**.
2.  Locate your VM and find the **External IP** column.
3.  Click the link or paste the IP into a browser. You should see the "Apache2 Debian Default Page."

---

## 2. Customizing the Web Server

Apache serves files from a "Document Root." In Debian, this is `/var/www/html/`.

### Commands for Content Customization
| Command | Purpose |
| :--- | :--- |
| `$(hostname)` | Returns the name of the VM (e.g., `my-first-vm`). |
| `$(hostname -i)` | Returns the **Internal IP** address of the VM. |
| `>` (Pipe/Redirect) | Overwrites a file with the output of a command. |

### Creating a Dynamic "Hello World" Page
To display the VM name and Internal IP on the website:
```bash
echo "Hello world from $(hostname) $(hostname -i)" > /var/www/html/index.html
```
*   **Result:** When you refresh the External IP in your browser, it will now show: `Hello world from my-first-vm 10.128.0.2`.

---

## 3. Visual Representations

### Web Request Architecture
```text
[ User Browser ] 
       |
       | Request (External IP: 34.x.x.x)
       V
[ GCP Firewall ] --- (Checks if Port 80 is ALLOWED)
       |
       V
[ Compute Engine VM ]
       |
       |--- [ Apache Service ]
       |           |
       |           |--- [ Reads /var/www/html/index.html ]
       |
       V
[ HTTP Response Sent to User ]
```

### IAM & Permission Mental Model
```text
[ Your User Account ] -> [ IAM: Compute Admin Role ] -> [ SSH Access ]
                                                               |
                                                               V
[ Inside Linux OS ] -> [ sudo su ] -> [ Full Root Control ]
```

---

## 4. Mental Models & Analogies

### Analogy: The Restaurant
*   **Compute Engine:** The kitchen building.
*   **Apache:** The chef you hired to serve food (web pages).
*   **APT:** The delivery service (like UberEats for software) that brings the chef’s tools to the kitchen.
*   **Index.html:** The specific menu you want the chef to show the customers.
*   **Firewall:** The front door. Even if the chef is ready, if the door is locked (Port 80 closed), customers can't eat.

---

## 5. Important Commands Summary

```bash
# Become root user
sudo su

# Update package lists
apt update

# Install Apache HTTP Server
apt install apache2

# View the default index file
ls /var/www/html

# Overwrite the index file with custom text
echo "Hello World" > /var/www/html/index.html

# Check server name and IP
hostname
hostname -i

# Ensure service is running
sudo service apache2 start
```

---

## 6. Common Mistakes & Exam Traps

### ❌ Mistake 1: Permission Denied
*   **Symptom:** Running `apt update` results in "Permission denied."
*   **Fix:** You forgot to run `sudo su` or prefix the command with `sudo`.

### ❌ Mistake 2: Site Not Loading (Timeout)
*   **Symptom:** Browser spins forever when clicking the External IP.
*   **Fix:** Ensure the VM has the **"Allow HTTP Traffic"** checkbox enabled (which creates a VPC Firewall rule for Port 80).

### ❌ Mistake 3: Confusing `>` and `>>`
*   **`>`**: Overwrites the file (Deletes old content).
*   **`>>`**: Appends to the file (Adds to the bottom).

---

## 🎯 10 Certification Practice Questions

**Q1. You have installed Apache on a VM, but cannot access the test page via the External IP. What is the first thing you should check in GCP?**
A. If the VM has a GPU.
B. If the VPC Firewall allows traffic on Port 80.
C. If the Internal IP is static.
D. If the VM is in the `us-central1` region.
*   **✅ Answer: B**

**Q2. Which Linux command allows you to switch to the root user to perform administrative tasks?**
A. `apt update`
B. `hostname -i`
C. `sudo su`
D. `ls /var/www/html`
*   **✅ Answer: C**

**Q3. In which directory does Apache look for the default `index.html` file on a Debian system?**
A. `/etc/apache2`
B. `/home/user/`
C. `/var/www/html`
D. `/root/www`
*   **✅ Answer: C**

**Q4. You want to display the internal IP of your VM on your webpage. Which command provides this?**
A. `hostname`
B. `hostname -i`
C. `cat /etc/os-release`
D. `echo $USER`
*   **✅ Answer: B**

**Q5. What is the purpose of running `apt update` before `apt install apache2`?**
A. To restart the VM.
B. To download the Apache software.
C. To refresh the local database of available software versions.
D. To open Port 80 in the firewall.
*   **✅ Answer: C**

**Q6. You use the command `echo "Welcome" > index.html`. What happens to the previous content of `index.html`?**
A. It is preserved.
B. It is moved to a backup file.
C. It is overwritten and lost.
D. The command fails if the file is not empty.
*   **✅ Answer: C**

**Q7. A developer says they can access the web server using the internal IP from another VM in the same VPC, but you cannot access it from your laptop. Why?**
A. Internal IPs are only for communication inside the Google network.
B. The developer has a better internet connection.
C. Apache only works on internal networks.
D. The external IP is disabled by default.
*   **✅ Answer: A**

**Q8. Which of the following is an example of an Internal IP in a standard GCP VPC?**
A. `34.120.5.10`
B. `10.128.0.2`
C. `172.217.12.14`
*   **✅ Answer: B** (10.x.x.x is a private/internal range).

**Q9. What does the `-y` flag do in `apt install apache2 -y`?**
A. It stands for "Yesterday," installing the previous version.
B. It automatically confirms the installation prompt.
C. It makes the installation "Yield" to other processes.
D. It restarts the VM after installation.
*   **✅ Answer: B**

**Q10. How can you find the External IP of your VM without using the command line?**
A. Look at the VM details in the GCP Console.
B. Check your billing statement.
C. It is the same as the VM's name.
D. Ping the VM from your local computer.
*   **✅ Answer: A**

---

## ⚡ 30-Second Revision
*   **Sudo su:** Become the boss (root).
*   **APT:** The software installer.
*   **Apache:** The web server service.
*   **Port 80:** The "door" for HTTP traffic.
*   **External IP:** How the world sees your VM.
*   **Internal IP:** How the VM sees itself inside GCP.
*   **`/var/www/html/`:** The home of your website files. ☁️