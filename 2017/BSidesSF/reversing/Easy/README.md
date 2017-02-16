# Easy

## Challenge

> This one is easy.
> 
> * [easy-32](easy-32)
> * [easy-64](easy-64)

## Solution

The absolute basics of reverse engineering ("reversing") is to extract information out of binary files. In this challenge, we are presented with two binaries, one claiming to be a 32-bit architecture and one for a 64-bit archicture. Since I have a 64-bit machine, I downloaded the `easy-64` binary.

There's no need to execute it, because the very first thing we want to do is inspect the binary directly to see if there are any plaintext strings in it that are useful to us. The [`strings` command](https://linux.die.net/man/1/strings) is perfect for this. Running `strings` against the binary gives us a lot of output:

```sh
root@kali:~/Downloads# strings easy-64
/lib64/ld-linux-x86-64.so.2
libc.so.6
gets
puts
__stack_chk_fail
strcmp
__libc_start_main
__gmon_start__
GLIBC_2.4
GLIBC_2.2.5
UH-P
AWAVA
AUATL
[]A\A]A^A_
What is the password?
the password
FLAG:db2f62a36a018bce28e46d976e3f9864
Wrong!!
;*3$"
GCC: (Ubuntu 5.4.0-6ubuntu1~16.04.4) 5.4.0 20160609
crtstuff.c
__JCR_LIST__
deregister_tm_clones
__do_global_dtors_aux
completed.7585
__do_global_dtors_aux_fini_array_entry
frame_dummy
__frame_dummy_init_array_entry
easy.c
__FRAME_END__
__JCR_END__
__init_array_end
_DYNAMIC
__init_array_start
__GNU_EH_FRAME_HDR
_GLOBAL_OFFSET_TABLE_
__libc_csu_fini
_ITM_deregisterTMCloneTable
puts@@GLIBC_2.2.5
_edata
__stack_chk_fail@@GLIBC_2.4
__libc_start_main@@GLIBC_2.2.5
__data_start
strcmp@@GLIBC_2.2.5
__gmon_start__
__dso_handle
_IO_stdin_used
gets@@GLIBC_2.2.5
__libc_csu_init
__bss_start
main
_Jv_RegisterClasses
__TMC_END__
_ITM_registerTMCloneTable
.symtab
.strtab
.shstrtab
.interp
.note.ABI-tag
.note.gnu.build-id
.gnu.hash
.dynsym
.dynstr
.gnu.version
.gnu.version_r
.rela.dyn
.rela.plt
.init
.plt.got
.text
.fini
.rodata
.eh_frame_hdr
.eh_frame
.init_array
.fini_array
.jcr
.dynamic
.got.plt
.data
.bss
.comment
```

If we read carefully, we can see the flag, right there in plain sight. If you missed it, we can filter the output using the venerable [`grep` command](https://linux.die.net/man/1/grep).

```sh
root@kali:~/Downloads# strings easy-64 | grep FLAG
FLAG:db2f62a36a018bce28e46d976e3f9864
```

And there we have it. :) That *was* easy.

## Bonus: What is the password?

Despite not needing to run the binary, we *could* run it. When we do actually execute, we are greeted with some output asking us "What is the password?" One of the most common default passwords is `password`, so we can try that:

```sh
root@kali:~/Downloads# ./easy-64 
What is the password?
password
Wrong!!
```

Unfortunately, `password` is not the correct password, and we are told it is "Wrong!!" We can use `strings` again to aid us here.

Looking back on the full output of the `strings` command, we can see that one of the lines of output is the prompt: "What is the password?" That line is followed immediately by another line, which reads `the password`. We can use `strings` piped to `grep` again, this time telling `grep` to show us some context lines, to show that this is so:

```sh
root@kali:~/Downloads# strings easy-64 | grep -A 1 "What is the password" # the -A option shows "one line of context after a match"
What is the password?
the password
```

It's reasonable to assume that the program source code prints the prompt, and then immediately compares that string that the user enters with some built-in string to determine if the entered string matches. If that is so, a string near the prompt would be the password. In this case, that string is literally `the password`. Let's see what happens when we use that string (`the password`) as the password:

```sh
root@kali:~/Downloads# ./easy-64 
What is the password?
the password
FLAG:db2f62a36a018bce28e46d976e3f9864
```

Bingo! The password turned out to be `the password`, and we are rewarded with the same flag. :)
