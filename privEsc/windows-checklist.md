# Windows Privilege Escalation Notes & Checklist
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
        - if you have write access to a DLL loaded by a service or, more commonly, if the DLL is missing and you have write access to a directory in the Windows PATH 
    - `winPEAS quiet servicesinfo` to check for this
2. Registry Exploits
    - AutoRuns
        - Autoruns are commands that run at startup and are configured in the registry. If you have write access to a program that runs at startup, you can overwrite that program with your malicious program. You must be able to restart the system, or expect it to restart in order to utilize these for PrivEsc.
        - On Windows 10, the exploit will run with the privs of the last logged in user.
        - `winPEAS quiet applicationsinfo` to check for this
    - AlwaysInstallElevated
        - MSI Files are used to install applications. If Windows allows you to run an MSI file with elevated privs, you can create and run a malicious MSI file using msfvenom to elevate privs.
        - Requires two registry settings to be enabled
            - System Level: `HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer` 
            - User Level: `HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer`
        - `winPEAS quiet windowscreds` to check for this 
3. Passwords
    - Passwords saved in registry
        - `winPEAS quiet filesinfo userinfo ` to check for this
            - only looks in certain locations, can query registry directly for a full search. 
    - Saved credentials 
        - Windows' runas command, which allows users to run commands with the privs of other uses, can let users save their credentials on the system.
        - `winPEAS quiet cmd windowscreds` to check for this
    - Configuration files
        - Search for passwords in files or for configuration files. Don't start at system root unless necessary. Start with home directories, temp directories, or sketchy program directories.
        - `winPEAS quiet cmd searchfast filesinfo` to check for this
    - SAM
        - SAM file contains user password hashes and the hashes are encrypted with a key found in the SYSTEM file. 
        - Potential file locations:
            - `C:\Windows\System32\config`
            - `C:\Windows\Repair`
            - `C:\Windows\System32\config\RegBack`
        - if readable, copy these files back to kali, dump them with pwdump, and try to crack with hashcat
    - Pass the Hash
        - Some services will accept hashes instead of passwords. If you can grab hashes you can use pth-winexe to pawn a command prompt
4. Scheduled Tasks 
    - tasks that get executed periodically or triggered by an event. Usually at the privs of the creator, but can be made to run as SYSTEM or admin.
    - can sometimes query for these, but usually have to rely on clues like finding scripts or log files
5. Insecure GUI Apps 
    - on older versions of Windows, certain GUI apps run with administrator privileges. You can try to spawn a command prompt from within the app with builtin Windows functionality.
        -  For example, if a program has the ability to open a file, you may try to open a command prompt by typing `file://c:/windows/system32/cmd.exe` 
6. Startup Apps
    - Programs to run at startup by creating shortcuts (.lnk files) to the programs in `C:\ProgramData\Microsoft\Windows\Start Menu\ Programs\StartUp` (Global directory)
    - if you can write files in this directory, you can drop a reverse shell executable here. 
7. Installed Applications
    - Enumerate running programs and notice anything "interesting"
        - `tasklist /v`
        - `./seatbelt.exe NonstandardProcesses`
        - `./winPEASany.exe quiet processinfo`
    - Check versions using /? , -h, or config files in the Program Files directory
    - Search exploitdb for a corresponding exploit
8. Hot Potato
    - Spoofing + NTLM relay attack. works on Windows 7, 8, and early versions of 10
    - [Blog] (https://pentestlab.blog/2017/04/13/hot-potato/)
9. Token Impersonation
    - Service accounts usually have SeImpersonate and SeAssignPrimaryToken privileges enabled, which allow the Service account to impersonate the access tokens of users, including SYSTEM. 
    - Juicy Potato, Rogue Potato, PrintSpoofer
10. Port Forwarding
    - Run exploit code on calling and forward it to an internal port on Windows using plink
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

# Resources
- [HackTricks Book Checklist](https://book.hacktricks.xyz/windows/checklist-windows-privilege-escalation)
- [pwdump](https://github.com/Neohapsis/creddump7.git)
- [PayloadsAllTheThings' Windows PrivEsc Methods](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md)