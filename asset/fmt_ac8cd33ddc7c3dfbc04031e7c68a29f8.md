
# H\*\*\*\*\*\*\*\* 4.01.11

> 原文：[`www.pediy.com/kssd/pediy03/forum829.htm`](https://www.pediy.com/kssd/pediy03/forum829.htm)

## 软件信息

软件：`H\*\*\*\*\*\*\*\*` 4.01.11   

保护方式：授权文件保护，未注册时每次启动会有一个提示注册的对话框   

该版本的保护方式与以前版本的保护方式不同，由序列号方式改为授权文件方式，注册时要求输入授权文件的全部内容。这对小弟的破解工作倒没有什么影响，小弟偏爱暴力法破解（本人比较懒，懒得算注册号），且喜欢装入程序，从头单步运行，直到弹出注册对话框（当然加壳的程序除外）。   

## 调试过程

先把 `H\*\*\*\*\*.exe` 拷到别的目录，作为备份。   

启动 `trw2000`，打开 `C:\Program Files\H\*\*\*\*\*\*\*\* 4\H\*\*\*\*\*.exe`，装入，按 `F10`。若按 `F10` 带过一个 `call` 时，弹出提示注册对话框，就关闭对话框，`trw2000` 会自动弹出。实在不行的话，退出程序，`trw2000` 也会自动弹出。用鼠标选中刚才的 `call`，按 `F9` 设断点，`g`，程序会正常运行。关闭程序，重新装入，全速运行，在刚才的断点中断。按 `F8` 进入 `call` 内，按 `F10` 重复刚才的过程，用不了几次，就能找到关键部位。请看(本人用 `log2` 命令无法存盘，只好 `keyin` 了，哪位大哥知道 `log2` 的用法，不妨指点小弟一下)：

```
42E279 8B837001000    mov eax,[ebx+170]; [ebx+170]=[4fdd30]=0   

42e27f 85c0            test eax,eax   

42e281 0f85d9000000   jnz near 42e360; (no jump)   
```

这时如果敲入 `rfl z` 命令，`g`，哈哈，注册对话框没有出现。很明显 `4fdd30` 这个全局变量中放的就是注册标志,若为 0，就是没有注册。现在，重新装入程序，`bc`，下 `bpm 167:4fdd30 w`，`g`。来到下面这一段：

```
436fcd e85ec9feff    call 423930   

436fd2 83c404        add esp,4   

436fd5 a330dd4f00   mov [4fdd30],eax; eax=0   
```

是谁让我们的 `eax` 变成 0 的？一定是上面那个 `call` 了。`bc`，下 `bpx 42e279`，`g`。退出 `hypersnap`，从新装入，`g`。   

`trw` 在 `436fcd` 中断，按 `F8` 进入。废话少说，来到下面这一段：

```
423a6c 33c0           xor eax,eax ;原来在这里了。   

423a6e 5b             pop ebx   

423a6f 8b4df4   

...   

423a7c c3             ret   
```

记下代码(建议大家多抄几行，我的是 `e870f606005f5e33c0`)，启动 `uedit`，查找 `e870f606005f5e33c0`，找到后将 `33c0` 改为 `9090`。存盘，试一下。   

居然弹出一个对话框，说程序被损坏了，怎么办呢？有了，`ctrl+m` 呼出 `trw`，下 `bpm lockmytask`，`g`。按 `ok` 按钮，`trw` 立即弹出。按几次 `F12`，回到 `H\*\*\*\*\*` 的领空，看看上一行是什么？原来是 `call user32!messagebox`。早知是这样，下 `bpx messagebox` 不久完了吗。现在怎么办呢？往上看看，有没有比较语句，能跳过 `messagebox`，若找不到，就 `ret` 到上一层 `call` 中，找一找能不能跳过这个 `call`。我 `ret` 了三四次，找到下面这一段：

```
42de41 e89a5cffff call 423ae0   

42de46 85c0      test eax,eax   

42de48 751d      jnz 42de67;   
```

把 `751d` 改为 `eb1d` 就行了，不再出程序被损坏的 `messagebox` 了，程序可以运行了，抓下的图片中没有公司标记了，行了吗？不行，程序现在动不动就出现非法操作，然后只能退出。下面是 `win98` 的提示信息。

```
H\*\*\*\*\* 在 00de:f5fa0f3d 的模块   

<未知> 中导致无效页错误。   

Registers:   

EAX=0073fad4 CS=0167 EIP=f5fa0f3d EFLGS=00010246   

EBX=0073fad4 SS=016f ESP=00640004 EBP=00640024   

ECX=006400a8 DS=016f ESI=8174cedc FS=229f   

EDX=bff768d5 ES=016f EDI=006400d0 GS=0000   

Bytes at CS:EIP:   

Stack dump:   

bff768c9 006400d0 0073fad4 006400ec 006400a8 006401dc bff768d5 0073fad4 006400b8 bff882eb 006400d0 0073fad4 006400ec 006400a8 f5fa0f3d 00640294   
```

我比较了一下，每次退出时的 `ip`，寄存器都是一样的，于是有了一个办法。装入程序，下 `bp if (eax==73fad4)`，因为每次出错时的 `eax` 都是 `73fad4`。`g`。程序启动了，画面出来了，似乎一切正常。动一动鼠标，没动几下，出来一个蓝屏，`trw` 说它不能中断。上帝啊，连 `trw` 都不能中断，我要绝望了。   

还有一招，内存补丁。希望你备份的 `H\*\*\*\*\*.exe` 还在，把它考回来。我用的补丁是 `ppatcher` 3.93，这可是 `ppatcher` 的作者 `thewd` 亲自 `email` 给我的。`thewd` 挺负责任的，有信必回，你要是能对 `ppatcher` 提出什么意见或建议，他还会发给你一个最新版的 `ppatcher`。扯远了。`ppatcher.ppc` 的内容如下：

```
#Process Patcher Configuration File   

Version=3.93   

PatchAuthor=thewd   

PatchContactInformation=thewd@hotmail.com   

DisplayName=h\*\*\*\*\*.exe v4.01.11   

Filename=h\*\*\*\*\*.exe   

Filesize=1056768   

Address=0x42de48:0x75:0xeB   

Address=0x4337e6:0x75:0xeB   

Address=0x4387e5:0x75:0xeB   

Address=0x42e281:0x0f:0xe9   

Address=0x42e282:0x85:0xda   

Address=0x42e283:0xd9:0x00   

Address=0x42e284:0x00:0x00   

Address=0x42e285:0x00:0x00   

Address=0x42e286:0x00:0x90   

#End of Configuration File   
```

这回行了吧。`ppatcher.ppc` 所修改的地址跟我刚才说的不太一样，因为光修改 `423a6c` 处的 `xor eax,eax`，还有一点问题。按菜单中的关于和注册两项时，还会出现非法操作。修改也很简单，只要下 `bpm 4fdd30`，就很容易找到比较点了。   

罗里罗唆说了半天，总结一下。   

1. `4fdd30` 是一个全局变量，存放的是注册标志，跟注册有关的判断都以它为依据。   
2. 程序有自校验功能，但在 `42de48` 处可以跳过。   
3. 程序还有暗桩，修改之后动不动就非法操作。本人功力不够找不到，只好用内存补丁。比较遗憾。请高手们赐教。

注：`H\*\*\*` 3.53 和 3.63.01 的自校验方式与 4.01.11 不同，修改文件后，再运行会提示有个文件找不到，而且用 `trw2000` 也无法装入。颇为奇怪。那位有兴趣共同研究一下。3.63.01 避注册框的比较点是 `45ae8b`。

多谢 `bnbnf` 兄的提示，终于破解了 `H\*\*\*` 3.63.01。方法如下：

先打开资源管理器，找到 `C:\Program Files\H\*\*\*\*\*` 目录，再运行 `trw2000`，按 `ctrl+M` 呼出 `trw2000`，下 `bpx createfilea`，然后 `g` 回到资源管理器，在管理器中运行 `hsdx.exe`，`trw` 会弹出两次。第一次不用管，`g`。`trw` 第二次弹出时按 `F12` 会回到 `hshelper!.text` 的天空。关键代码就在这里。请看

```
6308105b 50          push eax ；eax 指向 hsdx.exe   

6308105c ff1538600863 call kernel32!createfilea   

63081062 83f8ff     cmp eax,-1   

63081065 8945f8     mov [ebp-8],eax   

63081068 0f84a1000000 jz 63081110 ；若已修改过 hsdx.exe，这里就是 no jump 。   
```

单步执行到这里时，下 `rfl z`，然后 `g`。程序界面终于出来了。具体修改过程就不说了。另外更正一下，上次提到 3.63.01 的关键点是错的，正确地址是 `416006`。

## 注册 H\*\*\*\*\*\*\*\* ver. 3.63.01 算法

软件名称：`H\*\*\*\*\*\*\*\*` ver. 3.63.01

主要目的：寻找自己的注册码

破解难度：中等难度(算码部分稍微复杂了一点)

不说废话了，直接进入正题。输入大名 `zombieys` 和注册码 `1212-1212-1212-1212-1212-1212`(为什么不填 `78`?我的内存里到处都是 `78`，找不到啦，所以换换)，下 `s 30:0 l ffffffff '1212'`，`bpm` 搜到的地址。点注册，按几次 `f12` 和 `f10` 来到这里：

### 算 NAME 的 CALL

```
:004096DB 55                       push ebp   

:004096DC 8BEC                     mov ebp, esp   

:004096DE 81EC04010000             sub esp, 00000104   

:004096E4 8B4D08                   mov ecx, dword ptr [ebp+08]   

:004096E7 33D2                     xor edx, edx   

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:00409705(U)   

|   

:004096E9 8A01                     mov al, byte ptr [ecx]                 //读取输入的注册码   

:004096EB 84C0                     test al, al                              //在 A-Z 之间才要   

:004096ED 7418                     je 00409707   

:004096EF 3C41                     cmp al, 41                                 //过滤掉-(2dhex)   

:004096F1 7C0F                     jl 00409702   

:004096F3 8A01                     mov al, byte ptr [ecx]   

:004096F5 3C5A                     cmp al, 5A   

:004096F7 7F09                     jg 00409702   

:004096F9 888415FCFEFFFF           mov byte ptr [ebp+edx-00000104], al      //将过滤后掉-后的 24 个注册码->[ebp+edx-00000104]   

:00409700 EB01                     jmp 00409703   

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   

|:004096F1(C), :004096F7(C)   

|   

:00409702 4A                       dec edx   

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:00409700(U)   

|   

:00409703 41                       inc ecx   

:00409704 42                       inc edx   

:00409705 EBE2                     jmp 004096E9   

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:004096ED(C)   

|   

:00409707 80A415FCFEFFFF00          and byte ptr [ebp+edx-00000104], 00   

:0040970F 53                       push ebx   

:00409710 56                       push esi   

:00409711 8D85FCFEFFFF            lea eax, dword ptr [ebp+FFFFFEFC]   

:00409717 57                       push edi   

:00409718 50                       push eax   

:00409719 E812A20200              call 00433930   

:0040971E 8BD0                     mov edx, eax   

:00409720 59                       pop ecx   

:00409721 81FA80000000             cmp edx, 00000080   

:00409727 8955FC                   mov dword ptr [ebp-04], edx   

:0040972A 0F8DC5000000             jnl 004097F5   

:00409730 A170504800              mov eax, dword ptr [00485070]   

:00409735 3BD0                     cmp edx, eax   

:00409737 0F8EB8000000             jle 004097F5   

:0040973D 0FBEB405FBFEFFFF         movsx esi, byte ptr [ebp+eax-00000105]  //取注册码第 5 位   

:00409745 83EE41                   sub esi, 00000041                      //让后-A(41HEX)->ESI   

:00409748 33FF                     xor edi, edi   

:0040974A 33C9                     xor ecx, ecx   

:0040974C 893578304900             mov dword ptr [00493078], esi   

:00409752 85D2                     test edx, edx   

:00409754 894D08                   mov dword ptr [ebp+08], ecx   

:00409757 7E5D                     jle 004097B6   

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:004097B4(C)   

|   

:00409759 80BC0DFCFEFFFF41          cmp byte ptr [ebp+ecx-00000104], 41    //注册码要在 A-Z 之间   

:00409761 8D9C0DFCFEFFFF           lea ebx, dword ptr [ebp+ecx-00000104]   

:00409768 0F8C87000000             jl 004097F5   

:0040976E 80BC3DFCFEFFFF5A         cmp byte ptr [ebp+edi-00000104], 5A   

:00409776 7F7D                     jg 004097F5   
```
