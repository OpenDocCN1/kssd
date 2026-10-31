
# [原创]暴风影音2009(Config.dll)ActiveX远程栈溢出漏洞

暴风影音2009(Config.dll)ActiveX远程栈溢出漏洞

by `bugvuln(bugvuln_at_gmail.com)`

`niklen(niklenxyz_at_gmail.com)`

## 描述

暴风影音是国内一款相当流行的万能播放器

`63aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0S2L8$3k6W2L8X3N6Q4x3X3g2U0L8$3#2Q4x3V1j5`.

## 受影响的系统

暴风影音2009 <=`[3.09.04.17]`

## 细节

`clsid:BD103B2B-30FB-4F1E-8C17-D8F6AADBCC05`

`C:\Program Files\StormII\Config.dll`

`Sub SetAttributeValue (`

`ByVal lpQueryStr  As String ,`

`ByVal bstrAttributeName  As String ,`

`ByVal lpValueStr  As String`

`)`

当参数`lpQueryStr`是一个超长字符串时，发生栈溢出，利用堆填充技术，攻击者可以很轻松的利用此漏洞执行任意代码

## 分析

```
.text:10009A4C                push    ebp
.text:10009A4D                mov    ebp, esp
.text:10009A4F                sub    esp, 208h      ; 开辟208h的堆栈空间
.text:10009A55                cmp    [ebp+Source], 0 ; 判断参数1是否为空
.text:10009A59                jz      short loc_10009AA8
.text:10009A5B                cmp    [ebp+arg_C], 0  ; 判断参数3是否为空
.text:10009A5F                jz      short loc_10009AA8
.text:10009A61                push    [ebp+Source]    ; 参数1
.text:10009A64                lea    eax, [ebp+String]
.text:10009A6A                push    eax            ; eax正好指向ebp-208h的堆栈区域
.text:10009A6B                call    ds:wcscpy      ; oh，my god，不进行参数合法性检查，直接开始拷贝操作，
.text:10009A6B                                        ; 哦豁了，eax指向的堆栈区域全部被超长非法参数占领了-\_-！
.text:10009A71                pop    ecx
.text:10009A72                lea    eax, [ebp+String]
.text:10009A78                pop    ecx
.text:10009A79                push    [ebp+arg_8]
.text:10009A7C                push    offset String  ; "/@"
.text:10009A81                push    offset aSS      ; "%s%s"
.text:10009A86                push    eax            ; String
.text:10009A87                call    ds:swprintf    ; 上面的拷贝直接影响到这里的swprintf,相当于再对eax指向的堆栈进行一次拷贝操作
                               ; 没有上边的拷贝，这里也要出问题
.text:10009A8D                add    esp, 10h
.text:10009A90                lea    eax, [ebp+String]
.text:10009A96                push    [ebp+arg_C]
.text:10009A99                push    eax
.text:10009A9A                call    sub_10001201
.text:10009A9F                mov    ecx, eax
.text:10009AA1                call    sub_1000CC9A
.text:10009AA6                jmp    short locret_10009AAD
.text:10009AA8 ; ---------------------------------------------------------------------------
.text:10009AA8
.text:10009AA8 loc_10009AA8:                          ; CODE XREF: sub_10009A4C+Dj
.text:10009AA8                                        ; sub_10009A4C+13j
.text:10009AA8                mov    eax, 80004005h
.text:10009AAD
.text:10009AAD locret_10009AAD:                        ; CODE XREF: sub_10009A4C+5Aj
.text:10009AAD                leave
.text:10009AAE                retn    10h           ; 就这样返回，哦豁了
```

`ModLoad: 41f50000 41fc7000  C:\WINDOWS\system32\mshtmled.dll`

`ModLoad: 10000000 10020000  C:\Program Files\StormII\Config.dll`

`ModLoad: 63380000 633f8000  C:\WINDOWS\system32\jscript.dll`

`(eec.ee8): Illegal instruction - code c000001d (first chance)`

`(eec.ee8): Illegal instruction - code c000001d (!!! second chance !!!)`

`eax=80004005 ebx=100116b0 ecx=0175f998 edx=00030001 esi=0039fe98 edi=00000000`

`eip=00410061 esp=0175f5ec ebp=00410041 iopl=0        nv up ei pl nz ac po nc`

`cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000            efl=00000212`

\*\*\* ERROR: Module load completed but symbols could not be loaded for `C:\Program Files\Internet Explorer\IEXPLORE.EXE`

`IEXPLORE+0x10061:`

`00410061 ff              ???`

## 解决办法

在厂商没有推出相应的补丁之前，

建议用户通过注册表对相应的`CLSID:BD103B2B-30FB-4F1E-8C17-D8F6AADBCC05`设置`Killbit`

或者将以下文本保存为`.REG`文件并导入:

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Internet Explorer\ActiveX Compatibility\{BD103B2B-30FB-4F1E-8C17-D8F6AADBCC05}]

"Compatibility Flags"=dword:00000400
```

--EOF--

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
