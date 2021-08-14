# Linux PrivEsc Tools
Helpful scripts/tools for Linux PrivEsc 
# Enumeration Tools
## Linux Smart Enumeration 
script that will gradually expose privesc info depending on its importance
- [GitHub Repo](https://github.com/diego-treitos/linux-smart-enumeration)
- One-liner: `wget "https://github.com/diego-treitos/linux-smart-enumeration/raw/master/lse.sh" -O lse.sh;chmod 700 lse.sh`
- `-l {0,1,2}` 0-2 args can be used to show most important results (0 is default, most important)

## LinEnum
script that extracts large amount of info from target system. also greps files for common password keywords (or chosen keyword). 
- [GitHub Repo](https://github.com/rebootuser/LinEnum)
- ` ./LinEnum.sh -s -k keyword -r report -e /tmp/ -t`
- `-e {dir}` to export interesting files to a directory. makes it easier to analyze these files if you copy the directory to your local machine.

## Linuxprivchecker
- [GitHub Repo](https://github.com/linted/linuxprivchecker)

## BeRoot
- [GitHub Repo](https://github.com/AlessandroZ/BeRoot)
- It has been added to the pupy project as a post exploitation module (so it will be executed in memory without touching the disk).

## unix-privesc-check
- [Download](https://pentestmonkey.net/tools/audit/unix-privesc-check)
- preloaded into most kali VMs at `/usr/share/unix-privesc-check/unix-privesc-check`

# Exploitation Tools
## GTFO Bins
Tool to discover and exploit Unix binaries that can be used to bypass local security restrictions in misconfigured systems. 
- [Github Pages](https://gtfobins.github.io/)

# Utilities
## Reverse Shell Generator
Tool that can be used to generate reverse shells natively
- [GitHub Repo](https://github.com/mthbernardes/rsg)
- Create a python reverse shell: `rsg {IP-address} {port} python`
