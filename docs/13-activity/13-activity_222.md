

# 看雪论坛读书月第一题 答案提交 by lelfei

> 原文：[`www.pediy.com/kssd/pediy10/68255.html`](https://www.pediy.com/kssd/pediy10/68255.html)

拿到 cm.exe 后要沐浴、斋戒、祭天、做法、看风水、祭出 PEiD 观察之类的的活动就略过不说了~~~~  
  
直接 OD 载入，看入口处代码，有 msvcrt.\_\_set\_app\_type 之类的 API，看来是 MFC 程序了，直接查找当前模块中的名称，找名称 MFC42.#3092\_CWnd::GetDlgItem，搜到 2 个，就是这二个函数获取 User name 和 Key 了。下断，输入试炼码：lelfei 14141414 后确定，断下了！  

代码:

```
00401CD3  push    3E9
00401CD8  mov     ecx, dword ptr \[ebp-4\]
00401CDB  call    <jmp.&MFC42.#3092\_CWnd::GetDlgItem>
00401CE0  mov     ecx, eax
00401CE2  call    <jmp.&MFC42.#3874\_CWnd::GetWindowTextA>    ;  SN
00401CE7  mov     ecx, dword ptr \[ebp-4\]
00401CEA  add     ecx, 60
00401CED  call    00401E20
00401CF2  push    eax
00401CF3  mov     ecx, dword ptr \[ebp-4\]
00401CF6  add     ecx, 60
00401CF9  call    <jmp.&MFC42.#2915\_CString::GetBuffer>
00401CFE  mov     dword ptr \[ebp-8\], eax
00401D01  mov     ecx, dword ptr \[ebp-4\]
00401D04  add     ecx, 64
00401D07  push    ecx
00401D08  push    3EA
00401D0D  mov     ecx, dword ptr \[ebp-4\]
00401D10  call    <jmp.&MFC42.#3092\_CWnd::GetDlgItem>
00401D15  mov     ecx, eax
00401D17  call    <jmp.&MFC42.#3874\_CWnd::GetWindowTextA>    ;  UN

00401D39  push    edx                                        ; /s
00401D3A  call    <jmp.&MSVCRT.strlen>                       ; \\len(UN)
00401D3F  add     esp, 4
00401D42  test    eax, eax
00401D44  jnz     short 00401D5C                             ;  len(UN)=0?
00401D46  push    0
00401D48  push    0
00401D4A  push    004035D0
00401D4F  mov     ecx, dword ptr \[ebp-4\]
00401D52  call    <jmp.&MFC42.#4224\_CWnd::MessageBoxA>
00401D57  jmp     00401E0F
00401D5C  mov     eax, dword ptr \[ebp-8\]
00401D5F  push    eax                                        ; /s
00401D60  call    <jmp.&MSVCRT.strlen>                       ; \\len(SN)
00401D65  add     esp, 4
00401D68  test    eax, eax
00401D6A  jnz     short 00401D82                             ;  len(SN)=0?
00401D6C  push    0
00401D6E  push    0
00401D70  push    004035C0
00401D75  mov     ecx, dword ptr \[ebp-4\]
00401D78  call    <jmp.&MFC42.#4224\_CWnd::MessageBoxA>
00401D7D  jmp     00401E0F
00401D82  mov     dword ptr \[ebp-10\], 0                      ;  i=0
00401D89  /mov     ecx, dword ptr \[ebp-C\]                    ;  UN
00401D8C  |add     ecx, dword ptr \[ebp-10\]
00401D8F  |movsx   edx, byte ptr \[ecx\]                       ;  UN(i)
00401D92  |test    edx, edx                                  ;  UN(i)=0?
00401D94  |je      short 00401DBD                            ;  跳出
00401D96  |mov     eax, dword ptr \[ebp-C\]                    ;  UN
00401D99  |add     eax, dword ptr \[ebp-10\]
00401D9C  |movsx   ecx, byte ptr \[eax\]                       ;  UN(i)
00401D9F  |cmp     ecx, 61                                   ;  "a"
00401DA2  |jle     short 00401DBD                            ;  UN(i)>"a"
00401DA4  |mov     edx, dword ptr \[ebp-C\]                    ;  UN
00401DA7  |add     edx, dword ptr \[ebp-10\]
00401DAA  |movsx   eax, byte ptr \[edx\]                       ;  UN(i)
00401DAD  |cmp     eax, 7A                                   ;  "z"
00401DB0  |jge     short 00401DBD                            ;  UN(i)<"z"
00401DB2  |mov     ecx, dword ptr \[ebp-10\]
00401DB5  |add     ecx, 1
00401DB8  |mov     dword ptr \[ebp-10\], ecx
00401DBB  \\jmp     short 00401D89
00401DBD  cmp     dword ptr \[ebp-10\], 6                      ;  用户名为 6 位小写字母

00401DFB  mov     ecx, dword ptr \[ebp-8\]                     ;  SN
00401DFE  push    ecx
00401DFF  mov     edx, dword ptr \[ebp-C\]                     ;  UN
00401E02  push    edx
00401E03  mov     eax, dword ptr \[ebp-14\]
00401E06  push    eax
00401E07  call    00401BB0                                   ;  关键，跟入

```

  
用户名必须为 6 位，我的名字刚好是 6 位，继续跟踪：  
  
代码:

```
00401BB0  push    ebp
00401BB1  mov     ebp, esp
00401BB3  push    -1
00401BB5  push    004026F0                                   ;  SE 处理程序安装
00401BBA  mov     eax, dword ptr fs:\[0\]
00401BC0  push    eax
00401BC1  mov     dword ptr fs:\[0\], esp
00401BC8  push    ecx
00401BC9  sub     esp, 50
00401BCC  push    ebx
00401BCD  push    esi
00401BCE  push    edi
00401BCF  mov     dword ptr \[ebp-10\], esp
00401BD2  mov     eax, dword ptr \[ebp+8\]
00401BD5  sub     eax, 10
00401BD8  mov     dword ptr \[ebp-14\], eax
00401BDB  mov     ecx, dword ptr \[ebp-14\]
00401BDE  mov     dword ptr \[ecx\], 004017F0                  ;  设置返回地址入口
00401BE4  mov     dword ptr \[ebp-18\], 0                      ;  i=0
00401BEB  mov     dword ptr \[ebp-1C\], 0                      ;  j=0
……花指令
00401BFE  cmp     dword ptr \[ebp-18\], 6                      ;  i>6?
00401C02  jge     short 00401C71
……花指令
00401C10  mov     edx, dword ptr \[ebp+C\]                     ;  UN
00401C13  add     edx, dword ptr \[ebp-18\]
00401C16  movsx   eax, byte ptr \[edx\]                        ;  UN(i)
00401C19  mov     ecx, dword ptr \[ebp+10\]                    ;  SN
00401C1C  add     ecx, dword ptr \[ebp-1C\]
00401C1F  movsx   edx, byte ptr \[ecx\]                        ;  SN(j)
00401C22  add     edx, 1B                                    ;  SN(j)+1B
00401C25  cmp     eax, edx                                   ;  SN(j)+1B=UN(i)?
00401C27  je      short 00401C2E                             ;  不跳则出错
00401C29  call    004017B0                                   ;  MsgBox("继续努力")
……花指令
00401C3A  mov     eax, dword ptr \[ebp+C\]                     ;  UN
00401C3D  add     eax, dword ptr \[ebp-18\]
00401C40  movsx   ecx, byte ptr \[eax\]                        ;  UN(i)
00401C43  mov     edx, dword ptr \[ebp+10\]                    ;  SN
00401C46  add     edx, dword ptr \[ebp-1C\]
00401C49  movsx   eax, byte ptr \[edx+1\]                      ;  SN(j+1)
00401C4D  add     eax, 20                                    ;  SN(j+1)+20
00401C50  cmp     ecx, eax                                   ;  SN(j+1)+20=UN(i)?
00401C52  jle     short 00401C5D                             ;  
00401C54  mov     ecx, dword ptr \[ebp-14\]
00401C57  mov     dword ptr \[ecx\], 004017B0                  ;  修改返回入口，跳过后面的注册码检测过程
00401C5D  mov     edx, dword ptr \[ebp-1C\]
00401C60  add     edx, 2                                     ;  j=j+2
00401C63  mov     dword ptr \[ebp-1C\], edx
00401C66  mov     eax, dword ptr \[ebp-18\]
00401C69  add     eax, 1                                     ;  i=i+1
00401C6C  mov     dword ptr \[ebp-18\], eax
00401C6F  jmp     short 00401BFE
00401C71  mov     dword ptr \[ebp-4\], 0
……花指令
00401C84  mov     dword ptr \[ebp-20\], 29C
00401C8B  mov     ecx, dword ptr \[ebp-20\]
00401C8E  mov     byte ptr \[ecx\], 6                          ;  内存访问异常
00401C91  jmp     short 00401C9E                             ;  异常后返回这里
00401C93  call    004019F0                                   ;  异常后会检测父进程
00401C98  mov     eax, 00401C9E
00401C9D  retn
00401C9E  mov     dword ptr \[ebp-4\], -1
00401CA5  mov     ecx, dword ptr \[ebp-C\]
00401CA8  mov     dword ptr fs:\[0\], ecx
00401CAF  pop     edi
00401CB0  pop     esi
00401CB1  pop     ebx
00401CB2  mov     esp, ebp
00401CB4  pop     ebp
00401CB5  retn                                               ;  返回到前面设置的入口 4017F0

```

在 00401C93 处的 call 004019F0 会检测父进程过程很清晰，都是用 TerminateProcess 结束父进程或自己退出，只需要把 00401A75，00401B46，00401B8F 三个条件跳转直接改为 JMP 就可以一劳永逸了。  
  
执行到这里可以看到注册码与用户名的关系为：  
Username 每一位 ASC 值分别减 1B 和 20 排列组成 Key  
整理代码为：  
代码:

```
    For i = 1 To Len(UN)
        j = Asc(Mid$(UN, i, 1))
        SN = SN & Chr(j - &H1B) & Chr(j - &H20)
    Next

```

重新输入用户名为 lelfei，key 为 QLJEQLKFJENI 继续跟踪：  
代码:

```
004017F0  push    ebp
004017F1  mov     ebp, esp
004017F3  sub     esp, 80
004017F9  push    ebx
004017FA  push    esi
004017FB  push    edi
004017FC  mov     dword ptr \[ebp-4\], 00403588                ;  STR="ABCDEFGHIJKLMNOPQRSTUVWXY"
00401812  mov     eax, dword ptr \[ebp-4\]           ;  STR
00401815  mov     dword ptr \[ebp-8\], eax           ;  j=0
00401818  mov     dword ptr \[ebp-28\], 0            ;  i=0
0040181F  cmp     dword ptr \[ebp-28\], 18           ;  do until i=0x18
00401823  je      004018BB
……花指令
00401835  movsx   ecx, byte ptr \[40416C\]           ;  UN(0)
0040183C  mov     edx, dword ptr \[ebp-4\]
0040183F  movsx   eax, byte ptr \[edx\]              ;  STR(j)
00401842  add     eax, 20                          ;  STR(j)+20 转换为小写
00401845  cmp     ecx, eax
00401847  je      short 0040186A                   ;  if STR(j)+20<>UN(0) then----
00401849  push    0040416C                         ; /s = "lelfep"
0040184E  call    <jmp.&MSVCRT.strlen>             ; \\len(UN)
00401853  add     esp, 4
00401856  movsx   ecx, byte ptr \[eax+40416B\]       ;  UN(len(UN)-1)=UN(5)最后一位
0040185D  mov     edx, dword ptr \[ebp-4\]
00401860  movsx   eax, byte ptr \[edx\]              ;  STR(j)
00401863  add     eax, 20                          ;  STR(j)+20 转换为小写
00401866  cmp     ecx, eax
00401868  jnz     short 00401873                   ;  if STR(j)+20=UN(5) then----
0040186A  mov     ecx, dword ptr \[ebp-4\]           ;  合并成：if STR(j)+20=UN(1) or STR(j)+20=UN(6) then
0040186D  add     ecx, 1                           ;  j=j+1
00401870  mov     dword ptr \[ebp-4\], ecx           ;  end if----
00401873  mov     edx, dword ptr \[ebp-28\]          ;  i
00401876  mov     eax, dword ptr \[ebp-4\]
00401879  mov     cl, byte ptr \[eax\]               ;  STR(j)
0040187B  mov     byte ptr \[ebp+edx-24\], cl        ;  NEWSTR(i)=STR(j)
0040187F  mov     edx, dword ptr \[ebp-28\]
00401882  add     edx, 1                           ;  i=i+1
00401885  mov     dword ptr \[ebp-28\], edx
00401888  mov     eax, dword ptr \[ebp-4\]
0040188B  add     eax, 2                           ;  j=j+2
0040188E  mov     dword ptr \[ebp-4\], eax
……花指令
0040189D  mov     ecx, dword ptr \[ebp-4\]
004018A0  movsx   edx, byte ptr \[ecx\]              ;  STR(j)
004018A3  test    edx, edx
004018A5  jnz     short 004018B6                   ;  if STR(j)=0
004018A7  cmp     dword ptr \[ebp-28\], 18
004018AB  jge     short 004018B6                   ;  and  i<0x18 then
004018AD  mov     eax, dword ptr \[ebp-8\]
004018B0  add     eax, 1
004018B3  mov     dword ptr \[ebp-4\], eax           ;  j=1
004018B6  jmp     0040181F                         ;  loop
004018BB  push    0                                          ; /Style = MB\_OK|MB\_APPLMODAL
004018BD  push    00403560                                   ; |Title = "错了"
004018C2  push    00403554                                   ; |Text = "继续努力!"
004018C7  movsx   ecx, byte ptr \[4040F6\]                     ; |SN(A)
004018CE  sub     ecx, 55                                    ; |SN(A)-55
004018D1  neg     ecx                                        ; |
004018D3  sbb     ecx, ecx                                   ; |
004018D5  inc     ecx                                        ; |
004018D6  push    ecx                                        ; |hOwner,只有在 SN(A)=55 时才为 1，其余时候为 0
004018D7  call    dword ptr \[<&USER32.MessageBoxA>\]          ; \\当 hOwner=1 时不会弹出错误窗口
004018DD  mov     dword ptr \[ebp-28\], eax                    ;  如果有弹出窗口，这里会有返回值
004018E0  cmp     dword ptr \[ebp-28\], 0                      ;  有返回值时：
004018E4  je      short 004018FB                             ;  终止程序
004018E6  call    dword ptr \[<&KERNEL32.GetCurrentProcess>\]  ; \[GetCurrentProcess
004018EC  mov     dword ptr \[ebp-2C\], eax
004018EF  push    0                                          ; /ExitCode = 0
004018F1  mov     edx, dword ptr \[ebp-2C\]                    ; |
004018F4  push    edx                                        ; |hProcess
004018F5  call    dword ptr \[<&KERNEL32.TerminateProcess>\]   ; \\TerminateProcess

```

这个地方有点意思，利用设置 MessageBoxA 的值为 1，来设置是否显示 MessageBoxA，再根据返回值判断是否跳过终止进程代码。  
只有当 SN(A)=55 时，hOwner=1，MessageBoxA 就不会显示。即可得出 KEY 最后一位必须为“p”。  
总结这一部分代码为：  
代码:

```
    STR = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
    i = 0
    j = 0
    Do Until i = &H18
        k = Asc(Mid$(STR, j + 1, 1)) + &H20
        If k = Asc(Mid$(UN, 1, 1)) Or k = Asc(Mid$(UN, 6, 1)) Then j = j + 1
        NEWSTR = NEWSTR & Mid$(STR, j + 1, 1)
        i = i + 1
        j = j + 2
        If j >= Len(STR) And i < &H18 Then j = 1
    Loop

```

再次输入用户名：lelfep，试炼码：QLJEQLKFJEUP，继续跟踪：  
代码:

```
004018FB  mov     dword ptr \[ebp-28\], 0            ;  i=0
00401902  mov     dword ptr \[ebp-30\], 5            ;  k=5
00401909  mov     dword ptr \[ebp-34\], 0            ;  n=0
00401910  cmp     dword ptr \[ebp-28\], 0C           ;  do until (i=0xC)----大循环开始
00401914  je      004019D5
0040191A  mov     eax, dword ptr \[ebp-30\]          ;  k
0040191D  lea     ecx, dword ptr \[eax\*4-4\]         ;  k\*4-4
00401924  mov     dword ptr \[ebp-38\], ecx          ;  m=k\*4-4
00401927  mov     edx, dword ptr \[ebp-28\]          ;  do----小循环开始
0040192A  movsx   eax, byte ptr \[edx+4040EC\]       ;  SN(i)
00401931  mov     ecx, dword ptr \[ebp-38\]          ;  m
00401934  movsx   edx, byte ptr \[ebp+ecx-24\]       ;  NEWSTR(m)
00401939  mov     ecx, dword ptr \[ebp-38\]
0040193C  add     ecx, 1                           ;  m=m+1
0040193F  mov     dword ptr \[ebp-38\], ecx
00401942  cmp     eax, edx                         ;  if SN(i)=NEWSTR(m) then
00401944  je      short 00401959                   ;  exit do
00401946  mov     edx, dword ptr \[ebp-34\]
00401949  add     edx, 1                           ;  n=n+1
0040194C  mov     dword ptr \[ebp-34\], edx
0040194F  cmp     dword ptr \[ebp-34\], 4
00401953  jle     short 00401957                   ;  if n>4 then
00401955  jmp     short 00401959                   ;  exit do
00401957  jmp     short 00401927                   ;  loop----继续小循环
00401959  cmp     dword ptr \[ebp-34\], 5
0040195D  jnz     short 0040199E                   ;  if n=5 then----
0040195F  mov     dword ptr \[ebp-3C\], 00403578     ;  ASCII "ABCDEFGHIJKLMN"
00401966  mov     dword ptr \[ebp-40\], 00403568     ;  ASCII "OPQRSTUVWXYZ"
0040196D  push    eax
0040196E  lea     eax, dword ptr \[ebp-3C\]
00401974  push    eax
00401975  push    dword ptr \[ebp-40\]
00401978  pop     eax
00401979  pop     eax
0040197A  pop     eax
0040197B  mov     dword ptr \[ebp-34\], eax
0040197E  cmp     dword ptr \[ebp-34\], 0
00401982  jle     short 0040198B
00401984  call    004017B0                         ;  MsgBox("继续努力")
00401989  jmp     short 00401999
0040198B  push    eax
0040198C  lea     eax, dword ptr \[ebp-3C\]
00401992  push    eax
00401993  push    dword ptr \[ebp-40\]
00401996  pop     eax
00401997  pop     eax
00401998  pop     eax
00401999  call    004017B0                         ;  MsgBox("继续努力")
0040199E  mov     eax, dword ptr \[ebp-28\]          ;  end if n=5----
004019A1  add     eax, 2                           ;  i=i+2
004019A4  mov     dword ptr \[ebp-28\], eax
004019A7  mov     ecx, dword ptr \[ebp-30\]
004019AA  sub     ecx, 1                           ;  k=k-1
004019AD  mov     dword ptr \[ebp-30\], ecx
……花指令
004019BC  cmp     dword ptr \[ebp-30\], 0
004019C0  jnz     short 004019C9                   ;  if k=0 then
004019C2  mov     dword ptr \[ebp-30\], 6            ;  k=6
004019C9  mov     dword ptr \[ebp-34\], 0            ;  n=0
004019D0  jmp     00401910                         ;  loop====继续大循环
004019D5  call    00401770                         ;  MsgBox("过关")
004019DA  pop     edi
004019DB  pop     esi
004019DC  pop     ebx
004019DD  mov     esp, ebp
004019DF  pop     ebp
004019E0  retn

```

总结这段代码为：  
代码:

```
    i = 0
    k = 5
    n = 0
    Do Until i = 12
        m = k \* 4 - 4
        Do
            If Mid$(SN, i + 1, 1) = Mid$(NEWSTR, m + 1, 1) Then Exit Do
            m = m + 1
            n = n + 1
            If n > 4 Then Exit Do
        Loop
        If n = 5 Then
            MsgBox ("wrong...")
            Exit Sub
        End If
        i = i + 2
        k = k - 1
        If k = 0 Then k = 6
        n = 0
    Loop
    MsgBox ("right!")

```

到这里已经可以写出程序的主流程了：  
代码:

```
Private Sub txtText1\_Change()
    Dim i As Long, j As Long, k As Long, m As Long, n As Long
    Dim UN As String, SN As String, STR As String, NEWSTR As String
    
    UN = txtText1.Text
    If Len(UN) <> 6 Then Exit Sub
    For i = 1 To 6
        If Not (Mid$(UN, i, 1) > "a" And Mid$(UN, i, 1) < "z") Then Exit Sub
    Next
    If Mid$(UN, 6, 1) <> "p" Then Exit Sub
    For i = 1 To Len(UN)
        j = Asc(Mid$(UN, i, 1))
        SN = SN & Chr(j - &H1B) & Chr(j - &H20)
    Next
    txtText2.Text = SN
    
    STR = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
    i = 0
    j = 0
    Do Until i = &H18
        k = Asc(Mid$(STR, j + 1, 1)) + &H20
        If k = Asc(Mid$(UN, 1, 1)) Or k = Asc(Mid$(UN, 6, 1)) Then j = j + 1
        NEWSTR = NEWSTR & Mid$(STR, j + 1, 1)
        i = i + 1
        j = j + 2
        If j >= Len(STR) And i < &H18 Then j = 1
    Loop
    txtText3.Text = NEWSTR
    
    Do Until i = 12
        m = k \* 4 - 4
        Do
            If Mid$(SN, i + 1, 1) = Mid$(NEWSTR, m + 1, 1) Then Exit Do
            m = m + 1
            n = n + 1
            If n > 4 Then Exit Do
        Loop
        If n = 5 Then
            MsgBox ("wrong...")
            Exit Sub
        End If
        i = i + 2
        k = k - 1
        If k = 0 Then k = 6
        n = 0
    Loop
    MsgBox ("Right!")
End Sub

```

到这里可以看出，用户名是有限制的，必须为 b--y 之间的小写字母，而当用户名为 v--y 时注册码会出现非字母导致注册失败，因此用户名为 6 位 b--u 之间的字母，并且最后一位必须为 p。  
继续深入思考，NEWSTR 的每一位是有规律的，基本上是隔一位取一个大写字母，只有当字母与注册码指定位相同时才会跳过一个字母，因此每一位字母误差在+1 左右，越到后面误差越大。  
基本字母范围为：  
序号：0123456789ABCDEF01234567  
字母：ACEGIKMOQSUWYBDFHJMOQSUW----NEWSTR  
用户：---bdfhjlnprt--acehjlnpr  
第 1 位范围限定位是 0x10--0x14，即用户名范围为 c--l，误差范围为 b--n  
第 2 位范围限定位是 0x0C--0x10，即用户名范围为 t--c，误差范围为 t--u,b--e  
第 3 位范围限定位是 0x08--0x0C，即用户名范围为 l--t，误差范围为 l--u  
第 4 位范围限定位是 0x04--0x08，即用户名范围为 d--l，误差范围为 d--m  
第 5 位范围限定位是 0x00--0x04，即用户名范围为 b--d，误差范围为 b--e  
第 6 位为固定位 p  
其中第 4，5 位误差范围+1，第 1，2，3 位误差范围+2。  
可以用随机选择范围内的字母组合成用户名，然后测试用户名是否合法的方式做 KeyGen。  
VB 写的 KeyGen 见附件。  
  
给出一组可用码：  
gerecp  
LGJEWRJEHCUP

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || vb\_Source\_Code.rar  
 |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || PEDIY01\_EXE.rar  
 |

源代码没写注释，就在这补一下吧  
  

代码:

```
Option Explicit

'初始 STR
Private Const STR As String = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"

'生成 1--nMax 之间的随机数
Private Function GetRndNum(nMax As Long) As Long
    Randomize
    GetRndNum = Int(Rnd \* nMax) + 1
End Function

'由用户名生成序列号
Private Function GetSN(UN As String) As String
Dim i As Long, j As Long, SN As String

    For i = 1 To Len(UN)
        j = Asc(Mid$(UN, i, 1))
        SN = SN & Chr(j - &H1B) & Chr(j - &H20)
    Next
    GetSN = SN
End Function

'由用户名生成 NEWSTR
Private Function GetNewStr(UN As String) As String
Dim i As Long, j As Long, k As Long, NEWSTR As String

    Do Until i = &H18
        k = Asc(Mid$(STR, j + 1, 1)) + &H20
        If k = Asc(Mid$(UN, 1, 1)) Or k = Asc(Mid$(UN, 6, 1)) Then j = j + 1
        NEWSTR = NEWSTR & Mid$(STR, j + 1, 1)
        i = i + 1
        j = j + 2
        If j >= Len(STR) And i < &H18 Then j = 1
    Loop
    GetNewStr = NEWSTR
End Function

'校验序列号，返回值为校验用户名的每一位合成 0bxxxxxx，校验位正确时 x=1
Private Function CheckSN(SN As String, NEWSTR As String) As Long
Dim i As Long, k As Long, m As Long, n As Long
Dim iRet As Long

    i = 0
    k = 5
    n = 0
    iRet = 0
    
    Do Until i = 12
        m = k \* 4 - 4
        Do
            If Mid$(SN, i + 1, 1) = Mid$(NEWSTR, m + 1, 1) Then Exit Do
            m = m + 1
            n = n + 1
            If n > 4 Then Exit Do
        Loop
        iRet = iRet \* 2 + IIf(n = 5, 0, 1)
        i = i + 2
        k = k - 1
        If k = 0 Then k = 6
        n = 0
    Loop
    CheckSN = iRet
End Function

'获取一个合法用户名
Private Function GetValidUser() As String
    Dim SN1(), SN2(), SN3(), SN4(), SN5(), SN6 As String
    Dim ID(1 To 5) As Long
    Dim UN As String, SN As String, NEWSTR As String
    Dim iCheck As Long, iCount As Long
    
    Const MAX\_TIMES = &H20    '循环最大次数，达到时重新生成每一位用户名
    
    SN1() = Array("b", "c", "d", "e", "f", "g", "h", "i", "j", "k", "l", "m", "n") '用户名第一位
    SN2() = Array("t", "u", "b", "c", "d", "e") '用户名第二位
    SN3() = Array("l", "m", "n", "o", "p", "q", "r", "s", "t", "u") '用户名第三位
    SN4() = Array("d", "e", "f", "g", "h", "i", "j", "k", "l", "m") '用户名第四位
    SN5() = Array("b", "c", "d", "e") '用户名第五位
    SN6 = "p" '用户名第六位
    
    Do
        iCount = 0
        iCheck = 0
        ID(1) = GetRndNum(UBound(SN1)) '随机生成用户名第 1--5 位
        ID(2) = GetRndNum(UBound(SN2))
        ID(3) = GetRndNum(UBound(SN3))
        ID(4) = GetRndNum(UBound(SN4))
        ID(5) = GetRndNum(UBound(SN5))
        
        Do
            UN = SN1(ID(1)) & SN2(ID(2)) & SN3(ID(3)) & SN4(ID(4)) & SN5(ID(5)) & SN6 '组合用户名
            SN = GetSN(UN) '获取序列号
            NEWSTR = GetNewStr(UN) '获取 NEWSTR
            
            iCheck = CheckSN(SN, NEWSTR) '校验序列号
            Select Case True
                Case (iCheck And &H20) = 0  '从第一位开始检测，某位不正确时重新生成
                    ID(1) = GetRndNum(UBound(SN1))
                Case (iCheck And &H10) = 0
                    ID(2) = GetRndNum(UBound(SN2))
                Case (iCheck And &H8) = 0
                    ID(3) = GetRndNum(UBound(SN3))
                Case (iCheck And &H4) = 0
                    ID(4) = GetRndNum(UBound(SN4))
                Case (iCheck And &H2) = 0
                    ID(5) = GetRndNum(UBound(SN5))
                Case (iCheck And &H1) = 0
                    iCount = MAX\_TIMES  '最后一位不正确？那是不可能的
            End Select
            iCount = iCount + 1  '循环次数，防止死循环
        Loop Until iCheck = &H3F Or iCount >= MAX\_TIMES  '找到合法用户名或达到循环次数？
    Loop Until iCheck = &H3F '=0b00111111  '找到合法用户了？
    
    GetValidUser = UN  '返回用户名
End Function

Private Sub cmdCommand1\_Click()
    txtText1.Text = GetValidUser
    txtText2.Text = GetSN(txtText1.Text)
End Sub

```