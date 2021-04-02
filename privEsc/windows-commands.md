# Basic Enumeration Commands
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
- show services 
    - `sc query state=all`

## Enumerating files
- search registry for a password
    - `reg query HKLM /f password /t REG_SZ /s`
- search files for passwords
    - `findstr /si password *.txt | *.xml | *.xls`
- tree of C drive
    - `tree /F /A c:\ > tree.txt`
-search for all PDFs
    - `dir /a /s /b c:\*.pdf*`

# Other
- disable firewall
    - `netsh advfirewall set currentprofile state off`
    - `netsh advfirewall set allprofiles state off`
- DNS zone transfer
    - `nslookup, set type=any, ls -d domain > results.txt, exit`
