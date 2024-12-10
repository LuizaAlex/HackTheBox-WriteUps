# HTB Cap Walkthrough

## Overview

**Cap** is an easy-difficulty Linux machine hosted on Hack The Box. It features an HTTP server that allows users to perform administrative functions, including network captures. Due to improper access controls, the machine is vulnerable to **Insecure Direct Object Reference (IDOR)**, enabling access to other users' data. By analyzing a captured file, plaintext credentials are revealed, which allow initial access. Finally, Linux capabilities are exploited to escalate privileges to root.

---

## Task 1: How many TCP ports are open?

We began by running an **Nmap** scan:
```bash
nmap -sC -sV -oN nmap_scan.txt <target_ip>
```


<img width="664" alt="Screenshot 2024-12-10 at 22 52 01" src="https://github.com/user-attachments/assets/f669711f-cc8f-4df5-baf3-7cef857e4c43">


The scan revealed **three open ports**:
- 21 (FTP)
- 22 (SSH)
- 80 (HTTP)

Answer: `3`

---

## Task 2: What is the [something] in the URL `/[something]/[id]` after running a "Security Snapshot"?

### Steps:
1. Checked the HTTP service on port 80, which is powered by Gunicorn (a Python-based HTTP server).

On<img width="657" alt="Screenshot 2024-12-10 at 22 52 52" src="https://github.com/user-attachments/assets/244bc258-0907-44ff-9152-56f17c81bb95">

2. Navigated to the Dashboard and explored the **IP Config** section.
<img width="647" alt="Screenshot 2024-12-10 at 22 53 47" src="https://github.com/user-attachments/assets/9e0858a4-06e4-4530-a4f3-bcdffbb15298">


 the IP Config page, we observed a link to download a packet capture file. After triggering a "Security Snapshot," the browser was redirected to a URL with the structure `/data/[id]`.


<img width="668" alt="Screenshot 2024-12-10 at 22 53 56" src="https://github.com/user-attachments/assets/e1317b53-4fb9-4930-9ae1-d1048f8c4f76">


Answer: `data`



---

## Task 3: Are you able to access other users' scans?


Yes, by modifying the `id` value in the URL, we could access scans belonging to other users due to the lack of proper access controls (**IDOR vulnerability**).


---

## Task 4: What is the ID of the PCAP file that contains sensitive data?


### Steps:
1. Downloaded the packet capture (PCAP) file from the `/data/[id]` URL.
2. Opened the file in **Wireshark** for analysis.


Within the PCAP file, we discovered sensitive data, including credentials:
```
Username: nathan
Password: Buck3tH4TF0RM3!
```
<img width="666" alt="Screenshot 2024-12-10 at 22 54 59" src="https://github.com/user-attachments/assets/9e98d3fb-fc75-4206-b69f-ae5c6e2e2b40">


Answer: The ID was found in the URL of the PCAP file.


---

## Task 5: Which application layer protocol in the PCAP file contains the sensitive data?


The sensitive data (credentials) were transmitted over the **FTP protocol**.




<img width="630" alt="Screenshot 2024-12-10 at 22 55 30" src="https://github.com/user-attachments/assets/e2f2eee5-9307-46b8-9738-3e2e6155f934">



Answer: `FTP`



---

## Task 6: On what other service does Nathan's password work?


Using Nathan's credentials, we successfully authenticated on:
- **FTP** (port 21)
- **SSH** (port 22)

Answer: `SSH`

---


## Task 7: Submit the flag located in Nathan's home directory.

### Steps:
1. Logged into the machine via SSH using Nathan's credentials:
   ```bash
   ssh nathan@<target_ip>
   ```

   
2. Navigated to Nathan's home directory and retrieved the user flag:
   ```bash
   cat /home/nathan/user.txt
   ```

   
   
<img width="605" alt="Screenshot 2024-12-10 at 22 56 19" src="https://github.com/user-attachments/assets/63f45edf-07fd-4aae-909d-121304a98137">

---


## Task 8: What is the full path to the binary with special capabilities that can be abused to escalate privileges?


### Steps:

1. Searched for binaries with elevated capabilities using:
2. 
   ```bash
   getcap -r / 2>/dev/null
   ```

   <img width="647" alt="Screenshot 2024-12-10 at 22 56 37" src="https://github.com/user-attachments/assets/3ddcc140-56db-46c0-8f0a-84a45335ca7b">

3. Found the following binary with special capabilities:

   
   ```
   /usr/bin/python3.8 = cap_setuid+ep
   ```

The `cap_setuid` capability allows switching the user ID to `0` (root) without the SUID bit. By leveraging this capability, we gained a root shell.


### Exploitation:

1. Launched Python and executed the following commands:

   
   ```python
   import os
   os.setuid(0)
   os.system("/bin/bash")
   ```
   

   <img width="645" alt="Screenshot 2024-12-10 at 22 56 57" src="https://github.com/user-attachments/assets/e99c22b0-a1d3-4b26-94c8-3aefe7d1940a">

Answer: `/usr/bin/python3.8`


2. Retrieved the root flag:

   
   ```bash
   cat /root/root.txt
   ```
   
<img width="614" alt="Screenshot 2024-12-10 at 22 57 07" src="https://github.com/user-attachments/assets/eaa8c541-0588-4011-813e-0e6188efa245">



---

## Conclusion
This walkthrough demonstrated how to:
1. Identify and exploit an **IDOR vulnerability**.
2. Analyze a **PCAP file** to extract sensitive data.
3. Use credentials to gain initial access.
4. Leverage Linux capabilities to escalate privileges and obtain root access.

---
