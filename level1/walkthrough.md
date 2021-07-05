Login as `level1`.
```shell
┌──$ [~/42/2021/rainfall]
└─>  ssh 192.168.1.75 -p 4242 -l level1
level1@192.168.1.75's password: 1fe8a524fa4bec01ca4ea2a869af2a02260d4a7d5fe7e7c24d8617e6dca12d3a
  GCC stack protector support:            Enabled
  Strict user copy checks:                Disabled
  Restrict /dev/mem access:               Enabled
  Restrict /dev/kmem access:              Enabled
  grsecurity / PaX: No GRKERNSEC
  Kernel Heap Hardening: No KERNHEAP
 System-wide ASLR (kernel.randomize_va_space): Off (Setting: 0)
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
No RELRO        No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   /home/user/level1/level1
```
A `SUID` executable is located in the home directory.
```shell
level1@RainFall:~$ ls -l
total 8
-rwsr-s---+ 1 level2 users 5138 Mar  6  2016 level1
level1@RainFall:~$
```
```gdb
level1@RainFall:~$ gdb ./level1
(gdb) break main
Breakpoint 1 at 0x8048483
(gdb) run
Starting program: /home/user/level1/level1

Breakpoint 1, 0x08048483 in main ()
(gdb) disassemble
Dump of assembler code for function main:
   0x08048480 <+0>:		push   %ebp
   0x08048481 <+1>:		mov    %esp,%ebp
=> 0x08048483 <+3>:		and    $0xfffffff0,%esp
   0x08048486 <+6>:		sub    $0x50,%esp
   0x08048489 <+9>:		lea    0x10(%esp),%eax
   0x0804848d <+13>:	mov    %eax,(%esp)
   0x08048490 <+16>:	call   0x8048340 <gets@plt>
   0x08048495 <+21>:	leave
   0x08048496 <+22>:	ret
End of assembler dump.
(gdb)
```

```shell
level1@RainFall:~$ ./level1 <<< $(python -c 'print("+"*75)'); echo $?
32
level1@RainFall:~$ ./level1 <<< $(python -c 'print("+"*76)'); echo $?
Illegal instruction (core dumped)
132
level1@RainFall:~$ ./level1 <<< $(python -c 'print("+"*77)'); echo $?
Segmentation fault (core dumped)
139
```
GDB allows to print the names and data types of all defined functions.
```gdb
level1@RainFall:~$ gdb -q ./level1
Reading symbols from /home/user/level1/level1...(no debugging symbols found)...done.
(gdb) info functions
All defined functions:

Non-debugging symbols:
0x080482f8  _init
0x08048340  gets
0x08048340  gets@plt
0x08048350  fwrite
0x08048350  fwrite@plt
0x08048360  system
0x08048360  system@plt
0x08048370  __gmon_start__
0x08048370  __gmon_start__@plt
0x08048380  __libc_start_main
0x08048380  __libc_start_main@plt
0x08048390  _start
0x080483c0  __do_global_dtors_aux
0x08048420  frame_dummy
0x08048444  run
0x08048480  main
0x080484a0  __libc_csu_init
0x08048510  __libc_csu_fini
0x08048512  __i686.get_pc_thunk.bx
0x08048520  __do_global_ctors_aux
0x0804854c  _fini
(gdb) disassemble run
Dump of assembler code for function run:
   0x08048444 <+0>:	push   %ebp
   0x08048445 <+1>:	mov    %esp,%ebp
   0x08048447 <+3>:	sub    $0x18,%esp
   0x0804844a <+6>:	mov    0x80497c0,%eax
   0x0804844f <+11>:	mov    %eax,%edx
   0x08048451 <+13>:	mov    $0x8048570,%eax
   0x08048456 <+18>:	mov    %edx,0xc(%esp)
   0x0804845a <+22>:	movl   $0x13,0x8(%esp)
   0x08048462 <+30>:	movl   $0x1,0x4(%esp)
   0x0804846a <+38>:	mov    %eax,(%esp)
   0x0804846d <+41>:	call   0x8048350 <fwrite@plt>
   0x08048472 <+46>:	movl   $0x8048584,(%esp)
   0x08048479 <+53>:	call   0x8048360 <system@plt>
   0x0804847e <+58>:	leave
   0x0804847f <+59>:	ret
End of assembler dump.
```

Seems that `run()` makes a system call.
Examine memory address `0x8048584` in string format shows which syscall is made.
```gdb
(gdb) x/s 0x8048584
0x8048584:	 "/bin/sh"
(gdb) print (char*) 0x8048584
$16 = 0x8048584 "/bin/sh"
```

By analysing binary with Ghidra, its confirmed that `run()` execute a shell.
```ghidra
void run(void)
{
  fwrite("Good... Wait what?\n",1,0x13,stdout);
  system("/bin/sh");
  return;
}
```

Print content of `eax` register before function calls
```gdb
(gdb) disassemble run
Dump of assembler code for function run:
   0x08048444 <+0>:	push   %ebp
   0x08048445 <+1>:	mov    %esp,%ebp
   0x08048447 <+3>:	sub    $0x18,%esp
   0x0804844a <+6>:	mov    0x80497c0,%eax
   0x0804844f <+11>:	mov    %eax,%edx
   0x08048451 <+13>:	mov    $0x8048570,%eax
   0x08048456 <+18>:	mov    %edx,0xc(%esp)
   0x0804845a <+22>:	movl   $0x13,0x8(%esp)
   0x08048462 <+30>:	movl   $0x1,0x4(%esp)
   0x0804846a <+38>:	mov    %eax,(%esp)
   0x0804846d <+41>:	call   0x8048350 <fwrite@plt>
   0x08048472 <+46>:	movl   $0x8048584,(%esp)
   0x08048479 <+53>:	call   0x8048360 <system@plt>
   0x0804847e <+58>:	leave
   0x0804847f <+59>:	ret
End of assembler dump.
(gdb) x/s 0x8048570
0x8048570:	 "Good... Wait what?\n"
(gdb) x/s 0x8048584
0x8048584:	 "/bin/sh"
```
