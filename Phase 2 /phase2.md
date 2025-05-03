
# Phase 2: Visual Analysis with a SIEM Dashboard

In this phase, we focused on leveraging a SIEM platform—**Splunk**—to collect, analyze, and visualize logs from both the attacker and victim environments. The goal was to monitor the attack activity from multiple perspectives and gain a deeper understanding of how the attack unfolded.

To fulfill the project requirements, we completed the following:

- **Integrated logs** from both the victim (Metasploitable3) and attacker (Kali Linux) environments into Splunk.
- **Visualized the attack behavior** using Splunk dashboards and searches to identify authentication patterns, brute-force attempts, and indicators of compromise.
- **Compared logs** from both environments to correlate actions performed by the attacker with reactions and logs recorded on the victim system.

This setup allowed us to clearly observe the full lifecycle of the attack and demonstrate how a SIEM can be used for forensic analysis, monitoring, and alerting.

Below, we walk through each step in setting up Splunk and performing the required tasks, supported with screenshots and log evidence.

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

After extracting Splunk, we proceeded to start the service and complete the initial setup required for first-time use.

We navigated to the Splunk binary directory and started the service using the following command:

```bash
cd /opt/splunk/bin
sudo ./splunk start --accept-license
```
![WhatsApp Image 1446-11-05 at 12 08 56](https://github.com/user-attachments/assets/8dd76f0b-0dfd-4e14-82f9-63879c8095a7)
![Step 1](https://github.com/user-attachments/assets/35471fbe-205b-49f5-b020-8a6eab2c1b53)

The --accept-license flag was used to automatically approve the Splunk license agreement. During the first-time startup, Splunk prompted us to create an administrator account. We set a username and a secure password as part of the setup process.

The initialization process included several checks and operations:

Validation of installation files
Setup of directories for logs, sessions, hash storage, and configurations
Generation of RSA private keys for internal communication
Configuration of certificate files (privKeySecure.pem)
Startup of the Splunk server daemon (splunkd)
After all checks passed, Splunk began listening on its default service ports:

- 8000 → Web Interface
- 8089 → Management Port
- 8065 → App Server Port
- 8191 → KV Store
Once the setup completed, the web interface became available at:
```bash
http://kali:8000
```
This URL provides access to the Splunk dashboard, where we can begin configuring data inputs, monitoring logs, and building visualizations.

---

### Step 3: Configuring Splunk to Receive Logs

We then opened the browser from the Kali attacker VM and go to `http://kali:8000`.  
Then we login using the admin credentials that were created during the initial setup.


After that, we navigated to:

```
Search → Data → Forwarding and Receiving
```
![Add Data](https://github.com/user-attachments/assets/175a382a-c763-44de-8144-4e52e0d35e64)


We then navigated to add new in the “Receive data” section and began the setup to allow Splunk to accept forwarded log data. And from there, we created a new TCP input on port `9997` to accept logs from external sources.

![Receive Data](https://github.com/user-attachments/assets/a3282a32-c71a-42cc-8767-6a8c0e8aafc7)

![Configure Port](https://github.com/user-attachments/assets/f3c3e1c4-3980-474e-9a71-b6f0564b3464)


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

The script uses the `paramiko` library to try multiple username and password combinations. It successfully established a connection using the credentials:

- **Username:** `vagrant`
- **Password:** `vagrant`

Once valid credentials were found, the script executed a command (`uname -a`) on the victim machine to confirm shell access and saved the credentials inside a file named `credentials.txt`.

This confirmed that the brute-force attempt worked and gave us remote access to the target over SSH.


![Brute Force Script](https://github.com/user-attachments/assets/abada982-d5d7-4be0-9901-ebe0f2656be6)
![WhatsApp Image 1446-11-05 at 13 33 14](https://github.com/user-attachments/assets/c6678d92-dcdf-4ee3-9032-d7a80cdaccae)


---
### Background System Activity: Cron Sessions from the Victim Machine
![image](https://github.com/user-attachments/assets/fd8bb2d1-b88a-43b2-8082-fd5b56e79002)
These log entries are not related to the brute-force attack directly, but rather to routine system tasks. Specifically:

- The logs show pam_unix(cron:session) messages, which indicate that cron jobs (scheduled tasks) were opened and closed for the root user.
- Each entry states either:
-   session opened for user root by (uid=0) or
-     session closed for user root

This is standard behavior in Unix-like systems, where background tasks (like system cleanups or automated scripts) run periodically as the root user through the cron daemon. These entries are timestamped and grouped closely by time, showing that the system is regularly running scheduled tasks.

While these logs aren't directly tied to attack attempts, including them shows that the log forwarding setup is functioning correctly and capturing all types of authentication events—not just SSH logins. It helps validate the integrity and completeness of the log ingestion pipeline from the victim machine to Splunk.

---

### Step 8: Visualizing the Attack Logs in Splunk

After forwarding the logs and executing the brute-force attack, we used Splunk's **Search & Reporting** dashboard to visualize the captured authentication events from `/var/log/auth.log`.

We ran the following query to generate a time-based chart showing SSH activity:

```spl
index=* source="/var/log/auth.log" "sshd" | timechart count by user
```

![Log Visualization](https://github.com/user-attachments/assets/f4ed8d61-9d1c-461b-b774-f1ed27c0cfe4)

---
This generated both line chart and bar chart 
<img width="942" alt="barchart" src="https://github.com/user-attachments/assets/2b8eb011-3097-403f-adbb-a8c683fecb18" />
<img width="942" alt="linechart" src="https://github.com/user-attachments/assets/c01653ba-1d93-4030-8005-bfd61b6a3f1d" />

---

The raw logs captured from /var/log/auth.log in Splunk provide granular visibility into each authentication event. These logs clearly show multiple SSH authentication failures originating from the attacker's IP address (172.20.10.7) targeting user accounts like root and vagrant. The presence of numerous failure messages confirms the brute-force nature of the attack, where the system repeatedly attempted to guess login credentials. This raw log data not only validates the trends seen in the bar and line charts, but also provides the exact timestamps, usernames, and source IPs involved, making it a critical reference for forensic analysis and attack attribution. 
**to see the entire logs capture, please see the PDFs attached below** 

![WhatsApp Image 1446-11-05 at 15 01 52](https://github.com/user-attachments/assets/b5a3570a-60a4-4907-9e15-493a15d8c6fd)

#### 📉 Line Chart Complete View (with logs): 
[lineChart.pdf](https://github.com/user-attachments/files/20023136/lineChart.pdf)


#### 📊 Bar Chart Complete View (with logs):
[barchart.pdf](https://github.com/user-attachments/files/20023137/barchart.pdf)

The bar chart shows the number of SSH authentication events over time, categorized by user: root, vagrant, and NULL. The NULL category represents authentication attempts where the username could not be identified—commonly a result of malformed login attempts or failed brute-force attempts that didn’t provide valid usernames. These dominate the chart, indicating a high frequency of invalid or unauthenticated login attempts. The root and vagrant bars appear much smaller in comparison, showing a lower number of login attempts targeting valid usernames.

The line chart further highlights the same trends but with clearer temporal patterns. Spikes in the NULL line indicate bursts of invalid SSH attempts during specific periods—aligning with our brute-force testing window. The vagrant and root lines show smaller, consistent peaks, suggesting that these accounts were also targeted but less aggressively. The presence of regular time gaps between spikes may correspond to automated retry intervals or cooling-off periods enforced by the attacker script or security mechanisms.

Together, these visualizations confirm the occurrence of a brute-force attack focused on random or guessed usernames, with intermittent targeted attempts on specific user accounts. This supports our Phase 2 objective of detecting and analyzing attack behavior using SIEM dashboards.

#### Authentication Result Breakdown (Accepted vs Failed)
To gain deeper insight into whether those login attempts succeeded or failed, we ran the following query:
```spl
index=* source="/var/log/auth.log" ("Failed" OR "Accepted") 
| eval status=if(searchmatch("Failed"), "Failed", "Accepted") 
| timechart span=1h count by status
```
![WhatsApp Image 1446-11-05 at 12 18 11](https://github.com/user-attachments/assets/c49c1588-4229-4076-95a8-3a9f2a995e82)

This bar chart visualizes the authentication outcomes across time intervals—showing a clear dominance of failed attempts with only a few successful ones.

Additionally, by querying the authentication logs from /var/log/auth.log for keywords like "Failed" and "Accepted", we were able to directly observe the underlying events that fueled the visualizations. The failed login logs show repeated SSH authentication failures for various usernames including root, vagrant, and several invalid or non-existent users such as msfadmin. These logs clearly indicate a brute-force pattern—many login attempts originating from the same source IP (172.20.10.7), each targeting different usernames in rapid succession. This aligns with the behavior of automated scripts attempting to guess credentials. On the other hand, the few "Accepted" entries confirm that the attacker eventually succeeded in authenticating using the vagrant account. The presence of both failure and success entries in the logs provides strong forensic evidence of a real brute-force attack, confirming the trends highlighted in both the bar and line charts.

![WhatsApp Image 1446-11-05 at 15 02 29](https://github.com/user-attachments/assets/352242c4-4194-4c16-b31c-cada4a72bd29)
![WhatsApp Image 1446-11-05 at 15 01 08](https://github.com/user-attachments/assets/1dd52857-b7f3-411b-b2f1-8ecdf842428b)


These results validate the brute-force behavior: a large number of failed login attempts followed by rare successful logins, likely when the correct credentials were guessed. The spacing of bars also reveals how the attack was executed in timed waves, possibly with retry delays or rate-limiting from the target system.


---
## Conclusion 
In Phase 2, we successfully integrated logs from both the victim (Metasploitable3) and attacker (Kali) environments into Splunk, a SIEM platform, to observe and analyze the behavior of our brute-force attack.

We began by configuring Splunk on Kali to receive logs over TCP port 9997, enabling the ingestion of system logs from the victim machine. Once the SSH brute-force attack was executed—targeting multiple usernames using an automated Python script—the forwarded logs captured authentication events in real time.

Using Splunk’s Search & Reporting dashboard, we visualized these logs in various forms:

- Bar Charts and Line Charts: These charts revealed a sharp increase in SSH activity during the attack window. Most events were tied to NULL usernames, highlighting the attacker’s attempt to authenticate with invalid or malformed usernames. Meanwhile, valid users like root and vagrant were targeted less frequently, with vagrant eventually showing successful login entries.
- Raw Logs: By filtering for keywords such as "Failed" and "Accepted", we examined the underlying log messages, confirming our visual findings. The logs captured multiple repeated failures from the same IP address, consistent with brute-force tactics, followed by a few successful entries that validated a breach.
Together, these visualizations and logs provided deep visibility into the pattern and outcome of the attack:

It was automated, with regular time intervals between attempts.
It primarily targeted invalid usernames, indicating a broad sweep approach.
It eventually succeeded, which poses critical security implications.
This phase demonstrated how SIEM tools like Splunk can effectively detect, visualize, and validate cyberattacks, transforming raw log data into actionable insights. It also highlights the importance of real-time monitoring and alerting mechanisms for mitigating such threats before successful compromise occurs.
