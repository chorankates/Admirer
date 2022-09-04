# [22 - Admirer](https://app.hackthebox.com/machines/Admirer)

![Admirer.png](Admirer.png)

## description
> 10.10.10.187

## walkthrough

### recon

```
 $ nmap -sV -sC -A -Pn -p- admirer.htb
Starting Nmap 7.80 ( https://nmap.org ) at 2022-09-03 16:48 MDT
Nmap scan report for admirer.htb (10.10.10.187)
Host is up (0.062s latency).
Not shown: 65532 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
| ssh-hostkey:
|   2048 4a:71:e9:21:63:69:9d:cb:dd:84:02:1a:23:97:e1:b9 (RSA)
|   256 c5:95:b6:21:4d:46:a4:25:55:7a:87:3e:19:a8:e7:02 (ECDSA)
|_  256 d0:2d:dd:d0:5c:42:f8:7b:31:5a:be:57:c4:a9:a7:56 (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
| http-robots.txt: 1 disallowed entry
|_/admin-dir
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Admirer
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

haven't seen ftp in a minute - also thanks for looking at robots.txt contents nmap

### 80

waiting for nmap, presumed web.

pretty basic image gallery with some 'cute' phrases under 'inspiring' images

towards the bottom:
> <h2>Get in touch</h2>
> <form method="post" action="#"><!-- Still under development... This does not send anything yet, but it looks nice! -->


`/admin-dir` gives a 403

the actual contents of robots.txt

```
User-agent: *

# This folder contains personal contacts and creds, so no one -not even robots- should see it - waldo
Disallow: /admin-dir

```

probably a username in `waldo`

```
$ gobuster dir -u http://admirer.htb/admin-dir -x txt -r -t 40 -w ~/git/ctf/tools/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://admirer.htb/admin-dir
[+] Method:                  GET
[+] Threads:                 40
[+] Wordlist:                /home/conor/git/ctf/tools/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              txt
[+] Follow Redirect:         true
[+] Timeout:                 10s
===============================================================
2022/09/03 17:02:09 Starting gobuster in directory enumeration mode
===============================================================
/contacts.txt         (Status: 200) [Size: 350]
```

oi, that's a bit silly.

a few more usernames
  * p.wise
  * r.nayyar
  * a.bialki
  * l.galecki
  * h.helberg
  * b.rauch

robots.txt said contacts _and creds_, so guessing we need to use these names to find their creds via gobuster

or just keep waiting, because

```
/credentials.txt      (Status: 200) [Size: 136]
```

nice.

```
$ cat credentials.txt 
[Internal mail account]
w.cooper@admirer.htb
fgJr6q#S\W:$P

[FTP account]
ftpuser
%n?4Wz}R$tTF7

[Wordpress account]
admin
w0rdpr3ss01!
```

### 21

```
$ ftp admirer.htb
Connected to admirer.htb.
220 (vsFTPd 3.0.3)
Name (admirer.htb:conor): anonymous
530 Permission denied.
ftp: Login failed
```

hrm. msfconsole shows an exploit for vsftpd 2.3.4, and exploitdb only shows a DoS for 3.0.3


## flag

```
user:
root:
```
