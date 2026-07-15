+++
title = "Roboleaks: Copilot leaking private issues in public"
tags = [
    "security",
    "career",
    "opensource"
] 
date = "2026-07-15"
toc = true
+++

# The bug

There was some serendipity in finding this, and it culminated in two disparate issues being identified. When working at GitHub I was one of the program managers for the Bug Bounty program and triaged hundreds of reports - one of those was the issue that is fixed by github.com/github/clipboard-copy-element/pull/87. As you can see in the issue, this actually links back to an issue in a private repository - the entire body of that issue was dumped into initial comment by Copilot (it has since been scrubbed by GitHub staff after me flagging it) which resulted in me getting a notification about it.

This led to identification of the aforementioned issues:

1. Most interesting - Copilot will leak the entire content of the root comment of whatever issue it was asked to fix, even if that content is private and the fix target is public.
2. The fix introduced in github.com/github/clipboard-copy-element/pull/87 was woefully incomplete and I was able to identify over a dozen variants that were still exploitable.

# Usage patterns

Many companies maintain private repositories containing their IP and other sensitive information (e.g. bugs/security issues/etc) and public libraries that they either maintain or have forked for their usage. GitHub, for example, has many public repositories that are actively used as dependencies for their products. Normal operations there are to address _security-related_ bug fixes for those in **private** forks of those repositories and have the fixes ready for either offline merges or rapid movement through PRs with minimal fanfare - historically you could see this in [Backup Utils](https://github.com/github/backup-utils) where security issues were generally included with very generic text under "Bug fixes".

# Attacker value

Admittedly this report is a bit messy since it's really two separate issues, so I'll break them down separately.

## Copilot leaks

This one isn't exploitable by an adversary - hence the blog post. This is something that teams need to be aware of with a mixed private/public presence like GitHub has. It becomes easy to accidentally leak sensitive content from a private repository in a public setting. While in most cases this may just be functional bugs, as we saw here it was actually a security issue that uncovered more security issues.

## Copy/paste discrepancies

This one is a bit more "real" and was directly exploitable. In short, non-printable characters are valid in filenames on many platforms (e.g. macOS, Linux) and are also valid in a git context. Since these didn't get flagged in GitHub's UI, nor were they obviously visible when viewed in other contexts, it was possible to have a copyable code block (e.g. [Homebrew's installation instructions](https://github.com/Homebrew/install)) that actually run `install.sh[invisible_character_here]` rather than `install.sh` when executed.

This was predicated on a fair bit of social engineering since you need to get someone to actually go to your malicious repo and run the command without looking at the repo content so it's a decidedly low severity bug.

# What you should be aware of

The main thing to look for in this case is the Copilot scenario - review PRs in your public repos that were created by `Copilot` and contain references to private repos in their body. This is, of course, only relevant if you have an operational model that would have private/public overlap.

If you _do_ have a model like this, I strongly recommend staging fixes in private forks of the public repository and then doing an offline merge (or PR with approving parties ready to act) to minimize exploitable window. 

Also, of course, keep in mind that AI isn't a replacement for critical thought. In the case of [the issue that spawned all of this](github.com/github/clipboard-copy-element/pull/87) there was no variant analysis done and as a result the "fix" is so narrow that it left over a dozen variants publicly exploitable and unaddressed.

# Disclosure & Timelines
- June 15, 2026 - Reported to GitHub Bug Bounty
- June 30, 2026 - GitHub closes the report as not having significant security risk, providing a $200 "thank you" award
- June 30, 2026 - Disclosure requested
- July 14, 2026 - GitHub confirms that the unicode-related issues have been resolved
- July 15, 2026 - Disclosure granted, with the the following note regarding the reported Copilot functionality:

> This behavior is expected given how the coding agent operates: when an agent task is assigned, the agent works from the context it is provided and surfaces that context (issue references, rationale, links) into its output, such as the PR description and task activity. There is no cross-repo access boundary being crossed here; the agent does not gain visibility into private repositories on its own. The private details appeared publicly because the agent was assigned to perform work in a public repository, where its inputs and output are visible outside the repo.
>
> Importantly, when you assign the agent to an issue targeting a public repository, we surface an explicit warning at assignment time that the issue description will be visible to users outside that repository. So the exposure is a documented and user-acknowledged consequence of assigning the agent to a public repo, rather than a silent data leak or an access-control bypass. For that reason, this as expected agent behavior rather than a separate product vulnerability.
>
> That said, we agree the outcome is not ideal for security-sensitive work, and we are evaluating feature improvements in this area, but don't have anything to announce at the moment.
