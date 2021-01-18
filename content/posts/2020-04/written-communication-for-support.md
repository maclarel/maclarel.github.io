+++
title = "Written communication for support"
tags = [
    "support",
    "writing",
    "bestpractices"
]
date = "2020-04-12"
toc = true
+++

I've been working in customer facing roles for quite a while now. Granted, these have largely had the luxury of being in software companies, but I've also done a brief retail stint and I'd like to think the same skills apply in all cases.

While this will be by no means exhaustive, I want to go over two high-level concepts that I feel are extremely important, and can sometimes be overlooked even by veterans in customer facing positions.

# Managing Expectations
This may sound simple, and it is a straight forward concept, but it's all too oft ignored. We'll dive into some examples below as to how easy it is to really nail this and create a great customer experience.

## Setting Expectations
Have you ever received this email when working with a company?

```
Hi <Name>,

Thanks for reaching out for assistance!

We understand that you're encountering problem <X> which is blocking your work. Thanks so much for providing the steps to reproduce this problem! I'll see if I can reproduce this problem here, and will be in touch.

Regards,
<Friendly Support Person>
```

Hmmmm... What's wrong here?

- The person asking for help is politely acknowledged and greeted.
- The problem is restarted for clarity, including acknowledging the severity of the issue.
- The reproducible testcase is acknowledged.
- Next steps are specified.
- There's a friendly closing.

Can't put your finger on it? There's no timeline associated with the next steps!

This seems pretty minor, but let's flesh this scenario out a little bit, and let's say that I'm the person asking for assistance. I've clearly run into a problem and it's significant enough that I've taken the time to send in a request for assistance because I can't progress on my work. It's wonderful that I've reached someone willing to help, and that they understand the problem at a high enough level to paraphrase it, but now I'm left with absolutely no understanding of when I'll hear back.

You may be reading this and think that the next steps are solely with the person troubleshooting the problem on the Support side, but you'd be partially wrong. With no firm timeline established, the affected user who submitted the ticket also has to proactively reach out to Support if they need an update, since they don't know when they'll hear back. This makes for a poor customer experience since they're left wondering if they're going to hear back in an hour, a day, a week, or never.

All to often I'll see this scenario play out, even with industry veterans, and almost without fail the very next entry in the ticket (regardless of timeline) is from the customer saying "Any updates?".

Let's do a small tweak, and outline the impact it has:

```
Hi <Name>,

Thanks for reaching out for assistance!

We understand that you're encountering problem <X> which is blocking your work. Thanks so much for providing the steps to reproduce this problem! I'll see if I can reproduce this problem here, and will provide you with an update by close of business tomorrow. 

Regards,
<Friendly Support Person>
```

All we've done here is provide a timeline of getting a response by the end of the following day, but let's dive into what that entails:

- The customer now knows when they'll receive next contact, so they know they don't need to follow up before then unless something changes.
- This gives the customer a clear path to update the ticket owner on changing priorities. For example, is this actually blocking their whole team and not just them? Is there a business critical function that needs to happen by the end of the same day, meaning tomorrow is too late?
- The ticket owner now has an obligation to address the issue on the timeline they've specified, meaning they can better prioritize, instead of just throwing the issue into their (potentially large) backlog.
 
## Resetting Expectations
In a perfect world, the severity of the problem in the example above was well communicated and ticket owner would provide a material update before the close of business the following day. Great, case closed (or at least notably progressed), but we all know that's not always what happens.

Let's talk about resetting expectations that were set.

Things change. The customer who submitted that example ticket isn't the only person having problems. Let's assume for a moment that the ticket owner got hit with a much more important problem - like another customer being completely down and blocked - and they haven't had time to address our example problem.

There are two likely scenarios, and outcomes, of this.

### Scenario 1 -  The ticket does not receive an update before close of business the following day

This is the most common scenario I see from less experienced folks. Unfortunately, it tends to drive a far more negative outcome than you may expect at face value. Fortunately, it's a very easy problem to address!

Having dealt with customers as a front-line customer support person, an escalation point, a team lead, a main point-of-contact, and as a direct customer advocate, for almost 14 years I can quite comfortably tell you that the #1 cause of customer frustration is not receiving timely updates.

In this case, the customer is left without any information as to the status of their request and feels ignored. It's a safe bet that (assuming they haven't given up) that ticket owner is going to come in the next morning and see "Any updates?" front and center on that ticket. The customer won't be happy.

### Scenario 2 - The ticket is updated before the close of business the following day saying that there's no progress

Prior to the end of the following day, the ticket owner sends off this update:

```
Hi <Name>,

I just wanted to send you an update to note that I'm still looking into this here. I'll be in touch ASAP with next steps.

Regards,
<Friendly Support Person>
```

We're making progress with this, but still not perfect! Let's take a look at Scenario 3 for how we can reliable salvage this.

### Scenario 3 - The ticket is updated, and expectations are reset

All we're going to do here is a minor tweak to the wording above. This will let us reset expectations with the customer and keep the next steps entirely on the Support side (save for a possible escalation).

```
Hi <Name>,

I just wanted to send you an update to note that I'm still looking into this here. I'll be looking into this first thing tomorrow morning and will provide you with an update by mid-day tomorrow.

Regards,
<Friendly Support Person>
```

The customer has been informed of the delay, understands the prioritization moving forward, and knows to expect an update at the discussed time. This, in general, will keep them happy!
 
## Further breakdown
Obviously there's nuance to this, but as discussed above this is the most common cause for ticket escalations and overall poor customer experience. Overall length of time to (successfully) close a ticket is a distant second in terms of drivers for negative customer satisfaction behind a lack of communication.

While this scenario can be intimidating, I'll also not that you're significantly less likely to get directly negative feedback from a customer when providing a wrong answer, as long as it's done in a timely fashion and leaves the door open for them to follow-up with you.

# Professional Written Communication
I'll start out by noting that there's a difference between formal communication and professional communication. A lot of this will come down to the company that you're working with, and the "voice" that they are trying to present. For the sake of this section, we're not discussing formal writing as that's it's own skill in many cases.

To start out with, this is not intended to "shame" anyone who is not a native speaker of whatever language they are communicating in. English, in particular, is a disaster of a language and (in my opinion) it's reasonably ignorant to judge someone based on their grasp of a non-native language. If you fall into this bucket, I'd strongly recommend the use of a Linter to help catch spelling and grammatical issues, such as linter-write-good for Atom.

With that out of the way, the high-level things we want to note here are:

- Organization of thoughts
- Clear and concise wording
- Proper punctuation, spelling, and grammar

## Organizing your Thoughts 

Issues can be complex, and responses can necessitate jumping between multiple logical scenarios. When you find yourself in this scenario, it can be very helpful to include things like headers (for example, the "Organizing your Thoughts" header here) to more clearly define context.

Similarly, it can be helpful to have a very high level summary, sometimes using bullet points or a numbered list, to specifically outline the information that will be further discussed... as we've also done here.

Clear and Concise Wording

Time is valuable. If you can clearly, and politely, communicate a thought in two sentences instead of two paragraphs, do it. I'll provide examples of good, and bad, responses below. We'll strip the pleasantries since we're just focusing on content.

### Example 1

```
I can see that in function findTheThing you've used a brute-force search across the array to find duplicate values. This type of approach is quite inefficient with large data sets.

You may want to consider loading values into a set for evaluation as this will likely perform much better.
```


### Example 2

```
 I can see that in function findTheThing you've used a brute-force search across the array to find duplicate values. While a brute-force search is simple to implement, and will always find a solution if it exists, its cost is proportional to the number of candidate solutions – which in many practical problems tends to grow very quickly as the size of the problem increases 

You may want to consider loading values into a set for evaluation as this will likely perform much better. A set will allow you to only store the required values until a duplicate is found, which will likely result in significantly smaller memory utilization.
```

I'll note that both of these are completely valid responses that most customers would be happy to receive. In order to keep you, the reader of this article, paying attention I've limited the verbosity that could have otherwise been included in `Example 2`.

At the end of the day here, you'll need to know your audience. In the example above, if it's safe to assume that your reader knows what a set is from a computer science perspective there's probably no need to explain how it should be implemented, or how it will differ from a brute-force search of an entire array.

## Punctuation, Spelling, and Grammar

I'm not going to dive into whether or not you should use Oxford commas, or try to note that you need to ensure you have sentence structure that your 10th grade English teacher would give you an A+ on. The goal here is to be legible, avoid ambiguity, and avoid reader frustration. 

- Use paragraphs to logically divide thoughts.
- Avoid run-on sentences.
- Know the difference between your/you're, its/it's, there/their/they're, etc.
  - Again, a Linter will help here if you're not a native English speaker.
- Proof-read your messages.

I'll paraphrase a message I received recently which I think is a prime example of how not to write something.

```
Hi Logan,

Hope you are doing well,

I see you have a problem and I would be more than happy to help you fix this problem that you described in your original message.

Please can you send me the following information - what version you are using, what operating system you are using for example Windows or Mac, and what browser? These information's will help to resolve your issue more quickly so it would be very helpful if you can provide them quickly thanks!

Regards,
<Name>
```

I actually cringed when reading the message that inspired this... and while writing this. Can I understand what they need? Absolutely. Did I have to re-read this a few times to understand? You bet.

As a "fun" exercise, try rewriting this in the comments below and see if you can clean it up!

# Wrapping up
I know that this all seems very basic, and it is, but it's the kind of thing where practice makes perfect. It's also very easy to get into bad habits like not proof-reading messages before sending or not providing enough information in responses. Experience certainly helps.

There are many other concepts we could discuss, and maybe they'll be done so in another post, but I'll leave you all with one final recommendation:

If you are providing someone with technical instructions, be sure to test the instructions first or specifically outline if they have not been tested. Nothing is worse than being on the receiving end of something that doesn't work at all when dealing with an "expert". Mistakes certainly happen, but this one can generally be avoided pretty easily.
