
## SQL Injection (SQLi)
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


## AUTHN Attacks
## External Entities Injection (XXE)
## Insecure Direct Object Reference (IDOR)
