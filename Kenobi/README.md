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

* We also had another interesting port in our initial nmap scan, the 111 port with service rpcbind that in this case is being used to access a network file system. Enumerate this to find a mount! 

![alt text](https://github.com/DarioBeneventi/TryHackMe_Machines/blob/main/Kenobi/images/image6.png?raw=true)

 ### Exploitation
 


 ### Privilege Escalation



### Extra Notes
Used the following links to complete this lab 
http://pentestmonkey.net/tools/php-reverse-shell 
https://medium.com/@klockw3rk/privilege-escalation-leveraging-misconfigured-systemctl-permissions-bc62b0b28d49 
