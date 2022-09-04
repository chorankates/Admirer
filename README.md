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
```




## flag

```
user:
root:
```
