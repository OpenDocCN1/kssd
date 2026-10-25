
# Themida VM 代码还原

> 原文：[`www.pediy.com/kssd/pediy10/82265.html`](https://www.pediy.com/kssd/pediy10/82265.html)

## 文章信息

- 【文章标题】: Themida VM 代码还原
- 【文章作者】: hawk
- 【作者邮箱】: hkp2000hk@sina.com
- 【作者主页】: http://localhost
- 【作者 QQ 号】: 99224775
- 【软件名称】: Themida.不知道
- 【下载地址】: 自己搜索下载
- 【保护方式】: 虚拟机
- 【使用工具】: 老三样
- 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

## 详细过程

### 虚拟机分析

IDA 分析目标发现虚拟机，应该是 Themida CISC 版本及加密参数不详。

虚拟机入口 `0x1EAE4`，`ContextBase` `0x1E800` `MethodTableSize` `0xA8`

首先 IDA 手工修复代码，虚拟机主体为一函数，160+Method 分别建立函数。

把 `0x1EB4E` 主循环扒下来

```
//----------------------------------------------------//
lodsb
xor     al, bl
push    7BB9h
mov     [esp], edx
mov     dh, 28h
jmp     loc_22604
shl     dh, 6
push    ebx
mov     bl, 4Eh
sub     bl, 11h
sub     bl, 0EAh
xor     dh, bl
pop     ebx
jmp     loc_1F55B
xor     al, dh
jmp     loc_23352
mov     edx, [esp]
add     esp, 4
push    ecx
jmp     loc_1FF5D
push    ebx
jmp     loc_1F06D
mov     bh, 6Eh
add     bh, 0DDh
jmp     loc_20342
neg     bh
shl     bh, 2
sub     bh, 1
not     bh
xor     bh, 0B3h
mov     ch, bh
pop     ebx
jmp     loc_20878
sub     al, ch
mov     ecx, [esp]
jmp     loc_20CB2
add     esp, 4
sub     bl, 7Eh
sub     bl, 2Ah
sub     bl, al
jmp     loc_21116
push    ecx
mov     cl, 2Ah
add     bl, cl
jmp     loc_22287
pop     ecx
push    edx
mov     dl, 0C3h
xor     dl, 84h
jmp     loc_22B10
sub     dl, 0DBh
jmp     loc_2273F
sub     dl, 0F7h
xor     dl, 0Bh
jmp     loc_1F6D9
add     bl, dl
pop     edx
movzx   eax, al
jmp     dword ptr [edi+eax*4]
//-------------------修复后的代码--------------------//
LODSB
XOR     AL, BL
XOR  AL, 53H
SUB  AL, 9FH
SUB     BL, AL
MOVZX   EAX, AL
JMP     DWORD PTR [EDI+EAX*4]
//---------------------------------------------------//
```

### 逆向工程实现

VS2008 新建工程

```
ref  class  _VM_Analyze
{
  public:
    static  String^  GetHexString(UInt32 HexValue,Int32 Size);
    UInt32  ImageBase;
    array<Byte>  ^ImageData;
    UInt32  ContextBase;
    UInt32  ContextKey;
    UInt32  CodePointer;
    StringWriter  ^AnalyzeResult;
    Byte  ReadByte();
    UInt16  ReadWord();
    UInt32  ReadDword();
    Boolean  Decode();
    Void  LoadImage(String^ ImagePath);
    _VM_Analyze(UInt32 Key);
};

Boolean  _VM_Analyze::Decode()
{
    /*
    LODSB
    XOR     AL, BL
    XOR     AL, 53H
    SUB     AL, 9FH
    SUB     BL, AL
    MOVZX   EAX, AL
    JMP     DWORD PTR [EDI+EAX*4]
    */
    Byte  Opcode=ReadByte();
    Byte  Key=ContextKey;
    Opcode=Opcode^Key;
    Opcode=Opcode^0x53;
    Opcode=Opcode-0x9F;
    Key=Key-Opcode;
    ContextKey=ContextKey&0xFFFFFF00|Key;
    switch (Opcode)
    {
    default:
      {
        UInt32  K=ContextKey;
        UInt32  P=CodePointer;
        String  ^s=AnalyzeResult->ToString();
        throw  s;
      }
    }
}
```

OllyICE 加载目标 下断 `MainLoop` `JumpMethodTable`

VS2008 F5 抛出异常 `Opcode=0x??` `ContextKey=0x????????`

OllyICE `JumpMethodTable` 条件断 `AL==0x??`

对比 `ContextKey==EBX` 恭喜清洗代码应该没有什么 BUG

按照 `Opcode==0x??` 拔下对应的 `Method` 代码

清洗之

一般都是

```
LODS?
(ADD|SUB|???) EAX, EBX
(ADD|???????) EAX, 0x???????
(ADD|???????) EAX, 0x???????
(ADD|???????) EBX, EAX
????????
????????
JMP MainLoop
```

清洗后的代码贴到 `switch(Opcode)` 里面

```
case  0x2E:
  /*
  LODSB
  XOR     AL, BL
  ADD     AL, 0CAH
  ADD     AL, 1BH
  ADD     BL, AL
  MOVZX   EAX, AL
  LEA     EAX, [EDI+EAX*4]
  PUSH  EAX
  */
  {
    Byte  arg1=ReadByte();
    Byte  Key=ContextKey;
    arg1=arg1^Key;
    arg1=arg1+0xCA;
    arg1=arg1+0x1B;
    Key=Key+arg1;
    ContextKey=ContextKey&0xFFFFFF00|Key;
    AnalyzeResult->WriteLine(String::Format("PUSH\t{0}",GetHexString(ContextBase+arg1*4,4)));
  }
  return  true;
```

扒掉取 `arg1` 和解码更新 `Key` 的代码

剩下的代码原样输出

本来 160+ 的 `Method` 现在大概只要处理 20-40 个吧

原先的 `VMCode` 经过处理之后就成功从 CISC 映射到了 X86

```
//-----------------------------映射过的代码---------------------------//
PUSH  1E81CH
POP  EDX
POP  DWORD PTR [EDX]
PUSH  1E814H
POP  EDX
POP  DWORD PTR [EDX]
PUSH  1E800H
POP  EDX
POP  DWORD PTR [EDX]
PUSH  1E810H
POP  EDX
POP  DWORD PTR [EDX]
PUSH  1E80CH
POP  EDX
POP  DWORD PTR [EDX]
MOV  BYTE PTR [1E828H], 1
PUSH  1E80CH
POP  EDX
POP  DWORD PTR [EDX]
PUSH  1E808H
POP  EDX
POP  DWORD PTR [EDX]
PUSH  1E804H
POP  EDX
POP  DWORD PTR [EDX]
PUSH  1E818H
POP  EDX
POP  DWORD PTR [EDX]
MOV  EDX, ESP
PUSH  EDX
PUSH  4
POP  EAX
ADD  [ESP], EAX
POP  ESP
PUSH  1E810H
POP  EDX
PUSH  DWORD PTR [EDX]
PUSH  ESP
PUSH  1E810H
POP  EDX
PUSH  EDX
PUSH  1E804H
PUSH  1E810H
POP  EDX
POP  EDX
POP  EDX
POP  DWORD PTR [EDX]
PUSH  ESP
PUSH  14H
POP  EAX
SUB  [ESP], EAX
PUSHF
POP  DWORD PTR [1E81CH]
POP  ESP
PUSH  1E80CH
POP  EDX
PUSH  DWORD PTR [EDX]
PUSH  1E800H
POP  EDX
PUSH  DWORD PTR [EDX]
PUSH  EDX
PUSH  1E800H
POP  EDX
PUSH  DWORD PTR [EDX]
POP  EDX
PUSH  1E80CH
POP  EDX
PUSH  DWORD PTR [EDX]
POP  EDX
POP  EDX
PUSH  1E814H
PUSH  1E818H
POP  EDX
PUSH  DWORD PTR [EDX]
POP  EDX
POP  EDX
PUSH  DWORD PTR [EDX]
PUSH  1E80CH
POP  EDX
PUSH  DWORD PTR [EDX]
PUSH  1E80CH
POP  EDX
PUSH  DWORD PTR [EDX]
PUSH  1E810H
POP  EDX
PUSH  DWORD PTR [EDX]
PUSH  1E80CH
POP  EDX
PUSH  DWORD PTR [EDX]
POP  EAX
ADD  [ESP], EAX
PUSHF
POP  DWORD PTR [1E81CH]
POP  EDX
POP  EAX
XOR  [ESP], EAX
PUSHF
POP  DWORD PTR [1E81CH]
PUSH  1E80CH
POP  EDX
PUSH  EDX
PUSH  EDX
PUSH  4
POP  EAX
ADD  [ESP], EAX
POP  EDX
POP  EDX
POP  DWORD PTR [EDX]
PUSH  1E818H
POP  EDX
PUSH  DWORD PTR [EDX]
PUSH  1E818H
PUSH  1E800H
PUSH  1E814H
POP  EDX
POP  EDX
PUSH  DWORD PTR [EDX]
POP  EDX
POP  EDX
PUSH  DWORD PTR [EDX]
POP  EAX
XOR  [ESP], EAX
PUSHF
POP  DWORD PTR [1E81CH]
PUSH  1E818H
POP  EDX
PUSH  EDX
PUSH  SMALL 6B21H
PUSH  1E808H
POP  EDX
PUSH  DWORD PTR [EDX]
POP  EDX
PUSH  SMALL 9594H
POP  EDX
POP  EDX
POP  DWORD PTR [EDX]
PUSH  1E80CH
POP  EDX
PUSH  SMALL WORD PTR [EDX]
PUSH  DWORD PTR [1E810H]
POP  EDX
PUSH  EDX
PUSH  0FFFFFFF4H
POP  EAX
ADD  [ESP], EAX
POP  EDX
PUSH  EDX
PUSH  1E804H
POP  EDX
PUSH  DWORD PTR [EDX]
PUSH  SMALL 6C0EH
PUSH  SMALL 0D566H
POP  EDX
POP  EDX
POP  EDX
POP  SMALL WORD PTR [EDX]
PUSH  DWORD PTR [1E810H]
POP  EDX
PUSH  EDX
PUSH  0FFFFFFF6H
POP  EAX
ADD  [ESP], EAX
PUSH  SMALL 8E94H
PUSH  SMALL 5A8FH
POP  EDX
POP  EDX
PUSH  EDX
PUSH  EDX
PUSH  1E810H
PUSH  SMALL 0CE98H
PUSH  SMALL 152DH
POP  EDX
POP  EDX
POP  EDX
PUSH  1E814H
POP  EDX
POP  DWORD PTR [EDX]
MOV  EBX, 0
```



```
  PUSH  1E818H  
  POP  EDX  
  PUSH  EDX  
  PUSH  1E818H  
  POP  EDX  
  PUSH  DWORD PTR [EDX]  
  POP  EDX  
  POP  EDX  
  MOV  EBX, 0  
  PUSH  DWORD PTR [EDX]  
  MOV  EBX, 0  
  PUSH  EDX  
  PUSH  SMALL 0BD8FH  
  PUSH  SMALL 0C90CH  
  POP  EDX  
  POP  EDX  
  PUSH  DWORD PTR [1E814H]  
  POP  EDX  
  MOV  EBX, 0  
  POP  DWORD PTR [EDX]  
  PUSH  EDX  
  PUSH  1E808H  
  POP  EDX  
  POP  EDX  
  MOV  EBX, 0  
  JXXX  3000011H  
  JNX  23B6CH  
  //-----------------------------------------------------------//  
```

Themida 的 `VMCode` 也是经过混淆的(该死的多态变形)

一开始的代码大概意思就是

```
  POPF  
  POP  EDI  
  POP  ESI  
  POP  EBP  
  POP  ESP//VM 是堆栈机 不用 ESP 再说了 POPA 也不改 ESP  
  POP  EBX  
  POP  EDX  
  POP  ECX  
  POP  EAX  
```

搞定 VM 寄存器 地址啦

```
  #define  R_EFLAG  "1E81CH"  
  #define  R_EDI  "1E814H"  
  #define  R_ESI  "1E800H"  
  #define  R_EBP  "1E810H"  
  #define  R_ESP  "1E80CH"  
  #define  R_EBX  "1E80CH"  
  #define  R_EDX  "1E808H"  
  #define  R_ECX  "1E804H"  
  #define  R_EAX  "1E818H"  
```

```
  //------------------------------经过清洗的 X86 版虚拟代码--------------------//  
  MOV  EDX, 1E81CH  
  POP  DWORD PTR [EDX]  
  MOV  EDX, 1E814H  
  POP  DWORD PTR [EDX]  
  MOV  EDX, 1E800H  
  POP  DWORD PTR [EDX]  
  MOV  EDX, 1E810H  
  POP  DWORD PTR [EDX]  
  MOV  EDX, 1E80CH  
  POP  DWORD PTR [EDX]  
  MOV  BYTE PTR [1E828H], 1  
  MOV  EDX, 1E80CH  
  POP  DWORD PTR [EDX]  
  MOV  EDX, 1E808H  
  POP  DWORD PTR [EDX]  
  MOV  EDX, 1E804H  
  POP  DWORD PTR [EDX]  
  MOV  EDX, 1E818H  
  POP  DWORD PTR [EDX]  
  MOV  EDX, ESP  
  PUSH  EDX  
  MOV  EAX, 4  
  ADD  [ESP], EAX  
  POP  ESP  
  MOV  EDX, 1E810H  
  PUSH  DWORD PTR [EDX]  
  PUSH  ESP  
  MOV  EDX, 1E810H  
  POP  DWORD PTR [EDX]  
  PUSH  ESP  
  MOV  EAX, 14H  
  SUB  [ESP], EAX  
  PUSHF  
  POP  DWORD PTR [1E81CH]  
  POP  ESP  
  MOV  EDX, 1E80CH  
  PUSH  DWORD PTR [EDX]  
  MOV  EDX, 1E800H  
  PUSH  DWORD PTR [EDX]  
  MOV  EDX, 1E814H  
  PUSH  DWORD PTR [EDX]  
  MOV  EDX, 1E80CH  
  PUSH  DWORD PTR [EDX]  
  MOV  EDX, 1E80CH  
  PUSH  DWORD PTR [EDX]  
  MOV  EDX, 1E810H  
  PUSH  DWORD PTR [EDX]  
  MOV  EDX, 1E80CH  
  MOV  EAX, [EDX]  
  ADD  [ESP], EAX  
  PUSHF  
  POP  DWORD PTR [1E81CH]  
  POP  EDX  
  POP  EAX  
  XOR  [ESP], EAX  
  PUSHF  
  POP  DWORD PTR [1E81CH]  
  PUSH  1E80CH  
  PUSH  EDX  
  MOV  EAX, 4  
  ADD  [ESP], EAX  
  POP  EDX  
  POP  EDX  
  POP  DWORD PTR [EDX]  
  MOV  EDX, 1E818H  
  PUSH  DWORD PTR [EDX]  
  MOV  EDX, 1E818H  
  MOV  EAX, [EDX]  
  XOR  [ESP], EAX  
  PUSHF  
  POP  DWORD PTR [1E81CH]  
  PUSH  1E818H  
  PUSH  SMALL 6B21H  
  MOV  EDX, [1E808H]  
  PUSH  SMALL 9594H  
  POP  EDX  
  POP  EDX  
  POP  DWORD PTR [EDX]  
  MOV  EDX, 1E80CH  
  PUSH  SMALL WORD PTR [EDX]  
  PUSH  DWORD PTR [1E810H]  
  MOV  EAX, 0FFFFFFF4H  
  ADD  [ESP], EAX  
  POP  EDX  
  POP  SMALL WORD PTR [EDX]  
  PUSH  DWORD PTR [1E810H]  
  MOV  EAX, 0FFFFFFF6H  
  ADD  [ESP], EAX  
  MOV  EDX, 8E945A8FH  
  MOV  EDX, 1E814H  
  POP  DWORD PTR [EDX]  
  MOV  EDX, 1E818H  
  PUSH  DWORD PTR [EDX]  
  MOV  EDX, [1E814H]  
  POP  DWORD PTR [EDX]  
  JXXX  3000011H  
  JNX  23B6CH  
```

现在吧 `X86` 代码空间的虚拟代码在映射一把

吧 `Register` 也映射回来

```
  //----------------------------------真正的 X86 版代码--------------------------------//  
  POPF  
  POP  EDI  
  POP  ESI  
  POP  EBP  
  POP  EBX  
  MOV  BYTE PTR [1E828H], 1  
  POP  EBX  
  POP  EDX  
  POP  ECX  
  POP  EAX  
  PUSH  ESP  
  ADD  DWORD PTR [ESP], 4  
  POP  ESP  
  PUSH  EBP  
  PUSH  ESP  
  POP  EBP  
  PUSH  ESP  
  SUB  DWORD PTR [ESP], 14H  
  PUSHF  
  POPF  
  POP  ESP  
  PUSH  EBX  
  PUSH  ESI  
  PUSH  EDI  
  PUSH  EBX  
  XOR  [ESP], EBX  
  PUSHF  
  POPF  
  POP  EBX  
  PUSH  EAX  
  XOR  [ESP], EAX  
  PUSHF  
  POPF  
  POP  EAX  
  PUSH  BX  
  POP  SMALL WORD PTR [EBP+0FFFFFFF4H]  
  PUSH  EBP  
  ADD  DWORD PTR [ESP], 0FFFFFFF6H  
  POP  EDI  
  PUSH  EAX  
  POP  DWORD PTR [EDI]  
  JXXX  3000011H  
  JNX  23B6CH  
```

再次清洗一边

```
  //-------------------------------------最终的成品面世啦--------------------------------//  
  POPF  
  POP  EDI  
  POP  ESI  
  POP  EBP  
  POP  EBX  
  MOV  BYTE PTR [1E828H], 1  
  POP  EBX  
  POP  EDX  
  POP  ECX  
  POP  EAX  
  ADD    ESP, 4  
  PUSH  EBP  
  MOV    EBP, ESP  
  SUB    ESP, 14H  
  PUSH  EBX  
  PUSH  ESI  
  PUSH  EDI  
  XOR    EBX, EBX  
  XOR    EAX, EAX  
  MOV  [EBP-0CH], BX  
  LEA  EDI, [EBP-0AH]  
  MOV    [EDI], EAX  
  JXXX  3000011H  
  JNX  23B6CH  
```

至此全部工序就此结束

```
  //-----------------------------------------------------------------------------------------------//  
```

### 【经验总结】

虚拟机的本质其实就是代码的多态变形，难点也就是对多态代码的归一化处理，从明文到密文是单向和确定的，

但是从密文到明文却是多项和不确定的。规则复杂的吓人，盼望会玩太极的牛人快快写一工具出来救广大人民于水火。

### 【版权声明】

本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2009 年 02 月 17 日 22:18:27
