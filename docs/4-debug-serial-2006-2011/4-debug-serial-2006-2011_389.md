
# Ashampoo Magical Snap 2.31 算法详细分析

> 原文：[`www.pediy.com/kssd/pediy10/77691.html`](https://www.pediy.com/kssd/pediy10/77691.html)

**【破文标题】** `Ashampoo Magical Snap 2.31` 算法分析

**【破文作者】** `Playboysen`

**【作者邮箱】** `playboysen@126.com`

**【破解工具】** `PEiD`, `OD`

**【破解平台】** `Windows XP SP2`

**【软件名称】** `Ashampoo Magical Snap 2.31(20081105)`

**【软件大小】** `12.7 MB`

**【软件授权】** 共享版($14.99)

**【软件语言】** 英文

**【原版下载】** `http://www.ashampoo.com/dl/0224/ashampoo_magicalsnap2_se.exe`

**【保护方式】** 注册码

**【软件简介】** `Magical Snap` 是一款专业的屏幕截图工具。它添加了编辑工具和效果工具还有非常华丽的用户界面。特点：3 种不同的抓图模式；多张抓图；紧密结合--支持全屏抓图；抓图编辑工具；高质量效果图容易使用的鼠标工具。支持抓取任意形状、滚动窗口等。

**【破解声明】** 一点心得，愿与大家分享 `o(∩_∩)o` 版权所有，转载注明作者！

**【破解内容】**

初步踩点：软件无壳，`VC7.0` 编译，单一注册码保护，注册有错误提示，未注册所抓图片有文字水印。

假设注册码为 `AMASAo-77nhEB-96HD2N`（这里说明一下，注册码并不是一开始就假设成这样的，我一开始粗略跟踪用的假码是 `playboysen963852`，跟踪一遍将假码修正为 `playbo-ysenhu-an9638`，再跟踪一遍修正假码为 `AMASbo-ysenhu-an9638`，再跟踪一次将假码修正为 `AMASAo-77enhu-an9638` 的，第五遍跟踪才将注册码修正成这样的(当然其实这已经被“修正”为真注册码了`^\_^`)自己跟踪时可随便输入一个假码，自由练习）

尝试 `MessageBoxA`、`MessageBoxW`、`GetDlgItemTextA` 断错误提示无果，最终发现软件使用的是 `GetDlgItemTextW` 断点，汗~~~

```
004455DE   > \53               push ebx                                  ;  Case 1 of switch 0044559C
004455DF   .  55              push ebp
004455E0   .  8BAC24 380A0000 mov ebp,dword ptr ss:[esp+A38]
004455E7   .  57              push edi
004455E8   .  6A 20           push 20                                   ; /Count = 20 (32.)
004455EA   .  8D5424 28       lea edx,dword ptr ss:[esp+28]             ; |
004455EE   .  52              push edx                                  ; |Buffer
004455EF   .  68 F1030000     push 3F1                                  ; |ControlID = 3F1 (1009.)
004455F4   .  55              push ebp                                  ; |hWnd
004455F5   .  FF15 58144800   call dword ptr ds:[<&USER32.GetDlgItemTextW>]  ; \GetDlgItemTextW
004455FB   .  8D4424 24       lea eax,dword ptr ss:[esp+24]             ;  注册码出现
004455FF   .  50              push eax                                  ; /Arg1
00445600   .  8D4C24 18       lea ecx,dword ptr ss:[esp+18]             ; |
00445604   .  E8 47DAFBFF     call ashsnap.00403050                     ; \关键处，F7 进入
00445609   .  68 4C774E00     push ashsnap.004E774C                     ; /Arg1 = 004E774C
0044560E   .  8D4C24 20       lea ecx,dword ptr ss:[esp+20]             ; |
00445612   .  C78424 380A0000>mov dword ptr ss:[esp+A38],0             ; |
0044561D   .  E8 2EDAFBFF     call ashsnap.00403050                     ; \同上，关键
```

跟入 `00445604` 处关键算法 `call`

代码:

```
00403050  /$  83EC 40         sub esp,40
00403053  |.  8B4424 44       mov eax,dword ptr ss:[esp+44]
......
00403068  |.  C706 00000000   mov dword ptr ds:[esi],0
0040306E  |.  E8 0DF50200     call ashsnap.00432580                     ;  kernel32.WideCharToMultiByte
00403073  |.  8B15 98204800   mov edx,dword ptr ds:[482098]
......
004030C3  |.  52              push edx
004030C4  |.  E8 573A0500     call ashsnap.00456B20                     ;  关键 call,F7 进入
004030C9  |.  83C4 20         add esp,20
```

跟入 `004030C4` 关键 `call`

代码:

```
00456B20  /$  8B4424 14       mov eax,dword ptr ss:[esp+14]
00456B24  |.  8B4C24 10       mov ecx,dword ptr ss:[esp+10]
......
00456B3F  |.  55              push ebp
00456B40  |.  E8 EBFDFFFF     call ashsnap.00456930                     ;  关键 call,F7 进入
00456B45  |.  83C4 14         add esp,14
00456B48  |.  84C0            test al,al
00456B4A  |.  74 20           je short ashsnap.00456B6C
```

这里提醒大家，分析算法的时候，在比较关键的地方遇到 `call` 应该先进入大致浏览一下，否则你很可能错过关键处，导致你接下去无从下手。

好，我们接着跟入 `00456B40` 关键 `call`

代码:



```
00456930   /$  83EC 44         sub esp,44
00456933   |.  83C9 FF        or ecx,FFFFFFFF
00456936   |.  33C0           xor eax,eax
00456938   |.  53            push ebx
00456939   |.  55            push ebp
0045693A   |.  56            push esi
0045693B   |.  8B7424 54      mov esi,dword ptr ss:[esp+54]
0045693F   |.  57            push edi
00456940   |.  8BFE          mov edi,esi                                  ;  注册码放入 EDI
00456942   |.  F2:AE         repne scas byte ptr es:[edi]                 ;  计算注册码的长度
00456944   |.  F7D1          not ecx
00456946   |.  49            dec ecx
00456947   |.  83F9 14       cmp ecx,14                                   ;  注册码必须 20 位
0045694A   |.  0F85 BE010000  jnz ashsnap.00456B0E
00456950   |.  8A4E 06       mov cl,byte ptr ds:[esi+6]                  ;  注册码第七位必须是"-"
00456953   |.  B0 2D         mov al,2D
00456955   |.  3AC8          cmp cl,al
00456957   |.  0F85 B1010000  jnz ashsnap.00456B0E
0045695D   |.  3846 0D       cmp byte ptr ds:[esi+D],al                  ;  注册码第十四位必须是"-"
00456960   |.  0F85 A8010000  jnz ashsnap.00456B0E
00456966   |.  6A 04         push 4
00456968   |.  8D4424 20      lea eax,dword ptr ss:[esp+20]
0045696C   |.  56            push esi
0045696D   |.  50            push eax
0045696E   |.  E8 BD24FFFF    call ashsnap.00448E30                      ;  提取注册码前四位
00456973   |.  8D6E 04       lea ebp,dword ptr ds:[esi+4]
00456976   |.  6A 01         push 1
00456978   |.  8D4C24 68      lea ecx,dword ptr ss:[esp+68]
0045697C   |.  55            push ebp
0045697D   |.  51            push ecx
0045697E   |.  E8 AD24FFFF    call ashsnap.00448E30                      ;  提取注册码第五位
00456983   |.  8D56 05       lea edx,dword ptr ds:[esi+5]
00456986   |.  6A 01         push 1
00456988   |.  8D4424 48      lea eax,dword ptr ss:[esp+48]
0045698C   |.  52            push edx
0045698D   |.  50            push eax
0045698E   |.  E8 9D24FFFF    call ashsnap.00448E30                      ;  提取注册码第六位("-"被忽略)
00456993   |.  8D4E 07       lea ecx,dword ptr ds:[esi+7]
00456996   |.  6A 02         push 2
00456998   |.  8D5424 38      lea edx,dword ptr ss:[esp+38]
0045699C   |.  51            push ecx
0045699D   |.  52            push edx
0045699E   |.  E8 8D24FFFF    call ashsnap.00448E30                      ;  提取注册码第八九位
004569A3   |.  8D46 09       lea eax,dword ptr ds:[esi+9]
004569A6   |.  6A 02         push 2
004569A8   |.  8D4C24 61      lea ecx,dword ptr ss:[esp+61]
004569AC   |.  50            push eax
004569AD   |.  51            push ecx
004569AE   |.  E8 7D24FFFF    call ashsnap.00448E30                      ;  提取注册码第十、十一位
004569B3   |.  8D56 0B       lea edx,dword ptr ds:[esi+B]
004569B6   |.  6A 02         push 2
004569B8   |.  8D4424 54      lea eax,dword ptr ss:[esp+54]
004569BC   |.  52            push edx
004569BD   |.  50            push eax
004569BE   |.  E8 6D24FFFF    call ashsnap.00448E30                      ;  分析注册码第十二、十三位
004569C3   |.  83C4 48       add esp,48
004569C6   |.  8D4E 0E       lea ecx,dword ptr ds:[esi+E]
004569C9   |.  8D5424 2F      lea edx,dword ptr ss:[esp+2F]
004569CD   |.  6A 03         push 3
004569CF   |.  51            push ecx
004569D0   |.  52            push edx
004569D1   |.  E8 5A24FFFF    call ashsnap.00448E30                      ;  分离注册码 15-17 位
004569D6   |.  8D46 11       lea eax,dword ptr ds:[esi+11]
004569D9   |.  6A 02         push 2
004569DB   |.  8D4C24 26      lea ecx,dword ptr ss:[esp+26]
004569DF   |.  50            push eax
004569E0   |.  51            push ecx
004569E1   |.  E8 4A24FFFF    call ashsnap.00448E30
004569E6   |.  8D56 13       lea edx,dword ptr ds:[esi+13]
004569E9   |.  6A 01         push 1
004569EB   |.  8D4424 4E      lea eax,dword ptr ss:[esp+4E]
004569EF   |.  52            push edx
004569F0   |.  50            push eax
004569F1   |.  E8 3A24FFFF    call ashsnap.00448E30                      ;  分离出注册码最后一位
004569F6   |.  8D4C24 50      lea ecx,dword ptr ss:[esp+50]              ;  字符串组合“onh96HN”
......
00456A54   |.  E8 D723FFFF    call ashsnap.00448E30
00456A59   |.  8B7C24 68      mov edi,dword ptr ss:[esp+68]             ;  固定字符串"sk()44$$GFSNM099023$"
00456A5D   |.  8D4424 40      lea eax,dword ptr ss:[esp+40]             ;  字符串组合"onh96HNA77AMAS"
00456A61   |.  57            push edi
00456A62   |.  6A 0E         push 0E                                    ;  常数 14(参数之一)
00456A64   |.  50            push eax
00456A65   |.  E8 46040000    call ashsnap.00456EB0                     ;  截取"sk()44$$GFSNM099023$"后 6 位放入 EDX
00456A6A   |.  57            push edi
00456A6B   |.  8D4C24 50      lea ecx,dword ptr ss:[esp+50]
00456A6F   |.  6A 0E         push 0E                                    ;  常数 14(参数之一)
00456A71   |.  51            push ecx
00456A72   |.  E8 B9030000    call ashsnap.00456E30                     ;  这里直接关系到 00456A8E 的值,关键
00456A77   |.  50            push eax
00456A78   |.  8D5424 4C      lea edx,dword ptr ss:[esp+4C]
00456A7C   |.  68 B0B94900    push ashsnap.0049B9B0                     ;  %04x
00456A81   |.  52            push edx
00456A82   |.  E8 6B0AFFFF    call ashsnap.004474F2
00456A87   |.  83C4 30       add esp,30
00456A8A   |.  8D7C24 14      lea edi,dword ptr ss:[esp+14]             ;  从注册码中提取出的值"EBD2"
00456A8E   |.  8D4424 24      lea eax,dword ptr ss:[esp+24]             ;  这个值是上面的关键 call 计算出的
00456A92   |>  8A10           /mov dl,byte ptr ds:[eax]                 ;  这一段循环很显然是比较上面两个值是否相等
00456A94   |.  8ACA           |mov cl,dl
......
00456AAE   |.  3ACB          |cmp cl,bl
00456AB0   |.^ 75 E0         \\jnz short ashsnap.00456A92
00456AB2   |>  33C0          xor eax,eax
00456AB4   |.  EB 05        jmp short ashsnap.00456ABB
```

追入 `00456A72` 关键处

代码:



# 汇编代码分析

## 初始计算逻辑

单步走出此函数，执行以下代码片段。该部分主要计算字符串长度及基于前三位的初始值，并调用后续运算函数。

```assembly
00456E30  /$  8B5424 0C       mov edx,dword ptr ss:[esp+C]
00456E34  |.  57            push edi
00456E35  |.  8BFA          mov edi,edx
00456E37  |.  83C9 FF       or ecx,FFFFFFFF
00456E3A  |.  33C0          xor eax,eax
00456E3C  |.  F2:AE         repne scas byte ptr es:[edi]           ; 求"sk()44$$GFSNM099023$"长度
00456E3E  |.  F7D1          not ecx
00456E40  |.  49            dec ecx
00456E41  |.  5F            pop edi
00456E42  |.  83F9 03       cmp ecx,3
00456E45  |.  72 18         jb short ashnap.00456E5F                ; 我们设"sk()44$$GFSNM099023$"为 Y
00456E47  |.  0FBE42 01     movsx eax,byte ptr ds:[edx+1]          ; Y 的第二位 k
00456E4B  |.  0FBE0A        movsx ecx,byte ptr ds:[edx]            ; Y 的第一位 s
00456E4E  |.  0FBE52 02     movsx edx,byte ptr ds:[edx+2]          ; Y 的第三位 (
00456E52  |.  C1E0 04       shl eax,4                               ; 对这三位做运算
00456E55  |.  0BC1          or eax,ecx
00456E57  |.  C1E0 10       shl eax,10
00456E5A  |.  0BC2          or eax,edx
00456E5C  |.  C1E0 03       shl eax,3                               ; 到这里算出值 EAX=37980140
00456E5F  |>  8B4C24 08     mov ecx,dword ptr ss:[esp+8]           ; 常数 14
00456E63  |.  8B5424 04     mov edx,dword ptr ss:[esp+4]
00456E67  |.  56            push esi                                ; 注册码值
00456E68  |.  51            push ecx
00456E69  |.  52            push edx
00456E6A  |.  50            push eax                                ; 上面是四个参数,EAX 即可上面算出的值
00456E6B  |.  E8 90FEFFFF   call ashnap.00456D00                    ; 得出另外一个运算函数
00456E70  |.  8BC8          mov ecx,eax                             ; 运算结果放入 ECX 保存，EAX 继续运算
00456E72  |.  83C4 0C       add esp,0C
00456E75  |.  C1E8 09       shr eax,9                               ; 又是一串运算，位移、与、或、异或等等
00456E78  |.  8BD1          mov edx,ecx                             ; 但是不用怕,这种平铺直叙的运算我们可懒得去逆向
00456E7A  |.  25 00F87F00   and eax,7FF800                          ; 直接复制出来,Delphi 嵌入汇编搞定
00456E7F  |.  81E2 80070000 and edx,780                             ; 不过说实话,这些运算基本上在高级语言中都可以直接还原的
......
00456EA5  |.  33C2          xor eax,edx
00456EA7  |.  5E            pop esi
00456EA8  |.  33C1          xor eax,ecx                             ; 到这里，终于算完了，幸亏运算很简单
00456EAA  \.  C3            retn
```

## 关键校验点：第八、九位验证

继续单步跟踪，发现一处关键的校验逻辑，用于验证注册码的第八和第九位。

```assembly
......
00456AEA  |.  885E 01       mov byte ptr ds:[esi+1],bl
00456AED  |>  8B7424 68     mov esi,dword ptr ss:[esp+68]
00456AF1  |.  3BF3          cmp esi,ebx
00456AF3  |.  74 0F         je short ashnap.00456B04
00456AF5  |.  8D4424 10     lea eax,dword ptr ss:[esp+10]           ; 一定要注意，这里是最最最关键的地方
00456AF9  |.  50            push eax                                ; 这是一处关键校验,验证注册码的第八、九位
00456AFA  |.  E8 20F1FFFF   call ashnap.00455C1F                    ; 我因为刚开始忽略了这里，走了很多弯路
00456AFF  |.  83C4 04       add esp,4                               ; 到底关键在哪里,等会单步下去你就知道了
00456B02  |.  8906          mov dword ptr ds:[esi],eax
```

**追入 `00456AFA` 处的 `call` 指令：**

该子函数负责解析并校验注册码的第八、九位。

```assembly
......
00455BDC  |>  0FB60E        movzx ecx,byte ptr ds:[esi]             ; 注册码第八位放入
00455BDF  |.  46            inc esi
00455BE0  |.  83F9 2D       cmp ecx,2D                              ; 比较是不是“-”
00455BE3  |.  8BD1          mov edx,ecx
00455BE5  |.  74 05         je short ashnap.00455BEC
00455BE7  |.  83F9 2B       cmp ecx,2B                              ; 比较是不是“+”
00455BEA  |.  75 04         jnz short ashnap.00455BF0
00455BEC  |>  0FB60E        movzx ecx,byte ptr ds:[esi]
00455BEF  |.  46            inc esi
00455BF0  |>  33C0          xor eax,eax
00455BF2  |>  83F9 30       /cmp ecx,30                             ; 比较是不是数字
00455BF5  |.  7C 0A         |jl short ashnap.00455C01
00455BF7  |.  83F9 39       |cmp ecx,39
00455BFA  |.  7F 05         |jg short ashnap.00455C01
00455BFC  |.  83E9 30       |sub ecx,30
00455BFF  |.  EB 03         |jmp short ashnap.00455C04
00455C01  |>  83C9 FF       |or ecx,FFFFFFFF                       ; 如果不是数字就直接跳到这里，就错误了，不信你可以试试
00455C04  |>  83F9 FF       |cmp ecx,-1
00455C07  |.  74 0C         |je short ashnap.00455C15             
00455C09  |.  8D0480        |lea eax,dword ptr ds:[eax+eax*4]       ; [eax+eax*4]其实就是[5*eax]
00455C0C  |.  8D0441        |lea eax,dword ptr ds:[ecx+eax*2]       ; [ecx+eax*2]放入 EAX
00455C0F  |.  0FB60E        |movzx ecx,byte ptr ds:[esi]            ; 这两步是关键运算，所得出的 EAX 值会作为最终校验的关键
00455C12  |.  46            |inc esi                                ; 提前透露下,根据注册码第八九位运算出的 EAX 值应该等于 4Dh
00455C13  |.^ EB DD         \jmp short ashnap.00455BF2
00455C15  |>  83FA 2D       cmp edx,2D                              ; 比较第八位是不是“-”
00455C18  |.  5F            pop edi
00455C19  |.  5E            pop esi
00455C1A  |.  75 02         jnz short ashnap.00455C1E               ; 对应上面的 cmp 指令
00455C1C  |.  F7D8          neg eax
00455C1E  |>  C3            retn
```

## 前四位与第五位校验

单步走出上述两个 `call` 后，回到主逻辑，此处校验注册码的前四位及第五位。

```assembly
004030CC  |.  8845 00       mov byte ptr ss:[ebp],al                ; 把返回值保存备用
004030CF  |.  BE 241F4800   mov esi,ashnap.00481F24                 ; 固定字符串"AMAS"
004030D4  |.  8D4424 08     lea eax,dword ptr ss:[esp+8]            ; 这里是注册码的前四位
004030D8  |.  53            push ebx
004030D9  |.  8DA424 00000000 lea esp,dword ptr ss:[esp]
004030E0  |>  8A10          /mov dl,byte ptr ds:[eax]               ; 这个循环在做比较
004030E2  |.  8A1E          |mov bl,byte ptr ds:[esi]               ; 说明注册码前四位应该是"AMAS"
004030E4  |.  8ACA          |mov cl,dl
......
004030FD  |.  83C6 02       |add esi,2
00403100  |.  84C9          |test cl,cl
00403102  |.^ 75 DC         \jnz short ashnap.004030E0
00403104  |>  33C0          xor eax,eax
00403106  |.  EB 05         jmp short ashnap.0040310D
00403108  |>  1BC0          sbb eax,eax
0040310A  |.  83D8 FF       sbb eax,-1
0040310D  |>  85C0          test eax,eax
0040310F  |.  5B            pop ebx
00403110  |.  75 16         jnz short ashnap.00403128
00403112  |.  57            push edi
00403113  |.  BF 94204800   mov edi,ashnap.00482094                 ; 这里是一个固定字符"A"
00403118  |.  8D7424 50     lea esi,dword ptr ss:[esp+50]
0040311C  |.  B9 02000000   mov ecx,2
00403121  |.  33C0          xor eax,eax
00403123  |.  F3:A6         repe cmps byte ptr es:[edi],byte ptr ds:[esi] ; 这里是注册码的另一处校验，注册码第五位必须是“A”
00403125  |.  5F            pop edi
00403126  |.  74 04         je short ashnap.0040312C                ; 这里必须跳，否则错误
00403128  |>  C645 00 00    mov byte ptr ss:[ebp],0
0040312C  |>  5E            pop esi
```



### 汇编代码分析

```
00445622   .  8A4424 14       mov al,byte ptr ss:\[esp+14\]
00445626   .  84C0           test al,al
00445628   .  8B7C24 20      mov edi,dword ptr ss:\[esp+20\]
0044562C   .  8A5C24 1C      mov bl,byte ptr ss:\[esp+1C\]
00445630   .  C68424 340A0000>mov byte ptr ss:\[esp+A34\],1
00445638   .  0F84 13020000  je ashsnap.00445851                    ;  关键
0044563E   .  8B7424 18      mov esi,dword ptr ss:\[esp+18\]
00445642   .  83FE 0A        cmp esi,0A
00445645   .  0F85 3C010000 jnz ashsnap.00445787                    ;  跳过去,就会验证最后一处
0044564B   .  84DB          test bl,bl
0044564D   .  74 08        je short ashsnap.00445657
......
0044566D   .  8D4C24 24      lea ecx,dword ptr ss:\[esp+24\]
00445671   .  51           push ecx                                    ; /String2
00445672   .  68 4C774E00  push ashsnap.004E774C                       ; |String1 = ashsnap.004E774C
00445677   .  FF15 DC124800 call dword ptr ds:\[<&KERNEL32.lstrcpyW>\]  ; \lstrcpyW
0044567D   .  68 18944800  push ashsnap.00489418                       ;  提示试用期延长，即符合条件的注册码即认为是官方试用码
......
00445787   > \83FE 4D        cmp esi,4D                                ;  这里就是在检验 00455C09 处得出的值，呵呵
0044578A   .  0F85 C1000000  jnz ashsnap.00445851                    ;  关键跳
00445790   .  8D4C24 24      lea ecx,dword ptr ss:\[esp+24\]
00445794   .  51           push ecx                                    ; /String2
00445795   .  68 4C774E00  push ashsnap.004E774C                       ; |String1 = ashsnap.004E774C
0044579A   .  FF15 DC124800 call dword ptr ds:\[<&KERNEL32.lstrcpyW>\]  ; \lstrcpyW
004457A0   .  68 78934800  push ashsnap.00489378                       ;  "Your key is valid. Thank you very much for buying the software. :-)"
004457A5   .  8D9424 30020000 lea edx,dword ptr ss:\[esp+230\]
```

在 `00445787` 处一个 `cmp` 指令莫名其妙的在比较一处值，我走了好多弯路就是找不到程序到底在比较什么，最后尝试数次内存断点后终于明白了这个值是从哪来的。

到这里，算法基本完了，剩下的工作肯定是总结算法，有精力就搞个注册机。

大致总结如下：

1. 注册码必须 `20` 位。
2. 第七位和第十四位必须是 `"`(连字符)。
3. 程序把注册码分成九份进行验证（忽略 `"` 连字符），大致分割为：

   注册码：`AMASAo-77nhEB-96HD2N`
   
   分割后：`AMAS A o 77 nh EB 96H D2 N`
   
   设为：`K1 K2 K3 K4 K5 K6 K7 K8 9`
   
   对应关系：`K1K2` 是 `“AMASA”`；`K6K8` 处的字符是 `00456A72` 处的函数（如果懒得去逆向，可以直接复制汇编代码从而 `Delphi` 或者 `C++` 嵌入汇编）计算出的，其值取决于 `K3K5K7K9K4K1`；`K4` 处的两个字符经过 `00455C09` 处的计算结果应该为 `4Dh`。

一组可用注册码 `AMASAo-77nhEB-96HD2N`，有兴趣可以试一试。

正确注册后，软件将注册码明码保存在以下位置（方便大家多次调试）：

代码:

```
Windows Registry Editor Version 5.00

\[HKEY\_CURRENT\_USER\\Software\\Ashampoo\\Ashampoo Magical Snap 2\]
"RegistrationKey"="AMASAo-77nhEB-96HD2N"

\[HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Ashampoo\\Ashampoo Magical Snap 2\]
"RegistrationKey"="AMASAo-77nhEB-96HD2N"
```

规避风险，着重探讨算法，少发注册机 (*^__^*) ……
