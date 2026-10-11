
# 简单分析一下 HS 驱动保护 - Hook 篇

> 原文：[`www.pediy.com/kssd/pediy12/141127.html`](https://www.pediy.com/kssd/pediy12/141127.html)

这些天一直在逆向 `HS`，不过鉴于自己蹩脚的汇编水平，以及等于零的逆向经验，以至于十多天时间过去了仍没有把握 `HS` 的系统流程。

但是细节之处却发现许多值得我们学习借鉴的技术，今天抽出点时间来写一点，与大家共同分享。

今天我们主要看一下 `HS` 对于内核钩子的处理方式。

想要将几个流程函数剥离出来而不掺杂其他的不相干的杂质是困难的，因为其内部都有着千丝万缕的联系。

在这里我选择了 `HS` 对于 `shadow ssdt` 的处理为例，相对来说这里的封装更纯净一些。

`HS` 内部维护了很多全局结构数组以及全局链表，逆向过程中主要就是与它们在打交道，所以掌握了这些结构的定义形式，很多问题都能迎刃而解。

我们看一下 `shadow ssdt` 所对应的全局结构：

```
struct{
  ULONG: FunctionNameAddress;      //+0x00 函数名地址
  ULONG: FunctionIndex;            //+0x04 函数索引
  ULONG: ProxyAddress;             //+0x08 代理函数地址
  ULONG: PreAddress;               //+0x0C hook 之前的地址
  ULONG: HasBeenHooked;            //+0x10  是否已经安装了 hook
  ULONG: Flag;                     //+0x14
}shadow_ssdt_hook_info

struct{
  ULONG: FunctionName:             //函数名称字串指针
  ULONG: PreAddress;               //hook 之前的地址
  PVOID: BufferForJmpCode;         //跳转指令的地址
  BYTE : HasBeenHooked;           //+0xC 是否已经 Hook 的标志
  BYTE : IsHookCallInstruction;    //+0xD 是否在 call 指令处进行 Hook
  ULONG: unknow1;                 //+0x10
  ULONG: unknow2;                 //+0x14
  ULONG: FunctionFlag;            //+0x18
  ULONG: unknow3;                 //+0x1C
}shadow_ssdt_hook_info2
```

上面的结构对应于直接挂钩 `shadow ssdt` 表，下面的结构对应于在 `shadow ssdt` 函数中做 `inline Hook`。

<对于 `ssdt` 的处理，内部也维护了类似的两张表>

看一下主要的流程函数：

![](img/c453121b07bf82b0170a500d9173482a.jpg)

主要的两个调用分别完成了对于指定 `shadow ssdt` 的两种 `Hook`。

另外这里我们也可以看到 `HS` 对于锁的应用，像这样的应用在 `HS` 中随处可见。

有时间的话再将 `HS` 中使用的各种锁单独提取出来与大家共同学习，因为我觉得这对于我们平日的编程特别是安全性与稳定性上有莫大的好处，而网上的相关介绍却很少。

直接的 `SSDT Hook` 很简单，就不说了，我们看一下 `inline Hook`。

想一想我们平时进行的 `inline Hook`，一般步骤如下：

1. 定位到要进行 `Hook` 的地址处
2. `nop` 掉 5 个或者更多的字节，写入 `jmp` 指令调转到我们的代理函数
3. 执行原指令，平衡堆栈等一列操作..
4. 跳转回 `nop` 之后的地址处继续执行

`HS` 也是这样做的，不过它的跳转都是先跳转到一个全局缓冲，这个全局缓冲是一个中间跳，再由它跳转到代理函数。

看一下 `HS` 是怎么做的。

`ShadowSSDT_Hook_Inline` 只是一个包装函数，过滤了一下全局变量，然后调用 `Hook_InlineDo` 进行 `inline Hook`。

代码：



### Hook_InlineDo 汇编代码

```
Hook_InlineDo proc near

OpcodeLength      = dword ptr -2Ch
JmpCode           = dword ptr -28h
JmpCode_          = dword ptr -24h
temp              = dword ptr -20h
WriteAble         = dword ptr -1Ch
TotalDecodedLength    = dword ptr -14h
PreAddressIp      = dword ptr -10h
DecodedLength     = dword ptr -0Ch
JmpCodeDump       = dword ptr -8
JmpCodeDump_      = dword ptr -4

JmpBackBufferAddress  = dword ptr  8    //回跳的缓冲地址
PreAddress        = dword ptr  0Ch  //进行 Hook 的起始地址
JmpToProxyBufferAddress  = dword ptr  10h  //跳至代理函数的缓冲地址
IsHookCallInstruction  = byte ptr  14h    //是否是对 call 指令进行 Hook
ReturnValue       = dword ptr  18h

       push    ebp
       mov     ebp, esp
       sub     esp, 2Ch
       push    esi
       push    edi
       mov     byte ptr [ebp+JmpCode], 90h
       mov     byte ptr [ebp+JmpCode+1], 90h
       mov     byte ptr [ebp+JmpCode+2], 90h
       mov     byte ptr [ebp+JmpCode+3], 90h
       mov     byte ptr [ebp+JmpCode_], 90h
       mov     byte ptr [ebp+JmpCode_+1], 90h
       mov     byte ptr [ebp+JmpCode_+2], 90h
       mov     byte ptr [ebp+JmpCode_+3], 90h
       mov     [ebp+OpcodeLength], 0
       mov     [ebp+TotalDecodedLength], 0
       mov     [ebp+DecodedLength], 0
       push    0
       mov     eax, [ebp+PreAddress]
       push    eax
       call    CheckFunctionWhetherBeenHooked     ; 这里验证一下首字节 看看有没有被 hook 掉
       mov     [ebp+PreAddress], eax              ; 如果被 hook 了 而且是绝对跳转 这里返回绝对跳转的地址
       cmp     [ebp+JmpBackBufferAddress], 0
       jz      short loc_21ACD
       cmp     [ebp+JmpToProxyBufferAddress], 0
       jz      short loc_21ACD
       cmp     [ebp+PreAddress], 0
       jnz     short loc_21AD4

loc_21ACD:                                         ; ...
       xor     al, al
       jmp     loc_21CCE
; -------------------------------------------------------------------------

loc_21AD4:                                         ; ...
       movzx   ecx, [ebp+IsHookCallInstruction]
       cmp     ecx, 1                             ; 要进行 hook 的是 call 还是别的神马
       jz      loc_21BF7
       mov     edx, [ebp+PreAddress]
       mov     [ebp+PreAddressIp], edx

loc_21AE7:                                         ; ...
       lea     eax, [ebp+OpcodeLength]
       push    eax
       mov     ecx, [ebp+PreAddressIp]
       push    ecx
       call    DisAsm
       mov     [ebp+DecodedLength], eax
       mov     edx, [ebp+TotalDecodedLength]
       add     edx, [ebp+DecodedLength]
       mov     [ebp+TotalDecodedLength], edx
       mov     eax, [ebp+PreAddressIp]
       add     eax, [ebp+DecodedLength]
       mov     [ebp+PreAddressIp], eax
       cmp     [ebp+TotalDecodedLength], 5
       jb      short loc_21AE7                    ; 确保足够 5 字节长也就是一条 jmp 指令的长度
       cmp     [ebp+TotalDecodedLength], 1Bh
       jbe     short loc_21B1C
       xor     al, al
       jmp     loc_21CCE
; -------------------------------------------------------------------------

loc_21B1C:                                         ; ...
       push    1                                  ; 对非 call 处进行 hook
       push    8
       mov     ecx, [ebp+JmpBackBufferAddress]    ; 定位到 jmp xxxx 的缓冲后面
       push    ecx
       call    MakeSpecialAddressWriteAble
       mov     [ebp+WriteAble], eax
       mov     edx, [ebp+PreAddress]
       movzx   eax, byte ptr [edx]
       cmp     eax, 0E9h
       jnz     short loc_21B6A                    ; 不是 jmp 指令的话跳转
       mov     byte ptr [ebp+JmpCode], 0E9h
       mov     ecx, [ebp+PreAddress]
       mov     edx, [ecx+1]                      ; 定位到机器码偏移
       mov     eax, [ebp+PreAddress]
       lea     ecx, [eax+edx+5]                   ; 计算绝对地址
       mov     [ebp+temp], ecx
       mov     edx, [ebp+JmpBackBufferAddress]
       add     edx, 5
       mov     eax, [ebp+temp]
       sub     eax, edx                            ; 计算偏移地址
       mov     [ebp+temp], eax
       mov     ecx, [ebp+temp]
       mov     [ebp+JmpCode+1], ecx               ; 写入到缓冲
       mov     [ebp+TotalDecodedLength], 0
       jmp     short loc_21BA0
; -------------------------------------------------------------------------

loc_21B6A:                                         ; ...
       mov     byte ptr [ebp+JmpCode], 0E9h       ; jmp 指令
       mov     edx, [ebp+PreAddress]
       add     edx, [ebp+TotalDecodedLength]      ; 回跳的目的地址
       mov     eax, [ebp+TotalDecodedLength]
       mov     ecx, [ebp+JmpBackBufferAddress]
       lea     eax, [ecx+eax+5]                   ; 目的地址 = jmp 偏移+5+当前指令地址
       sub     edx, eax                            ; jmp 偏移 = 目的地址 -(当前指令地址+5)
       mov     [ebp+temp], edx
       mov     ecx, [ebp+temp]
       mov     [ebp+JmpCode+1], ecx               ; 完成 jmp code
       mov     ecx, [ebp+TotalDecodedLength]
       mov     esi, [ebp+PreAddress]
       mov     edi, [ebp+JmpBackBufferAddress]
       mov     edx, ecx
       shr     ecx, 2
       rep movsd                                    ; 复制要执行的指令段
       mov     ecx, edx
       and     ecx, 3
       rep movsb

loc_21BA0:                                         ; ...
       mov     eax, [ebp+JmpBackBufferAddress]
       add     eax, [ebp+TotalDecodedLength]
       mov     ecx, [ebp+JmpCode]
       mov     [eax], ecx
       mov     edx, [ebp+JmpCode_]
       mov     [eax+4], edx                        ; 写入回跳的缓冲之中
       mov     eax, [ebp+WriteAble]
       push    eax
       push    8
       mov     ecx, [ebp+JmpBackBufferAddress]
       push    ecx
       call    MakeSpecialAddressWriteAble
       mov     edx, [ebp+PreAddress]
       mov     eax, [edx]
       mov     [ebp+JmpCode], eax
       mov     ecx, [edx+4]
       mov     [ebp+JmpCode_], ecx                ; 前 8 个字节
       mov     byte ptr [ebp+JmpCode], 0E9h
       mov     edx, [ebp+PreAddress]
       add     edx, 5
       mov     eax, [ebp+JmpToProxyBufferAddress]
       sub     eax, edx
       mov     [ebp+temp], eax
       mov     ecx, [ebp+temp]
       mov     [ebp+JmpCode+1], ecx
       mov     edx, [ebp+JmpCode]
       mov     [ebp+JmpCodeDump], edx
       mov     eax, [ebp+JmpCode_]
       mov     [ebp+JmpCodeDump_], eax
       jmp     loc_21C94
; -------------------------------------------------------------------------

loc_21BF7:                                         ; ...
       mov     [ebp+TotalDecodedLength], 5
       mov     ecx, [ebp+TotalDecodedLength]
       mov     esi, [ebp+PreAddress]
       lea     edi, [ebp+JmpCode]
       mov     edx, ecx
       shr     ecx, 2
       rep movsd                                    ; 复制 5 个字节 call xxxx
       mov     ecx, edx
       and     ecx, 3
       rep movsb
       mov     byte ptr [ebp+JmpCode], 0E9h       ; jmp 指令
       mov     eax, [ebp+PreAddress]
       add     eax, [ebp+TotalDecodedLength]
       add     eax, [ebp+JmpCode+1]               ; 跳转的目标地址
       mov     [ebp+temp], eax                    ; 这里是绝对地址 也就是回跳的地址
       mov     ecx, [ebp+JmpBackBufferAddress]
       add     ecx, [ebp+TotalDecodedLength]
       mov     edx, [ebp+temp]
       sub     edx, ecx                            ; 回跳的目的地址 - (指令地址+指令长度) = 偏移
       mov     [ebp+temp], edx
       mov     eax, [ebp+temp]
       mov     [ebp+JmpCode+1], eax               ; 写入缓冲
       push    1
       push    8
       mov     ecx, [ebp+JmpBackBufferAddress]
       push    ecx
       call    MakeSpecialAddressWriteAble
       mov     [ebp+WriteAble], eax
       mov     edx, [ebp+JmpBackBufferAddress]
       mov     eax, [ebp+JmpCode]
       mov     [edx], eax                         ; 写入回跳 jmp 命令
       mov     ecx, [ebp+JmpCode_]
       mov     [edx+4], ecx
       mov     edx, [ebp+WriteAble]
       push    edx
       push    8
       mov     eax, [ebp+JmpBackBufferAddress]
       push    eax
       call    MakeSpecialAddressWriteAble
       mov     ecx, [ebp+PreAddress]
       mov     edx, [ecx]
       mov     [ebp+JmpCode], edx
       mov     eax, [ecx+4]
       mov     [ebp+JmpCode_], eax
       mov     ecx, [ebp+PreAddress]
       add     ecx, 5
       mov     edx, [ebp+JmpToProxyBufferAddress]
       sub     edx, ecx
       mov     [ebp+temp], edx
       mov     eax, [ebp+temp]
       mov     [ebp+JmpCode+1], eax
       mov     ecx, [ebp+JmpCode]
       mov     [ebp+JmpCodeDump], ecx
       mov     edx, [ebp+JmpCode_]
       mov     [ebp+JmpCodeDump_], edx

loc_21C94:                                         ; ...
       push    1
       push    8
       mov     eax, [ebp+PreAddress]
       push    eax
       call    MakeSpecialAddressWriteAble
       mov     [ebp+WriteAble], eax
       mov     ecx, [ebp+JmpCodeDump_]
       push    ecx
       mov     edx, [ebp+JmpCodeDump]
       push    edx
       mov     eax, [ebp+PreAddress]
       push    eax
       call    ExInterlockedCompareExchange64Pack ;  这里正式进行 Hook
       mov     ecx, [ebp+ReturnValue]
       mov     [ecx], eax
       mov     [ecx+4], edx
       mov     edx, [ebp+WriteAble]
       push    edx
       push    8
       mov     eax, [ebp+PreAddress]
       push    eax
       call    MakeSpecialAddressWriteAble
       mov     al, 1

loc_21CCE:                                         ; ...
       pop     edi
       pop     esi
       mov     esp, ebp
       pop     ebp
       retn    14h
Hook_InlineDo endp
```



这个函数封装了两种操作,一种是在函数开头的 `inline Hook` 一种是对于函数中 `call` 指令的 `inline Hook`,我们分段看一下

![](img/3419f7269eaa29801b77b09f0c19b9e2.jpg)

首先是必要的变量初始化,以 `JmpCode` 为起始的 8 个字节都填入了 `nop` 指令<机器码 `0x90`>,

然后调用 `CheckFunctionWhetherBeenHooked` 验证了一下 `PreAddress` 参数地址起始的指令,

传参 `PreAddress` 指定了进行 Hook 的起始地址

![](img/4eb020b6d687cecaf83034df981d2406.jpg)

这个函数检测了一下起始地址是否是 `FF25` 或者 `E9` 形式的 `jmp`,也就是检测目标函数有没有被 Hook 掉,

如果正常的话返回值与传参相同,否则做相应的处理

<这里因为传入的 `Flag` 是 `0`,所以相当于只是过滤了 `FF25` 形式的 `jmp`,如果是 `FF25` 形式的 `jmp` 将返回跳转的绝对地址>

接下来是针对 `call` 指令进行 Hook 与否的跳转分支,先看一下对于非 `call` 指令的函数头 `inline Hook`:

![](img/868091919c6ca1b7e99f74e76b4f6c2f.jpg)

`HS` 内嵌了一个简易的反汇编引擎,由于我除了 `OD` 也没用过别的引擎,也没能看出来这个到底是神马

不过函数形式应该是这样的:

代码:

```
ULONG DisAsm(ULONG:Address, PULONG:OpcodeLength);
```

第一个传参为要反汇编的指令地址,第二个参数返回指令的操作码的长度,函数整体返回指令的长度

既然要对函数头进行 `inline Hook`,就要定位出足够的空间进行指令写入而且定位点不能截断在一条指令的字节之间,

所以内嵌一个反汇编引擎是非常必要的

`HS` 对于非 `call` 指令的 `inline Hook` 基本都是在函数头直接 `jmp`,所以这里需要至少定位出 5 个字节的空间

成功定位之后跳转到了 `loc_21B1C`

![](img/9b312b47dc1f19430651e60bfc50e4e4.jpg)

这个代码片段针对首条指令是否为 `jmp` 指令进行了分别的处理,结合接下来的操作一起看一下:

![](img/3d3386d5d599a111ef3b911880213110.jpg)

两个分支的流程分别如下:

A:如果目标函数已经被 Hook<首条指令为 `E9 jmp`>,就将回跳的目的地址写成当前 Hook 的目标地址,转到 C

B:目标函数没有被 Hook,回跳的目的地址为目标函数地址+`TotalDecodedLength`,并将长度为 `TotalDecodedLength` 的原指令写入回跳缓冲之中,转到 C

C:将回跳指令写入缓冲

最后写入跳转到 `HS` 代理函数的 `jmp` 指令<这里是跳转到一个中间跳的缓冲>

![](img/df8fc472626097c34258030eb062188b.jpg)

我尝试着写成了 C 的形式:<未经编译可能存在错误>

代码:

```
#define JmpCodeLength 5
#define CallCodeLength 5

BOOLEAN Hook_InlineDo(PVOID JmpBackBuffer, ULONG PreAddress, PVOID JmpToProxyBuffer, 
         BOOLEAN IsHookCallInstruction, PLARGE_INTEGER ReturnValue)
{
  ULONG  OpcodeLength = 0;
  ULONG  DecodedLength = 0;
  ULONG  TotalDecodedLength = 0;
  ULONG  AddressTemp;
  ULONG  WriteAble;

  ULONG  JmpCode[2];
  ULONG  Offset;

  FillMemory(&JmpCode,sizeof(ULONG)*2,0x90);

  PreAddress = CheckFunctionWhetherBeenHooked(PreAddress,0);

  if ( (JmpBackBuffer == NULL)||(JmpToProxyBuffer == NULL)||(PreAddress == 0) ) return FALSE;

  //这个 if else 主要是将回跳信息写入 JmpBackBuffer
  if ( IsHookCallInstruction == FALSE )
  {
    AddressTemp = PreAddress;

    do 
    {
      DecodedLength = HsDisasm(PreAddress,&OpcodeLength);
      TotalDecodedLength = TotalDecodedLength + DecodedLength;

    } while ( TotalDecodedLength < JmpCodeLength );

    if ( TotalDecodedLength > 0x1B ) return FALSE;

    WriteAble = MakeSpecialAddressWriteAble(JmpBackBuffer,8,1);

    if ( *(PUCHAR)PreAddress == 0xE9 )    //首条指令是 jmp
    {
      *(PUCHAR)&JmpCode = 0xE9;
      Offset = *(PULONG)(PreAddress + 1);          //得到机器码偏移
      AddressTemp = Offset + PreAddress + JmpCodeLength;    //计算 jmp 的绝对地址
      Offset = AddressTemp - (ULONG)JmpBackBuffer - JmpCodeLength;

      *(PULONG)((ULONG)&JmpCode + 1) = Offset;        //写入缓冲

      TotalDecodedLength = 0;
    }
    else           //首条指令非 jmp
    {
      *(PUCHAR)&JmpCode = 0xE9;
      Offset = PreAddress + TotalDecodedLength - (JmpBackBuffer + TotalDecodedLength + JmpCodeLength);  //跳转偏移
      *(PULONG)((ULONG)&JmpCode + 1) = Offset;  //写入缓冲

      RtlMoveMemory(&JmpBackBuffer,(PVOID)PreAddress,TotalDecodedLength);  //写入原指令
    }

    RtlMoveMemory(((PVOID)((ULONG)JmpBackBuffer + TotalDecodedLength)), &JmpCode, sizeof(ULONG)*2);  //写入回跳 jmp 指令

    MakeSpecialAddressWriteAble((ULONG)JmpBackBuffer,8,WriteAble);  //恢复页面

    *(PUCHAR)&JmpCode = 0xE9;
    Offset = (ULONG)JmpToProxyBuffer - PreAddress - JmpCodeLength;
    *(PULONG)((ULONG)&JmpCode + 1) = Offset;        //准备跳转至代理函数的数据
  }

  else    //对 call 进行 Hook
  {
    RtlMoveMemory(&JmpCode,(PVOID)PreAddress,CallCodeLength);

    *(PUCHAR)&JmpCode = 0xE9;
    AddressTemp = *(PULONG)((ULONG)&JmpCode + 1) + PreAddress + CallCodeLength;    //取到 call 的绝对地址
    Offset = AddressTemp - JmpBackBuffer - CallCodeLength;           //计算偏移
    *(PULONG)((ULONG)&JmpCode + 1) = Offset;

    WriteAble = MakeSpecialAddressWriteAble(JmpBackBuffer,8,1);

    RtlMoveMemory((PVOID)JmpBackBuffer,&JmpCode,sizeof(ULONG)*2);

    MakeSpecialAddressWriteAble(JmpBackBuffer,8,WriteAble);

    RtlMoveMemory(&JmpCode, (PVOID)PreAddress, sizeof(ULONG)*2);
    Offset = (ULONG)JmpToProxyBuffer - PreAddress - CallCodeLength;
    *(PULONG)((ULONG)&JmpCode + 1) = Offset;      //准备跳转至代理函数的数据
  }

  WriteAble = MakeSpecialAddressWriteAble(PreAddress,8,1);

  *ReturnValue = ExInterlockedCompareExchange64Pack(PreAddress,JmpCode);

  MakeSpecialAddressWriteAble(PreAddress,8,WriteAble);

  return TRUE;
}
```

目的很明显的,如果目标函数之前已经被 Hook 了,那么 `HS` 将覆盖这个 Hook,当 `HS` 过滤完之后再跳转到之前的 HOOK,

这样 `HS` 就得到了主导权

如果函数头是 `FF25` 形式的 `jmp` 呢?那么此时操作的函数头地址就是 `jmp` 后面的绝对地址,`HS` 此时的操作便是针对于这个地址的,

如果这里是你的代理函数,那么此时 `HS` 已经修改了你的代码

不同于我们平时使用的断中断+修改 `CR0` 这种写入 Hook 的方式,`HS` 巧妙地使用了一个原子操作来完成 Hook,非常值得借鉴:

![](img/d050d607579c4eba445fcdcf3b06c61c.jpg)

![](img/af9cf96b864f664812cb79800d78072f.jpg)

另外关键函数 `MakeSpecialAddressWriteAble` 由于涉及到的页表页目录相关的知识我还没掌握,等以后再逆一下

写的有点乱七八糟,不知能不能有人看懂.

今天就写这点把,关于其他方面,改天再来讨论
