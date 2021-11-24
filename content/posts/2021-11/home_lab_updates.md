+++
title = "Home lab updates v2"
tags = [
    "homelab",
    "linux"
]
date = "2021-11-23"
toc = true
+++

Looks like the last time I posted any updates about my home lab setup was back in April of 2020, so I guess we're do for an update!

# What has changed?

- [Server rebuild](../../2021-05/server_rebuild/)
- New (2nd hand) 24 port gigabit switch
- 900W UPS
- Synology DS918 NAS

# So what am I doing with all these toys?

The server rebuild is probably one of the bigger things here. While I was already hosting VMs and Docker containers on this box when it was running Windows, it was unreliable at best thanks to forced updates/reboots in Windows 10. Sigh. I rebuilt this server on Ubuntu 20.04 LTS and haven't regretted it for a second. Resource utilization is super low, so I can run more stuff on it (also playing around with Jellyfin now), and since it's Linux the Docker support is significantly better (obviously) so I've been able to completely do away with any VM-based workloads.

Next up in technical significance is the DS918 NAS. What a game changer coming from my old NetGear NAS! Bonded 1Gbit NICs for super fast transfers, Docker support (so I've got my failover PiHole server hosted here now), and built-in applications for cloud sync so I can also automate all of my offsite backups of critical data. Worth every cent, and despite my best efforts I still haven't even broken 50% utilization of the disk space I put into it!

The UPS and network switch are purely utilitarian. The UPS is connected to my server via USB so I can script a shutdown sequence if it's anything more than a momentary outage, but beyond that this will keep my whole network up and running for about 90 minutes in the event of a powerloss. Great in case I'm in the middle of a meeting! The switch is just an upgrade over my full 8-port switch that was in use previously... may move to a PoE switch soon though... been eyeing another new AP!


