
# 行政法律文书管理系统

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-283.htm`](https://www.pediy.com/kssd/pediy07/pediy7-283.htm)

暴破一文书软件（Delphi 代码分析;找打印代码入口）

**【破解作者】** 蓝色光芒

**【使用工具】** `Ollydbg`, `PEiD`, `AspackDie1.4`, `UEdit32`

**【破解平台】** `XP`

**【软件名称】** `dbimp.exe` (V:3.7.7.7590)

**【下载地址】** 天天下载

**【软件简介】** 《XX 行政法律文书管理系统》严格按照公安部 2004 年新颁发的《公安机关办理行政案件程序规定》的要求，完整比对《公安行政法律文书（式样）》......

**【软件大小】** 991KB (Unpack 后 3.82M)

**【加壳方式】** `ASPack 2.12`

**【破解声明】** 本文章仅作技术研究，不对任何阅读后产生的事件负责

---

## 破解内容

### 一、查壳

用 `PEiD` 发现是 `Aspack2.12` 加的壳，用 `AspackDie1.4` 脱之，再用 `PEiD` 查看是 Delphi 编写

### 二、分析暴破

1. 用 `OD` 载入，直接运行，不报错误，有异常发生
2. 重新载入，直接到输入注册码的表单，输入一个注册码，HOHOHO:1234567890（后来发现这里如果注册号不是数字，前面判断直接退出了)，点“注册”后弹出错误提示，在 `OD` 里下断 `BP MessageBoxA`，继续注册，程序停在：

```
77D3ADD7 u>   833D C4D3D677 00 cmp     dword ptr ds:[77D6D3C4], 0
77D3ADDE     0F85 377E0100    jnz     77D52C1B
77D3ADE4     6A 00            push    0
77D3ADE6     FF7424 14        push    dword ptr ss:[esp+14]
77D3ADEA     FF7424 14        push    dword ptr ss:[esp+14]
77D3ADEE     FF7424 14        push    dword ptr ss:[esp+14]
77D3ADF2     FF7424 14        push    dword ptr ss:[esp+14]
77D3ADF6     E8 03000000      call    MessageBoxExA
77D3ADFB     C2 1000          ret     10
```

`Ctrl+F9`，到 `RET` 位置后，`F8`

```
004ABDB0   |.   53           push    ebx                     ; /Style
004ABDB1   |.   57           push    edi                     ; |Title
004ABDB2   |.   56           push    esi                     ; |Text
004ABDB3   |.   8B45 FC      mov     eax, dword ptr ss:[ebp-4]     ; |
004ABDB6   |.   8B40 30      mov     eax, dword ptr ds:[eax+30]    ; |
004ABDB9   |.   50           push    eax                     ; |hOwner
004ABDBA   |.   E8 A1C2F5FF   call    <jmp.&user32.MessageBoxA>     ; \MessageBoxA
004ABDBF   |.   8945 F8      mov     dword ptr ss:[ebp-8], eax  <-----------停在这里
004ABDC2   |.   33C0         xor     eax, eax
004ABDC4   |.   5A           pop     edx
004ABDC5   |.   59           pop     ecx
004ABDC6   |.   59           pop     ecx
004ABDC7   |.   64:8910      mov     dword ptr fs:[eax], edx
004ABDCA   |.   68 30BE4A00 push    004ABE30
004ABDCF   |>   8B45 EC      mov     eax, dword ptr ss:[ebp-14]
004ABDD2   |.   3B45 E8      cmp     eax, dword ptr ss:[ebp-18]
004ABDD5   |.   74 38        je      short 004ABE0F
004ABDD7   |.   6A 1D        push    1D
004ABDD9   |.   6A 00        push    0
004ABDDB   |.   6A 00        push    0
004ABDDD   |.   8B4D B8      mov     ecx, dword ptr ss:[ebp-48]
004ABDE0   |.   8B55 B0      mov     edx, dword ptr ss:[ebp-50]
004ABDE3   |.   2BCA         sub     ecx, edx
004ABDE5   |.   D1F9         sar     ecx, 1
004ABDE7   |.   79 03        jns     short 004ABDEC
004ABDE9   |.   83D1 00      adc     ecx, 0
004ABDEC   |>   03CA         add     ecx, edx
004ABDEE   |.   51           push    ecx
004ABDEF   |.   8B55 B4      mov     edx, dword ptr ss:[ebp-4C]
004ABDF2   |.   8B45 AC      mov     eax, dword ptr ss:[ebp-54]
004ABDF5   |.   2BD0         sub     edx, eax
004ABDF7   |.   D1FA         sar     edx, 1
004ABDF9   |.   79 03        jns     short 004ABDFE
004ABDFB   |.   83D2 00      adc     edx, 0
004ABDFE   |>   03D0         add     edx, eax                  ; |
004ABE00   |.   52           push    edx                     ; |X
004ABE01   |.   6A 00        push    0                       ; |InsertAfter = HWND_TOP
004ABE03   |.   8B45 FC      mov     eax, dword ptr ss:[ebp-4]     ; |
004ABE06   |.   8B40 30      mov     eax, dword ptr ds:[eax+30]    ; |
004ABE09   |.   50           push    eax                     ; |hWnd
004ABE0A   |.   E8 A1C3F5FF   call    <jmp.&user32.SetWindowPos>    ; \SetWindowPos
004ABE0F   |>   8B45 F0      mov     eax, dword ptr ss:[ebp-10]
```

由后面的 `SetWindowPos` 和前面 `GetWindowPos` 的调用，不难看出这是调用的 `Application.MessageBox` 来显示的提示框

因此继续 `Ctrl+F9,F8`，来到：

```
004ABE30     .   8B45 F8      mov     eax, dword ptr ss:[ebp-8]
004ABE33     .   5F           pop     edi                     ;  0012F33C
004ABE34     .   5E           pop     esi
004ABE35     .   5B           pop     ebx
004ABE36     .   8BE5         mov     esp, ebp
004ABE38     .   5D           pop     ebp
004ABE39     .   C2 0400      ret     4
```

看来还作了一次封装

不理会，继续 `Ctrl+F9,F8`

```
0060B316    > \6A 00          push    0
0060B318    .   B9 58B46000   mov     ecx, 0060B458
0060B31D    .   BA A4B46000   mov     edx, 0060B4A4
0060B322    .   A1 ACFA6800   mov     eax, dword ptr ds:[68FAAC]
0060B327    .   8B00          mov     eax, dword ptr ds:[eax]
0060B329    .   E8 A609EAFF   call    004ABCD4  <--调用 Application.MessageBox
0060B32E    >   33C0          xor     eax, eax <--停在这里
0060B330    .   5A           pop     edx
0060B331    .   59           pop     ecx
0060B332    .   59           pop     ecx
```

然后往上找，一直找到：

```
0060AF4C    .   55           push    ebp
0060AF4D    .   8BEC         mov     ebp, esp
0060AF4F    .   B9 0D000000 mov     ecx, 0D
0060AF54    >   6A 00        push    0
0060AF56    .   6A 00        push    0
0060AF58    .   49           dec     ecx
```

这一段，理论上讲，如果没有封装，这就是按钮的入口地址，既:`ButtonClick`

然后下断 `0060AF4C`，继续回去注册

停下来后，`F8` 向前来到这里

```
0060AFD5    .   58           pop     eax
0060AFD6    .   E8 6D9FDFFF call    00404F48
0060AFDB    .   0F85 EA000000 jnz     0060B0CB          //关键跳转改成 JZ 0060B0CB F9 后提示成功.^-^
0060AFE1    .   33C0         xor     eax, eax
0060AFE3    .   55           push    ebp
0060AFE4    .   68 BCB06000 push    0060B0BC
0060AFE9    .   64:FF30     push    dword ptr fs:[eax]
0060AFEC    .   64:8920     mov     dword ptr fs:[eax], esp
0060AFEF    .   BA 02000080 mov     edx, 80000002
0060AFF4    .   8B45 FC      mov     eax, dword ptr ss:[ebp-4]
0060AFF7    .   E8 8004E4FF call    0044B47C
0060AFFC    .   B1 01        mov     cl, 1
0060AFFE    .   BA 20B46000 mov     edx, 0060B420                 ;  ASCII "SoftWare\\Dbimp\\Dbimp1.0"
0060B003    .   8B45 FC      mov     eax, dword ptr ss:[ebp-4]
0060B006    .   E8 D504E4FF call    0044B4E0                      //调用 Reg.OpenKey()
0060B00B    .   84C0         test    al, al
```

提示成功后发现还是非注册版，日

回到刚才这里，发现是先写如了注册表的。：）

`Ctrl+F2`，`BP RegQueryValueExA`

`F9`

一直当出现了 `'RegName'` 时停止

`Ctrl+F9,F8` 来到调用地

```
0044B604   /$  53           push    ebx
0044B605   |.   56           push    esi
0044B606   |.   57           push    edi
0044B607   |.   55           push    ebp
0044B608   |.   51           push    ecx
0044B609   |.   8BE9         mov     ebp, ecx
0044B60B   |.   8BFA         mov     edi, edx
0044B60D   |.   8BF0         mov     esi, eax
0044B60F   |.   8BC5         mov     eax, ebp
0044B611   |.   33C9         xor     ecx, ecx
0044B613   |.   BA 08000000 mov     edx, 8
0044B618   |.   E8 3B7DFBFF call    00403358
0044B61D   |.   8D45 04      lea     eax, dword ptr ss:[ebp+4]
0044B620   |.   50           push    eax
0044B621   |.   6A 00        push    0
0044B623   |.   8D4424 08    lea     eax, dword ptr ss:[esp+8]
0044B627   |.   50           push    eax
0044B628   |.   6A 00        push    0
0044B62A   |.   8BC7         mov     eax, edi
0044B62C   |.   E8 CB99FBFF call    00404FFC
0044B631   |.   50           push    eax                     ; |ValueName
0044B632   |.   8B46 04      mov     eax, dword ptr ds:[esi+4]     ; |
0044B635   |.   50           push    eax                     ; |hKey
0044B636   |.   E8 35BFFBFF call    <jmp.&advapi32.RegQueryValueE>; \RegQueryValueExA
```



```
0044B63B   |.  85C0           test    eax, eax <----停在这里  

0044B63D   |.  0F94C3        sete    bl  

0044B640   |.  8B0424        mov     eax, dword ptr ss:[esp]  

0044B643   |.  E8 6CFDFFFF   call    0044B3B4  

0044B648   |.  8845 00      mov     byte ptr ss:[ebp], al  

0044B64B   |.  8BC3         mov     eax, ebx  

0044B64D   |.  5A           pop     edx  

0044B64E   |.  5D           pop     ebp  

0044B64F   |.  5F           pop     edi  

0044B650   |.  5E           pop     esi  

0044B651   |.  5B           pop     ebx  

0044B652   \.  C3           ret  
```

由以上代码看出，以上程序是 `TRegistry` 的类方法，

因此我们继续 `Ftrl+F9,F8`

一直来到这里：

```
00674918    .  B1 01          mov     cl, 1  

0067491A    .  BA DC4E6700    mov     edx, 00674EDC                    ;  ASCII "SoftWare\\Dbimp\\Dbimp1.0"  

0067491F    .  8B45 E0        mov     eax, dword ptr ss:[ebp-20]  

00674922    .  E8 B96BDDFF    call    0044B4E0  

00674927    .  84C0           test    al, al  

00674929    .  74 10          je      short 0067493B  

0067492B    .  8D4D F8        lea     ecx, dword ptr ss:[ebp-8]  

0067492E    .  BA FC4E6700    mov     edx, 00674EFC                    ;  ASCII "RegName"  

00674933    .  8B45 E0        mov     eax, dword ptr ss:[ebp-20]  

00674936    .  E8 6D6DDDFF    call    0044B6A8  

0067493B    >  8D4D F4        lea     ecx, dword ptr ss:[ebp-C] <-----停在这里  

0067493E    .  BA 0C4F6700    mov     edx, 00674F0C                    ;  ASCII "RegID"  

00674943    .  8B45 E0        mov     eax, dword ptr ss:[ebp-20]  

00674946    .  E8 5D6DDDFF    call    0044B6A8  

0067494B    .  837D F4 00     cmp     dword ptr ss:[ebp-C], 0  

0067494F    .  0F84 EE000000  je      00674A43  

00674955    .  837D F8 00     cmp     dword ptr ss:[ebp-8], 0  

00674959    .  0F84 E4000000  je      00674A43  

0067495F    .  8D4D DC        lea     ecx, dword ptr ss:[ebp-24]  

00674962    .  BA 1C4F6700    mov     edx, 00674F1C                    ;  ASCII "HDDBIP"  

00674967    .  8B45 F4        mov     eax, dword ptr ss:[ebp-C]  

0067496A    .  E8 FDE0FEFF    call    00662A6C  

0067496F    .  8B45 DC        mov     eax, dword ptr ss:[ebp-24]  

00674972    .  50            push    eax  

00674973    .  8D55 D8        lea     edx, dword ptr ss:[ebp-28]  

00674976    .  8B45 F8        mov     eax, dword ptr ss:[ebp-8]  

00674979    .  E8 F64ED9FF    call    00409874  

0067497E    .  8B55 D8        mov     edx, dword ptr ss:[ebp-28]  

00674981    .  58            pop     eax  

00674982    .  E8 C105D9FF    call    00404F48    <-------关键调用  

00674987    .  0F84 97000000  jnz     00674A24   <-------关键跳转  

0067498D    .  C605 38A46B00 >mov      byte ptr ds:[6BA438], 1  

00674994    .  C605 39A46B00 >mov      byte ptr ds:[6BA439], 1  

0067499B    .  BA 2C4F6700    mov     edx, 00674F2C  

006749A0    .  8B45 FC        mov     eax, dword ptr ss:[ebp-4]  

006749A3    .  E8 6060E1FF    call    0048AA08  

006749A8    .  8B45 FC        mov     eax, dword ptr ss:[ebp-4]  

006749AB    .  8B80 9C030000  mov     eax, dword ptr ds:[eax+39C]  

006749B1    .  8B80 08020000  mov     eax, dword ptr ds:[eax+208]  

006749B7    .  BA 02000000    mov     edx, 2  

006749BC    .  E8 C770DEFF    call    0045BA88  

006749C1    .  50            push    eax  
```

我们在说说这一段程序

```
0067493E    .  BA 0C4F6700    mov     edx, 00674F0C                    ;  ASCII "RegID"  

00674943    .  8B45 E0        mov     eax, dword ptr ss:[ebp-20]  

00674946    .  E8 5D6DDDFF    call    0044B6A8  

0067494B    .  837D F4 00     cmp     dword ptr ss:[ebp-C], 0  

0067494F    .  0F84 EE000000  je      00674A43  
```

`0067493E`,`00674943`,`00674946` 这 3 行实际上是

`RegID := Reg.ReadString('RegID');`

`RegID` 是一个字符串变量,`Reg` 是一个 `TRegistry` 类的实例

`0067494B` 这一行翻译成 Delphi 程序就是

`if Length(RegID)>0 then begin`

`else`

  `00674A43: XXXX`

`end`

逐行向下：

```
00674982    .  E8 C105D9FF    call    00404F48    <-------关键调用  

00674987    .  0F84 97000000  jnz     00674A24   <-------关键跳转  
```

发现跳转实现，试着修改 `jz 00674A24`

`F9`，呵呵，非注册字样没有了，事情向前了一大步。：）

一般情况，这类软件都会在打印那里作手脚。

比如注册版不能打印

来到打印预览窗口，果然不能打印，看来修改不切底

`BP MessageBoxA`

点打印按钮

停下后，`Ctrl+F9,F8` 发现又是调用的 `Application.MessageBox;`

于是继续 `Ctrl+F9,F8,Ctrl+F9,F8`

来到：

```
0056A44C    .  55            push    ebp  

0056A44D    .  8BEC         mov     ebp, esp  

0056A44F    .  33C0         xor     eax, eax  

0056A451    .  55            push    ebp  

0056A452    .  68 83A45600   push    0056A483  

0056A457    .  64:FF30      push    dword ptr fs:[eax]  

0056A45A    .  64:8920      mov     dword ptr fs:[eax], esp  

0056A45D    .  6A 00        push    0  

0056A45F    .  B9 8CA45600  mov     ecx, 0056A48C  

0056A464    .  BA 98A45600  mov     edx, 0056A498  

0056A469    .  A1 ACFA6800  mov     eax, dword ptr ds:[68FAAC]  

0056A46E    .  8B00        mov     eax, dword ptr ds:[eax]  

0056A470    .  E8 5F18F4FF call    004ABCD4  

0056A475    .  33C0         xor     eax, eax  

0056A477    .  5A            pop     edx  

0056A478    .  59            pop     ecx  

0056A479    .  59            pop     ecx  

0056A47A    .  64:8910      mov     dword ptr fs:[eax], edx  

0056A47D    .  68 8AA45600  push    0056A48A  

0056A482    >  C3            ret                     ;  RET used as a jump to 0056A48A  

0056A483    .^ E9 D89FE9FF  jmp     00404460  

0056A488    .^ EB F8        jmp     short 0056A482  

0056A48A    >  5D            pop     ebp  

0056A48B    .  C3            ret  
```

咋一看，丫，象是作了 `SEH`，仔细一看，呵呵只不过是 Delphi 的异常处理

翻译以上代码竟然是：

```
Procedure xxxxxClick(Sender: TObject);  

begin  

  Try  

    Application.MessageBox('由于你不是注册用户，该功能不能使用.....','打印报表',64)  

   except  

   end;  

end;  
```

仅此而已，我晕，怎么回事，

```
0056A48A    >  5D            pop     ebp  

0056A48B    .  C3            ret  
```

后，`F8` 来到：

```
0048BFF2   |> \\66:83BB 220100>cmp     word ptr ds:[ebx+122], 0  

0048BFFA   |.  74 0E        je      short 0048C00A  

0048BFFC   |.  8BD3         mov     edx, ebx  

0048BFFE   |.  8B83 24010000 mov     eax, dword ptr ds:[ebx+124]  

0048C004   |.  FF93 20010000 call    dword ptr ds:[ebx+120]  

0048C00A   |>  5B            pop     ebx                   ;  00D446D0<---停在这里  

0048C00B   \\.  C3            ret  

0048C00C   /$  53            push    ebx  

0048C004   |.  FF93 20010000 call    dword ptr ds:[ebx+120]  
```

从这句看来，很明显是在调用类的方法，以上代码翻译后就是

`if Self.fOnClick<>NIL then`

  `Self.fOnClick(Self);`

因此不难看出，那个打印按钮的真正代码也就是那个 `Try.......`

怎么回事呀，难道说试用版里根本就没有打印的代码，只有正式版才有。

`Tel` 一个正版用户，说注册后就可以使用，汗！！

估计是代码指向的问题，就下断 `0048C004`

凡是对象的那个类的 `OnClick` 事件的调用都拦下来研究研究

然后点打印功能后面一个按钮

：停下后，`F7`，来到

```
0056B5B4    .  55            push    ebp  

0056B5B5    .  8BEC         mov     ebp, esp  

0056B5B7    .  83C4 F4      add     esp, -0C  

0056B5BA    .  53            push    ebx  

0056B5BB    .  56            push    esi  
```

然后根据经验，在附近找找以 `push ebp` 或者 `push ebx` 开头的程序段，

首先怀疑仅靠着的前面一段：

```
0056B560    .  53            push    ebx  

0056B561    .  56            push    esi  

0056B562    .  8B1D 70F86800 mov     ebx, dword ptr ds:[68F870]    ;  unpacked.00690EC0  

0056B568    .  8B35 20F36800 mov     esi, dword ptr ds:[68F320]    ;  unpacked.006BA0EC  

0056B56E    .  8B06        mov     eax, dword ptr ds:[esi]  

0056B570    .  8B80 D4010000 mov     eax, dword ptr ds:[eax+1D4]  

0056B576    .  E8 4D03F6FF call    004CB8C8  

0056B57B    .  EB 22        jmp     short 0056B59F  

0056B57D    >  8B03        mov     eax, dword ptr ds:[ebx]  

0056B57F    .  E8 C00C0000 call    0056C244  

0056B584    .  8B03        mov     eax, dword ptr ds:[ebx]  

0056B586    .  E8 510E0000 call    0056C3DC  

0056B58B    .  8B03        mov     eax, dword ptr ds:[ebx]  

0056B58D    .  E8 721DFDFF call    0053D304  

0056B592    .  8B06        mov     eax, dword ptr ds:[esi]  
```



```
0056B594    .   8B80 D4010000   mov     eax, dword ptr ds:[eax+1D4]  

0056B59A    .   E8 4106F6FF     call    004CBBE0  

0056B59F    >   8B06            mov     eax, dword ptr ds:[esi]  

0056B5A1    .   8B80 D4010000   mov     eax, dword ptr ds:[eax+1D4]  

0056B5A7    .   80B8 A1000000   >cmp    byte ptr ds:[eax+A1], 0  

0056B5AE    .^  74 CD           je      short 0056B57D  

0056B5B0    .   5E              pop     esi  

0056B5B1    .   5B              pop     ebx  

0056B5B2    .   C3              ret  
```

下断 `0056B560` 后，无论怎么运行怎么点击，都不能运行到这里，更加怀疑了，

于是在那个打印按钮的事件开始处改成：`JMP 0056B560`

`F9` 运行，发现呵呵，能够打印了：）

但是打印后，程序要出错。于是怀疑还有地方不干净，仔细研究这一段代码，

发现这两句：

```
0056B57F    .   E8 C00C0000     call    0056C244  
0056B586    .   E8 510E0000     call    0056C3DC  
```

似乎是在隐藏或者释放这个预览对象，干脆 `NOP` 掉，

运行看看，成了。。。

不报错，能正常打印。

然后还要修改一个编辑窗口的打印按钮，让那个打印按钮的事件和预览的事件一样。

然后还要修改一处，软件退出时的注册提示，整个修改就算完了。

如果要研究算法：

就进这里：

```
00674982    .   E8 C105D9FF     call    00404F48    <-------关键调用
```

对应修改 `EXE` 后，程序正常运行。。。

`4dd466d0e1e0cc123afbe504da2462ca`
