+++
title = "Don't be like CISA - use Tuvok"
tags = [
    "security",
    "career",
    "opensource"
]
date = "2026-06-10"
toc = true
+++

# Let's start with a horror story

You're a large organization and you rely on a lot of contractors. Sometimes those contractors are not the smartest, or think they're smart when they really aren't. Hell, maybe they're actually employees. Either way, they perceive security controls as a nuisance to be worked around instead of actually protecting them, or they just can't figure out how to actually use the tools they're given.

Maybe they've accidentally pushed your IP up to a public repository, or maybe they've done what we saw at CISA a few weeks ago _and pushed a huge pile of credentials/secrets to a public repository_ rather than storing them somewhere secure.

Believe me when I say that from many years of experience on the inside of GitHub, this is a weekly (if not daily) occurrence with many organizations. Often they'll catch it, but sometimes, like with CISA, it lingers for _far too long_.

# WTF is Tuvok?

While I'm sure there's commercial EASM tooling that covers some of this, why pay for something that's all available from public APIs and a little OSINT? In steps Tuvok (nee Garak, but it turns out Nvidia has done something with that... too many Star Trek fans).

[Tuvok](https://github.com/maclarel/tuvok) is a tool that will use a fine-grained PAT (WIP to run a GitHub App) to retrieve _the names_ of all internal/private repositories and all organization members. From there it will scan the _public_ repository and gists of those organization members for anything that matches the name of an internal/private repository or a list of user-defined keywords (e.g. "CISA"/"cisa", "secrets", etc.) in its name or README content. For matches found, it will then run Trufflehog against that repository to try to flag any potential secrets contained therein. 

After that's all done, it'll generate a report for human review, and can then be further tuned with an allowlist to negate false-positive findings/known scenarios.

# What does it need?

- Python 3.11+ and uv
- trufflehog v3 on $PATH (brew install trufflesecurity/trufflehog/trufflehog)
- A fine-grained GitHub PAT scoped to the target org with:
    - Org permissions: Members: read
    - Repo permissions: Metadata: read, Contents: read

That's it. Minimal risk to the organization, and the potential to very quickly catch possible leaks.

# Limitations

This relies on a PAT at the time of writing, so for a larger organization it will very likely run into rate limiting issues. Once converted to a GitHub App this should roughly 3x the throughput that it can handle, but it may be valuable to use the `--users-file` functionality to split your userbase into more manageable chunks. For an organization of a few hundred employees you'll likely be fine with the PAT, but for an enterprise you'll want to further break this down.

# PRs welcome

Find something that sucks? Open a PR. Happy to collab, and AI submissions are welcome as long as they can be backed up with legitimate reasoning. 

Don't be CISA, keep an eye on what's out there. Accidents happen, but so does abject stupidity. Tuvok can help to prevent both and costs you literally nothing.
