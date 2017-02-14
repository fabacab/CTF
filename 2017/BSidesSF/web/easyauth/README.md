# easyauth

# Challenge

> Can you gain admin access to this site?
> 
> http://easyauth-afee0e67.ctf.bsidessf.net
> 
> * [easyauth.php](easyauth.php)

# Solution

This challenge description links to a web page and provides one attachment, a PHP script that (presumably) contains part of the code running the website.

The first thing to do is to access the website linked in the description with a normal web browser. We simply click the link. We're presented with a login page on which there is a simple form. The form asks for a username and password, as is typical of most web login forms. The login page also contains a visible note telling us *not* to bruteforce this page, and to try logging in using the username `guest` and the password `guest`.

When we use the suggested credentials, we're presented with another page that informs us we are logged in and showing us that our web browser has saved a cookie.


