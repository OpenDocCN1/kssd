
# X 舞团某外挂简单分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-557.htm`](https://www.pediy.com/kssd/pediy09/pediy09-557.htm)

> X 舞团某外挂简单分析

> 这个游戏曾经外挂泛滥。自从用了 HS 后，随着 HS 的进化。外挂逐渐消失。前段时间突然发现出现一个能连 P.而且分数相当 BT 的外挂。最重要的是它能在 HS 的淫威下生存.对其工作方式十分感兴趣。遂 IDA 之。

> 外挂据说是在美国的 TW 留学生写的。`aspack` 的壳。无 anti,无花，无 VM.在这个 VM 横行的时代.看上确实不像国人的作品.:-)

### 1. 侦查

> 要分析当然先要用用.看看什么情况.这一步我没有亲自做.就问问了身边玩这款游戏的朋友.

> 外挂工作的时候.无论按得结果如何.在其他玩家那里都是 perfect 的效果.分数也是正常.(在不开启 BT 功能情况下).如果开了 BT 功能则 miss 一次有 500W 分.

> 从上面的信息,凭直是修改了封包,而且应该是挂在 `send` 上改的.如果是改游戏的代码.他完全可以把本地显示的效果也改了.当时 HS 已经是全代码段效验了.所以修改游戏代码的可能性不大.(不是不可能.有点麻烦.这个挂的作者可能比较懒).

### 2. 验证

> `IDA HooK.dll.(附件里面有下载)`

> 看导出表.有个名为 `HookProc` 的函数.

> 里面有个 `CallNextHookEx`.可以判断这个外挂是通过 window hook 注入的游戏.

```
CODE:00445E04                   ; LRESULT __stdcall HookProc(int,WPARAM,LPARAM)
CODE:00445E04                   public HookProc
CODE:00445E04                   HookProc         proc near        
CODE:00445E04
CODE:00445E04
...
CODE:00445E12
CODE:00445E14 E8 FF FE FF FF              call    sub_445D18
...                      
...
CODE:00445E3A E8 49 08 FC FF              call    CallNextHookEx
CODE:00445E3F 5D                         pop     ebp
CODE:00445E40 C2 0C 00                   retn    0Ch
```

> 看看注入后干了什么.进入 `sub_445D18`

```
CODE:00445D18                   InstallSendHook proc near               ; CODE XREF: HookProc+10p
CODE:00445D18 53                         push    ebx
CODE:00445D19 56                         push    esi
CODE:00445D1A 51                         push    ecx
CODE:00445D1B BE 48 88 44 00            mov     esi, offset unk_448848
CODE:00445D20 E8 1B 06 FC FF            call    GetCurrentProcess
CODE:00445D20
CODE:00445D25 A3 64 88 44 00            mov     ds:CurProcessHandle, eax
CODE:00445D2A 68 9C 5D 44 00            push    offset LibFileName ; "ws2_32.dll"
CODE:00445D2F E8 CC 06 FC FF            call    LoadLibraryA
CODE:00445D2F
CODE:00445D34 8B D8                     mov     ebx, eax
CODE:00445D36 68 A8 5D 44 00            push    offset s_Send    ; "send"
CODE:00445D3B 53                         push    ebx             ; hModule
CODE:00445D3C E8 3F 06 FC FF            call    GetProcAddress_1
CODE:00445D3C
CODE:00445D41 A3 60 88 44 00            mov     ds:pWS_SEND, eax
CODE:00445D46 C6 06 B8                  mov     byte ptr [esi], 0B8h
CODE:00445D49 C6 46 05 FF                mov     byte ptr [esi+5], 0FFh
CODE:00445D4D C6 46 06 E0                mov     byte ptr [esi+6], 0E0h
CODE:00445D51 C6 46 07 00                mov     byte ptr [esi+7], 0
CODE:00445D55 54                         push    esp             ; lpNumberOfBytesRead
CODE:00445D56 6A 08                      push    8               ; nSize
CODE:00445D58 68 50 88 44 00            push    offset opcodebuffer ; lpBuffer
CODE:00445D5D A1 60 88 44 00            mov     eax, ds:pWS_SEND
CODE:00445D62 50                         push    eax             ; lpBaseAddress
CODE:00445D63 A1 64 88 44 00            mov     eax, ds:CurProcessHandle
CODE:00445D68 50                         push    eax             ; hProcess
CODE:00445D69 E8 CA 06 FC FF            call    ReadProcessMemory
CODE:00445D69
CODE:00445D6E B8 F8 59 44 00            mov     eax, offset SendProc
CODE:00445D73 89 46 01                  mov     [esi+1], eax
CODE:00445D76 54                         push    esp             ; lpNumberOfBytesWritten
CODE:00445D77 6A 08                      push    8               ; nSize
CODE:00445D79 56                         push    esi             ; lpBuffer
CODE:00445D7A A1 60 88 44 00            mov     eax, ds:pWS_SEND
CODE:00445D7F 50                         push    eax             ; lpBaseAddress
CODE:00445D80 A1 64 88 44 00            mov     eax, ds:CurProcessHandle
CODE:00445D85 50                         push    eax             ; hProcess
CODE:00445D86 E8 15 07 FC FF            call    WriteProcessMemory
CODE:00445D86
CODE:00445D8B A1 60 88 44 00            mov     eax, ds:pWS_SEND
CODE:00445D90 A3 44 88 44 00            mov     ds:WS_SEND, eax
CODE:00445D95 5A                         pop     edx
CODE:00445D96 5E                         pop     esi
CODE:00445D97 5B                         pop     ebx
CODE:00445D98 C3                         retn
```

> 很清楚吧.一看就是挂接的 `send` 函数.注意一下挂接的方式.不是 E8 E9.是

```
mov reg32,imm32
jmp reg32
```

> 这种挂接不太容易检测.因为 `reg32` 就有好几个.而且 `mov` 和最后 `jmp` 之间可以插垃圾.模式匹配的检测方式不太好用了.

> 这个值得学习.

> 这下可以肯定这个挂是挂接在 `send` 上修改游戏数据来作弊的.所以别人能看见效果.而本机看不见.

> 只要跟入 `SendProc` 看看挂怎么解密游戏的封包和看看修改了哪些字段就知道是如何工作的了.

> 关于怎么在 HS 的淫威下生存,现在很清楚了.HS 只保护了游戏.这个挂只挂接了 `send`.而且是 `jmp reg32` 式的 hook.所以 Hs 没有检测.

> 就算 HS 检测.也检测不到 `jmp reg32` 这种 hook.

> 记得 NP 从 10XX 版开始就开始保护 socket 函数了.而且检测了 `jmp reg32` 这种 hook.

> HS 还有待改进.

> 对封包的算法感兴趣的可以自己看 IDA.

> 下面直接给出结果

```
byte[] RawPacketBuffer={xxxxx}//这个就是 send 的数据.

           int KeyLength=RawPacketBuffer[2]=Convert.ToByte((~((RawPacketBuffer[2]>>7&0xFF)|(RawPacketBuffer[2]<<1&0xFF)))&0xFF);
           int i,KeyIndex,DataIndex;
           for (i = RawPacketBuffer.Length - KeyLength-3,KeyIndex=0,DataIndex=KeyLength+3; i > 0; --i)
           {
               RawPacketBuffer[DataIndex] ^= RawPacketBuffer[KeyIndex+3];
               KeyIndex++;
               if (KeyIndex == KeyLength)
                   KeyIndex = 0;
               DataIndex++;
           }
           byte GameDataKey = Convert.ToByte( RawPacketBuffer[KeyLength + 4] - 1);
           for (i = RawPacketBuffer.Length - KeyLength - 5, DataIndex = KeyLength + 5; i > 0; --i)
           {
               RawPacketBuffer[DataIndex] ^=GameDataKey;
               DataIndex++;
               GameDataKey--;
           }
//到这里的时候 RawPacketBuffer 就是明文了.
```

> 具体修改哪一个字段就自己去看 IDA 了.:-)

> 有矛就有盾

> 怎么防御呢。

> 作为一个成熟的游戏。修改结构或者改变算法是不太可能的事情。太浪费人力物力。而且改了算法别人也能逆出来。

> 比较经济的解决方式就是效验 `send` 函数。效验方法就各显神通了。

`4dd466d0e1e0cc123afbe504da2462ca`
