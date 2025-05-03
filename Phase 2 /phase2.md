
# 📊 ICS344 – Phase 2: Visual Analysis with a SIEM Dashboard

This phase aims to integrate logs from the attacker and/or victim environments into a SIEM tool and visualize/analyze those logs to better understand the attack activity.

---

## SIEM Setup – Splunk

As required in the project, we used **Splunk** as our SIEM platform and configured it on the attacker environment (Kali Linux).

---

###  Step 1: Downloading and Extracting Splunk

To begin the SIEM setup, we downloaded the official Splunk installer for Linux from Splunk’s website. We used the `.tgz` archive format suitable for manual extraction.

The following actions were performed:

1. **Downloaded the Splunk package** using `wget`, which fetched version `9.4.1` of the 64-bit Linux release.
2. **Moved the downloaded archive** to the `/opt` directory, which is commonly used for installing optional or third-party software.
3. **Extracted the archive** in `/opt` using the `tar` command, which created a directory containing all necessary Splunk files and folders.

These steps ensured Splunk was properly unpacked and ready for configuration within the attacker machine (Kali Linux), as required for SIEM dashboard setup.

```bash
wget https://download.splunk.com/products/splunk/releases/9.4.1/linux/splunk-9.4.1-83dbab203ac8-linux-amd64.tgz
sudo mv splunk-9.4.1-83dbab203ac8-linux-amd64.tgz /opt/
cd /opt/
sudo tar xvzf splunk-9.4.1-83dbab203ac8-linux-amd64.tgz
```

![Step 0](https://github.com/user-attachments/assets/ec06de2a-057f-429e-b6c8-7d7da21f0e10)

---

###  Step 2: First Launch of Splunk

We launched Splunk for the first time using:

```bash
cd /opt/splunk/bin
sudo ./splunk start --accept-license
```

It prompted for admin credentials, generated necessary keys, and verified port availability for the web UI (`http://kali:8000`).

![Step 1](https://github.com/user-attachments/assets/35471fbe-205b-49f5-b020-8a6eab2c1b53)

---

### Step 3: Configuring Splunk to Receive Logs

After logging into Splunk at `http://kali:8000`, we navigated to:

```
Settings → Forwarding and Receiving → Receive data
```

We opened the “Receive data” section and began the setup to allow Splunk to accept forwarded log data.

![Receive Data](https://github.com/user-attachments/assets/a3282a32-c71a-42cc-8767-6a8c0e8aafc7)

From there, we created a new TCP input on port `9997` to accept logs from external sources.

![Configure Port](https://github.com/user-attachments/assets/f3c3e1c4-3980-474e-9a71-b6f0564b3464)

#### Additional View – Splunk Data Configuration

To finalize the setup, we accessed the **"Add Data"** section from the Splunk dashboard.  
This menu allows configuration of inputs and ensures logs can be ingested correctly from various sources.

![Add Data](https://github.com/user-attachments/assets/175a382a-c763-44de-8144-4e52e0d35e64)

---

### Step 4: Installing Splunk Forwarder on the Victim Machine

To forward logs from the victim environment (Metasploitable3), we installed the Splunk Universal Forwarder.  
This lightweight version of Splunk is designed specifically for collecting and sending logs to a remote Splunk server.

```bash
wget -O splunkforwarder-9.4.1...deb "https://download.splunk.com/products/universalforwarder/releases/..."
```

![Download Command](https://github.com/user-attachments/assets/9327fa54-ee0f-4e61-9901-346a1b419096)

After running the installer, it showed the license agreement that we needed to accept to proceed.

![License Prompt](https://github.com/user-attachments/assets/7c247fb2-18af-461d-9a78-be822477dbfc)

Once the download completed successfully, the `.deb` package was saved and ready for installation.

![Download Complete](https://github.com/user-attachments/assets/6f915a6d-aed2-4fca-a9ac-a920e3ceb116)

---

### Step 5: Connect Forwarder to Splunk Server

After installation, we configured the forwarder to send logs to our Splunk instance:

```bash
sudo /opt/splunkforwarder/bin/splunk add forward-server 172.20.10.7:9997
```

We were then prompted to enter the Splunk username and password.

![Connect Forwarder](https://github.com/user-attachments/assets/465cd4f8-bce3-4bae-81fc-9cf5fa42bd58)

---

### Step 6: Configure Log Monitoring

Next, we configured the forwarder to monitor the file `/var/log/auth.log`, which contains authentication and login attempts.

```bash
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log
```

![Monitor Log File](https://github.com/user-attachments/assets/cb1e69cd-e4e9-413a-8570-733a2d797be2)

---

### Step 7: Executing Custom SSH Brute Force Script

Before analyzing the logs in Splunk, we executed a custom Python script (`ssh_bruteforce.py`) on the attacker machine to simulate an SSH brute-force attack against the victim (Metasploitable3).

The script uses the `paramiko` library to try multiple username and password combinations.

Several connection attempts failed due to SSH protocol issues — specifically when the server didn’t respond properly to the connection, causing timeout or banner read errors.

![Brute Force Script](https://github.com/user-attachments/assets/abada982-d5d7-4be0-9901-ebe0f2656be6)

---

### 📊 Step 8: Visualizing the Attack Logs in Splunk

After setting up the forwarder and generating SSH brute-force activity, we used the Splunk Search interface to visualize the data.

Query used:

```spl
index=* source="/var/log/auth.log" "sshd" | timechart count by user
```

![Log Visualization](https://github.com/user-attachments/assets/f4ed8d61-9d1c-461b-b774-f1ed27c0cfe4)

#### 📉 Line Chart View:
This view helped us track how login attempts changed over time for each user.

[lineChart.pdf](https://github.com/user-attachments/files/19983750/lineChart.pdf)

#### 📊 Bar Chart View:
The bar chart provided a clearer comparison between users during each time interval.

[lineChart.pdf](https://github.com/user-attachments/files/19983751/lineChart.pdf)
---
