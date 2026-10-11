
# 记一游戏修改器的制作

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-16.htm`](https://www.pediy.com/kssd/pediy08/pediy8-16.htm)

前语：近来人比较懒，看雪真是人才辈出啊，发现自己落后太多了，看来以后要加紧学习了...

这个游戏是 `ToHeart2 XRATED` 附带的一个小游戏，一日无聊就拿它开刀了（最后有游戏的截图）

使用工具：`OllyDbg`, `金山游侠 2002`

用 `OD` 载入游戏（小提醒一下：最好打开 `OD` 后，把 `OD` 的优先级调低，再载入游戏，不然的话 `OD` 的分析时间太长了），

就会停在 `OEP` 上：

```
00463D30  >  $  6A 60         push 60                            ;  (initial cpu selection) //OD 停在这里
00463D32   .  68 F0316200    push SSS.006231F0
00463D37   .  E8 E01A0000    call SSS.0046581C
00463D3C   .  BF 94000000    mov edi,94
00463D41   .  8BC7           mov eax,edi
00463D43   .  E8 78FEFFFF    call SSS.00463BC0
00463D48   .  8965 E8        mov dword ptr ss:[ebp-18],esp
00463D4B   .  8BF4           mov esi,esp
00463D4D   .  893E           mov dword ptr ds:[esi],edi
00463D4F   .  56             push esi                             ; /pVersionInformation
00463D50   .  FF15 78F05F00  call dword ptr ds:[<&KERNEL32.GetVersion>; \GetVersionExA
```

嗯，程序应该是没壳的，直接调试就可以了。

打开金山游戏侠后按 `OD` 的 `F9` 键，运行程序，

游戏开始后，按 `OD` 的暂停键，再用金山游侠找数字 3（一开始是 3 条命的）。

为什么要暂停游戏呢？因为此东西比较 `BT`，呼出金山游戏侠后还不能暂停的，

我把这个情况说给我一个经常玩游戏的同学，他也说："不是吧，这么厉害的游戏，金山游侠也停不住"。

找到数字 3 后回到 `OD`，继续游戏，等生命数少一次后再找，找了两次后，就找到这里了：

| 地址 | 单字节 | 双字节 | 四字节 |
| :--- | :--- | :--- | :--- |
| | | | |
| :--- | :--- | :--- | :--- |
| `092EAD81` | `[001]` | `[49921]` | `[3087909633]` |
| `092B119D` | `[001]` | `[24321]` | `[1560633089]` |
| `06D6C2EC` | `[001]` | `[00001]` | `[0000000001]` |
| `01488A88` | `[001]` | `[21249]` | `[4244460289]` |
| `01488988` | `[001]` | `[19457]` | `[4244458497]` |
| `01488888` | `[001]` | `[17665]` | `[4244456705]` |
| `00CFCA50` | `[001]` | `[00001]` | `[0000000001]` |

看到 `DWORD` 是 1 的只有两个，`6D6C2EC` 和 `CFCA50`，我直觉的想人物的生命值应该是个 `static` 常量，

应该是高一点的地址所以就试着更改 `6D6C2EC` 地址上面的数值，结果是成功的，可以修改生命数了，

之后在此地址上面下 `DWORD` 的 `ACCESS` 中断，继续游戏，

不到半秒后，游戏再次被断下：

```
0042AED7  |.  897424 28     mov dword ptr ss:[esp+28],esi
0042AEDB  |.  E8 400B0300   call SSS.0045BA20
0042AEE0  |.  E9 D9010000   jmp SSS.0042B0BE
0042AEE5  |>  399F 94000000 cmp dword ptr ds:[edi+94],ebx        ;  中断在此
0042AEEB  |.  0F8F A9000000 jg SSS.0042AF9A
0042AEF1  |.  B9 A0DE7200   mov ecx,SSS.0072DEA0
0042AEF6  |.  E8 858EFDFF   call SSS.00403D80
0042AEFB  |.  A1 44736A00   mov eax,dword ptr ds:[6A7344]
0042AF00  |.  3858 41       cmp byte ptr ds:[eax+41],bl
```

即然是这里判断的，在 `CALL` 的尾改就可以

按翻页键，很快就找到此 `CALL` 的尾了：

```
0042B141  |.  8B15 44736A00 mov edx,dword ptr ds:[6A7344]
0042B147  |.  89B2 F40C0000 mov dword ptr ds:[edx+CF4],esi
0042B14D  |.  8BB7 94000000 mov esi,dword ptr ds:[edi+94]        ;  第 2 次内存中断
0042B153  |.  B9 A0DE7200   mov ecx,SSS.0072DEA0
0042B158  |.  E8 238CFDFF   call SSS.00403D80
0042B15D  |.  A1 44736A00   mov eax,dword ptr ds:[6A7344]
0042B162  |.  89B0 F80C0000 mov dword ptr ds:[eax+CF8],esi
0042B168  |.  8BBF 8C000000 mov edi,dword ptr ds:[edi+8C]
0042B16E  |.  B9 A0DE7200   mov ecx,SSS.0072DEA0
0042B173  |.  E8 088CFDFF   call SSS.00403D80
0042B178  |.  8B0D 44736A00 mov ecx,dword ptr ds:[6A7344]
0042B17E  |.  89B9 FC0C0000 mov dword ptr ds:[ecx+CFC],edi
0042B184  |.  8B4C24 20     mov ecx,dword ptr ss:[esp+20]
0042B188  |.  5F            pop edi
0042B189  |.  5E            pop esi
0042B18A  |.  5D            pop ebp
0042B18B  |.  64:890D 00000>mov dword ptr fs:[0],ecx
0042B192  |.  5B            pop ebx
0042B193  |.  83C4 1C       add esp,1C
0042B196  \\.  C3            retn
0042B197      CC            int3
0042B198      CC            int3
0042B199      CC            int3
0042B19A      CC            int3
0042B19B      CC            int3
```

在 `42B14D` 断下后，寄存器的值是：

`EAX` `00CFBD60`

`ECX` `059DFD18`

`EDX` `00CFBD60`

`EBX` `00000000`

`ESP` `059DFCF8`

`EBP` `0043FB10 SSS.0043FB10`

`ESI` `00000000`

`EDI` `06D6B0A8`

`EIP` `0042B14D SSS.0042B14D`

`EDI` 是 `6D6B0A8`，习惯性的看了一下堆栈：

```
059DFCF8   059EA48C
059DFCFC   06D6B0A8
059DFD00   0043FB10  SSS.0043FB10
059DFD04   059DFD68
059DFD08   77FB7E64  ntdll.77FB7E64
059DFD0C   004784AA  RETURN to SSS.004784AA from ntdll.RtlLeaveCriticalSection
059DFD10   00CFBB14
```

`[ESP+4]` = `EDI`，一个很有利用价值的情报 `:D`

按 `F8` 单步运行到 `42B188` 处，`EDI` 已经被清 0 了，可是堆栈依然没变，

可以用堆栈的这个地址来改内存了 `:)`

已经没有空间写代码了，所以直接往下跳，找到程序尾的空白处：

```
0042B188     /E9 C3331D00   jmp SSS.005FE550
0042B18D     |90            nop
0042B18E     |90            nop
0042B18F     |90            nop
0042B190     |90            nop
0042B191     |90            nop
```

二进制码是 `E9 C3 33 1D 00 90 90 90 90 90`

```
005FE545      00            db 00
005FE546      00            db 00                             ;  保存生命数
005FE547      00            db 00
005FE548      00            db 00                             ;  保存 Level 级数
005FE549      00            db 00
005FE54A      00            db 00
005FE54B      00            db 00
005FE54C      00            db 00
005FE54D      00            db 00
005FE54E      00            db 00
005FE54F      00            db 00
005FE550      60            pushad                            ;  保存现场
005FE551      9C            pushfd                            ;  保存现场
005FE552      8B7C24 28     mov edi,dword ptr ss:[esp+28]     ;  把这个 EDI = [ESP+4]的值取出,要注意 PUSHAD 和 PUSHFD 后堆栈的值
005FE556      33C0          xor eax,eax                       ;  EAX 清 0
005FE558      33DB          xor ebx,ebx                       ;  EBX 清 0
005FE55A      A0 46E55F00   mov al,byte ptr ds:[5FE546]       ;  生命数放入 AL,要注意的是前面的 EAX 已经清 0 了
005FE55F      8A1D 48E55F00 mov bl,byte ptr ds:[5FE548]       ;  Level 级数放入 BL,要注意前面的 EBX 已经清 0 了
005FE565      85FF          test edi,edi                       ;  测试 EDI 指针是否为空
005FE567      74 0C         je short SSS.005FE575             ;  是空指针就不处理,跳走
005FE569      8987 94000000 mov dword ptr ds:[edi+94],eax     ;  更改生命值
005FE56F      899F 80000000 mov dword ptr ds:[edi+80],ebx     ;  更改 Level 级数
005FE575      9D            popfd                             ;  恢复现场
005FE576      61            popad                             ;  恢复现场
005FE577      5F            pop edi                           ;  到下面都是原来程序的代码
005FE578      5E            pop esi
005FE579      5D            pop ebp
005FE57A      64:890D 00000>mov dword ptr fs:[0],ecx
005FE581      5B            pop ebx
005FE582      83C4 1C       add esp,1C
005FE585      C3            retn
005FE586      00            db 00
005FE587      00            db 00
```

二进制码是 `60 9C 8B 7C 24 28 33 C0 33 DB A0 46 E5 5F 00 8A 1D 48 E5 5F 00 85 FF 74 0C 89 87 94 00 00 00 89 9F 80 00 00 00 9D 61 5F 5E 5D 64 89 0D 00 00 00 00 5B 83 C4 1C C3`

测试了一下，`OK`，成功；

这一次有别与上次，上一次修改类似游戏的方法是改 `EXE` 文件，

这一次就做个外挂吧，现在流行外挂 `:D`

外挂的原理也不难，可以通过寻找窗口名的方法来找到游戏窗口，

再用 `GetWindowThreadProcessID()`得到进程的 `ID`，接着用 `OpenProcess()`

打开进程，最后用 `WriteProcessMemory()`改写内存数值就行了

这几个 `API` 最容易出错的是 `WriteProcessMemory()`了，所以大家小心点 `:)`

```c
//定义一下要修改的内容

BYTE mem1[10] = { 0xE9, 0xBF, 0x33, 0x1D, 0x00, 0x90, 0x90, 0x90, 0x90, 0x90 }; //0042B188
BYTE mem2[54] = { 0x60, 0x9C, 0x8B, 0x7C, 0x24, 0x28, 0x33, 0xC0, 0x33, 0xDB, 0xA0,
     0x46, 0xE5, 0x5F, 0x00, 0x8A, 0x1D, 0x48, 0xE5, 0x5F, 0x00, 0x85,
     0xFF, 0x74, 0x0C, 0x89, 0x87, 0x94, 0x00, 0x00, 0x00, 0x89, 0x9F,
```
