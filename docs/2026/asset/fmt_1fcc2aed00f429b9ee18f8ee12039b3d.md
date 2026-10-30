
```
# Child-SP          RetAddr               Call Site
00 ffff830c`219db4b8 ffff8260`628695df     nt!KeUserModeCallback
01 ffff830c`219db4c0 ffff8260`62869580     win32kfull!pppUserModeCallback+0x2b
02 ffff830c`219db510 ffff8260`628693b7     win32kfull!ClientPrinterThunk+0x74
03 ffff830c`219db550 ffff8260`628ffca6     win32kfull!UMPDOBJ::Thunk+0x73
04 ffff830c`219db5c0 ffff8260`628ffa27     win32kfull!UMPDDrvEnableDriver+0x96
05 ffff830c`219db640 ffff8260`628ff9bc     win32kfull!UMPD_ldevLoadDriver+0x5f
06 ffff830c`219db830 ffff8260`62bba3b1     win32kfull!UMPD_ldevLoadDriverWrap+0xc
07 ffff830c`219db860 ffff8260`62939ff2     win32kbase!hdcOpenDCW+0x1d1
08 ffff830c`219db940 ffff8260`62939e66     win32kfull!GreResetDCInternal+0x11a
09 ffff830c`219dba10 fffff807`3adcb285     win32kfull!NtGdiResetDC+0xd6
0a ffff830c`219dba90 00007ffa`4b056f04     nt!KiSystemServiceCopyEnd+0x25
0b 0000008d`cc32f618 00007ffa`4b9d97bf     win32u!NtGdiResetDC+0x14
0c 0000008d`cc32f620 00007ffa`4b9ffe5d     gdi32full!ResetDCWInternal+0x16b
0d 0000008d`cc32f720 00007ff7`c10d16b3     gdi32full!ResetDCA+0x3d
0e 0000008d`cc32f750 0000023a`02839c00     Exploit!main+0x73 [D:\SelfLearn\C++Project\Exploit\Exploit\2021-40449-POC.cpp @ 236]
0f 0000008d`cc32f758 00000000`00000000     0x0000023a`02839c00
```

### 漏洞触发

漏洞触发位置汇编代码：`rax`即为`vuln_ptr`

```
ffff8260`6293a06c 488b83b80a0000  mov     rax,qword ptr [rbx+0AB8h]
ffff8260`6293a073 4885c0          test    rax,rax
ffff8260`6293a076 741c            je      win32kfull!GreResetDCInternal+0x1bc (ffff8260`6293a094)
Branch    win32kfull!GreResetDCInternal+0x1a0:
ffff8260`6293a078 488b4df7        mov     rcx,qword ptr [rbp-9]
ffff8260`6293a07c 488b5130        mov     rdx,qword ptr [rcx+30h]
ffff8260`6293a080 488b8b08070000  mov     rcx,qword ptr [rbx+708h]
ffff8260`6293a087 488b9208070000  mov     rdx,qword ptr [rdx+708h]
ffff8260`6293a08e ff15fcdf2000    call    qword ptr [win32kfull!_guard_dispatch_icall_fptr (ffff8260`62b48090)]
```

此处CFG并没有实现，只是一条跳转指令

```
0: kd> uf poi(win32kfull!_guard_dispatch_icall_fptr)
Flow analysis was incomplete, some code may be missing
win32kfull!guard_dispatch_icall_nop:
ffff8260`62942a10 ffe0            jmp     rax
```

当执行流程第二次来到``call qword ptr [win32kfull!_guard_dispatch_icall_fptr (ffff8260`62b48090)]``时

```
0: kd> g
Breakpoint 1 hit
win32kfull!GreResetDCInternal+0x1b6:
ffff8260`6293a08e ff15fcdf2000    call    qword ptr [win32kfull!_guard_dispatch_icall_fptr (ffff8260`62b48090)]
0: kd> r rax
rax=ffff82606293c910
0: kd> g
Breakpoint 1 hit
win32kfull!GreResetDCInternal+0x1b6:
ffff8260`6293a08e ff15fcdf2000    call    qword ptr [win32kfull!_guard_dispatch_icall_fptr (ffff8260`62b48090)]
0: kd> r rax
rax=6161616161616161
```

此时`vuln_ptr`已经为`0x6161616161616161`，继续执行触发BSOD

```
CONTEXT:  ffff830c219daf40 -- (.cxr 0xffff830c219daf40)
rax=6161616161616161 rbx=ffff8204857cf070 rcx=4141414141414141
rdx=0000023a0283b320 rsi=0000000000000000 rdi=0000000000000000
rip=ffff826062942a10 rsp=ffff830c219db938 rbp=ffff830c219db9b1
r8=0000000000010000  r9=ffff820480602990 r10=0000000000001d91
r11=0000000000000000 r12=0000000001211d91 r13=0000000000000000
r14=0000000000000000 r15=0000000000000000
iopl=0         nv up ei pl nz na pe nc
cs=0010  ss=0018  ds=002b  es=002b  fs=0053  gs=002b             efl=00010202
win32kfull!guard_dispatch_icall_nop:
ffff8260`62942a10 ffe0            jmp     rax {61616161`61616161}
Resetting default scope

PROCESS_NAME:  Exploit.exe

STACK_TEXT:
ffff830c`219db938 ffff8260`6293a094     : ffff8204`857cf070 00000000`00000000 00000000`00000000 00000000`00000000 : win32kfull!guard_dispatch_icall_nop
ffff830c`219db940 ffff8260`62939e66     : 00000000`62210ae8 00000000`00000000 ffff830c`219dba44 ffff8204`856e7030 : win32kfull!GreResetDCInternal+0x1bc
ffff830c`219dba10 fffff807`3adcb285     : 00000000`62210ae8 ffffe78f`6a70e080 0000008d`cc32f720 ffff830c`219dbaa8 : win32kfull!NtGdiResetDC+0xd6
ffff830c`219dba90 00007ffa`4b056f04     : 00007ffa`4b9d97bf 0000023a`02856170 0000008d`cc32f6b9 00000000`62210ae8 : nt!KiSystemServiceCopyEnd+0x25
0000008d`cc32f618 00007ffa`4b9d97bf     : 0000023a`02856170 0000008d`cc32f6b9 00000000`62210ae8 0000023a`04200dc0 : win32u!NtGdiResetDC+0x14
0000008d`cc32f620 00007ffa`4b9ffe5d     : 0000ad36`32354121 00000000`00000000 00000000`00000000 0000023a`00000001 : gdi32full!ResetDCWInternal+0x16b
0000008d`cc32f720 00007ff7`c10d16b3     : 0000023a`02839c00 00000000`00000000 00000000`00000000 00000000`00000000 : gdi32full!ResetDCA+0x3d
0000008d`cc32f750 0000023a`02839c00     : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : Exploit!main+0x73 [D:\SelfLearn\C++Project\Exploit\Exploit\2021-40449-POC.cpp @ 236]
0000008d`cc32f758 00000000`00000000     : 00000000`00000000 00000000`00000000 00000000`00000000 00007ff7`c10d18e0 : 0x0000023a`02839c00
```

# 参考链接

- 8ffK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8Y4Z5H3d9s2j5H3y4W2W2d9L8r3#2c8g2W2y4u0e0W2c8V1x3V1S2Z5y4Y4M7.
- bacK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6G2L8r3I4&6M7s2N6F1i4K6u0r3b7$3q4D9L8r3u0S2j5$3E0t1k6h3I4D9

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2022-2-9 19:35
被Jimpp编辑
，原因：

[#漏洞分析](https://bbs.kanxue.com/forum-150-1-153.htm)
[#UAF](https://bbs.kanxue.com/forum-150-1-158.htm)
[#Windows](https://bbs.kanxue.com/forum-150-1-160.htm)
