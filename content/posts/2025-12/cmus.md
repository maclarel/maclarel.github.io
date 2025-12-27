+++
title = "Listening to music from the command line like god intended"
tags = [
    "linux"
]
date = "2025-12-27"
toc = true
+++

# Really dude?

Yeah.

# y tho

I got rid of Spotify. I'd been a subscriber for ages, and being completely honest the value is there for _consumers_, but they've got some seriously shady business practices and really aren't of value to musicians beyond discoverability. I'd also realized that the more I thought about my interaction with Spotify the more music became a backing track to something else rather than the actual focus. 

There's totally a place for ambient music, and I have stuff like that on most of the day, however I don't need to be _paying_ for that.

# So what now?

Well I already have a massive collection of (primarily) legally obtained music through rips of a decent CD collection or digital downloads accompanying albums I own, so I figured I'd do the double-whammy...

- Be more intentional about what I listen to
- Remove the convenience that is a plague on enjoyment

# From the command line?

Yeah. I took a look at multiple GUI-focused options on Linux (Strawberry, VLC, mpv, etc...) and nothing really clicked. I want something simple - like foobar2000 or WinAmp, but also not unimaginably hideous like most Qt apps are. That pushed me over to command line stuff.

I started out by looking at mpd + rmpc, but it's more complicated than it's worth for my setup. I just wanted something that would play my music and offer the most basic of controls.

I started out looking at [moc](https://moc.daper.net/) but it was obnoxious to get working, so working through [the list of options in the Arch Wiki](https://wiki.archlinux.org/title/List_of_applications/Multimedia#Audio) I decided to look at [cmus](https://cmus.github.io/) and it has been a match made in heaven. 

# Why cmus?
- Simple interface
- Vim bindings
- Great user documentation
- Incredibly lightweight
- Works with playerctl with no additional futzing about

10/10 recommended.

# What about podcasts and stuff?

This was actually a bit of a concern at first, as I listen to a lot of work-related podcasts on Spotify. It turns out that they're virtually all available through other mediums. To that end, I'm trying out [Podcast Republic](https://podcastrepublic.net/) on my mobile devices. If that doesn't work, YouTube or PocketCasts are probably the next best options.

# Ok, now the bad

The one thing I'm really going to miss is the discoverability of new bands. Spotify has absolutely nailed this in a way that I haven't seen repeated elsewhere. I wouldn't have found a majority of the bands I have in heavy rotation if it weren't for stumbling across them in Spotify mixes or recommendations. I'm optimistic that I'll be able to find bands organically both through local concert scenes and being more mindful about what I'm listening to, but we'll see... If this is a deal breaker I might have to hold my nose and resubscribe.
