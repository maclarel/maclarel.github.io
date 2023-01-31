+++
title = "January's woes"
tags = [
    "ctf",
    "career"
]
date = "2023-01-31"
toc = true
+++

# It's ~Morphin~ Bitchin time!

I try to stay pretty positive with this blog, so I'll keep this reasonably short. 

The Apple Silicon Macs are amazing performance, battery life, power consumption, etc... They're also nearly worthless for my usecase as a developer and user of predominantly x86 applications. 

- Fortunately, most of the Python ecosystem seems to support aarch64 so much of my code works. The problems in Python come when you need something that's x86 native... like trying to use `pwntools`. That's just a non-starter, so now I have to SSH out to another box to work on anything with that. Annoying. 
- Ruby's ecosystem though... just no. So many critical gems just don't have aarch64 support (yet?) which means I effectively can't do any validation locally. Again... I have to remote to something else to get work done.
- Docker? Nah. Emulation with Docker Desktop on macOS flat out doesn't work. I can create images for other architectures... maybe... sometimes... but I can't run them. See above.
- VMs? Nah. Non-native architecture emulation through UTM or VMWare Fusion is not a thing despite advertising. I did a drag race trying to set up a Win 10 (x86) VM between UTM (emulated x86) and a lower-spec personal machine. Both VMs had the same number of CPU cores, RAM, and access to local nvme storage. My x86 box had the install complete and booted in under 6 minutes. The UTM installation completed after about 53 minutes, and still had not successfully booted after another 40 minutes. See above again :\

It's kind of ridiculous to have a laptop like this when the only features I actually use are the nice screen (if it isn't docked) and the battery life. I'm grateful to have it, and for non-work purposes it's pretty great, but if you're looking at one of these for purposes _other_ than purpose-built Apple stuff I'd suggest looking at alternatives.

It's entirely likely that I'm missing some solutions to these problems, but the frustration stands. I've got a multi-thousand dollar machine that can't do simple tasks I require on a daily basis for work (and personal stuff).

# Now on to the fun!

I took part in another CTF competition this past month with the goal of helping test a new CTF platform for internal events. Single player teams, jeopardy style, 24 hour access. I came in 13th out of the roughly 65 players who completed any challenges, and > 110 signed up.

Honestly, it's not a result I'm happy with. 

Event categories included web, pwn, reversing, full pwn (machine), and forensics for a total of 15 challenges. I completed 5 of the 8 web challenges and 1 of the 2 forensics challenges.

I made extremely good headway on one of the two pwn challenges, but that's still an area well over my head. Plus side, I learned a lot about buffer overflows and ret2libc attacks. Still a total novice, though.

The frustrating thing for me is that I made such little progress in the full pwn/machine space as this is an area I tend to focus on in events. In this case I spent nearly 11 hours on a single challenge only to realize 15 minutes before the end of the event that the RCE vuln had been staring me in the face the entire time and I just hadn't realized what it was.

Lesson for others: If you see something and you don't know what it is, don't give up on your research easily. If it's insignificant, you'll probably still learn something. If it isn't, it might be what you need. In this case it was a string being surfaced (and then consumed) by the remote app that turned out to be [pickled data](https://docs.python.org/3/library/pickle.html). One small tweak to it and I would have had RCE. Boo.

Not that it matters in retrospect, but if I had caught that vulnerability earlier, I would have had a top 3 finish with just the points from that one challenge as only one person completed it.

The forensics challenge I finished was just deobfuscating Excel macros, which pointed me to a neat tool I hadn't seen before - https://github.com/DissectMalware/XLMMacroDeobfuscator. Worked like a charm, though for a simpler example.

Either way, another great learning experience. The next one for me will be nSec 2023 in Montreal. If you know a team looking for a mediocre hacker to join you... give me a shout!

# What's coming up in February?

Not a lot. Taking a little more time for myself in the coming month as part of the "decompression and reprioritization" I outlined in my last post.

I plan to continue poking at some challenges on HackTheBox as I work toward `Pro Hacker` rank (lol), and I have some work projects on the go that should be a lot of fun to talk about once they get a bit further on. Expect some fun articles about both of those :D

Beyond that... check out [Project Diablo 2](http://live.projectdiablo2.com/) if you enjoyed Diablo 2 back in the day and wish there had been some evolution of it that took a different and more thorough direction than D2: Resurrected. I've burned nearly 200 hours on this since the current ladder season started, and I'm sure there are dozens more to come!
