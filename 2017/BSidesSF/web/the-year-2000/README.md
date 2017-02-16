# the-year-2000

## Challenge

> Wait, what year is it?
> 
> http://theyear2000.ctf.bsidessf.net/

## Solution

In this challenge we are linked to a simplistic, static website that claims to have "no flags."

![Screenshot of "the-year-2000" homepage challenge.](screenshots/theyear2000-homepage.png)

Indeed, viewing [the source code of the page](assets/http_theyear2000.ctf.bsidessf.net_.html) similarly reveals nothing much of interest. There are two images, `bg.jpg` and `fire.gif`, but both are in the root folder, so there are no obvious directories to begin searching in. The only clues on the page are the list of tools that the author claims to have used to make the page:

* html
* notepad++
* git
* apache

[HTML](https://en.wikipedia.org/wiki/HTML) is simply the markup language used to construct Web pages, so that's redundant and unhelpful. [Notepad++](https://notepad-plus-plus.org/) is a simple text editor for the Microsoft Windows operating system. This could indicate that the author uses Windows as their main computing platform, but is similarly unhelpful to us here. [Git](https://git-scm.com/) is a popular source control management ("SCM") system (sometimes also called a "version control system (VCS)", and is sometimes used by developers to copy the contents of source code from one server to another, so this could be useful to us. Finally, Apache is the eponymous name of [the Apache HTTP (Web) server](https://httpd.apache.org/) developed by [the Apache Software Foundation](https://apache.org), which may also be useful to us.

The first thing we want to do is get more information about the website itself. We only have two useful clues to go on: it was made with git, and is being served by Apache. To begin, we shouldn't just take the author's word for it (after all, they also said "there are no flags here," and we can assume that is an attempt at misdirection).

To verify that the author indeed used git, we can check for the existence of one or more metadata files that git uses. The most obvious of these is a `.git` directory at the root of a git repository. If the author did indeed use Git during their development process, there might be a `.git` folder somewhere on the website. Since we don't know what other directories exist on this website, we should just check the current one. This means we simply access [http://theyear2000.ctf.bsidessf.net/.git](http://theyear2000.ctf.bsidessf.net/.git) by loading that address into our Web browser.

Sure enough, we are greeted with a default Apache error page reading:

> # Forbidden
> 
> You don't have permission to access /.git/ on this server.

Two things are interesting about this. First of all, the server didn't respond with a "Not Found" error. It said "Forbidden." That's our first clue that a `.git` folder actually exists! And second, notice that even though we asked for `/.git`, we were redirected to `/.git/`. This is further confirmation that the directory *exists*, but is not *listable*, probably because this server has been configured to disallow [directory listings](https://wiki.apache.org/httpd/DirectoryListings).

Since we can surmise that a git repository is present, we can further verify this assumption by trying to access some of git's metadata files directly. The next-most obvious location to check is to see if the local Git configuration file, `.git/config` exists. Again, we simply need to load that URL path ([http://theyear2000.ctf.bsidessf.net/.git/config](http://theyear2000.ctf.bsidessf.net/.git/config)) in our browser.

We're greeted with a standard git configuration file:

```
[core]
    repositoryformatversion = 0
    filemode = true
    bare = false
    logallrefupdates = true
[user]
    email = thezuck@therealzuck.zuck
    name = Mark Zuckerberg
```

This tells us that the author of this page is someone named `Mark Zuckerberg` whose email address is `thezuck@therealzuck.zuck`. That's not immediately useful during a CTF challenge, but in "real life," we could use this information for other purposes. But for our purposes in solving the CTF challenge, the important thing about this discovery is that we have confirmed the existence of git repository.

Furthermore, we can see from the configuration file that this repository has been logging reference updates (due to the line in the `[core]` section that reads `logallrefupdates = true`). This means whenever the user changed branches, added a commit, or performed any other action that changed where the tip of a branch, tag, or other "reference" pointed to, a record of this change was probably logged to git's local reference log (or "reflog"). This reference log can later be accessed by the user with [the `git reflog` command](https://git-scm.com/docs/git-reflog). Git stores a separate log file for each "ref" (branch, etc.) in the `.git/logs` subdirectory. Although the names of refs in a given repository are defined by their user, one universal ref is the `HEAD` ref, which simply points to the tip of the current branch.

Knowing this, we can look at the reflog for the `HEAD` ref by accessing the URL [http://theyear2000.ctf.bsidessf.net/.git/logs/HEAD](http://theyear2000.ctf.bsidessf.net/.git/logs/HEAD). Sure enough, this gives us the reflog for `HEAD`, showing us a brief history of the repository's actions:

```
0000000000000000000000000000000000000000 e039a6684f53e818926d3f62efd25217b25fc97e Mark Zuckerberg <thezuck@therealzuck.zuck> 1486853661 +0000  commit (initial): First commit on my website
e039a6684f53e818926d3f62efd25217b25fc97e 9e9ce4da43d0d2dc10ece64f75ec9cab1f4e5de0 Mark Zuckerberg <thezuck@therealzuck.zuck> 1486853667 +0000  commit: Fixed a spelling error
9e9ce4da43d0d2dc10ece64f75ec9cab1f4e5de0 e039a6684f53e818926d3f62efd25217b25fc97e Mark Zuckerberg <thezuck@therealzuck.zuck> 1486853668 +0000  reset: moving to HEAD~1
e039a6684f53e818926d3f62efd25217b25fc97e 4eec6b9c6e464c35fff1efb8444dd0ac1ae67b30 Mark Zuckerberg <thezuck@therealzuck.zuck> 1486853672 +0000  commit: Wooops, didn't want to commit that. Rebased.
```

We read the reflog from top to bottom, left to right. It's mostly self-explanatory: the first two fields of hexadecimal digits are git hashes that refer to some object, such as a commit, a tag, and so on. The one on the left is the old hash to which this ref (in this case, `HEAD`) pointed at, and the one on the right is the new one. The third field is the user's name and email address from the `config` file (which we saw was "Mark Zuckerberg"). This is followed by the timestamp of the log entry in [Unix time](https://en.wikipedia.org/wiki/Unix_time) and timezone (`1486853661 +0000` is equivalent to Sat, 11 Feb 2017 22:54:21 GMT according to any [Unix time converter](http://www.onlineconversion.com/unix_time.htm)). The next field records which action the user took. This reflog shows only `commit` and `reset` actions, and finally a "reason," which is usually a commit message or details about the action that the user took.

There are two pieces of useful information we can glean from this reflog. The first is that we have specific git hashes. These hashes tell us the URLs that we need to access to manually reconstruct the git repository data itself. The second piece of information that's useful to is the final commit message in this reflog: "Wooops, didn't want to commit that. Rebased."

Hmm, what didn't Mark Zuckerberg want to commit? Let's find out!

To find the missing commit, we will want to reconstruct his repository locally so we can explore it ourselves. That means downloading all the git objects and metadata files and placing them into the expected locations on our own computer.

We start by creating a new folder. Let's call it `theyear2000`. Inside that, we want to recreate the `.git` directory. And inside that, we'll want to create a directory called `logs`. We can do this in one command:

```sh
mkdir -p theyear2000/.git/logs
```

Now let's move into the `.git` directory oursevles so we can begin downloading files and reconstructing the repository:

```sh
cd theyear2000/.git
```

First, we'll want to download a copy of the reflog we were just looking at and the `config` file we found earlier into their appropriate places to begin the process of reconstructing this git repository locally. We can simply use our Web browser's "Save" function to download them into the correct folder. When we're done, we should have a filesystem layout that currently looks like this, as shown by the output of [the `tree` command](https://linux.die.net/man/1/tree):

```sh
$ tree # we run this after changing into the `.git` directory
.
├── config
└── logs
    └── HEAD

1 directory, 2 files
```

However, if we try to run a normal git command with just these files, such as `git log`, we see that we don't yet have quite enough in place for git to consider this a real repository:

```sh
$ git log
fatal: Not a git repository (or any of the parent directories): .git
```

That's because we're still missing some important metadata files. Specifically, at the very minimum, we need:

* `.git/HEAD`, the file pointing at the "current" `HEAD` object,
* `.git/refs/heads/<ref>`, a file that actually names the current branch (usually called `master` by default convention), and
* `.git/objects/<subdirectory>/<object file>`, the actual binary data of a given git object. (More on this in a bit.)

So let's begin by accessing each of these in turn. The first is easy. We simply access [http://theyear2000.ctf.bsidessf.net/.git/HEAD](http://theyear2000.ctf.bsidessf.net/.git/HEAD) in our browser. This file is a plain text file that reads:

```
ref: refs/heads/master
```

This is how git knows which branch you're currently on, for example. In this case, we can see that Mark Zuckerberg was on the `master` branch. Let's save this file inside our local `.git` directory and move on.

Next, we need to access the file at `refs/heads/master`, which would be at [http://theyear2000.ctf.bsidessf.net/.git/refs/heads/master](http://theyear2000.ctf.bsidessf.net/.git/refs/heads/master). Sure enough, we're greeted with another plaintext file:

```
4eec6b9c6e464c35fff1efb8444dd0ac1ae67b30
```

This is a git hash. Moreover, notice that it exactly matches the final entry in the reflog we found earlier. That is to say, this is the tip of the `master` branch.

Now we can start downloading the actual git object data itself. We could either go backwards, by downloading the tip (the object named in the `refs/heads/master` file and at the bottom-right of the information we can read out of the reflog), or we can go forwards, starting at the very first commit and retracing each successive action in the reflog in turn. It doesn't particularly matter, but in this case I chose to go backwards because the "interesting" action in the reflog was the "Wooops" commit near the end.

To download git objects, we need to know a little bit about how Git stores these objects. A [git "object"](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects) is simply a file that contains some data. The file is given a unique key (or name), so that we can later retrieve that data by naming it using the key. (This is what is meant by "git has a simple key-value store.") Git hashes are the names ("keys") given to specific git objects. When creating objects such as commits, git writes the data into a file or files whose file locations are derived from these hashes. Git always stores these files inside a subdirectory of its `objects` metadata directory whose name is the first two characters of the hash.

In the case of this most recent commit's hash (`4eec6b9c6e464c35fff1efb8444dd0ac1ae67b30`), this means we should expect to find this object data inside a directory located at `.git/objects/4e/`. We can test this assumption by once again accessing this URL at [http://theyear2000.ctf.bsidessf.net/.git/objects/4e/](http://theyear2000.ctf.bsidessf.net/.git/objects/4e/). We are again greeted by a "Forbidden" message, proving that the directory exists.

The next step is to deduce the filename of the object. Git names files with the remainder of the hash, the part that isn't used for the directory name. In this case, that's `ec6b9c6e464c35fff1efb8444dd0ac1ae67b30` (the same hash with the first two characters removed). Let's try this by accessing the URL at [http://theyear2000.ctf.bsidessf.net/.git/objects/4e/ec6b9c6e464c35fff1efb8444dd0ac1ae67b30](http://theyear2000.ctf.bsidessf.net/.git/objects/4e/ec6b9c6e464c35fff1efb8444dd0ac1ae67b30). This time, since the git object data is saved in a binary format (not text), our Web browser asks us if we'd like to download and save the file. So let's save it and then move it to the same place as it was on the server, inside our local `.git/objects/4e/` directory.

When we do, we should now have a filesystem layout like this:

```sh
$ tree
.
├── HEAD
├── config
├── logs
│   └── HEAD
├── objects
│   └── 4e
│       └── ec6b9c6e464c35fff1efb8444dd0ac1ae67b30
└── refs
    └── heads
        └── master

5 directories, 5 files
```

At this point when we try `git log` again, we get a different error:

```sh
$ git log
error: Could not read e039a6684f53e818926d3f62efd25217b25fc97e
fatal: Failed to traverse parents of commit 4eec6b9c6e464c35fff1efb8444dd0ac1ae67b30
```

Here we clearly see that git is trying to locate, but cannot read (or find) the object called `e039a6684f53e818926d3f62efd25217b25fc97e`. Knowing what we do about git object storage, we would expect this file to be in `.git/objects/e0/39a6684f53e818926d3f62efd25217b25fc97e`, but sure enough there is no such file in our local repository. Let's add it. We do this in the same way as we did for the first object. We simply access the URL at [http://theyear2000.ctf.bsidessf.net/.git/objects/e0/39a6684f53e818926d3f62efd25217b25fc97e](http://theyear2000.ctf.bsidessf.net/.git/objects/e0/39a6684f53e818926d3f62efd25217b25fc97e) and save that file locally in the same directory structure. When we do that, our local repostiory layout looks like this, with two objects instead of one:

```sh
$ tree
.
├── HEAD
├── config
├── logs
│   └── HEAD
├── objects
│   ├── 4e
│   │   └── ec6b9c6e464c35fff1efb8444dd0ac1ae67b30
│   └── e0
│       └── 39a6684f53e818926d3f62efd25217b25fc97e
└── refs
    └── heads
        └── master

6 directories, 6 files

```

Running `git log` now gives us meaningful output:

```sh
$ git log
commit 4eec6b9c6e464c35fff1efb8444dd0ac1ae67b30
Author: Mark Zuckerberg <thezuck@therealzuck.zuck>
Date:   Sat Feb 11 22:54:32 2017 +0000

    Wooops, didn't want to commit that. Rebased.

commit e039a6684f53e818926d3f62efd25217b25fc97e
Author: Mark Zuckerberg <thezuck@therealzuck.zuck>
Date:   Sat Feb 11 22:54:21 2017 +0000

    First commit on my website
```

This information matches the information we read from the reflog, so we know we're on the right track. Unfortunately, we still don't have enough of the repository reconstructed to find out what the *contents* of any of these commits were, as evidenced by running `git show`:

```sh
$ git show
fatal: unable to read tree 0ce1cbf654058dd4b9ba0df440a02aef408f76da
```

Here, again, we are told that we can't read (or indeed find) a given object. This object is not a commit, though, it's a tree (a git object that is somewhat analogous to a directory inside of git itself). Notice that this git hash never appeared in the reflog because it's not an object that a user interacts with directly. Nevertheless, we can download it just as we did the other two. Doing so and running `git show` again, however, reveals that we're still missing another object:

```sh
$ git show
fatal: unable to read tree f3a3f88425975542bb0058651867f8090fed250f
```

We know what to do by now. Download this object, save it in the appropriate place, and try again. This time, we get something slightly different:

```sh
$ git show
fatal: unable to read 7c57d178eea98e174f3d6ef521126117478085ed
commit 4eec6b9c6e464c35fff1efb8444dd0ac1ae67b30
Author: Mark Zuckerberg <thezuck@therealzuck.zuck>
Date:   Sat Feb 11 22:54:32 2017 +0000

    Wooops, didn't want to commit that. Rebased.
```

We can see the commit message, but we are still missing an object (evidenced by the line `fatal: unable to read 7c57d178eea98e174f3d6ef521126117478085ed`), so let's grab that object as well and try again. We still have another missing object:

```sh
$ git show
fatal: unable to read e16b652d659d50fc5e7aecae789e743c0a8fa035
commit 4eec6b9c6e464c35fff1efb8444dd0ac1ae67b30
Author: Mark Zuckerberg <thezuck@therealzuck.zuck>
Date:   Sat Feb 11 22:54:32 2017 +0000

    Wooops, didn't want to commit that. Rebased.
```

Let's grab that object, too, and try once more:

```sh
$ git show
commit 4eec6b9c6e464c35fff1efb8444dd0ac1ae67b30
Author: Mark Zuckerberg <thezuck@therealzuck.zuck>
Date:   Sat Feb 11 22:54:32 2017 +0000

    Wooops, didn't want to commit that. Rebased.

diff --git a/index.html b/index.html
index 7c57d17..e16b652 100644
--- a/index.html
+++ b/index.html
@@ -15,7 +15,7 @@ pre {
 </style>
 </head>
 <body>
-<h1>Welcome to my homepage!!!!</h1>
+<h1>Welcome to my homepage, there are no flags here.!!!!</h1>
 <hr>
 <p>I made this website all by myself using these tools
 <ul>
```

Progress! We have now reconstructed part of the git repository. Our filesystem layout looks like this:

```sh
$ tree
.
├── HEAD
├── config
├── index
├── logs
│   └── HEAD
├── objects
│   ├── 0c
│   │   └── e1cbf654058dd4b9ba0df440a02aef408f76da
│   ├── 4e
│   │   └── ec6b9c6e464c35fff1efb8444dd0ac1ae67b30
│   ├── 7c
│   │   └── 57d178eea98e174f3d6ef521126117478085ed
│   ├── e0
│   │   └── 39a6684f53e818926d3f62efd25217b25fc97e
│   ├── e1
│   │   └── 6b652d659d50fc5e7aecae789e743c0a8fa035
│   └── f3
│       └── a3f88425975542bb0058651867f8090fed250f
└── refs
    └── heads
        └── master

10 directories, 11 files
```

Unfortunately, nothing has shown us the flag yet. This is because we've followed the git commit history itself, but the reflog shows a `reset`:

```
9e9ce4da43d0d2dc10ece64f75ec9cab1f4e5de0 e039a6684f53e818926d3f62efd25217b25fc97e Mark Zuckerberg <thezuck@therealzuck.zuck> 1486853668 +0000  reset: moving to HEAD~1
```

Notice that we downloaded many objects, but none were `9e9ce4da43d0d2dc10ece64f75ec9cab1f4e5de0`. We can clearly see in the reflog that this object is a commit, thanks to the previous line:

```
e039a6684f53e818926d3f62efd25217b25fc97e 9e9ce4da43d0d2dc10ece64f75ec9cab1f4e5de0 Mark Zuckerberg <thezuck@therealzuck.zuck> 1486853667 +0000  commit: Fixed a spelling error
```

However, this commit is nowhere to be seen in the `git log` output:

```sh
$ git log
commit 4eec6b9c6e464c35fff1efb8444dd0ac1ae67b30
Author: Mark Zuckerberg <thezuck@therealzuck.zuck>
Date:   Sat Feb 11 22:54:32 2017 +0000

    Wooops, didn't want to commit that. Rebased.

commit e039a6684f53e818926d3f62efd25217b25fc97e
Author: Mark Zuckerberg <thezuck@therealzuck.zuck>
Date:   Sat Feb 11 22:54:21 2017 +0000

    First commit on my website
```

The reason is because it was written out of the commit history by Mark Zuckerberg's very last action, also visible in the reflog:

```
e039a6684f53e818926d3f62efd25217b25fc97e 4eec6b9c6e464c35fff1efb8444dd0ac1ae67b30 Mark Zuckerberg <thezuck@therealzuck.zuck> 1486853672 +0000  commit: Wooops, didn't want to commit that. Rebased.
```

In [Git, "rebasing"](https://git-scm.com/book/en/v2/Git-Branching-Rebasing) is a technique that integrates changes from one branch into another, but does so by rewriting the git commit history. In this case, our fictional Mark Zuckerberg wanted to remove a commit from the git history and so reattached another commit as the child of the previous commit, but didn't *actually remove* the unwanted commit. That's the one we want, and we know its name from the reflog. Let's just get it, same as we did all the others, by accessing [http://theyear2000.ctf.bsidessf.net/.git/objects/9e/9ce4da43d0d2dc10ece64f75ec9cab1f4e5de0](http://theyear2000.ctf.bsidessf.net/.git/objects/9e/9ce4da43d0d2dc10ece64f75ec9cab1f4e5de0) this time.

Once saved in the correct place, we want to see its content, so this time we'll ask `git show` to show us that object, specifically:

```sh
$ git show 9e9ce4da43d0d2dc10ece64f75ec9cab1f4e5de0
fatal: unable to read tree bd72ee2c7c5adb017076fd47a92858cef2a04c11
```

Aha, another missing tree. We've done this before, so let's simply follow the rabbit hole again. This leads us to downloading the following additional objects:

* `bd72ee2c7c5adb017076fd47a92858cef2a04c11`
* `7baff32394e517c44f35b75079a9496559c88053`

Now, our filesystem layout looks like this:

```sh
$ tree
.
├── HEAD
├── config
├── index
├── logs
│   └── HEAD
├── objects
│   ├── 0c
│   │   └── e1cbf654058dd4b9ba0df440a02aef408f76da
│   ├── 4e
│   │   └── ec6b9c6e464c35fff1efb8444dd0ac1ae67b30
│   ├── 7b
│   │   └── aff32394e517c44f35b75079a9496559c88053
│   ├── 7c
│   │   └── 57d178eea98e174f3d6ef521126117478085ed
│   ├── 9e
│   │   └── 9ce4da43d0d2dc10ece64f75ec9cab1f4e5de0
│   ├── bd
│   │   └── 72ee2c7c5adb017076fd47a92858cef2a04c11
│   ├── e0
│   │   └── 39a6684f53e818926d3f62efd25217b25fc97e
│   ├── e1
│   │   └── 6b652d659d50fc5e7aecae789e743c0a8fa035
│   └── f3
│       └── a3f88425975542bb0058651867f8090fed250f
└── refs
    └── heads
        └── master

13 directories, 14 files
```

And now, asking to show the commit object reveals the flag:

```sh
$ git show 9e9ce4da43d0d2dc10ece64f75ec9cab1f4e5de0
commit 9e9ce4da43d0d2dc10ece64f75ec9cab1f4e5de0
Author: Mark Zuckerberg <thezuck@therealzuck.zuck>
Date:   Sat Feb 11 22:54:27 2017 +0000

    Fixed a spelling error

diff --git a/index.html b/index.html
index 7c57d17..7baff32 100644
--- a/index.html
+++ b/index.html
@@ -43,3 +43,4 @@ ______________________
 </pre>
 </marquee>
 </body></html>
+Your flag is... FLAG:what_is_HEAD_may_never_die
```

## Bonus: Tools

Now that you know a lot more about git repository structure and how to download and reconstruct its pieces manually, you may be intersted to automate this process in the future. Two good tools exist for this purpose:

* [DVCS Ripper](https://github.com/kost/dvcs-ripper), a set of Perl scripts that can download ("rip") SVN, Git, Mercurial/hg, bzr repositories that are exposed on websites, and
* [GitTools](https://github.com/internetwache/GitTools), triplet of Python scripts that can automatically find and download exposed Git repositories.

Enjoy!
