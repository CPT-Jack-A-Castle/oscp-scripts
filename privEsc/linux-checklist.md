# Linux Privilege Escalation Notes & Checklist
1. Service Exploits
    - Basically just enumerate services/programs and their versions then google for exploits 
    - Tweak public exploits where necessary
2. Weak File Permissions
    - system files that contain confidential information are readable
    - system files that are used by the OS are writeable (i.e. /etc/passwd)
    - readable backup files might contain files with sensitive information (i.e. a config file with a plaintext password)
        - common backup file locations: user home dirs, `/`, `/tmp`, `/var/backups`
3. Sudo
    - if password is known and user in sudoers file, just `sudo su`
    - use shell escape sequences if you are limited to running sudo to certain programs
        - [GTFOBins](https://gtfobins.github.io/)
    - abuse intended functionality of sudoable programs
        - for example, if the program that is sudo-able can read a file, try reading /etc/shadow
    - if `env_keep+=LD_PRELOAD` is in output of `sudo -l` then possible to privesc with [malicious preload file](https://github.com/dopamin3rgic/oscp-scripts/blob/main/privEsc/scripts/malicious-preload.c)
        - compile malicious preload file `gcc -fPIC -shared -nostartfiles -o /tmp/preload.so malicious-preload.c`
        - `sudo LD_PRELOAD=/tmp/preload.so {sudoable program}`
    - if `env_keep+=LD_LIBRARY_PATH` is in output of `sudo -l` then possible to privesc with a [malicious shared library file](https://github.com/dopamin3rgic/oscp-scripts/blob/main/privEsc/scripts/malicious-shared-lib.c)
        - run ldd against a sudoable program to see what shared libraries are used: `ldd {sudoable program}`
            - might need to attempt this attack on different files (some work better than others)
        - compile malicious lib file: `gcc -o {filename} -shared -fPIC malicious-shared-lib.c`
            - filename must be exactly the same as the targeted shared library file from previous step
        - `sudo LD_LIBRARY_PATH=/path/to/filename {sudoable program}`
4. Cron Jobs
    - if you can write to files associated with cron jobs, then you can have the cron job run a malicious payload
        - payload will likely be custom depending on how the cronjob is using the writable file. often possible to use a bash reverse shell. 
    - if the cron job script does not use an absolute path and one of the path dirs is writable, you can create amalicious script with the same name and the job will run it
        - cat crontab files to see if the PATH variable is set, and check to see if any dirs are writable
        - handy command to use when waiting for a cronjob to run: `watch -n 1 ls -l /path/to/target/dir`
    - if a wildcard is used in a cronjob, like a job that creates backups of all the files in a dir using a wildcard, this can be exploited by planting a file with a malicious payload as the filename in that directory.
        - utilize [GTFOBins](https://gtfobins.github.io/) for ideas on how to exploit the commands using the wildcard
5. SUID/SGID executables
    - identify SUID/SGID files
    - use shell escape sequences with SUID / SGID files
        - utilize [GTFOBins](https://gtfobins.github.io/) for ideas
        - env variables `LD_LIBRARY_PATH` and `LD_PRELOAD` get ignored for SUID/SGID files
    - use known exploits for SUID files (find exploits the same way you would find service/program exploits)
    - run `strace` on SUID/SGID files and identify if any shared objects are not found. If the location of this shared object is in a writable directory, you can create a malicious shared object
        - `strace /path/to/SUID/file 2>&1 | grep -iE "open|access|no such file"`
            - `2>&1` is important because it'll redirect stderr to stdout before piping to grep
        - compile and use exploit in the same fashion as the` malicious-shared-lib.c` process described in step 3
    - if a SUID/SGID runs a program by name, rather than by absolute path (i.e. `bash` and not `/bin/bash`) you can place a malicious executable of the same name in a directory that gets searched before the location of the real program.
        - since $PATH is modifiable by a user, you can edit the PATH to look in the user's home directory, or wherever else you have write access to. 
        - use `strings`, `strace`, or `ltrace` to identify what programs a SUID/SGID program might be using. 
        - drop a rev shell executable in the target directory
        - change your path and run the SUID: `PATH=.:$PATH /path/to/suid-program`
    - in some shells before Bash < 4.2-048 user functions with absolute path names can be defined and can be exported to child processes. These functions will take precedence over the actual executable being called.
        - verify bash version is less than 4.2-048: `bash --version`
        - create a malicious function with the same name as the vulnerable SUID/SGID program & export the function
            - `function /path/to/suid-program { /bin/bash -p; }`
            - `export -f /path/to/suid-program`
        - execute the suid program
    - if a SUID file runs another program via bash (e.g. by using system, execve, etc.), env variables can be inherited. If bash version <4.4 and the SUID program is running commands with bash, then you can run a SUID program with debugging enabled and insert a malicious payload into the PS4 environment variable (i.e. command substitution or running a malicious executable) 
        - `env -i SHELLOPTS=xtrace PS4='$(cp /bin/bash /tmp/rootbash; chown root /tmp/rootbash; chmod +s /tmp/rootbash)' /path/to/suid-program`
6. Passwords & Keys
    - weak password storage and password re-used
        - services might store passwords in plaintext config files and these passwords might be reused by root account
        - check history files for commands that may have taken passwords as CL args                                                                                                                                               
        - check config files of services (i.e. openvpn)
    - weak file permissions on SSH keys
7. NFS
    - Network files systems allow remote users to access, create, and modify files. These files inherit the remote user's id and group id even if they do not exist on the NFS server. If "root squashing" is disabled  (`no_root_squash`) and included in a writable share configuration, a remote "root" can create files on the share as the local root.  
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
- [g0tmi1k's Linux PrivEsc Guide](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/)