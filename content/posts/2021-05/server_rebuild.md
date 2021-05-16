+++
title = "Server Rebuild"
tags = [
    "homelab"
]
date = "2021-05-16"
toc = true
+++

# What did I do?

Half fueled by beer, and half by a playlist of keygen tracks, I decided that it would be a good idea to finally getting around to rebuilding my long-lived server... No more Windows and its infuriating forced updates (which you used to be able to override), lack of competent OS level event scripting for integration with my UPS, or garbage-tier support from Docker. I wanted something that I'd finally have the level of control I wanted for something that backs pretty much eveverything I use outside of work.

# Why did I do it?

There were a few things, eluded to above, that really drove this:

- Alcohol
- "I've done this a million times, it'll only take 2 hours tops"
- A need for better scripting around UPS status
- A need to avoid repeat downtime due to forced OS updates/reboots
- Really inefficient memory usage on the server due to poor Docker support, pushing me to VMs for several usecases 
- With Docker on Windows, there was (is?) no way to expose network shares to containers, so I was forced even more over to VMs
- I was sick and tired of needing to use an RDP client to manage my server when I could just be scripting things instead

# Prep work

Arguably the largest amount of work here was ensuring that I had all of my configs and persistent data backed up so that it would survive the migration. My old setup was using Hyper-V and Docker backed on a software RAID array - more on the RAID setup later, as it was annoying - so I wanted to get all of that data somewhere that I could either mount it and access it, or easily rebuild it. For the most part, this was just wrangling config files out of VMs and moving them over to my NAS or [repo](https://github.com/maclarel/homelab) so that I could pull them later. 

Next up was planning out the build, partition schemes and all. My server had three main volumes originally:

- 256GB SSD
- 1TB RAID-1
- 500GB HDD

I didn't want to waste time ever dual booting as there was simply no reason to for a machine that lives in my closet, so I split up the SSD as follows (leaving some extra space if I need to grow any partition later):

```
/ 30g
/home 50g
/data 130g
/swap 4g
```

I'm not sold on the `swap` ever being useful, but it's there. I had many occasions on Windows where I would run out of memory due to idiotic memory leaks with WSL/Docker, so I figured better safe than sorry. Based on what I'm seeing so far... this really wasn't needed.

The `/data` partition is there simply if I need it for apps requiring high disk performance, but so far Plex is quite happy running off of the RAID array and pulling data from my NAS. This may wind up just sitting idle. Similarly, not much use for `/home`. This split was really just so if I ever wanted to reinstall the OS I could maintain those partitions without losing data.

Everything else in terms of data access was just understanding how to mount my shares, and that was pulled from my existing VMs.

With my data set aside, I was ready to move forward...

# The reinstall

Well, installing Ubuntu Server is as braindead as it ever has been. Zero drama there.

Once that was online, I tried to mount my NTFS drives only to find two surprises:

1. I could mount my Windows RAID array and read the data, yay!
1. I could only mount the NTFS partitions in read-only because the filesystems had apparently been corrupted by an earlier reboot during startup... Joy.

Fortunately, I had backups of any data I needed, so no real issue there, but it did mean that I needed to rebuild these. It turns out creating a software RAID on Linux is pretty braindead as well:

```
parted -a optimal /dev/sda mklabel gpt
parted -a optimal /dev/sda mkpart primary btrfs 0% 100%
parted -a optimal /dev/sda set 1 raid on
parted -a optimal /dev/sda print

parted -a optimal /dev/sdb mklabel gpt
parted -a optimal /dev/sdb mkpart primary btrfs 0% 100%
parted -a optimal /dev/sdb set 1 raid on
parted -a optimal /dev/sdb print

mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sd[ab]1
mdadm --detail /dev/md0
mdadm --examine /dev/sd[ab]1

mkfs.btrfs /dev/md0

mount /dev/md0 /mnt/tesla
```

From there, it was just a matter of adding it to `/etc/fstab` and I was off to the races. Same for my network shares. Easy stuff.

Moving over `nginx` and `pihole` was trivial, and in fact somewhat improved, as they were both Docker based anyway.

Now the fun part - moving my "Linux ISO gathering" setup (torrent box + vpn) and Plex over to Docker. I decided to start with Plex - pretty much zero drama there, plug & play with `plexinc/pms-docker` though I did have to rebuild my library and unregister the old (now duplicate) server.

qBittorrent though, that was a challenge. Let's talk about that.

# Challenges

Originally, I was running qBittorrent in a VM so that I could send all of my network traffic out through my VPN provider without needing to impact the host's networking, but this meant maintaining a VM just for this purpose. I originally went this way after the rebuild on Ubuntu Server but I'm going to be frank in saying that qemu/kvm are fucking miserable to manage from command line, and I had no interest in setting up an X env to use virtman or anything else... I was just going to figure it out with Docker.

Through this, I found `https://github.com/tprasadtp/protonvpn-docker` which is great - proxied networking through a container running the VPN, and container level reliance on things running. This still had some problems that I had to address through.

1. The relative container reliance offered through docker-compose just means the other container has to be up, not that the VPN needed to be operational.
1. The VPN "kill switch" functionality is unreliable at best in the container world.
1. With the VPN connected, I'd [lose all LAN connectivity to the server I was trying to expose](https://github.com/tprasadtp/protonvpn-docker/issues/15#issuecomment-841824116).

I was able to sort out the LAN issue in the Issue linked above, fortunately, but definitely burned a few hours there playing around with different ways to pass in custom `ip route` commands. Fortunately, the VPN client already handles this for me and the author of that container already exposed the functionality. It took some hunting to figure out, but it's now working. Awesome, now let's figure out a kill switch!

If you're unfamiliar with the VPN "kill switch", it's basically a way to ensure that no network traffic hits the internet from the machine in question if the VPN connection goes down. The VPN I'm using does this through managing IPTables rules, however this allows a brief gap where traffic can still slip out when used in a container setting. With rapid reconnects this isn't a huge problem, but better to just not have that surface anyway. I was eventually able to deal with this by forcing the client (`qBittorrent`) to bind to the VPN's network interface in the VPN container. This interface is ephemeral, but consistently named - if the VPN goes down, the client drops ALL connections as it no longer has a network interface at all, and when it comes back up it detects it and rebinds. This isn't as glorious as using some dedicated routing for it, but it works and is (at this point) more reliable than the kill switch.

# Where did it all land?

This brings me to where I am now. The server is up, I no longer have a single VM running (everything is in Docker), and I can manage everything from an SSH client on my phone. It's pretty much ideal. Functionality aside though... this actually saves me money.

When I was running these workloads on Windows, due to the VM reliance, I was consistently using 10-11GB of my 16GB RAM. With this setup on Linux, I'm now using *less than 1.5GB*, or an 85% reduction in resource usage. I've gone from trying to source another 16GB of RAM, to wondering what I'm going to do with all the extra resources I now have at my disposal.

More to come, I'm sure, as well as likely no end to problems, but this has been a fun ~2~ 10 hour project, and I am damn happy with the results at this point!
