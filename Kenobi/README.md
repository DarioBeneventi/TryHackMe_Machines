# Kenobi - TryHackMe

![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/Kenobi/images/kenobi_header.png?raw=true)

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

![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/Kenobi/images/image1.png?raw=true)

* In our SMB shares enumeration we see that there are 3 shares, 2 that we can access to READ/WRITE and 1 which we do not have access to.

![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/Kenobi/images/image2.png?raw=true)

* We connect to one of the shares called anonymous and find a log.txt file which we can recursively download with the command smbget -R or GET 

![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/Kenobi/images/image3.png?raw=true)
![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/Kenobi/images/image4.png?raw=true)
![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/Kenobi/images/image5.png?raw=true)

* When we open and analyze the log.txt file we find: 

1. Information generated for Kenobi when generating an SSH key for the user. 

2. Information about the ProFTPD server. 

* We also had another interesting port in our initial nmap scan, the port 111 with service rpcbind that in this case is being used to access a network file system. Enumerate this to find a mount! 

![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/Kenobi/images/image6.png?raw=true)

 ### Exploitation

* We have noticed that the port 21 uses ProFTPD 1.3.5 and after researching this we find out that this version is vulnerable to the mod_copy module exploit. 

![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/Kenobi/images/image7.png?raw=true)

* The mod_copy module implements SITE CPFR and SITE CPO commands, these can be used to copy files/directories. Any unauthenticated client can leverage these commands to copy files from any part of the filesystem to a chosen destination. 
* We know that the FTP service is running as the Kenobi user (from the file on the share) and an ssh key is generated for that user. So we're now going to copy Kenobi's private key using SITE CPFR and SITE CPTO commands because it is allowed! 
 
 ![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/Kenobi/images/image8.png?raw=true)

* As we have placed the ssh key from kenobi in the /var directory we will now mount this directory to our machine so we can have access to the private key of kenobi. 
 
 ![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/Kenobi/images/image9.png?raw=true)
 
* Bingo we now got access to the /var/tmp directory where we copied the id_rsa(private key) to, all we need to do now is copy the file to our machine, adjust the rights on the file and utilize it to login in the kenobi account.

 ![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/Kenobi/images/image10.png?raw=true)
 ![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/Kenobi/images/image11.png?raw=true)
 
 ### Privilege Escalation

* We will be using a method called Path Variable Manipulation to escalate our privileges.

 ![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/Kenobi/images/image12.png?raw=true)

* SUID bits can be dangerous, some binaries such as passwd need to be run with elevated privileges (as its resetting your password on the system), however other custom files could that have the SUID bit can lead to all sorts of issues.
* After refreshing what SUID/SGID/Sticky Bits are we will now be looking for these type of files and we find one that stands out called /usr/bin/menu.
* When we run this binary file it gives us 3 options.


### Extra Notes
Used the following links to complete this lab 
http://pentestmonkey.net/tools/php-reverse-shell 
https://medium.com/@klockw3rk/privilege-escalation-leveraging-misconfigured-systemctl-permissions-bc62b0b28d49 
