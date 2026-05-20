# GCP Associate Cloud Engineer - VM Automation: Startup Scripts & Bootstrapping

## Overview
Manually configuring every Virtual Machine (VM) is inefficient and prone to human error. In this section, we transition from manual configuration to **Automation**. We introduce the concept of **Bootstrapping** using **Startup Scripts**. This allows a VM to automatically install software, apply security patches, and configure web services the moment it boots up, requiring zero manual intervention after the "Create" button is clicked.

---

## 🎯 ACE Exam Focus

### Frequently Tested Areas
*   **Bootstrapping:** The process of installing software/updates automatically during the VM launch.
*   **Startup Script Location:** Found under the `Management` section during VM creation.
*   **Privileges:** Startup scripts run as **root** (no need to type `sudo` inside the script).
*   **Execution Timing:** They run every time the instance **boots** or **restarts**.
*   **The `-y` Flag:** Crucial for automation; it prevents the script from hanging by automatically answering "Yes" to prompts.

### Common Exam Trap
❌ **Assuming the script runs every time you log in.**
*   ✔ **Reality:** The script runs only when the VM **boots** (starts up) or **restarts**. If you just log in via SSH, it doesn't trigger again.

---

## 1. What is Bootstrapping?

**Bootstrapping** is the automated process of preparing a server to do its job. 
*   **Security:** Automatically applying the latest OS patches to ensure the VM is secure from day one.
*   **Software:** Installing runtimes (Java, Python), web servers (Apache, Nginx), or database clients.
*   **Consistency:** Ensuring every VM in a group is configured exactly the same way.

---

## 2. Option 1: Startup Scripts

A **Startup Script** is a file or a block of code provided to GCP that the VM executes during its initialization phase.

### Key Characteristics:
| Feature | Details |
| :--- | :--- |
| **User** | Runs as **Root**. |
| **Language** | Usually Bash (for Linux) or PowerShell (for Windows). |
| **Storage** | Can be pasted directly in the console or stored in **Cloud Storage (GCS)**. |
| **Reliability** | If the script fails, the VM will still start, but the software won't be there. |

---

## 3. Step-by-Step Workflow: Creating an Automated VM

1.  **Start Creation:** Go to `Compute Engine` > `VM Instances` > `Create`.
2.  **Configuration:** Name the VM (e.g., `vm-automated-apache`).
3.  **Firewall:** Check **"Allow HTTP traffic"** (Essential for web servers).
4.  **Access Management Tab:**
    *   Click the dropdown: **"Advanced Options"**.
    *   Select **"Management"**.
5.  **Add Script:** Scroll to the **"Startup script"** text box and paste your code.
6.  **Launch:** Click **Create**.
7.  **Wait:** Give the VM 1–3 minutes after it shows the green checkmark (it needs time to finish the background installation).

---

## 4. The Automation Script (Code Block)

This script automates everything we previously did manually.

```bash
#!/bin/bash
# 1. Refresh the package list
apt update 

# 2. Install Apache with the -y flag to avoid manual prompts
apt -y install apache2

# 3. Create a custom index page using the VM's hostname and IP
echo "Hello world from $(hostname) $(hostname -I)" > /var/www/html/index.html
```

### 🚩 Flag Breakdown:
*   `#!/bin/bash`: Tells the VM to use the Bash interpreter.
*   `-y`: Stands for **"Assume Yes."** Without this, the script stops and waits for a user to press 'Y', causing the automation to hang forever.
*   `$(hostname -I)`: Fetches the internal IP address.

---

## 5. Visual Representations

### Automation Lifecycle
```text
[ User Clicks Create ]
          |
          V
[ GCP Allocates Hardware ]
          |
          V
[ OS Starts Booting ]
          |
          V
[ Startup Script Executes (Root) ] ----> [ apt update ]
          |                        ----> [ apt install apache2 ]
          |                        ----> [ Create index.html ]
          V
[ VM Ready & Web Server Online ]
```

### The Three Automation Options (Comparison)
*This section introduces the concepts discussed in the next steps.*

| Method | Speed of Boot | Complexity | Best For... |
| :--- | :--- | :--- | :--- |
| **Startup Script** | Slower (Installs on boot) | Low | Dynamic updates, security patches. |
| **Instance Template** | Medium | Medium | Scaling groups of identical VMs. |
| **Custom Image** | **Fastest** (Pre-installed) | High | Massive scaling where every second counts. |

---

## 6. Mental Models & Analogies

### Analogy: The "Auto-Cook" Kitchen
*   **Manual Install (Previous Step):** You buy a kitchen, then you walk in and manually turn on the stove, chop the onions, and cook the meal.
*   **Startup Script (This Step):** You buy a kitchen and leave a **"Recipe Card"** (Startup Script) on the counter. The moment the kitchen opens, an automated robot (Bootstrapping) reads the card and cooks the meal for you before you even arrive.

---

## 7. Important Notes & Observations

*   **⚠️ The "Silent" Failure:** If your script has a typo, GCP won't tell you "Creation Failed." The VM will turn green, but the website won't work. You must check the **Serial Port Logs** to debug startup script errors.
*   **External IP:** Even with a script, you still need to use the **External IP** to view the result in your browser.
*   **Permissions:** You don't need to add `sudo` to commands in the startup script box; GCP runs these as the highest-level user automatically.

---

## 8. Common Mistakes & Exam Traps

### ❌ Mistake 1: Forgetting the `-y`
*   **Problem:** Your script has `apt install apache2`. 
*   **Result:** The VM waits for a "Yes" that never comes. The web server never gets installed.

### ❌ Mistake 2: Forgetting the Firewall
*   **Problem:** The script runs perfectly, but you didn't check "Allow HTTP Traffic."
*   **Result:** The web server is running, but Google's network blocks everyone from seeing it.

---

## 🎯 10 Certification Practice Questions

**Q1. What is the primary purpose of "Bootstrapping" in GCP?**
A. To increase the cost of a VM.
B. To manually configure a VM via SSH.
C. To automatically install software and patches during instance launch.
D. To delete a VM after it finishes a task.
*   **✅ Answer: C**

**Q2. Where in the VM creation console is the field for the Startup Script located?**
A. Under the "Networking" tab.
B. Under "Advanced Options" > "Management".
C. In the "Machine Type" dropdown.
D. Under the "Security" tab only.
*   **✅ Answer: B**

**Q3. A Startup Script runs with which user privileges by default?**
A. The user who created the VM.
B. Guest.
C. Root (Superuser).
D. Service Account.
*   **✅ Answer: C**

**Q4. Why is the `-y` flag essential when using `apt install` in a startup script?**
A. It speeds up the internet connection.
B. It makes the installation "Yield" to the OS.
C. It prevents the script from stopping to ask for user confirmation.
D. It ensures the VM is in the "Yellow" health state.
*   **✅ Answer: C**

**Q5. You updated your startup script to install a new version of a software. When will this change take effect on an ALREADY running VM?**
A. Immediately.
B. Never; startup scripts only run on the first boot.
C. The next time the instance is restarted or rebooted.
D. Every 24 hours automatically.
*   **✅ Answer: C**

**Q6. You want to ensure your VMs always have the latest security patches when they are created. What is the most efficient way to do this without manual intervention?**
A. Use a Startup Script with `apt update` and `apt upgrade`.
B. Log in to every VM via SSH after creation.
C. Use a smaller machine type.
D. Disable the firewall.
*   **✅ Answer: A**

**Q7. What happens if a startup script contains an error?**
A. The VM fails to create and returns an error.
B. The VM creates successfully, but the script tasks are not completed.
C. The VM will keep rebooting in a loop.
D. GCP automatically fixes the script error.
*   **✅ Answer: B** (This makes debugging tricky!)

**Q8. Which of these can be performed by a startup script? (Select all that apply)**
A. Installing a web server.
B. Applying OS patches.
C. Setting up environment variables.
D. Deleting the GCP Project.
*   **✅ Answer: A, B, and C**

**Q9. Which symbol is used in Bash to redirect the output of a command to a file, overwriting its contents?**
A. `>>`
B. `<`
C. `>`
D. `|`
*   **✅ Answer: C**

**Q10. True or False: Startup scripts can only be used for Linux VMs.**
A. True
B. False
*   **✅ Answer: B** (Windows VMs use PowerShell startup scripts).

---

## ⚡ 30-Second Revision
*   **Startup Script:** Automation via code on boot.
*   **Bootstrapping:** The act of "self-starting" a server's config.
*   **Location:** Management tab.
*   **Privilege:** Always Root.
*   **Pro Tip:** Always use `-y` for package installs.
*   **Use Case:** Ideal for installing Apache or updating security patches automatically. ☁️