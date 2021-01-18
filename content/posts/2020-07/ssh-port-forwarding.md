+++
title = "SSH Port Forwarding"
tags = [
    "linux",
    "ssh",
    "tutorial"
]
date = "2020-07-19"
toc = true
+++

Have a service that you don't want to directly expose, or that doesn't have good security, that you still need to access? Today we're going to look at using SSH to accomplish this!

This tutorial uses `nc`, `UFW/IPTables`, and `SSH` to demonstrate how to forward any port over an SSH connection on TCP 22. You can extend this to just about any service. For example, this works for accessing websites, or other services that are exposed through a web browser.

[![SSH Port Forwarding](http://img.youtube.com/vi/hb80XpmLJ1U/0.jpg)](https://www.youtube.com/watch?v=hb80XpmLJ1U)
