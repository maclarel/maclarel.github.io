+++
title = "CyberApocalypse 2023 - A CTF Competition"
tags = [
    "career",
    "ctf"
]
date = "2023-03-23"
toc = true
+++

# Another month, another CTF event 

HackTheBox runs a large, free, CTF event every year called CyberApocalypse. These events tend to have a theme, though it's largely irrelevant to the actual challenges (or often their content), however it creates a fun little narrative to go along with it and sometimes gives contextual clues.

I participated again this year with a bit of a rag-tag crew from my work, and it was fun as usual. Unfortunately the event also happened to overlap with some previously planned travel, so I only really got one good day of time to poke at the challenges. With that said, that was enough time to help our team get a bit of a foothold, and for me to learn some new things!

This year's event had several categories that we've seen before (reversing, pwn, web, forensics, "misc", hardware), as well as some new ones (ML, blockchain), and notably not a single "full pwn/machine" challenge which was quite disappointing for me as those tend to be what I focus on.

# First off, how did we do? 

I'm writing this the day before the event ends, however I suspect we won't move too much further up or down at this point. Our overall placement is looking to be roughly in the top 10% of teams (~600th out of 6400+ teams) *Edit: We finished 654th out of 6,482 teams*. For a random group of folks doing this for funsies, and having _zero_ experience in several of the categories (blockchain, ML, hardware) I'm quite happy with where we're sitting even if we could have done better with some more time.

As of writing this, our team has completed 24 of the 74 challenges available. Again, overall certainly not a competitive outcome, but progress in comparison to previous events!

# The challenges

Without spoiling the challenges, there are a few that I found particularly interesting, though reasonably straightforward...

- A challenge requiring rapid responses to server prompts with math questions and specific error/bounds handling. In short, a fantastic introductory experience for using `pwntools` to script out payloads based on server responses.
- A challenge where you're provided with a packet capture and need to extract a flag. There were some hints in the various captured HTTP requests that a webshell had been uploaded, so for the most part it was a matter of retrieving the correct file. From there, the PHP code of the webshell was heavily obfuscated, but functional. To solve the challenge you could either (likely) deobfuscate the PHP _or_ since it all resolved into a single function, just add a `print()` call referencing the function to dump out the deobfuscated code which happened to contain the flag in a code comment.
- A reasonably simple SQLi challenge demonstrating why staggering username/password lookups are not ideal. I suspect, based on the flag, that there were actually a few solutions to this challenge, however since there was a SQLi opportunity due to not using prepared statements it was possible to add a `UNION SELECT` to the query to force the injection of another password hash that would be used for the eventual comparison.
- One near and dear to my heart, a password manager with a GraphQL interface containing an IDOR vulnerability allowing the modification of other users' credentials as long as you were authenticated (i.e. there's no check if what you're updating is actually accessible to your user, just that you _are_ a user).
- And finally, from the ones I completed that left an impact, after nearly 20 years of working with Linux I was introduced to `rbash` and its restrictions. In short, this was a restricted shell that only allowed very specific commands to be used, and environment variables like `PATH` were read-only. Fortunately, the use of shell built-ins was possible, and `echo` can be used to both list directory contents (e.g. `echo /*`) and the content of files (e.g. `echo /flag*`).

There were several others that I solved as well, however these ranged from reasonably trivial to relatively uninteresting (possibly complex, but not overly challenging). Others on my team with some more experience in reversing worked through some challenges in that space, and we paired up on some other blackbox coding challenges leading to some extra points and learnings!

# Overall

It's likely that my perception of this event is a little bit soured by the time constraints I had, but it was great to see some other folks engage with the team even if only for brief periods of time. 

I look forward to more of these, though I do feel that previous CyberApocalypse events (and other miscellaneous HTB events) did offer a somewhat more engaging experience _for the types of challenges I find interesting_. I do certainly hope that they bring back machine/full pwn challenges as I personally find those to be the most engaging and rewarding. I do hope that, similar to previous years, the web challenges I did not solve are released as permanently available challenges for members so that I can pick them up and continue to work on them as I have time.
