+++
title = "State of the Lab (late 2023)"
tags = [
    "homelab",
    "linux"
]
date = "2023-12-23"
toc = true
+++

# Didn't you just blog that this was going to be a 2024 project?

![](https://c.tenor.com/lZkfdEToSHEAAAAC/tenor.gif)

Or, more specifically, I got a bit too excited after writing the last post and some hardware I was eyeing up went on sale. Since this update is happening _today_ (hopefully) I figured this might be a good time to finally do a full walkthrough of what hardware and services I'm running.

# The hardware

## Old & busted

As noted, I'm finally replacing my venerable home server. The current (former) "server" is an i7 4770k with 32GB of RAM, a 256GB SATA SSD, a 1TB SATA SSD, and a RAID-1 array of 1TB spinning rust. It is old, but it has worked remarkably well over the past few years. With that noted, as I venture more into the security research space and my desire to host more services at home grows, the core count and 32GB limit of the platform has started to become a limiting factor. This machine is still extremely capable, and will probably turn into an emulation box/HTPC in its next life, albeit with the RAID array removed.

## New hotness

The new "server" is a Beelink SER5 with a Ryzen 5800H, 64GB RAM, a 500GB NVMe drive, and I'll be porting over the 1TB SSD from the previous build. This build nearly doubles the per-thread performance compared to the 4770k, while doubling the core/thread count, and doubling the RAM. A larger SSD may be needed in the long run, but the current 1TB will only be about half utilized with my current needs.

## The complete hardware stack

- Beelink SER5 mini-PC
- Synology DS918 NAS
- CyberPower CP1500AVRLCD UPS
 
 Yep, that's it. I had considered getting several of the Beelink boxes to run Proxmox in an HA or cluster configuration, but I simply don't have the need nor do I want the additional headache and cost of running a separate storage plane.

 The UPS is nothing special. Even with the current power draw, this can power my entire stack for about 30 minutes. With the existing server shut down that extends to nearly an hour, and I expect it to land around the 45 minute mark after the Beelink box is up and running to replace the power-hog that is the current i7-based server.

 ## The network stack

This is where there's a bit more going on. Across my home, which is by no means large, I have an assortment of smart devices (Fire TVs, Chromecasts, etc), a few dozen IoT devices (cameras, outlets, etc), 5 actively used compupters, and a plethora of other devices. As I also work from home, network reliability is very important to me, as is the ability to segregate traffic on my network.

 ### Network hardware
 
- UniFi Dream Machine
- DLink 24-port unmanaged Gigabit switch
- DLink 5-port unmanaged Gigabit switch
- TP-Link 5-port unmanaged Gigabit PoE switch
- 2x UniFi U6 Pro APs
- 1x UniFi Nano-AC AP
- 1x UniFi AC Mesh AP
- goCoax MoCA 2.5 Adapter

The AC Mesh AP services my backyard since it is outdoor rated, and the other APs are spread through the garage (Nano AC) and floors (U6 Pros) of my house. Overall this means that I've got pretty much "full speed" WiFi coverage anywhere on my (quite small) property. I went with Ubiquiti UniFi gear for most of this due to the simplicity of setting it up and, in my experience, the relatively reliability of it without the management overhead of something like MikroTik.

The switches in the network are all "dumb". While I do plan to upgrade the main 24-port switch that acts as backhaul for everything over to a UniFi PoE switch this year, I have limited need for network segregation on the physical level. I maintain  "Prod", IOT, and guest networks on my LAN, however IOT and Guest networks are WiFi only at this time. While I'd love to have some more wired opportunities, my house lacks cabling to support it, and this is partially where the MoCA adapter has come in.

Since I don't have in-wall ethernet wiring, and realistically I don't see the value in tearing open my walls to add it, I've opted to handle my network backhaul between floors over coax. I was a bit reluctant to do this initially, hearing concerns about the added latency, single duplex nature, etc... however in practice this hasn't been an issue. Yes, there's an added 2-3ms of latency, but I'm unable to saturate the link this gives me and I don't foresee a usecase where I need > 2.5GbE between floors for quite a while. This will come eventually, but isn't a priority right now.

Near-term I'll _likely_ be picking up a UniFi 16 port Gigabit switch, an unmanaged 8 port PoE switch, and likely another 5 port unmanaged DLink switch. The TP-Link switch I bought is unfortunately faulty, with only 3 working ports, so I'm hoping to replace that and remove my need for PoE injectors entirely. These will probably be spring/summer upgrades, and something I need to be cost-conscious on.

# The software

Obviously the network software goes without saying, so I'm going to focus on detailing the services I'm running on my Server & NAS, and why I've chosen them. 

To start off with, I run virtually everything in Docker containers. I wrote about this [back in 2021](https://www.maclaren.dev/posts/2021-11/home_lab_updates/#so-what-am-i-doing-with-all-these-toys) but the reduced management requirement & significantly reduced resource utilization of running services in containers over individual VMs has been a god-send in terms of making the most of my hardware. With this noted, I do run a number of VMs, and in fact these were a driving force behind the in-progress server replacement.

## VMs

As noted, I do run several VMs, and one notable change with the rebuild is that I'll be virtualizing my current server in its entirety since I'll be running Proxmox on the new host:

- Containers (Ubuntu Server)
- Ansible (Ubuntu Server)
- 3 Linux sandbox VMs (Ubuntu Server)
- Kali (Debian-based)
- Windows 10
- Windows 11

Breaking these down a little further, let's talk about the uses cases.

### Ubuntu Server

While I'm not the biggest fan of Cannonical's decisions or ways they do business, there's something to be said for having a homogenous setup internally, and in my expericen Ubuntu comes with fewer (or perhaps, different) headaches compared to running straight Debian. This also simplifies my update processes since all of my managed Linux endpoints are using `apt` for package management.

### Container VM

This is effectively virtualizing my existing server. Since Proxmox will handle the hypervisor layer for me, I don't really want to also run my containers on bare metal if I don't need to. This will be home to many services running in Docker (eventually Podman, maybe) and ideally little to nothing else.

### Ansible VM

I use Ansible to enforce various configurations across my servers as well as to manage updates, TLS certificate renewal, and some backups. Right now this is managing all of my VMs, but may be expanded out to my physical nodes in the future.

Reasonably typical usecase here, and I like having this on its own VM for logical segregation, though I wish Ansible didn't require unfettered root access (via `become` to operate). I maintain all of my playbooks [on GitHub](https://github.com/maclarel/ansible_playbooks) for ease of version control and development.

### Sandbox VMs

These don't have dedicated use right now, and are mostly for tinkering with my Ansible playbooks. Now that I have more robust hardware, these may end up as a layer to experiment with K3s, however it's more likely that these will host whatever services I am poking at for vulnerability research since I don't necessarily want those running on "prod" hosts.

### Kali VM

I work in the security space and Kali is a great tool. While the packages it comes with can be installed pretty much anywhere, and I have most on my other machines, I've found having the VM to be exceptionally useful. I do not leave my main PC powered on, and my laptop is a work-managed endpoint, so having a machine dedicated to tooling that would otherwise set off work-related alarm bells is extremely helpful when I'm on the road or otherwise don't want to physically leave my work setup.

### Windows VMs

I need these for work as well as vulnerability testing. In short, I work with a lot of client-side applications, so these are important testbeds for me despite not using Windows personally or professionally in any other capacity.

## Containers

### [ddclient](https://docs.linuxserver.io/images/docker-ddclient/)

`ddclient` handles Dynamic DNS for me, updating a record under my domain that allows me to remote back in to my home network via Wireguard without the need (or ability) to have a static IP from my ISP.

### [Wyze Bridge](https://github.com/mrlt8/docker-wyze-bridge)

I use Wyze "security" cameras around my property. Like the rest of the IoT devices, these live on their own network, and similarly are only outdoor-facing to minimize exposure for when Wyze repeats the same mistakes that seem to plague all camera vendors and share my cameras to someone else. 

The Wyze Bridge service integrates with the Wyze API and transforms the camera streams into RTSP. While Wyze does offer some form of RTSP firmware, it's not supported, and it has several caveats. This keeps my exiting functionality fully intact, while also giving me easy local access to my camera feeds in a browser or when integrated through another service.

I'd like to pull these feeds into Frigate at some point, however I've had some massive performance problems with Frigate in the past, and also having [some significant concerns with its security model](https://github.blog/2023-12-13-securing-our-home-labs-frigate-code-review/).

### [Homepage](https://github.com/gethomepage/homepage)

Homepage does what it says on the tin. I use this as a landing page for everything I run internally. A one-stop shop for all the links I need, weather, and search. The default location for all of my new browser tabs.

### [Jellyfin](https://jellyfin.org/)

An open source Plex replacement, Jellyfin allows me to stream my thousands of hours of "Linux ISOs" to my client devices. Great for audio and video, no complaints!

### [Joplin](https://github.com/laurent22/joplin)

Joplin is a note taking app that supports Markdown and gives me the ability to store all of my data locally on my network, encrypted at rest and in transit, and still support syncing with all of my mobile devices as long as they have network connectivity to the server (via VPN or otherwise). While it's not as powerful as something like Notion, the ability to encrypt and store all of my notes locally is very important to me since this is also used for work purposes.

### [Mealie](https://github.com/mealie-recipes/mealie)

Mealie is a new addition to my list of services, but in short its a cookbook where I can either scrape websites for recipe information or I can enter it myself. Very useful as I experiment with different techniques when cooking barbeque!

### [Nginx](https://nginx.org/en/)

This doesn't need any introduction to the audience reading this blog, but in short I use Nginx as both a web server and reverse proxy to give me a TLS front-end for all of my containers.

To that end, virtually all of my services do not offer (or at least not trivially) out-of-the-box support for TLS, so proxying access is quite important. To accomplish this, most of my containers have their network ports only exposed to `127.0.0.1` and I then "publicly" (to my network) expose those as TLS-enabled endpoints through Nginx.

I'm aware that Nginx Proxy Manager exists, however I also use Nginx professionally so I see great value in digging into all of this manually and in some cases [this greatly simplifies troubleshooting](https://github.com/mrlt8/docker-wyze-bridge/issues/989#issuecomment-1763431176).

### [PiHole](https://pi-hole.net/)

This is my ad-blocker of choice for my network, and also handles local DNS records (managed through Ansible as well). I also run this in a container on my Synology NAS so that I've got failover in the event that either host goes down.

### [Portainer](https://www.portainer.io/)

This one is an experiment. On paper, Portainer gives me a level of management for my Docker Compose stacks, however I've found that with the free version there are many limitations on things that are otherwise trivial for me to do via SSH or through Ansible (e.g. updating all stacks to pull the latest images). Not sure if this will stick around, but we will see!

### QBittorrent & Wireguard

This is a bespoke setup where I'm running [QBitTorrent](https://github.com/linuxserver/docker-qbittorrent) with its networking routed through a Wireguard container connecting to my VPN of choice. No one needs to know that I secretly download SUSE Linux ISOs...

### [SearXNG](https://github.com/searxng/searxng)

SearX (or SearXNG in this case) is an anonymized search aggregator. In short, this gives me results from my search engines of choice, but performs those searches in a form that can't be tied back to an individual (beyond an IP address) and as such helps get rid of a significant amount of tracking, ad personalization, etc. 

The quality of results, unfortunately, does suffer from this but I find the privacy tradeoffs to be conceptually worthwhile.

### [Wireguard](https://github.com/linuxserver/docker-wireguard)

And last, but absolutely not least, I run Wireguard in a container. This is mostly [to deal with some bizarre limitations with how Ubiquiti implements their own VPN solutions](https://www.maclaren.dev/posts/2022-09/bell_udm/#wireguard), but it's aslo nice knowing that I've got more complete control over how it all works. 

Unlike UniFi software, I also don't mind publicly exposing Wireguard as an endpoint.

# What's next?

Whew, that's a lot of writing, but that's the current state of my lab & network. At the end of the day, PiHole is the only thing here that really does a "mission critical" job for me, with Wireguard coming in as a close second. The others exist for convenience, and frankly have also made for really fun targets to do security research with and I've been in touch with some maintainers to get bugs addressed... Let the CVEs flow!

Next up will be the network upgrades I had referenced, hopefully bringing me to my "endgame" state as it exists for the next few years until 10GbE reaches a more affordable point. Some other thoughts, permitted by the additional hardware, would be to set up an Active Directory host and manage the Windows VMs there to facilitate some learning in that space since it's clearly extremely hot in the security world. 

Lastly, a workflow shift on my side is that I use my existing bare-metal server for a fair bit of experimentation work when I need to run Linux (or x86 in general) workloads and I'm stuck working on my Apple Silicon Mac. While I've got the sandbox VMs, these see a lot of churn and aren't something that I want to exist outside of a "lab" capacity - same deal with the Kali VM. I may wind up creating a "development" VM where I've got all of the dev tools that I need in a one-stop-shop that I can hit on my local network or over VPN.
