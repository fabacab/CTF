# Skipper

## Challenge

> The given binary will give you the password... if you meet its criteria!

> [skipper-32](skipper-32)

> [skipper-64](skipper-64)

# Solution
Initially I ran ltrace against the binary to get a feel for the control-flow of the program.
```
$ ltrace ./skipper-32 
....
read(3, "kali\n", 1024)                                                   = 5
printf("Computer name: %s\n", "kali"Computer name: kali
)                                     = 20
strcmp("kali", "hax0rz!~")                                                = 1
printf("Sorry, your computer's name - %s"..., "kali"Sorry, your computer's name - kali - is not correct!
)                     = 53
raise(9, 0xffbf5b2c, 68, 4 <no return ...>
+++ killed by SIGKILL +++
```
Disassembling the main function in radare2 shows that the program runs three checks on the computer to determine name, OS version and CPU type. 
![main](https://github.com/R3dCr3sc3nt/BSidesSF-2017/blob/master/reversing/Skipper/main.png)

Just after the strcmp calls there is a JE that executes if the result of `test eax, eax` is 0. The jump bypasses the error message if the strcmp results in 0 (the strings are equal). 
![jump](https://github.com/R3dCr3sc3nt/BSidesSF-2017/blob/master/reversing/Skipper/jump.png)

Using breakpoints in the debug mode of radare2 makes it possible to edit the eax register before each JE call. The flag is displayed once all three jumps are taken succesfully.
![debug](https://github.com/R3dCr3sc3nt/BSidesSF-2017/blob/master/reversing/Skipper/debug.png)
