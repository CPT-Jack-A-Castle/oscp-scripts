# Linux Privilege Escalation Notes & Checklist
1. Service Exploits
    - Basically just enumerate services/programs and their versions then google for exploits 
    - Tweak public exploits where necessary
2. Weak File Permissions
    - system files that contain confidential information are readable
    - system files that are used by the OS are writeable (i.e. /etc/passwd)
    - readable backup files might contain files with sensitive information (i.e. a config file with a plaintext password)
        - common backup file locations: user home dir, /, /tmp, /var/backups
3. Sudo
    - if password is known and user in sudoers file, just `sudo su`
    - use shell escape sequences if you are limited to running sudo to certain programs
        - [GTFOBins](https://gtfobins.github.io/)
    - abuse intended functionality of sudo-able programs
        - for example, if the program that is sudo-able can read a file, try reading /etc/shadow
4. Cron Jobs
5. SUID/SGID executable
6. Passwords & Keys
7. NFS
8. Kernel Exploits
    - Should be a last resort, pretty often it crashes the system.
    - Tools: 
        - [Linux Exploit Suggester](https://github.com/jondonas/linux-exploit-suggester-2)
            - `./linux-exploit-suggester.pl`
        - Searchsploit
            - enumerate kernel version first: `uname -a`
            - `searchsploit linux kernel {version}`
            - adjust version number/distribution in search as needed
    
# Resources
- [PayloadsAllTheThings' Linux PrivEsc Methods](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md)