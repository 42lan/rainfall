Login as `level0`.
```shell
┌──$ [~/42/2021/rainfall]
└─>  ssh 192.168.1.75 -p 4242 -l level0
level0@192.168.1.75's password: level0
  GCC stack protector support:            Enabled
  Strict user copy checks:                Disabled
  Restrict /dev/mem access:               Enabled
  Restrict /dev/kmem access:              Enabled
  grsecurity / PaX: No GRKERNSEC
  Kernel Heap Hardening: No KERNHEAP
 System-wide ASLR (kernel.randomize_va_space): Off (Setting: 0)
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
No RELRO        No canary found   NX enabled    No PIE          No RPATH   No RUNPATH   /home/user/level0/level0
```
```shell
level0@RainFall:~$ ls -l
total 732
-rwsr-x---+ 1 level1 users 747441 Mar  6  2016 level0
level0@RainFall:~$ ./level0
Segmentation fault (core dumped)
level0@RainFall:~$ ./level0 ls
No !
level0@RainFall:~$ strace ./level0
-bash: /usr/bin/strace: Input/output error
```
Input/output error mostly due to bad blocks on the disk (hardware issues).
```gdb
level0@RainFall:~$ gdb ./level0
(gdb) break main
Breakpoint 1 at 0x8048ec3
(gdb) run
Starting program: /home/user/level0/level0

Breakpoint 1, 0x08048ec3 in main ()
(gdb) disas
Dump of assembler code for function main:
   0x08048ec0 <+0>:	push   %ebp
   0x08048ec1 <+1>:	mov    %esp,%ebp
=> 0x08048ec3 <+3>:	and    $0xfffffff0,%esp
   0x08048ec6 <+6>:	sub    $0x20,%esp
   0x08048ec9 <+9>:	mov    0xc(%ebp),%eax
   0x08048ecc <+12>:	add    $0x4,%eax
   0x08048ecf <+15>:	mov    (%eax),%eax
   0x08048ed1 <+17>:	mov    %eax,(%esp)
   0x08048ed4 <+20>:	call   0x8049710 <atoi>
   0x08048ed9 <+25>:	cmp    $0x1a7,%eax      # 0x1a7 = 423
   0x08048ede <+30>:	jne    0x8048f58 <main+152>
```
```shell
level0@RainFall:~$ ./level0 423
$
$ ls
ls: cannot open directory .: Permission denied
$ id
uid=2030(level1) gid=2020(level0) groups=2030(level1),100(users),2020(level0)
$ cat /home/user/level1/.pass
1fe8a524fa4bec01ca4ea2a869af2a02260d4a7d5fe7e7c24d8617e6dca12d3a
```
