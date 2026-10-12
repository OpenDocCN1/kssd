
# 00B206D3

`00B206D3     8B0D 9455B300       mov ecx,dword ptr ds:[B35594]            ; 按键精灵.00495260`

`00B206D9     3341 20             xor eax,dword ptr ds:[ecx+20]`

提示被调试程序无法处理异常，`Shift+F9` 忽略，`game over` 程序退出。

天哪，我错在哪里啦！我怎么就到不了`push ebp //到达地球人都知道的位置`

想想第一次脱壳，没这么顺利也是正常的，那么我们再试试其他大大的方法。在后来三、四天里，我用了能找到的脱标准壳的方法来脱它，TMD 没一个成功的。这期间每天晚上 `19` 点到 `1` 点，我都在学习研究各各 `armadillo` 标准壳的脱法，搞的每天上班没精神。还是没有一点收获，总不能半途而废吧？去论坛问问各位大虾吧...

期间 `loveboom` 大大成给予提示说：“可能不是标准双进程壳或是高版本的，所以你按双进程的来脱是不行的”。可对于新手的我来说，说了等于白说。希望下次小弟提问时，各们大大能提供详细一点的资料... `^_^`

后来还多亏 `wangli_com` 大大的指点...

先找 `oep`，在 `OD` 中重开程序，下 `bp WaitForDebugEvent`，`F9` 运行。

`77E7A6CF K>   55                 push ebp   //来到这里，F2 清除断点`

`77E7A6D0     8BEC                mov ebp,esp`

`77E7A6D2     81EC 9C000000       sub esp,9C`

`77E7A6D8     53                 push ebx`

`77E7A6D9     56                 push esi`

`77E7A6DA     57                 push edi`

看堆栈窗口

`0012DA98   0046AD67  /CALL 到 WaitForDebugEvent 来自 按键精灵.0046AD61`

`0012DA9C   0012EB5C  |pDebugEvent = 0012EB5C`

`0012DAA0   000003E8  \Timeout = 1000. ms`

在 `0012EB5C` 上下转存中跟随

在下 `bp WriteProcessMemory,F9` 运行

`77E7ADB9 K>   55                 push ebp   //来到这里`

`77E7ADBA     8BEC                mov ebp,esp`

`77E7ADBC     51                 push ecx`

`77E7ADBD     51                 push ecx`

`77E7ADBE     8B45 0C             mov eax,dword ptr ss:[ebp+C]`

`77E7ADC1     53                 push ebx`

`77E7ADC2     8945 F8             mov dword ptr ss:[ebp-8],eax`

看数据转存窗口

`0012EB5C  01 00 00 00 68 03 00 00  ...h..`

`0012EB64  C4 00 00 00 01 00 00 80  ?....€`

`0012EB6C  00 00 00 00 00 00 00 00  ........`

`0012EB74  00 CC 43 00 02 00 00 00  .藽.... //大家看到没有 43CC00 就是 OEP`

再在 `OD` 中重开程序，下硬件断点：`he WaitForDebugEvent`，运行停住。回到壳空间，查找常数 `FFFFFFFF`，得到结果：

`0046B3D6   OR EAX, FFFFFFF8`

`0046B3F1   OR EDX, FFFFFFF8`

`0046B41A   OR ECX, FFFFFFF8`

`0046B8F0   OR EDX, FFFFFFF8`

`0046B90B   OR ECX, FFFFFFF8`

`0046B933   OR EAX, FFFFFFF8`

在 `0046B3D6` 上双击，来到这代码处，朝上几行看：

`0046B38A     83BD D0F5FFFF 00     cmp dword ptr ss:[ebp-A30],0  //这就是 tDasm 大侠文中提到的关键代码。`

`0046B391     0F8C A9020000        jl 按键精灵.0046B640`

`0046B397     8B8D D0F5FFFF        mov ecx,dword ptr ss:[ebp-A30]`

`0046B39D     3B0D E4B54900        cmp ecx,dword ptr ds:[49B5E4]`

`0046B3A3     0F8D 97020000        jge 按键精灵.0046B640`

`0046B3A9     8B95 44F6FFFF        mov edx,dword ptr ss:[ebp-9BC]`

`0046B3AF     81E2 FF000000        and edx,0FF`

`0046B3B5     85D2                test edx,edx`

`0046B3B7     0F84 AD000000        je 按键精灵.0046B46A`

`0046B3BD     6A 00               push 0`

`0046B3BF     8BB5 D0F5FFFF        mov esi,dword ptr ss:[ebp-A30]`

`0046B3C5     C1E6 04             shl esi,4`

`0046B3C8     8B85 D0F5FFFF        mov eax,dword ptr ss:[ebp-A30]`

`0046B3CE     25 07000080          and eax,80000007`

`0046B3D3     79 05               jns short 按键精灵.0046B3DA`

`0046B3D5     48                  dec eax`

`0046B3D6     83C8 F8              or eax,FFFFFFF8           //断在这里向上看`

`0046B3D9     40                  inc eax`

`0046B3DA     33C9                xor ecx,ecx`

`0046B3DC     8A88 809A4900        mov cl,byte ptr ds:[eax+499A80]`

`0046B3E2     8B95 D0F5FFFF        mov edx,dword ptr ss:[ebp-A30]`

`0046B3E8     81E2 07000080        and edx,80000007`

`0046B3EE     79 05               jns short 按键精灵.0046B3F5`

`0046B3F0     4A                  dec edx`

`0046B3F1     83CA F8              or edx,FFFFFFF8`

`0046B3F4     42                  inc edx`

`0046B3F5     33C0                xor eax,eax`

`0046B3F7     8A82 819A4900        mov al,byte ptr ds:[edx+499A81]`

`0046B3FD     8B3C8D 60524900      mov edi,dword ptr ds:[ecx*4+495260]`

`0046B404     333C85 60524900      xor edi,dword ptr ds:[eax*4+495260]`

`0046B40B     8B8D D0F5FFFF        mov ecx,dword ptr ss:[ebp-A30]`

`0046B411     81E1 07000080        and ecx,80000007`

`0046B417     79 05               jns short 按键精灵.0046B41E`

`0046B419     49                  dec ecx`

`0046B41A     83C9 F8              or ecx,FFFFFFF8`

`0046B41D     41                  inc ecx`

`0046B41E     33D2                xor edx,edx`

`0046B420     8A91 829A4900        mov dl,byte ptr ds:[ecx+499A82]`

`0046B426     333C95 60524900      xor edi,dword ptr ds:[edx*4+495260]`

`0046B42D     8B85 D0F5FFFF        mov eax,dword ptr ss:[ebp-A30]`

`0046B433     99                  cdq`

`0046B434     B9 1C000000          mov ecx,1C`

`0046B439     F7F9                idiv ecx`

`0046B43B     8BCA                mov ecx,edx`

`0046B43D     D3EF                shr edi,cl`

`0046B43F     83E7 0F             and edi,0F`

`0046B442     03F7                add esi,edi`

`0046B444     8B15 D4B54900        mov edx,dword ptr ds:[49B5D4]`

`0046B44A     8D04B2              lea eax,dword ptr ds:[edx+esi*4]`

`0046B44D     50                  push eax`

`0046B44E     8B8D D0F5FFFF        mov ecx,dword ptr ss:[ebp-A30]`

`0046B454     51                  push ecx`

`0046B455     E8 FF1F0000          call 按键精灵.0046D459`

`0046B45A     83C4 0C             add esp,0C`

`0046B45D     25 FF000000          and eax,0FF           《--- 修改此处`

`0046B462     85C0                test eax,eax`

在 `0046B38A` 处下一硬件执行断点，运行停住。按照 `tDasm` 的方法，修改代码为：

```
0046B45D     FF05 48EB1200        inc dword ptr ds:[12EB48]
0046B463     C705 E8B54900 010000>mov dword ptr ds:[49B5E8],1
0046B46D   ^ E9 18FFFFFF           jmp 按键精灵.0046B38A
```

把 `12EB48` 处置 `0`，去掉所有硬件断点，并在 `0046b640` 处下断，运行，停住。好，所有代码都强制解压完成。

运行 `LordPE`，选择第 `2` 个进程（有 `2` 个同名进程），即可完全 `dump` 出来了。

`dump` 出来了还不能用啦，怎么办？修复 `IAT` 表。怎么修复？`wangli_com` 大大没说。查看其它大大的脱文，大都是跳过 `magic jmp` 后用 `Imprec1.6f` 选择进程，填入 `Oep`，自动搜索。我试我试，怎么不行找不到 `IAT` 数据。经过 `N` 次失败后，我都要对 `armadillo` 妥协了...

怎么办了？自己搞不定就去查资料啦。后来我在密界脱壳文集中找到了 `tDasm` 大大的原文〈Armadillo 3.6 主程序脱壳〉〈Armadillo 3.6 主程序 IAT 处理〉，照着方法又脱了一遍非常顺利。当到了 `IAT` 处理的时候，`tDasm` 大大文中提到的方法一时无法理解，照着做都会跟飞。

没办法继续找方法哎。当看到 `jwh51` 大大的《Armadillo COPYMEMEII 之 DUMP 的一个 LOADPE 小插件》，我又照着脱了一遍，哈哈原来有这么方便的方法啦，跟 `tDasm` 大大的方法原理差不多，当步骤简单的多，新手脱不容易出错，强烈推荐。

在当我看到骨灰 `C` 姐姐的《实战 Armadillo3.60 Original CopyMem-ll +Debug-Blocker -----UltraEdit》文章后，解开了困惑了我三、四天的 `IAT` 表修复问题（建议新手可以学习但最好不用骨灰 `C` 姐姐的脱壳方法，她的原理和 `tDasm`、`jwh51` 等大大的方法一样，但过程复杂，容易出错）

好，言归正传，我们来修复 `IAT` 表...

我们用 `OD` 载入 `dump` 出来的文件，`F8` 单步执行。

```
0043CC00 1>   55                 push ebp  //程序入口 OEP
0043CC01     8BEC                mov ebp,esp
0043CC03     6A FF               push -1
0043CC05     68 189C4400         push 111.00449C18
0043CC0A     68 5ECD4300         push 111.0043CD5E
0043CC0F     64:A1 00000000      mov eax,dword ptr fs:[0]
0043CC15     50                 push eax
0043CC16     64:8925 00000000    mov dword ptr fs:[0],esp
0043CC1D     83EC 68             sub esp,68
0043CC20     53                 push ebx
0043CC21     56                 push esi
0043CC22     57                 push edi
0043CC23     8965 E8             mov dword ptr ss:[ebp-18],esp
0043CC26     33DB                xor ebx,ebx
0043CC28     895D FC             mov dword ptr ss:[ebp-4],ebx
0043CC2B     6A 02               push 2
0043CC2D     FF15 5C4B4400       call dword ptr ds:[444B5C]       //执行到这里我们的程序就飞了
```

`OD` 重头来一遍，当执行到 `43CC2D`，我们在数据窗口 `ctrl_G 444b5c` 是不是看到很多 `77` 之类的东西，哈哈，这就是 `IAT` 表的存放地。我们顺着向上看，来到数据窗口顶部地址 `444000` 就是 `IAT` 表的起始地址，我们拿笔记下。

在 `OD` 重新载入 `按键精灵 3.exe`，忽略所有异常，`bp DebugActiveProcess`，`F9` 断下。看堆栈窗口：

`0012DA9C    0046ABDB  /CALL 到 DebugActiveProcess 来自 按键精灵.0046ABD5`

`0012DAA0    000003CC  \ProcessId = 3CC   ―――＞子进程句柄`

打开另一个 `OD` 附加 `3CC` 这个子进程。然后 `ALT+F9` 返回程序

`00485000 按>- EB FE             jmp short 按键精灵.<ModuleEntryPoint>   //断在这里`

`00485002     0000              add byte ptr ds:[eax],al`

`00485004     0000              add byte ptr ds:[eax],al`

`00485006     5D                pop ebp`

`00485007     50                push eax`

`00485008     51                push ecx`

还原代码，不然会是死循环，还原开头两行二进制：`60 E8` 为 `60 E8`
