# Capture The Flag (CTF)

[Capture The Flag competitions](https://github.com/AnarchoTechNYC/meta/wiki/InfoSec#ctfs-and-hacking-games) often supply a number of small attachments or puzzle files. This repository is a collection of those assets and associated write-ups describing how to approach the challenge. The aim is to build an archive of challenges that I've personally encountered while playing CTFs with friends or alone, or found elsewhere and used as self-education exercises.

This repository is currently an experiment to figure out how to effectively introduce inexperienced hackers to CTFs and to penetration testing more generally. Unlike many CTF writeups which simply describe how a given participant or team solved a challenge, I aim to include a wealth of supplemental yet relevant background information into my writeups. Similarly, I aim to solve as many challenges as I can *manually*, i.e., without the aid of popular tools, in order to ensure that I fully understand why a given hack actually works.

**This is still a work in progress. I'm not entirely sure how I want to organize this repo or with what voice/format I will be writing the CTF challenge writeups.**

## Repository structure

> :construction: This is a work in progress. Suggestions welcome.

This repository is structured in a manner optimized for learning from CTF writeups. Its directory hierarchy is as follows:

* *Year*
    * *CTF name/event*
        * `README.md` file containing:
            * CTF format
            * Flag format
            * Source code for challenges, if released
            * Notes, if any
        * *Challenge category*
            * *Challenge name*
                * `README.md` file containing the writeup itself
                * `attachments/` folder containing any downloadable attachments given in the challenge clue.
                * `screenshots/` folder containing any screenshots taken during the challenge and used in the writeup.
                * `loot/` folder containing any assets acquired during the challenge itself.
