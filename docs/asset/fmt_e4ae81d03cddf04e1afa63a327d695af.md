
# XoftSpy 4.13 的注册算法分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-286.htm`](https://www.pediy.com/kssd/pediy07/pediy7-286.htm)

## 标题
XoftSpy 4.13 的注册算法分析

## 作者
`forever[RCT]`

## 语言
`VC`

## 工具
`ida4.6`, `ollydbg1.1`

## 正文
这个软件的算法很简单，正好拿来做逆向分析。我就会捏软柿子。呵呵。

因为这个软件注册失败会弹出一个对话框来提示您，所以在 `ollydbg` 中在函数 `MessageBox` 上下断点很容易就找到软件的关键算法处。这个软件装载时间很长，所以如果您也要分析它，建议用 `ollydbg` 来 attach 一下。我就是这么的。:)

其他的都省略了。这里只分析一下关键的算法部分：

```asm
.text:00417690 sub_417690      proc near
.text:00417690
.text:00417690
.text:00417690 var_28          = dword ptr -28h
.text:00417690 var_24          = dword ptr -24h
.text:00417690 var_20          = dword ptr -20h
.text:00417690 var_1C          = dword ptr -1Ch
.text:00417690 var_18          = dword ptr -18h
.text:00417690 var_14          = dword ptr -14h
.text:00417690 var_10          = dword ptr -10h
.text:00417690 var_C           = dword ptr -0Ch
.text:00417690 var_4           = dword ptr -4
.text:00417690 arg_myName      = dword ptr  4
.text:00417690 arg_myCode      = dword ptr  8
.text:00417690
.text:00417690                 push    0FFFFFFFFh
.text:00417692                 push    offset unknown_libname_428
.text:00417697                 mov     eax, large fs:0
.text:0041769D                 push    eax
.text:0041769E                 mov     large fs:0, esp
.text:004176A5                 sub     esp, 1Ch
.text:004176A8                 push    ebx
.text:004176A9                 push    esi
.text:004176AA                 push    edi
.text:004176AB                 mov     eax, [esp+34h+arg_myName]
.text:004176AF                 push    offset byte_48FA48  //这里是 0
.text:004176B4                 push    eax
.text:004176B5                 mov     [esp+3Ch+var_4], 1
.text:004176BD                 call    __mbscmp
.text:004176C2                 add     esp, 8
.text:004176C5                 test    eax, eax
.text:004176C7                 jz      loc_4178F5
//注册名为空则退出
.text:004176CD                 mov     ecx, [esp+34h+arg_myCode]
.text:004176D1                 push    offset byte_48FA48
.text:004176D6                 push    ecx
.text:004176D7                 call    __mbscmp
.text:004176DC                 add     esp, 8
.text:004176DF                 test    eax, eax
.text:004176E1                 jz      loc_4178F5
//注册码为空则退出
.text:004176E7                 mov     edx, [esp+34h+arg_myCode]
.text:004176EB                 cmp     dword ptr [edx-8], 21
.text:004176EF                 jl      loc_4178F5
//注册码长度小于 21 位则退出
.text:004176F5                 lea     eax, [esp+34h+var_18]
.text:004176F9                 push    11
.text:004176FB                 push    eax
.text:004176FC                 lea     ecx, [esp+3Ch+arg_myCode]
.text:00417700                 call    ?Left@CString@@QBE?AV1@H@Z ; CString::Left(int)
.text:00417705                 mov     ecx, [esp+34h+var_18]
//取注册码左边 11 个字符
.text:00417709                 xor     esi, esi
//mov 指令执行后 ecx 是字符串的地址
.text:0041770B                 xor     edx, edx
//这是指针和变量的区别，变量一般用 lea
.text:0041770D                 mov     byte ptr [esp+34h+var_4], 2
.text:00417712                 mov     [esp+34h+var_14], esi
.text:00417716                 xor     eax, eax
.text:00417718
.text:00417718 loc_417718:
.text:00417718                 movsx   edi, byte ptr [eax+ecx]
//取一个字符(符号扩展的，所以 edi 是整型数)
.text:0041771C                 add     edx, edi
//累加到 edx
.text:0041771E                 inc     eax
//索引加 1
.text:0041771F                 cmp     eax, 0Ah
.text:00417722                 jle     short loc_417718
//这里用的 jle，所以累加 11 个字节
.text:00417724                 mov     [esp+34h+var_10], edx
//保存累加和
.text:00417728                 mov     edx, [esp+34h+arg_myName]
.text:0041772C                 xor     eax, eax
.text:0041772E                 mov     ecx, [edx-8]
//这个位置是 CString 类型的变量的长度
.text:00417731                 test    ecx, ecx
.text:00417733                 jle     short loc_417744
//用户名长度小于等于 0 则跳
.text:00417735
.text:00417735 loc_417735:
.text:00417735                 movsx   edi, byte ptr [eax+edx]
//取用户名一个字符
.text:00417739                 add     esi, edi
//累加到 esi
.text:0041773B                 inc     eax
.text:0041773C                 cmp     eax, ecx
.text:0041773E                 jl      short loc_417735
//(注意这里用的 jl)
.text:00417740                 mov     [esp+34h+var_14], esi
//保存用户名累加和
.text:00417744
.text:00417744 loc_417744:
.text:00417744                 mov     ecx, [esp+34h+arg_myCode]
.text:00417748                 push    ebp
.text:00417749                 lea     edx, [esp+38h+var_28]
.text:0041774D                 mov     eax, [ecx-8]
.text:00417750                 lea     ecx, [esp+38h+arg_myCode]
.text:00417754                 add     eax, -12
//注册码长度减去 12
.text:00417757                 push    eax
.text:00417758                 push    edx
.text:00417759                 call    ?Right@CString@@QBE?AV1@H@Z ; CString::Right(int)
.text:0041775E                 lea     eax, [esp+38h+var_1C]
//上面是取右半部分，不要左边的 12 个字符
.text:00417762                 push    3
.text:00417764                 push    eax
.text:00417765                 lea     ecx, [esp+40h+var_28]
.text:00417769                 mov     byte ptr [esp+40h+var_4], 3
.text:0041776E                 call    ?Left@CString@@QBE?AV1@H@Z ; CString::Left(int)
.text:00417773                 mov     ecx, [esp+38h+var_1C]
//取右半部分的左边 3 个字符
.text:00417777                 mov     byte ptr [esp+38h+var_4], 4
.text:0041777C                 push    ecx       ; char *
.text:0041777D                 call    _atoi
//转换成整数
.text:00417782                 cdq
.text:00417783                 add     esp, 4
.text:00417786                 sub     eax, edx
.text:00417788                 lea     edx, [esp+38h+var_20]
.text:0041778C                 mov     esi, eax
//结果放在这里
.text:0041778E                 push    3
.text:00417790                 push    3
.text:00417792                 push    edx
.text:00417793                 lea     ecx, [esp+44h+var_28]
.text:00417797                 sar     esi, 1
//除以 2（符号右移 1 位）
.text:00417799                 call    ?Mid@CString@@QBE?AV1@HH@Z ; CString::Mid(int,int)
.text:0041779E                 mov     eax, [esp+38h+var_20]
//依次再取 3 个字符
.text:004177A2                 mov     byte ptr [esp+38h+var_4], 5
.text:004177A7                 push    eax       ; char *
.text:004177A8                 call    _atoi
//转换成整数
.text:004177AD                 cdq
.text:004177AE                 add     esp, 4
.text:004177B1                 and     edx, 3
.text:004177B4                 add     eax, edx
.text:004177B6                 lea     ecx, [esp+38h+var_24]
.text:004177BA                 push    3
```
