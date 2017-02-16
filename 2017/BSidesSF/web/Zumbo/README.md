# Zumbo

The Zumbo challenge is a three-part challenge.

## Challenge 1

> Welcome to ZUMBOCOM....you can do anything at ZUMBOCOM.
> 
> Three flags await. Can you find them?
> 
> http://zumbo-8ac445b1.ctf.bsidessf.net

## Solution 1

In this challenge, we are presented with the homepage of a simple website calling itself "Zumbo Dot Com."

![Screenshot of "Zumbo Dot Com" homepage.](zumbo-dot-com-homepage.png)

The only interesting thing on the website is a a counter, which reports that "This site has been visited *some number* times." Refreshing the web page increments the counter, and the counter always seems to count up. This is a good indication that there is some server-side code running on the website keeping track of the number of hits.

The first thing we need to do is explore the website to see how this counter, or other parts of the site, might be implemented. In the most basic case, we can simply [use any decent Web browser's "View source" features](http://www.computerhope.com/issues/ch000746.htm) to do this.

* [View the Zumbo Dot Com homepage's HTML source](http_zumbo-8ac445b1.ctf.bsidessf.net_index.template.html)

Reading the source of the HTML page, we see a bunch of JavaScript and CSS. These are both technologies that run inside of the browser (the client), so aren't going to help us figure out how the counter works. However, at the very end of the HTML source we see an HTML comment:

```html
<!-- page: index.template, src: /code/server.py -->
```

This could be anything, but one thing stands out: the `index.template` portion is the same as the URL in our Web browser. Indeed, accessing the Zumbo Dot Com homepage *redirected* us to the page at `/index.template`. The comment seems to indicate that the file at `/code/server.py` is producing this output. We can test that assumption simply by trying to access a bunch of different, unlinked URLs on the site.

The first one we can try is the path listed in the code comment: `/code/server.py`. To do so, we just replace `index.template` in our browser's address bar with `code/server.py`. When we access [http://zumbo-8ac445b1.ctf.bsidessf.net/code/server.py](http://zumbo-8ac445b1.ctf.bsidessf.net/code/server.py), we're greeted with another, plaintext page that reads simply:

```
[Errno 2] No such file or directory: u'code/server.py'
```

We can again examine the source code. Doing so reveals a similar HTML comment as before, but with the `page:` portion changed:

```
[Errno 2] No such file or directory: u'code/server.py'
<!-- page: code/server.py, src: /code/server.py -->
```

So it appears that whatever we put into the address bar of the site is echoed ("reflected") into this part of the HTML comment. More interesting, however, is that the `src` part has *not* changed. This could mean that whatever is at the `/code/server.py` URL, which is likely a Python script (identifiable by the `.py` ending) is serving these files. This is an assumption, but makes sense because the format of the error message shown on screen matches that of a basic Python error. The giveaway is the end: `u'code/server.py'` is [how Python denotes Unicode-encoded strings](https://docs.python.org/2/howto/unicode.html).

We want to get at this file to get more information about how the website is constructed, but asking for `/code/server.py` did not work. The "No such file or directory" error indicates that we have asked for the wrong URL. So, next, let's simply try `server.py` by loading [http://zumbo-8ac445b1.ctf.bsidessf.net/server.py](http://zumbo-8ac445b1.ctf.bsidessf.net/server.py) into our browser.

Success! This gives us another plain-text page, whose contents is in fact the `server.py` script. Viewing the source of this page reveals the same kind of comment at the end:

```py
import flask, sys, os
import requests

app = flask.Flask(__name__)
counter = 12345672


@app.route('/<path:page>')
def custom_page(page):
    if page == 'favicon.ico': return ''
        global counter
        counter += 1
    try:
        template = open(page).read()
    except Exception as e:
        template = str(e)
    template += "\n<!-- page: %s, src: %s -->\n" % (page, __file__)
    return flask.render_template_string(template, name='test', counter=counter);

@app.route('/')
def home():
    return flask.redirect('/index.template');

if __name__ == '__main__':
    flag1 = 'FLAG: FIRST_FLAG_WASNT_HARD'
    with open('/flag') as f:
        flag2 = f.read()
    flag3 = requests.get('http://vault:8080/flag').text

    print "Ready set go!"
    sys.stdout.flush()
    app.run(host="0.0.0.0")

<!-- page: server.py, src: /code/server.py -->
```

Reading this source code, it's clear that the `server.py` file is a Python application that uses [the Flask Web-serving microframework](http://flask.pocoo.org/) to generate HTML pages.

Moreover, the source code here reveals the flag, which is set to a variable called `flag1`:

```python
    flag1 = 'FLAG: FIRST_FLAG_WASNT_HARD'
```

Indeed, with a little knowledge of how Web servers map URLs onto files in a filesystem, the first flag wasn't hard. :)

## Challenge 2

> Welcome to ZUMBOCOM....you can do anything at ZUMBOCOM.
> 
> Three flags await. Can you find them?
> 
> http://zumbo-8ac445b1.ctf.bsidessf.net

## Solution 2

For the second part of the Zumbo challenge, we begin by inspecting the source code of the `server.py` file again. There is clearly another variable, this time called `flag2` in a Python block in the source code:

```py
    with open('/flag') as f:
        flag2 = f.read()
```

With some programming experience, you can easily deduce that this snippet opens a file (located at `/flag`) and then reads its contents into a variable. If you didn't know that, you could search [the Python documentation for the `open()` built-in function](https://docs.python.org/2/library/functions.html#open). In any event, it seems that the second flag is going to be in the file on the server located at `/flag` on its filesystem.

The first thing we can try is simply accessing the `flag` URL at [http://zumbo-8ac445b1.ctf.bsidessf.net/flag](http://zumbo-8ac445b1.ctf.bsidessf.net/flag), however this fails with the familiar "No such file or directory" error. Viewing the source of this page, we can again see the familiar comment:

```html
<!-- page: flag, src: /code/server.py -->
```

The import thing about this comment is that the `page:` being reported is `flag`, not `/flag`, the latter of which is the one we want. This is happening because the *root* of the server is actually inside the `/code` directory on the server, so when we ask for `/flag` in the URL, we are actually asking for `/code/flag` in the filesystem, which is not a file that exists. We need to go up a level in the filesystem, and we need to use the URL to do so.

Filesystem paths have two special directories, one called `.` (a single dot) which means "the current directory" and another called `..` (two dots) which means "the parent directory." Since our flag file is in the *parent* of the server's directory (and we, as Web site visitors, are in the server's context), we need to ask the server to go up a directory level first. We do this by asking for `../flag` rather than simply asking for `flag`.

This would make our URL [http://zumbo-8ac445b1.ctf.bsidessf.net/../flag](http://zumbo-8ac445b1.ctf.bsidessf.net/../flag). Unfortuantely, asking for that directly in a browser's address bar (depending on one's browser), usually removes the `../` part. To get around this, we can URL-encode that portion of the address in order to instruct the browser to send a *literal* dot-dot-slash as part of the URL.

URL encoding is simply a syntax for encoding literal characters as part of a URL. It is also known by the term [percent encoding](https://en.wikipedia.org/wiki/Percent-encoding), because each encoded sequence begins with a `%` character. Following the `%` character is a hexadecimal integer that maps to a [UTF-8](https://en.wikipedia.org/wiki/UTF-8) encoded [code point](https://en.wikipedia.org/wiki/Code_point). In UTF-8, a dot (`.`) is at code point `0x2e` (hexadecimal 2E), and a forward-slash (`/`) is at code point `0x2f` (hexadecimal 2F).

The URL path we want to access is `../flag`, so percent-encoding this becomes `%2e%2e%2fflag`. Now, we can access the URL at [http://zumbo-8ac445b1.ctf.bsidessf.net/%2e%2e%2fflag](http://zumbo-8ac445b1.ctf.bsidessf.net/%2e%2e%2fflag) in our browser and we are greeted with the contents of the `/flag` file. Viewing source, we see the flag and the familiar comment, with the expected `page:` value:

```html
FLAG: RUNNER_ON_SECOND_BASE

<!-- page: ../flag, src: /code/server.py -->
```

This technique of navigating around a filesystem in ways that the application developer did not defend against is called a *[path traversal attack](https://en.wikipedia.org/wiki/Directory_traversal_attack)*.
