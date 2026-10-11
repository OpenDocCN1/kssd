
# TimeToPhoto v 2.64591 代码修补术

> 原文：[`www.pediy.com/kssd/pediy12/128832.html`](https://www.pediy.com/kssd/pediy12/128832.html)

**【破文标题】**：TimeToPhoto v 2.64591 代码修补术

**【破文作者】**：zaas\[PYG\]

**【破解工具】**：`OllyICE`，`PEiD v0.94`

**【破解平台】**：`WinXP`

**【软件类别】**：国外软件/图像处理

**【软件授权】**：共享版

**【软件介绍】**：`TimeToPhoto` 可以在用户的数码照片印上拍照日期和时间，同时它还可以让用户在照片上写上简单的标签。

**【破解声明】**：我是一只小菜鸟，偶得一点心得，愿与大家分享

---

## 破解内容

首先查壳：`Microsoft Visual Studio .NET 2005 -- 2008` -> `Microsoft Corporation *`

以为是很简单的软件，`od` 载入一看，有点头晕：区段有 15 个，代码是加密的，在内存中解压缩出来的，例如

Memory map, 条目 31

- 地址=`005ED000`
- 大小=`000B0000` (720896.)
- 属主=`TimeToPh 00400000`
- 区段=`.text1`
- 包含=`SFX,代码`
- 类型=`Imag 01001002`
- 访问=`R`
- 初始访问=`RWE`

这样的程序就有点烦了，不能直接修改代码，比较痛苦。自己看了一下，软件代码段实际解压到 `005A000` 区段了。

在该区段搜索字符串，有收获：

- 地址=`005AC55D`
- 反汇编=`mov edx, 005AC5D4`
- 文本字符串=`REGISTERED VERSION`

双击进入：

代码:

```
005AC554    E8 8F300000     call    005AF5E8
005AC559    84C0            test    al, al
005AC55B    74 12           je      short 005AC56F
005AC55D    BA D4C55A00     mov     edx, 005AC5D4                     ; REGISTERED VERSION
005AC562    8B83 98030000   mov     eax, dword ptr [ebx+398]
005AC568    E8 9737EEFF     call    0048FD04
005AC56D    EB 10           jmp     short 005AC57F
005AC56F    BA 08C65A00     mov     edx, 005AC608                     ; EVALUATION VERSION
```

显然上部的 call 是关键 call 了，跟进可知：

代码:

```
005AF5E8    55              push    ebp
005AF5E9    8BEC            mov     ebp, esp
005AF5EB    6A 00           push    0
005AF5ED    6A 00           push    0
005AF5EF    53              push    ebx
005AF5F0    33C0            xor     eax, eax
005AF5F2    55              push    ebp
005AF5F3    68 71F65A00     push    005AF671
005AF5F8    64:FF30         push    dword ptr fs:[eax]
005AF5FB    64:8920         mov     dword ptr fs:[eax], esp
005AF5FE    8D55 F8         lea     edx, dword ptr [ebp-8]
005AF601    B8 8CF65A00     mov     eax, 005AF68C                     ; CERT_NT
005AF606    E8 FD53E6FF     call    00414A08
005AF60B    8B45 F8         mov     eax, dword ptr [ebp-8]
005AF60E    8D55 FC         lea     edx, dword ptr [ebp-4]
005AF611    E8 BED6E5FF     call    0040CCD4
005AF616    8B45 FC         mov     eax, dword ptr [ebp-4]
005AF619    BA A8F65A00     mov     edx, 005AF6A8                     ; Valid
005AF61E    E8 5177E5FF     call    00406D74
005AF623    0F94C0          sete    al
005AF626    C705 0CA95C00 1>mov     dword ptr [5CA90C], 14
005AF630    84C0            test    al, al
005AF632    75 0C           jnz     short 005AF640
005AF634    C705 0CA95C00 1>mov     dword ptr [5CA90C], 14
005AF63E    EB 14           jmp     short 005AF654
```

当 `eax`=`edx`=`valid` 时，注册成功。跟踪可知，启动验证，`NAG`，关于窗口，转换图片时均通过这个 call 做一次验证，因此，

修改 `005AF623 0F94C0 sete al` 为 `setne al` 即可达到目的。

但事情并没有这么简单，当你载入超过 20 张以上图片时，软件会弹出窗口，未注册版不能一次处理 20 张以上图片云云。。。

注意到以上代码：`005AF626 C705 0CA95C00 1>mov dword ptr [5CA90C], 14`

`0x14`=20，似乎 `[5CA90C]` 是一个全局变量。对它下硬件断点，再次载入图片时，成功断下：

代码:

```
005B237D    3B05 0CA95C00   cmp     eax, dword ptr [5CA90C]
005B2383    7F 0C           jg      short 005B2391
005B2385    55              push    ebp
005B2386    8B45 F8         mov     eax, dword ptr [ebp-8]
```

显然，超过 20 张图片就会跳到显示 `NAG` 要求注册的部分了。简单任务，修改 `005B2383 7F 0C jg short 005B2391` 为 `7F00` 就没问题了。

接下来就是转换图片部分，软件的转换部分也是通过内存中自解压源代码实现的，经过多次运行，发现软件存在一个 bug，载入图片可以是 20 张，而可以正常转换的软件却只有 13 张，为此研究了很久后来发现是软件作者的失误，推测是 `0x13` 写成 13 了，因此只能转换 13 张图片：

代码:

```
005B3BC1    E8 22BAFFFF     call    005AF5E8
005B3BC6    84C0            test    al, al
005B3BC8    0F85 9E000000   jnz     005B3C6C
005B3BCE    55              push    ebp
005B3BCF    33C0            xor     eax, eax
005B3BD1    E8 76FDFFFF     call    005B394C
005B3BD6    59              pop     ecx
005B3BD7    55              push    ebp
005B3BD8    B8 01000000     mov     eax, 1
005B3BDD    E8 6AFDFFFF     call    005B394C
005B3BE2    59              pop     ecx
005B3BE3    55              push    ebp
005B3BE4    B8 02000000     mov     eax, 2
005B3BE9    E8 5EFDFFFF     call    005B394C
005B3BEE    59              pop     ecx
005B3BEF    55              push    ebp
005B3BF0    B8 03000000     mov     eax, 3
005B3BF5    E8 52FDFFFF     call    005B394C
005B3BFA    59              pop     ecx
005B3BFB    55              push    ebp
005B3BFC    B8 04000000     mov     eax, 4
005B3C01    E8 46FDFFFF     call    005B394C
005B3C06    59              pop     ecx
005B3C07    55              push    ebp
005B3C08    B8 05000000     mov     eax, 5
005B3C0D    E8 3AFDFFFF     call    005B394C
005B3C12    59              pop     ecx
005B3C13    55              push    ebp
005B3C14    B8 06000000     mov     eax, 6
005B3C19    E8 2EFDFFFF     call    005B394C
005B3C1E    59              pop     ecx
005B3C1F    55              push    ebp
005B3C20    B8 07000000     mov     eax, 7
005B3C25    E8 22FDFFFF     call    005B394C
005B3C2A    59              pop     ecx
005B3C2B    55              push    ebp
005B3C2C    B8 08000000     mov     eax, 8
005B3C31    E8 16FDFFFF     call    005B394C
005B3C36    59              pop     ecx
005B3C37    55              push    ebp
005B3C38    B8 09000000     mov     eax, 9
005B3C3D    E8 0AFDFFFF     call    005B394C
005B3C42    59              pop     ecx
005B3C43    55              push    ebp
005B3C44    B8 0A000000     mov     eax, 0A
005B3C49    E8 FEFCFFFF     call    005B394C
005B3C4E    59              pop     ecx
005B3C4F    55              push    ebp
005B3C50    B8 0B000000     mov     eax, 0B
005B3C55    E8 F2FCFFFF     call    005B394C
005B3C5A    59              pop     ecx
005B3C5B    55              push    ebp
005B3C5C    B8 0C000000     mov     eax, 0C
005B3C61    E8 E6FCFFFF     call    005B394C
005B3C66    59              pop     ecx
005B3C67    E9 16010000     jmp     005B3D82
005B3C6C    90              nop
005B3C6D    90              nop
005B3C6E    90              nop
005B3C6F    90              nop
005B3C70    90              nop
005B3C71    90              nop
005B3C72    90              nop
005B3C73    90              nop
005B3C74    90              nop
005B3C75    90              nop
005B3C76    90              nop
005B3C77    90              nop
005B3C78    90              nop
```

当注册验证无误的时候，程序会直接跳到 `005B3C6C`，这里有大段的 `Nop`。如果是未注册版本，顺序执行上边的 call 为 20 个图片添加水印。因此推测，软件注册成功会在这里添加正常转换的代码。为此找了很久，限于水平（也可能是没有耐心）没有找到相关的 call。既然软件前 20 个图片可以正常转换，何不自己修补下代码，让他循环起来把所有图片都处理了不就完了嘛！在 call 里边跟了跟，发现载入的图片数在这里：

代码:

```
005B396B    8B45 08         mov     eax, dword ptr [ebp+8]
005B396E    8B40 FC         mov     eax, dword ptr [eax-4]
005B3971    8B80 CC050000   mov     eax, dword ptr [eax+5CC]
005B3977    8B80 EC020000   mov     eax, dword ptr [eax+2EC]
005B397D    E8 3A3DE7FF     call    004276BC
005B3982    48              dec     eax
```

这个 call 得出的就是载入图片的总数。查找 `ebp` 的位置，改写代码
