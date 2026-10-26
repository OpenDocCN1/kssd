
```
debug032:0106DB8D                                                 ; edi - pProcName
debug032:0106DB92 8B 7D E8                mov     edi, [ebp+var_18]
debug032:0106DB95 8B 45 F4                mov     eax, [ebp+var_C]
```

---

## 在 `debug032:0106E474` 处中断获取所有AIP位置的脚本

---

``` 
#include "idc.idc"

#define  FstVirAddr  0x00401000

#define  magic1offset   0x09

static main(void)

{
  auto haip, i,j, obj, AIPNum, Magic2, pAIPTable, itemsize;

  obj = GetRegValue("ebx");

  AIPNum = Dword(obj+0x18);

  Magic2 = Dword(obj+0xE0);

  itemsize = Dword(obj+0xE4);

  haip = fopen("d:\\aip.txt","w");

  pAIPTable = GetRegValue("esi");

  for(i=0;i<AIPNum;i++)

  {

    writelong(haip,Dword(pAIPTable+i*itemsize+magic1offset)+Magic2+FstVirAddr,0);

  }

  // ending sign

  writelong(haip,BADADDR,0);

  fclose("haip");

  return;

}
```

---

## 获取到的AIP地址表局部

``` 
Offset      0  1  2  3  4  5  6  7   8  9  A  B  C  D  E  F
00000000   28 12 40 00 30 12 40 00  38 12 40 00 40 12 40 00   (.@.0.@.8.@.@.@.
00000010   60 12 40 00 78 12 40 00  90 12 40 00 A0 12 40 00   `.@.x.@.?@.?@.
00000020   A8 12 40 00 B8 12 40 00  C0 12 40 00 D8 12 40 00   ?@.?@.?@.?@.
00000030   E0 12 40 00 F0 12 40 00  F8 12 40 00 00 13 40 00   ?@.?@.?@...@.
00000040   20 13 40 00 30 13 40 00  38 13 40 00 40 13 40 00    .@.0.@.8.@.@.@.
```

---

程序中大概有几十处左右无法用这种方法获取函数名字：向堆栈中填入调用地址+0x05，调用`sub_2090000`。它们或是被偷代码，或是ESSP。这没有什么快捷的方法，花点功夫，只是一个体力活。

在处理ESSP时，一个比较有意思的例子是`comctl32.dll`中的`InitCommonControls()`，因为它代码实现就一句指令`mov eax, eax`，所以跳到堆中又立即返回调用处继续执行。我差一点将其`nop`掉。

原程序会注册不同窗口组件的回调函数，在这些函数中进行检测AIP处的操作码是否为E8。这些地方在原程序编写时就已经嵌入，没有现成的表可以查询它们的位置。但有机器码特征可以查询。不需要程序出错时，一处一处修改解决。

### 位置

1. `004DF054` `004DEF68`, `FOnCreate()`
2. `004E5710` (E8: `4e5727`) `004E565C`, `OnActivate()`
3. `004E220C` `004E2198`, `FOnClick()` Help -> About
4. `004E2C4F`
5. `004E27F5`
6. `004E250F`(E8: `4e2528`)
7. E8: `4df523`
8. `4E0aa3`(`4e0ac5`)
9. `4e1546`(E8: `4e1564`)
10. `4e5ec4`(E8: `4e5ee6`)

---

## 检查代码段的原型

``` 
Sub_checkE8 proc
push edx
push edi
mov edx, offset Fun
push [edx]
pop edx
movzx edi, dl            ; 正常情况下应该是E8
lea edi, [edi-0D69B23Eh]
mov edx,0D69B156h
add edi, edx
test edi, edi
jz GoodBoy

BadBoy:
…     ;  Do some crazy thing

GoodBoy:
Pop edi
Pop edx
Ret
Sub_checkE8 endp
```

---

## 获取程序中E8检测处

``` 
#include "idc.idc"

static main(void)

{
  auto i,j,he8;

  he8 = fopen("d:\\e8.txt","w");

  if ( !he8 )

  {
    Message("file not open");

    return;

  }

  for(i=0x401000;i<0x401000+0x1ee000;i++)

  {

    if( (Dword(i) == 0x000000E9) && (Byte(i+4)==0x0) )

    {

      if ( ((Byte(i+5)&0xf0) == 0x50)&&((Byte(i+6)&0xf0) ==

0x50) )

      {

        if ( Byte(i+7) == 0xE9 )

          fprintf(he8,"E8: %x\r\n",i);

      }

    }

  }

  fclose(he8);

  return;

}
```

---

## [obfuscatee]

Asprotect混淆了一些Delphi库函数。

程序中被混淆的库函数，入口处有两种类型的代码指令，如下

``` 
68 00 00 23 02          push    2230000h
C3                      retn

E9 F7 69 E3 01          jmp     near ptr 2240000h
```

重建这些指令段的任务在`Fun5()`中完成。

### 填充第一种类型

``` 
debug032:0106EA10 C6 03 68                mov     byte ptr [ebx], 68h
debug032:0106EA13 8D 43 01                lea     eax, [ebx+1]
debug032:0106EA16 8B 55 F8                mov     edx, [ebp+var_8]
debug032:0106EA19 89 10                   mov     [eax], edx
debug032:0106EA1B 83 C3 05                add     ebx, 5
debug032:0106EA1E C6 03 C3                mov     byte ptr [ebx], 0C3h
```

### 填充第二种类型

``` 
debug032:0106EA23 8B 45 F8                mov     eax, [ebp+var_8]
debug032:0106EA26 2B C3                   sub     eax, ebx
debug032:0106EA28 83 E8 05                sub     eax, 5
debug032:0106EA2B 8D 53 01                lea     edx, [ebx+1]
debug032:0106EA2E 89 02                   mov     [edx], eax
debug032:0106EA30 C6 03 E9                mov     byte ptr [ebx], 0E9h
```

### 原程序中一共有25处

``` 
debug032:01081BB8 00 00 23 02       MemBlock_LenA4 dd 2230000h
debug032:01081BBC 00 00 24 02       dd offset unk_2240000
debug032:01081BC0 00 00 25 02       dd offset unk_2250000
debug032:01081BC4 00 00 26 02       dd offset unk_2260000
debug032:01081BC8 00 00 27 02       dd offset unk_2270000
debug032:01081BCC 00 00 28 02       dd offset unk_2280000
debug032:01081BD0 00 00 29 02       dd offset unk_2290000
debug032:01081BD4 00 00 2A 02       dd offset unk_22A0000
debug032:01081BD8 00 00 2B 02       dd offset unk_22B0000
debug032:01081BDC 00 00 2C 02       dd offset unk_22C0000
debug032:01081BE0 00 00 2D 02       dd offset unk_22D0000
debug032:01081BE4 00 00 2E 02       dd offset unk_22E0000
debug032:01081BE8 00 00 2F 02       dd offset unk_22F0000
debug032:01081BEC 00 00 30 02       dd offset unk_2300000
debug032:01081BF0 00 00 31 02       dd offset unk_2310000
```

对付这种方法，可以还原或补节的方法来完成。很简单很无聊，不用多讲。

获取原程序中被混淆处的地址：在`Fun5()`中合适位置下断点，或在原程序中遍历。

### 在原程序中遍历实现

``` 
#include "idc.idc"

#define   MinVirMem     0x2230000
#define   MaxVirMem     0x23b0000

static main(void)

{
  auto i,j;

  for(i=0x401000;i<=0x401000+0xF8000;i++)

  {

    if ( Byte(i) == 0xE9 )

    {

      if ( ((Dword(i+1)+i+5) <= MaxVirMem) && ((Dword(i+1)+i+5) >= MinVirMem) ){

        Message("er:%x,ee:%x\r\n", i, Dword(i+1)+i+5);

        i = i+4;

      }

    }

    else if ( (Byte(i) == 0x68) && (Byte(i+5) == 0xC3) )

    {

       if ( (Dword(i+1) >= MinVirMem) && (Dword(i+1)<=MaxVirMem)  ){

         Message("er:%x,ee:%x\r\n",i, Dword(i+1));

         i = i+5;

       }

    }

  }

  return;

}
```

---

## [emulator]

还原虚拟代码，人工是不可能完成的任务，只有依靠脚本的力量。请参见附件。

题外话：如果虚拟代码也按照补区段的方法去完成，还有意义吗？那不正达到壳隐藏信息的目的了吗？

`protector.dll`中有两种虚拟机，一种模拟自己代码段，另一种模拟原程序的代码段。前一种中规中矩，后一种很黄很暴力：操作码全部换成伪操作码，在VM中又嵌入第二层VM。这第二层VM模拟了`mov`、`sub`、`add`、`cmp`一共四种类型的指令，但没有实现带有SIB字节的指令和`reg8`为目标操作数的指令。这里的指令类型不同于具体的操作码，比如`Inc`操作码，`Dec`操作码都可以分别模拟成`add`、`sub`类型指令。

VM的工作主要由两个函数来完成，`Executor()`执行已经被模拟化的代码段（函数），`Emulator()`模拟化代码段。

如果你在程序中发现这样的代码段，这样的代码段就是被虚拟机模拟执行。

``` 
68 00 00 00 00          push    0
68 XX XX XX XX          push    offset Emulatee
68 XX XX XX XX          push    offset EmulRec
E8 XX XX XX XX          call    Executor
```

虚拟器一共有三个重要的结构：`AsEmulIns`, `AsEmulInsHdr`, `AsEmulContext`。

不同的`Aspr`版本，三个结构会有很大的变化：伪opcode和真实opcode的换算关系、同一意义数据域有不同的偏移量（同一偏移量处的数域有不同的意义）、...

被模拟的代码段由一个`AsEmulInsHdr`和若干`AsEmulInsItem`来表示，每一段代码行用一个`AsEmulInsItem`来表示。

当前CPU的执行状态用`AsEmulContext`来表示，虚拟机运行的结果反映在`AsEmulContext`各个域的变化上。

模拟器按照操作码，分情况来处理。如

`push ebp(55)`的处理函数流： `Op5X() -> Push()`

`pop ebp(5C)`的处理函数流：`Op5X() -> Pop()`

我所见到的这个版本的VM没有实现浮点运算指令和`0F`为前缀码的大部分指令，不知道原因是demo版本的限制，还是模拟器仍在进化中。

### 取操作码 Opcode，跳转

``` 
debug032:0106B269 8A 83 B4 00 00 00       mov     al, [ebx+0B4h]
debug032:0106B26F 02 46 50                add     al, [esi+50h]
debug032:0106B272 25 FF 00 00 00          and     eax, 0FFh
debug032:0106B277 C1 E8 04                shr     eax, 4
debug032:0106B27A 83 E0 0F                and     eax, 0Fh
debug032:0106B27D 83 F8 0F                cmp     eax, 0Fh    ; switch 16 cases
debug032:0106B280 0F 87 27 01 00 00       ja      gameover
debug032:0106B286 FF 24 85 8D B2 06 01    jmp     ds:off_106B28D[eax*4] ; switch jump
```

### 取指令位

``` 
debug032:01069219 8A 9D B4 00 00 00 mov     bl, [ebp+0B4h]
debug032:0106921F 02 5E 50          add     bl, [esi+50h]
debug032:01069222 80 E3 0F          and     bl, 0Fh
```

### 判断是否是7字头的操作码（0）

``` 
80 C1 90                add     cl, 90h
80 E9 10                sub     cl, 10h
```
