# matryoshka

# Challenge

> After a lecture on files and the structure of the file system, William James was accosted by a little old lady.

> "Your theory that the file system is the primary unit of storage has a very convincing ring to it, Mr. James, but it's wrong. I've got a better theory," said the little old lady.

> "And what is that, madam?" Inquired James politely.

> "That every file we create is just inside of an archive,"

> Not wishing to demolish this absurd little theory by bringing to bear the masses of computer scientific evidence he had at his command, James decided to gently dissuade his opponent by making her see some of the inadequacies of her position.

> "If your theory is correct, madam," he asked, "what is this archive stored in?"

> "You're a very clever man, Mr. James, and that's a very good question," replied the little old lady, "but I have an answer to it. And it is this: The first archive is stored in a second, far larger, archive."

> "But what is this second archive stored in?" persisted James patiently.

> To this the little old lady crowed triumphantly. "It's no use, Mr. James – it's archives all the way down."

> Note: Flag does not follow the "Flag:" format but is recognizable

> [file.bin](file.bin)

## Solution
As the challenge description describes this file has been archived and compressed in an absurd amount of different compression algorithms. 

After hours of finding the appropriate compression tools I finally arrived at a .wav file. I opened it in VLC hoping for some audio output sadly it wasn't that simple. Running `cat` against the file gave me this output.
```
$ cat file.wav
RIFF�WAVEfmt dataw.. - ... - .... . .. -. -.-. .-. . -.. .. -... .-.. . ... .... .-. .. -. -.- .. -. --. -- --- .-. ... . -.-. --- -.. .
```
I wrote a small python script to decode the morse-code.
```
#!/usr/bin/env python

morse = ".. - ... - .... . .. -. -.-. .-. . -.. .. -... .-.. . ... .... .-. .. -. -.- .. -. --. -- --- .-. ... . -.-. --- -.. ."

CODE = {'.-': 'A',    '-...': 'B',  '-.-.': 'C',
        '-..': 'D',   '.': 'E',     '..-.': 'F',
        '--.': 'G',   '....': 'H',  '..': 'I',
        '.---': 'J',  '-.-': 'K',   '.-..': 'L',
        '--': 'M',    '-.': 'N',    '---': 'O',
        '.--.': 'P',  '--.-': 'Q',  '.-.': 'R',
        '...': 'S',   '-': 'T',     '..-': 'U',
        '...-': 'V',  '.--': 'W',   '-..-': 'X',
        '-.--': 'Y',  '--..': 'Z',

        '-----': '0', '.----': '1', '..---': '2',
        '...--': '3', '....-': '4', '.....': '5',
        '-....': '6', '--...': '7', '---..': '8',
        '----.': '9'
        }

morseDecoded = ''.join(CODE.get(i) for i in morse.split())

print morseDecoded
```
Running it gave me the flag.
```
$ ./decodeMorse.py 
ITSTHEINCREDIBLESHRINKINGMORSECODE
```
