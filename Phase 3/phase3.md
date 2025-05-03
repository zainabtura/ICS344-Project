# Phase 3: Defense Mechanism & Testing
In this phase of the project, our goal was to harden the SSH service on Metasploitable3 against automated brute-force attacks. We began by establishing a baseline “before” scenario—live-tailing the SSH authentication log and launching a scripted password-guessing attack from Kali—so that we could clearly demonstrate the system’s vulnerability. We then installed and configured Fail2Ban, a lightweight intrusion-prevention daemon that scans /var/log/auth.log for repeated SSH failures and dynamically injects iptables rules to block offending IP addresses. Finally, we validated the defense by repeating the same brute-force test and showing, in real time, that Fail2Ban automatically detected three consecutive failed login attempts, applied a temporary ban, and dropped all further SSH connections from the attacker’s IP.

Throughout this phase we used:
• tail -f /var/log/auth.log and tail -f /var/log/fail2ban.log to monitor authentication events and Fail2Ban actions in real time
• A custom Python brute-force script to attempt logins against port 22 on Metasploitable3
• apt-get install fail2ban iptables to deploy the monitoring and firewall tools on Metasploitable3
• sshpass on Kali to automate rapid, non-interactive password failures
• iptables (via Fail2Ban’s iptables-multiport banaction) to enforce bans at the network layer

By the end of this phase, any sequence of three invalid SSH logins from the same source triggers a ban in under a second, instantly blocking further connection attempts and effectively mitigating the brute-force threat.

---

### Step 1 – Prepare SSH Log Monitoring on Metasploitable3

On the Metasploitable3 VM, I started a live view of the SSH authentication log: 

```bash
sudo tail -f /var/log/auth.log
```

This ensured that, the moment our brute-force attack succeeded, the “Accepted password” entry would appear on-screen in real time. To capture that entry for the report, I then extracted the most recent 50 lines and filtered for “Accepted”: 

```bash
sudo tail -n50 /var/log/auth.log | grep Accepted
```
<img width="468" alt="Picture1" src="https://github.com/user-attachments/assets/4fe157de-9ddf-4c08-a38e-d1e6c5193d6f" />

<img width="468" alt="Picture2" src="https://github.com/user-attachments/assets/5d11d544-c761-4f3e-a44b-965b4d79156d" />

The screenshot confirms that SSH log monitoring was active during the brute-force attack. The command `tail -f /var/log/auth.log` successfully displayed authentication events in real time.

The presence of the log entry:
```bash
Accepted password for vagrant from 192.168.64.X port XXXXX ssh2
```
indicates that a login attempt using the `vagrant` credentials was successful. This confirms the effectiveness of the brute-force script and verifies that Metasploitable3 correctly logged the accepted connection in `/var/log/auth.log`.

By using `grep Accepted` on the recent logs, we were able to filter out the relevant line for reporting purposes, showing proof of access from the attacker's IP.

---

### Step 2 – Launch Brute-Force from Kali

From the Kali attacker machine, we re-ran our Python brute-force script against port 22:

```bash
python3 ssh_bruteforce.py   --host 192.168.64.7   --user root   --passlist /home/kali/pass.txt
```

This produced the following screenshot, which shows the script’s final result and confirms that the SSH service on Metasploitable3 accepted a valid credential.

Together, these steps provide the “before” evidence that port 22 on Metasploitable3 was still vulnerable to brute-force login.
![Picture3](https://github.com/user-attachments/assets/4355f147-6f01-43c2-aad6-437c0df123e4)
![Picture4](https://github.com/user-attachments/assets/6969f1ba-7dc8-415e-9812-a2ef1a4766ca)

---

### Step 3 – Install Defense Prerequisites on Metasploitable3

To begin, we updated the Metasploitable3 VM’s package index to ensure we’re installing the most current software versions. I ran:

```bash
sudo apt-get update
```
![ndjh](https://github.com/user-attachments/assets/57224b4d-2239-457b-9638-40a5652e3bc8)

With the package database up to date, we installed Fail2Ban, the tool that will monitor /var/log/auth.log for repeated SSH failures and automatically trigger bans when necessary. We also installed iptables to enforce those bans at the network level by dropping traffic from banned IP addresses. The command used was:
```bash
sudo apt-get install -y fail2ban iptables 
```
<img width="468" alt="s" src="https://github.com/user-attachments/assets/3284251d-b515-48e3-a3fe-73b206336f44" />

The output confirms that Fail2Ban was downloaded and installed without errors, making the daemon available for configuration. The installation output also indicates that iptables was already at the newest version, meaning our kernel’s firewall framework is ready.

Finally, we verified that the Fail2Ban service was running before we add any jail definitions. By running:

```bash
sudo service fail2ban status
```
<img width="468" alt="d" src="https://github.com/user-attachments/assets/1e8a0c8f-d8e3-4603-9ca0-91bf7516fc9c" />

The output “fail2ban is running” confirms that the daemon is active and prepared to enforce any jails we define in the next step.

---

### Step 4 – Configure & Verify the SSH Jail

Next, I defined the SSH-specific jail so that Fail2Ban would watch the SSH authentication log and ban any IP after three failed login attempts. I opened the local jail configuration with:

```bash
sudo nano /etc/fail2ban/jail.local 
```

In the editor, we added the [sshd] section, enabling the jail on port 22, pointing to /var/log/auth.log, and setting maxretry to 3, findtime and bantime to 600 seconds, with banaction = iptables-multiport. The screenshot shows the complete block in nano, confirming the correct parameters were entered.


<img width="468" alt="w" src="https://github.com/user-attachments/assets/2a0b8680-12c5-4ef3-a61c-cbbb494a7565" />

The meaning of each field is described as below:  
- enabled: turns the jail on  
- port: the SSH port to protect  
- filter: uses the built-in sshd filter  
- logpath: where SSH auth failures are logged  
- maxretry: failures before ban  
- findtime: window (seconds) in which failures count  
- bantime: ban duration (seconds)  
- banaction: uses iptables to drop traffic

After saving and exiting nano, I reloaded Fail2Ban to apply the new jail and verify that the SSH jail was live but had not yet banned any IPs, I ran:

```bash
sudo service fail2ban restart
sudo fail2ban-client status sshd
```
<img width="468" alt="ps" src="https://github.com/user-attachments/assets/49744dcc-6971-4c73-b7fb-1921ade5d7b6" />

The console output reports “[ OK ] Restarting authentication failure monitor fail2ban,” indicating that Fail2Ban has re-read its configuration and activated the SSH jail. The status report shows “Currently banned: 0” under the [sshd] jail, confirming that Fail2Ban is monitoring the correct log file and is ready to ban after failed logins.

Finally, I checked the iptables chain that Fail2Ban will use for SSH bans:

```bash
sudo iptables -L f2b-sshd -n --line-numbers
```

![ppp](https://github.com/user-attachments/assets/4f1f3839-2830-422c-b944-1aaa79c1b212)


As expected, the message “iptables: No chain/target/match by that name” appears, proving that the ban chain does not yet exist, it will only be created when the first IP is banned. With these steps complete, the SSH jail is configured and awaiting attack attempts for Step 5.

---

### Step 5 – Trigger and Observe a Ban

To validate our defense, I first performed three consecutive failed SSH logins from the Kali attacker machine by running:

```bash
ssh root@192.168.64.7
```
<img width="468" alt="erty" src="https://github.com/user-attachments/assets/8c658255-d9ef-4039-a5a6-c57c9e71f168" />

and entering an incorrect password each time. 

Back on Metasploitable3, we then monitored the SSH authentication log in real time by running:

```bash
sudo tail -f /var/log/fail2ban.log
```
![tt](https://github.com/user-attachments/assets/6a9709f8-63f4-4ed7-b8fb-c0ec29e264b4)

The output confirms that the [sshd] jail was initialized.

With the SSH jail fully configured and running on Metasploitable3, the final validation is to generate repeated, rapid SSH failures from Kali and watch Fail2Ban automatically lock out the attacker’s IP. To do this, I opened a terminal on the Kali machine and installed sshpass so that I could script three incorrect-password attempts without manual interaction:

```bash
sudo apt update && sudo apt install -y sshpass
```

Then we opened a tail window to watch for the ban notice on Metasploitable3:

```bash
sudo tail -f /var/log/fail2ban.log
```
![yryry](https://github.com/user-attachments/assets/615fde3e-7bc5-43e9-9ab2-872ef7ecaa49)
![ddddd](https://github.com/user-attachments/assets/fc10c2a2-2ec5-41de-bb25-760cdbe1c207)

Once sshpass was in place, we ran the following loop to force three consecutive authentication failures against port 22 of 192.168.64.7:

```bash
for i in 1 2 3; do
  sshpass -p 'nottherootpw' ssh \
    -o PubkeyAuthentication=no \
    -o StrictHostKeyChecking=no \
    -o UserKnownHostsFile=/dev/null \
    root@192.168.64.7 true
done
```

Where:  
- `-p 'nottherootpw'` is our bogus password.  
- `-o PubkeyAuthentication=no` prevents SSH from even offering our key.  
- `-o StrictHostKeyChecking=no` & `-o UserKnownHostsFile=/dev/null` quiet any host-key warnings.
  
<img width="469" alt="444" src="https://github.com/user-attachments/assets/6844bc98-be45-4c9e-aede-2a6f6f8a83b3" />


Immediately after submitting the third bad password, we switched back to the Metasploitable3 terminal, where I had been tailing the Fail2Ban log in real time, and we can clearly see that the log quickly recorded that the sshd jail detected three failures, applied the ban timeout, and dropped the attacker’s IP.


<img width="468" alt="9" src="https://github.com/user-attachments/assets/b20a8ae4-c8af-4d1a-9462-ed74a5cb5f62" />

To further verify, we ran:

```bash
sudo fail2ban-client status sshd
```
<img width="468" alt="345" src="https://github.com/user-attachments/assets/3eab24b7-d8c8-4e29-a3b7-c3ba623db514" />

This output shows that Fail2Ban has registered exactly three failures and now holds one banned IP in its sshd jail.

Finally, I confirmed the actual firewall rule by listing the fail2ban-sshd chain:

```bash
sudo iptables -L f2b-sshd -n --line-numbers
```
<img width="468" alt="34567" src="https://github.com/user-attachments/assets/a1ac6a71-825c-4842-8b5d-da98603446f5" />

Together, these steps conclusively demonstrate that after three failed login attempts, Fail2Ban intercepts further SSH connections and injects a firewall rule to refuse traffic from the attacker’s IP—successfully mitigating the brute-force threat.

To demonstrate the effect of our SSH hardening, we first ran the exact same Python brute-force script from Kali before enabling Fail2Ban. Before enabling Fail2Ban, the script quickly discovered valid root credentials (vagrant:vagrant), as shown here.
![14](https://github.com/user-attachments/assets/93e56d10-da6f-46ee-85fc-690e186d389e)


After doing the defense by enabling our Fail2Ban configuration, we repeated the identical brute-force command. In the after scenario, we can see that no credentials can be cracked: every connection attempt is immediately reset (even for vagrant), proving that the firewall rules installed by Fail2Ban are actively blocking SSH connections from the attacker’s IP.
<img width="468" alt="000" src="https://github.com/user-attachments/assets/bb036cc8-dee9-4b75-940a-1523e7260939" />

By directly comparing before and after, we clearly see the security improvement: the same brute-force vector that once succeeded is now completely thwarted, validating our defense strategy.
