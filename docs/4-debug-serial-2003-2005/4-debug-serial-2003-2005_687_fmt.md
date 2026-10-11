
# ADSL 拨号计时器 v5.28 算法分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-388.htm`](https://www.pediy.com/kssd/pediy07/pediy7-388.htm)

**【破文标题】** `ADSL 拨号计时器 v5.28` 算法分析

**【软件名称】** `ADSL 拨号计时器 v5.28`

**【软件地址】** [`www.uncleme.com/showarticle.php?id=85`](http://www.uncleme.com/showarticle.php?id=85)

**【破文作者】** `KiLlL[DFCG][FCG]`

**【破解时间】** `2005-09-16 17:00`

**【破解声明】** 仅为技术交流之用！刚加入 `FCG`，高兴～

**【破解过程】**

看到一篇文章 [`www.it33.com.cn/soft/kftq/200578113841.html`](http://www.it33.com.cn/soft/kftq/200578113841.html)

“具体怎么使用，那要看你自己的意愿了和需要了。反正我是这样做的，我的软件 `ADSL 拨号计时器` 只在很早版本上出过注册机，后来的 `v3.70` 出过破解补丁——其实只是破掉了启动时提示注册的对话框，实质上根本没破解。用了上述的着法以后，到现在的 `v5.28` 版本，再没有过什么破解补丁或注册机。如果现在的 `v5.28` 版本谁能破解，将立即公布程序源码。”

说的很是有趣，于是下载看看，由于程序现在最新版 `7.02` 已经免费，所以看看也无妨了，结果出乎意料。

程序有壳，`aspack`，脱后 `dede` 查看：

```assembly
00489004   /.  55           push ebp                     ;  dede 可知这个是注册按钮的事件
00489005   |.  8BEC         mov ebp,esp
00489007   |.  6A 00        push 0
00489009   |.  6A 00        push 0
0048900B   |.  53           push ebx
0048900C   |.  8BD8         mov ebx,eax
0048900E   |.  33C0         xor eax,eax
00489010   |.  55           push ebp
00489011   |.  68 A5904800  push unpacked.004890A5
00489016   |.  64:FF30      push dword ptr fs:[eax]
00489019   |.  64:8920      mov dword ptr fs:[eax],esp
0048901C   |.  8D55 FC      lea edx,dword ptr ss:[ebp-4]
0048901F   |.  8B83 FC02000>mov eax,dword ptr ds:[ebx+2FC]
00489025   |.  E8 EA92FBFF  call unpacked.00442314        ;  读取假码
0048902A   |.  837D FC 00   cmp dword ptr ss:[ebp-4],0    ;  假码是否为空？
0048902E   |.  74 57        je short unpacked.00489087
00489030   |.  8BC3         mov eax,ebx
00489032   |.  E8 59FEFFFF  call unpacked.00488E90        ;  对假码进行运算
00489037   |.  84C0         test al,al                    ;  关键跳转
00489039   |.  74 36        je short unpacked.00489071    ;  注册失败就跳走
0048903B   |.  A1 100D4900  mov eax,dword ptr ds:[490D10]
00489040   |.  E8 E712F8FF  call unpacked.0040A32C
00489045   |.  D805 B490480>fadd dword ptr ds:[4890B4]
0048904B   |.  83C4 F4      add esp,-0C
```

跟进关键算法看看：

```assembly
00488E90   /$  55           push ebp                     ;  关键算法
00488E91   |.  8BEC         mov ebp,esp
00488E93   |.  B9 05000000  mov ecx,5
00488E98   |>  6A 00        /push 0
00488E9A   |.  6A 00        |push 0
00488E9C   |.  49           |dec ecx
00488E9D   |.^ 75 F9        \jnz short unpacked.00488E98
00488E9F   |.  51           push ecx
00488EA0   |.  53           push ebx
00488EA1   |.  56           push esi
00488EA2   |.  8945 FC      mov dword ptr ss:[ebp-4],eax
00488EA5   |.  33C0         xor eax,eax
00488EA7   |.  55           push ebp
00488EA8   |.  68 E08F4800  push unpacked.00488FE0
00488EAD   |.  64:FF30      push dword ptr fs:[eax]
00488EB0   |.  64:8920      mov dword ptr fs:[eax],esp
00488EB3   |.  33F6         xor esi,esi
00488EB5   |.  8D55 F8      lea edx,dword ptr ss:[ebp-8]
00488EB8   |.  8B45 FC      mov eax,dword ptr ss:[ebp-4]
00488EBB   |.  E8 D0FAFFFF  call unpacked.00488990
00488EC0   |.  8B45 F8      mov eax,dword ptr ss:[ebp-8]   ;  硬件号 "7FD9534C"
00488EC3   |.  E8 5CBAF7FF  call unpacked.00404924         ;  获得长度
00488EC8   |.  8BD8         mov ebx,eax                   ;  8
00488ECA   |.  85DB         test ebx,ebx
00488ECC   |.  7E 58        jle short unpacked.00488F26
00488ECE   |.  C745 F4 0100>mov dword ptr ss:[ebp-C],1
00488ED5   |>  8D45 F0      /lea eax,dword ptr ss:[ebp-10]  ;  以长度作为循环终止值
00488ED8   |.  50           |push eax
00488ED9   |.  B9 01000000  |mov ecx,1
00488EDE   |.  8B55 F4      |mov edx,dword ptr ss:[ebp-C]
00488EE1   |.  8B45 F8      |mov eax,dword ptr ss:[ebp-8]
00488EE4   |.  E8 93BCF7FF  |call unpacked.00404B7C        ;  mid(code,i,1)
00488EE9   |.  8B45 F0      |mov eax,dword ptr ss:[ebp-10]
00488EEC   |.  E8 2BBCF7FF  |call unpacked.00404B1C
00488EF1   |.  8A00         |mov al,byte ptr ds:[eax]      ;  取得 ascii
00488EF3   |.  25 FF000000  |and eax,0FF
00488EF8   |.  03F0         |add esi,eax                   ;  加到 esi
00488EFA   |.  8D55 E8      |lea edx,dword ptr ss:[ebp-18]
00488EFD   |.  69C6 4132000>|imul eax,esi,3241             ;  eax=esi*3241
00488F03   |.  E8 24FDF7FF  |call unpacked.00408C2C        ;  转成十进制
00488F08   |.  8B45 E8      |mov eax,dword ptr ss:[ebp-18]  ;  7->707575
00488F0B   |.  8D55 EC      |lea edx,dword ptr ss:[ebp-14]
00488F0E   |.  E8 C5260000  |call unpacked.0048B5D8        ;  较复杂的编码
00488F13   |.  8B55 EC      |mov edx,dword ptr ss:[ebp-14]  ;  707575 －》"2060320645206502065020645")
00488F16   |.  B8 100D4900  |mov eax,unpacked.00490D10
00488F1B   |.  E8 A0B7F7FF  |call unpacked.004046C0
00488F20   |.  FF45 F4      |inc dword ptr ss:[ebp-C]
00488F23   |.  4B           |dec ebx
00488F24   |.^ 75 AF        \jnz short unpacked.00488ED5
00488F26   |>  8D55 DC      lea edx,dword ptr ss:[ebp-24]
00488F29   |.  8B45 FC      mov eax,dword ptr ss:[ebp-4]
00488F2C   |.  8B80 FC02000>mov eax,dword ptr ds:[eax+2FC]
00488F32   |.  E8 DD93FBFF  call unpacked.00442314
```

这里面对硬件号进行了处理，分成两个步骤，首先，逐位取出 `ascii`，加到 `esi`，把 `esi*3241` 的结果转成十进制，得到临时码。再对其进行编码。

```vb
For i = 1 To Len(machineID)
    eax = Asc(Mid$(machineID, i, 1)) '取出 ascii
    esi = esi + eax '相加
    eax = esi * &H3241 '乘以 3241
    temp = temp & CStr(encode(eax)) '对结果编码
Next
```

编码过程有点复杂，请看：

```assembly
0048B5D8   /$  55           push ebp                     ;  把数字编码过程
0048B5D9   |.  8BEC         mov ebp,esp
0048B5DB   |.  81C4 DCFEFFF>add esp,-124
0048B5E1   |.  53           push ebx
0048B5E2   |.  56           push esi
0048B5E3   |.  57           push edi
0048B5E4   |.  33C9         xor ecx,ecx
0048B5E6   |.  898D E4FEFFF>mov dword ptr ss:[ebp-11C],ecx
0048B5EC   |.  898D ECFEFFF>mov dword ptr ss:[ebp-114],ecx
0048B5F2   |.  898D E8FEFFF>mov dword ptr ss:[ebp-118],ecx
0048B5F8   |.  898D F0FEFFF>mov dword ptr ss:[ebp-110],ecx
0048B5FE   |.  8BF2         mov esi,edx
0048B600   |.  8945 FC      mov dword ptr ss:[ebp-4],eax
0048B603   |.  8B45 FC      mov eax,dword ptr ss:[ebp-4]   ;  需要编码的数字，以第一个为例： 707575
0048B606   |.  E8 0195F7FF  call unpacked.00404B0C
0048B60B   |.  8DBD F4FEFFF>lea edi,dword ptr ss:[ebp-10C]  ;  707575
0048B611   |.  33C0         xor eax,eax
0048B613   |.  55           push ebp
0048B614   |.  68 5BB74800  push unpacked.0048B75B
0048B619   |.  64:FF30      push dword ptr fs:[eax]
0048B61C   |.  64:8920      mov dword ptr fs:[eax],esp
0048B61F   |.  8BC7         mov eax,edi
0048B621   |.  8B55 FC      mov edx,dword ptr ss:[ebp-4]
0048B624   |.  B9 FF000000  mov ecx,0FF                   ;  707575
0048B629   |.  E8 D292F7FF  call unpacked.00404900        ;  取得位数
0048B62E   |.  8A07         mov al,byte ptr ds:[edi]       ;  6
0048B630   |.  33D2         xor edx,edx                   ;  707575
0048B632   |.  8AD0         mov dl,al
0048B634   |.  42           inc edx
0048B635   |.  8955 F8      mov dword ptr ss:[ebp-8],edx
0048B638   |.  8B55 F8      mov edx,dword ptr ss:[ebp-8]
0048B63B   |.  81E2 0100008>and edx,80000001
0048B641   |.  79 05        jns short unpacked.0048B648
0048B643   |.  4A           dec edx
0048B644   |.  83CA FE      or edx,FFFFFFFE
0048B647   |.  42           inc edx
0048B648   |>  85D2         test edx,edx
0048B64A   |.  74 03        je short unpacked.0048B64F
0048B64C   |.  FF45 F8      inc dword ptr ss:[ebp-8]
0048B64F   |>  837D F8 0A   cmp dword ptr ss:[ebp-8],0A    ;  跟 10 比较
0048B653   |.  7D 07        jge short unpacked.0048B65C
0048B655   |.  C745 F8 0A00>mov dword ptr ss:[ebp-8],0A
0048B65C   |>  33DB         xor ebx,ebx
0048B65E   |.  8AD8         mov bl,al
```



# 反汇编代码分析

```
0048B660   |.  85DB         test ebx,ebx  
0048B662   |.  75 67        jnz short unpacked.0048B6CB  
0048B664   |.  E8 9BB6F7FF  call <jmp.&kernel32.GetTickCoun>; \[GetTickCount
0048B669   |.  33D2         xor edx,edx  
0048B66B   |.  52           push edx                    ; /Arg2 => 00000000  
0048B66C   |.  50           push eax                    ; |Arg1  
0048B66D   |.  8D85 F0FEFFF>lea eax,dword ptr ss:\[ebp\-110\]  ; |  
0048B673   |.  E8 E4D5F7FF  call unpacked.00408C5C           ; \\unpacked.00408C5C  
0048B678   |.  8B95 F0FEFFF>mov edx,dword ptr ss:\[ebp\-110\]  
0048B67E   |.  8BC7         mov eax,edi  
0048B680   |.  B9 FF000000  mov ecx,0FF  
0048B685   |.  E8 7692F7FF  call unpacked.00404900  
0048B68A   |.  EB 44        jmp short unpacked.0048B6D0  
0048B68C   |>   8D85 ECFEFFF>/lea eax,dword ptr ss:\[ebp\-114\]  
0048B692   |.  8BD7         |mov edx,edi                ;  707575  
0048B694   |.  E8 2F92F7FF  |call unpacked.004048C8  
0048B699   |.  8D85 ECFEFFF>|lea eax,dword ptr ss:\[ebp\-114\]  
0048B69F   |.  50           |push eax  
0048B6A0   |.  8D85 E8FEFFF>|lea eax,dword ptr ss:\[ebp\-118\]  
0048B6A6   |.  8BD7         |mov edx,edi                ;  707575  
0048B6A8   |.  E8 1B92F7FF  |call unpacked.004048C8  
0048B6AD   |.  8B95 E8FEFFF>|mov edx,dword ptr ss:\[ebp\-118\]  
0048B6B3   |.  58           |pop eax  
0048B6B4   |.  E8 7392F7FF  |call unpacked.0040492C  
0048B6B9   |.  8B95 ECFEFFF>|mov edx,dword ptr ss:\[ebp\-114\] ;  扩展成"707575707575"  
0048B6BF   |.  8BC7         |mov eax,edi  
0048B6C1   |.  B9 FF000000  |mov ecx,0FF  
0048B6C6   |.  E8 3592F7FF  |call unpacked.00404900  
0048B6CB   |>   803F 0A      cmp byte ptr ds:\[edi\],0A  
0048B6CE   |.^  72 BC       \\jb short unpacked.0048B68C  
0048B6D0   |>   8BC7         mov eax,edi  
0048B6D2   |.  8BD3         mov edx,ebx  
0048B6D4   |.  E8 0B74F7FF  call unpacked.00402AE4  
0048B6D9   |.  8BC6         mov eax,esi  
0048B6DB   |.  E8 8C8FF7FF  call unpacked.0040466C  
0048B6E0   |.  8B5D F8      mov ebx,dword ptr ss:\[ebp\-8\]  
0048B6E3   |.  D1FB         sar ebx,1  
0048B6E5   |.  79 03        jns short unpacked.0048B6EA  
0048B6E7   |.  83D3 00      adc ebx,0  
0048B6EA   |>   85DB         test ebx,ebx  
0048B6EC   |.  7E 47        jle short unpacked.0048B735     ;  edi 存放的就是位数+扩展后的字符串  ascii 码为 0637303735...  
0048B6EE   |>   0FB707       /movzx eax,word ptr ds:\[edi\]    ;  取 ascii，第一个为 3706  
0048B6F1   |.  05 75190000  |add eax,1975                 ;  +1975  
0048B6F6   |.  8945 F4      |mov dword ptr ss:\[ebp\-C\],eax   ;  507b  
0048B6F9   |.  83C7 02      |add edi,2  
0048B6FC   |.  8D85 E4FEFFF>|lea eax,dword ptr ss:\[ebp\-11C\]  
0048B702   |.  50           |push eax                   ; /Arg1  
0048B703   |.  8B45 F4      |mov eax,dword ptr ss:\[ebp\-C\]   ; |  
0048B706   |.  8985 DCFEFFF>|mov dword ptr ss:\[ebp\-124\],eax ; |  
0048B70C   |.  C685 E0FEFFF>|mov byte ptr ss:\[ebp\-120\],0    ; |  
0048B713   |.  8D95 DCFEFFF>|lea edx,dword ptr ss:\[ebp\-124\] ; |  
0048B719   |.  33C9         |xor ecx,ecx                ; |  
0048B71B   |.  B8 74B74800  |mov eax,unpacked.0048B774     ; |ASCII "%5.5d"  
0048B720   |.  E8 5BE1F7FF  |call unpacked.00409880        ; \\unpacked.00409880  
0048B725   |.  8B95 E4FEFFF>|mov edx,dword ptr ss:\[ebp\-11C\] ;  转成 10 进制： "20603"  
0048B72B   |.  8BC6         |mov eax,esi  
0048B72D   |.  E8 FA91F7FF  |call unpacked.0040492C  
0048B732   |.  4B           |dec ebx  
0048B733   |.^  75 B9       \\jnz short unpacked.0048B6EE    ;  取出前 5 位  
0048B735   |>   33C0         xor eax,eax  
0048B737   |.  5A           pop edx  
0048B738   |.  59           pop ecx  
```

## 编码逻辑分析

编码的方法是先扩展字符串，然后在前面加上位数，比如刚才的硬件第一位 7 临时码是 `707575`，扩展成为 “707575707575”，在最前面加上长度 6，成了 `chr(6) & "707575707575"`，然后根据 `ascii` 码每四位加上 1975 后转换成 10 进制数字。

```
Function encode(eax)  
    result = Chr$(Len(eax)) & eax & eax '扩展  
    For i = 1 To 10 Step 2  
        eax = Hex$(Asc(Mid$(result, i + 1, 1))) & Right$("0" & Hex$(Asc(Mid$(result, i, 1))), 2)  '取出两位的 ascii  
        eax = CInt("&H" & eax) + &H1975 '转换成数字并加 1975  
        encode = encode & eax  
    Next  
End Function  
```

最终，`707575` 转换成为：`2060320645206502065020645`。现在回到验证注册码程序。作者这里虽然每位都编码，但是最后只用到了机器码最后一位的编码，也就是 `"C"`->`"2034820901191142013718859"`

当然，这个编码跟前面几位也是有关系的，用到了他们的 `ascii` 和。

本来到这里就不错了，可惜作者下面的代码让你笑：

## 验证逻辑反汇编

```
00488F37   |.  8B45 DC      mov eax,dword ptr ss:\[ebp\-24\]   ;  假码  
00488F3A   |.  E8 ED13F8FF  call unpacked.0040A32C  
00488F3F   |.  D825 F08F480>fsub dword ptr ds:\[488FF0\]       ;  -1728  
00488F45   |.  83C4 F4      add esp,-0C  
00488F48   |.  DB3C24       fstp tbyte ptr ss:\[esp\]          ; |  
00488F4B   |.  9B           wait                     ; |  
00488F4C   |.  8D45 E0      lea eax,dword ptr ss:\[ebp\-20\]   ; |  
00488F4F   |.  E8 5013F8FF  call unpacked.0040A2A4        ; \\unpacked.0040A2A4  
00488F54   |.  8B45 E0      mov eax,dword ptr ss:\[ebp\-20\]   ;   "12343950"  
00488F57   |.  8D55 E4      lea edx,dword ptr ss:\[ebp\-1C\]  
00488F5A   |.  E8 79260000  call unpacked.0048B5D8        ;  较复杂的编码  
00488F5F   |.  8B45 E4      mov eax,dword ptr ss:\[ebp\-1C\]   ;  "1906919623196252014219109"  
00488F62   |.  8B15 100D490>mov edx,dword ptr ds:\[490D10\]   ;  最后一次的编码结果  
00488F68   |.  E8 FBBAAF7FF  call unpacked.00404A68        ;  比较是否一样  
00488F6D   |.  75 37        jnz short unpacked.00488FA6     ;  不一样就跳走，注册失败  
```

作者直接拿假码减去 1728，然后编码，然后比较两次编码是否一致，这个也太简单了吧。上面的编码白跟踪了 。最终的算法是：

`注册码＝机器码-1728`

++++++++++++++++++++++++++++++++++

这里我分析错了，由于作者没有对输入的数字进行转换，所以你无法输入十六进制数字。所以可能只有对编码求逆了。

但是可以利用下面的漏洞直接写入注册表。就是把那串数字直接写入注册表。

2005-9-17 18:00

++++++++++++++++++++++++++++++++++

真的没有想到亚，更没有想到的是如果在 `00488F6D` 爆破的话，存入注册表的竟然是正确的注册码：

`2.03482090119114E24` （`"2034820901191142013718859"`转成数字）

注册成功后，注册表里增加：

```
Windows Registry Editor Version 5.00  

\[HKEY\_CURRENT\_USER\\Software\\Microsoft\\Internet Explorer\\Netzc\]  

"Card"\="7FD9534C"  

"Code"\="2.03482090119114E24"  
```

删除成了没有注册。

【破解心得】作者估计太自信，可以说这个算法本来还有一点点麻烦，但是在错误思想的指导下要想搞定太容 易了。而作者用到的反破解除了加个弱壳以外，如果注册表中的注册码不正确，程序启动后会自动退出。

+++++++++++++++++++++++++++++++++++++++++

## 最新增加

除了可以直接写入注册表注册以外，终于搞定了逆运算：就是知道编码如何求编码前的字串：

```
Function decode2(eax)  
    Dim temp, tmp1, tmp2  
    For i = 1 To 25 Step 5  
        temp = CLng(Mid$(eax, i, 5)) - &H1975  
        For j = Asc("9") To Asc("0") Step -1  
            tmp1 = j \* 16 \* 16  
            tmp2 = temp - tmp1  
            If i = 1 And tmp2 > 0 And tmp2 < 16 \* 16 Then  
                decode2 = decode2 & Chr$(tmp2) & Chr$(tmp1)  
                Exit For  
            Else  
                If tmp2 >= Asc("0") And tmp2 < Asc("9") Then  
                    decode2 = decode2 & Chr$(tmp2) & Chr$(tmp1)  
                    Exit For  
                End If  
            End If  
        Next  
    Next  

decode2 = clng(Mid$(decode2, 2, Asc(Mid$(decode2, 1, 1))))+ 1728  
End Function  
```

++++++++++++++++++++++++++++++++++++++++

2005-9-18

今天又想了一下，原来我真笨亚，直接取没有编码前的数字就可以了，不需要编码，再反编码。

```
    For i = 1 To Len(machineID)  
        eax = Asc(Mid$(machineID, i, 1)) '取出 ascii  
        esi = esi + eax '相加  
    Next  
    eax = esi \* &H3241 '乘以 3241  
    eax = eax + 1728  
    sn = eax  
```

++++++++++++++++++++++++++++++++++++++++

2005-9-18 17:00

跟踪了下机器码的生成：

```
00488974   /$  53           push ebx  
00488975   |.  57           push edi  
00488976   |.  89C7         mov edi,eax  
00488978   |.  B8 01000000  mov eax,1  
0048897D   |.  0FA2         cpuid  
0048897F   |.  AB           stos dword ptr es:\[edi\] '保存 eax  
00488980   |.  89D8         mov eax,ebx  
00488982   |.  AB           stos dword ptr es:\[edi\] '保存 ebx  
00488983   |.  89C8         mov eax,ecx  
00488985   |.  AB           stos dword ptr es:\[edi\] '保存 ecx  
00488986   |.  89D0         mov eax,edx  
00488988   |.  AB           stos dword ptr es:\[edi\] '保存 edx  
```



```
00488989   |.  89D0         mov eax,edx  

0048898B   |.  AB           stos dword ptr es:[edi] '保存 edx  

0048898C   |.  5F           pop edi  

0048898D   |.  5B           pop ebx  

0048898E   \.  C3           retn  

EAX 00000F27  

ECX 00004400  

EDX BFEBFBFF  

EBX 00010809  

004889AB   |.  E8 C4FFFFFF  call unpacked.00488974  

004889B0   |.  8B0424       mov eax,dword ptr ss:[esp]           ;  eax:f27  

004889B3   |.  83C0 08      add eax,8                             ;  +8  

004889B6   |.  8B5424 04    mov edx,dword ptr ss:[esp+4]          ;  ebx 10809  

004889BA   |.  83C2 07      add edx,7                             ;  +7  

004889BD   |.  03C2         add eax,edx                            ;  f27+10810  

004889BF   |.  8B5424 08    mov edx,dword ptr ss:[esp+8]          ;  ecx 4400  

004889C3   |.  83C2 06      add edx,6                             ;  4400+6  

004889C6   |.  03C2         add eax,edx                            ;  1173f+4406  

004889C8   |.  8B5424 0C    mov edx,dword ptr ss:[esp+C]          ;  edx  BFEBFBFF  

004889CC   |.  83C2 05      add edx,5                             ;  +5  

004889CF   |.  03C2         add eax,edx                            ;  BFEBFC04 + 15B45  

004889D1   |.  8B5424 10    mov edx,dword ptr ss:[esp+10]         ;  EDX BFEBFBFF  

004889D5   |.  83C2 04      add edx,4                             ;  +4  

004889D8   |.  03C2         add eax,edx                            ;  BFEBFC03+ BFED5749  

004889DA   |.  8BCB         mov ecx,ebx                            ;  12fbc4  
```

最后结果："7FD9534C"

`4dd466d0e1e0cc123afbe504da2462ca`
