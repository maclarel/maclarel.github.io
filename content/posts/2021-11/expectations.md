+++
title = "Relentless follow-through in Support"
tags = [
    "career",
    "support",
    "bestpractices"
]
date = "2021-11-08"
toc = true
+++

Way back in April 2020 I wrote an article called [Written communication for support](../../2020-04/written-communication-for-support/) which covered some high level best practices around setting, managing, and resetting expectations. Think of this as a spiritual successor to that article, as we're going to talk about the other side of the coin - relentless follow-through.

To put "relentless follow-through" in more direct terms, I want to discuss some best practices around timeliness of responses _even when there's no news_, internal follow-up and expectation setting, and some options for collaboration (depending on the size/maturity of your business).

# Relentless sounds like a strong word... 

It is.

While I don't remember the exact etymology of this phrase, this is something that was said to me early in my career and it really stuck. Frankly, regardless of _technical_ skill, I'd argue that the ability to effectively manage expectations and follow-through on commitments is the single most important aspect of success for anyone customer facing. This comes with a *lot* of "gotchas" though.

- How do you know what you're saying is true or realistic?
- How can you advise on timelines?
- What other resources (human or otherwise) can you leverage?

We'll deconstruct a few of these, but before we get to those I want to try to sum up this entire concept in one paragraph in case we hit a TL;DR situation:

*Your ability, as a support engineer, to keep your customers informed of the progress of investigations is paramount. In terms of customer satisfaction this is often more important than the actual time to resolution.*

## Setting realistic expectations

Without rehashing too much content in the other article, the key point I want to get across here is that it doesn't help to bullshit people. I'm not suggesting airing dirty laundry, or providing exhaustive detail, but if something flat out cannot be done it doesn't help to imply that it might be. Similarly, it's not overly helpful to imply that extremely minor (low visibility/low impact) bugs will get fixed quickly. There are some wording techniques to help get around these, and that will help convey this messaging in a more subtle fashion.

```
Sorry to hear that you're running into <Issue X>. We've been able to confirm this as a bug here, and are following up with our teams internally to work on a fix. I can't advise on when, we'll be able to deliver a fix for this problem, however I'd suggest keeping an eye on our release notes where a fix would be advertised.

In the mean time, we'd suggest the following workaround to help mitigate this issue in your usage:

1. 
2.
3.
etc....

Please let us know if you have any further questions!
```

The wording here is strategic in a few ways:

- The issue in question is specifically called out and acknowledged.
- Ownership of the problem has been transferred to another *non-customer facing* organization.
- Expectations are set that a fix may never materialize, or at the very least may not be prioritized.
- The customer is provided with information on how they can see progress.
- A workaround is offered to help mitigate their business impact.

Five key factors in, effectively, two lines of text.

Building on this, you may also find yourself working in positions where you're working extremely closely with customers and have relationships _with them_ that you can leverage. Often, issues will be exaggerated or at the very least not fully understood by the people reporting them. When you've got this level of access, especially in a real-time fashion, it can be extremely helpful to spend a few minutes dissecting the actual problematic use-case to understand both if it's a realistic scenario, and whether or not it's something that's worth everyone's time to follow up on.

As an example, you may receive a ticket like this:

```
Hi Support, 

X part of the UI is broken in your product. This is blocking critical workflows on our side. See attached.
```

A few helpful questions to ask to start (either in your own head, or to the customer):

- Is the only way to do what they're trying to do (i.e. is this actually blocking work)?
  - Note: This is two-fold - both helps clarify the priority of the issue, as well as your ability to offer alternatives.
- Is this problem actually a problem, or is it "working as intended" and unlikely to be "fixed"?
- Does the person reporting the problem fully understand the scenario, or are they just relaying information?

If you find that the answer to most, or all, of these questions is "No." there's ample opportunity here to have a discussion with the complaining party about what's realistic to deliver on and to negotiate a proper prioritization for both you _and_ them as they'll also have stakeholders to report back to.

## Managing timelines 

This is a very tricky one in the Support world, and I don't want to imply that you'll ever have much control over it given the interrupt-driven nature of the work and heavy reliance on other parts of your organization. Despite the lack of control over the *technical outcome* in some scenarios, this is also the concept at the very core of "relentless follow-through". 

At a high level, this is your time to set the cadence for _standing_ communication and by that I mean the pace of communication _when there is nothing actionable to provide to the customer_. As a side note - some companies address through this "next response SLAs", which I'd generally argue are overly aggressive and detract from quality, but you may see that come up. I'd generally suggest setting the expectation _and delivering on it_ of an update _at least_ every two business days outlining current status. Working on an investigation that has been escalated to engineering and haven't heard back? No problem, note that the investigation is still ongoing and you'll provide another update on status before end-of-day two days from then. Haven't had a chance to look at an issue that's in your ballpark to address? Same deal, "apologies, still investigating", etc...

Similarly, you'll generally want to use a similar cadence for issues where you're reliant on someone else to provide you with an update. Escalation with engineering? Ping them every 2 days (or more often depending on issue impact). Waiting to hear back from your real estate agent? Ping them every 2 days. This is something applicable well beyond just the world of Support, and I think you'll find it helpful in just about any aspect of life. Similarly, I think you'll get a solid realization of when other people are doing it... and when they *aren't*.

Realizing that I just provided seemingly trivial examples following a statement about how tricky this is, that's intentional. The actual _execution_ of this concept is also trivial, yet this is the *number one* reason I've seen for escalations in any support organization I've worked at. Customers are left wondering as to what the status of their investigation is, or why it hasn't progressed, or why they haven't heard back on questions they've asked. Obviously if things are dragging on for extreme periods of time there will be some legitimate frustration, but this sort of communication technique _and cadence_ will generally curb a significant majority of non-technical escalations.

## Leveraging others

While this may seem like the basics of working on any sort of team, this is intentionally last in my list as both of the topics above play directly into this. I'm not going to spend a significant amount of time on this point, either, as it's always going to be subjective. I'll leave it simply by stating that the relationships/connections you build within your team/company/customers are *almost always* going to be your most valuable asset in terms of delivery. Short of extremely simplistic products, it is virtually impossible for an individual to be an expert on *everything* - your ability to know who to ask _and to leverage your relationship with them_ to get a timely response/deeper understanding will pay dividends even beyond the scope of an individual job.

# This all sounds so simple 

It does, and in some respects it is. The biggest challenge around all of this is the mindset shift to fit within it and the actual execution on a consistent basis. It may seem counterintuitive to send someone an update saying that there's no update, or to nag someone that you trust is doing the best they can, however in both cases it demonstrates *your* investment in the situation and that in all regards nothing, and no one, has been forgotten about.
