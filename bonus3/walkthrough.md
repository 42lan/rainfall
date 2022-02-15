Login as `bonus3`.
```shell
┌──$ [~/42/2021/rainfall/bonus3]
└─>  ssh 192.168.0.25 -p 4242 -l bonus3
bonus0@192.168.0.25's password: 71d449df0f960b36e0055eb58c14d0f5d0ddc0b35328d657f91cf0df15910587
```
A `SUID` executable is located in the home directory.
```shell
bonus3@RainFall:~$ ls -l
total 8
-rwsr-s---+ 1 end users 5595 Mar  6  2016 bonus3
```
It first open `/home/user/bonus3/bonus3` file in readonly mode
```gdb
   [...]
   0x08048502 <+14>:	mov    edx,0x80486f0 // "r"
   0x08048507 <+19>:	mov    eax,0x80486f2 // "/home/user/end/.pass"
   0x0804850c <+24>:	mov    DWORD PTR [esp+0x4],edx
   0x08048510 <+28>:	mov    DWORD PTR [esp],eax
   0x08048513 <+31>:	call   0x8048410 <fopen@plt>
   [...]
```
Then checks if return `fd` is not equals to 0 and that binary was runned with one argument. Otherwise it return -1.
```gdb
   0x08048533 <+63>:	cmp    DWORD PTR [esp+0x9c],0x0
   [...]
   0x0804853d <+73>:	cmp    DWORD PTR [ebp+0x8],0x2
```
Once, preliminary checks are passed it calls `fread` which reads 66 bytes (length of flag) from the stream pointed to by `fd` stroring them at the location pointed by `edx`.
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
