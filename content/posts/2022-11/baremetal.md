+++
title = "Bare metal"
tags = [
    "linux"
]
date = "2022-11-17"
toc = true
+++

# Vacations are dangerous

I decided to take about a week and a half off to decompress, and it has been a fair bit more productive than I expected it to be given that the plan was to do _nothing_...

I thought to myself:

> Hey Logan, you've got nothing going on for the next few days, and could afford to do some firefighting if needed... Why don't you do a bare metal Arch install on your main rig?

And I did...

# You already know I use Arch, I've told you 

I decided to run with Arch just because I was already familiar with it, and I love the rolling release approach that it uses instead of doing major upgrades like Ubuntu (though I use that on my server due to LTS). I also appreciate the overall approach of shipping with virtually nothing, and letting you (largely) do whatever you want. I find this all ironic given that Slackware was my first distro.

My worry with all of this is that I'd still need to frequently dualboot to play games, which is one of the main usecases for this machine despite the fact that I spent a lot of time working out of Linux VMs (Arch and Kali). As it turns out, I should have listened to the opinions of people that I had dismissed as Linux zealots... I'm _blown away_ by how well things are running in Linux under Proton and/or Wine. Lutris and Steam make it almost trivial to do, as well. Sure, there's some tinkering required, but that makes the reward of actually playing the game all that much sweeter.

**It feels like I'm using a computer again - not a toy** and I can't express how much I enjoy that. I haven't felt this way when using a computer since the early 2000s.

# But I'm a casual...

While this definitely isn't a fit for a super casual gamer, _however_ I am amazed by how seamless Valve has made things with Steam. I could definitely see using a gaming-focused distro being a good option, especially with the direction (bloat, ads, hardware lockouts, etc...) Windows is heading in, and has been heading in for over a decade now.

Maybe its just infatuation but I've spent more time in front of this computer in the past 3 days since doing this install than I have in the past few *months* outside of dedicated gaming time. Everything feels snappy, I can control everything just how I want it to, and the window manager (i3) does _my_ bidding. I actually have control over my system!

# An unexpected benefit

One of the things I had not considered before making the jump was just how much I appreciate having a really good window manager, and the overall design choices that many things have - and of course the focus on command line. I have terrible RSI in my right shoulder from years of awful posture and low-sensitivity FPS games, and with `i3` (or comparable window manager) and the aforementioned command line focus I virtually _never_ need to use my mouse outside of gaming.

On my Mac (work machine) I use [Amethyst](https://ianyh.com/amethyst/) (mentioned in earlier blogs) which does some of this, but the Windows/MacOS regime of forcing specific key combinations to do things and providing no way to override them leads to a mediocre experience when compared to something more customizable.

For the sake of fairness, [Powertoys](https://learn.microsoft.com/en-us/windows/powertoys/)' `FancyZones` is making headway on this in the Windows world, but is still leagues off of what any window manager on Linux (or BSD, etc...) can do. I'll also note that Amethyst has a [Windows port](https://amethystwindows.com/) however due to OS-level restrictions in how things can work, this loses a lot of practicality. Cool project, but not usable IMO.

# Setup was surprisingly quick

While I don't want to turn this entirely into a Windows bash-fest (or do I...? :thinking:), I was thoroughly surprised by how quickly I was able to get everything set up. Having used Linux in VMs, and MacOS, for years now I've got a good collection of dotfiles. It's hard to overstate how wonderful it is to just be able to clone a repo, set up a symlink, and have your desktop environment configured _exactly_ how you want it to be.

One of the other nice things is that virtually (literally?) everything in this world is stored as a configuration file. If I ever need to reinstall, I've already scripted out all the package installations I want, my `xrandr` config, `fstab`, and so on.

# What's the catch?

I mentioned earlier this isn't a great casual solution, and I stand by that. While Steam makes gaming on Linux trivial, Lutris is a different beast. It's still _vastly_ easier than trying to game via `wine` was a decade ago, but I've done more tinkering than I expected to get things running. With that said, pretty much everything works... even the stuff I've rigged to run off mounted NTFS partitions from my Windows installation.

Another catch is that _not everything_ can run here. Notably, for me, there's no way to use XBox Game Pass due to how the apps are downloaded. This is a bit of a bummer, but given how little I use it on PC I don't think I'll mind the occasional reboot into Wintendo.

Lastly, from a gaming perspective, there's some weirdness that I have yet to deal with in terms of using things like mod managers (Vortex, etc...). While this really only affects a handful of games, it's part of the reason I play them on PC.

# Loose ends

The endless tinkering continues... Some day it'll be worthy of [/r/unixporn](https://reddit.com/r/unixporn)!
