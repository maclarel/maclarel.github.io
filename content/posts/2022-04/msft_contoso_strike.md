+++
title = "Microsoft Strike CTF 2022 - Contoso Financial"
tags = [
    "ctf"
]
date = "2022-04-20"
toc = true
+++

# Premise

Microsoft runs an internal CTF event every so often called `Strike CTF`. While we've been asked not to share specifics around the challenges involved, I wanted to do a brief write-up, as it's certainly a unique one that catered to multiple skill levels and had an educational twist.

For anyone interested in doing some digging, the version Microsoft runs has also been seen in some B-Sides events (and other conferences) under the name `ShadowBank`.

# Event format

Unlike a more typical CTF, this event didn't just have a few "flags" to find - it had _fifty three_ and an indeterminate amount of _bonus_ flags.

Players were presented with a web application noted as being vulnerable (and oh boy was it...), and were instructed to find ways to identify and exploit many of the [OWASP Top 10](https://owasp.org/Top10/). Uniquely, this application would also _automatically detect and award_ exploitation, which made it very cool for beginners who wanted to poke around and try things.

The 50 _main_ challenges were focused around:

- Reflected XSS
- SQL injection
- Cracking hashed passwords
- Session hijacking (to a very small extent)
- XML/XPATH injection
- Exploitation of client-side validation
- Usage of hidden forms/weak validation
- OSINT/Phishing

Note: The scope was specifically the web application - shell access, if even possible, was out of scope.

The event ran for two weeks, with various hints and bonus assignments being dropped by the group running it along the way.

Overall, a pretty great package for someone who is getting into web app penetration testing like me!

# So how did it go? 

Well. _Too_ well... Within the first few hours I'd already dumped the entire backing database and gained admin privileges within the application. 

As it turns out, the real challenge here is understanding _all of the vulnerabilities_, not just exploiting them.

Going back to the automated scoring, if you perform some basic SQLi, like an auth bypass, you'll probably get some points awarded for it, but what if you do something more malicious with that form like update some data or drop a table? That might also give some points... What if you do that again in a different form, or URL, etc on the site?

Effectively it was a few hours to pwn the application and database, and then another dozen or more hours of hunting down the ways the organizers _wanted_ the application to be exploited. This lead to some really fun team work (which was explicitly permitted, we asked!) to do some divide and conquer on enumerating other vulnerabilities and challenges.

The other interesting thing here is that the 50+ "flags" *are not* enumerated from the start, so it's a hunt to exploit the application in any way you can, as well as trying to exploit it in ~50 different ways. You can imagine how surprised some of us were to find out that there were additional SQLi challenges that we hadn't found despite having already gone so far as giving ourselves infinite money and dropping the database.

# Bonus objectives

One of the really fun parts of this event is that there were also _completely unknown_ bonus challenges that could be scored. Effectively, if you could find a novel way of exploiting the application (e.g. session hijacking, persistent XSS, CSRF, Phishing, etc...) you could be awarded additional points from a hidden list that the organizers had.

While this meant that there was a *lot* of time spent trying to find novel approaches to things, and submitting them to the organizers for review, it did keep things quite fresh even after completing a vast majority of the base challenges.

I will note that one area here that was frustrating is that there were a small number of bonus objectives that were only available if you reviewed _out of scope_ parts of the event, and you'd only gain knowledge of them by intercepting traffic for said out of scope website. This was quite frustrating, but fortunately didn't come into play for overall standings - I've of course politely shared this feedback with the organizers. With that said, I found all of these since they could be identified passively - no need to attack anything out of scope, just look closely at the traffic you receive when loading it.

N.B. There's a fun easter egg if you try to get a web shell via arbitrary file upload ;)

# Final standings

Out of ~140 folks who entered and completed at least some challenges, I landed in first place by about a 3% margin on score!

Given that this was my first "competitive" CTF event I'm quite pleased, though it was certainly on the easier side compared to many of the boxes on HackTheBox/TryHackMe - at least in terms of understanding attack surface and getting a foothold/privilege escalation.

If any folks reading this have the opportunity to participate in one of the ShadowBank/Contoso Financial CTF events I'd *highly* encourage it - especially if you're newer to "hacking".

Now to wait for the prize swag to arrive. GG!
