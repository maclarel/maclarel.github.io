+++
title = "Starting out with a home lab"
tags = [
    "linux",
    "homelab"
]
date = "2020-04-04"
toc = true
+++

# Why do I want a home lab?

While only you, the reader, can answer this, there are many reasons that I've build one out. I imagine your reasoning will be similar.

- There are things you want to have, or automate, that are simply obnoxious otherwise.
- You've got specific technologies that you want to explore without a full scale investment.
- You're working toward certifications (or hands-on experience) with technologies that you want to be using for work.

Personally, I just want to tinker. I'm very lucky to have access to all of the tooling I need for my day job through work. With that said, there's a lot that I want to play with outside of that scope as well as many "quality of life" improvements that I want for my home network such as:

- Ad-blocking
- In-home media storage & streaming
- On-site Backups
- Home & network security
- Tinkering and learning

# But I don't have a server!
 The biggest misconception that people have about setting up a home lab is that they need to run out and buy a used Dell R710 (or something similar), load it with hard drives, and install [Proxmox](https://www.proxmox.com/) or [ESXi](https://www.vmware.com/content/vmware/vmware-published-sites/us/products/esxi-and-esx.html.html) just to get started.

Let's talk about the reality. Do you have an old  laptop? Maybe an old gaming rig that you recently replaced? Cool! Ignore the rack-mounted server as you're 100% ready to get started.

While Proxmox and ESXi are both awesome and great platforms to build your home lab on, you don't need them to test the waters. Regardless of what platform you're on, you can build VMs with [VirtualBox](https://www.virtualbox.org/wiki/Downloads), or if you're on Windows (10 or Server) specifically you can use [HyperV](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v) ([which I did a tutorial on a while back](https://www.thedatanewbie.com/2018/04/tutorial-set-up-linux-vm-on-windows.html). Lastly, you've got the ability across all of these platforms to play with containerization with [Docker](https://www.docker.com/) or [Kubernetes](https://kubernetes.io/).

While we won't dive into the details of each of these technologies here, the point is simply that all of this is already available for you to work with and learn even if it's on a 10 year old Dell desktop you bought from a library!

If you *really* don't have what you need here, there are tons of trial options with services like Amazon AWS EC2 or the free tier of Google Cloud Platform, not to mention things like partner codes for discounts with companies like [Linode](https://www.linode.com/).

# What does my home lab look like?
Building off of the info above, I don't have an "impressive" home lab by any stretch, but it still does everything I need. If IaaS isn't your thing, and you don't have a computer to play around with, you can pertty easily build out the same setup I'm using for under $500CDN all-in.

## My "lab"

- CPU - i5 4670k
- RAM -  16GB DDR3
- Storage - 256GB SSD, 2TB HDD, 2x 1TB HDD (RAID-1)
- NAS Storage - NetGear ReadyNAS NV+ w/ 4x 1.5TB HDD (RAID-5)
- Router - ASUS RT-N66U
- Switch - Unmanaged 8-port Gbit & 24-port 100Mbit switches

Yep, this is all old and busted... and it runs Windows 10.

The CPU is 7 years old, and both the RAM and HDDs were bought used. The NAS is 10 years old and can't handle transfer speeds above 15MB/s despite having a 1Gbit NIC.

But you know what? This setup still does everything I need. You can buy a comparable PC + hard drives for around $350CDN, and I got that NAS with its drives for $100 and a bottle of wine.

# What do I use that junk listed above to do?
I'll start by noting that I'm a hobbyist in this space. I use this setup to tinker and to handle some specific usecases for my home, as well as general learning. I'd be pretty happy to recommend this setup for any low(er) level IT certification training, and it's absolutely sufficient as a development/testing environment across a variety of operating systems.

With that out of the way, here's what I'm using this setup for:
- 4x Ubuntu Server VMs running under Hyper-V
  - ELK - Collecting system metrics and DNS info from all VMs
  - Plex - Serving up media content (720p/1080p CPU transcoding)
  - PiHole - Blocking adds and obnoxious traffic from my network.
  - Ansible - Configuration management and automating updates across all of my VMs
- 2x RHEL VMs for certification training/tinkering
- OpenVPN server into my home network
- Torrenting "Linux ISOs"
- Securing outgoing network traffic over VPN
- NAS
  - Backups from all machines on my network (via OS tooling or rsync)
  - Media storage (music, video, software, etc.)

A lot of these use cases are evolving. For example, I set up ELK stack mostly to play around with it and to refresh my knowledge since I'd been away from it for a while, but it's not particularly useful for an environment my size outside of tinkering.

# How do I keep track of all these things?
Fortunately, this is pretty simple to answer. Ansible handles basically everything for me, pulling my configs from my (currently private) GitHub repositories and syncing them across all machines where they're needed.

I've got a goal of eventually making all of that public, but want to be sure it doesn't have too much information in it regarding my network/services/configurations.

# What's next for my home lab?
As you've probably guessed, the possibilities here are somewhat endless, but here's a high level overview of what I'm looking to do with over the next couple of months/years:


- Hardware
  - Update my router with a more powerful configuration (e.g. Unifi devices)
  - Replace my NAS with something more modern (e.g. Synology 4 or 5 bay NAS)
- Software/Use cases
  - RTSP server for home security cameras
  - Replace various VMs with Docker containers to both free up resources and to gain further experience working with Docker
  - Further automation with Ansible to cover as many software configurations as possible 

# Where can I learn more? 
- https://www.reddit.com/r/homelab/
- https://www.reddit.com/r/HomeNetworking/
