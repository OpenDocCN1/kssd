
# Crunch v1.1

> 原文：[`www.pediy.com/kssd/pediy03/forum797.htm`](https://www.pediy.com/kssd/pediy03/forum797.htm)

欢迎有空光临小站：`http://go1.163.com/~st0ne/`

『技术研习』 ‘以前更新’

--------------------------------------------------------------------------------

## Crunch v1.1 中的一段有趣的反跟踪代码

### 【声明】

我写文章以交流为主，希望大家在转载时能保持文章的完整性。

### 【前言】

这是在对 `Crunch.EXE` 脱壳时发现的一点有趣的东西，就写了出来。其中利用一种特殊的方法---修改 LDT 表来进入 Ring0 级，然后在 Ring0 级检测 `Test_Debug_Installed` 标志来判断当前是否有调试器跟踪，这种反跟踪方法不用区别调试器的类型。`SoftICE`、`TRW2000`、天意都可以检测到，怎么样？有点意思吧。不过如果你要对付这种外壳加密软件的话，就要小心一点了，别被当得死死的哦。:-)

作者： ljttt  
写作日期： 2000-10-02

### 1、前面的大部分分析过程

前面的大部分分析过程同《对 Crunch v1.1 加壳程序的手动脱壳及反跟踪代码的一点分析》一文中所述，这里就不多说了。

### 2、跟踪到如下代码处，我们的目标开始了！

```asm
015F:0066C266 53                PUSH      EBX
015F:0066C267 E8C9100000        CALL      0066D335
015F:0066C26C 5B                POP       EBX
015F:0066C26D 899D683C0000      MOV       [EBP+00003C68],EBX
015F:0066C273 BB76060000        MOV       EBX,00000676
015F:0066C278 03DD              ADD       EBX,EBP
015F:0066C27A 899D643C0000      MOV       [EBP+00003C64],EBX
015F:0066C280 BB050A0000        MOV       EBX,00000A05
015F:0066C285 03DD              ADD       EBX,EBP
015F:0066C287 FFD3              CALL      EBX                                   <--内有反跟踪代码
015F:0066C289 90                NOP
015F:0066C28A 60                PUSHAD
015F:0066C28B BB85230000        MOV       EBX,00002385
015F:0066C290 03DD              ADD       EBX,EBP
015F:0066C292 53                PUSH      EBX
015F:0066C293 FF950A210000      CALL      [EBP+0000210A]
015F:0066C299 8985F6160000      MOV       [EBP+000016F6],EAX
```

### 3、在 CALL EBX 处按 F8 键进入看看

```asm
015F:0066C9FF FFA56C3C0000      JMP       [EBP+00003C6C]
015F:0066CA05 8CD8              MOV       AX,DS
015F:0066CA07 A804              TEST      AL,04
015F:0066CA09 740F              JZ        0066CA1A
015F:0066CA0B 8DB51B0A0000      LEA       ESI,[EBP+00000A1B]               <--ESI=0066CA1B（这是检测程序入口的地址
015F:0066CA11 BB460A0000        MOV       EBX,00000A46
015F:0066CA16 03DD              ADD       EBX,EBP                         <--EBX
015F:0066CA18 FFD3              CALL      EBX                             <--还要继续深入
015F:0066CA1A C3                RET
```

### 4、在 CALL EBX 处我们继续按 F8 键进入看看，这样我们就到了关键处

```asm
015F:0066CA46 60                PUSHAD
015F:0066CA47 53                PUSH      EBX                   <--这里实际上是为了留出空间保存 GDTR 的内容
015F:0066CA48 0F014424FE        SGDT      FWORD PTR [ESP-02]    <--获取 GDTR 的内容（6 个字节）
015F:0066CA4D 5B                POP       EBX                   <--EBX=GDTR 的 4 个字节的内容，实际上就是 GDT 的基地址
015F:0066CA4E 33C0              XOR       EAX,EAX               <--EAX=0
015F:0066CA50 0F00C0            SLDT      AX                    <--获取 LDT 表的选择器到 AX 中
015F:0066CA53 24F8              AND       AL,F8                 <--限定 AL 大小
015F:0066CA55 03D8              ADD       EBX,EAX               <--获取 LDT 表在 GDT 表中的偏移地址
015F:0066CA57 8A6B07            MOV       CH,[EBX+07]           <--获取第 8 个字节
015F:0066CA5A 8A4B04            MOV       CL,[EBX+04]           <--获取第 5 个字节
015F:0066CA5D C1E110            SHL       ECX,10                <--移动到高位
015F:0066CA60 668B4B02          MOV       CX,[EBX+02]           <--获取第 3-4 两个字节，此时 ECX 组合成为 LDT 表的基地址
015F:0066CA64 8D7908            LEA       EDI,[ECX+08]          <--获取 LDT 表第 9 个字节偏移地址指针
015F:0066CA67 FC                CLD
015F:0066CA68 8BC6              MOV       EAX,ESI               <--EAX=ESI=0066CA1B
015F:0066CA6A 66AB              STOSW                           <--保存两个字节，即 CA1B 到 EDI 指向的地址
015F:0066CA6C B8280000EC        MOV       EAX,EC000028          <--EAX=EC000028
015F:0066CA71 AB                STOSD                           <--保存四个字节，即 EC000028 到 EDI+2 指向的地址
015F:0066CA72 0FA4F010          SHLD      EAX,ESI,10            <--把 ESI=0066CA1B 进行移位，得到 0066
015F:0066CA76 66AB              STOSW                           <--保存两个字节，即 0066 到 EDI+6 指向的地址
015F:0066CA78 61                POPAD                             <--以上这些指令为了形成 LDTR，使之指向 0066CA1B，为以下的调用作准备
015F:0066CA79 9A000000000F00    CALL      000F:00000000         <--该调用实际上指向的 015F:0066CA1B 处
015F:0066CA80 C3                RET
```

### 5、我们在 CALL 000F:00000000 处按 F8 键继续跟踪分析

```asm
015F:0066CA1B 60                PUSHAD
015F:0066CA1C 33C0              XOR       EAX,EAX
015F:0066CA1E BF3B0A0000        MOV       EDI,00000A3B
015F:0066CA23 03FD              ADD       EDI,EBP
015F:0066CA25 CD20              INT       20 VXDCall Get_DDB                  <--判断 DDB
015F:0066CA2B 7502              JNZ       0066CA2F                          <--关键了！
015F:0066CA2D CD19              INT       19                                  <--超度
015F:0066CA2F CD20              INT       20 VXDCall Test_Debug_Installed     <--判断是否安装了调试环境
015F:0066CA35 7402              JZ        0066CA39                          <--关键了！
015F:0066CA37 CD19              INT       19
015F:0066CA39 61                POPAD
015F:0066CA3A CB                RETF
```

### 【后记】

这种检测跟踪的方法比较新鲜（对我来说 ^\_^）。我想这种方法可以检测调试环境，比单纯的检测 `SoftICE` 跟踪来得更有意思。

--------------------------------------------------------------------------------

《加密及解密技术交流站》由 ljtt 制作 版权所有

&copy;2000 -2001 All Rights Reserved

转载本站所有文章请注明出处，尊重作者的劳动也是尊重你自己。让我们一同撑起绿色的天空！ 4dd466d0e1e0cc123afbe504da2462ca
