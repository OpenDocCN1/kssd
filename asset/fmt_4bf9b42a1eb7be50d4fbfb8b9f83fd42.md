
# 一个十分简单的算法保护程序【原创】

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-376.htm`](https://www.pediy.com/kssd/pediy09/pediy09-376.htm)

> 一个十分简单的算法保护程序

> 日期：2007 年 1 月 29 日 破解人：林海雪原

> 【软件名称】： 软件版本：`3.33`
> 【软件大小】： KB
> 【下载地址】：http://
> 【软件简介】：
> 【软件限制】：
> 【破解声明】：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！
> 【破解工具】：

> 【破解过程】：

> 这是一个十分简单的算法保护程序,特别对新手有用,所以就发到这,希望对新手能有一点帮助!

> 用 `PEID` 查一下:Borland Delphi `6.0` - `7.0` 虽然我们不能迷信这个结果,但先用它看看还是有必要的. 既然是 Delphi 的,就可以用 `DEDE` 分析了.其实这个程序保护较弱,不用 `DEDE` 也是一样的(用字符串参考,你会得到很多有用的东西!!):

```
004E6C45      mov edx,winchm.004E7178          ASCII "Single-user License"
004E6C68      mov edx,winchm.004E7194          ASCII "2-user License"
004E6C8B      mov edx,winchm.004E71AC          ASCII "5-user License"
004E6CAE      mov edx,winchm.004E71C4          ASCII "10-user License"
004E6CD1      mov edx,winchm.004E71DC          ASCII "20-user License"
004E6CF4      mov edx,winchm.004E71F4          ASCII "50-user License"
004E6D17      mov edx,winchm.004E720C          ASCII "Site License"
004E6DBE      mov edx,winchm.004E7178          ASCII "Single-user License"
004E6DE1      mov edx,winchm.004E7194          ASCII "2-user License"
004E6E04      mov edx,winchm.004E71AC          ASCII "5-user License"
004E6E27      mov edx,winchm.004E71C4          ASCII "10-user License"
004E6E4A      mov edx,winchm.004E71DC          ASCII "20-user License"
004E6E6D      mov edx,winchm.004E71F4          ASCII "50-user License"
004E6E90      mov edx,winchm.004E720C          ASCII "Site License"
004E6F3C      mov ecx,winchm.004E7224          ASCII "-user License"
004E6FAA      mov edx,winchm.004E720C          ASCII "Site License"
004E7097      mov ecx,winchm.004E7224          ASCII "-user License"
004E7114      mov edx,winchm.004E720C          ASCII "Site License"
```

> 在 `DEDE` 中,有这样一个窗体会引起你注意:`TfrmRegister`!!

> 事件如下:

> `Register`

```
TfrmRegister.FormCreate                   004E7648   \*(为了防范作者玩火,这应注意一下,不用断的)
TfrmRegister.btnCleanInfoClick            004E7768
TfrmRegister.BitBtn3Click                  004E78FC   ***
TfrmRegister.Action1Update                 004E796C   *
TfrmRegister.Action1Execute                004E79F0   *
TfrmRegister.Label1Click                   004E7B98   *
TfrmRegister._PROC_004E6494                004E6494
TfrmRegister._PROC_004E6558                004E6558
TfrmRegister._PROC_004E6A10                004E6A10
TfrmRegister._PROC_004E6A85                004E6A85
TfrmRegister._PROC_004E6BB8                004E6BB8
TfrmRegister._PROC_004E7235                004E7235
TfrmRegister._PROC_004E72D4                004E72D4
TfrmRegister._PROC_004E73D5                004E73D5
TfrmRegister._PROC_004E7460                004E7460
TfrmRegister._PROC_004E762D                004E762D
TfrmRegister._PROC_004E772C                004E772C
TfrmRegister._PROC_004E7964                004E7964
TfrmRegister._PROC_004E7BA1                004E7BA1
TfrmRegister._PROC_004E7BD0                004E7BD0
TfrmRegister._PROC_004E7BD8                004E7BD8
```

> 在上面的事件中只有加了"\*"的能引人注目,先不管,用 `OD` 载入,在上面 4 处下断在说了,`OD` 中断入口如下:

```
00512914 w> $ 55              push ebp
00512915    . 8BEC            mov ebp,esp
00512917    . B9 06000000     mov ecx,6
0051291C    > 6A 00           push 0
0051291E    . 6A 00           push 0
00512920    . 49              dec ecx
00512921    .^ 75 F9         jnz short winchm.0051291C
00512923    . 51              push ecx
00512924    . 53              push ebx
00512925    . B8 AC245100     mov eax,winchm.005124AC
0051292A    . E8 A93BEFFF     call winchm.004064D8
0051292F    . 33C0            xor eax,eax
00512931    . 55              push ebp
```

> 这是一个非标准的 delphi 入口,作者在 dpr 文件里弄了自己的代码!这说明 `PEID` 识别正确(绝大部份时候都是正确的).在上面 4 处下断,运行!结果未中断,而出来个框框,说"15 days left for trial period",在下面找到这个串:

```
004EAAC1   |.  8B86 10030000   mov eax,[dword ds:esi+310]   ; |
004EAAC7   |.  8945 F4         mov [dword ss:ebp-C],eax     ; |
004EAACA   |.  C645 F8 00      mov [byte ss:ebp-8],0        ; |
004EAACE   |.  8D55 F4         lea edx,[dword ss:ebp-C]     ; |
004EAAD1   |.  33C9            xor ecx,ecx                  ; |
004EAAD3   |.  B8 84AB4E00     mov eax,winchm.004EAB84      ; |ASCII "%d days left for trial period"
004EAAD8   |.  E8 D3F5F1FF     call winchm.0040A0B0         ; \\winchm.0040A0B0
004EAADD   |.  8B55 FC         mov edx,[dword ss:ebp-4]
```

> 这个框框还有个"entr reg code"的按钮,我点了下,立马中断了:

```
004E796C   /.  55              push ebp
004E796D   |.  8BEC            mov ebp,esp
004E796F   |.  6A 00           push 0
004E7971   |.  6A 00           push 0
```

> (`TfrmRegister.Action1Update    004E796C   *`)

> 看看上下文,感觉没有紧要之处,放行后,它要我输入 name 和 License code,现在就各有所爱了,我就用 scxtb 和 123456789 ,点 register,又中断于下:

```
004E79F0   /.  55              push ebp
004E79F1   |.  8BEC            mov ebp,esp
004E79F3   |.  33C9            xor ecx,ecx
004E79F5   |.  51              push ecx
004E79F6   |.  51              push ecx
```

> (`TfrmRegister.Action1Execute    004E79F0   *`)

> 同样,上下看看,这次收获可不小了:

```
004E7AC9   |.  A1 0C485100     mov eax,[dword ds:51480C]
004E7ACE   |.  8B00            mov eax,[dword ds:eax]
004E7AD0   |.  33D2            xor edx,edx
004E7AD2   |.  E8 F5CFF1FF     call winchm.00404ACC
004E7AD7   |.  74 0C           je short winchm.004E7AE5
004E7AD9   |.  B8 687B4E00     mov eax,winchm.004E7B68      ;  ASCII "Thank you!"           ===========>注册成功!!
004E7ADE   |.  E8 15F6F4FF     call winchm.004370F8
004E7AE3   |.  EB 0A           jmp short winchm.004E7AEF
004E7AE5   |>  B8 7C7B4E00     mov eax,winchm.004E7B7C      ;  ASCII "Illegal registration code!"=====>注册失败!!
004E7AEA   |.  E8 09F6F4FF     call winchm.004370F8
004E7AEF   |>  C786 4C020000 0>mov [dword ds:esi+24C],1
```

> 看来正确与否,就此见分晓了!主要代码如下:

```
004E7A4E   |.  E8 D146F9FF     call winchm.0047C124
004E7A53   |.  8B45 F4         mov eax,[dword ss:ebp-C]     ;  (ASCII "scxtb")
004E7A56   |.  8D55 F8         lea edx,[dword ss:ebp-8]
004E7A59   |.  E8 360DF2FF     call winchm.00408794
004E7A5E   |.  8B4D F8         mov ecx,[dword ss:ebp-8]
004E7A61   |.  BA 487B4E00     mov edx,winchm.004E7B48      ;  ASCII "RegName"
004E7A66   |.  8BC3            mov eax,ebx
004E7A68   |.  E8 BFAFF5FF     call winchm.00442A2C         ;  写注册表
004E7A6D   |.  8D45 EC         lea eax,[dword ss:ebp-14]
004E7A70   |.  50              push eax
004E7A71   |.  8D55 E0         lea edx,[dword ss:ebp-20]
004E7A74   |.  8B86 0C030000   mov eax,[dword ds:esi+30C]
004E7A7A   |.  E8 A546F9FF     call winchm.0047C124
004E7A7F   |.  8B45 E0         mov eax,[dword ss:ebp-20]    ;  (ASCII "123456")
004E7A82   |.  8D55 E4         lea edx,[dword ss:ebp-1C]
004E7A85   |.  E8 0A0DF2FF     call winchm.00408794
004E7A8A   |.  8B45 E4         mov eax,[dword ss:ebp-1C]
004E7A8D   |.  8D55 E8         lea edx,[dword ss:ebp-18]
004E7A90   |.  E8 AF0AF2FF     call winchm.00408544         ;  转为大写
004E7A95   |.  8B45 E8         mov eax,[dword ss:ebp-18]    ; |
004E7A98   |.  B1 01           mov cl,1                     ; |加解密的标志:1,加密
```
