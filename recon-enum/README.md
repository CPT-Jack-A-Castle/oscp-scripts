# Reconnaissance, enumeration, information gathering
The goal of this phase is to thoroughly understand your target and the attack surface, and to determine what the best path to exploitation is. 

# Passive reconnaissance
Review openly available information without direct interaction with the target.

## Offsec's examples for passive recon:
- Company Website
- Whois enumeration
- Google dorking: [Google Hacking Db](https://www.exploit-db.com/google-hacking-database)
    - `filetype:php site:megacorpone.com`
    - `site:megacorpone.com -filetype:html`
    - `intitle:"index of" "parent directory"`
- [Netcraft Site Report](https://sitereport.netcraft.com/) 
    - Has whois inofmration and site technology
- [Recon-ng](https://github.com/lanmaster53/recon-ng)
    - `recon/domains-hosts/bing_domain_web`
    - `recon/domains-hosts/censys_domain`
    - `recon/domains-hosts/certificate_transparency`
    - `recon/domains-hosts/google_site_web`
    - `recon/domains-hosts/netcraft`
    - `recon/domains-hosts/spyse_subdomains`
    - `recon/domains-hosts/shodan_hostname`
    - `recon/domains-credentials/pwnedlist/leak_lookup`
- TheHarvester
    - `theharvester -d megacorpone.com -b bing`
- [shodan](https://www.shodan.io/) and list of [shodan filters](https://github.com/JavierOlmedo/shodan-filters)
- [security headers](https://securityheaders.com/) - scan a site for missing security headers
- [SSL Server Test](https://www.ssllabs.com/ssltest/)
- [pastebin](https://pastebin.com/)
- GitHub, gitlab, sourceforge, stackoverflow
    - `filename:users` `filename:conf` search operators while browsing repositories
    - [gitrob](https://github.com/michenriksen/gitrob) 
    - [gitleaks](https://github.com/zricethezav/gitleaks)
 - [linkedin2username](https://github.com/initstring/linkedin2username) - tool to make usernames from linkedin accounts
    - `python3 linkedin2username.py -u sockaccount@email.com -c <company name> -s 20 -j 10 -k "executive, sales, accountant, information, cyber, engineer, chief, vice, president, human, resources, technology, business, intelligence"`

# Active reconnaissance 
Information gathering techniques that involve direct interaction with target services.

## DNS
Query DNS servers using host, nslookup, dig, or other tools.
- Important record types: NS, A, MX, PTR, CNAME, TXT
- Some nameservers will disallow zone transfers, some will allow. This may even differ across the same org's nameservers so **query each one**. 

- zone transfer 
    - `host -l {name-server}` 
    - sometimes even internal DNS namespace will be exposed in exceptionally bad misconfiguration.
    - `dnsrecon -d {domain} -t axfr`
- DNSRecon - python script that automates lots of DNS tasks
    - `dnsrecon -d {domain} -t std`
- dnsenum
    - `dnsenum {domain}`

## nmap
golden standard port scanning tool with advanced features
- `nmap -sC -A -p- {ip-address}`
- `nmap -sS -sU {ip-address}`
- `nmap -sn {ip-range} -oG ping-sweep.gnmap`
- OffSec mentioned NSE scripts: `smb-os-discovery` , `dns-zone-transfer`


## web apps
techniques for discovering web applciation vulnerabilities
- `nmap -p 80,443 --script "http-headers, vulners" {ip-range}`
- fuff - fuzzing tool that can be used to discover subdomains, hidden directories/files, vhosts, etc. 
    - subdomains
        - `ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://FUZZ.{domain}`
    - hidden directories/files/extensions , smallish
        - `ffuf -w /opt/useful/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u http://{domain}/FUZZ/ -recursion`
        - `ffuf -w /opt/useful/SecLists/Discovery/Web-Content/raft-large-files.txt:FUZZ -u http://{domain}/FUZZ -recursion`
        - `ffuf -w /opt/useful/SecLists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://{domain}/indexFUZZ`
    - beefy lists:
        - `ffuf -w /opt/useful/SecLists/Discovery/Web-Content/combined_directories.txt:FUZZ -u http://domain/FUZZ -recursion`
        - `ffuf -w /opt/useful/SecLists/Discovery/Web-Content/combined_words.txt:FUZZ -u http://domain/FUZZ -recursion`
    - vhosts
        - `ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u https://{ip-address}/ -H 'Host: FUZZ.{domain}.com'`
- [nikto](https://github.com/sullo/nikto) - web server vulnerability scanner
    - `nikto {ip-address} -timeout 20` (default port 80)
- manual inspection
    - check `robots.txt` , `.svn`, `.DS_STORE`, and view the page source
### Offsec Staff recommended wordlists
- `/usr/share/wordlists/SecLists/Discovery/Web-Content/raft-large-files.txt`
- `/usr/share/wordlists/SecLists/Discovery/Web-Content/raft-large-directories.txt`

## SMB
- Scan a network range for SMB enabled hosts
    - `nmap -v -p 139,445 -oG smb.gnmap {ip-range}`
- Run NSE scripts to enumerate SMB enabled hosts
    - `nmap -Pn -p 445 --script "((smb* and safe) and not intrusive) or duplicates" -iL smb-hosts.txt`
- NBTscan is a program for scanning IP networks for NetBIOS name information
    - `sudo nbtscan -r {ip-address}`
- `enum4linux -a -v {ip-address}`

## NFS
- enumerate hosts for RPC on port 111 and look for NFS shares 
    - `nmap -sV -p 111 --script=rpcinfo {ip-range}`
- run NFS NSE scripts for hosts with discovered NFS shares
    `nmap -p 111 --script=nfs* {ip-address}`
- attempt to mount any discovered NFS shares
    - `sudo mount -o nolock {ip-address}:{share-name} {~/path/to/local/folder}`
    - if you get perimssions issues trying to open up certain files, look at course notes for the workaround (ch 7.4)

## SMTP
versions 1 and 2c rely on community strings for security
- `nmap -p 25 --script "smtp-commands smtp-open-relay" {ip-address}`
- [smtp-user-enum](https://www.kali.org/tools/smtp-user-enum/) tool that can be usd to find users
    - `smtp-user-enum  -U wordlist.txt -t {ip-address} -v -w 20`

## SNMP
- `sudo nmap -sU --open -p 161 {ip-range} -oG snmp.gnmap`
- discover potential community strings
    - `onesixtyone -c /usr/share/wordlists/SecLists/Discovery/SNMP/snmp.txt -i snmp-hosts.txt`
- when community strings are known
    - enumerate entire MIB Tree: `snmpwalk -v {1|2c} -c {community-string} {ip-address}`
    - enumerate windows users: `snmpwalk -v {1|2c} -c {community-string} {ip-address} 1.3.6.1.4.1.77.1.2.25`
    - enumerate windows processes: `snmpwalk -v {1|2c} -c {community-string} {ip-address} 1.3.6.1.2.1.25.4.2.1.2`
    - enumerate open TCP ports: `snmpwalk -v {1|2c} -c {community-string} {ip-address} 1.3.6.1.2.1.6.13.1.3`
    - enumerate installed software: `snmpwalk -v {1|2c} -c {community-string} {ip-address} 1.3.6.1.2.1.25.6.3.1.2`
- [braa](https://github.com/mteg/braa) - tool to bruteforce OIDs and their relevant info

### Useful OIDs
| String | Information | 
| ---- | ---- |
| 1.3.6.1.2.1.25.1.6.0 |    System Processes |
| 1.3.6.1.2.1.25.4.2.1.2 |  Running Programs |
| 1.3.6.1.2.1.25.4.2.1.4 | Processes Path |
| 1.3.6.1.2.1.25.2.3.1.4 |  Storage Units | 
| 1.3.6.1.2.1.25.6.3.1.2 |  Software Name | 
| 1.3.6.1.4.1.77.1.2.25 |   User Accounts | 
| 1.3.6.1.2.1.6.13.1.3 |    TCP Local Ports |


# All-in-one tools
## AutoRecon 
multithreaded reconnaisance tool that performs port scanning, service detection, etc. Useful for CTFs. 
- [GitHub Repo](https://github.com/Tib3rius/AutoRecon)
- `autorecon 127.0.0.1 {ip-address}`