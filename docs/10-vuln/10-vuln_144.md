
# MS11-081 中 IE9 导致的一个 CHM 漏洞简单分析

> 原文：[`www.pediy.com/kssd/pediy12/141323.html`](https://www.pediy.com/kssd/pediy12/141323.html)

这个漏洞只存在于 IE9，会导致无法下载 CHM 中内嵌的文件，在没有安装 KB2586448 更新的机器上测试

首先新建一个 `1.html` 文件，内容如下

代码:

```
<a href = "1.rar">1.rar</a>
```

然后和一个 `1.rar` 文件用 HTML Help Workshop 一起编译成 CHM

打开 CHM 点击 `1.rar` 链接，也可以在 IE 地址栏输入 `mk:@MSITStore:XXX(具体路径)\\1.chm::/1.rar`，报错调试

停在 `CDownloadUtilities::MarshalBindContextToStream` 函数内，这个函数为下载 `1.rar` 文件做一些准备工作

声明如下

`HRESULT CDownloadUtilities::MarshalBindContextToStream(CInterThreadMarshal **ppITM<eax>, CDownloadThreadParam *pDTP, wchar_t *szURL, IBindCtx *pBC)`

简单分析下其代码

代码:

```
CDownloadUtilities::MarshalBindContextToStream(CInterThreadMarshal **ppITM<eax>, CDownloadThreadParam *pDTP, wchar_t *szURL, IBindCtx *pBC):
7003D23A  mov         edi,edi  
7003D23C  push        ebp  
7003D23D  mov         ebp,esp  
7003D23F  sub         esp,0Ch  ;HRESULT hr<[ebp-4h]>, IUnknown *pUnk<[ebp-8h]>, IEUserBroker *pIEUB<[ebp-0Ch]>
7003D242  push        ebx  
7003D243  push        esi  
7003D244  push        edi  
7003D245  lea         edx,[ebp-8]  ;edx = &pUnk
7003D248  push        edx  ;&pUnk
7003D249  mov         edi,eax  ;edi = ppITM
7003D24B  mov         eax,dword ptr [ebp+10h]  ;eax = pBC
7003D24E  mov         ecx,dword ptr [eax]  ;ecx = pBC->lpVtbl
7003D250  push        offset __GUID_0000000e_0000_0000_c000_000000000046 (6FF65DA8h)  ;&IID_IUnknown
7003D255  push        eax  ;pBC
7003D256  call        dword ptr [ecx]  ;eax = pBC->lpVtbl->QueryInterface(pBC, &IID_IUnknown, &pUnk)
7003D258  xor         ebx,ebx  
7003D25A  mov         dword ptr [ebp-4],eax  ;hr = eax
7003D25D  cmp         eax,ebx  
7003D25F  jl          CDownloadUtilities::MarshalBindContextToStream+0C6h (7003D300h)  ;if (FAILED(hr)) goto 7003D300h
7003D265  push        3  
7003D267  push        offset string L"mk:" (70019BB4h)  
7003D26C  push        dword ptr [ebp+0Ch]  
7003D26F  call        dword ptr [__imp__wcsnicmp (6FE91364h)]  
7003D275  add         esp,0Ch  
7003D278  test        eax,eax  
7003D27A  jne         CDownloadUtilities::MarshalBindContextToStream+74h (7003D2AEh)  ;if (_wcsnicmp(szURL, L"mk:", 3)) goto 7003D2AEh
7003D27C  mov         eax,dword ptr [ebp-8]  ;eax = pUnk
7003D27F  mov         ecx,dword ptr [eax]  ;ecx = pUnk->lpVtbl
7003D281  push        eax  ;pUnk
7003D282  call        dword ptr [ecx+8]  ;pUnk->lpVtbl->Release(pUnk)
7003D285  push        ebx  ;0
7003D286  lea         eax,[ebp+10h]  ;eax = &pBC
7003D289  push        eax  ;&pBC
7003D28A  push        ebx  ;0
7003D28B  push        ebx  ;0
7003D28C  push        ebx  ;0
7003D28D  push        ebx  ;0
7003D28E  mov         dword ptr [ebp-8],ebx  ;pUnk = 0
7003D291  call        dword ptr [__imp__CreateAsyncBindCtxEx@24 (7023B14Ch)]  ;eax = CreateAsyncBindCtxEx(0, 0, 0, 0, &pBC, 0)
7003D297  mov         esi,dword ptr [edi]  ;esi = *ppITM
7003D299  mov         dword ptr [ebp-4],eax  ;hr = eax
7003D29C  cmp         esi,ebx  
7003D29E  je          CDownloadUtilities::MarshalBindContextToStream+74h (7003D2AEh)  ;if (*ppITM == 0) goto 7003D2AEh
7003D2A0  call        CInterThreadMarshal::~CInterThreadMarshal (701D7894h)  ;(*ppITM)->~CInterThreadMarshal()(this<esi>)
7003D2A5  push        esi  
7003D2A6  call        operator delete (6FEA35D9h)  ;delete *ppITM
7003D2AB  pop         ecx  
7003D2AC  mov         dword ptr [edi],ebx  ;*ppITM = 0
7003D2AE  cmp         dword ptr [ebp-4],ebx  
7003D2B1  jl          CDownloadUtilities::MarshalBindContextToStream+0C6h (7003D300h)  ;if (FAILED(hr)) goto 7003D300h
7003D2B3  mov         eax,dword ptr [edi]  ;eax = *ppITM
7003D2B5  cmp         eax,ebx  
7003D2B7  je          CDownloadUtilities::MarshalBindContextToStream+89h (7003D2C3h)  ;goto 7003D2C3h
7003D2B9  mov         ecx,dword ptr [ebp+8]  ;ecx = pDTP
7003D2BC  mov         dword ptr [ecx+18h],eax  ;*(pDTP + 18h) = eax
7003D2BF  mov         dword ptr [edi],ebx  ;*ppITM = 0
7003D2C1  jmp         CDownloadUtilities::MarshalBindContextToStream+0BDh (7003D2F7h)  
7003D2C3  push        4  
7003D2C5  call        operator new (6FE9E771h)  ;eax = new LPSTREAM
7003D2CA  pop         ecx  
7003D2CB  cmp         eax,ebx  
7003D2CD  je          CDownloadUtilities::MarshalBindContextToStream+99h (7003D2D3h)  ;if (eax == 0) goto 7003D2D3h
7003D2CF  mov         dword ptr [eax],ebx  ;[eax] = 0
7003D2D1  jmp         CDownloadUtilities::MarshalBindContextToStream+9Bh (7003D2D5h)  
7003D2D3  xor         eax,eax  
7003D2D5  mov         ecx,dword ptr [ebp+8]  ;ecx = pDTP
7003D2D8  mov         dword ptr [ecx+18h],eax  ;*(pDTP + 18h) = eax
7003D2DB  mov         dword ptr [ebp-4],8007000Eh  ;hr = 8007000Eh
7003D2E2  cmp         eax,ebx  
7003D2E4  je          CDownloadUtilities::MarshalBindContextToStream+0BDh (7003D2F7h)  ;if (eax == 0) goto 7003D2F7h
7003D2E6  push        eax  
7003D2E7  push        dword ptr [ebp-8]  ;pUnk
7003D2EA  push        offset _IID_IBindCtx (6FF2AB8Ch)  
7003D2EF  call        _CoMarshalInterThreadInterfaceInStream@12 (701FF7F2h)  ;eax = CoMarshalInterThreadInterfaceInStream(&IID_IBindCtx, pUnk, eax)
7003D2F4  mov         dword ptr [ebp-4],eax  ;hr = eax
7003D2F7  mov         eax,dword ptr [ebp-8]  ;eax = pUnk
7003D2FA  mov         ecx,dword ptr [eax]  ;ecx = pUnk->lpVtbl
7003D2FC  push        eax  ;pUnk
7003D2FD  call        dword ptr [ecx+8]  ;pUnk->lpVtbl->Release(lpVtbl)
7003D300  call        LCIEUnifiedFrame (6FF9CD1Fh)  
7003D305  test        al,al  
7003D307  je          CDownloadUtilities::MarshalBindContextToStream+128h (7003D362h)  
7003D309  lea         eax,[ebp-0Ch]  
7003D30C  push        eax  
7003D30D  call        dword ptr [__imp_CoCreateUserBroker (6FE925A4h)]  
7003D313  test        eax,eax  
7003D315  js          CDownloadUtilities::MarshalBindContextToStream+128h (7003D362h)  
7003D317  lea         eax,[ebp+0Ch]  
7003D31A  push        eax  
7003D31B  push        dword ptr [ebp-0Ch]  
7003D31E  push        offset _IID_IEUserBroker (6FF37F6Ch)  
7003D323  call        dword ptr [__imp__CoMarshalInterThreadInterfaceInStream@12 (6FE91FECh)]  
7003D329  mov         dword ptr [ebp-4],eax  
7003D32C  cmp         eax,ebx  
7003D32E  jne         CDownloadUtilities::MarshalBindContextToStream+11Fh (7003D359h)  
7003D330  mov         edi,dword ptr [ebp+8]  
7003D333  mov         eax,dword ptr [edi+14h]  
7003D336  mov         esi,dword ptr [ebp+0Ch]  
7003D339  cmp         eax,ebx  
7003D33B  je          CDownloadUtilities::MarshalBindContextToStream+109h (7003D343h)  
7003D33D  mov         ecx,dword ptr [eax]  
7003D33F  push        eax  
7003D340  call        dword ptr [ecx+8]  
7003D343  mov         dword ptr [edi+14h],esi  
7003D346  cmp         esi,ebx  
7003D348  je          CDownloadUtilities::MarshalBindContextToStream+116h (7003D350h)  
7003D34A  mov         eax,dword ptr [esi]  
7003D34C  push        esi  
7003D34D  call        dword ptr [eax+4]  
7003D350  mov         eax,dword ptr [ebp+0Ch]  
7003D353  mov         ecx,dword ptr [eax]  
7003D355  push        eax  
7003D356  call        dword ptr [ecx+8]  
7003D359  mov         eax,dword ptr [ebp-0Ch]  
7003D35C  mov         ecx,dword ptr [eax]  
7003D35E  push        eax  
7003D35F  call        dword ptr [ecx+8]  
7003D362  mov         eax,dword ptr [ebp-4]  
7003D365  pop         edi  
7003D366  pop         esi  
7003D367  pop         ebx  
7003D368  leave  
7003D369  ret         0Ch  
7003D36C  nop  
7003D36D  nop  
7003D36E  nop  
7003D36F  nop  
7003D370  nop  
```

简单改写成 C++代码

代码:



### 代码
```
HRESULT CDownloadUtilities::MarshalBindContextToStream(CInterThreadMarshal **ppITM<eax>, CDownloadThreadParam *pDTP, wchar_t *szURL, IBindCtx *pBC)
{
  IEUserBroker *pIEUB;
  IUnknown *pUnk;
  HRESULT hr;
  if (SUCCEEDED(hr = pBC->QueryInterface(&IID_IUnknown, &pUnk)))
  {
    if (!_wcsnicmp(szURL, L"mk:", 3))
    {
      pUnk->Release();
      pUnk = NULL;
      hr = CreateAsyncBindCtxEx(0, 0, 0, 0, &pBC, 0);
      if (*ppITM)
      {
        (*ppITM)->~CInterThreadMarshal();
        delete *ppITM;
        *ppITM = NULL;
      }
    }
    if (SUCCEEDED(hr))
    {
      if (*ppITM)
      {
        *(pDTP + 18h) = *ppITM;
        *ppITM = NULL;
      }
      else
      {
        IStream **ppStm = new LPSTREAM;
        if (ppStm)
        {
          *ppStm = 0;
        }
        else
        {
          ppStm = NULL;
        }
        *(pDTP + 18h) = ppStm;
        hr = 8007000Eh;
        if (ppStm)
        {
          hr = CoMarshalInterThreadInterfaceInStream(&IID_IBindCtx, pUnk, ppStm);
        }
      }
      pUnk->Release();
    }
  }
  //omit
  return hr;
}
```

首先函数调用 `pBC->QueryInterface` 获得 `pBC` 对象的 `IUnknown` 接口 `pUnk`，由于 `IBindCtx` 只继承自 `IUnknown`，所以 `IUnknown` 接口和 `IBindCtx` 接口地址实际相同，当下载 CHM 内嵌的文件时，`CDownloadUtilities::MarshalBindContextToStream` 函数的 `szURL` 参数为形如 `mk:XXX` 的字符串，所以函数调用 `pUnk->Release` 又设 `pUnk=NULL`，而 `pUnk` 原本的计数应为 1，`pUnk->Release` 彻底释放了 `pBC` 对象，于是函数调用 `CreateAsyncBindCtxEx` 重新获得 `pBC` 对象，`pBC` 是更新了但 `pUnk` 仍然为 `NULL`，此外函数又彻底释放了 `*ppITM` 对象并设 `*ppITM=NULL`，于是之后函数会调用 `CoMarshalInterThreadInterfaceInStream` 列集接口指针，但调用时第二个参数为 `pUnk`，但 `pUnk=NULL`，实际应为 `pBC`，这还不是大问题，紧跟着的 `pUnk->Release` 才是祸根，由于 `pUnk=NULL`，取 `pUnk` 的虚函数表指针造成访问违例，程序报错，不过幸好之前 `pUnk` 设为了 `NULL`，不然后果就更糟糕了![](img/afde35a5f3b3a4675abb3221cf07c6d1.jpg "124.gif")，解决方法应该是换用 `pBC`。当 `szURL` 参数不是形如 `mk:XXX` 的字符串时就不会产生问题了。

附件是编译好的一个 CHM，其中的 `1.rar` 是本人写的一个小工具，说明见本人博客 [`hi.baidu.com/promised_lu`](http://hi.baidu.com/promised_lu)

上传的附件

| ![](img/31a2c422ae120fd0b6c241a4e6e83632.jpg) || chm.7z |

`KB2586448` 把 `CreateAsyncBindCtxEx(0, 0, 0, 0, &pBC, 0);` 改成了 `CreateAsyncBindCtxEx(0, 0, 0, 0, &pUnk, 0);` 解决了问题

函数声明改成了 `HRESULT CDownloadUtilities::MarshalBindContextToStream(IBindCtx *pBC<eax>, CInterThreadMarshal **ppITM<ecx>, CDownloadThreadParam *pDTP, wchar_t *szURL)`

代码如下

代码:



```
.text:101AD3AA                 mov     edi, edi
.text:101AD3AC                 push    ebp
.text:101AD3AD                 mov     ebp, esp
.text:101AD3AF                 sub     esp, 0Ch
.text:101AD3B2                 push    ebx
.text:101AD3B3                 push    esi
.text:101AD3B4                 push    edi
.text:101AD3B5                 lea     edx, \[ebp+pUnk\]
.text:101AD3B8                 push    edx
.text:101AD3B9                 push    offset stru\_100D5E20
.text:101AD3BE                 mov     edi, ecx
.text:101AD3C0                 mov     ecx, \[eax\]
.text:101AD3C2                 push    eax
.text:101AD3C3                 call    dword ptr \[ecx\]
.text:101AD3C5                 xor     ebx, ebx
.text:101AD3C7                 mov     \[ebp+var\_4\], eax
.text:101AD3CA                 cmp     eax, ebx
.text:101AD3CC                 jl      loc\_101AD46D
.text:101AD3D2                 push    3               ; MaxCount
.text:101AD3D4                 push    offset aMk      ; "mk:"
.text:101AD3D9                 push    \[ebp+szURL\]     ; Str1
.text:101AD3DC                 call    ds:\_\_imp\_\_wcsnicmp
.text:101AD3E2                 add     esp, 0Ch
.text:101AD3E5                 test    eax, eax
.text:101AD3E7                 jnz     short loc\_101AD41B
.text:101AD3E9                 mov     eax, \[ebp+pUnk\]
.text:101AD3EC                 mov     ecx, \[eax\]
.text:101AD3EE                 push    eax
.text:101AD3EF                 call    dword ptr \[ecx+8\]
.text:101AD3F2                 push    ebx             ; reserved
.text:101AD3F3                 lea     eax, \[ebp+pUnk\]
.text:101AD3F6                 push    eax             ; ppBC
.text:101AD3F7                 push    ebx             ; pEnum
.text:101AD3F8                 push    ebx             ; pBSCb
.text:101AD3F9                 push    ebx             ; dwOptions
.text:101AD3FA                 push    ebx             ; pbc
.text:101AD3FB                 mov     \[ebp+pUnk\], ebx
.text:101AD3FE                 call    CreateAsyncBindCtxEx
.text:101AD404                 mov     esi, \[edi\]
.text:101AD406                 mov     \[ebp+var\_4\], eax
.text:101AD409                 cmp     esi, ebx
.text:101AD40B                 jz      short loc\_101AD41B
.text:101AD40D                 call    sub\_10347BB6
.text:101AD412                 push    esi             ; lpMem
.text:101AD413                 call    sub\_100135D9
.text:101AD418                 pop     ecx
.text:101AD419                 mov     \[edi\], ebx
.text:101AD41B
.text:101AD41B loc\_101AD41B:                           ; CODE XREF: sub\_101AD3AA+3Dj
.text:101AD41B                                         ; sub\_101AD3AA+61j
.text:101AD41B                 cmp     \[ebp+var\_4\], ebx
.text:101AD41E                 jl      short loc\_101AD46D
.text:101AD420                 mov     eax, \[edi\]
.text:101AD422                 cmp     eax, ebx
.text:101AD424                 jz      short loc\_101AD430
.text:101AD426                 mov     ecx, \[ebp+arg\_0\]
.text:101AD429                 mov     \[ecx+18h\], eax
.text:101AD42C                 mov     \[edi\], ebx
.text:101AD42E                 jmp     short loc\_101AD464
.text:101AD430 ; ---------------------------------------------------------------------------
.text:101AD430
.text:101AD430 loc\_101AD430:                           ; CODE XREF: sub\_101AD3AA+7Aj
.text:101AD430                 push    4               ; dwBytes
.text:101AD432                 call    sub\_1000E771
.text:101AD437                 pop     ecx
.text:101AD438                 cmp     eax, ebx
.text:101AD43A                 jz      short loc\_101AD440
.text:101AD43C                 mov     \[eax\], ebx
.text:101AD43E                 jmp     short loc\_101AD442
.text:101AD440 ; ---------------------------------------------------------------------------
.text:101AD440
.text:101AD440 loc\_101AD440:                           ; CODE XREF: sub\_101AD3AA+90j
.text:101AD440                 xor     eax, eax
.text:101AD442
.text:101AD442 loc\_101AD442:                           ; CODE XREF: sub\_101AD3AA+94j
.text:101AD442                 mov     ecx, \[ebp+arg\_0\]
.text:101AD445                 mov     \[ecx+18h\], eax
.text:101AD448                 mov     \[ebp+var\_4\], 8007000Eh
.text:101AD44F                 cmp     eax, ebx
.text:101AD451                 jz      short loc\_101AD464
.text:101AD453                 push    eax             ; szURL
.text:101AD454                 push    \[ebp+pUnk\]      ; pUnk
.text:101AD457                 push    offset stru\_1009ABFC ; riid
.text:101AD45C                 call    CoMarshalInterThreadInterfaceInStream
.text:101AD461                 mov     \[ebp+var\_4\], eax
.text:101AD464
.text:101AD464 loc\_101AD464:                           ; CODE XREF: sub\_101AD3AA+84j
.text:101AD464                                         ; sub\_101AD3AA+A7j
.text:101AD464                 mov     eax, \[ebp+pUnk\]
.text:101AD467                 mov     ecx, \[eax\]
.text:101AD469                 push    eax
.text:101AD46A                 call    dword ptr \[ecx+8\]
.text:101AD46D
.text:101AD46D loc\_101AD46D:                           ; CODE XREF: sub\_101AD3AA+22j
.text:101AD46D                                         ; sub\_101AD3AA+74j
.text:101AD46D                 call    sub\_1010CD9F
.text:101AD472                 test    al, al
.text:101AD474                 jz      short loc\_101AD4CF
.text:101AD476                 lea     eax, \[ebp+var\_C\]
.text:101AD479                 push    eax
.text:101AD47A                 call    ds:iertutil\_58
.text:101AD480                 test    eax, eax
.text:101AD482                 js      short loc\_101AD4CF
.text:101AD484                 lea     eax, \[ebp+szURL\]
.text:101AD487                 push    eax             ; szURL
.text:101AD488                 push    \[ebp+var\_C\]     ; pUnk
.text:101AD48B                 push    offset stru\_100A7FDC ; riid
.text:101AD490                 call    ds:\_\_imp\_CoMarshalInterThreadInterfaceInStream
.text:101AD496                 mov     \[ebp+var\_4\], eax
.text:101AD499                 cmp     eax, ebx
.text:101AD49B                 jnz     short loc\_101AD4C6
.text:101AD49D                 mov     edi, \[ebp+arg\_0\]
.text:101AD4A0                 mov     eax, \[edi+14h\]
.text:101AD4A3                 mov     esi, \[ebp+szURL\]
.text:101AD4A6                 cmp     eax, ebx
.text:101AD4A8                 jz      short loc\_101AD4B0
.text:101AD4AA                 mov     ecx, \[eax\]
.text:101AD4AC                 push    eax
.text:101AD4AD                 call    dword ptr \[ecx+8\]
.text:101AD4B0
.text:101AD4B0 loc\_101AD4B0:                           ; CODE XREF: sub\_101AD3AA+FEj
.text:101AD4B0                 mov     \[edi+14h\], esi
.text:101AD4B3                 cmp     esi, ebx
.text:101AD4B5                 jz      short loc\_101AD4BD
.text:101AD4B7                 mov     eax, \[esi\]
.text:101AD4B9                 push    esi
.text:101AD4BA                 call    dword ptr \[eax+4\]
.text:101AD4BD
.text:101AD4BD loc\_101AD4BD:                           ; CODE XREF: sub\_101AD3AA+10Bj
.text:101AD4BD                 mov     eax, \[ebp+szURL\]
.text:101AD4C0                 mov     ecx, \[eax\]
.text:101AD4C2                 push    eax
.text:101AD4C3                 call    dword ptr \[ecx+8\]
.text:101AD4C6
.text:101AD4C6 loc\_101AD4C6:                           ; CODE XREF: sub\_101AD3AA+F1j
.text:101AD4C6                 mov     eax, \[ebp+var\_C\]
.text:101AD4C9                 mov     ecx, \[eax\]
.text:101AD4CB                 push    eax
.text:101AD4CC                 call    dword ptr \[ecx+8\]
.text:101AD4CF
.text:101AD4CF loc\_101AD4CF:                           ; CODE XREF: sub\_101AD3AA+CAj
.text:101AD4CF                                         ; sub\_101AD3AA+D8j
.text:101AD4CF                 mov     eax, \[ebp+var\_4\]
.text:101AD4D2                 pop     edi
.text:101AD4D3                 pop     esi
.text:101AD4D4                 pop     ebx
.text:101AD4D5                 leave
.text:101AD4D6                 retn    8
```
