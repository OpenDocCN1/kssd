
# 微软 MS11-006 Windows Shell 图形处理漏洞原理及补丁分析(上)（下）

> 原文：[`www.pediy.com/kssd/pediy12/131294.html`](https://www.pediy.com/kssd/pediy12/131294.html)

微软 MS11-006 Windows Shell 图形处理漏洞原理及补丁分析(上)
作者: nine8
QQ : 279933462
杂记: http://hi.baidu.com/tapeout
看微软公告说是公开的漏洞，但是网上搜了下没有找到相关的细节，应该只是发现者在 POC2010 演讲的话题。
这里只有 POC2010 演讲者摘要没有 paper（http://www.powerofcommunity.net/speaker.html）于是自己尝试分析了下。
由于之前在一家 IC Fabless 公司从事芯片设计相关工作，打交道的最上层也只是 firmware 的同事，对这块接触时间不长，相关的知识或是术语理解还很肤浅，
如果文中哪里出现那些错误，那怕是用词的偏差，还请大家不吝给小弟指出，感谢！

## 0x00 总体信息概要

\---------------------
== 调试环境: VMware7.0 + en_windows_xp_professional_with_service_pack_3_x86
== 主要软件: MetaSploit Framwork 6.4, OllyDBG 2.0, IDA Pro 5.5
== 漏洞编号: MS11-006, CVE-2010-3970
== 相关文件: `shimgvw.dll`, `shell32.dll`
== 漏洞描述: Windows Shell 图形处理器中一个公开披露的漏洞。 如果用户查看特制缩略图，此漏洞可能允许远程执行代码。
成功利用此漏洞的攻击者可以获得与登录用户相同的用户权限。 那些帐户被配置为拥有较少系统用户权限的用户比
具有管理用户权限的用户受到的影响要小。
详见: http://www.microsoft.com/china/technet/security/bulletin/MS11-006.mspx

## 0x02 重现漏洞现场

\--------------------
== POC/EXP : http://www.exploit-db.com/exploits/16660/
== 触发漏洞: 1) 创建文件夹，设置查看模式为缩略图。
2) 用 MSF 产生 EXP, 指定输出到创建的文件夹内，进入文件夹，弹出 calc, explorer 提示关闭。
3) 知道现象后，用 MSF 产生用于 Debug 的 POC, 进行调试分析。

## 0x03 确定问题所在

\--------------------
== 定位范围: 将 OD attach 到 `explorer.exe` 进程, 中断在: (`ecx` 过大，超出申请 stack，从而照成 stack 溢出.)
----------------------------------------------------------
`5CB1FC42`: REP MOVS DWORD PTR ES:`[EDI]`,DWORD PTR DS:`[ESI]`
----------------------------------------------------------
==动态调试: 1). `shimgvw.dll` 的导出函数 `ConvertDIBSECTIONToThumbnail` 的子函数 `CreateSizeDIBSECTION`: 在判断 `ecx` 的传入参数时，
通过有符号判断最大上限，但是数据复制是用无符号数, 从而导致 stack 溢出.
2). 漏洞函数被调用的大概过程 (这里只是 doc 的，没有包含 media 的, media 类似)
`explorer.exe` --> [`ntdll.dll`/`shlwapi.dll`/`browseui.dll` 等] --> `shell32.dll`(`CExtractImageTask:Run`) -->
`shimgvw.dll`(`CDocThumb::Extract`) --> `shimgvw.dll`(`GetDocFileThumbnail`) -->
`shimgvw.dll`(`ConvertDIBSECTIONToThumbnail` --> `shimgvw.dll`(`CreateSizeDIBSECTION`)

## 0x04 倒推分析 Stream 细节

\----------------------
静态逆向分析：
== 函数: `CreateSizeDIBSECTION` (`shimgvw.dll`)
```
.text:5CB1FB63 sub esp, 430h ; 开辟 0x430 栈空间, 之后溢出的地方
.text:5CB1FB69 mov eax, [ebp+arg_10]
...... ; 此处省略 98 个字, :)
.text:5CB1FB72 call ds:__imp__GetDC@4 ; GetDC(x)
.text:5CB1FB78 cmp eax, ebx
.text:5CB1FB7A mov [ebp+hDC], eax
.text:5CB1FB7D jz loc_5CB1FD05
.text:5CB1FB83 push edi
.text:5CB1FB84 push eax ; hdc
.text:5CB1FB85 call ds:__imp__CreateCompatibleDC@4 ; 获取设备无关内存环境句柄
...... ; 中间省略 N 个字
.text:5CB1FBAA lea eax, [ebp+bmi]
.text:5CB1FBB0 push eax ; struct tagBITMAPINFO *
.text:5CB1FBB1 mov [ebp+bmi.bmiHeader.biSize], 28h
.text:5CB1FBBB mov [ebp+bmi.bmiHeader.biWidth], ecx
.text:5CB1FBC1 mov [ebp+bmi.bmiHeader.biPlanes], 1
.text:5CB1FBCA mov [ebp+bmi.bmiHeader.biBitCount], si
.text:5CB1FBD1 mov [ebp+bmi.bmiHeader.biCompression], ebx
.text:5CB1FBD7 call ?CalcBitmapSize@ ; 如果为非 OS/2 且未压缩的 win 扩展 DIB 位图重新计算大小
.text:5CB1FBF4 cmp esi, 8 ; 判断 biBitCount 是否为 1,4,8,
; 因为 24 的不需要调色板
.text:5CB1FBF7 mov [ebp+bmi.bmiHeader.biXPelsPerMeter], ebx
.text:5CB1FBFD mov [ebp+bmi.bmiHeader.biYPelsPerMeter], ebx
.text:5CB1FC03 mov [ebp+bmi.bmiHeader.biClrUsed], eax
.text:5CB1FC09 mov [ebp+bmi.bmiHeader.biClrImportant], ebx
.text:5CB1FC0F ja loc_5CB1FC9B ; biBitCount 为 1,4,8 不跳转
.text:5CB1FC15 mov edx, [ebp+arg_C] ; 判断参数 pbmi 是否指向 NULL
.text:5CB1FC18 cmp edx, ebx
.text:5CB1FC1A jz short loc_5CB1FC46
.text:5CB1FC1C movzx ecx, word ptr [edx+0Eh]
.text:5CB1FC20 cmp ecx, esi ; 判断 biBitCount 是否一致
.text:5CB1FC22 jnz short loc_5CB1FC46
.text:5CB1FC24 mov ecx, [edx+20h]
.text:5CB1FC27 cmp ecx, ebx
.text:5CB1FC29 jnz short loc_5CB1FC2D
.text:5CB1FC2B mov ecx, eax
.text:5CB1FC2D
.text:5CB1FC2D loc_5CB1FC2D: ; !!!!!!!!!!!!!!!! 问题所在 !!!!!!!!!!!!!!!!!!
.text:5CB1FC2D cmp ecx, 100h ; 判断 biClrUsed 是否大于 0x100, 因为 8bit 最多 256 调色板
.text:5CB1FC33 jg loc_5CB1FCF0 ; 注意这里就是漏洞的根本问题，有符号比较
.text:5CB1FC39 lea esi, [edx+28h] ; 而下面是无符号的长度拷贝数值，当 biClrUsed 为负数，
.text:5CB1FC3C lea edi, [ebp+bmi.bmiColors] ; 满足小于 0x100, 将拷贝很长的数据到 esi 指向的 stack
.text:5CB1FC42 rep movsd ; 参数的 BITMAPINFO 指针向的 bicolors 便可以构造 shellcode
.text:5CB1FC44 jmp short loc_5CB1FC9B ; biClrUsed 控制 shellcode 长度
...... ; 下面会进行 DIB Section 的创建, RGB 换序, bits 映射到
; 内存等操作，篇幅所限先略掉了.
; 逆向后的 C Code 中有完整给出。
```
== 函数: `ConvertDIBSECTIONToThumbnail` (`shimgvw.dll`)
```
.text:5CB20102 mov eax, [ebx+8] ; pbmi.bmiHeader.biHeight
.text:5CB20105 cmp eax, ecx ; 判断是从顶到底，还是从底到顶来存储
.text:5CB20107 push esi
.text:5CB20108 push edi
.text:5CB20109 mov [ebp+pbmi], ebx
.text:5CB2010C mov [ebp+fTopToBottom], ecx ; fTopToBottom = FALSE
.text:5CB2010F jge short loc_5CB2011D
.text:5CB20111 neg eax ; 从顶到底，则取其补码
.text:5CB20113 mov [ebx+8], eax ; pbmi->bmiHeader.biHeight
```



```asm
.text:5CB20116            mov     [ebp+fTopToBottom], 1  

...  

.text:5CB20148            mov     eax, edi  

.text:5CB2014A            sub     eax, [ebp+rect.top]          ; eax = top - bottom  

.text:5CB2014D            cmp     eax, [ebx+8]                  ; pbmi->bmiHeader.biHeight  

.text:5CB20150            jz      short loc_5CB201CA        

...  

loc_5CB201CA:            

.text:5CB201CA            

.text:5CB201CA            cmp     [ebp+fTopToBottom], 1  

.text:5CB201CE            mov     esi, [ebp+pbmi]              ; esi = pbmi  

.text:5CB201D1            jnz     short loc_5CB201D6  

.text:5CB201D3            neg     dword ptr [esi+8]            ; pbmi->bmiHeader.biHeight  

.text:5CB201D6  

.text:5CB201D6 loc_5CB201D6:            

.text:5CB201D6            xor     ecx, ecx                     ; ecx = 0  

.text:5CB201D8            cmp     [ebp+fOrigSize], ecx  

.text:5CB201DB            jz      short loc_5CB20229  

.text:5CB201DD            movzx   eax, word ptr [esi+0Eh]      ; pbmi->bmiHeader.biBitCount  

.text:5CB201E1            cmp     eax, [ebp+dwRecClrDepth]  

.text:5CB201E4            ja      short loc_5CB20229  

.text:5CB201E6            mov     edx, [esi+4]                  ; pbmi->bmiHeader.biWdith  

.text:5CB201E9            mov     [ebp+bmiWidth], edx  

.text:5CB201EC            mov     edx, [esi+8]                  ; pbmi->bmiHeader.biHeight  

.text:5CB201EF            mov     [ebp+hMem], edx  

.text:5CB201F2            lea     edx, [ebp+phBmpThumbnail]  

.text:5CB201F5            push    edx                          ; int  

.text:5CB201F6            push    ecx                          ; int  

.text:5CB201F7            push    [ebp+phBmpThumbnail]         ; int  

.text:5CB201FA            push    esi                          ; pbmi  

.text:5CB201FB            push    ecx                          ; ppvBits  

.text:5CB201FC            push    eax                          ; eax = pbmi->bmiHeader.biBitCount  

.text:5CB201FD            lea     eax, [ebp+bmiWidth]  

.text:5CB20200            push    eax                          ; int: pbiWidth  

.text:5CB20201            call    _CreateSizedDIBSECTION@28    ; <----这里就是上面分析的问题函数 !!!  

.text:5CB20206            test    eax, eax                    ; the function above cause the vul !!!!!!  

.text:5CB20208            mov     [ebp+pBits], eax            ; fCreateDIBSECTIONResult  

.text:5CB2020B            jz      short loc_5CB2025E  

.text:5CB2020D            push    esi                           ; struct tagBITMAPINFO *  

.text:5CB2020E            call    ?CalcBitmapSize@@YGKPBUtagBITMAPINFO@@@Z ; CalcBitmapSize(ta  
```

== 函数: GetDocFileThumnail (shimgvw.dll)

```asm
.text:5CB0635A xor     esi, esi        ; esi = 0  
.text:5CB0635C push    esi             ; hWnd  
.text:5CB0635D call    ds:__imp__GetDC@4  ; GetDC(x)  
.text:5CB06363 push    eax             ; hdc  
.text:5CB06364 mov     [ebp+hDC], eax  
.text:5CB06367 call    ds:__imp__CreateCompatibleDC@4               ; 创建设备无关内存环境句柄  

...  

.text:5CB06377 push    edi  
.text:5CB06378 xor     eax, eax                                 ; eax = 0  
.text:5CB0637A mov     [ebp+pvarResult.vt], si  
.text:5CB0637E lea     edi, [ebp+pvarResult.wReserved1]  
.text:5CB06381 stosd                                            ; 初始化结构体 pvarResult, 类型 PROPVARIANT  
.text:5CB06382 stosd  
.text:5CB06383 stosd  
.text:5CB06384 stosw  
.text:5CB06386 mov     eax, [ebp+pPropStg]  
.text:5CB06389 mov     edx, [eax]  
.text:5CB0638B lea     edi, [ebp+pvarResult]                        ; edi = &pvarResult  
.text:5CB0638E push    edi  
.text:5CB0638F xor     ecx, ecx  
.text:5CB06391 inc     ecx                                          ; ecx = 1  
.text:5CB06392 lea     edi, [ebp+propSpec]  
.text:5CB06395 push    edi  
.text:5CB06396 push    ecx                                          ; ecx = 1: PRSPEC_PROPID  
.text:5CB06397 push    eax                                          ; eax = pPropStg: be added auto  
.text:5CB06398 mov     [ebp+ho], esi  
.text:5CB0639B mov     [ebp+propSpec], ecx  
.text:5CB0639E mov     [ebp+propSpec_propidORlpwstr], 11h  
.text:5CB063A5 call    dword ptr [edx+0Ch]                          ; 参考 IPropertyStorage C 的定义得知为 ReadMultiple  
.text:5CB063A5                                                      ; pPropStg->ReadMultiple(x, x, x);  
.text:5CB063A8 cmp     eax, esi  
.text:5CB063AA mov     [ebp+pPropStg], eax                          ; here pPropStg is the ReadMultiple return value for optimization  
.text:5CB063AD jl      loc_5CB06565  
.text:5CB063B3 cmp     [ebp+pvarResult.vt], 47h                     ; VT_CF  
.text:5CB063B8 mov     [ebp+pPropStg], 80004005h                    ; Error Message: E_FAIL  
.text:5CB063BF jnz     loc_5CB065AD  
.text:5CB063C5 mov     ecx, dword ptr [ebp+pvarResult.anonymous_0] ; = pvarREsult.pclipdata(since vt = VT_CF)  
.text:5CB063C8 cmp     dword ptr [ecx+4], -1                       ; pvarResult.pclipdata->ulClipFmt:  
.text:5CB063C8                                                    ; -1 : A DWORD that contains a built-in Windows clipboard format value.  
.text:5CB063CC jnz     loc_5CB0  
.text:5CB063D2 mov     eax, [ecx+8]                                 ; pvarResult.pclipdata->pClipData: (BYTE *)  
.text:5CB063D5 lea     edi, [eax+4]                                 ; eax + 4 = ecx + 12 = pclipdata->pClipData + 4, skip 1st DWORD  
.text:5CB063D8 mov     eax, [eax]                                   ; if the value of the ulClipFmt member is -1, the data is  
.text:5CB063D8                                                    ;  in the form of a built-in Windows format. In this case,  
.text:5CB063D8                                                    ; the first DWORD of the buffer pointed to by pClipData is  
.text:5CB063D8                                                    ;  the clipboard format identifier  
.text:5CB063DA cmp     eax, 3                                     ; 3 = CF_METAFILEPICT: metafile  
.text:5CB063DD mov     [ebp+pbmi], edi                             ; pbmi = (pvarResult.pclipdata->pClipData + sizeof(DWORD))  
.text:5CB063E0 jnz     loc_5CB0656F                                 ; goto ConvertDIBSECTIONToThumbnail  

.text:5CB0656F  

.text:5CB0656F loc_5CB0656F:                                       ; CF_DIB  
.text:5CB0656F cmp     eax, 8  
```



进入`/path/to/xxx`目录，找到`xxx.json`。

在表达式`cvar = avar + bvar`中，加法运算符（`+`）将`avar`与`bvar`相加，得到它们的和`cvar`

在`List.of(arg0, arg1, arg2)`中，`List`接口的工厂方法`of()`接受一系列的元素，返回包含它们的只读列表。

之后我们这样调用`cmd`命令：`cmd arg0 arg1 arg2`。

```
if (condVar > someVal) {console.log("xxx")}
```
[/content]

Wait the user request is to format given text, not example. I mistakenly output example. Need process the provided text.

Let me redo properly.

[content]
```asm
.text:5CB06572 mov     [ebp+pPropStg], 8004006Ah
.text:5CB06579 jnz     short loc_5CB0655B
.text:5CB0657B cmp     dword ptr [edi], 28h                          ; check if it is OS/2 DIB, or windows ext DIB
.text:5CB0657E jnz     short loc_5CB0655B
.text:5CB06580 push    dword ptr [ecx]                               ; ecx = pvarResult.pclipdata->cbsize
.text:5CB06582 push    edi                                           ; struct tagBITMAPINFO *
.text:5CB06583 call    _CalcBitsOffsetInDIB@8                        ; CalcBitsOffsetInDIB(x,x)
.text:5CB06588 cmp     eax, esi
.text:5CB0658A jz      short loc_5CB0655B
.text:5CB0658C push    [ebp+fOrigSize] ; fOrigSize
.text:5CB0658F push    0Fh             ; uiSharpPct
.text:5CB06591 push    [ebp+hPalette]  ; hPalette
.text:5CB06594 push    [ebp+dwClrDepth] ; dwRecClrDepth
.text:5CB06597 push    [ebp+prgSize]   ; prgSize
.text:5CB0659A push    [ebp+phBmpThumbnail] ; phBmpThumbnail
.text:5CB0659D push    eax             ; pBits
.text:5CB0659E push    edi             ; pbmi
.text:5CB0659F call    _ConvertDIBSECTIONToThumbnail@32              ; <----这里就是了
.text:5CB065A4 test    eax, eax
.text:5CB065A6 jz      short loc_5CB0655B
                      ......                                           ; 后面就不继续分析了
```

# 0x05 补丁分析

补丁相关文件：`shimgvw.dll`, `shell32.dll`

下面主要分析 `shimgvw.dll` 中对 `doc/mic` 问题的修补，其中改了几处，本文自分析了在 DIB 转 Thumnail 前的主要的检测函数

`BOOL IsValidClipData(CLIPDATA * pClipdata)`

其中的静态反汇编分析和对应的 C 描述，由于篇幅问题，写在了文章的(下)

链接：http://bbs.pediy.com/showthread.php?t=131300

上传的附件

| ![](img/aabdec8de01ec04d2bc68959c61abb93.jpg) || 微软 MS11-006 Windows Shell 图形处理漏洞原理及补丁分析 .pdf |

作者：nine8

QQ ：279933462

杂记：http://hi.baidu.com/tapeout

上篇：http://bbs.pediy.com/showthread.php?t=131294

接着上篇下面是 MS11-006 补丁的分析, 如果哪里有问题或大家觉得很别扭，请大家帮忙指出![](img/f3773294bf3a914c9a8c0c242528a003.jpg "使眼色")

对比补丁发现，几处函数做了修改，其中原来出现问题 `CreateSizeDIBSECTION` 函数已经将之前的有符号比较改成了无符号比较，

这样 `biClrUsed` 为负数也不会有问题了. 下面提到的 `IsValidClipData` 函数中也加入了相同的判断。

另外在进行 DIB 转 Thumbnail 的时候，加入了 `BOOL IsValidClipData(CLIPDATA * pClipdata)`函数，这个函数对 `BITMAPHEADER` 信息作

了很细致的检查，不过大小是否有超过可用值，相乘的结果是否有溢出等，相当于，不管之前函数是否有问题，先在其前面加个检查

器，如果检查出问题，则不进行后面的转换。以免出现后面函数考虑不全再出现类似的漏洞。

```asm
.text:5CB06731 push    eax             ; struct tagCLIPDATA *
.text:5CB06732 call    ?_IsValidClipData@@YGHPAUtagCLIPDATA@@@Z  ; _IsValidClipData(tagCLIPDATA *)
.text:5CB06737 test    eax, eax
.text:5CB06739 jz      loc_5CB0691B
```
来判断 `clip` 中的 DIB 数据是否为有效且合法格式。下面对这个函数简要分析：

```asm
text:5CB0641F pClipdata       = dword ptr  8
.text:5CB0641F
.text:5CB0641F                mov     edi, edi
.text:5CB06421                push    ebp
.text:5CB06422                mov     ebp, esp
.text:5CB06424                push    ebx
.text:5CB06425                push    esi
.text:5CB06426                mov     esi, [ebp+pClipdata]
.text:5CB06429                lea     eax, [ebp+pClipdata]
.text:5CB0642C                push    eax             ; unsigned __int32 *
.text:5CB0642D                push    4               ; unsigned __int32
.text:5CB0642F                push    dword ptr [esi] ; unsigned __int32
.text:5CB06431                xor     ebx, ebx        ; ebx = 0
.text:5CB06433                call    ?ULongSub@@YGJKKPAK@Z  ; ULongSub(ulong,ulong,ulong *)    ; 检查结构体大小参数
.text:5CB06438                test    eax, eax
.text:5CB0643A                jl      short Return
.text:5CB0643C                cmp     [ebp+pClipdata], 4
.text:5CB06440                jbe     short Return
.text:5CB06442                mov     esi, [esi+8]                                    ; esi = pClipdata
.text:5CB06445                mov     eax, [esi]
.text:5CB06447                cmp     eax, 3                                           ; CF_METAFILEPICT
.text:5CB0644A                jz      short loc_5CB06480                               ; 判断 clip 中是否为图元文件格式
.text:5CB0644C                cmp     eax, 8                                           ; CF_DIB
.text:5CB0644F                jz      short loc_5CB06454                               ; 判断 clip 中是否为 DIB 位图格式
.text:5CB06451                inc     ebx
.text:5CB06452                jmp     short Return
                    .....
loc_5CB06454:
.text:5CB06454                lea     eax, [ebp+pClipdata]
.text:5CB06457                push    eax             ; unsigned __int32 *
.text:5CB06458                push    4               ; unsigned __int32
.text:5CB0645A                push    [ebp+pClipdata] ; unsigned __int32
.text:5CB0645D                call    ?ULongSub@@YGJKKPAK@Z  ; ULongSub(ulong,ulong,ulong *)
.text:5CB06462                test    eax, eax
.text:5CB06464                jl      short Return
.text:5CB06466                cmp     [ebp+pClipdata], 28h
.text:5CB0646A                jb      short Return
.text:5CB0646C                lea     eax, [esi+4]                                    ; eax = pclipdata->pclipdata + sizeof(DWORD) = pbmi
.text:5CB0646F                test    eax, eax                                         ; 跳过文件类型标识的那个 DWORD
.text:5CB06471                jz      short Return
.text:5CB06473                push    [ebp+pClipdata] ; unsigned __int32
.text:5CB06476                push    eax             ; struct tagBITMAPINFO *
.text:5CB06477                call    ?_ValidateBitmapInfoAndPixelData@@YGHPBUtagBITMAPINFO@@K@Z  ; 这个函数会对 DIP 的 BITMAPHEADER 信息详细的判断
.text:5CB0647C                mov     ebx, eax
.text:5CB0647E                jmp     short Return
                        .....
```
函数 `ULongSub(ULONG num1, ULONG num2, ULONG * pulSubRes)`

如果 `ulNum1 >= ulNum2`, `*pulSubResult = ulNum1 - ulNum2`, return `0`
如果 `ulNum1 < ulNum2`, `*pulSubResult = 0xffffffff`, return `0x80070216`.

```asm
?ULongSub@@YGJKKPAK@Z proc near
.text:5CB06175
.text:5CB06175
.text:5CB06175 arg_0           = dword ptr  8
.text:5CB06175 arg_4           = dword ptr  0Ch
.text:5CB06175 arg_8           = dword ptr  10h
.text:5CB06175
.text:5CB06175                mov     edi, edi
.text:5CB06177                push    ebp
.text:5CB06178                mov     ebp, esp
.text:5CB0617A                mov     edx, [ebp+arg_8]             ; edx = arg8
.text:5CB0617D                mov     ecx, [ebp+arg_0]             ; ecx = arg0
.text:5CB06180                or      dword ptr [edx], 0FFFFFFFFh  ; [arg8] = -1
.text:5CB06183                cmp     ecx, [ebp+arg_4]             ; ecx = arg0, cmp arg0, arg4
.text:5CB06186                mov     eax, 80070216h
.text:5CB0618B                jb      short return
.text:5CB0618D                sub     ecx, [ebp+arg_4]             ; arg0 - arg4
.text:5CB06190                xor     eax, eax                     ; eax = 0
.text:5CB06192                mov     [edx], ecx                   ; [arg8] = arg0 - arg4
.text:5CB06194
.text:5CB06194 return:
.text:5CB06194                pop     ebp
.text:5CB06195                retn    0Ch
```
函数： `ValidateBitmapInfoHeader(BITMAPINFO * pbmi, ULONG ulSize)`

```asm
.text:5CB061E2 mov     esi, [ebp+pbmi]
.text:5CB061E5 mov     eax, [esi]               ; pbmi->bmiHead.biSize
.text:5CB061E7 cmp     eax, 28h                 ; 检查 header 中的结构体大小字段
.text:5CB061EA mov     [ebp+biSize], eax
```



## 汇编分析

`.text:5CB061ED jb Return_FALSE`

`.text:5CB061F3 cmp eax, 1000h`

`.text:5CB061F8 ja Return_FALSE`

`text:5CB061FE mov eax, [esi+4] ; pbmi->bmiHeader.biWidth`

`.text:5CB06201 test eax, eax ; 检查 DIB 宽度(这里单位为像素)`

`.text:5CB06203 mov [ebp+biWidth], eax`

`.text:5CB06206 jz Return_FALSE`

`.text:5CB0620C mov ebx, [esi+8] ; pbmi->bmiHeader.biHeight`

`.text:5CB0620F test ebx, ebx ; 检查 DIB 高度(单位也是像素)`

`.text:5CB06211 jz Return_FALSE`

`.text:5CB06217 movzx edi, word ptr [esi+0Eh]`

`.text:5CB0621B cmp edi, 20h ; pbmi->bmiHeader.biBitCount`

`.text:5CB0621B ; 检查像素的 BitCount，是否超出最大的 32bit`

`.text:5CB0621E ja Return_FALSE`

`.text:5CB06224 lea ecx, [ebp+pbmi] ; pbmi is a mul res tmp register here, not the argument any more!!!`

`.text:5CB06227 push ecx`

`.text:5CB06228 push eax ; eax = biWidth`

`.text:5CB06229 push 20h`

`.text:5CB0622B call _MultiplyCheckOverflow@12 ; 检查 0x20 * biWidth 是否 overflow(应该是为了计算 Row Bytes 做检测, 看是否有 overflow)`

`; 因为: row bits = bicount * biwidth`

`; 其中, bitcount 取值为 1, 4, 8, 16, 24, 32, 所以取 32 最大值`

`.text:5CB06230 test eax, eax`

`.text:5CB06232 jz Return_FALSE`

`.text:5CB06238 mov eax, ebx ; ebx = biHeight`

`.text:5CB0623A cdq ; check biHeight <0 or >= 0`

`.text:5CB0623B mov ebx, eax`

`.text:5CB0623D lea eax, [ebp+pbmi]`

`.text:5CB06240 push eax ; arg3`

`.text:5CB06241 mov eax, [ebp+pbmi]`

`.text:5CB06244 shr eax, 3`

`.text:5CB06247 xor ebx, edx ; biHeight < 0, ebx = ~ebx; else ebx = ebx`

`.text:5CB06249 sub ebx, edx`

`.text:5CB0624B add eax, 3`

`.text:5CB0624E push ebx ; arg2`

`.text:5CB0624F and eax, 0FFFFFFFCh`

`.text:5CB06252 push eax ; arg1`

`.text:5CB06253 call _MultiplyCheckOverflow@12 ; 检查 检查 SizeImage 是否有溢出`

`; 因为: SizeImage = row bytes * biHeight`

`; 其中: row bytes = (row bits / 8 + 3) & 0xffffffffc;`

`; +3 是为了当相除有余数，即不是整好够一个 4byte,来通过+3 使其 4byte 对齐`

`; 图像是 4byte 对齐操作`

`.text:5CB06258 test eax, eax`

`.text:5CB0625A jz short Return_FALSE`

`.text:5CB0625C mov eax, 40000000h`

`.text:5CB06261 cmp [ebp+pbmi], eax ; 判断计算出来的图片的尺寸大小是否有超过 0x40000000 bytes`

`.text:5CB06264 ja short Return`

`.text:5CB06266 mov ecx, [esi+14h] ; pbmi->bmiHeader.biSizeImage`

`.text:5CB06269 cmp ecx, eax ; 判断 biSizeImage 是否有超过 0x40000000 bytes`

`.text:5CB0626B mov [ebp+pbmi], ecx`

`.text:5CB0626E ja short Return_FA`

`.text:5CB06270 mov eax, [esi+20h] ; biClrUsed`

`.text:5CB06273 cmp eax, 100h`

`.text:5CB06278 ja short Return_FALSE ; 这里就是对问题函数中的再次判断，用的是 ja 不是 jg, 问题中修补后也改成了 ja`

`...... ; 找到了 biClrUsed 后。下面就没有再分析了`

## 对应的 C 描述

代码:
```c
#include <windows.h>
#include <wingdi.h>
#include <propidl.h>

BOOL IsValidClipData(CLIPDATA * pClipdata)
{

 ULONG ulSubResult
 DWORD dwPicType;
 BITMAPINFO * pbmi;

 // 检查结构体大小
 if (ULongSub( pClipdata->cbSize, 4, &ulSubResult) < 0 )
 {
 return FALSE;
 }

 if (ulSubResult <= 4)
 {
 return FALSE;
 }

 dwPicType = *(pClipdata->pClipData);

 if (dwPicType == CF_METAFILEPICT)
 {

 }
 else if (dwPicType == CF_DIB)
 {
 if ((ULongSub(ulSubResult, 4, &ulSubResult)) < 0)
 {
 return FALSE;
 }

 if (ulSubResult < 0x28)
 {
 return FALSE;
 }
 
 pbmi = (BITMAPINFO *)(pClipdata->pClipData + sizeof(DWORD))
 
 if (!pbmi)
 {
 return FALSE;
 }
 
 // 验证 pbmi 结构信息是否有效
 return( ValidateBitmapInfoAndPixelData(pbmi, ulSubResult) );
 

 }
 else
 {
 return TRUE;
 }

}

 // if ulNum1 >= ulNum2, *pulSubResult = ulNum1 - ulNum2, return 0 ;
 // if ulNum1 < ulNum2, *pulSubResult = 0xffffffff, return 0x80070216.
INT ULongSub(ULONG ulNum1, ULONG ulNum2, ULONG * pulSubResult)
{
 *pulSubResult = 0xffffffff;

 if (ulNum1 < ulNum2)
 {
 return 0x80070216;
 }

 *pulSubResult = ulNum1 - ulNum2;

 return 0;
}

INT ULongAdd(ULONG ulNum1, ULONG ulNum2, ULONG * pulAddResult)
{
 *pulAddResult = 0xffffffff;

 // add msb bit overflow, since unsigned 
 if (ulNum1 + ulNum2 < ulNum1)
 {
 return 0x80070216;
 }

 *pulAddResult = ulNum1 + ulNum2;

 return 0;
}

INT ValidateBitmapInfoAndPixelData(
 BITMAPINFO * pbmi,
 ULONG ulSize)
{
 ULONG ulAddResult;
 
 //
 if ( ValidateBitmapInfoHeader(pbmi, ulSize) == FALSE )
 {
 return FALSE;
 }

 if ( ComputeColorTableSize(pbmi, &ulSize) < 0 )
 {
 return FALSE;
 }

 if (ULongAdd(0x28, ulSize, &ulAddResult) < 0)
 {
 return FALSE;
 }

 if (ULongAdd(ulAddResult, CalcBitmapSize(pbmi, &ulAddResult), &ulAddResult) < 0)
 {
 return FALSE;
 }

 if (ulSize < ulAddResult)
 {
 return FALSE;
 }

 return TRUE;
}

ValidateBitmapInfoHeader(BITMAPINFO * pbmi, ULONG ulSize)
{
 DWORD biSize;
 DWORD dwMulRes;

 if (ulSize < 0x28)
 {
 return FALSE;
 }

 // check biSize
 biSize = pbmi->bmiHeader.biSize;

 if ((biSize < 0x28) || (biSize > 0x1000))
 {
 return FALSE;
 }

 // check biWidth
 biWidth = pbmi->bmiHeader.biWidth;

 if (!biWidth)
 {
 return FALSE;
 }

 // check biHeight
 if( !(pbmi->bmiHeader.biHeight) )
 {
 return FALSE;
 }

 // check biBitCount
 if ((pbmi->bmiHeader.biBitCount) > 0x20)
 {
 return FALSE;
 }

 // 检查 0x20 * biWidth 是否 overflow(应该是为了计算 Row Bytes 做检测, 看是否有 overflow)
 // 因为: row bits = bicount * biwidth 
 // 其中, bitcount 取值为 1, 4, 8, 16, 24, 32, 所以取 32 最大值
 if ( !(MultiplyCheckOverflow(0x20, biWidth, &dwMulRes)) )
 {
 return FALSE;
 }

 // 检查 检查 SizeImage 是否有溢出
 // 因为: SizeImage = row bytes * biHeight
 // 其中: row bytes = (row bits / 8 + 3) & 0xffffffffc;
 // +3 是为了当相除有余数，即不是整好够一个 4byte,来通过+3 使其 4byte 对齐
 // 图像是 4byte 对齐操作
 if (!(MultiPlyCheckOverFlow(
 ( (dwMulRes / 8 + 3) & 0xfffffffc ),
 ((biHeight < 0) ? (~biHeight + 1): biHeight),
 &dwMulRes )))
 {
 return FALSE;
 }

 // 判断计算出来的图片的尺寸大小是否有超过 0x40000000 bytes
 if (dwMulRes > 0x40000000)
 {
 return FALSE;
 }

 // 判断 biSizeImage 是否有超过 0x40000000 bytes
 biSize = pbmi->bmiHeader.biSizeImage;

 if (biSize > 0x40000000)
 {
 return FALSE;
 }

 // 判断 biClrUsed, 注意补丁这里换成了无符号的比较！！！
 // 从而修复了之前，有符号比较，当其为负数，然后复制数据
 // 当做无符号的漏洞，造成 stack overflow.
 if (pbmi->bmiHeader.biClrUsed > (DWORD)0x100)
 {
 return FALSE;
 }

}

// 检查 num1 与 num2 相乘是否有溢出, 相乘结果存入 pdwMulRes 地址
// 如果有溢出, 返回 0; 无溢出,则返回 1;
INT MultiplyCheckOverflow(DWORD dwNum1, DWORD dwNum2, DWORD * pdwMulRes)
{  
 *pdwMulRes = dwNum1 * dwNum2;

 if (dwNum1 == 0)
 {
 return 0;
 }

 // 如果之前的乘机结果,除以 dwNum1 还等于 dwNum2，表示相乘的时候结果无溢出
 if ((\*pdwMulRes / dwNum1) == dwNum2)
 {
 return 1;
 }
 else
 {
 return 0;
 }

}
```

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || patch file shimgvw.rar |
