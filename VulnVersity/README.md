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