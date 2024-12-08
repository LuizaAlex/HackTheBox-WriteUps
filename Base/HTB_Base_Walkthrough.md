
# HTB Walkthrough: Base

This walkthrough covers the steps taken to complete the HTB machine **Base**, highlighting key vulnerabilities and methods used to gain both user and root access.

---

## Task 1: Which two TCP ports are open on the remote host?

The first step was to perform an Nmap scan to enumerate open ports and services:
```bash
nmap -sC -sV TARGET_IP
```
<img width="666" alt="Screenshot 2024-12-08 at 18 01 12" src="https://github.com/user-attachments/assets/777795d0-83aa-4c19-973e-445a14a2ff59">


---

## Task 2: What is the relative path on the webserver for the login page?
Based on the Nmap results, I observed an HTTP server running on port 80. Accessing the target's IP address and port in a browser, I navigated to the login page.

<img width="658" alt="Screenshot 2024-12-08 at 18 14 43" src="https://github.com/user-attachments/assets/5de61250-013b-4af3-b042-7d636e37481f">
---

## Task 3: How many files are present in the /login directory?


I modified the URL from /login/login.php to /login, which listed several files within the directory.


<img width="658" alt="Screenshot 2024-12-08 at 18 15 14" src="https://github.com/user-attachments/assets/6e708006-5f55-4ce1-8245-0d5663b9ecfe">

---

## Task 4: What is the file extension of a swap file?

A quick Google search confirmed that the common file extension for swap files is .swp. Swap files may also use extensions like .swo.



---

## Task 5: Which PHP function is being used in the backend code to compare the user-submitted username and password to the valid username and password?

I downloaded the login.php.swp file and inspected its contents. 

<img width="528" alt="Screenshot 2024-12-08 at 18 16 14" src="https://github.com/user-attachments/assets/87915f39-5c2f-4c29-88ac-f22c76a7c678">

Using the strings command to display readable text, I found that the strcmp function was being used to compare strings.

<img width="588" alt="Screenshot 2024-12-08 at 18 16 23" src="https://github.com/user-attachments/assets/1119e6d3-8ee6-49de-8f72-f1c3f27903f9">


<img width="639" alt="Screenshot 2024-12-08 at 18 16 45" src="https://github.com/user-attachments/assets/74b9ed3f-865a-49ca-9d2f-e1f19164ec84">

---

## Task 6: In which directory are the uploaded files stored?

To investigate the upload functionality, I used BurpSuite to intercept a login request and modified the POST data to bypass authentication:

<img width="658" alt="Screenshot 2024-12-08 at 18 17 02" src="https://github.com/user-attachments/assets/ca00d02c-86e3-43d7-855a-4796812f43b2">


Change the POST data as follows to bypass the login.

username[]=admin&password[]=pass

<img width="664" alt="Screenshot 2024-12-08 at 18 17 30" src="https://github.com/user-attachments/assets/89ccb99d-4b0c-4215-b7ee-91bf71078e3c">

After logging in, I created a simple PHP file (test.php) containing the phpinfo() function, which outputs the PHP configuration details. Once uploaded, the web application confirmed the successful upload of the file.

<img width="633" alt="Screenshot 2024-12-08 at 18 18 16" src="https://github.com/user-attachments/assets/dfbe3bef-5ea3-42a5-896e-4f03385d642c">

Using gobuster, I enumerated directories and discovered a directory named _uploaded. Navigating to this directory, I confirmed that the uploaded file was present.

<img width="662" alt="Screenshot 2024-12-08 at 18 18 02" src="https://github.com/user-attachments/assets/e77bc11d-30c3-4d52-9fcc-9c78226099f9">

---

## Task 7: Which user exists on the remote host with a home directory?
To gain a reverse shell, I uploaded a PHP reverse shell script from pentestmonkey(https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php). Before uploading, I modified the script to use the IP address of my attack machine and a chosen port.


<img width="627" alt="Screenshot 2024-12-08 at 18 18 36" src="https://github.com/user-attachments/assets/4e9cd00a-9c1c-4b79-a2f8-98a57b602ac3">



I then started a listener on my attack machine:

```bash
nc -lvnp PORT  
```

By accessing the uploaded reverse shell file on the target's website, I established a connection back to my listener. 

<img width="662" alt="Screenshot 2024-12-08 at 18 19 08" src="https://github.com/user-attachments/assets/d5a000cd-13b4-42ec-ae34-04ea349ff3c8">


Once connected, I upgraded the shell for better usability.

<img width="492" alt="Screenshot 2024-12-08 at 18 19 23" src="https://github.com/user-attachments/assets/2c1d363e-a2e1-423f-9be7-b35bfd42eb7a">

Exploring the directories, we found our friend john in home.

<img width="657" alt="Screenshot 2024-12-08 at 18 19 43" src="https://github.com/user-attachments/assets/e17d1531-f43d-4c77-a9fe-7b3b90b0f308">

---

## Task 8: What is the password for the user present on the system?

Exploring the directories, I found a config.php file in /var/www/html/login, which contained the password for the user john. Using this information, I was able to log in as john.
<img width="623" alt="Screenshot 2024-12-08 at 18 19 59" src="https://github.com/user-attachments/assets/c937e7b7-a151-42c3-b658-938a31292c83">

Now we can connect as john.

<img width="595" alt="Screenshot 2024-12-08 at 18 20 44" src="https://github.com/user-attachments/assets/190a3cb5-b4f5-42f7-9920-1cdc31211135">


---

## Task 9: What is the full path to the command that the user john can run as root on the remote host?

Running the sudo -l command revealed that john could execute the find command as root.

<img width="659" alt="Screenshot 2024-12-08 at 18 20 58" src="https://github.com/user-attachments/assets/5f8cf958-508b-45f2-a4f2-57abda6e0f7c">


---
## Task 10: What action can the find command use to execute commands?

The find command has an -exec action that can be used to execute commands.
<img width="635" alt="Screenshot 2024-12-08 at 18 21 15" src="https://github.com/user-attachments/assets/35fc9f4d-21cc-41b1-9202-546bf08fa434">


---

## User Flag

The user flag was located in the user directory of john.

---

## Root Flag

Using the ability to execute the find command as root, I performed a privilege escalation to gain root access.


<img width="538" alt="Screenshot 2024-12-08 at 18 21 34" src="https://github.com/user-attachments/assets/113690aa-579b-478e-8d93-bb83b56d7803">

And retrieve the root flag.

<img width="459" alt="Screenshot 2024-12-08 at 18 21 42" src="https://github.com/user-attachments/assets/33904de4-ff85-414c-bd91-be1d13a8e44e">


---

## Conclusion

The **Base** machine provided insights into common vulnerabilities, such as directory traversal, swap file inspection, and improper privilege management. By methodically exploring each vector, I successfully gained both user and root access.
