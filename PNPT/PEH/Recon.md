# Network Scanning
### NMAP
Host Discovery
```
sudo netdiscover -a <int> -r <network>/<CIDR>
```

**Network Scan**
```
nmap <ip>/<CIDR>
```
**IP Scan**
```
nmap <ip>
```

| Tag | Syntax                   |
| --- | ------------------------ |
| -p  | Port                     |
| -sV | Service enumeration      |
| -sT | TCP                      |
| -sU | UDP                      |
| -Pn | Ping                     |
| -sS | Stealth                  |
| -sC | Default Script           |
| -t  | timing (1-4)             |
| -sX | Christmas (all bits on)  |
| -O  | Operating System         |
| -A  | OS, Ver, Script, tracert |
| -v  | Verbose                  |
| -h  | help                     |


### SSH Enumeration

Start with connecting
```
ssh <ip>
```
Check for encryption
```
ssh <ip> -oKexAlgorithims=+<algorithim>
```


### SMB Enumeration

**Metasploit tools**
```
use auxiliary/scanner/smb/smb_version
```
**Try Connecting**
```
smbclient -L\\\\<ip>\\
smbclient -L\\\\<ip>\\ADMIN$
smbclient -L\\\\<ip>\\IPC$
```

**Detect SMB Policy
```
nmap --script=smb2-security-mode.nse -p 445 <target>
```
 Look for 
```
Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

```

Troubleshooting
**Question**: My enum4linux and/or smbclient are not working. I am receiving "Protocol negotiation failed: NT_STATUS_IO_TIMEOUT". How do I resolve?
**Resolution**:
On Kali, edit /etc/samba/smb.conf
Add the following under global:
client min protocol = CORE
client max protocol = SMB3

# Web Scanning
### HTTP/S Enumeration

Nikto
```
nikto -h http://<host>
```
FFUF
```
ffuf -c -w <wordlist> -u <url>/FUZZ
```
Note that this removes file size exact match 
```
-fs <bytes>
```

- BurpSuite
- ZAP
- Wappalyzer

**Assetfinder**
[[Heath Subdomain Script]]
Install
```
go get -u github.com/tomnomnom/assetfinder
```
Show subdomains & related
```
assetfinder <domain>
```
More Strict
```
assetfinder --subs-only <domain>
```
**Amass**
[Github](https://github.com/owasp-amass/amass)
[[Heath Subdomain Script]]

**HTTProbe**
*Use to identify active subdomains*
[[Heath Subdomain Script]]
```
cat <domain list> | httprobe
```
```
cat <domain list> | httprobe -s -p https:443
```
```
cat <domain list> | httprobe -s -p https:443 | sed 's/https\?:\/\///' | tr -d ':443'
```
**Want to find cool webpages?**
```
cat <domain wordlist> | grep dev
cat <domain wordlist> | grep test
cat <domain wordlist> | grep stag
cat <domain wordlist> | grep admin
```
**GoWittness**
*Takes screenshots of web pages*
```
go get -u https://github.com/sensepost/gowitness.git
```
Scan one webpage
```
gowittness single <url>
```
**MISC**
[BB Methodology](https://www.youtube.com/watch?v=uKWu6yhnhbQ)
[Recon Playlist](https://www.youtube.com/watch?v=MIujSpuDtFY&list=PLKAaMVNxvLmAkqBkzFaOxqs3L66z2n8LA)
