# Prudentialv2

## Challenge

> We fixed our challenge from two years ago.
> 
> [http://54.202.82.13/](http://54.202.82.13/)

## Solution

This challenge presents us with a webpage whose only contents is a typical "Login" form. The form has two fields, one marked "Name" and the other "Password" just as you might expect to see anywhere else on the Web. Presumably, we need to enter the correct values into the fields to acquire the flag.

Rather than spending time making random guesses, which would almost certainly be a waste, the very first thing we should do is get some more information about how the page is constructed. This is most easily accomplished with [any decent Web browser's "View source" features](http://www.computerhope.com/issues/ch000746.htm). Reading the source of the page shows a typical HTML document, with a linked stylesheet and the form we saw:

```html
<html>
<head>
	<title>level1</title>
	<link rel='stylesheet' href='style.css' type='text/css'>
</head>
<body>


<section class="login">
	<div class="title">
		<a href="./index.txt">Level 1</a>
	</div>

	<form method="get">
		<input type="text" required name="name" placeholder="Name"/><br/>
		<input type="text" required name="password" placeholder="Password" /><br/>
		<input type="submit"/>
	</form>
</section>
</body>
</html>
```

One line in particular stands out: there is a hyperlink (created with [the `<a>` HTML element](https://developer.mozilla.org/docs/Web/HTML/Element/a)) to a file called `index.txt`. We can simply load that page in our Web browser to have a look at it.

* [View the `./index.txt` file.](loot/http_54.202.82.13_index.txt)

This file is almost identical to the source code we just viewed. Since its name is `index.txt`, and its contents is almost exactly the same as the first web page we loaded, a reasonable guess is that this is in fact the *same* source code as the other page. The fact that we can see the source itself, rather than its evaluated output, is almost certainly caused by the fact that the file extension has been changed to `.txt`. This tells the server to treat the file as "plain text" rather than executable code, and so it gives us the full file contents itself.

There is one one difference between the contents of this page and the previous one: there is an additional block of code near the middle of the file right after the opening `<body>` tag. Its code looks like this:

```php
<?php
require 'flag.php';

if (isset($_GET['name']) and isset($_GET['password'])) {
    $name = (string)$_GET['name'];
    $password = (string)$_GET['password'];

    if ($name == $password) {
        print 'Your password can not be your name.';
    } else if (sha1($name) === sha1($password)) {
      die('Flag: '.$flag);
    } else {
        print '<p class="alert">Invalid password.</p>';
    }
}
?>
```

This is an embedded [PHP](https://php.net/) code snippet and is presumably the code being evaluated and executed on the first page. You need some familiarity with PHP to understand what this code is doing, but there isn't much  of it, so even if you don't have PHP experience it's not too hard to look up each and every command in the [PHP documentation](https://php.net/docs.php). Let's examine each one briefly by manually reading the code and referring to the PHP documentation to learn what each line is doing.

The first line of code is simply [the PHP opening tag](https://secure.php.net/manual/language.basic-syntax.phptags.php) itself (`<?php`), and the last line is the PHP closing tag (`?>`), "which tell PHP to start and stop interpreting the code between them."

The very next line uses the [`require` statement](https://php.net/require), whose documentation says it "is identical to [`include`](https://php.net/include)". Following the link to `include`, we read that it "includes and evaluates the specified file." In this case, the "specified file" is `flag.php`. Since the file we are examining is `index.txt`, and we have assumed that this is the same code running at the website's homepage (because most servers are configured to use files named either `index.html` or `index.php` as a default), we can surmise that `flag.php` here refers to the file accessible at [http://54.202.82.13/flag.php](http://54.202.82.13/flag.php). Unfortunately, loading that page reveals a blank screen; it wasn't a "Not Found" error, so the file is probably actually there, but it doesn't *output* anything useful either.

Moving on, the next line is [an `if` condition](https://php.net/if) using [the `isset()` function](https://php.net/isset) twice, once on the `$_GET['name']` variable and the second time on the `$_GET['password']` variable. These variable names (`name` and `password`) match the login form's two fields. The `$_GET` part is a [PHP "superglobal"](https://secure.php.net/manual/language.variables.superglobals.php), which "are built-in variables that are always available in all scopes." Even more specifically, [the `$_GET` superglobal](https://secure.php.net/manual/reserved.variables.get.php) in particular is "an associative array of variables passed to the current script via the URL parameters." So this line simply checks if both a `name` and a `password` parameter was included in the query string portion of the URL used to access this page.

We can easily test this assumption by typing arbitrary values into the login form. Sure enough, if we submit the form with the values `testname` and `testpassword`, our browser loads the URL `http://54.202.82.13/?name=testname&password=testpassword`. Interestingly, we are also greeted with the message "Invalid password." This message exists in the source code, which we'll examine more closely in just a second.

Returning to the PHP code we're reading, we can see the next two lines are setting two PHP variables, one called `$name` and one called `$password`, both taken from the URL via the PHP superglobal variable:

```php
    $name = (string)$_GET['name'];
    $password = (string)$_GET['password'];
```

The important thing to note about these lines is that both variables are [*type cast*](https://secure.php.net/manual/language.types.type-juggling.php#language.types.typecasting) to `string`s. This simply forces the values of the URL to be interpreted by PHP as string values. In other words, if we entered `1` as the "name" in the form field, this type cast ensures that the name is treated as the PHP literal value `'1'` (notice the single quotes), rather than as the numeric integer value `1`. Same with the "password" field. The purpose of doing this is to prevent [type confusion (or "type juggling") attacks](https://web.archive.org/web/20160310134410/https://www.owasp.org/images/6/6b/PHPMagicTricks-TypeJuggling.pdf); seeing this code alerts us to the fact that this avenue of attack is (probably) already sealed off.

Next, the code checks whether the `$name` variable, set earlier, "is equal to" the `$password` variable, also set earlier:

```php
    if ($name == $password) {
        print 'Your password can not be your name.';
    }
```

If the variables are determined to be equal, the code prints "Your password can not be your name." Again, we can test our understanding of this easily enough: just enter the same value in the login form's "name" field as the value for its "password" field. Using `test` for both values and submitting the form confirms this result.

The next conditional is of primary interest because it is clearly dealing with the flag:

```php
    } else if (sha1($name) === sha1($password)) {
      die('Flag: '.$flag);
    }
```

Here we see another PHP function used twice: [the `sha1()` function](https://php.net/sha1). The documentation says this function is used to "calculate the sha1 hash of a string." If the result of this function applied to the `$name` variable is *identical* (because of the triple-equals, `===`, meaning not merely "equal" but identical) to the result of this same function applied to the `$password` variable, then the code will [`die()`](https://php.net/die), which exits the script while outputing the given message. In this case, the message is the flag.

So we now know that our challenge is to provide input through the login form that meets the following criteria:

* the "name" and "password" fields must not be equal, and
* the two different values entered must result in an identical value after having the `sha1()` function applied to them.

The PHP manual simply says that this function "calculates the sha1 hash of a string." This code therefore tests whether the [SHA-1](https://en.wikipedia.org/wiki/SHA-1) hash of the `$name` and `$password` variable are identical and only outputs the flag if they are. (This is not really close to how normal "login" mechanisms work, so this *isn't* actually a login form. It's just a cryptography riddle that requires some knowledge of PHP-backed Web pages to attempt.)

> :beginner: If you've never encountered a "hash" in this context before, this challenge will seem incredibly complex. Searching the Internet for "hash" probably reveals more pages about cannabis than computer security at first, but adding "computer" to your searches will get you [many](http://www.webopedia.com/TERM/H/hashing.html), [many](http://unixwiz.net/techtips/iguide-crypto-hashes.html), [many](https://en.wikipedia.org/wiki/Hash_function) more [relevant results](http://www.computerhope.com/jargon/h/hashing.htm). Another tip for beginners, especially if you've found the regular Wikipedia article impenetrable, is to check if there is a "simple" version of the Wikipedia page. In this case, you're in luck: the [Simple Wikipedia article for "Cryptographic hash function"](https://simple.wikipedia.org/wiki/Cryptographic_hash_function) is relatively straightforward by comparison.
> 
> In the end, a "hash" in the context of computer security is simply a value that ("cryptographically") represents some other value. The idea, in theory, is that two different original values will never be represented by the same two ultimate values after they have been "hashed." In practice, however, weaknesses in cryptographic hash functions (whether by design due to flaws in their algorithm, or by mistakes introduced through their actual implementation) sometimes result in two different values being hashed to the same value. When this happens, the hash function is said to be "cryptographically broken," because if you can hash two different values and get the same resulting value, then there is little point to the hash from a security perspective in the first place.

It's now clear that the point of this challenge is to provide two *different* inputs whose SHA1 hash values are identical. But since the whole point of a hash function is to produce *different* outputs for different inputs, how are we to do this?

If the programmer of this code were a little less careful, it might have been possible to trick their code into hashing two identical *inputs* (which would, of course, produce two identical outputs). However, as we read, they were in fact careful about their variable types (by explicitly type casting the input variables to be `string`s) and about their equality checks (by using the `===` operator to check for exact sameness, not mere equivalence). The only remaining option is to use a [collision attack](https://en.wikipedia.org/wiki/Collision_attack). That is, we must provide two different inputs that nevertheless produce the same hash value as their output. This, of course, is exactly what cryptographic hash algorithms are designed to make difficult.

To understand exactly how difficult, though, it's worth taking some time to further understand just a little bit of what a hash algorithm actually *does*. (I am not a cryptographer and even if I was, a full dissection of cryptographic hash functions is beyond the scope of this document, so I'm going to focus merely on the perspective of an end-user.) The SHA-1 algorithm always produces 160 bits of output, regardles of how much input it was given. You can see this yourself by using [the `shasum(1)` command](https://linux.die.net/man/1/shasum) available on most UNIX-like systems, or with the [File Checksum Integrity Verifier](https://www.microsoft.com/en-us/download/details.aspx?id=11533) application provided by Microsoft for Windows systems:

```sh
$ for input in "test" "othertest"; do echo -n "$input" | shasum --algorithm 1; done;
a94a8fe5ccb19ba61c4c0873d391e987982fbbd3  -
3fcab50babaf08fc09223d11ff2b6bf3a4ee5c27  -
```

In the above example, two inputs of different length are provided. The first input (`test`) is only four characters long, and produces hexadecimal output that is forty characters long. The second input (`othertest`) is nine characters long, but also produces hexadecimal output that is forty characters long. A single hexadecimal character can be one of sixteen possible values (`0` through `9`, which is ten possibilities, or the values `a` through `f`, which is an additional six, totalling sixteen). This means that each character position shown above offers sixteen bits (mathematically, 16<sup>1</sup> = 16). Stringing forty sixteen-bit characters one after the other produces a number that is equal to [16<sup>40</sup>](https://www.wolframalpha.com/input/?i=16%5E40). That number is 1,461,501,637,330,902,918,203,684,832,716,283,019,655,932,542,976, which is the total number of possible outputs the SHA-1 algorithm can produce. I don't even know how to pronounce that number verbally.

Taken at face value, this is a daunting task. At the time of this writing, contemporary consumer-grade computers are simply not fast enough to search that many possibilities for two inputs whose outputs are both the same resulting value when run through the SHA-1 algorithm. A cluster of supercomputers could probably do it in some amount of time, but even they might need much more time than the duration of the Capture The Flag game (which only lasts a mere 48 hours), meaning that a brute-force search is impractical, at least with respect to acquiring the flag from this challenge. We have to use some *much* faster way to find a hash collision in the SHA-1 algorithm.

The fastest way to do this, of course, is to ask the Internet if anyone else has *already* done this. A simple Internet search for [`SHA-1 hash collision`](https://duckduckgo.com/?q=sha-1 hash collision) turns up numerous hits. Indeed, [a SHA-1 hash collision *has* been found and published](https://security.googleblog.com/2017/02/announcing-first-sha1-collision.html). On February 23<sup>rd</sup>, 2017, Google published a post stating:

> we are [releasing two PDFs](https://shattered.it/) that have identical SHA-1 hashes but different content.

Since the challenge's PHP code does not check that its input matches a *specific* hash, only that the result of two hashing operations are the same as each other, these PDFs are exactly what we need: two different inputs that have identical SHA-1 hashes. In this case, the inputs are PDF files, `shattered-1.pdf` and `shattered-2.pdf`. Both produce the SHA-1 hash value `38762cf7f55934b34d179ae6a4c80cadccbb7f0a`. We can use one PDF for the form field's "name" parameter so that the code's call to `sha1($name)` produces the aforementioned hash, and the other PDF for the form field's "password" parameter, which will make the call to `sha1($password)` return an identical result.

Let's first verify that this does, in fact, actually work. We can download the PDFs and run `shasum` over them as we did with our test input using [Bash brace expansion](https://www.gnu.org/software/bash/manual/html_node/Brace-Expansion.html) to save some typing:

```sh
$ shasum --algorithm 1 shattered-{1,2}.pdf
38762cf7f55934b34d179ae6a4c80cadccbb7f0a  shattered-1.pdf
38762cf7f55934b34d179ae6a4c80cadccbb7f0a  shattered-2.pdf
```

As a sanity check, we can use a different algorithm to make sure that these files are in fact not the same file, or more simply use [the `diff` command](https://linux.die.net/man/1/diff) to check for differences:

```sh
$ shasum --algorithm 256 shattered-{1,2}.pdf
2bb787a73e37352f92383abe7e2902936d1059ad9f1ba6daaa9c1e58ee6970d0  shattered-1.pdf
d4488775d29bdef7993367d541064dbdda50d383f89f0aa13a6ff2e0894ba5ff  shattered-2.pdf
$ diff shattered-{1,2}.pdf
Binary files shattered-1.pdf and shattered-2.pdf differ
```

Great, we have two identical SHA-1 hash values despite having two files whose contents are different, as evidenced by the SHA-256 hash values and as reported by `diff`.

The next step is to take these PDF files and provide them as input to the challenge's PHP code. A Web browser's user interface doesn't actually allow us to place a *file* into a textual form field, so this isn't going to be as simple as filling in the form. Of course, we don't actually care about the form's specifics at all, since it's the *result* of submitting the form field that we actually care about. That result is simply accessing a URL with a query string. Of course, a Web browser doesn't exactly let us put a PDF file into the address bar, either, so we'll need some other way of taking the file contents in the PDFs and translating them into a URL.

A further complication is the fact that the PDF files are, well, files. In contrast, a URL is text. We somehow need a way to represent a file's raw content in the form of a URL. The answer to this conundrum is a special URL syntax called URL-encoding, or sometimes called [percent-encoding](https://en.wikipedia.org/wiki/Percent-encoding), because each encoded sequence begins with a percent (`%`) character. Following the `%` character is a pair of hexadecimal digits signifying the value of an 8-bit byte. In other words, putting `%41` into any part of a URL is interpreted to mean "a byte value of `0x41`, in hex." (Hexadecimal numbers are often distinguished from other base digits with the `0x` prefix.) Using the [ASCII](https://en.wikipedia.org/wiki/Ascii) and UTF-8 [character encoding](https://en.wikipedia.org/wiki/Character_encoding) standards, this maps to the capital letter `A`. Similarly, inserting `%20` into any part of a URL ultimately maps to a literal `SP` ("single space") in these same character sets. Of course, nothing is limiting us to entering values that only map to characters we can type with a standard keyboard layout. [The percent-encoding scheme allows us to represent arbitrary binary data](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_arbitrary_data) in the same way.

Let's take a closer look at the PDF files to understand how we might do this.

If we opened a PDF file in a PDF viewer, we would see a graphical representation of the binary data in the file after it is interpreted by the PDF viewer application itself. That's not what we want, because we have no way of translating that back into individual byte sequences for use in a URL. Instead, we want to see the binary data in the PDF files *before* it is rendered "as a PDF" to our screen.

The tools for doing this are called ["hex editors"](https://en.wikipedia.org/wiki/Hex_editor) or "binary file editors." As the name implies, these are file editors that let us inspect a file's raw data, without any special program interpretation. A similar kind of program is a "hex dump viewer," which simply displays the contents of certain data (like files) in the way a hex editor would (usually referred to as a "[hexdump](https://en.wikipedia.org/wiki/Hex_dump)"), but doesn't offer editing capabilities. These simpler programs will do for us, since we just want to inspect the contents of the PDFs, not change them.

A popular hexdump viewer available by default on most UNIX-like systems is [`xxd(1)`](https://linux.die.net/man/1/xxd). (Windows users have multiple options as well, but they are often not included by default. For example, Microsoft provides a program called [Binary Editor](https://msdn.microsoft.com/en-us/library/cb4x6esf.aspx) in their Visual Studio development suite.) In the simplest form of `xxd`'s operation, we simply provide a file for it to inspect, or feed it some data from a previous command. For example, here's how we can prove to ourselves that the hexadecimal value `41` is, in fact, interpreted as a capital letter `A`:

```sh
$ echo -n 'A' | xxd
0000000: 41                                       A
```

As you can see, we supply the single input character `A` to the `echo` command and [use a pipe](http://tldp.org/HOWTO/Bash-Prog-Intro-HOWTO-4.html) to feed that character to the `xxd` command. The `xxd` program then inspects its input and shows it to us in raw hexadecimal format. The numbers to the left of the colon are "line" numbers (counting as byte offsets, represented in hexadecimal themselves, and starting from `0`). Then we are shown the inspected contents itself, which in this example is the single byte `41` (which, remember, is in hex, not decimal). Finally, way over on the right is an ASCII representation of the raw data. In this case, since we supplied ASCII text as input to the `xxd` program to begin with, we see the same ASCII on the right.

Next, let's use `xxd` to inspect the raw data in the PDF files. Since the files are pretty big (about 413KB each), let's limit `xxd`'s output to the first one-hundred fifty bytes of the `shattered-1.pdf` file:

```sh
$ xxd -l 150 shattered-1.pdf 
0000000: 2550 4446 2d31 2e33 0a25 e2e3 cfd3 0a0a  %PDF-1.3.%......
0000010: 0a31 2030 206f 626a 0a3c 3c2f 5769 6474  .1 0 obj.<</Widt
0000020: 6820 3220 3020 522f 4865 6967 6874 2033  h 2 0 R/Height 3
0000030: 2030 2052 2f54 7970 6520 3420 3020 522f   0 R/Type 4 0 R/
0000040: 5375 6274 7970 6520 3520 3020 522f 4669  Subtype 5 0 R/Fi
0000050: 6c74 6572 2036 2030 2052 2f43 6f6c 6f72  lter 6 0 R/Color
0000060: 5370 6163 6520 3720 3020 522f 4c65 6e67  Space 7 0 R/Leng
0000070: 7468 2038 2030 2052 2f42 6974 7350 6572  th 8 0 R/BitsPer
0000080: 436f 6d70 6f6e 656e 7420 383e 3e0a 7374  Component 8>>.st
0000090: 7265 616d 0aff                           ream..
```

Here we can see the binary values of the PDF data itself. It begins, [as all PDF files do](http://resources.infosecinstitute.com/pdf-file-format-basic-structure/), with a header that declares the version number of the PDF file format the rest of the document conforms to. In this case, it's `%PDF-1.3`, which we can see in the far right column of `xxd`'s output. Notice that as before, this right-most column simply represnts the *same* data as the middle set of columns, but converted into ASCII. We can again prove this to ourselves by taking the very first character of the PDF header (which in this case is an ASCII `%`) and running it through `xxd` as before:

```sh
$ echo -n '%' | xxd
0000000: 25                                       %
```

Sure enough, the hexadecimal value `0x25` maps to an ASCII `%`. We can do the same thing with the next character, the `P` in "PDF" to see what an ASCII-encoded `P`'s value in hex is:

```sh
$ echo -n 'P' | xxd
0000000: 50                                       P
```

So an ASCII `%` is hexadecimal `25` and an ASCII `P` is hexadecimal `50`. These two bytes are the first bytes in every PDF file. Converting this hexdump into a URL's percent-encoded string is now relatively straightforward: we simply insert a `%` character between every two hex numbers in the middle set of columns and remove the spaces. That is to say, the first group of digits, which is hexadecimal `2550`, becomes `%25%50` in a URL, and so on down the line.

Recall that the target code is available at `http://54.202.82.13/` and expects to receive a URL with two query string parameters, `name` and `password`. That means we need to construct a URL that looks like `http://54.202.82.13/?name=<URL-ENCODED-CONTENT-OF-FIRST-PDF>&password=<URL-ENCODED-CONTENT-OF-SECOND-PDF>`, with the two `<URL-ENCODED-CONTENT…>` parts replaced with the actual percent-encoded sequences of the raw binary data from their respective PDF files.

Recall also that the PDF files are pretty large by human standards. They are 416 kilobytes each. That's four hundred sixteen *thousand* ("kilo") bytes, way more than anyone can be expected to encode "by hand." Instead of manually URL-encoding the hexdump, copying its output, and pasting it into a Web browser's address bar (which might not even work anyway; can your clipboard handle that much text?), let's just ditch the Web browser. After all, a Web browser is just one way to make HTTP network requests, and that's all we're trying to do.

Another way to make HTTP requests is to use the [`curl(1)`](https://curl.haxx.se/docs/manpage.html) program, also often available by default on most UNIX-like systems. (It can also be installed on Windows or any other system by [downloading the appropriate file](https://curl.haxx.se/download.html) for your operating system.) This program simply provides a command-line interface to make the same kind of network requests that your Web browser does but without the graphical interface provided by the browser.

In our case, we want to use it to automatically perform the URL-encoding of the PDF files, placing those URL-encoded byte sequences into an appropriate URL, and then making the network request itself. If you've never used `curl` before, you might struggle with its command line options and syntax. A careful reading of its manual page will usually clear up any confusion, but there are *a lot* of options, so there's no substitute for hands-on experimentation. In our case, we want to use its `--header`, `--get`, and `--data-urlencode` options.

> :beginner: One great way to learn how to use `curl` is to take advantage of certain Web browser's developer tools's ["Copy as cURL" feature](https://web.archive.org/web/20160429025159/https://daniel.haxx.se/blog/2015/11/23/copy-as-curl/). This super-handy shortcut makes it easy to construct a `curl` command line invocation that mimics what your Web browser just did. You can use this to learn the semantics of the command line program more gently, without needing to spend hours reading the manual.

The `curl` command we'll use is:

```sh
curl 'http://54.202.82.13/' --header 'Host: 54.202.82.13' --get --data-urlencode name@shattered-1.pdf --data-urlencode password@shattered-2.pdf
```

Breaking this down:

* The first [word](https://www.gnu.org/software/bash/manual/html_node/Word-Splitting.html) (`curl`) invokes the command.
* The second word, i.e., the first [command line argument](https://bash.cyberciti.biz/guide/Shell_command_line_parameters#What_is_a_command_line_argument.3F) (`'http://54.202.82.13/'`), is the base URL to which we are going to send a network request. In this case, that's simply the target URL we've been examining throughout the challenge.
* The third word (`--header`) is a long [option](http://www.tldp.org/LDP/abs/html/options.html) indicating that the next argument will be an HTTP Header.
* That argument is the `Host` [HTTP Request Header](https://en.wikipedia.org/wiki/List_of_HTTP_header_fields#Request_fields) and its actual value, which in this case is the same as the address of the target server.
* The next word (`--get`) is a long option that tells `curl` to use the `GET` [HTTP Request Method](https://en.wikipedia.org/wiki/HTTP#Request_methods), and to treat any of the various `--data` options as query string parameters.
* Following that is another long option (`--data-urlencode`) which instructs `curl` to URL-encode the data referred to by the very next word.
* That very next word is `name@shattered-1.pdf`. This is `curl`-specific syntax that sets the parameter called `name` to be the value of the data read from the file (indicated by the `@`) called `shattered-1.pdf`.
* Finally, we do the same with the second parameter: `--data-urlencode password@shattered-2.pdf`.

Executing this command, however, does not give us what we were hoping for:

```html
$ curl 'http://54.202.82.13/' --header 'Host: 54.202.82.13' --get --data-urlencode name@shattered-1.pdf --data-urlencode password@shattered-2.pdf
<html>
<head><title>414 Request-URI Too Large</title></head>
<body bgcolor="white">
<center><h1>414 Request-URI Too Large</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

Thankfully, this error is extremely clear: the URL was too long ([a "URI" is a superset of a "URL"](https://stackoverflow.com/questions/176264/what-is-the-difference-between-a-uri-a-url-and-a-urn/176274#176274)), and the server refused to process all of it. After all, the URL was larger than `416KB * 2` bytes. That's a *very* long URL. However, we are now presented with a new problem: if we cannot use the full contents of the two PDFs, will we still have two inputs whose SHA-1 hashes are the same? Fortunately for us, the answer is still *yes*. To see why, we need to refer to the actual academic paper published by the researchers who provided the two PDF files we are using.

In §2 of [their paper](https://shattered.it/static/shattered.pdf), headlined "Our contributions," the authors describe their work as "an *identical-prefix collision attack*, where a given prefix *P* is extended with two distinct *near-collision block pairs* such that they collide for any suffix *S*." An "identical-prefix collision attack" simply means two input values that *start with* identical data. (It is a form of [chosen-prefix collision attack](https://blog.cloudflare.com/why-its-harder-to-forge-a-sha-1-certificate-than-it-is-to-find-a-sha-1-collision/#chosenprefixattacks).) "Two distinct near-collision block pairs" simply means that two different values are then appended to this identical prefix. Finally, "such that they collide for any suffix" means that any follow-on data appended to these now different byte sequences will produce the same hash, *regardless of the rest of this appended data*. In other words, we don't need to use the full PDF files. We just need some amount of data from the *start* of both files. Specifically, we need the identical prefix and the near-collision block pairs. We can discard the suffix because it will have zero effect on the hashed output.

How much data from the start of the PDFs do we need? This, too, is answered by their paper. Page 4 of their paper prominently features a hexdump called "Table 2" and is helpfully labelled "Identical prefix of our collision." The hexdump they show is the first 192 bytes of each PDF. We can inspect the first 192 bytes ourselves to confirm that they are indeed identitcal across the two files:

```sh
xxd -l 192 shattered-1.pdf > 1-pdf.192 # save a hexdump from the first PDF to the file 1-pdf.192
xxd -l 192 shattered-2.pdf > 2-pdf.192 # and do the same for the second
diff {1,2}-pdf.192                     # and then show any differences between the two
```

There will be no output from `diff` because the two prefixes are identical, as promised. This also means these bytes alone are not enough to acquire the flag. Since these two byte streams are identical, we should *expect* an identical SHA-1 hash. Moreover, the code in the challenge rejects identical input values for the `$name` and `$password` variables. So, next, we need to append the "near-collision block pairs." These are also helpfully displayed under "Table 1," which also displays (a pair of) hexdumps (one for the first PDF and one for the second). It is on Page 3 and is also helpfully labelled "Colliding message blocks for SHA-1." We can simply count the number of bytes in one of the pairs, which are helpfully grouped as hexadecimal digits in a table: 16 across and 4 down. Two groups of 16 * 4 = 64 * 2 = 128. So the near-collision block pairs are each 128 bytes long.

Putting this together with the identical prefix of 192 bytes gives us a total of 320 bytes (because a 192 byte prefix plus two 64 byte collision block pairs gives a total of 320 bytes). This means we only need the first 320 bytes from each PDF to produce a collision. Naturally, let's test this!

We'll need to create two new files, let's call them `name.dat` and `password.dat`. The `name.dat` file should be the first 320 bytes of the `shattered-1.pdf`, and the `password.dat` file will be the first 320 bytes of the `shattered-2.pdf` file. We know we can see a hexdump of raw file contents using `xxd`, but another useful feature of `xxd` is the ability to do the reverse. That is, it can take a hexdump and convert it into a raw binary file. This means we can take the output of `xxd` and pipe that output back into a second invocation of `xxd` to truncate a file at an exact byte offset.

The `-r` option to `xxd` is what "reverses" its operation:

```sh
# Put the first 320 bytes of shattered-1.pdf into name.dat
xxd -l 320 shattered-1.pdf | xxd -r > name.dat
# Do the same with the second PDF, but put those bytes into password.dat
xxd -l 320 shattered-2.pdf | xxd -r > password.dat
```

Let's confirm that these two files have *different* contents:

```sh
$ diff name.dat password.dat
Binary files name.dat and password.dat differ
```

And now, for the moment of truth, let's compute the SHA-1 hash for these two *different* 320 byte streams:

```sh
$ shasum --algorithm 1 {name,password}.dat
f92d74e3874587aaf443d1db961d4e26dde13e9c  name.dat
f92d74e3874587aaf443d1db961d4e26dde13e9c  password.dat
```

Presto! We have created the shortest possible byte streams with this prefix and these collision blocks whose contents hash to the same SHA-1 output.

> :bulb: If you want to prove this to yourself you could repeat the procedure a second time, but this time take one byte less from each file, and hash those. The hashes will be different:
> 
> ```sh
> $ xxd -l 319 shattered-1.pdf | xxd -r > name.dat
> $ xxd -l 319 shattered-2.pdf | xxd -r > password.dat
> $ diff {name,password}.dat
> Binary files name.dat and password.dat differ
> $ shasum --algorithm 1 {name,password}.dat
> aed0141331c04c118a5eb3823cd567f7f10c5a59  name.dat
> 2e872ce1efb97b5641dbb37b7b88777602418f20  password.dat
> ```
> 
> So as you can see, we need *at least* 320 bytes from each PDF.

Now that we have what are effectively our hash collision payload files, we simply need to rerun the `curl` command from earlier with the new, shorter files in place of the full PDFs:

```sh
$ curl 'http://54.202.82.13/' --header 'Host: 54.202.82.13' --get --data-urlencode name@name.dat --data-urlencode password@password.dat
<html>
<head>
	<title>level1</title>
	<link rel='stylesheet' href='style.css' type='text/css'>
</head>
<body>

Flag: FLAG{AfterThursdayWeHadToReduceThePointValue}
```

For our hard work, we are rewarded with the flag! :D
