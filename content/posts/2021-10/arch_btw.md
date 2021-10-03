+++
title = "I use Arch, btw"
tags = [
    "linux"
]
date = "2021-10-03"
toc = true
+++

# I use Arch, btw

I've been using Linux for ages, nearly two decades at this point. I actually dug out my old Mandrake and Slackware CDs while going through a box in my basement the other day... quite the flashback.

Despite that tenure on at least some of the hardware I've owned at any given time, I've never really used it as a daily driver for _desktop_ usage. That changed today when I finally sat down and set up Arch, complete with forgetting to install a boot loader, on my main rig.

One catch though... I did it in Virtual Box.

# But it's in a VM 

Yeah, yeah, yeah. I know. I'd prefer to have it on bare metal as well, however my main passtime outside of tinkering with technology is playing WoW Classic. At first you may think "Ok, that definitely runs under Wine/Proton, right?" and yes, it does, but Blizzard also has a long history of banning Linux gamers for perceived cheating, "using an unsupported operating system", or any other number of non-sensical reasons.

In any case, that has kept me using Windows as the main OS installed on bare metal. Flip side, Virtual Box has come a long way and is exceptionally good for virtualizing a desktop OS - especially when compared to Hyper-V which I was using previously.

All that to say, I've been working out of i3 on Arch all day despite it running in a VM and the only thing it has made me want to do is quit WoW so I can just use this without any additional overhead. It's wonderful. 

# How did the setup go?

The Arch Wiki's installation guide is reasonably good, though like all things Arch it expects you to do a fair bit of research for yourself. If you haven't toyed around with Arch before, or at least don't have a very good understanding of how to do things like manually configure a network interface from scratch, I'd suggest against doing this for your first time unless you're into learning-by-doing (like I am).

I've dabbled with Gentoo in the past, and this is nowhere near that level of confusing to setup, but it's still not a pretty GUI like you'll get with Ubuntu or Pop, etc... That said, that didn't really factor into my decision.

I had a few hiccups during installation, mostly due to not quite understanding what I was getting "for free" with the installation vs what was running on the live disk I used for the installation. This meant that I had to spend a good 30-45 minutes figuring out all of the packages I needed to get my networking functional (I ended up using `networkd` for the sake of having it in line with my other boxen). Beyond that, though, was an infuriating 90 minute stretch of trying to get i3 to work...

i3 relies on `i3-config-wizard` to do the initial configuration file creation to actually be able to start i3, however _that wizard requires X to be running despite still being a text interface_ which is beyond bizarre. It wasn't until I got xmonad running (and realizing that it wasn't for me) that I was able to actually get i3 configured and working. Haven't looked back since. After coming from a lot of tinkering with Amethyst, DisplayFusion, Yabai, and other "window managers" on Windows and MacOS, having something like i3 and it's integration with Xorg is... well, mindblowing. Full control over everything. It's perfect.

Without getting into the religious debates of what WM is the best, least bloated, etc... This is a fantastic example of the type of freedom you can get when you move away from Windows/MacOS and into an ecosystem that's literally built around user choice and flexibility. I've never been one to enjoy playing within the rules, and now I truly don't have to.

# How's it going? 

I'll do another post in a few weeks to recap using Arch + i3 as a daily driver for that period of time, but initial impressions are exclusively positive.

My experience of using Linux as a desktop OS in the past were mostly sullied by my own choices (Slackware as a daily driver? Eep!) or lack of knowledge at the time... but hey, that's how you learn. With the readily available knowledge between Wikis, colleages, friends, and Reddit it's now somewhat trivial to find answers to questions that would have taken weeks to sort out on IRC or forums (though I still enjoy both).

As an aside, the workspace approach here (and more importantly being able to trivially hotkey windows over to specific workspaces) is a game changer after coming from Windows/MacOS. I know that can all be done through various ways there as well, and I use those approaches, but it's *seamless* in i3 (and I'm sure many other WMs). I love it.

Coming over from MacOS for work purposes, and Windows for personal projects, this is also a breath of fresh air. I've got control over what's running, resource utilization is way down, and things are just much... snappier. Despite running this in a VM (and one that only has access to 1/3 my system resources) it is *vastly* more responsive than Windows 10 is on my beast or a PC, or MacOS on my max-spec 2019 MBP. It's really quite mind-boggling what I was living with, and how much better it could have been... despite running exactly the same applications!

The other thing that's of course beneficial here is that nearly all of my projects are based around stuff that just better works on Linux anyway. I don't need to deal with MacOS shipping with GNU Utils that are decades out of date, or the bizarre setup that is WSL (though WSL is still pretty cool!) - I've got the latest and greatest at my fingertips, and nothing to get in the way of my work... only the things that I've chosen to pull in to _support_ my work. Is this... freedom?

# What's next? 

Current things for me to tweak/understand:

- Can I get any GPU passthrough configured for this, and do I even need/want to? Performance is great for any non-gaming, and I can just pop out to Windows for that.
- Play around with some more i3 config to get tiling layout just how I want it, and align keybinds with what I'm using on my other systems to limit context switching.
- Play around with i3 gaps for a sexier layout, though I am loving the completely efficient use of space that i3 is giving me so far.
