Here is an initial attempt to solve part 3 of Zumbo.  As you can see below, I did not solve it.

The hint for the third flag is

```
flag3 = requests.get('http://vault:8080/flag').text
```

Simply typing the URL http://vault:8080/flag gives an error:

>This site can’t be reached
>
>vault’s server DNS address could not be found.

This URL has an interesting structure.  It has no "www", "com", or even any periods.  It also has a colon.  Perhaps I should look up the structure of URLS to see what it means:

Searching found me [this](http://www.skorks.com/2010/05/what-every-developer-should-know-about-urls/), which gives an example URL `ftp://some_user:some_path@blah.com/` and it says that "some_user" is the username and "some_path" is the password.  So maybe "vault" is the username and "8080" is the password.
The one thing missing from our URL is the @ sign, so what if we add it?
[A stackoverflow question](http://stackoverflow.com/questions/10050877/url-username-with) gives the example `http://username:password@www.my_site.com`, so let's try this:
We type `http://vault:8080@zumbo-8ac445b1.ctf.bsidessf.net/flag`, but this just redirects to the home page.  The answers to that stackoverflow question recommend escaping @, but `http://vault:8080%40zumbo-8ac445b1.ctf.bsidessf.net` just turns into a google search for the string "http://vault:8080%40zumbo-8ac445b1.ctf.bsidessf.net".  We can avoid this by following the instructions [here](http://vault:8080%40zumbo-8ac445b1.ctf.bsidessf.net):

>wfaulk said:
>I created a Search Engine named "No", gave it the keyword "null" and set the url to "http://%s".  Then set it to be the default search engine.  This effectively disables search. 
>
>(For those who might not know how to do that, right-click on the URL bar and select "Edit Search Engines".  In the window that pops up, add a new search engine.  The UI seems to vary between OSes, but it's an "Add" button under Linux and an easy-to-miss "+" button at the bottom of the list of search engines under MacOSX.)

However, after doing this it still doesn't work.  The page simply doesn't load when the above URL is typed in.

Maybe the password is "8080/flag", rather than "8080".  However, this turns out not to work either, even when the slash is escaped.

Searching around I found [this manual](https://www.gnu.org/software/wget/manual/html_node/URL-Format.html), which gives another possible meaning for the colon.  It gives the examples:

>http://host[:port]/directory/file
>
>ftp://host[:port]/directory/file

So perhaps "8080" is a "port" and "vault" is a "host".  (I could have figured this out earlier from the initial blog post.)

More searching leads to [this RFC](http://www.faqs.org/rfcs/rfc3986.html).  [RFC](https://en.wikipedia.org/wiki/Request_for_Comments) stands for "Request for Comments", and RFCs were originally responses to requests for comments about the internet, but now the name RFC has stuck for official writing about the standards of the internet.  This RFC gives an example:

```
  The following are two example URIs and their component parts:

         foo://example.com:8042/over/there?name=ferret#nose
         \_/   \______________/\_________/ \_________/ \__/
          |           |            |            |        |
       scheme     authority       path        query   fragment
          |   _____________________|__
         / \ /                        \
         urn:example:animal:ferret:nose
```

So "http" is the scheme, "vault:8080" is the "authority" (which apparently comprises <host>:<port>), and "/flag" is the "path".  This makes it very unlikely that "/flag" was part of the password as I had previously thought.

Later in this RFC, more is said about ports:

>The port subcomponent of authority is designated by an optional port
>   number in decimal following the host and delimited from it by a
>   single colon (":") character.
>
>      port        = *DIGIT
>
>   A scheme may define a default port.  For example, the "http" scheme
>   defines a default port of "80", corresponding to its reserved TCP
>   port number.  The type of port designated by the port number (e.g.,
>   TCP, UDP, SCTP) is defined by the URI scheme.  URI producers and
>   normalizers should omit the port component and its ":" delimiter if
>   port is empty or if its value would be the same as that of the
>   scheme's default.

So "8080" is an alternative port for http, not the default.  Let's look up the possible ports for "http".  I do a google search for "http ports", and the preview for the first result mentions "8080"!

It is an excerpt from the wikipedia article on [Port (computer networking)](https://en.wikipedia.org/wiki/Port_(computer_networking)):

>Port numbers are sometimes seen in web or other uniform resource locators (URLs). By default, HTTP uses port 80 and HTTPS uses port 443, but a URL like http://www.example.com:8080/path/ specifies that the web browser connects instead to port 8080 of the HTTP server.

Next I google "http port 8080" and I get [a page about the 8080 port](https://www.grc.com/port_8080.htm) which says:

>This port is a popular alternative to port 80 for offering web services. "8080" was chosen since it is "two 80's", and also because it is above the restricted well known service port range (ports 1-1023, see below).
>
>Its use in a URL requires an explicit "default port override" to request a web browser to connect to port 8080 rather than the http default of port 80. See the discussion of URL defaults and port overrides on the port 81 page.

At this point I wasn't sure what to do, so I started looking at some other people's solutions.  They all involve writing small python programs, and something called SSTI or Server Side Template Injection.  [This walkthrough and the things it links to](http://www.faqs.org/rfcs/rfc3986.html) seem promising, and I'll probably go through it another time.
