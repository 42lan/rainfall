Login as `level7`.
```shell
┌──$ [~/42/2021/rainfall]
└─>  ssh 192.168.0.19 -p 4242 -l level7
level7@192.168.0.19's password: f73dcb7a06f60e3ccc608990b0a046359d42a1a0489ffeefd0d9cb2d7c9cb82d
```
A `SUID` executable is located in the home directory.
```shell
level7@RainFall:~$ ls -l
total 8
-rwsr-s---+ 1 level8 users 5648 Mar  9  2016 level7
```
```gdb
(gdb) info functions
[...]
0x080484f4  m
0x08048521  main
[...]
```
Examing binary with GDB shows that inside `main()` function it opens `/home/user/level8/.pass` file, then reads 68 bytes from the given stream and stores them in the string `c` defined as global variable.
On the other hand the content of `c` variable is printed inside `m()` function.

Function `m()` can be called inside GDB simply by changing `EIP` register pointing to start of `m()` function.
```gdb
(gdb) break *main+202
Breakpoint 1 at 0x80485eb
(gdb) run AAAA BBBB
Starting program: /home/user/level7/level7 AAAA BBBB

Breakpoint 1, 0x080485eb in main ()
(gdb) print $eip
$1 = (void (*)()) 0x80485eb <main+202>
(gdb) set $eip=0x080484f4
(gdb) continue
Continuing.
 - 1630930163

Program received signal SIGSEGV, Segmentation fault.
0x08049960 in c ()
```




0x0804a008 -> $esp+0x1c == str1
```gdb
(gdb) x/16x 0x0804a008

			            fnPtr
            str1[0]     str1[1]     str1[2]     str1[3]
0x804a008:	0x00000001	0x0804a018	0x00000000	0x00000011
            str1[4]     str1[5]     str1[6]     str1[7]
0x804a018:	0x41414141	0x41414141	0x41414141	0x00000000
			            fnPtr
            str2[0]     str2[1]     str2[2]     str2[3]
0x804a028:	0x00000002	0x0804a038	0x00000000	0x00000011
            str2[4]     str2[5]     str2[6]     str2[7]
0x804a038:	0x42424242	0x42424242	0x00000000	0x00020fc
```
