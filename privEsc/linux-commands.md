# Basic Enumeration Commands
## Users and Groups 
- `whoami` , `id` 
- `cat /etc/passwd`
- `hostname`

##  OS/Arch
- `cat /etc/issue` 
- `uname -a`

## Processes/Applications
- `ps aux`
- `dpkg -l`

## Enumerating files
- `find / -writable -type d 2>/dev/null` 
- `find / -type f \( -iname "*.conf" -or -iname "*.config" \)  -writable  -exec ls -lthr "{}" 2>/dev/null  \;`
- `ls -la ~/.ssh`
- binaries with irregular timestamps
    - `for i in /usr/sbin /usr/bin /sbin /bin /snap/bin; do ls -la --time-style=full $i | grep -v '0000\|->' ; done`
    - exclude /local paths because it's not uncommon for users to create binaries in those directories. non-local directories are supposed to be managed by the system package manager.
    - exclude sym links because normal for those timestamps to have non-zero seconds. Packages installed with the system's package manager should have 0's for the seconds in the timestamp.

## Enumerating Scheduled Tasks
- `ls -lah /etc/cron*`
- `cat /etc/crontab`