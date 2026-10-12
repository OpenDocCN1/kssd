
# Fast Browser

> 原文：[`www.pediy.com/kssd/pediy04/kanxue190.htm`](https://www.pediy.com/kssd/pediy04/kanxue190.htm)

`Fast Browser` 是一个非常棒的多线程浏览器，它让你的网上冲浪变得更加简单和方便，极大地扩充了浏览器的功能。介绍和下载地址：`http://download.pchome.net/internet/browser/browser/875.html`

我个人很喜欢它，但是要注册才无功能限制。而且注册方式比较特别，输入`username`后，要上网检验是否合法，如果没有你名字的定单，那么`username`是非法的。

先用`fi`看一下`FastBrowser.exe`是`PECompact v1.40`的壳。很好脱，`冲击波 2k`查到`OEP`是`503254`，用`trw2k`载入执行到`503254`后，下`makepe`命令即可。

运行脱壳后的程序……ok！挺正常，慢……居然没有`skin`（“皮肤”）\*@!#%$#^&#$(……

看来它在`OEP`之前就加载`skin`了。看看原始目录发现有个`skindll.dll`挺可疑。把它改名，再运行原始的`FastBrowser.exe`，果然“皮肤”不见了。好，下断点`bpx loadlibrarya`；重新运行，在`loadlibrarya`处停下，看一下参数：`d \*(esp+4)`，果然是`skindll.dll`！F12 回到`FastBrowser.exe`领空：

```
\======================================================================================

:005034C0 55                      push ebp

:005034C1 8BEC                    mov ebp, esp

:005034C3 83C4F8                  add esp, FFFFFFF8

\* Possible StringData Ref from Code Obj ->"skindll.dll"

                                |

:005034C6 68F4345000              push 005034F4

\* Reference To: KERNEL32.LoadLibraryA, Ord:0000h

                                |

:005034CB E8E0FFFFFF              Call 005034B0

:005034D0 8945FC                  mov dword ptr [ebp\-04], eax <==停在这

:005034D3 6A0A                    push 0000000A

:005034D5 8B45FC                  mov eax, dword ptr [ebp\-04]

:005034D8 50                      push eax

\* Reference To: KERNEL32.GetProcAddress, Ord:0000h

                                |

:005034D9 E8DAFFFFFF              Call 005034B8

:005034DE 8945F8                  mov dword ptr [ebp\-08], eax

:005034E1 837DF800                cmp dword ptr [ebp\-08], 00000000

:005034E5 7403                    je 005034EA

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0050349F(C)

|

:005034E7 FF55F8                  call [ebp\-08]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:005034E5(C)

|

:005034EA E8B9FFFFFF              call 005034A8

:005034EF 59                      pop ecx

:005034F0 59                      pop ecx

:005034F1 5D                      pop ebp

:005034F2 C3                      ret

\======================================================================================
```

当函数返回后来到这里：

```
\* Referenced by a CALL at Address:

|:005034EA

|

:005034A8 E8A7FDFFFF              call 00503254

:005034AD C3                      ret

\=========================================================================================
```

哈哈……这里不就是`OEP`嘛！

往上看，在`005034C0`处，凭经念此处应该是`call`开始的第一条指令。 所以冒险把`OEP`修正为`005034C0`，用`peditor`改好后，运行，一切`ok`！

脱壳后的`FastBrowser.exe`再用`fi`查看，居然是`dephi`写的！（第一次`crack dephi`程序，心里没底，呵呵……），刚好手头上有反编译`dephi`工具`dede v2.5`，用`dede`装入`FastBrowser.exe`，点击`process`按扭开始反编译……。

趁反编译的空，先整理一下思路，前面讲过`FastBrowser.exe`加密的特别地方，但种类软件通常有个弱点，就是用一个`bool`型的全局变量来表示是否注册。所以主要找到它初始化的地方，将它改为`01`，就变成注册版了。

现在点击`dede`中的`procedures`按扭，可以看到几个函数，再点击右边的“Disassemble proc”按扭，输入`503254`（即原始的`OEP`）这是查看程序的人口函数（相当于主程序）：

```
00503254  55                       push    ebp

00503255  8BEC                     mov     ebp, esp

00503257  83C4EC                   add     esp, -$14

0050325A  53                       push    ebx

0050325B  56                       push    esi

0050325C  33C0                     xor     eax, eax

0050325E  8945EC                   mov     [ebp\-$14], eax

00503261  8945F0                   mov     [ebp\-$10], eax

00503264  B8CC2E5000               mov     eax, $00502ECC

\* Reference to: sysinit.@InitExe;

|

00503269  E8823FF0FF               call    004071F0

0050326E  8B1DD4655000             mov     ebx, [$5065D4]

00503274  33C0                     xor     eax, eax

00503276  55                       push    ebp

00503277  687D345000               push    $0050347D

\*\*\*\*\* TRY

|

0050327C  64FF30                   push    dword ptr fs:[eax]

0050327F  648920                   mov     fs:[eax], esp

00503282  A13C685000               mov     eax, dword ptr [$50683C]

00503287  C60000                   mov     byte ptr [eax], $00

0050328A  A140675000               mov     eax, dword ptr [$506740]

0050328F  C60000                   mov     byte ptr [eax], $00

00503292  A16C685000               mov     eax, dword ptr [$50686C]

00503297  C60001                   mov     byte ptr [eax], $01

|

0050329A  E80D1BFDFF               call    004D4DAC

0050329F  8B15DC625000             mov     edx, [$5062DC]  <====关键！！

005032A5  8802                     mov     [edx], al

005032A7  A1E0655000               mov     eax, dword ptr [$5065E0]

005032AC  803800                   cmp     byte ptr [eax], $00

005032AF  740A                     jz      005032BB

|

005032B1  E8EA54FDFF               call    004D87A0

005032B6  E9A7010000               jmp     00503462

|

005032BB  E82C25FDFF               call    004D57EC

005032C0  84C0                     test    al, al

005032C2  0F849A0000               jz      00503462

005032C8  A140675000               mov     eax, dword ptr [$506740]

005032CD  803800                   cmp     byte ptr [eax], $00

005032D0  7439                     jz      0050330B

005032D2  A1DC625000               mov     eax, dword ptr [$5062DC]

005032D7  803800                   cmp     byte ptr [eax], $00

005032DA  752F                     jnz     0050330B

005032DC  33C9                     xor     ecx, ecx

005032DE  B201                     mov     dl, $01

\* Reference to class TRegisterForm   <===注册窗口！！

|

005032E0  A1C8B14D00               mov     eax, dword ptr [$4DB1C8]

\* Reference to: forms.TCustomForm.Create(TCustomForm;boolean;Classes.TComponent);

|           or: forms.TDataModule.Create(TDataModule;boolean;Classes.TComponent);

|

005032E5  E83E92F4FF               call    0044C528

\==================================================================
```

看到`TRegisterForm`了吗？呵呵…… 往上看，全局变量`[$506740]`和`[$5062DC]`很可疑，经过测试`[$5062DC]`就是代表是否注册的全局变量。它在哪里初始化呢？再往上看，`0050329F`处就是了！我将它改为：

```
inc al

mov [5062dc],al

nop
```

再次运行，Mmm……已经是注册版了，但标题还是未注册的，真烦！用`w32asm`黄金版反汇编，查找字符串`"(Unregistered Version)"`

```
\=============================================================================

:004D4B53 E86CF2F2FF                 call 00403DC4

:004D4B58 803D805E500000              cmp byte ptr [00505E80], 00

:004D4B5F 752B                       jne 004D4B8C

:004D4B61 803D745E500000              cmp byte ptr [00505E74], 00

:004D4B68 7522                       jne 004D4B8C  <=======注意

:004D4B6A 84DB                       test bl, bl

:004D4B6C 751E                       jne 004D4B8C

:004D4B6E B80C7D5000                 mov eax, 00507D0C

\* Possible StringData Ref from Code Obj ->"(Unregistered Version)"

                                |

:004D4B73 BAF04B4D00                 mov edx, 004D4BF0

:004D4B78 E87BF4F2FF                 call 00403FF8

:004D4B7D B8107D5000                 mov eax, 00507D10

\* Possible StringData Ref from Code Obj ->"(Unregistered Version)"

                                |

:004D4B82 BAF04B4D00                 mov edx, 004D4BF0

:004D4B87 E86CF4F2FF                 call 00403FF8

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004D4B5F(C), :004D4B68(C), :004D4B6C(C)

|

:004D4B8C 33C0                       xor eax, eax

\=======================================================================
```

将`004D4B68`处`jne`该为`jmp`即可。

其实这个软件应该可以找到注册码的，关键在`50329a`处的`call`；我大概看了一下，它在`.\\Fast Browser\\Pro4`目录下查找`serialcode.key`和`.\\Fast Browser\\Pro4\\Search\\buy`两个`keyfile`……，不过我一向不喜欢数字游戏，还是请各位大虾出马吧！

`4dd466d0e1e0cc123afbe504da2462ca`
