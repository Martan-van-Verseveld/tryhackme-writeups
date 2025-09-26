first scan network using nmap
- found port 62337 open tcp, has http response.
- open on browser
- use user: "john", password: "password" and successfull login.

# Objective
## Task 1 - Flags
Gain a shell on the box and escalate your privileges!
- user.txt
- root.txt


# Solution
The solution to obtaining these two flags were to scan, enumerate and infultrate :).

## nmap scan
The scan I run is the following: `nmap -sC -sV -Pn -T4 -p- -vvv 10.10.34.94`, this will gather any response information, tries to identify the services ran and scan all 65536 ports from the host 10.10.34.94 (in my case).
```bash
# Nmap 7.95 scan initiated Fri Sep 26 05:56:22 2025 as: /usr/lib/nmap/nmap -sC -sV -Pn -T4 -p- -vvv -o nmap_results.txt 10.10.34.94
Nmap scan report for 10.10.34.94
Host is up, received user-set (0.026s latency).
Scanned at 2025-09-26 05:56:22 EDT for 36s
Not shown: 65531 closed tcp ports (reset)
PORT      STATE SERVICE REASON         VERSION
21/tcp    open  ftp     syn-ack ttl 63 vsftpd 3.0.3
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:10.8.48.62
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp    open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 e2:be:d3:3c:e8:76:81:ef:47:7e:d0:43:d4:28:14:28 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC94RvPaQ09Xx+jMj32opOMbghuvx4OeBVLc+/4Hascmrtsa+SMtQGSY7b+eyW8Zymxi94rGBIN2ydPxy3XXGtkaCdQluOEw5CqSdb/qyeH+L/1PwIhLrr+jzUoUzmQil+oUOpVMOkcW7a00BMSxMCij0HdhlVDNkWvPdGxKBviBDEKZAH0hJEfexz3Tm65cmBpMe7WCPiJGTvoU9weXUnO3+41Ig8qF7kNNfbHjTgS0+XTnDXk03nZwIIwdvP8dZ8lZHdooM8J9u0Zecu4OvPiC4XBzPYNs+6ntLziKlRMgQls0e3yMOaAuKfGYHJKwu4AcluJ/+g90Hr0UqmYLHEV
|   256 a8:82:e9:61:e4:bb:61:af:9f:3a:19:3b:64:bc:de:87 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBBzKTu7YDGKubQ4ADeCztKu0LL5RtBXnjgjE07e3Go/GbZB2vAP2J9OEQH/PwlssyImSnS3myib+gPdQx54lqZU=
|   256 24:46:75:a7:63:39:b6:3c:e9:f1:fc:a4:13:51:63:20 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJ+oGPm8ZVYNUtX4r3Fpmcj9T9F2SjcRg4ansmeGR3cP
80/tcp    open  http    syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
| http-methods:
|_  Supported Methods: POST OPTIONS HEAD GET
62337/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))
|_http-favicon: Unknown favicon MD5: B4A327D2242C42CF2EE89C623279665F
|_http-title: Codiad 2.8.4
|_http-server-header: Apache/2.4.29 (Ubuntu)
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Sep 26 05:56:58 2025 -- 1 IP address (1 host up) scanned in 36.46 seconds
```
Using this method we've found out that on port 62337 there's a webapp running called "Codiad" with version 2.8.4, this is a web portal code editor. When we try to open this page on the browser we are met with a login screen and we've hit our first wall.

## ftp
In the nmap scan we find out that there's an ftp server running on port 21 (default ftp port).
We can access the ftp on readonly using username "anonymous" and no password, after running a simple 'ls -alh' to list the files we find a weird folder called "...", we enter into here and find a files called "-", we copy this over to our host machine and read it.
The file we've copied over to our host contains the following:
```bash
Hey john,
I have reset the password as you have asked. Please use the default password to login.
Also, please take care of the image file ;)
- drac.
```
So now we know there's atleast two users called "john" and "drac". We focus on the use John first.

## Codiad
We move back to the webapp Codiad and try the user "john" with the most simple password "password", and... IT WORKS!
We're now logged into Codiad as the user john, first we check searchsploit if there's any exploits for this version of Codiad `searchsploit "Codiad 2.8.4"`:
```bash
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                                                                         |  Path
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Codiad 2.8.4 - Remote Code Execution (Authenticated)                                                                                                                                                                                                   | multiple/webapps/49705.py
Codiad 2.8.4 - Remote Code Execution (Authenticated) (2)                                                                                                                                                                                               | multiple/webapps/49902.py
Codiad 2.8.4 - Remote Code Execution (Authenticated) (3)                                                                                                                                                                                               | multiple/webapps/49907.py
Codiad 2.8.4 - Remote Code Execution (Authenticated) (4)                                                                                                                                                                                               | multiple/webapps/50474.txt
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```
