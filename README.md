# ICS344-Project
This repository contains the full implementation of our ICS344 course project. The course focuses on **Information Security**, and this project is designed to apply core concepts of system exploitation, attack detection, and defensive countermeasures in a hands-on, practical manner.

The project simulates a real-world security scenario by:
- Deploying a vulnerable target (Metasploitable3)
- Executing a brute-force attack from an attacker machine (Kali Linux)
- Integrating and visualizing the logs using a **SIEM (Splunk)**
- Applying and testing defensive tools such as **Fail2Ban** or **iptables**

---

## 📌 Project Phases

| Phase     | Description |
|-----------|-------------|
| **Phase 1**   | Set up a simulated attack lab environment using Metasploitable3 and Kali Linux. Launched and verified connectivity, then performed an **SSH brute-force attack** using a Python script. |
| **Phase 2**   | Collected and visualized logs using **Splunk**. Installed Splunk Enterprise on Kali and Universal Forwarder on Metasploitable3. Successfully monitored the attack and created meaningful visualizations of the logs. |
| **Phase 3**   | Deployed a **defensive tool** (e.g., Fail2Ban) to mitigate the attack. Reran the same attack to evaluate the effectiveness of the defense and analyzed its impact through Splunk logs. |

---

## 👥 Group Members

| Name              | Student ID   |
|-------------------|--------------|
| Zainab Alturaiki  | 202156350    |
| Sara Alshayeb     | 202174910    |
| Noor Alqatari     | 202174930    |
