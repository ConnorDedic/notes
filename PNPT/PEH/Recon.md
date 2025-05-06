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
- BurpSuite
- ZAP
- Wappalyzer
