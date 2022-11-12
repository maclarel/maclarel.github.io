+++
title = "Contributing to a CTF event"
tags = [
    "ctf",
    "ekoparty"
]
date = "2022-11-11"
toc = true
+++

# The project

I recently had the opportunity to join a team building out some challenges for the [Ekoparty](https://ekoparty.org) CTF event, as my company has been a sponsor for a number of years. Given that I've been reasonably involved in playing CTFs as of late, as well as helping organize events within my company, I figured it might be fun to try building some challenges as well!

I worked with two colleagues to put together a series of 4 challenges:
- Obfuscation (very easy)
- Security misconfiguration (easy/medium... or so I thought)
- Vulnerable web application (medium)
- Reversing/binary exploitation (very hard)

As I've got little to no knowledge in the area of the final challenge, I focused on the first three. Specifically, I designed the overall challenge flow, helped design those three challenges, and wrote the entirety of the second challenge.

# Designing the challenge flow

We had envisioned a set of sequential challenges, of increasing difficulty, all flowing into each other. We knew we wanted to cover a few different areas of challenge content, and that we wanted to work in our own product somehow. In the end, we decided on a "university" theme where students must first pass an admission test, then exploit a grading program to get access to another system. From that system, they can then leak an access token that will take them to the final challenge - a binary that the "admins" use to manage GitHub-hosted services (an API wrapper, basically).

Without going into too many details, as we're working on writeups for several of these challenges, we thought we had a winner in terms of our design. Unfortunately we hadn't realized that a somewhat non-traditional design was probably going to steer folks away from our challenges in favour of the dozens of others included in the event.

# The challenge I built...

... was originally based around [a reasonably well known Ruby vulnerability](https://staaldraad.github.io/post/2019-03-02-universal-rce-ruby-yaml-load/), however throughout testing we identified that there were some trivial bypasses to the intended solution given how we were hosting the challenge. This lead to a full rewrite of the challenge two days before the deadline... nothing like a little time crunch to get the creative juices flowing!

In the end, the challenge wound up being based on [using GitHub Actions](https://github.com/marketplace/actions/approve-pull-request), with some lax security configuration, to work around branch protection rules. This would allow the player to merge whatever code they wanted onto the default branch and leak secret values that were only accessible from that branch. When going through this, I was worried this challenge was going to be far too easy since all you had to do was create a branch, put a new workflow on it, open a PR, and use another branch to run a workflow to approve it... Out of 200+ players who attempted the challenge, only 7 suceeeded.

Ironically, _dozens_ of players spotted some leftover code from the original challenge that I kept as a rabbit hole, and wound up trying to use the aforementioned Ruby vuln. The real fun part there is that it was still a valid way to solve the challenge... but you had to find a way to get that code into the default branch to run it.

I'll have some more info about this challenge in the future as well :)

# Lessons learned

If you're going to be submitting challenges to a larger CTF event, there are a few things worth understanding up front:

- How are the challenges laid out? Is it jeopardy style, or more progressive?
- How may challenges are there in total?
- What's the average skill level of the players involved and are you an appealing target on CTFTime?
- What are the other types of challenges that have been submitted to the event?

I note these in particular as they're areas that we didn't fully understand coming into it. No shade toward the event organizers at all - this was on us to ask about and understand and we didn't do so... but we did learn from that!

## Event style

If you're participating in an event that's jeopardy style, building a challenge like ours that required players to work through stages sequentially is going to stick out like a sore thumb, and will likely be avoided in favour of ones that can be attempted quickly.

## Total number of challenges

If you're looking at an event with only a small number of challenges, you may have a bit more flexibility to try something out of the ordinary. On the other hand, if it's possible to build up a large enough score to win the event and still dodge a significant number of the challenges, you'll want to make yours appealing enough that people will attempt them, and difficult enough to challenge them.

## Average skill level

I'll be clear that this is said without a clear understading of the overall playerbase. We were very surprised by the low number of solves for our challenges. Out of 426 teams that posted a score, we had ~225 solves for stage 1, 7 for stage 2, and 3 for stage 3. No one solved the final challenge we submitted.

I think this is mostly on us for not meshing well with the overall event format (submitting a sequentially staged challenge to a jeopardy style event), but none the less a surprising outcome.

## Types of challenges

This one I think is a more general statement than just this event. Again, given the style of our challenges, we stuck out a bit. We had four different categories of challenges listed out under a single heading (since we were a sponsor), rather than having them mixed in with the other relevant sections. A "one-off" challenge style would have played far better into all of this.

With that all said, we learned a ton, and we're already planning for next year. It has also been inspiring to me, and already has me working on some new challenges since I've now got a better idea of how to build at least some basic ones after having solved dozens and helped to build a few others.

I want to share a huge thanks to Antonio and Jorge who worked on these challenges with me, as despite the less-than-ideal outcome I think we all learned a lot, and the feedback we got from event organizers and players was largely positive!
