	**Scanning**
![[Pasted image 20250412152832.png]]
![[Pasted image 20250412152912.png]]
]]![[Pasted image 20250412154217.png]]

```
ffuf -c -w <wordlist> -u <url>/FUZZ
```

add blackpearl.tcm to /etc/hosts

backup passwords maybe in /var/backups/shadow.bak

/var/log/nginx/access.log
/var/www/html/secret

interesting bins
/usr/bin/php7.3
chfn
passwd

may be able to change sites

writeable listeners

always go for a tty shell if you have a worse shell if possible

use a command like [(see here)](https://wiki.zacheller.dev/pentest/privilege-escalation/spawning-a-tty-shell)
```
python -c 'import pty; pty.spawn("/bin/sh")'
```
