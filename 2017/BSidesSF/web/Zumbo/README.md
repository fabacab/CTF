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

![Screenshot of "Zumbo Dot Com" homepage.](screenshots/zumbo-dot-com-homepage.png)

The only interesting thing on the website is a a counter, which reports that "This site has been visited *some number* times." Refreshing the web page increments the counter, and the counter always seems to count up. This is a good indication that there is some server-side code running on the website keeping track of the number of hits.

The first thing we need to do is explore the website to see how this counter, or other parts of the site, might be implemented. In the most basic case, we can simply [use any decent Web browser's "View source" features](http://www.computerhope.com/issues/ch000746.htm) to do this.

* [View the Zumbo Dot Com homepage's HTML source](loot/http_zumbo-8ac445b1.ctf.bsidessf.net_index.template.html)

Reading the source of the HTML page, we see a bunch of JavaScript and CSS. These are both technologies that run inside of the browser (the client), so aren't going to help us figure out how the counter works. However, at the very end of the HTML source we see an HTML comment:

```html
<!-- page: index.template, src: /code/server.py -->
```

This could be anything, but one thing stands out: the `index.template` portion is the same as the URL in our Web browser. Indeed, accessing the Zumbo Dot Com homepage *redirected* us to the page at `/index.template`. The comment seems to indicate that the file at `/code/server.py` is producing this output (because `src` is a common shorthand for "source"). We can test that assumption simply by trying to access a bunch of different, unlinked URLs on the site.

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

Success! This gives us another plain-text page, whose contents is in fact the `server.py` script. Viewing [the source of that page](loot/http_zumbo-8ac445b1.ctf.bsidessf.net_server.py) reveals the same kind of comment at the end:

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

The import thing about this comment is that the `page:` being reported is `flag`, not `/flag`, the latter of which is the one we want. This is happening because the current working directory of the server is actually inside the `/code` directory on the server, so when we ask for `/flag` in the URL, we are actually asking for `/code/flag` on its filesystem, which is not a file that exists. We need to go up a level in its filesystem, and we need to use the URL to do so.

Filesystem paths have two special directories, one called `.` (a single dot) which means "the current directory" and another called `..` (two dots) which means "the parent directory." Since our flag file is in the *parent* of the server's current directory (and we, as Web site visitors, are in the server's context), we need to ask the server to go up a directory level first. We do this by asking for `../flag` rather than simply asking for `flag`.

This would make our URL [http://zumbo-8ac445b1.ctf.bsidessf.net/../flag](http://zumbo-8ac445b1.ctf.bsidessf.net/../flag). Unfortuantely, asking for that directly in a browser's address bar (depending on one's browser), usually removes the `../` part. To get around this, we can URL-encode that portion of the address in order to instruct the browser to send a *literal* dot-dot-slash as part of the URL.

URL encoding is simply a syntax for encoding literal characters as part of a URL. It is also known by the term [percent encoding](https://en.wikipedia.org/wiki/Percent-encoding), because each encoded sequence begins with a `%` character. Following the `%` character is a hexadecimal integer that maps to a [UTF-8](https://en.wikipedia.org/wiki/UTF-8) encoded [code point](https://en.wikipedia.org/wiki/Code_point). In UTF-8, a dot (`.`) is at code point `0x2e` (hexadecimal 2E), and a forward-slash (`/`) is at code point `0x2f` (hexadecimal 2F).

The URL path we want to access is `../flag`, so percent-encoding this becomes `%2e%2e%2fflag`. Now, we can access the URL at [http://zumbo-8ac445b1.ctf.bsidessf.net/%2e%2e%2fflag](http://zumbo-8ac445b1.ctf.bsidessf.net/%2e%2e%2fflag) in our browser and we are greeted with the contents of the `/flag` file. Viewing source, we see the flag and the familiar comment, with the expected `page:` value:

```html
FLAG: RUNNER_ON_SECOND_BASE

<!-- page: ../flag, src: /code/server.py -->
```

This technique of navigating around a filesystem in ways that the application developer did not defend against is called a *[path traversal attack](https://en.wikipedia.org/wiki/Directory_traversal_attack)*.

## Challenge 3

> And the final stage, with real hacking included!
> 
> Welcome to ZUMBOCOM....you can do anything at ZUMBOCOM.
> 
> Three flags await. Can you find them?
> 
> http://zumbo-8ac445b1.ctf.bsidessf.net

## Solution 3

> :book: I did not solve this challenge during the CTF. :( After the CTF, I read others' writeups but did not find them optimally educational. So I'm including a narrative writeup of a solution anyway, in accordance with the purpose of this repository. 

The final part of the Zumbo challenge teases us by saying this is "the final stage, with real hacking included!" For this final challenge, we need an intimate understanding of Python. But to begin to figure it out, we just continue on the same path as we were on from parts one and two. In this case, that means going back to the `server.py` script and reading it more closely.

The clue to the third flag is clearly visible in the Flask application's source code:

```python
   flag3 = requests.get('http://vault:8080/flag').text
```

As before, we need some programming experience to understand what's happening here. A new variable, called `flag3`, is being initialized. It's a safe bet that the result of the program code that sets this variable is going to give us the third flag. The program code setting this variable is `requests.get('http://vault:8080/flag').text`, so we need to figure out what `requests` is, what its `get` method does, and what its `text` property means (although these are all pretty semantically meaningful).

At the very top of the `server.py` script is this line:

```python
import requests
```

So [`requests` is a Python library ("module")](http://python-requests.org/), a popular one used for [making HTTP requests and receiving responses](http://www.pythonforbeginners.com/requests/using-requests-in-python). (This is instantly discoverable with a simple Internet search for `python requests` or similar keywords.) Using its `get()` method is simply a programmtic way of loading a page, just like you might do in a Web browser. Its `text` property is simply a text-only representation of the response body.

Knowing that, it becomes clear that the third flag is to be found on yet another Web server, the called `vault`. The challenging part is that we don't know what `vault` is. All we know is that the target webserver serving us the Zumbo website knows what `vault` and that when *it* loads the page at `http://vault:8080/flag`, it receives the flag.

> :sweat_drops: During the CTF, my first instinct was to leverage the path traversal attack to try reading a file that would contain information about what the server thought the `vault` name referenced. Following that logic, I asked for [the `/etc/hosts` file](http://zumbo-8ac445b1.ctf.bsidessf.net/%2e%2e%2fetc%2fhosts). My hope was that the name `vault` was bound to a publicly-accessible IP address or other name as per the [standard file-based Linux name lookup mechanism for this file described in `hosts(5)`](https://linux.die.net/man/5/hosts). This [proved a dead-end](loot/http_zumbo-8ac445b1.ctf.bsidessf.net_%252e%252e%252fetc%252fhosts.html), however.
> 
> Unconvinced that I had exhausted the utility of the directory traversal attack vector, I tried accessing log files that I thought might contain information about a system called `vault`. This included the `/var/log/wtmp` and `/var/log/btmp` log files, which ordinarily contain information about users who authenticate to the system (i.e., users who log in). My hope was to be able to download this file and then use [`last -d -f downloaded-wtmp-file | grep vault`](http://explainshell.com/explain?cmd=last+-d+-f+downloaded-wtmp-file+%7C+grep+vault) to see if I could find an IP address associated with the "`vault`" name. Unfortunately, these were either not present or not readable.

At this point, there are two possible avenues of accessing the third flag:

* somehow read the variable `flag3` directly, or
* somehow execute our own request in the context of the server.

The first option would require a code path in the same Python block as the variable. That block again looks like this:

```py
if __name__ == '__main__':
    flag1 = 'FLAG: FIRST_FLAG_WASNT_HARD'
    with open('/flag') as f:
        flag2 = f.read()
    flag3 = requests.get('http://vault:8080/flag').text

    print "Ready set go!"
    sys.stdout.flush()
    app.run(host="0.0.0.0")
```

It's apparent that there are no opportunities to insert our own code after the line with `flag3` here since there are no places where this code block uses any user-supplied input. Therefore, our only remaining option is to see if we can execute code on the remote server, i.e., if we can find a [remote code execution (RCE)](https://en.wikipedia.org/wiki/Arbitrary_code_execution) vulnerability. To do this, we need to again re-examine the source code, this time looking for any and all places where we can supply input to the program to influence its behavior during execution.

This program is relatively small. The only place where user-supplied input is accepted is the URL. The relevant code block is this one:

```py
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
```

In this code, the value of the URL becomes the value of the `page` variable, so let's manually trace what this code does with `page`.

1. First, `page` is checked against the value `'favicon.ico'`. If it is equal to that string (i.e., if the URL we entered was `http://zumbo-8ac445b1.ctf.bsidessf.net/favicon.ico`), then we would `return` from the function, terminating execution. That's a dead-end for us. Let's move on.
1. The next line that uses the `page` variable is inside the `try` block. In this case, we `open()` a file given by the URL, `read()` its contents, and assign the contents of the file to the `template` variable. Let's follow this code path more closely:
    1. The next line where `template` is used in this code path is the second to last one, where the now-familiar HTML comment is being appended to a new line: `template += "\n<!-- page: %s, src: %s -->\n" % (page, __file__)`. (This line also makes use of the `page` variable, revealing [an XSS attack](https://en.wikipedia.org/wiki/Cross-site_scripting), but this is unhelpful to us because XSS attacks are executing on the client, in our Web browser, and we are trying to achieve *remote* code execution, on the server.) Moving on…
    1. …the last line in the function uses `template` again, this time passing it to [Flask's `render_template_string()` method](http://flask.pocoo.org/docs/0.12/api/#flask.render_template_string). This tells us that if we can control the contents of the `template` variable, we can get Flask to render whatever we want. This is useful because Flask's renderer is a templating language called [Jinja](http://jinja.pocoo.org/), and [one of the features of Jinja templates is the ability to execute at least some Python code](http://jinja.pocoo.org/docs/2.9/templates/#list-of-global-functions). Unfortunately, in this code execution path we have no control over the contents of `template`, because its contents were read from a file on disk in step 2, so this is another dead-end.

Back in step 2, the `page` variable instructs the code which file to try to `open()`. This is nested inside a `try` block to make sure that if the `open()` call fails for some reason, the program can handle the error. Since the `page` variable is controlled by *us* (it is user-supplied input), we can force the `open()` call to fail by supplying a URL of a file that we know does not exist. We've already encountered URLs like this in the very first part of the challenge, when we were greeted with a message like "No such file or directory."

Let's now explore that code path, and see what happens when we send a URL to a non-existent path. Let's try using `blahblahblah` as the `page` variable (by loading the address `http://zumbo-8ac445b1.ctf.bsidessf.net/blahlbahlbah`).

1. Once again, we supply a URL. This becomes the `page` variable in the code.
1. We know that we must *not* supply a URL of `favicon.ico`, because if we did the first line of the program would `return`, and execution would end. Since `blahblahlblah` does not equal `favicon.ico`, we continue executing.
1. Next, the code will `try` to `open()` the file called `blahblahblah`. This will *fail*, because no such file exists.
1. At this point the code exits the `try` block and jumps to the `except` block, setting the variable `e` to the Python error (which [Python calls an "exception"](https://wiki.python.org/moin/HandlingExceptions)). The interesting variable to us is now called `e`, not `page`.
1. The very next line uses the `e` variable by passing it through [the `str()` function](https://docs.python.org/2/library/functions.html#str). This function simply turns the error into a plain-text string. That string is then set to the `template` variable. This means if we can control the error, and we *can* control the error because the error is defined by the URL, which is something *we* provide to the program, then we can control the *template* that's rendered, like we saw earlier.

So that's our code path. The challenge now is how to write a URL that will meet the following criteria:

* The URL must not be a file that exists, or the string `favicon.ico`.
* The URL must be interpreted by the Jinja templating engine as executable Python code.

The Jinja documentation isn't quite so plainly worded about how to do this, but they do provide the information we need right near the top in [their *Synopsis* section](http://jinja.pocoo.org/docs/2.9/templates/#synopsis):

> The default Jinja delimiters are configured as follows:
> 
> * {% ... %} for [Statements](http://jinja.pocoo.org/docs/2.9/templates/#list-of-control-structures)
> * {{ ... }} for [Expressions](http://jinja.pocoo.org/docs/2.9/templates/#expressions) to print to the template output
> * {# ... #} for [Comments](http://jinja.pocoo.org/docs/2.9/templates/#comments) not included in the template output
> * #  ... ## for [Line Statements](http://jinja.pocoo.org/docs/2.9/templates/#line-statements)

Recall that our objective is to write a string that is both not a file name and is also a Python expression that will be evaluated by the Jinja template renderer. The only item in this list that matches what we need is the second, Jinja expressions. Its "expressions" are delimeted by a double brace on each side, so that means whatever our URL is, it will have to start with `{{` and end with `}}`.

Let's test our understanding of this. In Python, if we write the expression `1 + 1`, the response we should see is `2`. (You can prove this to yourself by running [`python -c 'print(1 + 1)'`](http://explainshell.com/explain?cmd=python+-c+%27print%281+%2B+1%29%27) at a command line.) In Jinja's syntax, this would be `{{ 1 + 1 }}`. So let's try [using that as the URL](http://zumbo-8ac445b1.ctf.bsidessf.net/%7B%7B 1 + 1 %7D%7D). Sure enough we're greeted with the correct answer:

```html
[Errno 2] No such file or directory: u'2'
<!-- page: 2, src: /code/server.py -->
```
