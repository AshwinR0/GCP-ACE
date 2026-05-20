# GCP Associate Cloud Engineer - Networking: Internal, External, and Static IP Addresses

## Overview
Networking is a core pillar of the GCP Associate Cloud Engineer exam. This section explores how Virtual Machine (VM) instances communicate with the world and with each other. We focus on the two types of IP addresses (**Internal** and **External**) and the two assignment behaviors (**Ephemeral** and **Static**). Understanding these is critical for maintaining stable application connectivity and controlling costs.

---

## 🎯 ACE Exam Focus

### Frequently Tested Areas
*   **IP Lifecycles:** Knowing that Ephemeral External IPs change when an instance is stopped and started.
*   **Static IP Reservation:** How to ensure a VM keeps the same IP address forever.
*   **Unused IP Billing:** The specific (and often counter-intuitive) billing rules for static IPs.
*   **Internal vs. External Scope:** Knowing that Internal IPs are not reachable from the public internet.
*   **VPC Reachability:** Understanding that internal IPs are unique to a network but can overlap across different corporate networks.

### Common Exam Trap
❌ **Believing Static IPs are free when the VM is turned off.**
*   ✔ **Reality:** Google actually charges **more** for an unused static IP than one that is actively attached to a running VM. This is to prevent "IP hoarding."

---

## 1. Internal vs. External IP Addresses

| Feature | Internal IP | External IP |
| :--- | :--- | :--- |
| **Reachability** | Only inside the GCP/Corporate Network. | Reachable from the public Internet. |
| **Uniqueness** | Unique within the VPC, but can be used in other separate networks. | Must be globally unique across the entire Internet. |
| **Default** | Every VM gets at least one. | Optional; chosen during or after creation. |
| **Stop/Start Behavior** | **Persists.** Remains the same when VM is restarted. | **Lost** (if Ephemeral). Changes upon restart. |

---

## 2. Ephemeral vs. Static IP Addresses

### Ephemeral IP (The Default)
*   **Meaning:** "Short-lived" or "Temporary."
*   **Behavior:** When you stop a VM, the external IP is released back into Google's pool. When you start the VM again, you get a **new, different** external IP.
*   **Problem:** If you pointed a domain name (like `www.myapp.com`) to that IP, the site will break when the VM restarts.

### Static IP (The Solution)
*   **Meaning:** "Fixed" or "Permanent."
*   **Behavior:** You "reserve" an IP address from Google. It belongs to your project until you manually release it.
*   **Flexibility:** You can move a static IP from one VM to another within the same project and region.

---

## 3. Step-by-Step Workflow: Reserving a Static IP

1.  **Navigate:** Go to `VPC Network` > `IP addresses` (or search "External IP addresses").
2.  **Reserve:** Click **Reserve Static Address**.
3.  **Configure:**
    *   **Name:** Give it a clear name (e.g., `web-server-static-ip`).
    *   **Network Tier:** Choose **Premium** (Google's private fiber) or **Standard**.
    *   **IP Version:** IPv4 (most common).
    *   **Type:** Regional (for VMs).
4.  **Attach:** Use the **Change** button to select the VM instance you want to associate with this IP.
5.  **Verify:** The previous Ephemeral IP is automatically released, and the new Static IP is now active.

---

## 4. Visual Representations

### IP Visibility Map
```text
      [ Public Internet ]
              |
      ( External IP: 34.x.x.x ) <--- Addressable by World
              |
      [ GCP VPC Network ]
              |
      ( Internal IP: 10.x.x.x ) <--- Addressable only by other GCP resources
              |
      [ VM Instance ]
```

### The Restart Behavior (Ephemeral)
```text
[ VM Running ]  --> IP: 34.123.6.112
[ VM Stopped ]  --> IP: NONE
[ VM Started ]  --> IP: 35.190.2.45 (CHANGED!)
```

### The Restart Behavior (Static)
```text
[ VM Running ]  --> IP: 34.200.1.1
[ VM Stopped ]  --> IP: 34.200.1.1 (RETAINED)
[ VM Started ]  --> IP: 34.200.1.1 (SAME)
```

---

## 5. Mental Models & Analogies

### Analogy: The Hotel Room vs. The House
*   **Ephemeral IP (Hotel Room):** You check in and get Room 101. You leave for a week (Stop VM). When you come back (Start VM), Room 101 is taken, so they give you Room 205. Your mail (traffic) won't find you.
*   **Static IP (Owned House):** You buy the address. Even if you go on vacation and the house is empty (Stop VM), the address stays yours. When you come back, your mail is still delivered to the same spot.

---

## 6. Important Notes & Best Practices

*   **⚠️ Billing Warning:** If you have a Static IP that is **not** attached to a running VM, you are billed at a higher rate. 
*   **Cleanup Rule:** When you delete a VM, the Static IP **remains** in your project. You must go to the External IP Addresses page and click **Release Static IP** to stop being charged.
*   **Internal Consistency:** Note that the Internal IP (e.g., `10.128.0.2`) typically stays the same for the life of the VM, even through stops/starts.

---

## 7. Common Mistakes & Exam Traps

### ❌ Mistake 1: Ignoring the "Release" Step
*   **Scenario:** You delete your VM but forget the Static IP.
*   **Result:** You get a surprise bill at the end of the month for "Unused Static IP."

### ❌ Mistake 2: Using Internal IPs for Public Webhooks
*   **Scenario:** Giving an external partner your `10.x.x.x` address.
*   **Result:** It will never work; internal IPs are invisible to the public internet.

---

## 🎯 10 Certification Practice Questions

**Q1. What happens to an Ephemeral External IP address when a VM instance is stopped and then started?**
A. It remains the same.
B. It is permanently deleted and cannot be replaced.
C. It changes to a new IP address.
D. It converts into an Internal IP.
*   **✅ Answer: C**

**Q2. You need to ensure that your VM instance maintains the same External IP address even after being deleted and recreated. What should you use?**
A. Ephemeral External IP
B. Static Internal IP
C. Reserved Static External IP
D. A Label
*   **✅ Answer: C**

**Q3. Which of the following is true regarding Internal IP addresses?**
A. They are globally unique across the internet.
B. They stay the same when an instance is stopped and started.
C. They are reachable from your home computer without a VPN/Proxy.
D. They are only assigned to Premium Tier networks.
*   **✅ Answer: B**

**Q4. You have a Static External IP that is NOT currently assigned to any running VM. How does Google bill you for this?**
A. You are not billed.
B. You are billed at a lower rate than if it were used.
C. You are billed at a higher rate than if it were used.
D. You are only billed if traffic is sent to it.
*   **✅ Answer: C** (To encourage releasing unused IPs).

**Q5. Can two different companies have the same Internal IP address (e.g., 10.128.0.2) in their respective GCP projects?**
A. No, all IPs in GCP must be globally unique.
B. Yes, because Internal IPs are private to their specific network.
*   **✅ Answer: B**

**Q6. Where in the GCP Console do you go to manage and release Static IP addresses?**
A. Compute Engine > VM Instances
B. IAM & Admin
C. VPC Network > IP addresses
D. Billing > Subscriptions
*   **✅ Answer: C**

**Q7. You want to point your company's domain (example.com) to a GCP VM. What is the best practice for the IP address?**
A. Use the default Ephemeral IP.
B. Use a Static External IP.
C. Use the Internal IP.
D. Use a different IP for every user.
*   **✅ Answer: B**

**Q8. A VM is stopped. In the console, you see an Internal IP but the External IP field is blank. What does this tell you?**
A. The VM never had an IP.
B. The VM was using an Ephemeral External IP.
C. The VM is using a Static External IP.
D. The firewall is blocking the IP.
*   **✅ Answer: B**

**Q9. True or False: You can move a reserved Static IP from a VM in `us-central1` to a VM in `asia-south1` if they are in the same project.**
A. True
B. False
*   **✅ Answer: B** (Standard static IPs are Regional).

**Q10. What is the best practice when you no longer need a project's VM and its associated static IP?**
A. Delete the VM and leave the IP for future use.
B. Delete the VM and explicitly release the Static IP.
C. Just stop the VM; it's cheaper.
D. Change the IP to Ephemeral before deleting.
*   **✅ Answer: B**

---

## ⚡ 30-Second Revision
*   **Internal IP:** Stays through restarts; Internal only; Default.
*   **External IP:** Changes through restarts (if Ephemeral); Public.
*   **Static IP:** Stays forever until you "Release" it.
*   **Billing Trap:** Unused Static IPs = Higher Cost.
*   **Location:** Managed under **VPC Network**. ☁️