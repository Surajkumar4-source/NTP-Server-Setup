# 🕒 Time Synchronization Between Virtual Machines Using Chrony 
## 📘 Introduction

Time synchronization is crucial for consistent system operations, especially in distributed systems and networked environments. When multiple virtual machines (VMs) interact—whether for logging, authentication, or monitoring—they must have a consistent and accurate system time.

This guide demonstrates how to set up **offline time synchronization** between **three virtual machines** using the **Chrony** time synchronization service. One VM will act as the **local NTP server**, and the others will act as **clients**, syncing their time from it. This setup **does not require internet access after configuration**.

---

## 🧠 Why Time Synchronization Matters

* 🗓️ **Accurate Logs**: Ensures logs across systems reflect the correct order of events.
* 🔐 **Authentication & Security**: Time-sensitive protocols (e.g., Kerberos) rely on synchronized clocks.
* 📈 **Monitoring Tools**: Metrics and dashboards need consistent timestamps.
* 🧪 **Testing & Simulations**: Helps maintain consistency when simulating multi-node environments.

---

## 🛠️ Tools Used: Chrony

### 🔹 What is Chrony?

Chrony is a versatile implementation of the Network Time Protocol (NTP). It is designed for:

* **Fast synchronization**
* **Better performance in virtual environments**
* **Ability to operate without a continuous network connection**

Compared to the older `ntpd`, Chrony is the recommended tool for modern systems.

---

## 🧭 Setup Overview

| VM      | Role       | IP Address Example |
| ------- | ---------- | ------------------ |
| **VM1** | NTP Server | `192.168.1.10`     |
| VM2     | NTP Client | `192.168.1.11`     |
| VM3     | NTP Client | `192.168.1.12`     |

All machines are assumed to be on the **same private network** (e.g., `192.168.1.0/24`).

---

## ✅ Configuration Steps

### 🔹 Step 1: Install Chrony (All VMs)

Run this on **VM1, VM2, and VM3**:

```bash
sudo apt update
sudo apt install chrony -y
```

---

### 🔹 Step 2: Configure VM1 as the NTP Server

1. Open the configuration file:

   ```bash
   sudo nano /etc/chrony/chrony.conf
   ```

2. Modify the following settings:

   * **Remove or comment out** existing `server` lines (internet servers).
   * Add this line to **allow the subnet** to access the server:

     ```bash
     allow 192.168.1.0/24
     ```
   * Optionally add a fallback to the **local hardware clock**:

     ```bash
     local stratum 10
     ```

3. Restart the chrony service:

   ```bash
   sudo systemctl restart chrony
   ```

4. Confirm the server is ready:

   ```bash
   chronyc sources
   ```

---

### 🔹 Step 3: Configure VM2 and VM3 as Clients

1. Open the configuration file on each client:

   ```bash
   sudo nano /etc/chrony/chrony.conf
   ```

2. Modify the config:

   * Remove or comment out the default `server` entries.
   * Add the IP of VM1 (the NTP server):

     ```bash
     server 192.168.1.10 iburst
     ```

3. Restart Chrony:

   ```bash
   sudo systemctl restart chrony
   ```

4. Verify synchronization:

   ```bash
   chronyc tracking
   ```

   or

   ```bash
   chronyc sources
   ```

---

### 🔍 Step 4: Confirm Synchronization

On VM2 or VM3, run:

```bash
timedatectl status
```

You should see:

```
System clock synchronized: yes
```

Or:

```bash
chronyc sources
```

You should see the IP of VM1 marked with an asterisk `*`, indicating that it is the **selected synchronization source**.

---

## 📌 Additional Notes

* Chrony works well even when clients are disconnected from the network for extended periods.
* The `iburst` option allows faster initial synchronization.
* This setup is ideal for **air-gapped labs**, **test environments**, or **cloud VMs with restricted internet access**.
* Replace `192.168.1.0/24` and `192.168.1.10` with your actual subnet and NTP server IP.

---

## ✅ Result

You now have a **fully offline, reliable time synchronization system** where:

* **VM1** acts as a local NTP server.
* **VM2 and VM3** stay in sync by pulling time from VM1.

No internet connection is needed after the initial setup.

---

