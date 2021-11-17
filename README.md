# Network Attacks Final Project

#### **ITN 261 - Network Attacks and Hacking**

###### Ronald Phillips



#### I. Introduction

​	This scenario will demonstrate an attack against two target machines.  The machines are installed per the source instructions in default configurations.  Due to the nature of the process, the root username and passwords are known ahead of time, but will not be used unless they can be extracted from the machines themselves.

​	The target IP addresses were obtained by a simple Nmap scan.

​	The attack box has the latest release of Kali Linux, and has had Nessus, armitage, and a few other extra utilities installed.  If possible, I plan to restrict the use of the msfconsole or armitage, and try to identify vulnerabilities and develop my own attacks.  The use of the MSF framework would make exploitation of both machines trivial. 



<u>**Topology**</u>

![Topology](/Topology.png)



| target                                                       | target2                                        | kali                                |
| ------------------------------------------------------------ | ---------------------------------------------- | ----------------------------------- |
| [Metasploitable](https://information.rapid7.com/download-metasploitable-2017.html) | [Damn Vulnerable Web App](https://dvwa.co.uk/) | [Kali Linux](https://www.kali.org/) |
| 192.168.1.209                                                | 192.168.1.210                                  | 192.168.1.86                        |
| msfadmin:msfadmin                                            | dvwa:dvwa                                      | kali:kali                           |



#### II. Target 1: Metasploitable

​	This machine is an imported Virtual Machine imported  into ESXi of [Metasploitable](https://information.rapid7.com/download-metasploitable-2017.html).  It is an intentionally vulnerable Linux machine designed for Penetration Testing.  The goal for this target will be to explore some of the more interesting vulnerabilities (because SMB is too easy to break).

 

##### A) Enumeration

##### 192.168.1.209

To begin, I usually start with a basic script scan using Nmap.  The output file containing results can be found [here](https://git.rjphillips.online/main/networkattacksproject/-/blob/main/target/scans/nmapinitial).



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

​	There are many open ports with at first glance looks like many different attack vectors.  In order to help decide where to start, we scan once more with Nmap, this time using the "--script=VULN" parameter.  This will apply a set of scripts to the machine that will attempt to identify known vulnerabilites 





#### II. Target 2: Damn Vulnerable Web Application (DVWA)

​	This machine is running the latest Ubuntu (21.04), and has had [Damn Vulnerable Web App](https://dvwa.co.uk/) installed in its default configuration.  

##### A) Enumeration

##### 192.168.1.210

​	Once again, I will start with a default script scan on the target through Nmap.  The results can be found [here](https://git.rjphillips.online/main/networkattacksproject/-/blob/main/target2/scans/nmapinitial). This time, the results are drasticly different. 

| Port   | State | Service | Version                         |
| ------ | ----- | ------- | ------------------------------- |
| 22/tcp | open  | ssh     | OpenSSH 8.4p1 Ubuntu 5ubuntu1.1 |
| 80/tcp | open  | http    | Apache httpd 2.4.46             |

​	We find only two ports, SSH on 22 and http on 80.  Obviously this is a web server, and must enumerate on the machine through other means.  Navigating to http:\\\192.168.1.210 in the browser presents us with a login screen:

![login]() 

​	The next step is to find out if there are any interesting directories on this web app.  We will start a  [DirBuster](https://www.kali.org/tools/dirbuster/) loaded with the directory-list-medium that comes in kali and scan. The results of this scan can be found [here](https://git.rjphillips.online/main/networkattacksproject/-/blob/main/target2/scans/DirBusterReport-192.168.1.210-80.txt).  Dirbuster does this by tracking responses from the web server.  A "Not Found" reply means nothing exists there, while "Permission Denied" will indicate that the directory exists.



##### B) Attacking the login page

​	The first step in attacking this machine is to get past the login page.  We must login to enumerate further.  For now, we can only infer that pages exist based on the return code of the dirbuster requests.  The first step is to have a look at the source code of [login.php](https://git.rjphillips.online/main/networkattacksproject/-/blob/main/target2/loginpage/source.txt).

​	Immediately we see the fields for username and password, and we also see something that appears to be an anti-CSRF token:

```html
<input type='hidden' name='user_token' value='b57304890574bb9053c0c4f48d307773' />
```

​	These types of tokens are meant to prevent Cross Site Request Forgeries by ensuring that the client that is requesting or posting http content to the site is the same one that opened the initial connection.  The web app will generate a random token and present it to the client on the first load of login.php.  

Using Burp Suite, The initial connection will also return a response from the server setting up a cookie:

![cookie](https://git.rjphillips.online/main/networkattacksproject/-/raw/main/target2/screens/setcookie.PNG)

​	My initial instinct is that the basic security feature at play here is a server-side check to compare the PHPSESSID to the user_token.
