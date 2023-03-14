+++
title = "March Madness"
tags = [
    "homelab",
    "career",
    "linux"
]
date = "2023-03-14"
toc = true
+++

Happy pi day!

# It has been a minute 

As someone working in the tech industry right now, let's just say I haven't been in the best frame of mind to blog about stuff, but I've got a few ideas for some content here, so maybe it'll all flow out at once :shrug: On an immediately bright note, I haven't been laid off (yet), so things continue to look up!

With that said, this article is going to be upbeat! I've been tinkering in downtime, and have done some stuff around the house, and with my lab, that I think is pretty neat though it's by no means groundbreaking stuff.

# Home network 

So nothing too crazy here, but since I moved into my current house about 7 years ago I've been consistently annoyed that I have no way to run ethernet or fibre cables up from my basement to the second floor of my house without pulling out and redoing drywall... and that's not something I want to be doing.

I wound up settling with just having WiFi on my second floor, where my spouse and I have our offices, and that "worked", but it was constantly annoying that I'd get latency spikes during meetings and would only be effectively accessing (less than) half of the bandwidth that I was actually paying for.

**MoCA has joined the chat**

I wound up purchasing some [MoCA adapters](https://www.amazon.ca/dp/B09RB1QYR9) and they have been rock solid so far. My whole network is currently only GbE, and with the 2.5Gbit adapters I'm consistently getting the full speeds my network infrastructure is capable of. A very nice change, and I can definitely recommend these specific adapters.

In a funny twist of fate, my ISP upgraded me from 1Gb/750Mb fibre to 1.5Gb/1Gb as part of troubleshooting, so I'm actually getting more bandwidth to my house than my network can support. What a time to be alive!

# The lab 

The home lab continues to evolve, slowly but surely. I'm still running the same basic hardware, however I've doubled the RAM on my server (maxing out its old platform at 32GB) and have a CPU upgrade in the mail taking me from a paltry 4 cores/threads to 4 cores/8 threads. Truly, I am experiencing the pinnacle of computing technology :P

While it's old and slow by modern standards, it's more than sufficient for the tinkering I'm doing. My "prod" services (DNS/Jellyfin/etc...) continue to work just fine on there, and the extra RAM has enabled me to resurrect a small herd of VMs to use both for IaaC/automation stuff as well as starting to explore k8s. Yep, I'm late to the party, but it's not stuff that I tend to deal with frequently at work, so this is all just for fun, and fun it has been so far!

# The main rig

Back in November I made the jump to Linux (Arch, btw) on my personal rig full time. When I made that change, the intent was to dual boot with Windows 10, however I have since blown away Windows entirely so this box is full Linux!

I did, to the shock of some fellow Linux users, pick up a new nVidia GPU. While there have been issues there in the past, it seems to work well with the proprietary drivers. Based on benchmarks I've seen, for the most part performance is on par with Windows. DLSS3 support isn't quite there, but it's coming through Proton, and apart from the occasional crash in Cyberpunk 2077 (which I'm just as likely to attribute to CDPR as I am Wine) things have been _surprisingly_ solid.

Really enjoying this jump, and not missing Windows at all. With that noted, I do now need to run some Windows VMs for work, but at least it's out of my day to day usage.

# The security world

I touched on a visit to nSec in May, and it turns out I won't just be there... [I'll be speaking at it](https://nsec.io/session/2023-behind-the-scenes-in-github-bug-bounty.html)! Come listen to me ramble for 30 minutes and then say hi :D

I've also, finally, been able to gather a small team for HackTheBox's events, and we'll be taking part in the CyberApocalypse event this coming weekend. No expectation of doing overly well in terms of placings, but as always it'll lead to some more learning and team-building, so I'm excited about that.

Anyway, that's it for now. Back shortly with an article about remote work!
