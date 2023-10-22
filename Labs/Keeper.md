# Keeper CTF (HTB)
https://app.hackthebox.com/machines/Keeper

## Port scan

### Tool used: nmap
<br>

```
nmap -p22,80 -sV keeper.htb
Starting Nmap 7.94 ( https://nmap.org ) at 2023-08-20 01:43 CDT
Nmap scan report for keeper.htb (10.10.11.227)
Host is up (0.046s latency).
rDNS record for 10.10.11.227: keeper

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Web

Browsing the URL points us in the direction of another link: **tickets.keeper.htb/rt**<br><br>

![web_01](./docs/keeper/keeper_01_web.png)
<br><br>

The login page indicates that they are using *Request Tracker*. A quick search gives us the default login credentials - [as expected] *root / password*.

![web_02](./docs/keeper/keeper_02_web.png)
<br><br>

Sure enough, the default credentials work and we are logged in.

![web_03](./docs/keeper/keeper_03_web.png)
<br><br>

## Enumeration

Browsing around the site, we find a "Users" section. 

![users](./docs/keeper/keeper_04_users.png)
<br><br>

When we view the properties for Lnorgaard, we see a comment about the user's initial password: *Welcome2023!*

![user_info](./docs/keeper/keeper_05_userinfo.png)
<br><br>

## SSH

Using the provided credentials - *lnorgaard / Welcome2023!* - we find the user.txt flag.

## user.txt

```
762175**************************
```

![ssh](./docs/keeper/keeper_06_ssh.png)
<br><br>

Back on the website, we see another ticket referencing a *Keepass* crash dump file, as well as the indication that they are using an older [vulnerable] version.

![dumpticket](./docs/keeper/keeper_07_ticket.png)
<br><br>

Back to our SSH session, we see a file called *RT30000.zip*. The zip file contains two files: *KeePassDumpFull.dmp* and *passcodes.kdbx*. After some searching, we come across an exploit PoC for *CVE-2023-32784* (https://github.com/vdohney/keepass-password-dumper).

For this to work, we need a Windows machine due to the tool's .NET requirement. Upon running the tool, we get the following output:

![pwdump](./docs/keeper/keeper_08_exploit.png)
<br><br>

We see part of the password, but at first glance the characters seem out of place. If we backtrack a little, back to *Request Tracker*, we see in the user's profile that they are Danish. So, we just try copying the partial password and search for it on your favorite search engine. 

The results indicate that the password is part of a Danish desert called "rødgrød med fløde". So, this must be the password for the Keepass database file, right? Our first snag was simply trying to type it out by hand - "rodgrod med flode". This did not work, so we tried simply copying the name straight from the search result: "rødgrød med fløde" (notice the 'o' character is different?), which unlocks the database file.

![keepass](./docs/keeper/keeper_09_keepass.png)
<br><br>

Attempting to SSH with 'root' and the provided password failed. Looking at the notes, we see that the key revealed is a PuTTY key. 

![creds](./docs/keeper/keeper_10_creds.png)
<br><br>

After using the PuTTY Key Generator and the provided password to convert the key to the OpenSSH format, we can now login as root!

![putty](./docs/keeper/keeper_11_putty.png)
<br><br>

```
chmod 0600 keeper.key

ssh root@keeper.htb -i keeper.key  
Enter passphrase for key 'keeper.key':  
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-78-generic x86_64) 
 
 * Documentation:  https://help.ubuntu.com 
 * Management:     https://landscape.canonical.com 
 * Support:        https://ubuntu.com/advantage 
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings 
 
You have new mail. 
Last login: Tue Aug  8 19:00:06 2023 from 10.10.14.41 
root@keeper:~#
```

After logging in, we can get the final root.txt flag.

## root.txt

```
71b2fb**************************
```