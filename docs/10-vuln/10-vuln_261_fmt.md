
# 中国游戏中心游戏大厅 ActiveX 远程栈溢出漏洞

> 原文：[`www.pediy.com/kssd/pediy10/87615.html`](https://www.pediy.com/kssd/pediy10/87615.html)

中国游戏中心游戏大厅 ActiveX 远程栈溢出漏洞

by `bugvuln(bugvuln_at_gmail.com)`

`niklen(niklenxyz_at_gmail.com)`

## 描述

中国游戏中心是中国一款著名的游戏平台，含棋牌，网络游戏等

`http://www.chinagames.net/`

## 受影响的系统

中国游戏中心游戏大厅 2009

## 细节

`clsid:75108B29-202F-493C-86C5-1C182A485C4C`

`C:\Program Files\Chinagames\iGame\CGAgent.dll`

`Sub CreateChinagames (ByVal lpszToken As String)`

参数 `lpszToken` 是一个超长字符串时，发生栈溢出，利用堆填充技术，攻击者可以很轻松的利用此漏洞执行任意代码

## 分析

```
.text:10001234                cmp    [ebp+lpString2], esi  ; 只判断了参数是否为空，但是没有判断长度-_-!!!
.text:10001237                jnz    short loc_10001243
...
.text:10001295                push    offset aIgame_exe  ; "\\\\igame.exe  "
.text:1000129A                push    eax            ; lpString1
.text:1000129B                call    edi  ; lstrcatW
.text:1000129D                push    [ebp+lpString2]  ; lpString2
.text:100012A0                lea    eax, [ebp+String1]
.text:100012A6                push    eax            ; lpString1
.text:100012A7                call    edi  ; lstrcatW  ; 没有对传递给 igame.exe 的参数进行长度检查，直接连接，
.text:100012A7                                ; 这下子，eax 指向的堆栈区域都被非法参数给占领了-_-！
.text:100012A7                                ;
.text:100012A9                push    44h            ; 结构体长度
.text:100012AB                lea    eax, [ebp+Dst]
.text:100012AE                push    esi            ; Val
.text:100012AF                push    eax            ; Dst
.text:100012B0                call    memset          ; 为 STARTUPINFO 作准备
.text:100012B5                add    esp, 0Ch
.text:100012B8                lea    eax, [ebp+ProcessInformation]
.text:100012BB                mov    [ebp+Dst], 44h
.text:100012C2                push    eax            ; lpProcessInformation
.text:100012C3                lea    eax, [ebp+Dst]
.text:100012C6                push    eax            ; lpStartupInfo
.text:100012C7                push    esi            ; lpCurrentDirectory
.text:100012C8                push    esi            ; lpEnvironment
.text:100012C9                push    10h            ; dwCreationFlags
.text:100012CB                push    esi            ; bInheritHandles
.text:100012CC                push    esi            ; lpThreadAttributes
.text:100012CD                lea    eax, [ebp+String1]
.text:100012D3                push    esi            ; lpProcessAttributes
.text:100012D4                push    eax            ; [igame.exe 参数]
.text:100012D5                push    esi            ; lpApplicationName
.text:100012D6                call    ds:CreateProcessW  ; 创建进程
.text:100012DC                test    eax, eax
.text:100012DE                jz      short loc_100012F0  ; 成功了，继续 xxoo
.text:100012E0                xor    eax, eax
.text:100012E2
.text:100012E2 loc_100012E2:                          ; CODE XREF: sub_10001218+26j
.text:100012E2                mov    ecx, [ebp+var_4]  ; (.text:100012A7)处那里导致把这里给覆盖了
.text:100012E5                mov    edx, [ebp+var_8]  ; 同上
.text:100012E8                mov    [ecx+4], edx    ; 互爆#_#
.text:100012EB                jmp    loc_10001429
...
.text:10001429 loc_10001429:                          ; CODE XREF: sub_10001218+D3j
.text:10001429                pop    edi
.text:1000142A                pop    esi
.text:1000142B                pop    ebx
.text:1000142C                leave
.text:1000142D                retn    8              ; 就这样返回，哦豁了
```

```
ModLoad: 01d10000 01d1d000  C:\Program Files\Chinagames\iGame\CGAgent.dll
ModLoad: 5f800000 5f8f2000  C:\windows\system32\MFC42u.DLL
ModLoad: 61be0000 61bed000  C:\windows\system32\MFC42LOC.DLL
ModLoad: 75bc0000 75c3d000  C:\WINDOWS\system32\jscript.dll
(39c.28c): Access violation - code c0000005 (first chance)
First chance exceptions are reported before any exception handling.
This exception may be expected and handled.
eax=41baf50e ebx=01d153ec ecx=00410041 edx=00410041 esi=003bd4f0 edi=00000000
eip=00410049 esp=01cff5ec ebp=00410041 iopl=0        nv up ei ng nz ac pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000            efl=00010296
*** ERROR: Module load completed but symbols could not be loaded for C:\Program Files\internet explorer\iexplore.exe
iexplore+0x10049:
00410049 004000          add    byte ptr [eax],al          ds:0023:41baf50e=??
```

## 解决办法

在厂商没有推出相应的补丁之前，

建议用户通过注册表对相应的 `CLSID:75108B29-202F-493C-86C5-1C182A485C4C` 设置 Killbit

或者将以下文本保存为`.REG` 文件并导入：

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Internet Explorer\ActiveX Compatibility\{75108B29-202F-493C-86C5-1C182A485C4C}]

"Compatibility Flags"=dword:00000400
```

## 厂商回应

2009.04.29 发邮件给 `gmservice@chinagames.net`

2009.04.30 木有收到回复。再发送了一封邮件

2009.04.30 还是不甩??发布此公告

--EOF--
