
### 汇编代码分析与校验绕过

```
00486819    push    0063F7C9              ;ASCII "readme.txt"
0048681E    jmp     short 00486846        ;后面生成绝对路径的代码就免了吧^\_^，直接跳走就成相对路径了。
```

至此已经解决了三处，还有没有呢？用 `F8` 向下跑：

```
0048684B  .  83C4 08         |add     esp, 8
0048684E  .  85C0            |test    eax, eax         ;那么 EAX 就是文件指针了
00486850  .  0F85 88000000  |jnz     004868DE           ;打开文件是否成功，成功就去校验文件的真假？？
```

```
00486856  |> 8A4424 10       |mov     al, byte ptr [esp+10]   ;准备检查下一个驱动器
0048685A  .  FEC0            |inc     al
0048685C  .  3C 5A           |cmp     al, 5A             ;"Z"=0x5A
0048685E  .  884424 10       |mov     byte ptr [esp+10], al
00486862  .^ 0F8E CEFDFFFF   \jle     00486636           ;AL 如果小于 Z,检查下一个驱动器
```

校验失败现在不属于我，我不管了。

```
00486868  .  8B83 C40A0000  mov     eax, dword ptr [ebx+AC4]  ;开始校验失败后的处理工作
0048686E  .  85C0            test    eax, eax
00486870  .  0F85 05010000  jnz     0048697B
00486876  .  8A4424 14       mov     al, byte ptr [esp+14]
0048687A  .  84C0            test    al, al
0048687C  .  0F84 D0000000  je      00486952
00486882  .  8D7C24 14       lea     edi, dword ptr [esp+14]
00486886  .  83C9 FF         or      ecx, FFFFFFFF
00486889  .  33C0            xor     eax, eax
0048688B  .  8D93 C4090000  lea     edx, dword ptr [ebx+9C4]
00486891  .  F2:AE           repne   scas byte ptr es:[edi]
00486893  .  F7D1            not     ecx
00486895  .  2BF9            sub     edi, ecx
00486897  .  8BC1            mov     eax, ecx
00486899  .  8BF7            mov     esi, edi
0048689B  .  8BFA            mov     edi, edx
0048689D  .  C1E9 02        shr     ecx, 2
004868A0  .  F3:A5           rep     movs dword ptr es:[edi], dword ptr [esi]
004868A2  .  8BC8            mov     ecx, eax
004868A4  .  33C0            xor     eax, eax
004868A6  .  83E1 03        and     ecx, 3
004868A9  .  F3:A4           rep     movs byte ptr es:[edi], byte ptr [esi]
004868AB  .  BF D4F76300     mov     edi, 0063F7D4
004868B0  .  83C9 FF         or      ecx, FFFFFFFF
004868B3  .  F2:AE           repne   scas byte ptr es:[edi]
004868B5  .  F7D1            not     ecx
004868B7  .  2BF9            sub     edi, ecx
004868B9  .  8BF7            mov     esi, edi
004868BB  .  8BD9            mov     ebx, ecx
004868BD  .  8BFA            mov     edi, edx
004868BF  .  83C9 FF         or      ecx, FFFFFFFF
004868C2  .  F2:AE           repne   scas byte ptr es:[edi]
004868C4  .  8BCB            mov     ecx, ebx
004868C6  .  4F              dec     edi
004868C7  .  C1E9 02        shr     ecx, 2
004868CA  .  F3:A5           rep     movs dword ptr es:[edi], dword ptr [esi]
004868CC  .  8BCB            mov     ecx, ebx
004868CE  .  83E1 03        and     ecx, 3
004868D1  .  F3:A4           rep     movs byte ptr es:[edi], byte ptr [esi]
004868D3  .  5F              pop     edi
004868D4  .  5E              pop     esi
004868D5  .  5D              pop     ebp
004868D6  .  5B              pop     ebx
004868D7  .  81C4 10020000  add     esp, 210
004868DD  .  C3              retn                 ;无光碟则失败，返回。
```

```
004868DE  |> 50              push    eax            ;文件指针入栈
004868DF  .  E8 A28D1600     call    005EF686       ;此 call 是干什么的已经不重要了，我有那个"readme.txt"文件了啊
004868E4  .  8B83 C40A0000  mov     eax, dword ptr [ebx+AC4]  ;取出“校验是否为真”的标志？？
004868EA  .  83C4 04         add     esp, 4
004868ED  .  85C0            test    eax, eax               ;检查标志
004868EF  .  75 51           jnz     short 00486942          ;跳走了，去光明大道^\_^
```

只要有 `readme.txt` 文件，内容不限，那怕它是空的，都会跳走，不会执行到下面的代码。

```
004868F1  .  8D7C24 10       lea     edi, dword ptr [esp+10]    ;好象在清除什么内容？？
004868F5  .  83C9 FF         or      ecx, FFFFFFFF
004868F8  .  33C0            xor     eax, eax
004868FA  .  8D93 C4090000  lea     edx, dword ptr [ebx+9C4]
00486900  .  F2:AE           repne   scas byte ptr es:[edi]
00486902  .  F7D1            not     ecx
00486904  .  2BF9            sub     edi, ecx
00486906  .  8BC1            mov     eax, ecx
00486908  .  8BF7            mov     esi, edi
0048690A  .  8BFA            mov     edi, edx
0048690C  .  C1E9 02        shr     ecx, 2
0048690F  .  F3:A5           rep     movs dword ptr es:[edi], dword ptr [esi]
00486911  .  8BC8            mov     ecx, eax
00486913  .  33C0            xor     eax, eax
00486915  .  83E1 03        and     ecx, 3
00486918  .  F3:A4           rep     movs byte ptr es:[edi], byte ptr [esi]
0048691A  .  BF D4F76300     mov     edi, 0063F7D4
0048691F  .  83C9 FF         or      ecx, FFFFFFFF
00486922  .  F2:AE           repne   scas byte ptr es:[edi]
00486924  .  F7D1            not     ecx
00486926  .  2BF9            sub     edi, ecx
00486928  .  8BF7            mov     esi, edi
0048692A  .  8BD9            mov     ebx, ecx
0048692C  .  8BFA            mov     edi, edx
0048692E  .  83C9 FF         or      ecx, FFFFFFFF
00486931  .  F2:AE           repne   scas byte ptr es:[edi]
00486933  .  8BCB            mov     ecx, ebx
00486935  .  4F              dec     edi
00486936  .  C1E9 02        shr     ecx, 2
00486939  .  F3:A5           rep     movs dword ptr es:[edi], dword ptr [esi]
0048693B  .  8BCB            mov     ecx, ebx
0048693D  .  83E1 03        and     ecx, 3
00486940  .  F3:A4           rep     movs byte ptr es:[edi], byte ptr [esi]
```

```
00486942  |> 5F              pop     edi        ;准备退出，通向光明的大门^\_^，直接从前面跳到这里岂不快哉？
00486943  .  5E              pop     esi
00486944  .  5D              pop     ebp
00486945  .  B8 01000000     mov     eax, 1     ;让 EAX=1，返回结果为　true
0048694A  .  5B              pop     ebx
0048694B  .  81C4 10020000  add     esp, 210
00486951  .  C3              retn               ;找到光盘的退出点
```

```
00486952  |> 8DBB C4020000  lea     edi, dword ptr [ebx+2C4]
00486958  .  83C9 FF         or      ecx, FFFFFFFF
0048695B  .  33C0            xor     eax, eax
0048695D  .  8D93 C4090000  lea     edx, dword ptr [ebx+9C4]
00486963  .  F2:AE           repne   scas byte ptr es:[edi]
00486965  .  F7D1            not     ecx
00486967  .  2BF9            sub     edi, ecx
00486969  .  8BC1            mov     eax, ecx
0048696B  .  8BF7            mov     esi, edi
0048696D  .  8BFA            mov     edi, edx
0048696F  .  C1E9 02        shr     ecx, 2
00486972  .  F3:A5           rep     movs dword ptr es:[edi], dword ptr [esi]
00486974  .  8BC8            mov     ecx, eax
00486976  .  83E1 03        and     ecx, 3
00486979  .  F3:A4           rep     movs byte ptr es:[edi], byte ptr [esi]
```

```
0048697B  |> 5F              pop     edi        ;准备退出
0048697C  .  5E              pop     esi
0048697D  .  5D              pop     ebp
0048697E  .  33C0            xor     eax, eax     ;让 EAX=0，返回结果为　false
00486980  .  5B              pop     ebx
00486981  .  81C4 10020000  add     esp, 210
00486987  \.  C3              retn                 ;未找到光盘的退出点
```

就这三处校验，涉及两个文件，而且不是游戏必需的文件，`readme.txt` 文件为空也可以，所以后来我干脆把：

```
00486644  jnz     00486856
```

改为：

```
00486644  jmp     00486942
```

保存修改，删掉那个 `readme.txt` 也照样可以进入游戏了，除了没有背景音乐外，其他一切正常。

### 二、实现背景音乐功能和作弊键功能

其实，没有背景音乐是因为背景音乐是以 `CD` 音轨的形式记录在光碟上的。未放入光碟，当然就没有背景音乐了。这通常有两种办法可以解决：跟踪、修改程序中播放音乐的相关代码，我晕倒；还有就是自己写个游戏外挂。我请出了 `VC6`，很快稿定了音乐问题，还加了个作弊键。
