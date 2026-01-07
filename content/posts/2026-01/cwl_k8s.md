+++
title = "CyberWarFare Labs K8s RTA Thoughts"
tags = [
    "linux",
    "security",
    "career"
]
date = "2026-01-07"
toc = true
+++

# TL;DR - It was $20

...and I got my money's worth as someone who hasn't done a huge amount of hands-on work with K8s in the past. I wouldn't pay full price for this at $100. There, saved you reading the whole thing.

# The course

The course is advertised as being multiple hours of video content and 180+ pages of supporting content, and it delivers albeit with that 180+ pages of content being the slides that are used in the videos rather than something separate.

If you dig into the FAQ it also states that it's expected to take about [20-30 hours](https://cyberwarfare.live/product/k8s-red-team-analyst-k8s-rta/), however this is wildly overblown. Having not touched K8s beyond labbing with minikube nearly 10 years ago, and going through the content at ~1.25x due to the presenter's slow voice and frequent repetition (more on this in a bit) my total time spent _including successful completion of the exam_ was around 5 hours and I've repeatedly been accused of having a very smooth and glassy brain.

The course content _is_ good, though nearly everything that it covers is incredibly contrived and unlikely to be seen in any sort of serious environment (e.g. completely exposed and unauthed control plane services are nearly half of the course content). It's great content to know the basics of navigating through a K8s cluster, using kubectl, buildilng config files, defining roles, etc... so from a fundamental standpoint alone I'd argue that I got my money's worth. Some neat TTPs, but don't expect to actually be able to apply any of this stuff verbatim. I expected a bit more depth given the advertised duration/volume of content, but it was still useful.

My only other gripe with the course content is that (all of?) the videos were shot in one take and have been left unedited - this means that in several of the videos, especially earlier on in the course, the instructor repeats the same section multiple times in a row clearly expecting cuts and edits. For $20 this is forgiveable but it's certainly not the level of professionalism I expected from paid training.

# The exam

The exam is a bit of a joke, to be honest. It took me under an hour to complete from start to finish. It's CTF style and there's strong hinting as to where the flags are, though it's up to you to figure out how to retrieve them.

This was honestly a bit disappointing as there were multiple mentions throughout the course of additional research being required for some exam questions, as well as having significant portions of the documented (possible) attack surface on K8s clusters being completely absent from the final exam.

# Closing thoughts

This is worthwhile if you can get it for dirt cheap, don't know a lot about Kubernetes and want a quick crash course on how to do the basics of enumeration, priv esc, and lateral movement. Don't pay full price.

I'm not sure if I'd pay for another course from CWL, but I'm not upset that I paid for this one /shrug

[Proof](https://labs.cyberwarfare.live/credential/achievement/695eb383823d7631cc46d378)

