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

# Enumerating files
- `find / -writable -type d 2>/dev/null` 
- `find / -type f \( -iname "*.conf" -or -iname "*.config" \)  -writable  -exec ls -lthr "{}" 2>/dev/null  \;`
- `ls -la ~/.ssh`

# Enumerating Scheduled Tasks
- `ls -lah /etc/cron*`
- `cat /etc/crontab`