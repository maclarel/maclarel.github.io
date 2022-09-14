+++
title = "Bell HomeHub 4000 + Ubiquiti Unifi Dream Machine VPN"
tags = [
    "unifi",
    "tech"
]
date = "2022-09-10"
toc = true
+++

This article is a bit more to-the-point than most others I've written, and is mostly to serve as a reference point for others trying to configure a VPN server with Unifi products behind the Bell HomeHub 4000.

# The HomeHub 4000 DMZ works... kinda

The HomeHub 4000 doesn't have `Bridge mode`, period. This leaves users with three options:

1) Find a way to use PPPoE directly (possible, but janky)
2) Use the "Advanced DMZ" feature on the HomeHub 4000 (Spoilers: It doesn't work _at all_)
3) Use the normal DMZ feature on the HomeHub 4000 and deal with double-NAT

After several days of troubleshooting I opted for option 3 from the list above. 

- The UDM doesn't have an SFP port, and I wasn't going to shell out for a media converter, so that removed option 1. 
- Option 2 resulted in my UDM getting the Bell public IP as expected, but under no circumstances could it send traffic. I've seen some other blog/forum posts noting that it can work if you add custom routes, but this feels so far off the supported/intended path that it's not worth the headache to troubleshoot if things go wrong.
- Option 3, at first, seemed to be totally broken, but wound up working fine...

# The UDM's VPN works... kinda 

When I was using bridge mode on my old connection, I happily made use of the built-in VPN solution that the UDM (and UDM Pro) line offer. No port forwarding required, no drama, and easy to handle networking for VPN clients, as well as individual credential sets.

Unfortuately for me, the Ubiquiti software _refuses to enable the VPN server if it detects that it's on a NAT'd connection_. Why it does this is completely beyond me, but it will flat out refuse to work _despite allowing you to enable it anyway_.

I spent _days_ troubleshooting this, thinking that the problem was the HomeHub 4000's "DMZ", but it was't. The Ubiquiti software just won't actually accept connections even if they originate from the same LAN (ruling out any firewall/NAT issues). 

Fortunately, the new `Teleport` feature does work, but that feels like more headache than it's worth _for my use case_ and I don't want to be an early adopter. It's also not possible, currently, to customize the subnet that Teleport VPN clients land on, and they have no access to the rest of the network(s), making it completely useless to me as the whole point is to access my LAN resources while AFK.

# Wireguard 

After wrestling with Teleport for a while, I realized I could just run my own Wireguard server. I have a finite number of devices I want to use, and having a config file/QR code to scan is much simpler than managing credentials, or needing specific one-time-use URLs, etc... I just want to get connected.

Docker and Wireguard to the rescue! Within 30 minute I was up and running - just needed to place my UDM in the "DMZ" of the HomeHub 4000's config, port forward to the configured UDP port for Wireguard's container on my server, and it all "just works"!

# Resources

If you're interested in playing around with Wireguard, the config is [this easy](https://github.com/maclarel/homelab/blob/master/wireguard/docker-compose.yml). I used parts from several tutorials beyond that, but [this one](https://codeopolis.com/posts/installing-wireguard-in-docker/) is probably the most succinct. 

Just run `docker logs` for your `wireguard` container and you'll get the QR codes for easy mobile device configuration, or hope to your configured `config` directory to pull out the `peer.conf` file(s) to use with your other systems. 

These are both then trivial to use with the official Wireguard apps available for non-Linux platforms, or of course with Wireguard proper if you're connecting from a Linux box.

# Back to normal

After a few frustrating weeks, I'm now actually working with a _better_ VPN config than I had before. No hassles with it, instantaneous connection, and great bandwidth speed compared to OpenVPN.

Hopefully Bell pushes a firmware update to the HomeHub 4000 with bridge mode some day, but I'm not holding my breath.
