
# [原创]暴风影音2009(`mps.dll`)ActiveX远程栈溢出漏洞

暴风影音2009(`mps.dll`)ActiveX远程栈溢出漏洞

by bugvuln(bugvuln_at_gmail.com), niklen(niklenxyz_at_gmail.com)

## 描述

暴风影音是国内一款相当流行的万能播放器

`be4K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0S2L8$3k6W2L8X3N6Q4x3X3g2U0L8$3#2Q4x3V1j5`.

## 受影响的系统

暴风影音2009 `<=[3.09.04.17]`

## 细节

`clsid:6BE52E1D-E586-474F-A6E2-1A85A9B4D9FB`

`C:\Program Files\StormII\mps.dll`

`Sub OnBeforeVideoDownload(ByVal URL  As String)`

当参数 `URL` 是一个超长字符串时，发生栈溢出，利用堆填充技术，攻击者可以很轻松的利用此漏洞执行任意代码

## 分析

```
sub_10014240
  |--.text:1001430A  call    sub_10014D40
  |--sub_10014D40
        |--.text:10014E37  call    dword ptr [eax+8] ; sub_1005DAF0
        |--sub_1005DAF0
              |--.text:1005DBA1  call    dword ptr [edx+4] ; sub_1005EB50
              |--sub_1005EB50
              |--.text:1005EB9A  call    sub_10060320
                    |--sub_10060320
                          |--.text:1006033C    call    ds:lstrcpynA
                    |    add    esp, 1348h
                    |--{ retn     14h

...

.text:1005EB6F                push    ebp
.text:1005EB70                push    esi
.text:1005EB71                mov     esi, [esp+1350h+lpString2]
.text:1005EB78                push    edi
.text:1005EB79                push    offset aStormbox_0 ; "stormbox"
.text:1005EB7E                mov     ebp, ecx
.text:1005EB80                push    esi            ; Str
.text:1005EB81                call    ds:strstr      ;检查下传递进来的参数是否含有"stormbox"
.text:1005EB87                add     esp, 8
.text:1005EB8A                test    eax, eax
.text:1005EB8C                jnz     loc_1005ED67    ;没有！那就xxoo了
.text:1005EB92                push    esi            ; lpString2
.text:1005EB93                lea     ecx, [esp+1358h+var_1034]
.text:1005EB9A                call    sub_10060320

...

.text:10060320                push    esi
.text:10060321                push    edi
.text:10060322                mov     edi, [esp+8+lpString2]
.text:10060326                mov     esi, ecx
.text:10060328                push    edi            ; lpString
.text:10060329                mov     dword ptr [esi], offset off_1007C5A4
.text:1006032F                call    ds:lstrlenA
.text:10060335                inc     eax
.text:10060336                push    eax            ; iMaxLength,唉，上面这是最后一次调用函数来进行长度检查，但是这里仅仅是为了这个最大值参数，
                               ; 还是没有考虑合法性-_-
.text:10060337                lea     eax, [esi+4]
.text:1006033A                push    edi            ; lpString2
.text:1006033B                push    eax            ; lpString1
.text:1006033C                call    ds:lstrcpynA    ; 拷贝到eax指向的堆栈区域，为即将到来的溢出做热身运动
.text:10060342                mov     eax, esi
.text:10060344                pop     edi
.text:10060345                pop     esi
.text:10060346                retn    4

...

.text:1005ED70                pop     edi
.text:1005ED71                pop     esi
.text:1005ED72                pop     ebp
.text:1005ED73                mov     large fs:0, ecx
.text:1005ED7A                add     esp, 1348h
.text:1005ED80                retn    14h           ; 就这样返回，哦豁了
```

```
ModLoad: 41f50000 41fc7000  C:\WINDOWS\system32\mshtmled.dll
ModLoad: 10000000 100e2000  C:\Program Files\StormII\mps.dll
ModLoad: 75ff0000 76055000  C:\Program Files\StormII\MSVCP60.dll
ModLoad: 02c60000 02c96000  C:\Program Files\StormII\meedb.dll
ModLoad: 02ca0000 02d2e000  C:\Program Files\StormII\splayers.dll
ModLoad: 02730000 0274e000  C:\Program Files\StormII\rndrmgr.dll
ModLoad: 02d30000 02e48000  C:\Program Files\StormII\SubDecoder.dll
ModLoad: 4b640000 4b7e6000  C:\WINDOWS\system32\d3d9.dll
ModLoad: 6dd20000 6dd26000  C:\WINDOWS\system32\d3d8thk.dll
ModLoad: 736d0000 73719000  C:\WINDOWS\system32\DDRAW.dll
ModLoad: 73b30000 73b36000  C:\WINDOWS\system32\DCIMAN32.dll
ModLoad: 74be0000 74c0c000  C:\WINDOWS\system32\OLEACC.dll
ModLoad: 72f70000 72f96000  C:\WINDOWS\system32\WINSPOOL.DRV
ModLoad: 76320000 76367000  C:\WINDOWS\system32\COMDLG32.dll
ModLoad: 02eb0000 02ed2000  C:\Program Files\StormII\mediainfo.dll
ModLoad: 719c0000 719fe000  C:\WINDOWS\system32\mswsock.dll
ModLoad: 60fd0000 61025000  C:\WINDOWS\system32\hnetcfg.dll
ModLoad: 71a00000 71a08000  C:\WINDOWS\System32\wshtcpip.dll
ModLoad: 7cf70000 7d0d8000  C:\WINDOWS\system32\quartz.dll
ModLoad: 63380000 633f8000  C:\WINDOWS\system32\jscript.dll

(a0.8b0): Access violation - code c0000005 (first chance)
First chance exceptions are reported before any exception handling.
This exception may be expected and handled.

eax=ffffff00 ebx=01b3f33c ecx=41414141 edx=00142f90 esi=01b3f328 edi=01b3f340
eip=41414141 esp=0175f588 ebp=01b3f338 iopl=0        nv up ei pl nz ac po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000            efl=00210212
41414141 ??              ???
```

## 解决办法

在厂商没有推出相应的补丁之前，建议用户通过注册表对相应的 `CLSID:6BE52E1D-E586-474F-A6E2-1A85A9B4D9FB` 设置 Killbit。或者将以下文本保存为 `.REG` 文件并导入：

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Internet Explorer\ActiveX Compatibility\{6BE52E1D-E586-474F-A6E2-1A85A9B4D9FB}]
"Compatibility Flags"=dword:00000400
```

--EOF--

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
