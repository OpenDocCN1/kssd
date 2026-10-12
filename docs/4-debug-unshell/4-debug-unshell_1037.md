
# SVKP 1.4x 脱壳记录

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-628.htm`](https://www.pediy.com/kssd/pediy08/pediy8-628.htm)

**【文章标题】**: 某外挂 `SVKP 1.4x` -> `Pavol Cerven` 脱壳

**【作者】**: `chasgone`(阳离子)

**【加壳方式】**: `SVKP 1.4x` -> `Pavol Cerven`

**【作者声明】**: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

\--------------------------------------------------------------------------------

## 【详细过程】

用 `peid` 查壳为 `SVKP 1.3x` -> `Pavol Cerven`，其实是高版本的壳。

### 一、oep

`od` 载入，忽略所有异常，在 `resource` 段下断点 `f9`，然后在 `code` 段下断点再 `f9` 就到 `oep` 了。

```
00401000         /EB 10             jmp short hx.00401012
00401002         |66:623A           bound di,dword ptr ds:[edx]
00401005         |43               inc ebx
00401006         |2B2B             sub ebp,dword ptr ds:[ebx]
00401008         |48               dec eax
00401009         |4F               dec edi
0040100A         |4F               dec edi
0040100B         |4B               dec ebx
0040100C         |90               nop
0040100D         -|E9 40744700      jmp 00878452
00401012         \\A1 33744700      mov eax,dword ptr ds:[477433]
00401017          C1E0 02          shl eax,2
0040101A          A3 37744700      mov dword ptr ds:[477437],eax
```

### 二、搞定 iat

`od` 重新载入，下断点 `bp GetModuleHandleA+5`。

`shift` + `f9` 断下，取消断点，`ctrl` + `f9` 返回主程序。

`ctrl` + `f` 搜索特征码：

```
cmp dword ptr ds:[ebx],251097CC
```

把这下面的所有的比较和跳转全部 `nop` 掉。

然后在搜索特征码：

```
mov dword ptr ds:[edi],eax
popad
```

改成：

```
popad
mov dword ptr ds:[edi],eax
```

然后在 `code` 段下断点 `f2`，然后 `f9` 就到 `oep` 了。

拿起 `importrec`，填 `oep=1000`，自动搜索 `iat`，全部有效，保存为树文件，后面有用。

### 三、搞定 replaced code

前面的两步和 `svkp 1.3x` 的一样，所以说的不详细，现在详细分析 `replaced code`，这是难点。

用上面得到的 `iat` 修复，`fixdump` 后，程序不能运行，`od` 载入脱壳后的文件，发现有很多的：

```
004011B4          E8 674F0700       call <jmp.&kernel32.ExitProcess>
```

和

```
00411879          E8 A04F0600         call <jmp.&user32.MessageBoxA>
```

原来程序把 `kernel32` 中的函数都换成 `ExitProcess` 公共接口了，把 `user32.dll` 里的函数换成 `MessageBoxA` 了。

现在就是要把这些 `ExitProcess` 换成本来的函数。

```
00401000         /EB 10             jmp short hx.00401012
00401002         |66:623A           bound di,dword ptr ds:[edx]
00401005         |43               inc ebx
00401006         |2B2B             sub ebp,dword ptr ds:[ebx]
00401008         |48               dec eax
00401009         |4F               dec edi
0040100A         |4F               dec edi
0040100B         |4B               dec ebx
0040100C         |90               nop
0040100D         -|E9 40744700      jmp 00878452
00401012         \\A1 33744700      mov eax,dword ptr ds:[477433]
00401017          C1E0 02          shl eax,2
0040101A          A3 37744700      mov dword ptr ds:[477437],eax
0040101F          52               push edx
00401020          6A 00            push 0
00401022          E8 F9500700      call hx.00476120    新建 eip
00401027          8BD0            mov edx,eax
00401029          E8 9EB80600      call hx.0046C8CC
0040102E          5A               pop edx
0040102F          E8 FCB70600      call hx.0046C830
00401034          E8 D3B80600      call hx.0046C90C
00401039          6A 00            push 0
0040103B          E8 F0CA0600      call hx.0046DB30
00401040          59               pop ecx
00401041          68 DC734700      push hx.004773DC
00401046          6A 00            push 0
00401048          E8 D3500700      call hx.00476120
```

在 `00401022` 处新建 `eip` 然后 `f7` 进去：

```
00476120         - FF25 FC834800     jmp dword ptr ds:[4883FC]
```

此时 `ds:[004883FC]=0DE076A9`。

正常情况下 `004883FC` 应该是 `api` 的入口，但是这里却指向壳中。

`f8` 进去，用 `f8` 和 `f7` 单步跟踪可以跟到这里：

```
0012FE25          58               pop eax
0012FE26          74 07           je short 0012FE2F
0012FE28          83E9 08         sub ecx,8
0012FE2B          74 2A           je short 0012FE57
0012FE2D         ^ EB DA           jmp short 0012FE09
0012FE2F          8B4431 FC       mov eax,dword ptr ds:[ecx+esi-4]
```

此时 `ds:[0DE0933F]=00488464` (`hx.00488464`)
`eax=00401022` (`hx.00401022`)

在命令窗口输入 `d 0DE0933F`。

此时可以看到有很大一张表：

```
0DE086CF  004884D0    hx.004884D0
0DE086D3  0046BFD6    hx.0046BFD6
0DE086D7  004884CC    hx.004884CC
0DE086DB  0046BF7C    hx.0046BF7C
0DE086DF  004884DC    hx.004884DC
0DE086E3  0046BCF9    hx.0046BCF9
...
0DE09337  00488464    hx.00488464
0DE0933B  00401048    hx.00401048
0DE0933F  00488464    hx.00488464
0DE09343  00401022    hx.00401022
```

这一张表的含义是什么呢？

奇数行表示正确的函数在 `iat` 中的位置。
偶数行就是需要修复所在的地址。
（可能我表达不大清楚，见笑了）

比如第一行和第二行为：

```
0DE086CF  004884D0    hx.004884D0
0DE086D3  0046BFD6    hx.0046BFD6
```

我们去 `0046BFD6` 看看。结果如下：

```
0046BFD6       E8 45A10000   call hx.00476120
```

我们再看看另一张表：

```
00476240         - FF25 BC844800         jmp dword ptr ds:[<&kernel32.GlobalUnloc>; kernel32.GlobalUnlock
00476246         - FF25 C0844800         jmp dword ptr ds:[<&kernel32.HeapAlloc>] ; ntdll.RtlAllocateHeap
0047624C         - FF25 C4844800         jmp dword ptr ds:[<&kernel32.HeapFree>]  ; ntdll.RtlFreeHeap
00476252         - FF25 C8844800         jmp dword ptr ds:[<&kernel32.InitializeC>; kernel32.InitializeCriticalSection
00476258         - FF25 CC844800         jmp dword ptr ds:[<&kernel32.Interlocked>; kernel32.InterlockedDecrement
0047625E         - FF25 D0844800         jmp dword ptr ds:[<&kernel32.Interlocked>; kernel32.InterlockedIncrement
```

`ds:[004884D0]=77E5A660` (`kernel32.InterlockedIncrement`)

即 `0047625E` 对应的就是 `004884D0`。

根据这一张表，我们就知道 `0046BFD6` 处应该怎么改了，应该改为 `call 0047625E`。

此 `dll` 的开始地址为 `4760E4`，结束地址为 `476342`，下面会用到这两个地址。

现在的任务就是写代码来修复了。

申请一块内存，我申请的是 `10000000`，大小为 `10000`。

写如下代码：

```
10000000         60                 pushad
10000001         9C                 pushfd
10000002         C705 00010010 CF86E> mov dword ptr ds:[10000100],0DE086CF
1000000C         A1 00010010        mov eax,dword ptr ds:[10000100]
10000011         8B18               mov ebx,dword ptr ds:[eax]
10000013         83C0 08           add eax,8
10000016         A3 00010010        mov dword ptr ds:[10000100],eax
1000001B         8B50 FC           mov edx,dword ptr ds:[eax-4]
1000001E         E8 1A000000       call 1000003D
10000023         83EB 05           sub ebx,5
10000026         2BDA               sub ebx,edx
10000028         895A 01            mov dword ptr ds:[edx+1],ebx
1000002B         A1 00010010        mov eax,dword ptr ds:[10000100]
10000030         3D 3F93E00D        cmp eax,0DE0933F
10000035       ^ 7E D5             jle short 1000000C
10000037         9D                 popfd
10000038         61                 popad
10000039       - EB FE             jmp short 10000039
1000003B         90                 nop
1000003C         90                 nop
1000003D         B8 E4604700        mov eax,4760E4
10000042         3958 02            cmp dword ptr ds:[eax+2],ebx
10000045         74 0C             je short 10000053
10000047         83C0 06           add eax,6
1000004A         3D 42634700        cmp eax,476342
1000004F       ^ 7E F1             jle short 10000042
10000051       - EB FE             jmp short 10000051
10000053         8BD8               mov ebx,eax
10000055         C3                 retn
```

在 `10000000` 新建 `eip` `f9` 执行，此时就全部替换了 `kernel32` 中以 `ExitProcess` 为公共接口的函数了。

下面就是要替换 `user32.dll` 中以 `MessageBoxA` 为公共接口的函数，还是要找出那两张表，然后写一段代码就行了。

下面我们开始找 `user32.dll` 中以 `MessageBoxA` 的地址表，步骤和上面一样。

可以跟到这里：

```
0012FF13         8B4431 FC      mov eax,dword ptr ds:[ecx+esi-4]          ; hx.00488D9C
```

`ds:[0DE2BD0A]=00488D9C` (`hx.00488D9C`)
`eax=00402F76` (`hx.00402F76`)

于是可以找到这张表为：

```
0DE2B09A  00488D60  hx.00488D60
0DE2B09E  0043FFBC  hx.0043FFBC
0DE2B0A2  00488B38  hx.00488B38
0DE2B0A6  0043FF68  hx.0043FF68
0DE2B0AA  00488BE8  hx.00488BE8
0DE2B0AE  0043FF1D  hx.0043FF1D
...
```



`0DE2BCEE` `0040B25E` `hx.0040B25E`

`0DE2BCF2` `00488B6C` `hx.00488B6C`

`0DE2BCF6` `0040B244` `hx.0040B244`

`0DE2BCFA` `00488B80` `hx.00488B80`

`0DE2BCFE` `0040B1C2` `hx.0040B1C2`

`0DE2BD02` `00488B80` `hx.00488B80`

`0DE2BD06` `004084B6` `hx.004084B6`

`0DE2BD0A` `00488D9C` `hx.00488D9C`

`0DE2BD0E` `00402F76` `hx.00402F76`

开始地址 `0DE2B09A`

结束地址 `0DE2BD0A`

再看看另一张表

```
004765A8  - FF25 2C8B4800  jmp dword ptr ds:[<&user32.ActivateKeybo>; user32.ActivateKeyboardLayout
004765AE  - FF25 308B4800  jmp dword ptr ds:[<&user32.AdjustWindowR>; user32.AdjustWindowRectEx
004765B4  - FF25 348B4800  jmp dword ptr ds:[<&user32.BeginPaint>]  ; user32.BeginPaint
004765BA  - FF25 388B4800  jmp dword ptr ds:[<&user32.CallNextHookE>; user32.CallNextHookEx

。。。。。。。。。。

00476938  - FF25 8C8D4800  jmp dword ptr ds:[<&user32.UpdateWindow>>; user32.UpdateWindow
0047693E  - FF25 908D4800  jmp dword ptr ds:[<&user32.WaitMessage>] ; user32.WaitMessage
00476944  - FF25 948D4800  jmp dword ptr ds:[<&user32.WinHelpA>]    ; user32.WinHelpA
0047694A  - FF25 988D4800  jmp dword ptr ds:[<&user32.WindowFromPoi>; user32.WindowFromPoint
00476950  - FF25 9C8D4800  jmp dword ptr ds:[<&user32.wsprintfA>]   ; user32.wsprintfA
00476956  - FF25 A08D4800  jmp dword ptr ds:[<&user32.GetSystemMenu>; user32.GetSystemMenu
```

修复代码为

```
10000000    60             pushad
10000001    9C             pushfd
10000002    C705 00010010 9AB0E>mov dword ptr ds:[10000100],0DE2B09A
1000000C    A1 00010010    mov eax,dword ptr ds:[10000100]
10000011    8B18           mov ebx,dword ptr ds:[eax]
10000013    83C0 08        add eax,8
10000016    A3 00010010    mov dword ptr ds:[10000100],eax
1000001B    8B50 FC        mov edx,dword ptr ds:[eax-4]
1000001E    E8 1A000000    call 1000003D
10000023    83EB 05        sub ebx,5
10000026    2BDA           sub ebx,edx
10000028    895A 01        mov dword ptr ds:[edx+1],ebx
1000002B    A1 00010010    mov eax,dword ptr ds:[10000100]
10000030    3D 0ABDE20D    cmp eax,0DE2BD0A
10000035  ^ 7E D5          jle short 1000000C
10000037    9D             popfd
10000038    61             popad
10000039  - EB FE          jmp short 10000039
1000003B    90             nop
1000003C    90             nop
1000003D    B8 A8654700    mov eax,4765A8
10000042    3958 02        cmp dword ptr ds:[eax+2],ebx
10000045    74 0C          je short 10000053
10000047    83C0 06        add eax,6
1000004A    3D 56694700    cmp eax,476956
1000004F  ^ 7E F1          jle short 10000042
10000051  - EB FE          jmp short 10000051
10000053    8BD8           mov ebx,eax
10000055    C3             retn
```

在 `10000000` 新建 `eip`，`f9`

到这里，代码就修复完了。

这个程序没有 `mov` 类的 `replaced code`。否则还要继续写代码修复，其实知道方法也不难。

现在可以拿起 `lordpe`，修正 `imagesize`，`dump`，用第二步得到的 `iat`，`fixdump`，运行成功

脱壳人：`chasgone`（阳离子）

`2006` 年 `6` 月 `5` 日
