+++
title = "HackTheBox - Cyber Apocalypse 2022"
tags = [
    "ctf"
]
date = "2022-05-24"
toc = true
+++

# Well, that was harder than expected

Many times in life it's great to have a humbling experience. Not that I had an ego in this space anyway, I'm super new to it, but I was going in expecting to be able to crank through a fair number of challenges for this event. Let's just say that didn't happen.

Cyber Apocalypse 2022 was a jeopardy style CTF event with categories across all of the usual HackTheBox challenge content. I decided to primarily focus on Web as I've got the most experience there, but also dabbled in a few others.

I also had some split brain going into this as another CTF event was taking place at exactly the same time, though was quite easier. None the less, between work, that event, and this one I had some serious split brain going on.

# The Challenges

Unforutunately I can't track down the names of any of the challenges, so I'll need to work from memory here. From what I recall, I complete two of the `web` challenges, one of the `pwn` challenges, and one of the `misc` challenges.

The `pwn` challenge, honestly, I think was broken. I was able to solve it by inputting random text of any length. I *think* it was meant to be a buffer overflow that required a specific length string, but something definitely went wrong.

The `misc` challenge was an interface that would accept file creation, and then compress those files. It didn't sanitize user-provided filenames, so it was possible to do LFI and pull in the flag quite directly (e.g. `Enter filename: ../../../../../../flag.txt`). Again, a bit of a freebie.

The `web` challenges were a touch more involved, and were quite fun, though obviously on the easier end. The first one I recall was a whitebox challenge, and was based around a website that would sign you up for a newsletter. While the email addresses were sanitized, and parameterized in queries, the website would also track the visitor IP and log that... unsanitized. Part of this logic also allowed for the use of `X-Forwarded-For` headers, and when paired with an unsanitized query and a `sqlite` database on the backend you get... *drumroll* free web shells:

```
X-Forwarded-For: honk', 'fart'); ATTACH DATABASE '/var/www/cmd.php' AS shell; CREATE TABLE shell.payload (payload text); INSERT INTO shell.payload VALUES('<?php system($_GET["cmd"]); ?>' ); -- 
```

The other challenge was a Markdown to PDF converter, and similarly was a whitebox challenge. This one was predicated around [a known vulnerability in `mdToPdf`](https://security.snyk.io/vuln/SNYK-JS-MDTOPDF-1657880), so once that vulnerability/component was identified the remainder of the challenge was trivialized.

# Final standings

Our team finished 919th out of 7024 teams (top 15%). For a first real competitive team event, with areas that several of us have _no_ expertise in (Hardware, Windows forensics, etc...) it was a great "learning experience".

We're also going to be participating in the DefCon CTF qualifiers at the end of May, and our goals there are very "realistic". Solving even a handful of challenges will be a win in our eyes...

# That other CTF?

The other CTF event was a follow up to the Microsoft-hosted event I wrote about last month. This one was quite similar, but new website, new vulnerabilities, and new bonuses. I finished 2nd out of 137 active participants, which shows quite the contrast in complexity between thoese events and ones like HackTheBox!
