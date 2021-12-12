# Reconnaissance, enumeration, information gathering
The goal of this phase is to thoroughly understand your target, enumerate vulnerabilities, and determine what the best path to exploitation is. 
# Passive reconnaissance
Review openly available information without direct interaction with the target.
# Active reconnaissance 
Information gathering techniques that involve direct interaction with target services.
## DNS
Query DNS servers using host, nslookup, dig, or other tools.
- Important record types: NS, A, MX, PTR, CNAME, TXT
- Some nameservers will disallow zone transfers, some will allow. This may even differ across the same org's nameservers so query each one. 
    - `host -l {name-server}` 
    - sometimes even internal DNS namespace will be exposed in exceptionally bad misconfiguration.
- DNSRecon - python script that automates lots of DNS tasks
    - `dnsrecon -d {domain} -t std`
## nmap
golden standard port scanning tool with advanced features
- `nmap -sC -A -p- {ip-address}`
## ffuf
fuzzing tool that can be used to discover subdomains, hidden directories/files, vhosts, etc. 
- subdomains
    - `ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://FUZZ.{domain}`
- hidden directories/files
    - `ffuf -w /opt/useful/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://{domain}/FUZZ`
    - `ffuf -w /opt/useful/SecLists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://domain/indexFUZZ`
- vhosts
# All-in-one tools
## AutoRecon 
multithreaded reconnaisance tool that performs port scanning, service detection, etc. Useful for CTFs. 
- [GitHub Repo](https://github.com/Tib3rius/AutoRecon)
- `autorecon 127.0.0.1 {ip-address}`