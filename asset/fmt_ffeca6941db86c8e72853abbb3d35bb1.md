
# 视维视频会议 5.1 体验版

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-127.htm`](https://www.pediy.com/kssd/pediy08/pediy8-127.htm)

## 说明

有朋友要找个视频会议系统，找了几个试用一下。看看这个好像还不错。但是只支持一个用户，厂家说可以免费注册，有可能是注册信息随便乱写的，没有收到注册码，于是就自己看看了。这里主要是给大家看看程序算法的实现。请支持国产软件。

- 系统：`windows2003`
- 使用工具：`OllyDbg 1.1`

### 1，注册码生成

进入服务器的管理程序，在许可管理里，两个文本框，一个是机器码，一个是注册码。

随便添入一个字符串后会提示成功，重新启动服务器。然后该不好使还是不好使。这里跟踪一下，看如何处理的。

`bp GetVolumeInformationA`

然后打开许可界面，程序就断下了：

```
0040EE27   .  6A 50           push    50                         ; /pFileSystemNameSize = 00000050
0040EE29   .  52             push    edx                        ; |pFileSystemNameBuffer
0040EE2A   .  50             push    eax                        ; |pFileSystemFlags
0040EE2B   .  8D5424 24      lea     edx, [esp+24]               ; |
0040EE2F   .  51             push    ecx                        ; |pMaxFilenameLength
0040EE30   .  52             push    edx                        ; |pVolumeSerialNumber
0040EE31   .  8D8424 880300>lea     eax, [esp+388]              ; |
0040EE38   .  68 04010000    push    104                        ; |MaxVolumeNameSize = 104 (260.)
0040EE3D   .  8D8C24 740100>lea     ecx, [esp+174]              ; |
0040EE44   .  50             push    eax                        ; |VolumeNameBuffer
0040EE45   .  51             push    ecx                        ; |RootPathName
0040EE46   .  FF15 80704100  call    [<&KERNEL32.GetVolumeInformationA>]      ; \GetVolumeInformationA ;取 C 盘卷标
0040EE4C   .  3BC3           cmp     eax, ebx
0040EE4E   .  74 2F         je      short 0040EE7F
0040EE50   .  395C24 18      cmp     [esp+18], ebx
0040EE54   .  75 15         jnz     short 0040EE6B
0040EE56   .  8D5424 18      lea     edx, [esp+18]
0040EE5A   .  8D8424 5C0100>lea     eax, [esp+15C]
0040EE61   .  52             push    edx
0040EE62   .  50             push    eax
0040EE63   .  E8 38FBFFFF    call    0040E9A0
0040EE68   .  83C4 08        add     esp, 8
0040EE6B   >  8B4C24 18      mov     ecx, [esp+18]
0040EE6F   .  8D9424 940000>lea     edx, [esp+94]
0040EE76   .  51             push    ecx
0040EE77   .  68 4CD54100   push    0041D54C                     ;  %x
0040EE7C   .  52             push    edx
0040EE7D   .  EB 14         jmp     short 0040EE93
0040EE7F   >  FF15 F4704100 call    [<&KERNEL32.GetLastError>]           ; [GetLastError
0040EE85   .  50             push    eax
0040EE86   .  8D8424 980000>lea     eax, [esp+98]
0040EE8D   .  68 4CD54100   push    0041D54C                     ;  %x
0040EE92   .  50             push    eax
0040EE93   >  FFD6          call    esi                      ;sprintf 把 C 盘卷标转换成 16 进字符串
0040EE95   .  83C4 0C       add     esp, 0C
0040EE98   .  8D4C24 24     lea     ecx, [esp+24]
0040EE9C   .  8D9424 5C0100>lea     edx, [esp+15C]
0040EEA3   .  C74424 24 C70>mov     dword ptr [esp+24], 0C7
0040EEAB   .  51             push    ecx                        ; /pBufferSize
0040EEAC   .  52             push    edx                        ; |Buffer
0040EEAD   .  FF15 84704100 call    [<&KERNEL32.GetComputerNameA>]           ; \GetComputerNameA  ;取得主机名称
0040EEB3   .  8DBC24 5C0100>lea     edi, [esp+15C]
0040EEBA   .  83C9 FF      or      ecx, FFFFFFFF
0040EEBD   .  33C0         xor     eax, eax
0040EEBF   .  8D9424 940000>lea     edx, [esp+94]
0040EEC6   .  F2:AE       repne   scas byte ptr es:[edi]              ;以下几行代码功能是把 C 盘卷标
0040EEC8   .  F7D1       not     ecx
0040EECA   .  2BF9       sub     edi, ecx                     ;连接在一起
0040EECC   .  8BF7       mov     esi, edi
0040EECE   .  8BFA       mov     edi, edx
0040EED0   .  8BD1       mov     edx, ecx
0040EED2   .  83C9 FF   or      ecx, FFFFFFFF
0040EED5   .  F2:AE       repne   scas byte ptr es:[edi]
0040EED7   .  8BCA       mov     ecx, edx
0040EED9   .  4F         dec     edi
0040EEDA   .  C1E9 02    shr     ecx, 2
0040EEDD   .  F3:A5     rep     movs dword ptr es:[edi], dword ptr [esi]   ;
0040EEDF   .  8BCA       mov     ecx, edx
0040EEE1   .  8D8424 940000>lea     eax, [esp+94]
0040EEE8   .  83E1 03    and     ecx, 3
0040EEEB   .  50         push    eax                          ;这里是保存连接后字符串的地址
0040EEEC   .  F3:A4     rep     movs byte ptr es:[edi], byte ptr [esi]
0040EEEE   .  8BCD       mov     ecx, ebp
0040EEF0   .  E8 7B020000   call    0040F170                  ;进行运算
0040EEF5   .  8DBC24 940000>lea     edi, [esp+94]
0040EEFC   .  83C9 FF   or      ecx, FFFFFFFF
0040EEFF   .  33C0     xor     eax, eax
0040EF01   .  8D55 68   lea     edx, [ebp+68]
0040EF04   .  F2:AE   repne   scas byte ptr es:[edi]
0040EF06   .  F7D1   not     ecx
0040EF08   .  2BF9   sub     edi, ecx
0040EF0A   .  8BC1   mov     eax, ecx
0040EF0C   .  8BF7   mov     esi, edi
0040EF0E   .  8BFA   mov     edi, edx
0040EF10   .  C1E9 02  shr     ecx, 2
0040EF13   .  F3:A5 rep     movs dword ptr es:[edi], dword ptr [esi]
0040EF15   .  8BC8   mov     ecx, eax
0040EF17   .  83E1 03  and     ecx, 3
0040EF1A   .  F3:A4 rep     movs byte ptr es:[edi], byte ptr [esi]
0040EF1C   .  8D8C24 940000>lea     ecx, [esp+94]
0040EF23   .  51     push    ecx
0040EF24   .  68 07040000 push    407
0040EF29   .  8BCD   mov     ecx, ebp
0040EF2B   .  E8 60540000 call    <jmp.&MFC42.#5953_CWnd::SetDlgItemTextA>      ;运算完后写到文本框里  
0040EF30   .  68 48D54100 push    0041D548                     ;  3.0%x
0040EF35   .  68 09040000 push    409
0040EF3A   .  8BCD   mov     ecx, ebp
0040EF3C   .  E8 4F540000 call    <jmp.&MFC42.#5953_CWnd::SetDlgItemTextA>
0040EF41   .  8B15 54D64100 mov     edx, [41D654]
0040EF47   .  B9 19000000 mov     ecx, 19
0040EF4C   .  33C0  xor     eax, eax
0040EF4E   .  8D7C24 30  lea     edi, [esp+30]
0040EF52   .  F3:AB rep     stos dword ptr es:[edi]
0040EF54   .  52     push    edx                          ; /IniFileName => "C:\WINNT\lrmcu.ini"
0040EF55   .  8D4424 34  lea     eax, [esp+34]                ; |
0040EF59   .  6A 64  push    64                          ; |BufSize = 64 (100.)
0040EF5B   .  50     push    eax                          ; |ReturnBuffer
0040EF5C   .  68 D8D54100 push    0041D5D8                     ; |Default = ""
0040EF61   .  68 40D54100 push    0041D540                     ; |code
0040EF66   .  68 E4D34100 push    0041D3E4                     ; |regcode   ;这里是注册码保存的地方。
0040EF6B   .  FF15 14714100 call    [<&KERNEL32.GetPrivateProfileStringA>]      ; \GetPrivateProfileStringA  
```

接下来后面有 `WritetPrivateProfileStringA`，可以看出，程序只是把注册码保存起来，并不做验证，验证是由服务程序启动时候检验的。

在看看 `0040F170` 处:

```
0040F170  /$  56           push    esi
0040F171  |.  8B7424 08    mov     esi, [esp+8]          ;取参数
0040F175  |.  57           push    edi
0040F176  |.  8BFE         mov     edi, esi
0040F178  |.  83C9 FF      or      ecx, FFFFFFFF
0040F17B  |.  33C0         xor     eax, eax
0040F17D  |.  33D2         xor     edx, edx
0040F17F  |.  F2:AE        repne   scas byte ptr es:[edi]  ;取参数长度
0040F181  |.  F7D1         not     ecx
```
