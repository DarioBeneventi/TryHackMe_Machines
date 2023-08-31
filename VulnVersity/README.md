# VulnVersity - TryHackMe

This box contains a webserver that is vulnerable due to having a upload form page in one of its directories, we find the directoy in question by using GoBuster. We also fuzz out what extensions are allowed with the Burpsuite snipe attack to be able to know what format our backdoor has to be. To escalate our privileges we look for a misconfiguration in the SUID files, systemctl in particular is vulnerable and therefore we create a system d unit file that creates a backdoor to our attack machine but this time with the privileges of the root user.


 ### Tools Used
 * Nmap
 * Gobuster
 * Burpsuite

 ### Vulnerabilities
 * Vulnerable web server - upload form page
 * Misconfigured SUID

### Enumeration phase
* With our initial scan we can see that 6 ports are open and that on port 3333 a web server is running, we also know that our operating system is Ubuntu. 

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



 

