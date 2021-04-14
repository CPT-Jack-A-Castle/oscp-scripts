# Windows PrivEsc Tools
Helpful recon scripts for Windows PrivEsc 

## WinPEAS
Tool that looks for privilege escalation misconfigurations and highlights them for the user in the results. Also possible to run certain checks individually. Avaiable as an exe and a bat.
- [GitHub Repo](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)
- Before running, add a registry key: `reg add HKCU\Console /v VirtualTerminalLevel /t REG_DWORD /d 1` (enables colors)
    - Might still be able to see colors if run through a reverse shell
- Reopen command prompt
- Run all checks while avoiding time-consuming searches: `.\winPEASany.exe quiet cmd fast`
- Run specific check category: `.\winPEASany.exe quiet cmd systeminfo`

## Seatbelt
Enumeration tool that outputs system info related to PrivEsc, but doesn't recommend any specific escalation. Output requires further investigation. 
- [GitHub Repo](https://github.com/GhostPack/Seatbelt)
- [Precompiled Binary](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries/blob/master/Seatbelt.exe)
- Possible to run specific checks, all checks, or selected combinations checks.
- Print help message (list available checks): `.\SeatBelt.exe`
- Run all enumeration checks: `.\SeatBelt.exe all`

## PowerUp & SharpUp
Useful to find a specific subset of privilege escalation misconfigurations. Should not be used as a standalone tool.
### PowerUp 
- written in powershell
- [GitHub Raw](https://raw.githubusercontent.com/PowerShellEmpire/PowerTools/master/PowerUp/PowerUp.ps1)
- to run from a cmd prompt 
    - open powershell with: `powershell -exec bypass`
    - dot source the script: `. .\PowerUp.ps1`
    - Invoke the AllChecks function: `Invoke-AllChecks` 
### SharpUp 
- C# version of PowerUp, useful when powershell isn't available. 
- [GitHub Repo](https://github.com/GhostPack/SharpUp)
- [Precompiled Binary](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries/blob/master/SharpUp.exe) 
- `.\Sharpup.exe`

## Access Check
Tool to check user access control rights on files, directories, services, and registry keys. However, it prompts the user to accept a EULA (usually only possible through RDP). If using a command line, must use an older version which still has the /accepteula CL flag.
- [Microsoft Docs](https://docs.microsoft.com/en-us/sysinternals/downloads/accesschk)