
# 电子文档阅览播放器 V1.30

> 原文：[`www.pediy.com/kssd/pediy05/pediy50647.htm`](https://www.pediy.com/kssd/pediy05/pediy50647.htm)

下载页面：[`www.skycn.com/soft/9401.html`](http://www.skycn.com/soft/9401.html)

软件大小：985 KB

软件语言：简体中文

软件类别：国产软件 / 共享版 / 电子阅读

应用平台：`Win9x/NT/2000/XP`

加入时间：2002-10-12 16:20:07

下载次数：2570

推荐等级：\*\*\*

【软件简介】：电子文档阅读播放器 `ESee` 能快速滚屏阅览 `Html`、`Txt` 文档，并能保持文件原有风格；可幻灯播放 `Jpeg` 图片；可快速播放 `Flash` 动画和 `Mp3`、`Mid` 音乐，可在您阅览文档和图片的同时实现 `Mp3`、`Mid` 的背景音乐播放。电子文档阅读播放器 `ESee` 还内置了实用的文件批量重命名和文本合并工具，可实现文件任意选择、排序、任意起始序列重命名，可将多个 `Html` 或 `Txt` 同类型文本合并成一个文件，也可将多个 `Html` 和 `Txt` 不同类型混合合并成一个文件，并能始终保持文件原有风格。电子文档阅读播放器 `Esee` 具有界面朴实，操作简单，功能实用等特点。是快速浏览电子文档，经常上网保存网页资料者的得力助手。

【软件限制】：功能限制

【作者声明】：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

【破解工具】：`TRW2000` 娃娃修改版、`Ollydbg1.09`、`PEiD`、`W32Dasm 9.0` 白金版

—————————————————————————————————

【过 程】：

`esee.exe` 无壳。`Borland Delphi 6.0` 编写。呵呵，我等菜鸟喜欢的类型呀。

机器码：`358301`

试炼码：`13572468`

—————————————————————————————————

```
:004A3830 E8C3010000            call 004A39F8

                                ===>取得机器码

:004A3835 8BF0                 mov esi, eax

                                ===>ESI\=EAX\=0005779D（H）=358301（D）

:004A3837 8D8640E20100         lea eax, dword ptr \[esi+0001E240\]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004A37D6(C)

|

:004A383D 8945F0               mov dword ptr \[ebp\-10\], eax

:004A3840 8D55F4               lea edx, dword ptr \[ebp\-0C\]

:004A3843 8BC6                 mov eax, esi

:004A3845 E80A55F6FF           call 00408D54

:004A384A B201                 mov dl, 01

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004A37D3(C)

|

:004A384C A12CCD4600           mov eax, dword ptr \[0046CD2C\]

:004A3851 E8D695FCFF           call 0046CE2C

:004A3856 8BD8                 mov ebx, eax

:004A3858 8D45EC               lea eax, dword ptr \[ebp\-14\]

:004A385B 8B4DF4               mov ecx, dword ptr \[ebp\-0C\]

\* Possible StringData Ref from Code Obj ->"您的软件编号(机器码):"

                                |

:004A385E BA40394A00           mov edx, 004A3940

:004A3863 E85410F6FF           call 004048BC

:004A3868 8B45EC               mov eax, dword ptr \[ebp\-14\]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004A37FE(C)

|

:004A386B 8D4DFC               lea ecx, dword ptr \[ebp\-04\]

\* Possible StringData Ref from Code Obj ->"输入注册码： "

                                |

:004A386E BA60394A00           mov edx, 004A3960

:004A3873 E8E8C5F9FF           call 0043FE60

                                ===>取得试炼码

:004A3878 84C0                 test al, al

:004A387A 0F8487000000         je 004A3907

:004A3880 8D4DF8               lea ecx, dword ptr \[ebp\-08\]

:004A3883 8BC6                 mov eax, esi

                                ===>EAX\=ESI\=0005779D（H）

:004A3885 03C0                 add eax, eax

                                ===>EAX\=0005779D + 0005779D=000AEF3A

:004A3887 BA04000000           mov edx, 00000004

:004A388C E82755F6FF           call 00408DB8

:004A3891 8B45F8               mov eax, dword ptr \[ebp\-08\]

                                ===>EAX\=AEF3A               注册码

:004A3894 8B55FC               mov edx, dword ptr \[ebp\-04\]

                                ===>EDX\=13572468             试炼码

:004A3897 E81811F6FF           call 004049B4

                                ===>比较 CALL！

:004A389C 741A                 je 004A38B8

                                ===>不跳则 OVER！

:004A389E 6A00                 push 00000000

\* Possible StringData Ref from Code Obj ->"提示"

                                |

:004A38A0 B970394A00           mov ecx, 004A3970

\* Possible StringData Ref from Code Obj ->"注册码错误!请与作者联系!"

                                ===>BAD BOY！

:004A38A5 BA78394A00           mov edx, 004A3978

:004A38AA A184564A00           mov eax, dword ptr \[004A5684\]

:004A38AF 8B00                 mov eax, dword ptr \[eax\]

:004A38B1 E88A32FCFF           call 00466B40

:004A38B6 EB4F                 jmp 004A3907

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004A389C(C)

|

:004A38B8 BA02000080           mov edx, 80000002

:004A38BD 8BC3                 mov eax, ebx

:004A38BF E80896FCFF           call 0046CECC

:004A38C4 B101                 mov cl, 01

                                ===>下面是保存注册信息！

\* Possible StringData Ref from Code Obj ->"software\\microsoft\\windows\\currentversion\\hxma"

                                ->"rk"

                                |

:004A38C6 BA9C394A00           mov edx, 004A399C

:004A38CB 8BC3                 mov eax, ebx

:004A38CD E85E96FCFF           call 0046CF30

:004A38D2 8B4DF0               mov ecx, dword ptr \[ebp\-10\]

\* Possible StringData Ref from Code Obj ->"x\_id"

                                |

:004A38D5 BAD8394A00           mov edx, 004A39D8

:004A38DA 8BC3                 mov eax, ebx

:004A38DC E87397FCFF           call 0046D054

:004A38E1 8BC3                 mov eax, ebx

:004A38E3 E8B495FCFF           call 0046CE9C

:004A38E8 8BC3                 mov eax, ebx

:004A38EA E8D9FEF5FF           call 004037C8

:004A38EF 6A00                 push 00000000

\* Possible StringData Ref from Code Obj ->"提示"

                                |

:004A38F1 B970394A00           mov ecx, 004A3970

\* Possible StringData Ref from Code Obj ->"成功注册,重启 ESee 生效"

                                ===>呵呵，胜利女神！

:004A38F6 BAE0394A00           mov edx, 004A39E0

:004A38FB A184564A00           mov eax, dword ptr \[004A5684\]

:004A3900 8B00                 mov eax, dword ptr \[eax\]

:004A3902 E83932FCFF           call 00466B40

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004A387A(C), :004A38B6(U)

|

:004A3907 33C0                 xor eax, eax

:004A3909 5A                   pop edx

:004A390A 59                   pop ecx

:004A390B 59                   pop ecx

:004A390C 648910               mov dword ptr fs:\[eax\], edx

:004A390F 6831394A00           push 004A3931

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004A392F(U)

|

:004A3914 8D45EC               lea eax, dword ptr \[ebp\-14\]

:004A3917 E89C0CF6FF           call 004045B8

:004A391C 8D45F4               lea eax, dword ptr \[ebp\-0C\]

:004A391F BA03000000           mov edx, 00000003

:004A3924 E8B30CF6FF           call 004045DC

:004A3929 C3                   ret
```

—————————————————————————————————

【算法 总 结】：

机器码的 `16` 进制值 `* 2` 的 `16` 进制结果

—————————————————————————————————

【完 美 爆 破】：

```
004A3894 8B55FC             mov edx, dword ptr \[ebp\-04\]

改为： 8B55F8             mov edx, dword ptr \[ebp\-08\]
```

呵呵，与 `004A3891` 处相映成趣。让真码去与真码比较，岂有不 OK 的？让程序自动保存真码！

—————————————————————————————————

【KeyMake 之 {69th} 内存注册机】：

中断地址：`004A3897`

中断次数：`1`

第一字节：`E8`

指令长度：`5`

内存方式：`EAX`

—————————————————————————————————

【注册信息保存】：

```
REGEDIT4

\[HKEY\_LOCAL\_MACHINE\\Software\\Microsoft\\Windows\\CurrentVersion\\hxmark\]

"x\_id"=dword:000759dd
```

—————————————————————————————————

【整 理】：

机器码：`358301`

注册码：`AEF3A`

—————————————————————————————————

```
         ,     \_/

        /| \_.-~/            \\\ ,        青春都一饷

        ( /~   /              \\~-.\ |\\
        \`\\\\  \_/                 \   ~\ )        忍把浮名

        _-~~~-.)  )\_\_/;;,.        \\' //'

        /'_,\ --~   \ ~~~- ,;;\_\_\_( (.-~~~-.        换了破解轻狂

        \`~ _( ,\_.--\\ (     ,;'' /    ~--   /.\_\`\\

        /~~//'   /' \`~\ ) /--..\_, )\_  \`~
```



```
  "  `~"  "     `”    /~'\`\   `\\\\~~\   

                 "     "   "~'  ""

                  Cracked By 巢水工作坊——fly [OCN][FCG]

                                  2003-04-23 3:12  4dd466d0e1e0cc123afbe504da2462ca
```
