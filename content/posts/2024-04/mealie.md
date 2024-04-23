+++
title = "Cooking up some more CVEs"
tags = [
    "security",
    "career"
]
date = "2024-04-09"
toc = true
+++

# Unlike state secrets, I don't want my brisket recipes to wind up in a Florida bathroom

As a follow up to [the last time I wrote about CVEs](https://www.maclaren.dev/posts/2023-12/2023/#i-found-some-vulnerabilities-and-got-some-cves-credited-to-me), I've amassed a few more to my name:

- CVE-2024-29191
- CVE-2024-29192
- CVE-2024-29193
- CVE-2024-31991
- CVE-2024-31992
- CVE-2024-31993
- CVE-2024-31994

The first three of these are related to `go2rtc` and are follow-ups from the earlier investigation done with [Frigate](https://github.blog/2023-12-13-securing-our-home-labs-frigate-code-review/). These were found in collaboration with [Jorge Rosillo](https://github.com/jorgectf) from GitHub Security Lab, and I recommend taking a read through [the advisory article](https://securitylab.github.com/advisories/GHSL-2023-205_GHSL-2023-207_go2rtc/) for details on these.

In this article, I'll focus on the findings related to [Mealie](https://mealie.io/). While these are lower severity than those that I/we found in Frigate & go2rtc, they ecompass Denial of Service (DoS), Server Side Request Forgery (SSRF), and local file inclusion.

# What did I find?

I want to start off by reassuring the Mealie users out there that the following mitigations are/were feasible to mitigate these findings regardless of the version of Mealie in use:

- If your instance is only exposed to your intranet, you're already reasonably secure
- If your instance is exposed to the internet, but segregated from your internal network, the SSRF findings do not affect you
- If neither of the above are the case, but you disabled signups _and_ you disabled the default account (enabled by default), you're reasonably secure

If you're not interested in the details, these were mostly addressed in [this Pull Request](https://github.com/mealie-recipes/mealie/pull/3368) and version 1.4.0 (and above) have far more secure defaults than in the past.

The very quick summary of these findings is that a low-privilege user (i.e. generic user account) with access to Mealie is able to:

- Map and retrieve content from any webserver on any network Mealie has access to (LFI/SSRF - [CVSS 5.0](https://nvd.nist.gov/vuln-metrics/cvss/v3-calculator?vector=AV:N/AC:L/PR:L/UI:N/S:C/C:H/I:N/A:N/E:F/RL:O/RC:C/CR:L/IR:X/AR:X/MAV:N/MAC:L/MPR:L/MUI:N/MS:C/MC:H/MI:N/MA:N&version=3.1))
  - Note: Extraction of content retrieval required administrator access or Debug mode.
- Perform DoS attacks against the Mealie server (DoS - [CVSS 3.8](https://nvd.nist.gov/vuln-metrics/cvss/v3-calculator?vector=AV:N/AC:L/PR:L/UI:N/S:C/C:N/I:N/A:L/E:F/RL:O/RC:C/CR:X/IR:X/AR:L/MAV:N/MAC:L/MPR:L/MUI:N/MS:C/MC:N/MI:N/MA:L&version=3.1))
- Perform DoS attacks against remote web servers due to lack of rate limiting (DoS - [CVSS 4.7](https://nvd.nist.gov/vuln-metrics/cvss/v3-calculator?vector=AV:N/AC:L/PR:L/UI:N/S:C/C:N/I:N/A:H/E:F/RL:O/RC:C/CR:X/IR:X/AR:M/MAV:N/MAC:L/MPR:L/MUI:N/MS:C/MC:N/MI:N/MA:L&version=3.1))

Note that the DoS vectors here do not permit any sort of amplication, and are purely volumetric in nature. Without critical mass of compromised Mealie servers, the impact is primarily focused on the Mealie servers themselves.

# Timeline

- Decemeber 15, 2023 - Findings are privately disclosed to Mealie maintainers [as per their instructions](https://github.com/mealie-recipes/mealie?tab=security-ov-file#reporting-a-vulnerability) with a noted disclosure date of March 14th, 2024.
- January 30, 2024 - An update is requested from the Mealie team.
- March 14, 2024 - An [issue](https://github.com/mealie-recipes/mealie/issues/3315) is opened in the [Mealie repository](https://github.com/mealie-recipes/mealie) requesting an update to avoid disclosure of actively exploitable vulnerabilities.
- March 15, 2024 - Mealie maintainers respond and request extension of disclosure timeline to March 25th.
- March 25, 2024 - An update is requested from the Mealie team.
- April 3rd, 2024 - Pull Request [3368](https://github.com/mealie-recipes/mealie/pull/3368) is merged, and version 1.4.0 is released containing fixes.
- April 9th,  2024 - Public disclosure of these findings & CVE assignment.

# What should I do if I'm running Mealie?

- Put it behind an authentication proxy and/or VPN - do not have it publicly exposed to the internet.
  - [Recommendations from the Mealie team](https://docs.mealie.io/documentation/getting-started/installation/security/)
- Restrict network access to your internal network. While the latest updates restrict the ability to interact with RFC1918 IP addresses, DNS rebinding is *likely* still viable.
- Disable signups and the default account unless you have a very good reason to keep these enabled.
- Do *not* run in a [development setting](https://docs.mealie.io/contributors/developers-guide/starting-dev-server/) for normal usage.
- Update to version 1.4.0 or above.

# What are the details?

Rather than reposting a repost of my original reports, you can read through it all on the [GitHub Security Lab Advisories site](https://securitylab.github.com/advisories/GHSL-2023-225_GHSL-2023-226_Mealie/).

# Any themes in the findings?

The very quick summary here is that if you're going to allow users to interact with external resources, you need to be very careful about what you allow them to do. In the case of Mealie, the ability to interact with external resources was a feature, but the lack of rate limiting and the ability to interact with internal resources opened this up to be used as not only a DoS vector (granted, one with limited impact) but also allowed an attacker to map out internal resources.

While neither of these are all that scary compared to some other things that I've found (like those in [Frigate](https://securitylab.github.com/advisories/GHSL-2023-190_Frigate/), they're all tools that can be used as part of a larger attack. In this case, the SSRF could be used to map out internal resources and the DoS could be used against those resources, Mealie itself, or against an external target.

A lot of folks with homelabs, myself included, are a bit lazy about resource limits on things like docker containers. This is a good reminder that we should be a bit more careful about what we allow our containers to do and where they can connect. A Mealie instance with a multi-gigabit connection and (far too many) CPU cores could be used to amplify a DoS attack against a target, and that's not something that we want to be a part of.

When you're self-hosting any service, be conscious of who has access to it, _how it can be accessed_ (just because you have auth turned on doesn't mean there isn't a bypass), and what _that service itself_ can access. Network segregation is extremely important *especially* if you're exposing services publicly, even behind a proxy.

# Closing thoughts

I've got two takeaways from this experience.

First off, kudos to the Mealie maintainers for acknowledging and addressing these findings despite being lower in severity. There have been some notable communication gaps, but when we have been able to connect they've been great to work with. You can find more details about their thoughts [in the release notes for version 1.4.0](https://github.com/mealie-recipes/mealie/releases/tag/v1.4.0). I'm supportive of the approach they've taken to address these - through preventing access to RFC 1918 addresses, and documenting recommendations for rate limiting. These aren't durable mitigations from a product perspective (external DNS/DNS rebinding can bypass the IP restrictions) but when paired with availability of information, more secure defaults, and deployment recommendations I believe these are sufficient.

Secondly, I think the timelines and communication failures around these findings really emphasize the value of formal reporting mechanisms for open source projects like GitLab's [Confidential Issues](https://docs.gitlab.com/ee/user/project/issues/confidential_issues.html) and GitHub's [Private Vulnerability Reports](https://docs.github.com/en/code-security/security-advisories/guidance-on-reporting-and-writing-information-about-vulnerabilities/privately-reporting-a-security-vulnerability). Email is archaic, and incredibly easy to miss. It's telling that the first response I received from an otherwise active team of maintainers only came after I opened an issue in the GitHub repository asking for an update.

To be extremely clear, the above statement is not attempting to speak negatively of Mealie's maintainers. The intent is to demonstrate the unreliability of email for anything requiring attention. As a maintainer, please make use of the tools freely available to you... and feel free to reach out to me if you want a run down on how to use GitHub's Private Vulnerability Reports!
