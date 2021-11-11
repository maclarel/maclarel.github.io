+++
title = "I still use Arch, btw"
tags = [
    "linux"
]
date = "2021-11-11"
toc = true
+++

Yep, the meme lives on. Still getting by with Arch as a daily driver. Still running it through VirtualBox.

# What's great?

Full control, and amazingly lightweight. Booted, and with my WM (i3) running, I'm measuring < 300MB of RAM usage and virtually zero CPU/disk usage. Windows or Mac OS sitting idle... yeah, not comparable. Itg's very refreshing having full control over just about every aspect of what I want. 

I'm getting so used to using i3, with only extremly minor modifications to bindings, that I've actually updated a lot of my Amethyst bindings to align with i3 defaults. It's just plain intuitive. Similarly, having a WM that's heavily keyboard driven, combined with qutebrowser and/or the Vimium extension in other browsers, makes it so that my hands rarely need to leave my keyboard. Obviously there are still some reasonably keyboard-unfriendly scenarios (*glances at Google sites*), but beyond that it's a pretty fantastic experience.

I will note that I really haven't tried gaming in this at all. I haven't poked at GPU passthrough, and there's really no point in even trying with the vGPU that VirtualBox provides. I _did_ get gzdoom running, but it was like a slideshow. Regardless, I've got Wintendo for that.

# What's just OK?

Now I'm going to preface this by noting that I know I'm not doing myself any favours by 1) using Arch, and 2) not using a major DE like Plasma.

The only thing that is really, really painful coming from Windows/MacOS is the lack of a reliable clipboard system without having to get something setup. Yes, I know that I'm getting *exactly* what I asked for, but I'm still going to complain as it's really my only complaint <_<

This is of course something I'll get around to addressing, but I had to bitch :P

# What sucks?

VirtualBox has some annoying bugs, and "features", but not in ways that actually break things. Some minor annoyuances include:

- Sometimes, but only sometimes, audio in my VM will get heavily distorted or "poppy". The only workaround is to relaunch VirtualBox. Even restoring the VM from a snapshot doesn't reproduce it, so this feels like something with the hypervisor rather than the Guest.
- Despite being in full-screen mode, with no way to escape the guest short of ctrl-alt-f, there are still other key combinations that will escape the guest... Most notably among these chorgs is Super-L which of course is a default in my WM, but every f&*king time I hit it to move a window around it locks my machine (via Windows). Need to figure that out still...

# So what's next?

Well, in my current work position I can't actually use Linux as my full time daily driver _however_ I have already moved all of my personal work over to it and LOVE it. Significantly less friction than trying to work on projects through WSL, fewer annoyances than using MacOS and its menagerie of ancient/outdated packages and lack of customization, and my fingers would fall off trying to outline all the reasons that I prefer it over working directly in Windows.

I guess this is all to say that despite running Windows under the covers, I have been, and will still be, spending more time in Linux than what's actually powering it. I'm OK with that for now. It'll be interesting to see where Win 11 goes. I wouldn't mind taking the full jump over to Linux, but it will likely always be as a dual-boot as when I game I don't want to have to tinker.
