# Kenobi - TryHackMe

![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/kenobi/images/kenobi_header.png?raw=true)

This room covers accessing a Samba share, manipulating a vulnerable version of ProFTPD (1.3.5) to gain initital access and escalate privileges to root via an SUID binary. 

 ### Tools Used
 * Nmap
 * Ncat
 * Smbclient
 * Smbget

 ### Vulnerabilities
 * Accessable SMB Shares 
 * ProFTPD 1.3.5 - mod_copy module
 * NFS mount /var
 * Path Variable Manipulation - SUID/SGID/Sticky Bits  

### Enumeration phase

* With our initial scan We can see that 7 ports are open on this machine, in particular 139 & 445 mean that this is a Samba server and also the port 21 in which we can ftp. 

![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/VulnVersity/images/image1.png?raw=true)
![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/VulnVersity/images/image2.png?raw=true)

* We will be using gobuster to fuzz for directories & search for something interesting.

![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/VulnVersity/images/image3.png?raw=true)

* Bingo we found a directory called internal, when we navigate to this page we can see a upload form page.

![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/VulnVersity/images/image4.png?raw=true)

 ### Exploitation
 
* With the info we gathered in the enum part of our pentest we will now try to upload a payload to the webserver by using the upload form page located in the /internal/ directory.

![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/VulnVersity/images/image5.png?raw=true)

* We get an error saying that the extension is not allowed.
* Luckily for us BurpSuite can help us determine what extensions are allowed by using it's snipe attack function in combination with a wordlist that contains payload extensions, make sure BurpSuite is configured to intercept all your browser traffic.
  1. First we go to Proxy - Intercept when we upload our file (shell in this case) once intercepted we right click the raw data and send it to the Intruder.
  2. In positions we wan't to specify what piece of text we wan't to fuzz, we do this by using the § character (see screenshot).
  3. In the Payload tab we load our wordlist of extensions we created and if needed remove the payload encoding (see screeenshot).
  4. Start the attack and look at the results tab to see what shell extension recieved a success response in the response sub-tab (see screeenshot).

![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/VulnVersity/images/image6.png?raw=true)
![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/VulnVersity/images/image7.png?raw=true)

* After our attack we now know that the .phtml extension can be uploaded to the webserver so we are going to use a PHP reverse shell as our payload. A reverse shell works by being called on the remote host and forcing this host to make a connection to you. So you'll listen for incoming connections, upload and have your shell executed which will beacon out to you to control! 
* We use the shell we found on [pentestmonkey](http://pentestmonkey.net/tools/php-reverse-shell) adjust it to our machine, upload it on the webserver and start a listener with netcat then we execute the shell by navigating to the uploaded shell. 

![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/VulnVersity/images/image8.png?raw=true)

* Now that we are inside we poke around and find our user which is called bill and his directory contains the user flag. 

![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/VulnVersity/images/image9.png?raw=true)

 ### Privilege Escalation

* Now that we have a initial foothold in the machine we would like to escalate our privileges and become the superuser. We will now do some more enumerating such as checking the config of SUID files.
* One file stands out which is called /bin/systemctl and looking at this [link](https://medium.com/@klockw3rk/privilege-escalation-leveraging-misconfigured-systemctl-permissions-bc62b0b28d49) we know why it is so important to have the rights to this file configured to superusers only.

![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/VulnVersity/images/image10.png?raw=true)
![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/VulnVersity/images/image11.png?raw=true)
![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/VulnVersity/images/image12.png?raw=true)

* SUID (set owner userId upon execution) is a special type of file permission given to a file. SUID gives temporary permissions to a user to run the program/file with the permission of the file owner (rather than the user who runs it).
* Now that we know the systemctl has been misconifgured by using a SUID Bit we will be creating a systemd unit file which is where systemctl references when starting a service, this service will contain our backdoor.
* We first need to look for a writeable directory, we can use the var/tmp directory for this as an example.

![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/VulnVersity/images/image13.png?raw=true)
![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/VulnVersity/images/image14.png?raw=true)
![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/VulnVersity/images/image15.png?raw=true)

* Now that our service is on the system we are going to use systemctl to start our backdoor service. Before starting the service we need to also run our netcat listener to capture the root shell.

![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/VulnVersity/images/image18.png?raw=true)
![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/VulnVersity/images/image16.png?raw=true)
![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/VulnVersity/images/image17.png?raw=true)

### Extra Notes
Used the following links to complete this lab 
http://pentestmonkey.net/tools/php-reverse-shell 
https://medium.com/@klockw3rk/privilege-escalation-leveraging-misconfigured-systemctl-permissions-bc62b0b28d49 
