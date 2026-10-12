
### 0070CFD6 - 0070D04E 段

```
0070CFD6     A1 C45E6A00        mov eax,dword ptr ds:[6A5EC4]  
0070CFDB     E8 B485D8FF        call 00495594  
0070CFE0     8B15 D88F7100     mov edx,dword ptr ds:[718FD8]  
0070CFE6     8902              mov dword ptr ds:[edx],eax  
0070CFE8     33C0              xor eax,eax  
0070CFEA     55                push ebp  
0070CFEB     68 B9D07000       push 70D0B9  
0070CFF0     64:FF30          push dword ptr fs:[eax]  
0070CFF3     64:8920          mov dword ptr fs:[eax],esp  
0070CFF6     A1 D88F7100       mov eax,dword ptr ds:[718FD8]  
0070CFFB     8B00              mov eax,dword ptr ds:[eax]  
0070CFFD     E8 9ED8D8FF       call 0049A8A0  
0070D002     A1 D88F7100       mov eax,dword ptr ds:[718FD8]  
0070D007     8B00              mov eax,dword ptr ds:[eax]  
0070D009     8B10              mov edx,dword ptr ds:[eax]  
0070D00B     FF92 88000000     call dword ptr ds:[edx+88]  
0070D011     8B0D B0867100     mov ecx,dword ptr ds:[7186B0]  
0070D017     A1 B48B7100       mov eax,dword ptr ds:[718BB4]  
0070D01C     8B00              mov eax,dword ptr ds:[eax]  
0070D01E     8B15 6C876E00     mov edx,dword ptr ds:[6E876C]  
0070D024     E8 CF1AD9FF       call 0049EAF8  
0070D029     A1 D88F7100       mov eax,dword ptr ds:[718FD8]  
0070D02E     8B00              mov eax,dword ptr ds:[eax]  
0070D030     8B10              mov edx,dword ptr ds:[eax]  
0070D032     FF92 88000000     call dword ptr ds:[edx+88]  
0070D038     68 DC050000       push 5DC  
0070D03D     E8 3A4DD0FF       call 00411D7C  
0070D042     A1 B0867100       mov eax,dword ptr ds:[7186B0]  
0070D047     8B00              mov eax,dword ptr ds:[eax]  
0070D049     BA 70D27000       mov edx,70D270 ; ASCII "Tag&Rename 3.2 rc 3 UNREGISTERED"  
0070D04E     E8 418AD6FF       call 00475A94  
```

### 004C20B1 段

```
004C20B1     C645 FB 00        mov byte ptr ss:[ebp-5],0  
```
// Patch 点: `C645 FB 01` `mov byte ptr ss:[ebp-5],1` ★

```
004C20B5     EB 2E             jmp short 004C20E5  
```

### 二、显示注册名

```
00684611     E8 8230D8FF       call 00407698  
00684616     8D45 E4           lea eax,dword ptr ss:[ebp-1C]  
00684619     8B55 F8           mov edx,dword ptr ss:[ebp-8]  
```
// `[ebp-8]` 指向注册名，最后一起修改

```
0068461C     E8 770DD8FF       call 00405398  
```

不要问我是如何找的，大家都会调试，只不过某些人在这方面用的时间多点而已。

### 六、`RegOpenKeyExA`

`Ctrl+F` 在“整个段块”搜索命令：`push 20019`

```
0128A289     68 19000200       push 20019  
```
// 找到这里

```
0128A28E     6A 00             push 0  
0128A290     8BC7             mov eax,edi  
0128A292     E8 8D97FFFF       call 01283A24  
0128A297     50               push eax  
0128A298     56               push esi  
0128A299     E8 36B4FFFF       call 012856D4 ; jmp to advapi32.RegOpenKeyExA  
```
// Call `RegOpenKeyExA`

```
0128A29E     85C0             test eax,eax  
0128A2A0     0F85 84000000    jnz 0128A32A  
```

点右键，搜索全部命令：`call 012856D4`

| 地址 | 反汇编 | 注释 |
| :--- | :--- | :--- |
| `0128A299` | `call 012856D4` | `jmp to advapi32.RegOpenKeyExA` |
| `0128A376` | `call 012856D4` | `jmp to advapi32.RegOpenKeyExA` |
| `0128A3E2` | `call 012856D4` | `jmp to advapi32.RegOpenKeyExA` |
| `0128A495` | `call 012856D4` | `jmp to advapi32.RegOpenKeyExA` |
| `0128A57D` | `call 012856D4` | `jmp to advapi32.RegOpenKeyExA` |
| `0129E1C3` | `call 012856D4` | `jmp to advapi32.RegOpenKeyExA` |
| `0129E260` | `call 012856D4` | `jmp to advapi32.RegOpenKeyExA` |

一般修改后面 3 个 `CALL` 后的跳转就行了，记住这几个地方

其实在本例中不需要修改这里，但是为了解说也 `Patch` 了

```
0128A57D     E8 52B1FFFF       call 012856D4 ; jmp to advapi32.RegOpenKeyExA  
0128A582     85C0             test eax,eax  
0128A584     75 30            jnz short 0128A5B6  
0129E1C3     E8 0C75FEFF       call 012856D4 ; jmp to advapi32.RegOpenKeyExA  
0129E1C8     85C0             test eax,eax  
0129E1CA     75 42            jnz short 0129E20E  
0129E260     E8 6F74FEFF       call 012856D4 ; jmp to advapi32.RegOpenKeyExA  
0129E265     85C0             test eax,eax  
0129E267     0F85 88000000    jnz 0129E2F5  
```

### 七、一起 `Patch` 吧

大体流程清楚了，下面开始 `Patch` 吧。复制 `TagRename.exe` 改名为 `Patch.eXe`

看看 `Patch.eXe` 的 `PE` 信息，`SizeOfImage=004A2000`，区段信息如下：

| No | Name | VSize | VOffset | RSize | ROffset | Charact |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 01 | | `0030E000` | `00001000` | `000F4A00` | `00000400` | `E0000040` |
| 02 | | `0000B000` | `0030F000` | `00004000` | `000F4E00` | `E0000040` |
| 03 | | `00002000` | `0031A000` | `00000000` | `000F8E00` | `E0000040` |
| 04 | | `00004000` | `0031C000` | `00003800` | `000F8E00` | `E0000040` |
| 05 | | `00001000` | `00320000` | `00000200` | `000FC600` | `E0000040` |
| 06 | | `00001000` | `00321000` | `00000000` | `000FC800` | `E0000040` |
| 07 | | `00001000` | `00322000` | `00000200` | `000FC800` | `E0000040` |
| 08 | | `00029000` | `00323000` | `00000000` | `000FCA00` | `E0000040` |
| 09 | `.rsrc` | `0012E000` | `0034C000` | `0009BA00` | `000FCA00` | `E0000040` |
| 0A | `.0000` | `00027000` | `0047A000` | `00026400` | `00198400` | `E0000040` |
| 0B | `.adata` | `00001000` | `004A1000` | `00000000` | `001BE800` | `E0000040` |

用 `LordPE` 把最后一个区段 `VSize` 和 `RSize` 都改为 `1500`，保存后 `LordPE` 自动把 `SizeOfImage` 改为 `004A2500`

用 `WinHex` 在 `Patch.eXe` 末尾添加 `1500H` 长度的 `00` 数据。当然，如果代码不多的话可以在程序中找空地。这样添加 `1500H` 长度后我们就获得了后面 `500H` 空白可以用，而不必管是否会被其他覆盖了。`008A2000`（偏移 `0X1BF800`）处我们就可以写代码了。

新开 `OllyDBG` 载入 `Patch.eXe`，我们要先 `Patch` 几个循环解码，再 `Patch` 上面六步分析的地方。

#### 1、第 1 个点

```
00401000     68 01A08700       push 87A001  
```
// 进入 `Ollydbg` 后暂停在这

```
00401005     E8 01000000       call 0040100B  
0040100A     C3                retn  
```
`F7` 单步走，看 `ASProtect` 解压壳代码

```
0087A0FD     BE A93C493D       mov esi,3D493CA9  
0087A102     81F6 543D493D     xor esi,3D493D54  
0087A108     B8 EB8B342E       mov eax,2E348BEB  
0087A10D     8B0A              mov ecx,dword ptr ds:[edx]  
0087A10F     0F8B 08000000     jpo 0087A11D  
0087A115     68 924FB86F       push 6FB84F92  
0087A11A     8BFB              mov edi,ebx  
0087A11C     5F                pop edi  
0087A11D     81C1 21A29555     add ecx,5595A221  
0087A123     80DB 8D           sbb bl,8D  
0087A126     81C1 4646A24E     add ecx,4EA24646  
0087A12C     51                push ecx  
0087A12D     68 AF332832       push 322833AF  
0087A132     B0 5B             mov al,5B  
0087A134     5F                pop edi  
0087A135     58                pop eax  
0087A136     81C1 0770A50C     add ecx,0CA57007  
0087A13C     0F81 00000000     jno 0087A142  
0087A142     890A              mov dword ptr ds:[edx],ecx  
0087A144     81EA 31F64154     sub edx,5441F631  
0087A14A     BB A208717E       mov ebx,7E7108A2  
0087A14F     81C2 2DF64154     add edx,5441F62D  
0087A155     8BC7              mov eax,edi  
0087A157     83EE 01           sub esi,1  
0087A15A     0F85 ADFFFFFF     jnz 0087A10D  
```
// 解码循环 第 1 个点 接口 ★

// `Patch` ①、 `E9 E17E0200` `jmp 008A2040`

`Patch` 代码：

```
008A2040     0F85 C780FDFF     jnz 0087A10D  
```
// `0087A15A` 代码挪这里执行

```
008A2046     C705 5AA18700 0F85>mov dword ptr ds:[87A15A],FFAD850F  
008A2050     66:C705 5EA18700 F>mov word ptr ds:[87A15E],0FFFF  
```
// 还原 `0087A15A` 处修改的代码

```
008A2059     C705 D8A18700 8C7E>mov dword ptr ds:[87A1D8],27E8C  
```
// `Patch` 第 2 个点

```
008A2063     E9 F880FDFF       jmp 0087A160  
```
// 继续流程

#### 2、第 2 个点

```
0087A192     FF30              push dword ptr ds:[eax]  
0087A194     B9 3390F94E       mov ecx,4EF99033  
0087A199     5F                pop edi  
0087A19A     0FB7CB            movzx ecx,bx  
0087A19D     81F7 1D90B243     xor edi,43B2901D  
0087A1A3     8BD7              mov edx,edi  
0087A1A5     81EF 92625E0E     sub edi,0E5E6292  
0087A1AB     81D1 52F1012A     adc ecx,2A01F152  
0087A1B1     81C7 63A4242F     add edi,2F24A463  
0087A1B7     8AEF              mov ch,bh  
```
