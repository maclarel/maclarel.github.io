+++
title = "nSec 2026, or Man vs Machine"
tags = [
    "ai",
    "security",
    "career"
]
date = "2026-05-19"
toc = true
+++

# Another fantastic week in Montreal

NorthSec was wonderful, as always. As I've noted in previous posts about it, NorthSec is my _do not miss_ cybersecurity event - both the conference and the CTF. While I've had the opportunity to speak at NorthSec in the past, this year was special - I got to run a workshop with a former colleague! More on that in a bit...

The speaker content this year was fantastic, and I again found myself with virtually no free time between delivering our workshop, attending talks, and talking shop with others in attendence. 

# Highlights & hot takes

Before I blab about my workshop, let's go through some highlights of the event for me.

## Living Off The Pipeline: Defensive Research, Weaponized

Recording @ https://www.youtube.com/live/q4mDyEnJ8ak?si=4hW4TTRqP8oSr4GB&t=10289

This is a must watch if you use GitHub Actions. Between the work of people like Francois, Adnan Khan, and John Stawinski I was already thinking twice (and twice again) any time I pushed a workflow to a public repository - this talk will hammer that home even more.

Francois and his team have built out an exploitation toolkit, described as Metasploit for GitHub Actions, that makes it so easy that you never even have to know what you're exploiting. It will automatically detect possible injections, generate payloads for them, and submit them to prove them out. The best part, in my opinion, is that despite its ability to do this all autonomously it **will prompt the user at every step** with warnings about what they're about to do.

If Boost Security isn't already on your radar with their Poutine tool, they should be with this. You can even try it out against their live CTF environment (covered in the talk) if you don't want to jump into using it on a real target.

## Hacking Browsers: The Easy Way

Recording @ https://www.youtube.com/live/q4mDyEnJ8ak?si=TlmAsj4H2lUOLMim&t=13279

Robbe's talk was a great insight into the reality that you don't have to be the kind of person who submits to pwn2own in order to have the capability to hack browsers in interesting ways. He details several cool findings that he's submitted over the past few years, how they were found, and the types of things to consider when bug hunting.

As with a lot of software bugs the issues are in the implementation - not necessarily "broken" code.

I definitely recommend this one if you're looking to expand your knowledge about the browser ecosystem, especially Electron browser implementation.

## Increasing detection engineering maturity with detection as code

Recording @ https://www.youtube.com/live/k6P8RyPGJ1M?si=3VMkT0S__D062lIj&t=16016

A long, long (like really long) time ago I was an idiot and I thought that if tools didn't have a GUI to manage them they were just a lousy hackjob. Then I had to manage more than just a single service on a single machine xD

Emilio talks through his company's migration from that type of management for SOC rules to using Sigma and being able to define rules-as-code - something that we actually also covered in our workshop.

While this doesn't seem like such a novel concept, it's a great thing to explore for those coming from significant legacy systems and wanting to understand what a migration looks like. Of particular value is the discussion of the rules submission, review, and promotion process. When you can democratize security in an organization by allowing the team(s) to submit pull requests with new rules/rule updates it can be significantly empowering _and_ increase velocity of rule creation and refinement.

## Commit, Push, Compromise: Attacking Modern GitHub Orgs

Recording @ https://www.youtube.com/live/-Q-_UwVDPaE?si=DsI_DeREbPacbBaz&t=3542

Similar to Francois' talk on day 1, Andrew and Max give an overview of current methods of attack against GitHub organizations. While slightly out of date on a few notes (though *many* vulnerable implementations still exist), the main takeaway here is that it's quite easy to make mistakes or assumptions about how things work.

Worth a watch if you're in charge of a GitHub org, or interested in pentesting that space.

## Red Teaming Mindset and Methodology

Recording @ https://www.youtube.com/live/-Q-_UwVDPaE?si=4U1q_brtcEuRYEQC&t=8855

A warning up front - like all of Charles' talks, this one is *dense* with information. While less technical than some others, he still managed to find a way to bring up internal Windows APIs (in detail, I might add) in a non-technical talk.

This talk is effectively a brain dump and talk-through of Red Teaming vs Penetration testing, and the considerations that need to go into it. Opsec, available access, pivots, imperfect knowledge, and specific considerations at each step of an attack.

I will, however, put in writing that I have some strong disagreements with a few of the points Charles brought up in this talk. To his credit, he also plainly stated that they're opinion and I think they're also legit viewpoints. Very specifically, he makes a point of calling out that "assumed breach" exercises are *not* "red teaming". While I understand where he's coming from, putting red teaming in a narrow box of "you must start from nothing and get to something, ideally undetected" I think does a significant disservice to internal teams (such as mine) and *significantly* discounts a real attack scenario - insider threats.

I'll write my own ramble on the topic at some point, likely after a few beers, but for now I'll just say that if you're interested in getting a better understanding of what red teaming is **you should watch this talk**.

## A Needle in a Haystack: Identifying an Infostealer Attack Through Trillions of Events in a Large-scale Modern SOC

Recording @ https://www.youtube.com/live/-Q-_UwVDPaE?si=K-eE3xK8muU4sPIc&t=12458

I'll openly admit that a lot of this talk was over my head when it comes to the data science side of it, but Francois is excellent at taking complex math stuff and explaining it in a way that can make sense to my glassy-smooth brain. 

Francois goes over the challenges of a managed SOC dealing with literal *trillions* of events and distilling those down to an actionable number of alerts. From memory, I believe going from the aforementioned trillions down to about 50 per week, per customer - an actionable amount for their staffing.

Also working at scale in a comparable space, there's a real art to this and it's cool to see the implementation behind it for Sophos.

## Measuring AI Ability to Complete Long Cybersecurity Tasks

Recording @ https://www.youtube.com/live/-Q-_UwVDPaE?si=UlbyLD_bM3Vta1GC&t=14587

As with last year, Jeremy remains one of my favourite speakers. He is truly an excellent presenter - being able to take broad, data-rich concept and turn it into a narrative journey for the listener.

Having taken part in this study, albeit not with extreme depth, I was very interested to understand what the outcome was. Unfortunately, it's about what you expect - AI is getting (already is) *incredibly good* at various tasks.

But remember what I noted about Robbe's talk earlier? The issues are often in the implementation. A human understanding of a problem space is still critically important and is an area that's going to be difficult, if not impossible, to automate away in anything that interacts with us meatbags.

I highly recommend watching this one if you're apprehensive of the current AI space. Things are moving fast, and this talk was probably the perfect segue into the CTF...

## Other talks

Honestly, everything I attended or caught parts of, were excellent. NorthSec CFP reviewers are great, and do a wonderful job of ensuring that everything presented is solid - no vendor pitches, no bullshit, no fluff. Check out the recordings _and the workshop recordings_ as you've got time. 

# The CTF

This year's CTF theme was Solarpunk, and as always the nSec team did a killer job. The challenges were all themed - hell, there were *storylines with plot decision points*. The challenges ranged from teh trivial beginner track to some so eye-wateringly difficult that I don't believe any team got all of the flags this year, let alone coming close to last year's tie that was decided by timing (<1 minute apart)! As a reminder, NorthSec's CTF is _exclusively_ in person, and includes multiple blackouts for things like Hacker Jeopardy and sleep. While some challenges (reversing) can be done offline, they're the extreme minority and there's no remote network access possible at all - teams are limited to the 8 players that they can physically fit at their assigned tables.

Now that all said, as much as we can enjoy a Solarpunk future and hopefully aspire to it, things right now are looking a lot more _cyber_-punk than they are _solar_-punk. The elephant in the room this year was the use of AI in solving challenges. The organizers were extremely up-front about this, even clarifying in a newsletter several weeks before the event that teams were welcome to use AI to any extent they wanted, but they asked that flags found with AI be noted as such with an additional argument (--agent) in the flag submission tool so they could do some information gathering.

Our team was in this to _learn_, not to win. Even with an unlimited AI budget we wouldn't have won since there were also physical-track challenges (RF, tamper-evident, social engineering, etc) that actually required human involvement. Ironically we finished all of those, except RF where we both lacked time and expertise to take it on. With that in mind, we made use of a _generous_ $50 of credit across Anthropic and Deepseek Pro (via opencode), and used the tooling to help us find entrypoints, refine payloads, and when it did autonomously solve challenges - to explain them in detail with thorough writeups that we could review. 

By contrast, one of the top teams spent nearly $3,000 CAD (yes, three _thousand_) on token spend across Anthropic and OpenAI models, deploying "agent swarms" to attempt to solve challenges. It's a safe bet that the spend of some other teams wasn't far behind that given conversations we had with a few of them earlier on. With that in mind, our final position on the scoreboard was actually a tremendous ego boost!

Perhaps more interesting, though, is that despite all of that, AI-first challenge solutions accounted for only about _half_ of the submitted flags - and that's with those teams being extremely open about what they used AI for... It was literally programmed into their harnesses. As a fun side note, virtually every team was using the 4.6 models from Anthropic as 4.7 models were functionally worthless due to constant AUP rejections - a colossal step backwards in functionality. That said, even with 4.6 we found that our sole `Pro` subscription was effectively worthless as it would get throttled working on only a single channel. More on this in another post to follow...

We found that many challenge creators included intentional rabbitholes for AI to get lost in, either through rate-limiting, unsolveable problems/endless loops of behaviour, or including _physical_ parts of challenge tracks. For example, one of the challenge tracks actually required an MFA code from a parallel track and it would have been (practically) impossible to brute force it. Any agent trying to figure out a bypass to that was just burning the team's money. Fortunately, since we aimed to be a bit more hands-on, we dodged _most_ of those challenges.

On the flip side of that, we had a multitude of challenges that we would have been completely unable to solve (primarily reversing/pwn) due to a skillset mismatch that Claude/Deepseek _absolutely trivialized_. At one point we set up a few of these challenges in a VM with Claude using `--dangerously-skip-permissions` and we came back to the entire challenge chains completed and flags ready for us in a thorough writeup... I can only imagine how frustrating that would be for challenge creators.

Our team, Cyber Crew, finished in 11th place out of the 89 teams (~700 players) in attendence. This was with an AI budget less than we spent on beer at lunch, and being two players short as they were event staff this year. I'm extremely proud of where we landed, and either in spite of, or thanks to, AI we learned a huge amount.

My hands are already tired from writing all of this, but stay tuned for more thoughts on the AI bit. For the moment, let's just say that some problem spaces are arguably now "solved", but I think others will live on with adversarial design for quite a while.

*A quick note coming back to this - one thing I want to call out is that, as far as has been discussed/I am aware, no team was making use of local models. The perforamnce delta is just too dramatic compared to the frontier models (and supporting hardware) to be feasible for a problem like this. Bespoke development, AUP bypasses for specific problem sets, phishing/malware development - sure. Tasks requiring significant agentic work and huge context windows? lol, no.

Before moving on, I want to give a shoutout to two challenge designers specifically.

## Rayan - SunBloom Library & Helios Fleet Network

I spent an inordinate amount of time on these two challenges, being able to solve Helios myself but being completely stumped as to the entrypoint of SunBloom (as was Claude _and_ Deepseek).

Both of these (web) challenge tracks had very fun designs, incorporating very _real_ problems into their design. The takeaways from these weren't "oh that's cute", but rather "I've actually seen this in the real world... several times".

SunBloom was particularly interesting, as a teammate and I spent hours doing enumeration, review, and threw quite a lot of AI time at it and just _could not_ find an entrypoint. Cleverly, the challenge description noted a second service as part of the challenge so we of course focused our efforts there - as did AI... it was a complete red-herring. We just needed to try harder (read as: fuzz better) to find the initial path traversal bug. Finally, after kicking things off with a better model (Opus 4.6 rather than Deepseek Pro/Sonnet 4.6) and taking a break for lunch we came back to all of the flags for it. A deeply disappointing solve after the time spent, but it was _very_ fun reading the walkthrough to understand the attack path and how it was designed.

Huge kudos to Rayan - also their first time submitting challenges to nSec! Hope to see you back next year!

## Joey - Multi Facteur Authentication

This track was mostly done by my team, but was very "physical", encompassing finding flags in meatspace, social engineering with forged IDs, OSINT, and **shockingly** _human interaction_. 

I only worked on little bits of the track, but I want to give props to Joey for not breaking character a single time in our interaction throughout the weekend. Really fun track - great work!

# Our workshop - Command & Conquer: A C2 primer for aspiring Red & Blue teamers

I've been on both sides of the fence (red and blue) in varying capacities throughout my career (professional or hobbyist), and have often found that the mixed experience there has been hugely valuable in problem solving. To that end, I've been trying to focus on sharing that knowledge in a productive fashion where I can. 

One thing I've come across is that a lot of folks know what command & control _is_ but not necessarily the intricacies of deploying it, building payloads, opsec considerations when operating agents on compromised endpoints, and in many cases have probably never been hands on with it. Similarly, from lived experience (maybe a myopic view, though), a lot of folks in offensive security - especially early career - have never stopped to think about what building detections looks like.

With that in mind, I teamed up with a former colleague of mine on the Blue side and we built out a workshop to act as a crash-course in C2 operations with Mythic and writing detection rules with Sigma and Yara. We had originally pitched this as a three hour workshop, however due to scheduling constraints we had to cut it down to two hours and with that had to remove some of the more in-depth scenarios.

Despite the timing setback, the feedback we got from attendees was excellent (please yell at us in the nSec discord if you feel differently!) so we're planning on expanding this further and will try to be back with a full-day training at some point, or at least some more pointed "200-level" additions that can act as focused workshops. Maybe we'll have two hours of _just_ red or _just_ blue next year? 

You can catch the full recording of the workshop @ https://www.youtube.com/watch?v=iB-BXCojjQA. The C2 infrastructure is offline (permanently), but all of the course materials are available to you for free, including an Ansible playbook to set up Mythic on your own Ubuntu VM. 

# In closing...

AI might take my job eventually, but for now I'm going to continue to learn as much as I can. I think it would be ignorant, in fact downright counterproductive, to not spend significant time working on better ways to harness LLMs in security workflows. I'd like to think on the leading (but definitely not _bleeding_) edge of that in my own work, but there's always more to learn. The work of some of the CTF teams to harness agent swarms was truly impressive to see after the event completed, but it also really hammered home that complex problem solving with AI is really going to be exclusively a corporate/government endeavour once subsidized costs (continue to) vanish.

Anyway, **go to NorthSec**. You will meet great people, you will learn a ton, and you will have a blast.
