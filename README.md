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
