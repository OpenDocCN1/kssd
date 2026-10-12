
# RealPlayer 6.0.10 到 6.0.14 整数溢出分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-327.htm`](https://www.pediy.com/kssd/pediy09/pediy09-327.htm)

> RealPlayer 6.0.10 到 6.0.14 整数溢出分析

今天早上茄子就告诉我 `cnbeta` 被挂马了，使用最近出现的各种漏洞。

其中比较有意思的是，一个脚本中使用没有见过的 `realplayer` 溢出。![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!")

先从 `realplayer` 目录 `rpplugins\\ierpplug.dll` 中的 `IIERPCtl.Import` 开始讲起，

下面是其声明

`HRESULT Import(`
`                        [in] BSTR file,`
`                        [in, optional, defaultvalue("")] BSTR playlist,`
`                        [in, optional, defaultvalue("")] BSTR clipInfo,`
`                        [in, optional, defaultvalue(0)] long bPlayFile,`
`                        [in, optional, defaultvalue(0)] long bCopyToMyMusic,`
`                        [out, retval] long* pVal);`

代码:

```
.text:65E91640  ; int __stdcall IIERPCtl_Import(int,int This,int file,int playlist,int clipInfo,int bPlayFile,int bCopyToMyMusic)
.text:65E91640 IIERPCtl_Import proc near               ; DATA XREF: .rdata:65EB545Co
.text:65E91640                                         ; .rdata:65EB54E4o
.text:65E91640
.text:65E91640 var_C           = dword ptr -0Ch
.text:65E91640 This            = dword ptr  0Ch
.text:65E91640 file            = dword ptr  10h
.text:65E91640 playlist        = dword ptr  14h
.text:65E91640 clipInfo        = dword ptr  18h
.text:65E91640 bPlayFile       = dword ptr  1Ch
.text:65E91640 bCopyToMyMusic  = dword ptr  20h
.text:65E91640                  push    ebp
.text:65E91641                  mov     ebp, esp
.text:65E91643                  push    ebx
.text:65E91644                  mov     ebx, [ebp+playlist] ;关键参数
.text:65E91647                  push    esi
.text:65E91648                  push    edi
.text:65E91649                  test    ebx, ebx
.text:65E9164B                  jnz     short loc_65E91652
.text:65E9164B
.text:65E9164D                  mov     [ebp+playlist], ebx
.text:65E91650                  jmp     short loc_65E91684
.text:65E91650
.text:65E91652 ; ---------------------------------------------------------------------------
.text:65E91652
.text:65E91652 loc_65E91652:                           ; CODE XREF: IIERPCtl_Import+Bj
.text:65E91652                  push    ebx             ; lpString
.text:65E91653                  call    ds:lstrlenW
.text:65E91659                  lea     edi, [eax+eax+2]
.text:65E9165D                  mov     eax, edi
.text:65E9165F                  add     eax, 3
.text:65E91662                  and     al, 0FCh
.text:65E91664                  call    __alloca_probe
.text:65E91664
.text:65E91669                  mov     esi, esp
.text:65E9166B                  push    0
.text:65E9166D                  push    0
.text:65E9166F                  push    edi
.text:65E91670                  push    esi
.text:65E91671                  push    0FFFFFFFFh
.text:65E91673                  push    ebx
.text:65E91674                  push    0
.text:65E91676                  push    0
.text:65E91678                  mov     byte ptr [esi], 0
.text:65E9167B                  call    ds:WideCharToMultiByte
.text:65E91681                  mov     [ebp+playlist], esi
.text:65E91681
.text:65E91684
.text:65E91684 loc_65E91684:                           ; CODE XREF: IIERPCtl_Import+10j
.text:65E91684                  mov     ebx, [ebp+file]
.text:65E91687                  test    ebx, ebx
.text:65E91689                  jnz     short loc_65E91690
.text:65E91689
.text:65E9168B                  mov     [ebp+file], ebx
.text:65E9168E                  jmp     short loc_65E916C2
.text:65E9168E
.text:65E91690 ; ---------------------------------------------------------------------------
.text:65E91690
.text:65E91690 loc_65E91690:                           ; CODE XREF: IIERPCtl_Import+49j
.text:65E91690                  push    ebx             ; lpString
.text:65E91691                  call    ds:lstrlenW
.text:65E91697                  lea     edi, [eax+eax+2]
.text:65E9169B                  mov     eax, edi
.text:65E9169D                  add     eax, 3
.text:65E916A0                  and     al, 0FCh
.text:65E916A2                  call    __alloca_probe
.text:65E916A2
.text:65E916A7                  mov     esi, esp
.text:65E916A9                  push    0
.text:65E916AB                  push    0
.text:65E916AD                  push    edi
.text:65E916AE                  push    esi
.text:65E916AF                  push    0FFFFFFFFh
.text:65E916B1                  push    ebx
.text:65E916B2                  push    0
.text:65E916B4                  push    0
.text:65E916B6                  mov     byte ptr [esi], 0
.text:65E916B9                  call    ds:WideCharToMultiByte
.text:65E916BF                  mov     [ebp+file], esi
.text:65E916BF
.text:65E916C2
.text:65E916C2 loc_65E916C2:                           ; CODE XREF: IIERPCtl_Import+4Ej
.text:65E916C2                  mov     ebx, [ebp+This]
.text:65E916C5                  test    ebx, ebx
.text:65E916C7                  jnz     short loc_65E916CD
.text:65E916C7
.text:65E916C9                  xor     esi, esi
.text:65E916CB                  jmp     short loc_65E916FC
.text:65E916CB
.text:65E916CD ; ---------------------------------------------------------------------------
.text:65E916CD
.text:65E916CD loc_65E916CD:                           ; CODE XREF: IIERPCtl_Import+87j
.text:65E916CD                  push    ebx             ; lpString
.text:65E916CE                  call    ds:lstrlenW
.text:65E916D4                  lea     edi, [eax+eax+2]
.text:65E916D8                  mov     eax, edi
.text:65E916DA                  add     eax, 3
.text:65E916DD                  and     al, 0FCh
.text:65E916DF                  call    __alloca_probe
.text:65E916DF
.text:65E916E4                  mov     esi, esp
.text:65E916E6                  push    0
.text:65E916E8                  push    0
.text:65E916EA                  push    edi
.text:65E916EB                  push    esi
.text:65E916EC                  push    0FFFFFFFFh
.text:65E916EE                  push    ebx
.text:65E916EF                  push    0
.text:65E916F1                  push    0
.text:65E916F3                  mov     byte ptr [esi], 0
.text:65E916F6                  call    ds:WideCharToMultiByte
.text:65E916F6
.text:65E916FC
.text:65E916FC loc_65E916FC:                           ; CODE XREF: IIERPCtl_Import+8Bj
.text:65E916FC                  mov     eax, [ebp+bCopyToMyMusic]
.text:65E916FF                  mov     ecx, [ebp+bPlayFile]
.text:65E91702                  mov     edx, [ebp+clipInfo]
.text:65E91705                  push    eax             ; int
.text:65E91706                  mov     eax, [ebp+playlist]
.text:65E91709                  push    ecx             ; int
.text:65E9170A                  mov     ecx, [ebp+file]
.text:65E9170D                  push    edx             ; int
.text:65E9170E                  push    eax             ; int
.text:65E9170F                  push    ecx             ; file
.text:65E91710                  push    esi             ; lpString
.text:65E91711                  call    sub_65E9C860   ;进入
```

代码:
