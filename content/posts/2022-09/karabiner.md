+++
title = "Karabiner is a game changer"
tags = [
    "keyboard",
    "tech"
]
date = "2022-09-04"
toc = true
+++

# What is Karabiner and why?

[Karabiner Elements](https://github.com/pqrs-org/Karabiner-Elements) is an open source MacOS application that can be used to intercept and modify any keystroke, combination, etc... on your machine and change it to do damn near anything you want. If you follow my blog or know me, you'll know that I'm a big fan of custom keyboards and the flexibility that they allow by way of QMK or similar firmware - Karabiner recreates a lot of that _functionality_ at the software level.

# What's the feature set?

Broadly, Karabiner Elements will let you intercept any keystroke and have it perform an arbitrary task for you. This could consist of changing a press of your Escape key to running a shell script instead, or noting that you want your spacebar to send a blank space character if tapped, but to toggle Caps Lock if held. 

On custom keyboards you can do a lot of this through firmware like QMK, which also means that it's portable (without software) to any machine you use, however if you find yourself working from a laptop keyboard and really missing Hyper/Meh/Layers/autoshift/etc... then you're in for a real treat with this software

One of the great things about Karabiner is that the configuration is dotfile driven, so you can sync your configs quite easily via git and symlinks (possibly another post about my dotfiles strategy later...), but I always prefer software that's config file driven rather than relying solely on `plist` files. 

# What am I using it for?

While I make use of QMK pretty extensively, it's not to the extent of folks like [Ben Vallack](https://www.youtube.com/c/BenVallack). None the less, I make extensive use of Hyper/Meh (various combinations of Meta/Alt/Ctrl/Shift) to control my window manager ([Amethyst](https://github.com/ianyh/Amethyst)) and virtual desktops. Additionally, I try to reduce chording (multi-keypress combinations) whenever reasonable - to that end I set all of my number row and symbol keys (e.g. 1, 2, 3, `,`, `.`, `[`) to enter their normal character if tapped, and their shifted value if held (e.g. `1` on tap, `!` if held).

Honestly, Karabiner has been a game changer for me. I've gone from dreading using my laptop's keyboard to only mildly disliking it. Given that I'm starting to travel a bit more again, that's a huge win in my books!

# Resources

If you're interested in playing around with Karabiner, you can check out the following resources:
- [The Karabiner subreddit](https://www.reddit.com/r/Karabiner/)
- [The Karabiner docs](https://karabiner-elements.pqrs.org/docs/)
- [My dotfiles](https://github.com/maclarel/dotfiles/blob/main/.config/karabiner/karabiner.json)
