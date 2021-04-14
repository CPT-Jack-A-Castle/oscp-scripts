# Linux Privilege Escalation

## Linux Checklist
1. Sudo without a password? 
2. Root SUID binaries
3. Checking /home user dirs
4. Cron jobs (linpeas can help identify invisible cron jobs)
5. Ports listening only on localhost
6. Checking subdirs under / (eg. /var, /opt) for unusual files
7. Processes running as root

# Windows Privilege Escalation 

## Windows Checklist
1. Service Misconfigurations
    - Insecure Service Properties
        - Change the config of a service, change the executable the service uses, restart the service
    - Unquoted Service Path
        - Service doesn't use quotes for a file path, and the path has spaces in it. 
        - Windows will try all possible permutations of the path until it lands on the expected file
        - See if you have write access into one of these permutation, put a rev shell binary with the expected file name in there
    - Weak Registry Permissions
        - Registry entries have their own ACLs. It is possible to modify the configuration of a service even if you cannot modify the service directly.
    - Insecure Service Executables
        - if you have write access to the service's executable, you can replace it with a revshell
    - DLL Hijacking
11. Kernel Exploit
    - Should be a last resort, pretty often it crashes the system.
    - Tools: 
        - [Windows Exploit Suggester](https://github.com/bitsadmin/wesng)
            - Uses the output of systeminfo to suggest exploits. Save output of systeminfo to systeminfo.txt
            - python wes.py systeminfo.txt -i 'Elevation of Privilege' --exploits-only
            - cross reference ouptut with the wiki below 
        - [Precompiled Kernel Exploits](https://github.com/SecWiki/windows-kernel-exploits)
        - [Watson](https://github.com/rasta-mouse/Watson)
            - Good for more recent versions of Windows

- [HackTricks Book Checklist](https://book.hacktricks.xyz/windows/checklist-windows-privilege-escalation)