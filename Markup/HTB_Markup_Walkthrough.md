
# HTB Walkthrough: Markup

This walkthrough covers the steps taken to complete the HTB machine **Markup**, including identifying vulnerabilities and leveraging them to gain initial and root access.

---

## Task 1: What version of Apache is running on the target’s port 80?

The first step was to perform an Nmap scan to enumerate open ports and services:
```bash
nmap -sC -sV TARGET_IP
```
<img width="651" alt="Screenshot 2024-12-07 at 16 50 51" src="https://github.com/user-attachments/assets/fbaa6fb1-4ed9-4171-92f8-ea56194d136d">

This revealed three open ports: **22 (SSH)**, **80 (HTTP)**, and **443 (HTTPS)**.  

Examining port 80, I identified the version of Apache running on the target web server.

---

## Task 2: What username:password combination logs in successfully?
<img width="964" alt="Screenshot 2024-12-07 at 14 21 33" src="https://github.com/user-attachments/assets/975d6a0a-c095-411a-8043-02ed4bba7166">

After identifying the web server on port 80, I began testing common username and password combinations. For example:

- `admin:12345`  
- `administrator:password`  
- `admin:password`  

The last combination, `admin:password`, was successful and granted access to the application.
<img width="565" alt="Screenshot 2024-12-07 at 16 51 22" src="https://github.com/user-attachments/assets/b49ace96-14dd-4532-9dfa-b006f184cd06">

---

## Task 3: What is the word at the top of the page that accepts user input?

Navigating through the website, I found a form on the **Order** page. This form accepts user input, which is essential for interacting with the target system.
<img width="662" alt="Screenshot 2024-12-07 at 16 51 48" src="https://github.com/user-attachments/assets/5a288206-8c87-432f-baaf-91e1374677b2">

---

## Task 4: What XML version is used on the target?

To determine the XML version, I used **BurpSuite** with the **FoxyProxy** plugin to intercept requests on port 8080. I submitted random data in the order form and inspected the request payload. This revealed the XML version used by the target.
<img width="653" alt="Screenshot 2024-12-07 at 16 56 00" src="https://github.com/user-attachments/assets/e84b7e82-8e10-4041-8f5f-fbfc3a37ad4e">


---

## Task 5: What does the XXE/XEE attack acronym stand for?

XXE stands for **XML External Entity**. This information was quickly verified with a simple Google search.

---

## Task 6: What username can we find on the webpage’s HTML code?

Inspecting the webpage's source code in Mozilla Firefox repeatedly loaded `index.php`. Switching to Chromium allowed me to view the source code correctly, where I found the username: `daniel`.
<img width="652" alt="Screenshot 2024-12-07 at 16 52 15" src="https://github.com/user-attachments/assets/e1ab1cad-523b-4a2f-b1d6-204fe63ff6bf">

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
<img width="660" alt="Screenshot 2024-12-07 at 16 52 34" src="https://github.com/user-attachments/assets/a6508dc4-2d99-4c0b-a776-3b12dd7585e1">

This injection retrieved Daniel's private key. After saving it locally, I adjusted its permissions to make it readable for SSH:
```bash
chmod 400 id_rsa
```

Then, I connected to the target using the private key:
```bash
ssh -i id_rsa daniel@TARGET_IP
```
<img width="329" alt="Screenshot 2024-12-07 at 15 19 04" src="https://github.com/user-attachments/assets/24a32174-3d1e-490a-b0c8-f35fd815f58b">
<img width="456" alt="Screenshot 2024-12-07 at 15 21 11" src="https://github.com/user-attachments/assets/74f8af87-2599-4a21-9a81-f28606f1d663">

Once inside, I navigated to the **Log-Management** folder to locate the required file.
<img width="622" alt="Screenshot 2024-12-07 at 16 53 43" src="https://github.com/user-attachments/assets/a204c876-6579-40e2-bd9d-6c6b97e3982a">

---

## Task 8: What executable is mentioned in the file mentioned before?

Using the `type` command, I inspected the file in the Log-Management folder and discovered references to an executable named `wevtutil`.
<img width="666" alt="Screenshot 2024-12-07 at 16 53 58" src="https://github.com/user-attachments/assets/3761507b-36a1-49af-bd58-903729f6f1a9">

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
<img width="666" alt="Screenshot 2024-12-07 at 16 57 32" src="https://github.com/user-attachments/assets/85775179-e47c-467e-bcfd-351a6510717e">
<img width="665" alt="Screenshot 2024-12-07 at 16 58 13" src="https://github.com/user-attachments/assets/3d74cc4a-355c-404c-a415-75d390d53372">


2. Edited **job.bat** to include the Netcat reverse shell command (Note - exit the powershell first):
   
<img width="663" alt="Screenshot 2024-12-07 at 16 58 54" src="https://github.com/user-attachments/assets/b1987228-3639-40c6-83bb-d1f5dda09063">

4. Started a listener on the attack box:
   ```bash
   nc -lvnp PORT
   ```

When the script executed, I received a shell as the Administrator. The root flag was located in:
`C:/Users/administrator/Desktop/root.txt`

---

## Conclusion

The **Markup** machine highlights common misconfigurations and vulnerabilities, including weak credentials, XXE injection, and improper file permissions. These provided a clear path to escalate privileges and gain full control of the target.
