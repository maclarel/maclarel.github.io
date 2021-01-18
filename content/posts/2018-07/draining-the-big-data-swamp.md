+++
title = "Draining the (Big Data) Swamp"
tags = [
    "bigdata",
    "bestpractices"
]
date = "2018-07-24"
toc = true
+++

Before we dive too far into this article, let's define a few key terms that will come up at several points:

- `Big Data` - Technology relating to the storage, management, and utilization of "Big Data" (e.g. enormous amounts of data/petabyte scale).
- `Data Lake` - A common term to refer to a storage platform for Big Data. In this case, let's assume Apache Hadoop (HDFS) or Amazon S3.
- `Data Warehouse` - A large store of data sourced from a variety of sources within a company, then used to guide management or business decisions.
- `ETL` - Extract, Transform, Load. The process of taking data from one source, making changes to it, and loading it into another location. The underpinning of Master Data Management, and basically all data movement.
- `Master Data Management (MDM)` - The concept of merging important data into a single point of reference. For a lot of Big Data applications, this means tying data back to a single person or actor.

# Now with that out of the way... why on earth is the name of this article talking about a Data Swamp? 

Coming from the RDBMS/data warehouse world - data normalization and MDM is a critical part of any project involving data. Generally there will be some source system that is feeding data in that is then "cleaned" (or normalized) to reduce duplication/inconsistencies and is then fed into a data warehouse which is used for reporting.

Let's say that you're handling data coming in from two ubiquitous Microsoft platforms - Active Directory and Exchange. Both of these have relatively consistent logging formats to track access, or at least to track behaviour, but where things get different quickly is who they say is accessing things.

With Active Directory logs (let's just say evtx format for the sake of argument), you'll see the account name as `DOMAIN\User`, or in some cases `user@domain.com`. With Exchange, however, you're going to see data coming in under the user's email address or, if there's no authentication configured, whatever email address was specified as the sender... in some cases this is going to be `user@domain.com` however frequently it can also be `firstname.lastname@domain.com` or any other infinite number of ways that an organization chooses to reference email addresses.

To put this into a more real world example, at a company I used to work with my login for the domain was in the format of a country code and employee ID, e.g. `CA123456`, so AD would track me under that ID and only that ID. At that same company I had two email addresses - `tdn@company.com`, and `thedatanewbie@company.com` (short name and long name). See where this gets confusing!?

# OK, now I understand that data is a mess... What's your point?

Now picture yourself on the receiving end of those logs and trying to piece this all together into a working scenario where you can say that I logged in at 8AM at my normal office place and send out fifteen emails to thirty recipients... That's where MDM comes into play!

Despite the seemingly strange approach, there's usually a method to the madness in this sort of thing. Generally all of these items are referenced together in some place... generally an Active Directory user profile or some LDAP based equivalent. Easy to use, right!? Maybe...

Attributes in an LDAP profile can be public, meaning anyone who can do a lookup on the user can see them, or they can be private which means that you have to be authorized to see them. In a lot of cases the public attributes are only going to be group memberships, a real name, and the user name - this leaves a lot out to dry.

Let's assume, though, that we have privileged access and can now read all the attributes - we've got our source of truth! Fantastic! Let's ignore the logistics of trying to maintain up to date LDAP lookups on demand for tens of thousands of users, let alone hundreds of thousands, and move on... 

At this point a consulting team that gets paid lots of money is going to implement some sort of ETL pipeline that pulls data from the source system, does enrichment to tie this all to the "one true version" of a user, and then loads it into the Data Warehouse which will be used for reporting.


# This all sounds great! We've identified how to enrich the data (ignoring the logistics of doing so), and now you've got a Data Warehouse you can report off of!

Absolutely, it is great, but here is where we get back to the title of this article! Everything I just described is extremely common in the relational database world (e.g. Oracle, MS SQL Server, etc...) for reporting with products like Cognos, Qlik, etc... but unfortunately it is a concept that seems to be routinely ignored in the Big Data world.

As noted earlier, a lot of people will take these logs from many places and store them in their Data Lake for future use/analysis/etc... While it's great that they get stored, what this means is that you've got likely petabytes worth of data that you're paying to store but are not using for anything and could not effectively use for anything that would not be extremely resource intensive to implement.

# This is why Big Data implementations fail!

Big Data is expensive, and its complex... it's the perfect storm of hard to do! With this said, all of the same concepts we talked about earlier still apply, they just have to be done differently! Running data through an ETL tool designed for an RDBMS that maybe operates on a total of 24 CPU cores and 128GB of RAM isn't going to cut it when you're dealing with enterprise levels of log files.

This is where I say it's expensive - not only do you need to have the expertise and knowledge of which tooling to use and how to use it, you need be willing to pay to make it happen. In a lot of cases for an enterprise handling authentication logs, email logs, endpoint logs, firewall logs, etc... this is going to be manifesting its self in the many dozens of CPU cores and hundreds of gigabytes of memory just to do the normalization. You can quickly see why there's hesitation to do this - who wants to spend upwards of half a million dollars on something that's just changing a user name in a log file!?

Unfortunately, without spending that money, those logs that are sitting in S3 (despite it being really cheap) are not really going to be useful if you want to use them down the road. This is where you wind up with the eponymous Data Swamp - a very murky lake full of data.

Many Data Lakes get set up as part of security initiatives so that there can be a postmortem on a breach, or even used proactively in the world of UEBA and Security Analytics, but without this MDM work (and in many cases even basic normalization) that company is going to wind up with the following challenges, to name a few:
Weeks or months of time spent identifying the characteristics and "shapes" of the data that is stored in the data lake.
The same cost that would have been incurred originally to now perform that MDM work (but under a time crunch).
Added costs and stress of trying to expedite an implementation of something to analyze the logs, or to have their security team(s) manually review them in a SIEM style tool like Splunk.
Really, it just circles back to what could have been done in the first place, and what would have allowed that company to have proactive tools in place, but even past that do immediate manual analysis or rules-based analysis of the data that was stored without having to incur all of the additional costs, stress, and time taken to do so after the fact.

# You're painting a pretty dire picture... Are all Big Data implementations like this?

NO! Thank goodness they are not, but the ones that fail all seem to have the same characteristics.

- No willingness to spend (usually this turns into all of this data living in S3 because it's cheap).
- Limited forethought into the end use case (e.g. SIEM, Security Analytics, etc...).
- No dedicated team for the platform, and thus no one to enable the end use case.

The ones that are successful, however, tend to have done the following:

- Brought in, or hired, the expertise to handle the data movement and transformation, and continue to engage with those teams if they are not permanent.
- Implement(ed) the Data Lake with a specific goal in mind, and identified the work required to make that happen.
- Engaged with, or at least consulted, vendors that would enable the use case once the data was available. 

# Let's wrap this up.

Is this technically easy? Absolutely not. It's not easy even in the RDBMS world where a few dozen gigabytes of sales data across 500 retails stores is considered a huge volume of data to report off of. What I'm saying at the end of this is that conceptually it's something that seems to get missed in a lot of Big Data implementations, and there are a lot of folks out there who have been blasting Big Data/Hadoop implementations for years despite the fact that a lot of those issues can be tied back to this at the end of the day.

If this struck a chord with you, and it's something you're having trouble with, keep an eye on this blog and the associated YouTube channel. We're going to be taking a look at a bunch of technology that enables this whole thing to work - Kafka, NiFi, Elasticsearch, Hadoop, etc... there's no shortage of platforms to let you do this work, and even choosing the "wrong" one generally won't hamstring you too much, but you have to make that leap to get started!
