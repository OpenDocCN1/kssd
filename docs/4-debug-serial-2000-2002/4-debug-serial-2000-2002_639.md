
# 金庸群侠传外挂程序

> 原文：[`www.pediy.com/kssd/pediy03/forum1163.htm`](https://www.pediy.com/kssd/pediy03/forum1163.htm)

金庸群侠外挂程序

by `6767 [BCG]`

用 `dede` 和 `si` 配合容易搞定，具体跳转该怎样修改可能要在测试后才能确定，下面是我的改法：

资源段有下列信息：

```
object TabSheet1: TTabSheet
    Caption = '登录注册'
    OnShow = TabSheet1Show    <- 这个看看

...

    object WebBrowser1: TWebBrowser
        Left = 0
        Top = 0
        Width = 307
        Height = 284
        TabStop = False
        Align = alClient
        TabOrder = 0
        OnTitleChange = WebBrowser1TitleChange    <- 这个也看看

...
```

先是 `TabSheet1Show` 中的代码：

```
0045395B  64FF30        push    dword ptr fs:[eax]
0045395E  648920        mov     fs:[eax], esp
* Possible String Reference to: '金庸外挂 I 之《神行太保》,    <- 改变标题，激活 WebBrowser1TitleChange 调用             衿?..'

|

00453961  BA003A4500    mov     edx, $00453A00
* Possible reference to Form1

|

00453966  A1F0684500    mov     eax, dword ptr [$4568F0]
* Reference to: Controls.TControl.SetText(System.AnsiString)    <-

|

0045396B  E8A037FDFF    call    00427110
00453970  8D45EC        lea     eax, [ebp-$14]
00453973  33D2          xor     edx, edx
.....
```

那么我们来看 `WebBrowser1TitleChange` 都作了些什么：

```
00453A6C  55            push    ebp
00453A6D  8BEC          mov     ebp, esp
00453A6F  83C4F4        add     esp, -$0C
00453A72  53            push    ebx
00453A73  56            push    esi
00453A74  33DB          xor     ebx, ebx
00453A76  895DF4        mov     [ebp-$0C], ebx
00453A79  8BF1          mov     esi, ecx
00453A7B  8BD8          mov     ebx, eax
00453A7D  33C0          xor     eax, eax
00453A7F  55            push    ebp
* Possible String Reference to: '镽﹝嬪﹞?

|

00453A80  68793B4500    push    $00453B79
***** TRY

|

00453A85  64FF30        push    dword ptr fs:[eax]
00453A88  648920        mov     fs:[eax], esp
* Possible String Reference to: 'xxxxxxxx'

|

00453A8B  BA903B4500    mov     edx, $00453B90    <- 觉得标题难受就把这里改掉
* Possible reference to Form1

|

00453A90  A1F0684500    mov     eax, dword ptr [$4568F0]
* Reference to: Controls.TControl.SetText(System.AnsiString)    <- 设窗口标题

|

00453A95  E87636FDFF    call    00427110
00453A9A  8BC6          mov     eax, esi     <- 接受前一函数的返回值
00453A9C  BAA03B4500    mov     edx, $00453BA0
* Reference to: System..WStrCmp()

|

00453AA1  E83A06FBFF    call    004040E0     <- 比较了
00453AA6  7507          jnz     00453AAF     <- 不跳就完蛋，改掉

              <- 改成 jmp 453b0b 运行没问题，不知能否正确接收到数据
00453AA8  8BC3          mov     eax, ebx
* Reference to: Forms.TCustomForm.Close()

|

00453AAA  E871C7FEFF    call    00440220
00453AAF  8BC6          mov     eax, esi
00453AB1  BAB83B4500    mov     edx, $00453BB8
* Reference to: System..WStrCmp()

|

00453AB6  E82506FBFF    call    004040E0
00453ABB  7516          jnz     00453AD3     <- 未注册在这里会跳走
00453ABD  8D45F4        lea     eax, [ebp-$0C]
00453AC0  8BD6          mov     edx, esi
* Reference to: System..LStrFromWStr(System.AnsiString; System.WideString)

|

00453AC2  E88D00FBFF    call    00403B54
00453AC7  8B45F4        mov     eax, [ebp-$0C]
* Reference to: Sysutils.StrToFloat(System.AnsiString)

|

00453ACA  E8054FFBFF    call    004089D4
00453ACF  DD5DF8        fstp    qword ptr [ebp-$08]
00453AD2  9B            wait
00453AD3  DD45F8        fld     qword ptr [ebp-$08]
00453AD6  83C4F4        add     esp, -$0C
00453AD9  DB3C24        fstp    tbyte ptr [esp]
00453ADC  9B            wait
00453ADD  8D45F4        lea     eax, [ebp-$0C]
* Reference to: Sysutils.FloatToStr(Extended)

|

00453AE0  E8BF4EFBFF    call    004089A4
00453AE5  8B45F4        mov     eax, [ebp-$0C]
* Possible String Reference to: '9412.11'

|

00453AE8  BAD03B4500    mov     edx, $00453BD0
* Reference to: System..LStrCmp()

|

00453AED  E8AA01FBFF    call    00403C9C
00453AF2  756F          jnz     00453B63     <- 跳下去也没戏了
00453AF4  8D45F4        lea     eax, [ebp-$0C]
00453AF7  8BD6          mov     edx, esi
* Reference to: System..LStrFromWStr(System.AnsiString; System.WideString)

|

00453AF9  E85600FBFF    call    00403B54
00453AFE  8B45F4        mov     eax, [ebp-$0C]
* Reference to: Sysutils.StrToFloat(System.AnsiString)

|

00453B01  E8CE4EFBFF    call    004089D4
* Reference to: System..ROUND()

|

00453B06  E835EEFAFF    call    00402940
* Reference to field TForm1.OFFS_031C

|

00453B0B  89831C030000  mov     [ebx+$031C], eax  <- 正常程序从这里或稍上处开始
00453B11  B201          mov     dl, $01
* Reference to control TForm1.TabSheet2 : TTabSheet     <-

|

00453B13  8B83D4020000  mov     eax, [ebx+$02D4]
* Reference to: Controls.TControl.SetVisible(bool)     <- 显示地图内容，下略

|

00453B19  E8B234FDFF    call    00426FD0
00453B1E  B201          mov     dl, $01
* Reference to control TForm1.TabSheet2 : TTabSheet

|

00453B20  8B83D4020000  mov     eax, [ebx+$02D4]
* Reference to: Comctrls.TTabSheet.SetTabVisible(bool)

|

00453B26  E84DBAFFFF    call    0044F578
00453B2B  B201          mov     dl, $01
* Reference to control TForm1.TabSheet2 : TTabSheet

|

00453B2D  8B83D4020000  mov     eax, [ebx+$02D4]
00453B33  8B08          mov     ecx, [eax]
00453B35  FF5160        call    dword ptr [ecx+$60]
00453B38  33D2          xor     edx, edx
* Reference to control TForm1.TabSheet1 : TTabSheet

|

00453B3A  8B83C8020000  mov     eax, [ebx+$02C8]
* Reference to: Controls.TControl.SetVisible(bool)

|

00453B40  E88B34FDFF    call    00426FD0
00453B45  33D2          xor     edx, edx
* Reference to control TForm1.TabSheet1 : TTabSheet

|

00453B47  8B83C8020000  mov     eax, [ebx+$02C8]
* Reference to: Comctrls.TTabSheet.SetTabVisible(bool)

|

00453B4D  E826BAFFFF    call    0044F578
* Reference to control TForm1.TabSheet2 : TTabSheet

|

00453B52  8B93D4020000  mov     edx, [ebx+$02D4]
* Reference to control TForm1.PageControl1 : TPageControl

|

00453B58  8B83C4020000  mov     eax, [ebx+$02C4]
* Reference to: Comctrls.TPageControl.SetActivePage(Comctrls.TTabSheet)

|

00453B5E  E8D5C0FFFF    call    0044FC38
00453B63  33C0          xor     eax, eax
00453B65  5A            pop     edx
00453B66  59            pop     ecx
00453B67  59            pop     ecx
00453B68  648910        mov     fs:[eax], edx
****** FINALLY

|

* Possible String Reference to: '^\[嬪\]?

|

00453B6B  68803B4500    push    $00453B80
00453B70  8D45F4        lea     eax, [ebp-$0C]
* Reference to: System..LStrClr(System.AnsiString)

|

00453B73  E898FDFAFF    call    00403910
00453B78  C3            ret
00453B79  E952F8FAFF    jmp     004033D0
00453B7E  EBF0          jmp     00453B70
```



\*\*\*\*\*\* END

```
00453B80 5E                     pop    esi
00453B81 5B                     pop    ebx
00453B82 8BE5                   mov    esp, ebp
00453B84 5D                     pop    ebp
00453B85 C3                     ret
```

关键的跳转处都列出来了，至于怎样改需要测试。

那个记录地址的文本文件是需要的，否则这个东东没用。`4dd466d0e1e0cc123afbe504da2462ca`
