Target: 10.10.44.180
Machine type: Linux (Ubuntu 3)

ports found open
22 OpenSSH9.1
80 nginx 1.18.0
8080 http-proxy
![[Pasted image 20250202000322.png]]


Webpage found
DIRs
/images
/assets
/#
:8080/silverpeas
![[Pasted image 20250201235807.png]]


potential user scr1ptkiddy 
potential service silverpeas

silverpeas confirmed
silverpeas version 2001-2022

potential exploits

https://github.com/RhinoSecurityLabs/CVEs/tree/master/CVE-2023-47323
https://rhinosecuritylabs.com/research/silverpeas-file-read-cves/
https://github.com/advisories/GHSA-4w54-wwc9-x62c

Exploit 1 (gaining access)
Use burp to catch the packet and edit bypass auth
![[Pasted image 20250206131342.png]]

![[Pasted image 20250206131417.png]]

![[Pasted image 20250206131440.png]]

![[Pasted image 20250206132019.png]]

Sweet! We are in!
![[Pasted image 20250206132602.png]]
lets try the exploit for unathed message access and look through some messages

http://10.10.195.213:8080/silverpeas/RSILVERMAIL/jsp/ReadMessage.jsp?ID=6
![[Pasted image 20250206132816.png]]
"Dude how do you always forget the SSH password? Use a password manager and quit using your silly sticky notes. ""

Username: tim
Password: "cm0nt!md0ntf0rg3tth!spa$$w0rdagainlol"

cat /var/log/auth* | grep -i pass


Once we find the passsword we try to su to root and it works!