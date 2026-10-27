
## 汇编代码分析

```
8053e633 c1e902          shr     ecx,2                                                                        //ecx = 参数个数

8053e636 8bfc            mov     edi,esp                                                        //edi-->当前esp

8053e638 3b35549a5580    cmp     esi,dword ptr [nt!MmUserProbeAddress (80559a54)]

8053e63e 0f83a8010000    jae     nt!KiSystemCallExit2+0x9f (8053e7ec)                //if(esi >7fff0000 )(跳走) 检查esi是否r3地址

8053e644 f3a5            rep movs dword ptr es:[edi],dword ptr [esi]                        //复制r3参数

8053e646 ffd3            call    ebx                                                                        //调用系统服务函数

8053e648 8be5            mov     esp,ebp

8053e64a 8b0d24f1dfff    mov     ecx,dword ptr ds:[0FFDFF124h]                //ecx-->CurrentThread

8053e650 8b553c          mov     edx,dword ptr [ebp+3Ch]                                //edx=trap_frame.edx 0xffffffff

8053e653 899134010000    mov     dword ptr [ecx+134h],edx                                //Kthread.Trapframe = 0xfffffffff
```

### nt!KiServiceExit

```
8053e659 fa              cli

8053e65a f7457000000200  test    dword ptr [ebp+70h],20000h                                //trap_frame.eflag测试划是否虚拟8086模式

8053e661 7506            jne     nt!KiServiceExit+0x10 (8053e669)                //是则跳走

8053e663 f6456c01        test    byte ptr [ebp+6Ch],1                                                //trap_frame.SegCs 是否为r3　选择子

8053e667 7457            je      nt!KiServiceExit+0x67 (8053e6c0)                //不是则跳走

8053e669 8b1d24f1dfff    mov     ebx,dword ptr ds:[0FFDFF124h]                //ebx-->CurrentThread

8053e66f c6432e00        mov     byte ptr [ebx+2Eh],0                                        //kthread.afterd= 0 清除＂警觉＂　apc 相关

8053e673 807b4a00        cmp     byte ptr [ebx+4Ah],0                                        //kthread．KAPC_STATE.UserApcPending

8053e677 7447            je      nt!KiServiceExit+0x67 (8053e6c0)                //没有apc挂起
..........
```

```
8053e6c0 8b54244c        mov     edx,dword ptr [esp+4Ch]                                 //edx = ktrap_frame.seh链

8053e6c4 648b1d50000000  mov     ebx,dword ptr fs:[50h]                                        //ebx = kpcr.DebugActive 0

8053e6cb 64891500000000  mov     dword ptr fs:[0],edx                                        //kpcr.tib.ExceptionList = edx 还原线程seh

8053e6d2 8b4c2448        mov     ecx,dword ptr [esp+48h]                                //ecx = ktrap_frame.PreviousPreviousMode

8053e6d6 648b3524010000  mov     esi,dword ptr fs:[124h]                                //esi-->CurrentThread

8053e6dd 888e40010000    mov     byte ptr [esi+140h],cl                                        //kthread.PreviousMode = ecx 还原先前模式

8053e6e3 f7c3ff000000    test    ebx,0FFh                                                                //当前线程是否在调试

8053e6e9 7579            jne     nt!KiSystemCallExit2+0x17 (8053e764)        //是，则跳走

8053e6eb f744247000000200 test    dword ptr [esp+70h],20000h                                //trap_frame.eflag测试划是否虚拟8086模式

8053e6f3 0f85eb080000    jne     nt!KiExceptionExit+0x12c (8053efe4)                //是，则跳走

8053e6f9 66f744246cf8ff  test    word ptr [esp+6Ch],0FFF8h                                        //trap_frame.cs 高１３位

8053e700 0f84b4000000    je      nt!KiSystemCallExit2+0x6d (8053e7ba)         //为0 则跳走

8053e706 66837c246c1b    cmp     word ptr [esp+6Ch],1Bh

8053e70c 660fba64246c00  bt      word ptr [esp+6Ch],0

8053e713 f5              cmc

8053e714 0f878e000000    ja      nt!KiSystemCallExit2+0x5b (8053e7a8)

8053e71a 66837d6c08      cmp     word ptr [ebp+6Ch],8

8053e71f 7405            je      nt!KiServiceExit+0xcd (8053e726)

8053e721 8d6550          lea     esp,[ebp+50h]

8053e724 0fa1            pop     fs                                                                        //还原r3 ss

8053e726 8d6554          lea     esp,[ebp+54h]                                                        //一路还原.....

8053e729 5f              pop     edi

8053e72a 5e              pop     esi

8053e72b 5b              pop     ebx

8053e72c 5d              pop     ebp

8053e72d 66817c24088000  cmp     word ptr [esp+8],80h                                        //检查trap_frame.cs　选择子的合法性

8053e734 0f87c6080000    ja      nt!KiExceptionExit+0x148 (8053f000)

8053e73a 83c404          add     esp,4                                                                //跳过trap_errcode

8053e73d f744240401000000 test    dword ptr [esp+4],1                                                // ss:0010:f959fdd0=0000001b　还是检查
```

### nt!KiSystemCallExitBranch

```
8053e745 7506            jne     nt!KiSystemCallExit2 (8053e74d)                        //trap_frame.cs是r3 选择子则跳走
......
```

### nt!KiSystemCallExit2

```
8053e74d f644240901      test    byte ptr [esp+9],1                                                //trap_frame.tf 位是否为 １ 单步状态

8053e752 75f8            jne     nt!KiSystemCallExit (8053e74c)                        //单步状态则跳走

8053e754 5a              pop     edx                                                                        //edx = tramp_frame.eip

8053e755 83c404          add     esp,4

8053e758 80642401fd      and     byte ptr [esp+1],0FDh                                        //1111 1101  清掉trap_frame.eflag 位２

8053e75d 9d              popfd                                                                                //还原eflag

8053e75e 59              pop     ecx                                                                        //ecx = tramp_frame.esp  r3 的栈顶

8053e75f fb              sti                                                                                        //开中断

8053e760 0f35            sysexit                                                                                //退出内模式
......
```

## sysenter / sysexit 操作

sysenter 所做操作: 箭头方向表示　　源　赋值给　目标

1. `SYSENTER_CS_MSR`--> `CS` 174  （对应`KGDT_R0_CODE`）
2. `SYSENTER_CS_MSR+8`--> `SS` （对应`KGDT_R0_DATA`）
3. `SYSENTER_EIP_MSR`--> `EIP` 176
4. `SYSENTER_ESP_MSR`--> `ESP` 175

sysexit 所做操作:

1. `SYSENTER_CS_MSR+16`--> `CS` 且 rpl位 自动置3 (对应`KGDT_R3_CODE`)
2. `edx` --> `eip`
3. `SYSENTER_CS_MSR+24`--> `SS` 且rpl位置3 (对应`KGDT_R3_DATA`)
4. `ecx`--> `esp`

## 引起反思

从这这两个指令操作,就可以想像为什么GDT中前几项表项的是那样的顺序!!

[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)

上传的附件：

- [protect1.png](javascript:void(0);)

  （106.02kb，963次下载）

- [protect2.png](javascript:void(0);)

  （15.53kb，951次下载）

![](https://bbs.kanxue.com/upload/attach/201112/242673_zzlzve8z0y1v7ml.png)
