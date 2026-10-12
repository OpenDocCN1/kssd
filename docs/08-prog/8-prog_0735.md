
# 获取 IE 历史记录探究性学习

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-303.htm`](https://www.pediy.com/kssd/pediy09/pediy09-303.htm)

> 突然对此感兴趣,所以研究一下,顺便给大家献献丑.

> 在互联网上搜索相关代码,你会发现多半以删除历史记录类的代码为主,而真正实现获取的获取 IE 记录的比较少.

> 先说说常规方法:利用 COM 接口 `IUrlHistory` 接口实现访问.

> 翻阅 SDK 你会发现,MS 对此的文档少的几乎无法使用，既无 `Example`，而且连基本的 `CLSID` 也没有提供。当然对于一些方法的参数的调用说明也是有问题的，比如说明是 `Reserved`，但是并没有制定要给定什么值，但实战中必须给 1 才能够正确调用。

> 这里就直接给一个网上的代码，很好理解。不作过多解释。

```c
#include <stdio.h>  
#include <windows.h>  
#include <UrlHist.h>   // IUrlHistoryStg2  
#include <shlobj.h>    // CLSID_CUrlHistory, SHAddToRecentDocs  
#include <atlbase.h>   // USES_CONVERSION;  

void GetIEHistory()
{
    USES_CONVERSION;//Unicode 转 Ansi 用
    CoInitialize(NULL); //初始化
    
    IUrlHistoryStg2* pUrlHistoryStg2 = NULL;
    HRESULT hr = CoCreateInstance(CLSID_CUrlHistory,
        NULL, CLSCTX_INPROC, IID_IUrlHistoryStg2,
        (void**)&pUrlHistoryStg2);
    
    /******if (SUCCEEDED(hr))
```



```
{
    hr = pUrlHistoryStg2->ClearHistory();
    pUrlHistoryStg2->Release();
}*/

IEnumSTATURL* pEnumURL;
hr = pUrlHistoryStg2->EnumUrls(&pEnumURL);

STATURL suURL;
ULONG pceltFetched;
suURL.cbSize = sizeof(suURL);
hr = pEnumURL->Reset();

while((hr = pEnumURL->Next(1, &suURL, &pceltFetched)) == S_OK)
{
    //hr = pUrlHistoryStg2->DeleteUrl(suURL.pwcsUrl, 0);
    printf("%s ", W2T(suURL.pwcsUrl));
}

pEnumURL->Release();
```



```
>  \[if gte vml 1\]><v:shape id="图片\_x0020\_40" o:spid="\_x0000\_i1027" type="#\_x0000\_t75" alt="http://images.csdn.net/syntaxhighlighting/OutliningIndicators/InBlock.gif" style='width:8.25pt;height:12pt;visibility:visible;mso-wrap-style:square'> <v:imagedata src="获取 IE 历史记录探究性学习.files/image004.gif" o:title="InBlock"/> </v:shape><!\[endif\]     pUrlHistoryStg2->Release();  

>  \[if gte vml 1\]><v:shape id="图片\_x0020\_41" o:spid="\_x0000\_i1026" type="#\_x0000\_t75" alt="http://images.csdn.net/syntaxhighlighting/OutliningIndicators/InBlock.gif" style='width:8.25pt;height:12pt;visibility:visible;mso-wrap-style:square'> <v:imagedata src="获取 IE 历史记录探究性学习.files/image004.gif" o:title="InBlock"/> </v:shape><!\[endif\]     CoUninitialize();  

>  \[if gte vml 1\]><v:shape id="图片\_x0020\_42" o:spid="\_x0000\_i1025" type="#\_x0000\_t75" alt="http://images.csdn.net/syntaxhighlighting/OutliningIndicators/ExpandedBlockEnd.gif" style='width:8.25pt;height:12pt;visibility:visible;mso-wrap-style:square'> <v:imagedata src="获取 IE 历史记录探究性学习.files/image008.gif" o:title="ExpandedBlockEnd"/> </v:shape><!\[endif\] }
```

原文地址请见 [`blog.csdn.net/yzzm521/archive/2006/12/26/1462789.aspx`](http://blog.csdn.net/yzzm521/archive/2006/12/26/1462789.aspx)

但这些能够找到的代码由一个问题：

如果你让上面的代码 LOOP 10 次的话，你会惊奇的发现内存将会迅速上升，当然不会下降。这貌似就是传说中的内存泄露，但是翻阅整个关于这个接口的 `MSDN` 文档，并找不到任何有价值的信息，当然常规的 `IUnknown::Release` 肯定已经是执行了的。可以想见问题不是出在这里。

故需我们拿上秘密武器 `OllyDbg` 开始调试：

我们单步会发现在 `p_IEnumSTATURL->Next(1,&statUrl,&bSusFill);` 这个语句一旦执行内存就会上升。

当然这个方法是填充一个结构。而这个结构的定义

```
typedef struct _STATURL {
    DWORD cbSize;
    LPWSTR pwcsUrl;
    LPWSTR pwcsTitle;
    FILETIME ftLastVisited;
    FILETIME ftLastUpdated;
    FILETIME ftExpires;
    DWORD dwFlags;
} STATURL, *LPSTATURL
```

如上

看这个结构中 `Offset + 04` 和 `Offset + 08` 两个字段是一个指针类型指向一个 Unicode 文本。按照猜测问题应该出在这里：

这个时候我们可以打开 `IDA` 来分析，当然直接用 `OD` 也是可以的，我们进入 `Next` 这个方法来到

```
6EE6AD2A <>    8BFF               mov edi,edi                     ;  CEnumSTATURL::Next(ulong,_STATURL *,ulong *)
6EE6AD2C    .  55                push ebp
6EE6AD2D    .  8BEC              mov ebp,esp
6EE6AD2F    .  53                push ebx
6EE6AD30    .  57                push edi
6EE6AD31    .  8B7D 14           mov edi,dword ptr ss:[ebp+14]
6EE6AD34    .  33DB             xor ebx,ebx
6EE6AD36    .  33C0             xor eax,eax
6EE6AD38    .  3BFB             cmp edi,ebx
6EE6AD3A    .  74 02            je short ieframe.6EE6AD3E
6EE6AD3C    .  891F             mov dword ptr ds:[edi],ebx
6EE6AD3E    >  395D 0C          cmp dword ptr ss:[ebp+C],ebx
6EE6AD41    .  74 55            je short ieframe.6EE6AD98
6EE6AD43    .  56               push esi
6EE6AD44    .  8B75 08          mov esi,dword ptr ss:[ebp+8]
6EE6AD47    .  395E 10          cmp dword ptr ds:[esi+10],ebx
6EE6AD4A    .^ 0F84 78FFFFFF     je ieframe.6EE6ACC8
6EE6AD50    >  8BCE             mov ecx,esi
6EE6AD52    .  E8 4C000000       call <ieframe.CEnumSTATURL::RetrieveNextUrlInfo(void)>
6EE6AD57    .  3BC3             cmp eax,ebx
6EE6AD59    .  75 3C            jnz short ieframe.6EE6AD97
6EE6AD5B    .  8B8E 68100000     mov ecx,dword ptr ds:[esi+1068]
6EE6AD61    .  3BCB             cmp ecx,ebx
6EE6AD63    .  74 32            je short ieframe.6EE6AD97
6EE6AD65    .  FFB6 5C100000     push dword ptr ds:[esi+105C]                     ; /Arg3
6EE6AD6B    .  8D46 14          lea eax,dword ptr ds:[esi+14]                     ; |
6EE6AD6E    .  FF71 04          push dword ptr ds:[ecx+4]                         ; |Arg2
6EE6AD71    .  50               push eax                                         ; |Arg1
6EE6AD72    .  E8 209E0800      call <ieframe.StrCmpNW(x,x,x)>                   ; \\ieframe.6EEF4B97
6EE6AD77    .  85C0             test eax,eax
6EE6AD79    .^ 75 D5            jnz short ieframe.6EE6AD50
6EE6AD7B    >  FF75 10          push dword ptr ss:[ebp+10]
6EE6AD7E    .  FF76 0C          push dword ptr ds:[esi+C]
6EE6AD81    .  FFB6 68100000     push dword ptr ds:[esi+1068]
6EE6AD87    .  53               push ebx
6EE6AD88    .  E8 1FDC0800      call <ieframe.CUrlHistory::s_GenerateSTATURL(ushort cons>
6EE6AD8D    .  3BC3             cmp eax,ebx
6EE6AD8F    .  7C 06            jl short ieframe.6EE6AD97
6EE6AD91     .  3BFB             cmp edi,ebx
6EE6AD93    .  74 02            je short ieframe.6EE6AD97
6EE6AD95    .  FF07             inc dword ptr ds:[edi]
6EE6AD97    >  5E               pop esi
6EE6AD98    >  5F               pop edi
6EE6AD99    .  5B                pop ebx
6EE6AD9A    .  5D               pop ebp
6EE6AD9B    .  C2 1000          retn 10
```

分析上面三个 `CALL` 我们发现最后一个 `s_GenerateSTATUrl` 引起了我们的注意：

```
.text:75CD8A94                mov     edi, 824h
.text:75CD8A99                push    edi
.text:75CD8A9A                lea     eax, [ebp-104Ch]
.text:75CD8AA0                push    eax
.text:75CD8AA1                push    16h
.text:75CD8AA3                call    ?_GetStringProp@CHistoryData@@QBEJKPAGK@Z ; CHistoryData::_GetStringProp(ulong,ushort *,ulong)
.text:75CD8AA8                test    eax, eax
.text:75CD8AAA                jge     short loc_75CD8AC8
.text:75CD8AAC                push    1               ; int
.text:75CD8AAE                lea     eax, [ebp-1050h]
.text:75CD8AB4                push    eax             ; int
.text:75CD8AB5                lea     eax, [ebp-104Ch]
.text:75CD8ABB                push    eax             ; unsigned __int16 *
.text:75CD8ABC                push    ebx             ; int
.text:75CD8ABD                mov     [ebp-1050h], edi
.text:75CD8AC3                call    _PrepareURLForDisplayUTF8W@16 ; PrepareURLForDisplayUTF8W(x,x,x,x)
.text:75CD8AC8

.text:75CD8AC8 loc_75CD8AC8:               ; CODE XREF: sub_75CD89B0+FAj
.text:75CD8AC8                lea     eax, [esi+4]
.text:75CD8ACB                push    eax             ; int
.text:75CD8ACC                lea     eax, [ebp-104Ch]
.text:75CD8AD2                push    eax             ; lpString
.text:75CD8AD3                 call    _SHStrDupW@8    ; SHStrDupW(x,x)
.text:75CD8AD8                mov     [ebp-1050h], eax
.text:75CD8ADE

.text:75CD8ADE loc_75CD8ADE:               ; CODE XREF: sub_75CD89B0-8D7F0j
.text:75CD8ADE                         ; sub_75CD89B0+CCj
.text:75CD8ADE                test    dword ptr [ebp+10h], 40000h
.text:75CD8AE5                jz      short loc_75CD8AFE
```

在这个 `CALL` 中多次用到 `SHStrDupW` 这个函数，熟悉 `C` 语言的朋友应该知道 `StrDup` 这个函数的用法，即便是查阅 `MSDN` 也会发现

> Makes a copy of a string in newly allocated memory.

**Syntax**

```
HRESULT  SHStrDup(      
   LPCTSTR *pszSource*,
   LPTSTR **ppwsz*
);
```

**Parameters**

`pszSource` 

[in] A pointer to the null-terminated string to be copied. 

`ppwsz` 

[out] A pointer to an allocated Unicode string containing the result. `SHStrDup` allocates memory for this string with `CoTaskMemAlloc`. You should free the string with `CoTaskMemFree` when it is no longer needed. 

我们需要用 `CoTaskMemFree` 释放内存

因此我们只需要简单得讲这两个字符串指针进行释放即可，代码如下：

```
void GetUrlHistoryMethod1()
{
    HRESULT hr = ::CoInitialize(NULL);
```



```c
    IUrlHistoryStg * p_IUrlHistoryStg;

    STATURL statUrl;

    IEnumSTATURL * p_IEnumSTATURL;

    hr = ::CoCreateInstance(
        CLSID_CUrlHistory,
        NULL,
        CLSCTX_INPROC_SERVER,
        IID_IUrlHistoryStg,
        (void **)&p_IUrlHistoryStg
        );

    if(!hr){
        hr = p_IUrlHistoryStg->EnumUrls(&p_IEnumSTATURL);
        if(!hr){
            ULONG bSusFill = FALSE;
            hr = p_IEnumSTATURL->Reset();
            statUrl.cbSize = sizeof(statUrl);
            hr = p_IEnumSTATURL->Next(1,&statUrl,&bSusFill);
            int n = 0;
            while(bSusFill && !hr)
            {
                /*
                typedef struct _STATURL
                {
                DWORD cbSize;
                LPWSTR pwcsUrl;
                LPWSTR pwcsTitle;
                FILETIME ftLastVisited;
                FILETIME ftLastUpdated;
                FILETIME ftExpires;
                DWORD dwFlags;
                }   STATURL;
                */

                printf(" [%d] Url : %S,Title : %S \r\n",n,statUrl.pwcsUrl,statUrl.pwcsTitle);
                n++;
                ::CoTaskMemFree(statUrl.pwcsTitle);
                ::CoTaskMemFree(statUrl.pwcsUrl);
                p_IEnumSTATURL->Next(1,&statUrl,&bSusFill);
            }
            p_IEnumSTATURL->Release();
        }
        p_IUrlHistoryStg->Release();
    }

    ::CoFreeAllLibraries();
    ::CoUninitialize();
}
```

当然你会发现内存占用不会递增了。

不过如果你对这种半文档化东西不是十分放心的话，我们可以考虑文档化的东西，但这需要我们自己动一些脑精：

如果我们跟踪 `IUrlHistory` 接口的实现你会发现它本质上还是调用的 `WININET` 函数实现。因此我们只需要逆向这部分即可。接着先前贴的那段代码：

我们发现 `CEnumSTATURL::RetrieveNextUrlInfo(void)` 这样一个名字比较引人注目

我们进入这个函数

```asm
?RetrieveNextUrlInfo@CEnumSTATURL@@AAEJXZ proc near

.text:75C4ADA3                             ; CODE XREF: CEnumSTATURL::Next(ulong,_STATURL *,ulong *)+28p
.text:75C4ADA3                             ; CEnumSTATURL::Next(ulong,_STATURL *,ulong *)+AC5C7p
.text:75C4ADA3
.text:75C4ADA3 var_4            = dword ptr -4
.text:75C4ADA3
.text:75C4ADA3 ; FUNCTION CHUNK AT .text:75C2A7B8 SIZE 00000041 BYTES
.text:75C4ADA3 ; FUNCTION CHUNK AT .text:75CF6F44 SIZE 00000043 BYTES
.text:75C4ADA3
.text:75C4ADA3                    mov     edi, edi
.text:75C4ADA5                    push    ebp
.text:75C4ADA6                    mov     ebp, esp
.text:75C4ADA8                    push    ecx
.text:75C4ADA9                    and     [ebp+var_4], 0
.text:75C4ADAD                     push    esi
.text:75C4ADAE                    mov     esi, ecx
.text:75C4ADB0                    cmp     dword ptr [esi+1068h], 0
.text:75C4ADB7                    jz      short loc_75C4ADDA
.text:75C4ADB9                    push    ebx
.text:75C4ADBA                     lea     ebx, [esi+1064h]
.text:75C4ADC0                    push    edi
.text:75C4ADC1
.text:75C4ADC1 loc_75C4ADC1:                       ; CODE XREF: CEnumSTATURL::RetrieveNextUrlInfo(void)+AC1B3j
.text:75C4ADC1                    push    ebx                ; int
.text:75C4ADC2                    push    dword ptr [esi+1068h] ; int
.text:75C4ADC8                    push    dword ptr [esi+10h] ; hEnumHandle
.text:75C4ADCB                    call    ?FindNextUrlCacheEntryBinary@@YGHPAXPAU_INTERNET_CACHE_ENTRY_INFOW@@PAK@Z ; FindNextUrlCacheEntryBinary(void *,_INTERNET_CACHE_ENTRY_INFOW *,ulong *)
.text:75C4ADD0                    test    eax, eax
.text:75C4ADD2                    jz      loc_75C2A7B8
.text:75C4ADD8
.text:75C4ADD8 loc_75C4ADD8:                      ; CODE XREF: CEnumSTATURL::RetrieveNextUrlInfo(void)-205AFj
.text:75C4ADD8                     ; CEnumSTATURL::RetrieveNextUrlInfo(void)+AC1C0j ...
.text:75C4ADD8                    pop     edi
.text:75C4ADD9                     pop     ebx
.text:75C4ADDA
.text:75C4ADDA loc_75C4ADDA:                       ; CODE XREF: CEnumSTATURL::RetrieveNextUrlInfo(void)+14j
.text:75C4ADDA                    lea     eax, [esi+1064h]
.text:75C4ADE0                    mov     edx, [eax]
.text:75C4ADE2                    mov     ecx, 3E8h
.text:75C4ADE7                    cmp     edx, ecx
.text:75C4ADE9                    pop     esi
.text:75C4ADEA                    ja      loc_75CF6F80
.text:75C4ADF0
.text:75C4ADF0 loc_75C4ADF0:                        ; CODE XREF: CEnumSTATURL::RetrieveNextUrlInfo(void)+AC1DFj
.text:75C4ADF0                    mov     [eax], ecx
.text:75C4ADF2                    mov     eax, [ebp+var_4]
.text:75C4ADF5                    leave
.text:75C4ADF6                    retn
.text:75C4ADF6 ?RetrieveNextUrlInfo@CEnumSTATURL@@AAEJXZ endp
```

唯一一个 Call：

```asm
; int __stdcall FindNextUrlCacheEntryBinary(HANDLE hEnumHandle,int,int)
.text:75C4ADFC ?FindNextUrlCacheEntryBinary@@YGHPAXPAU_INTERNET_CACHE_ENTRY_INFOW@@PAK@Z proc near
.text:75C4ADFC                     ; CODE XREF: CEnumSTATURL::RetrieveNextUrlInfo(void)+28p
.text:75C4ADFC
.text:75C4ADFC var_1014            = dword ptr -1014h
.text:75C4ADFC var_1010            = dword ptr -1010h
.text:75C4ADFC uBytes              = dword ptr -100Ch
.text:75C4ADFC var_1008            = dword ptr -1008h
.text:75C4ADFC NextCacheEntryInfo= _INTERNET_CACHE_ENTRY_INFOA ptr -1004h
.text:75C4ADFC var_4               = dword ptr -4
.text:75C4ADFC hEnumHandle         = dword ptr  8
.text:75C4ADFC arg_4               = dword ptr  0Ch
.text:75C4ADFC arg_8               = dword ptr  10h
.text:75C4ADFC
.text:75C4ADFC ; FUNCTION CHUNK AT .text:75C2A792 SIZE 00000026 BYTES
.text:75C4ADFC ; FUNCTION CHUNK AT .text:75CF6DF5 SIZE 0000006E BYTES
.text:75C4ADFC
.text:75C4ADFC                      mov     edi, edi
.text:75C4ADFE                     push    ebp
.text:75C4ADFF                     mov     ebp, esp
.text:75C4AE01                     mov     eax, 1014h
.text:75C4AE06                     call    __chkstk
.text:75C4AE0B                     mov     eax, ___security_cookie
.text:75C4AE10                     xor     eax, ebp
.text:75C4AE12                     mov     [ebp+var_4], eax
.text:75C4AE15                     mov     eax, [ebp+arg_4]
.text:75C4AE18                     and     [ebp+var_1010], 0
.text:75C4AE1F                      push    ebx
.text:75C4AE20                     mov     ebx, [ebp+hEnumHandle]
.text:75C4AE23                     push    esi
.text:75C4AE24                     mov     esi, [ebp+arg_8]
.text:75C4AE27                     mov     [ebp+var_1014], eax
.text:75C4AE2D                       mov     eax, 1000h
.text:75C4AE32                     push    edi
.text:75C4AE33                     mov     [ebp+uBytes], eax
.text:75C4AE39                     mov     [ebp+NextCacheEntryInfo.dwStructSize], eax
.text:75C4AE3F                     lea     eax, [ebp+uBytes]
.text:75C4AE45                     push    eax             ; lpdwNextCacheEntryInfoBufferSize
.text:75C4AE46                     lea     edi, [ebp+NextCacheEntryInfo]
.text:75C4AE4C                     mov     eax, edi
.text:75C4AE4E                     push    eax             ; lpNextCacheEntryInfo
```



# 汇编代码片段分析

## FindNextUrlCacheEntryA 调用流程

```asm
.text:75C4AE4F                   push    ebx                 ; hEnumHandle

.text:75C4AE50                   call    _FindNextUrlCacheEntryA@12 ; FindNextUrlCacheEntryA(x,x,x)

.text:75C4AE55                   test    eax, eax

.text:75C4AE57                   mov     [ebp+var_1008], eax

.text:75C4AE5D                   jz      loc_75C2A792

.text:75C4AE63 loc_75C4AE63:                          ; CODE XREF: FindNextUrlCacheEntryBinary(void *,_INTERNET_CACHE_ENTRY_INFOW *,ulong *)-2064Fj

.text:75C4AE63                   push    edi                 ; struct _INTERNET_CACHE_ENTRY_INFOA *

.text:75C4AE64                   call    ?_ValidateHistoryData@@YGHPAU_INTERNET_CACHE_ENTRY_INFOA@@@Z ; _ValidateHistoryData(_INTERNET_CACHE_ENTRY_INFOA *)

.text:75C4AE69                   push    dword ptr [esi]     ; int

.text:75C4AE6B                   push    [ebp+var_1014]  ; struct _INTERNET_CACHE_ENTRY_INFOW *

.text:75C4AE71                   push    edi                 ; struct _INTERNET_CACHE_ENTRY_INFOA *

.text:75C4AE72                   call    ?CacheEntryInfoAToCacheEntryInfoW@@YGHPAU_INTERNET_CACHE_ENTRY_INFOA@@PAU_INTERNET_CACHE_ENTRY_INFOW@@H@Z ; CacheEntryInfoAToCacheEntryInfoW(_INTERNET_CACHE_ENTRY_INFOA *,_INTERNET_CACHE_ENTRY_INFOW *,int)

.text:75C4AE77                   test    eax, eax

.text:75C4AE79                   jnz     loc_75CF6E41

.text:75C4AE7F loc_75C4AE7F:                          ; CODE XREF: FindNextUrlCacheEntryBinary(void *,_INTERNET_CACHE_ENTRY_INFOW *,ulong *)-20649j
.text:75C4AE7F                                      ; FindNextUrlCacheEntryBinary(void *,_INTERNET_CACHE_ENTRY_INFOW *,ulong *)+AC056j

.text:75C4AE7F                   cmp     [ebp+var_1010], 0

.text:75C4AE86                   jnz     loc_75CF6E57

.text:75C4AE8C loc_75C4AE8C:                          ; CODE XREF: FindNextUrlCacheEntryBinary(void *,_INTERNET_CACHE_ENTRY_INFOW *,ulong *)-20661j
.text:75C4AE8C                                      ; FindNextUrlCacheEntryBinary(void *,_INTERNET_CACHE_ENTRY_INFOW *,ulong *)+AC062j

.text:75C4AE8C                   mov     ecx, [ebp+var_4]

.text:75C4AE8F                   mov     eax, [ebp+var_1008]

.text:75C4AE95                   pop     edi

.text:75C4AE96                   pop     esi

.text:75C4AE97                   xor     ecx, ebp

.text:75C4AE99                   pop     ebx

.text:75C4AE9A                   call    @__security_check_cookie@4 ; __security_check_cookie(x)

.text:75C4AE9F                   leave

.text:75C4AEA0                   retn    0Ch

.text:75C4AEA0 ?FindNextUrlCacheEntryBinary@@YGHPAXPAU_INTERNET_CACHE_ENTRY_INFOW@@PAK@Z endp
```

它竟然就是调用的 `FindNextUrlCacheEntryA` 函数，而这个函数是文档化的：

其中这个函数会返回一个 `LPINTERNET_CACHE_ENTRY_INFO` 结构体，这个结构中有我们要的大部分信息，但是却缺少一个十分重要的内容：也就是我们的历史记录的标题，因此我们还需要继续看

## CUrlHistory::s_GenerateSTATURL

```cpp
public: static long __stdcall CUrlHistory::s_GenerateSTATURL(unsigned short const *, struct _INTERNET_CACHE_ENTRY_INFOW *)
```

方法，用调试工具跟踪会发现：

```asm
loc_75CD8AFE:                        ; CODE XREF: CUrlHistory::s_GenerateSTATURL(ushort const *,_INTERNET_CACHE_ENTRY_INFOW *,ulong,_STATURL *)+139j

.text:75CD8AFE                   mov     ecx, [ebp+lpHistoryData]

.text:75CD8B04                   test    ecx, ecx

.text:75CD8B06                   jz      short loc_75CD8AE7

.text:75CD8B08                   push    10h

.text:75CD8B0A                   call    ?_FindExtra@CHistoryData@@QBEPBUHISTEXTRA@@I@Z ; CHistoryData::_FindExtra(uint)

.text:75CD8B0F                   test    eax, eax

.text:75CD8B11                   jz      short loc_75CD8AE7

.text:75CD8B13                   cmp     byte ptr [eax+3], 1Fh

.text:75CD8B17                   jnz     short loc_75CD8AE7

.text:75CD8B19                   lea     ecx, [esi+8]

.text:75CD8B1C                   push    ecx                 ; int

.text:75CD8B1D                   add     eax, 4

.text:75CD8B20                   push    eax                 ; lpString

.text:75CD8B21                   call    _SHStrDupW@8    ; SHStrDupW(x,x)

.text:75CD8B26                   test    eax, eax

.text:75CD8B28                   mov     [ebp+var_1050], eax

.text:75CD8B2E                   jge     short loc_75CD8AE7

.text:75CD8B30                   jmp     loc_75CF71F0
```

当到这里的时候，`lpTitle` 就会出现了，因此核心点就是 `CHistoryData::_FindExtra(uint)` 函数。进而这个函数的 `THIS` 指针是一个由一个局部变量实现，向上翻阅这个变量的引用会发现：

```asm
.text:75CD89E7                   push    edx

.text:75CD89E8                   call    ?s_GetHistoryData@CHistoryData@@SGPAV1@PAU_INTERNET_CACHE_ENTRY_INFOW@@@Z ; CHistoryData::s_GetHistoryData(_INTERNET_CACHE_ENTRY_INFOW *)

.text:75CD89ED                   mov     [ebp+lpHistoryData], eax
```

这个函数会对这个变量进行赋值操作，查找其参数会发现

```asm
.text:75CD89C5                   mov     edx, [ebp+LPINTERNET_CACHE_ENTRY_INFO]

.text:75CD89C8                   test    edx, edx
```

这个参数来自传入参数 2，也就是除 `THIS` 指针以外的参数 1。

```asm
push    [ebp+lpStatUrl]

.text:75C4AD7E                   push    dword ptr [esi+0Ch]

.text:75C4AD81                   push    dword ptr [esi+1068h]

.text:75C4AD87                   push    ebx

.text:75C4AD88                   call    ?s_GenerateSTATURL@CUrlHist
```

调用的 `[esi+1068h]`，很显然是一个 ClassMember。而这个只可能跟

```asm
mov     ecx, esi

.text:75C4AD52                   call    ?RetrieveNextUrlInfo@CEnumSTATURL@@AAEJXZ ; CEnumSTATURL::RetrieveNextUrlInfo(void)
```

有关，因为 `ESI` 传递给 `ECX` 作为 `THIS` 指针

```asm
mov     edi, edi

.text:75C4ADA5                   push    ebp

.text:75C4ADA6                   mov     ebp, esp

.text:75C4ADA8                   push    ecx

.text:75C4ADA9                   and     [ebp+var_4], 0

.text:75C4ADAD                   push    esi

.text:75C4ADAE                   mov     esi, ecx

.text:75C4ADB0                   cmp     dword ptr [esi+1068h], 0

.text:75C4ADB7                   jz      short loc_75C4ADDA

.text:75C4ADB9                   push    ebx

.text:75C4ADBA                   lea     ebx, [esi+1064h]

.text:75C4ADC0                   push    edi

.text:75C4ADC1 loc_75C4ADC1:                     ; CODE XREF: CEnumSTATURL::RetrieveNextUrlInfo(void)+AC1B3j

.text:75C4ADC1                   push    ebx                 ; int

.text:75C4ADC2                   push    dword ptr [esi+1068h] ; int

.text:75C4ADC8                   push    dword ptr [esi+10h] ; hEnumHandle

.text:75C4ADCB                   call    ?FindNextUrlCacheEntryBinary@@YGHPAXPAU_INTERNET_CACHE_ENTRY_INFOW@@PAK@Z ; FindNextUrlCacheEntryBinary(void *,_INTERNET_CACHE_ENTRY_INFOW *,ulong *)

.text:75C4ADD0                   test    eax, eax

.text:75C4ADD2                   jz      loc_75C2A7B8
```

因此顺利找到引用，进入这个 `FindNextUrlCacheEntryBinary`

```asm
.text:75C4ADFC ; int __stdcall FindNextUrlCacheEntryBinary(HANDLE hEnumHandle,int,int)

.text:75C4ADFC ?FindNextUrlCacheEntryBinary@@YGHPAXPAU_INTERNET_CACHE_ENTRY_INFOW@@PAK@Z proc near
.text:75C4ADFC                                      ; CODE XREF: CEnumSTATURL::RetrieveNextUrlInfo(void)+28p

.text:75C4ADFC var_1014       = dword ptr -1014h

.text:75C4ADFC var_1010       = dword ptr -1010h

.text:75C4ADFC uBytes         = dword ptr -100Ch

.text:75C4ADFC var_1008       = dword ptr -1008h

.text:75C4ADFC NextCacheEntryInfo= _INTERNET_CACHE_ENTRY_INFOA ptr -1004h

.text:75C4ADFC var_4          = dword ptr -4
```



```
.text:75C4ADFC hEnumHandle     = dword ptr  8

.text:75C4ADFC arg_4           = dword ptr  0Ch

.text:75C4ADFC arg_8           = dword ptr  10h

.text:75C4ADFC

.text:75C4ADFC ; FUNCTION CHUNK AT .text:75C2A792 SIZE 00000026 BYTES

.text:75C4ADFC ; FUNCTION CHUNK AT .text:75CF6DF5 SIZE 0000006E BYTES

.text:75C4ADFC

.text:75C4ADFC                 mov     edi, edi

.text:75C4ADFE                 push    ebp

.text:75C4ADFF                 mov     ebp, esp

.text:75C4AE01                 mov     eax, 1014h

.text:75C4AE06                 call    __chkstk

.text:75C4AE0B                 mov     eax, ___security_cookie

.text:75C4AE10                 xor     eax, ebp

.text:75C4AE12                 mov     [ebp+var_4], eax

.text:75C4AE15                 mov     eax, [ebp+arg_4]

.text:75C4AE18                 and     [ebp+var_1010], 0

.text:75C4AE1F                 push    ebx

.text:75C4AE20                 mov     ebx, [ebp+hEnumHandle]

.text:75C4AE23                 push    esi

.text:75C4AE24                 mov     esi, [ebp+arg_8]

.text:75C4AE27                 mov     [ebp+var_1014], eax

.text:75C4AE2D                 mov     eax, 1000h

.text:75C4AE32                 push    edi

.text:75C4AE33                 mov     [ebp+uBytes], eax

.text:75C4AE39                 mov     [ebp+NextCacheEntryInfo.dwStructSize], eax

.text:75C4AE3F                 lea     eax, [ebp+uBytes]

.text:75C4AE45                 push    eax             ; lpdwNextCacheEntryInfoBufferSize

.text:75C4AE46                 lea     edi, [ebp+NextCacheEntryInfo]

.text:75C4AE4C                 mov     eax, edi

.text:75C4AE4E                 push    eax             ; lpNextCacheEntryInfo

.text:75C4AE4F                 push    ebx             ; hEnumHandle

.text:75C4AE50                 call    _FindNextUrlCacheEntryA@12 ; FindNextUrlCacheEntryA(x,x,x)

.text:75C4AE55                 test    eax, eax

.text:75C4AE57                 mov     [ebp+var_1008], eax

.text:75C4AE5D                 jz      loc_75C2A792

.text:75C4AE63

.text:75C4AE63 loc_75C4AE63:                       ; CODE XREF: FindNextUrlCacheEntryBinary(void *,_INTERNET_CACHE_ENTRY_INFOW *,ulong *)-2064Fj

.text:75C4AE63                 push    edi             ; struct _INTERNET_CACHE_ENTRY_INFOA *

.text:75C4AE64                 call    ?_ValidateHistoryData@@YGHPAU_INTERNET_CACHE_ENTRY_INFOA@@@Z ; _ValidateHistoryData(_INTERNET_CACHE_ENTRY_INFOA *)

.text:75C4AE69                 push    dword ptr [esi] ; int

.text:75C4AE6B                 push    [ebp+var_1014]  ; struct _INTERNET_CACHE_ENTRY_INFOW *

.text:75C4AE71                 push    edi             ; struct _INTERNET_CACHE_ENTRY_INFOA *

.text:75C4AE72                 call    ?CacheEntryInfoAToCacheEntryInfoW@@YGHPAU_INTERNET_CACHE_ENTRY_INFOA@@PAU_INTERNET_CACHE_ENTRY_INFOW@@H@Z ; CacheEntryInfoAToCacheEntryInfoW(_INTERNET_CACHE_ENTRY_INFOA *,_INTERNET_CACHE_ENTRY_INFOW *,int)

.text:75C4AE77                 test    eax, eax

.text:75C4AE79                 jnz     loc_75CF6E41

.text:75C4AE7F

.text:75C4AE7F loc_75C4AE7F:                       ; CODE XREF: FindNextUrlCacheEntryBinary(void *,_INTERNET_CACHE_ENTRY_INFOW *,ulong *)-20649j

.text:75C4AE7F                 ; FindNextUrlCacheEntryBinary(void *,_INTERNET_CACHE_ENTRY_INFOW *,ulong *)+AC056j

.text:75C4AE7F                 cmp     [ebp+var_1010], 0

.text:75C4AE86                 jnz     loc_75CF6E57

.text:75C4AE8C

.text:75C4AE8C loc_75C4AE8C:                       ; CODE XREF: FindNextUrlCacheEntryBinary(void *,_INTERNET_CACHE_ENTRY_INFOW *,ulong *)-20661j

.text:75C4AE8C                 ; FindNextUrlCacheEntryBinary(void *,_INTERNET_CACHE_ENTRY_INFOW *,ulong *)+AC062j

.text:75C4AE8C                 mov     ecx, [ebp+var_4]

.text:75C4AE8F                 mov     eax, [ebp+var_1008]

.text:75C4AE95                 pop     edi

.text:75C4AE96                 pop     esi

.text:75C4AE97                 xor     ecx, ebp

.text:75C4AE99                 pop     ebx

.text:75C4AE9A                 call    @__security_check_cookie@4 ; __security_check_cookie(x)

.text:75C4AE9F                 leave

.text:75C4AEA0                 retn    0Ch

.text:75C4AEA0 ?FindNextUrlCacheEntryBinary@@YGHPAXPAU_INTERNET_CACHE_ENTRY_INFOW@@PAK@Z endp

.text:75C4AEA0

.text:75C4AEA0 ; ---------------------------------------------------------------------------

.text:75C4AEA3                 align 8

.text:75C4AEA8 ; [00000006 BYTES: COLLAPSED FUNCTION FindNextUrlCacheEntryA(x,x,x). PRESS KEYPAD "+" TO EXPAND]

.text:75C4AEAE                 db 5 dup(90h)

.text:75C4AEB3

.text:75C4AEB3 ; =============== S U B R O U T I N E =======================================

.text:75C4AEB3

.text:75C4AEB3 ; Attributes: bp-based frame

.text:75C4AEB3

.text:75C4AEB3 ; private: long __thiscall CEnumSTATURL::RetrieveFirstUrlInfo(void)

.text:75C4AEB3 ?RetrieveFirstUrlInfo@CEnumSTATURL@@AAEJXZ proc near

.text:75C4AEB3                 ; CODE XREF: CEnumSTATURL::Next(ulong,_STATURL *,ulong *)-48p

.text:75C4AEB3

.text:75C4AEB3 var_8           = dword ptr -8

.text:75C4AEB3 var_4           = dword ptr -4

.text:75C4AEB3

.text:75C4AEB3 ; FUNCTION CHUNK AT .text:75CF6EC5 SIZE 0000007F BYTES

.text:75C4AEB3

.text:75C4AEB3                 mov     edi, edi

.text:75C4AEB5                 push    ebp

.text:75C4AEB6                 mov     ebp, esp

.text:75C4AEB8                 push    ecx

.text:75C4AEB9                 push    ecx

.text:75C4AEBA                 and     [ebp+var_4], 0

.text:75C4AEBE                 push    ebx

.text:75C4AEBF                 push    esi

.text:75C4AEC0                 push    edi

.text:75C4AEC1                 mov     ebx, 3E8h

.text:75C4AEC6                 mov     esi, ecx

.text:75C4AEC8                 push    ebx             ; uBytes

.text:75C4AEC9                 lea     edi, [esi+1064h]

.text:75C4AECF                 push    40h             ; uFlags

.text:75C4AED1                 mov     [edi], ebx

.text:75C4AED3                 call    ds:__imp__LocalAlloc@8 ; LocalAlloc(x,x)

.text:75C4AED9                 test    eax, eax

.text:75C4AEDB                 mov     [esi+1068h], eax

.text:75C4AEE1                 jz      loc_75CF6EC5

.text:75C4AEE7                 push    edi             ; int

.text:75C4AEE8                 lea     ecx, [esi+14h]

.text:75C4AEEB                 push    eax             ; struct _INTERNET_CACHE_ENTRY_INFOW *

.text:75C4AEEC                 push    ecx             ; lpSrcStr

.text:75C4AEED

.text:75C4AEED loc_75C4AEED:                       ; CODE XREF: CEnumSTATURL::RetrieveFirstUrlInfo(void)+AC05Cj

.text:75C4AEED                 call    ?FindFirstUrlCacheEntryBinary@@YGPAXPBGPAU_INTERNET_CACHE_ENTRY_INFOW@@PAK@Z ; FindFirstUrlCacheEntryBinary(ushort const *,_INTERNET_CACHE_ENTRY_INFOW *,ulong *)

.text:75C4AEF2                 test    eax, eax

.text:75C4AEF4                 mov     [esi+10h], eax

.text:75C4AEF7                 jz      loc_75CF6ED1

.text:75C4AEFD
```



```asm
.text:75C4AEFD loc_75C4AEFD:                  ; CODE XREF: CEnumSTATURL::RetrieveFirstUrlInfo(void)+AC019j
.text:75C4AEFD                                ; CEnumSTATURL::RetrieveFirstUrlInfo(void)+AC06Fj ...
.text:75C4AEFD             mov     eax, [edi]
.text:75C4AEFF             cmp     eax, ebx
.text:75C4AF01             ja      loc_75CF6F3D

.text:75C4AF07 loc_75C4AF07:                  ; CODE XREF: CEnumSTATURL::RetrieveFirstUrlInfo(void)+AC08Cj
.text:75C4AF07             mov     eax, [ebp+var_4]
.text:75C4AF0A             mov     [edi], ebx
.text:75C4AF0C             pop     edi
.text:75C4AF0D             pop     esi
.text:75C4AF0E             pop     ebx
.text:75C4AF0F             leave
.text:75C4AF10             retn
.text:75C4AF10 ?RetrieveFirstUrlInfo@CEnumSTATURL@@AAEJXZ endp

.text:75C4AF10 ; ---------------------------------------------------------------------------
.text:75C4AF11             db 5 dup(90h)

.text:75C4AF16 ; =============== S U B R O U T I N E =======================================
.text:75C4AF16 ; Attributes: bp-based frame
.text:75C4AF16 ; int __stdcall FindFirstUrlCacheEntryBinary(LPCWSTR lpSrcStr,struct _INTERNET_CACHE_ENTRY_INFOW *,int)
.text:75C4AF16 ?FindFirstUrlCacheEntryBinary@@YGPAXPBGPAU__INTERNET_CACHE_ENTRY_INFOW@@PAK@Z proc near
.text:75C4AF16                              ; CODE XREF: CEnumSTATURL::RetrieveFirstUrlInfo(void):loc_75C4AEEDp

.text:75C4AF16 var_1114           = dword ptr -1114h
.text:75C4AF16 uBytes             = dword ptr -1110h
.text:75C4AF16 hEnumHandle        = dword ptr -110Ch
.text:75C4AF16 FirstCacheEntryInfo= _INTERNET_CACHE_ENTRY_INFOA ptr -1108h
.text:75C4AF16 DstStr             = byte ptr -108h
.text:75C4AF16 var_4              = dword ptr -4
.text:75C4AF16 lpSrcStr           = dword ptr 8
.text:75C4AF16 arg_4              = dword ptr 0Ch
.text:75C4AF16 arg_8              = dword ptr 10h

.text:75C4AF16 ; FUNCTION CHUNK AT .text:75C4AFCC SIZE 0000000B BYTES
.text:75C4AF16 ; FUNCTION CHUNK AT .text:75CF6D67 SIZE 0000008E BYTES

.text:75C4AF16             mov     edi, edi
.text:75C4AF18             push    ebp
.text:75C4AF19             mov     ebp, esp
.text:75C4AF1B             mov     eax, 1114h
.text:75C4AF20             call    __chkstk
.text:75C4AF25             mov     eax, ___security_cookie
.text:75C4AF2A             xor     eax, ebp
.text:75C4AF2C             mov     [ebp+var_4], eax
.text:75C4AF2F             mov     eax, [ebp+lpSrcStr]
.text:75C4AF32             push    ebx
.text:75C4AF33             mov     ebx, [ebp+arg_4]
.text:75C4AF36             push    esi
.text:75C4AF37             mov     esi, [ebp+arg_8]
.text:75C4AF3A             push    edi
.text:75C4AF3B             push    104h            ; cchMultiByte
.text:75C4AF40             lea     ecx, [ebp+DstStr]
.text:75C4AF46             push    ecx             ; lpDstStr
.text:75C4AF47             push    eax             ; lpSrcStr
.text:75C4AF48             call    _SHUnicodeToAnsi@12 ; SHUnicodeToAnsi(x,x,x)
.text:75C4AF4D             and     [ebp+var_1114], 0
.text:75C4AF54             mov     eax, 1000h
.text:75C4AF59             mov     [ebp+uBytes], eax
.text:75C4AF5F             mov     [ebp+FirstCacheEntryInfo.dwStructSize], eax
.text:75C4AF65             lea     eax, [ebp+uBytes]
.text:75C4AF6B             push    eax             ; lpdwFirstCacheEntryInfoBufferSize
.text:75C4AF6C             lea     edi, [ebp+FirstCacheEntryInfo]
.text:75C4AF72             mov     eax, edi
.text:75C4AF74             push    eax             ; lpFirstCacheEntryInfo
.text:75C4AF75             lea     eax, [ebp+DstStr]
.text:75C4AF7B             push    eax             ; lpszUrlSearchPattern
.text:75C4AF7C             call    _FindFirstUrlCacheEntryA@12 ; FindFirstUrlCacheEntryA(x,x,x)
.text:75C4AF81             test    eax, eax
.text:75C4AF83             mov     [ebp+hEnumHandle], eax
.text:75C4AF89             jz      loc_75CF6D67

.text:75C4AF8F loc_75C4AF8F:                   ; CODE XREF: FindFirstUrlCacheEntryBinary(ushort const *,_INTERNET_CACHE_ENTRY_INFOW *,ulong *)+BDj
.text:75C4AF8F             push    edi           ; struct _INTERNET_CACHE_ENTRY_INFOA *
.text:75C4AF90             call    ?_ValidateHistoryData@@YGHPAU__INTERNET_CACHE_ENTRY_INFOA@@@Z ; _ValidateHistoryData(_INTERNET_CACHE_ENTRY_INFOA *)
.text:75C4AF95             push    dword ptr [esi] ; int
.text:75C4AF97             push    ebx           ; struct _INTERNET_CACHE_ENTRY_INFOW *
.text:75C4AF98             push    edi           ; struct _INTERNET_CACHE_ENTRY_INFOA *
.text:75C4AF99             call    ?CacheEntryInfoAToCacheEntryInfoW@@YGHPAU__INTERNET_CACHE_ENTRY_INFOA@@PAU__INTERNET_CACHE_ENTRY_INFOW@@H@Z ; CacheEntryInfoAToCacheEntryInfoW(_INTERNET_CACHE_ENTRY_INFOA *,_INTERNET_CACHE_ENTRY_INFOW *,int)
.text:75C4AF9E             mov     ebx, eax
.text:75C4AFA0             test    ebx, ebx
.text:75C4AFA2             jnz     loc_75CF6DC8

.text:75C4AFA8 loc_75C4AFA8:                   ; CODE XREF: FindFirstUrlCacheEntryBinary(ushort const *,_INTERNET_CACHE_ENTRY_INFOW *,ulong *)+BFj
.text:75C4AFA8                              ; FindFirstUrlCacheEntryBinary(ushort const *,_INTERNET_CACHE_ENTRY_INFOW *,ulong *)+ABECEj
.text:75C4AFA8             cmp     [ebp+var_1114], 0
.text:75C4AFAF             jnz     loc_75CF6DE9

.text:75C4AFB5 loc_75C4AFB5:                   ; CODE XREF: FindFirstUrlCacheEntryBinary(ushort const *,_INTERNET_CACHE_ENTRY_INFOW *,ulong *)+ABE5Aj
.text:75C4AFB5                              ; FindFirstUrlCacheEntryBinary(ushort const *,_INTERNET_CACHE_ENTRY_INFOW *,ulong *)+ABEDAj
.text:75C4AFB5             mov     ecx, [ebp+var_4]
.text:75C4AFB8             mov     eax, [ebp+hEnumHandle]
.text:75C4AFBE             pop     edi
.text:75C4AFBF             pop     esi
.text:75C4AFC0             xor     ecx, ebp
.text:75C4AFC2             pop     ebx
.text:75C4AFC3             call    @__security_check_cookie@4 ; __security_check_cookie(x)
.text:75C4AFC8             leave
.text:75C4AFC9             ret     0Ch
.text:75C4AFC9 ?FindFirstUrlCacheEntryBinary@@YGPAXPBGPAU__INTERNET_CACHE_ENTRY_INFOW@@PAK@Z endp
```

这里根据 PDB 信息中的 `CacheEntryInfoAToCacheEntryInfoW` 可以猜测出这里就是进行 Unicode-〉ANSI 的转换操作，其中

```asm
push    edi           ; struct _INTERNET_CACHE_ENTRY_INFOA *
.text:75C4AE72        call    ?CacheEntryInfoAToCacheEntryInfoW@@YGHPAU__INTERNET_CACH
```

的参数 `EDI` 来自

```asm
.text:75C4AE46        lea     edi, [ebp+NextCacheEntryInfo]
.text:75C4AE4C        mov     eax, edi
```



### 汇编代码片段

```
.text:75C4AE4E             push    eax             ; lpNextCacheEntryInfo
.text:75C4AE4F             push    ebx             ; hEnumHandle
.text:75C4AE50             call    _FindNextUrlCacheEntryA@12 ; FindNextUrlCacheEntryA(x,x,x)
```

那么问题也就解决了

### 函数分析与逆向

回到 `CUrlHistory::s_GenerateSTATURL` 函数，其中 `ARG.2` 我们就推测出应该是 `LPINTERNET_CACHE_ENTRY_INFOW`。

因此：

```
public: static class CHistoryData * __stdcall CHistoryData::s_GetHistoryData(struct _INTERNET_CACHE_ENTRY_INFOW *)
```

```
.text:75CD8434 ?s_GetHistoryData@CHistoryData@@SGPAV1@PAU_INTERNET_CACHE_ENTRY_INFOW@@@Z proc near
.text:75CD8434                        ; CODE XREF: CUrlHistory::s_QueryUrlCommon(ushort const *,ushort const *,ulong,_STATURL *)+9Ap
.text:75CD8434                        ; CommitHistoryData(CInternetShortcut *,IPropertyStore *)+F1p ...
.text:75CD8434 arg_0             = dword ptr  8
.text:75CD8434 ; FUNCTION CHUNK AT .text:75C4B1B3 SIZE 00000007 BYTES
.text:75CD8434 ; FUNCTION CHUNK AT .text:75CF6D17 SIZE 00000012 BYTES
.text:75CD8434                    mov     edi, edi
.text:75CD8436                    push    ebp
.text:75CD8437                    mov     ebp, esp
.text:75CD8439                    mov     eax, [ebp+arg_0]
.text:75CD843C                    mov     eax, [eax+_INTERNET_CACHE_ENTRY_INFOW.lpHeaderInfo]
.text:75CD843F                    test    eax, eax
.text:75CD8441                     jz      short loc_75CD845E
.text:75CD8443                    movzx   ecx, word ptr [eax]
.text:75CD8446                    cmp     ecx, 14h
.text:75CD8449                    jz      loc_75CF6D17
.text:75CD844F loc_75CD844F:                       ; CODE XREF: CHistoryData::s_GetHistoryData(_INTERNET_CACHE_ENTRY_INFOW *)+1E8E8j
.text:75CD844F                        ; CHistoryData::s_GetHistoryData(_INTERNET_CACHE_ENTRY_INFOW *)+1E8F0j
.text:75CD844F                    test    eax, eax
.text:75CD8451                    jz      short loc_75CD845E
.text:75CD8453                    cmp     word ptr [eax+2], 2
.text:75CD8458                    jnz     loc_75C4B1B3
.text:75CD845E loc_75CD845E:                       ; CODE XREF: CHistoryData::s_GetHistoryData(_INTERNET_CACHE_ENTRY_INFOW *)-8D27Fj
.text:75CD845E                        ; CHistoryData::s_GetHistoryData(_INTERNET_CACHE_ENTRY_INFOW *)+Dj ...
.text:75CD845E                    pop     ebp
.text:75CD845F                    retn    4
.text:75CD845F ?s_GetHistoryData@CHistoryData@@SGPAV1@PAU_INTERNET_CACHE_ENTRY_INFOW@@@Z endp
```

显然是返回 `lpHeaderInfo`

### 逆向 `_FindExtra` 函数

逆向：`CHistoryData::_FindExtra(uint)`

```
?_FindExtra@CHistoryData@@QBEPBUHISTEXTRA@@I@Z proc near
.text:75CD8408                    ; CODE XREF: CUrlHistory::AddUrlAndNotifyCP(ushort const *,ushort const *,ulong,int,IOleCommandTarget *,IUnknown *,uint *)-40920p
.text:75CD8408                    ; CHistoryData::_HasFragment(ushort const *)+Ap ...
.text:75CD8408 arg_0             = dword ptr  8
.text:75CD8408 ; FUNCTION CHUNK AT .text:75CD8475 SIZE 00000004 BYTES
.text:75CD8408                    mov     edi, edi
.text:75CD840A                    push    ebp
.text:75CD840B                    mov     ebp, esp
.text:75CD840D                    mov     eax, [ecx]
.text:75CD840F                    mov     edx, 0FFFFh
.text:75CD8414                    push    esi
.text:75CD8415                    and     eax, edx
.text:75CD8417 loc_75CD8417:                       ; CODE XREF: CHistoryData::_FindExtra(uint)+20j
.text:75CD8417                    add     eax, ecx
.text:75CD8419                    jz      short loc_75CD8475
.text:75CD841B                    mov     ecx, [eax]
.text:75CD841D                    and     ecx, edx
.text:75CD841F                    jz      short loc_75CD8475
.text:75CD8421                    movzx   esi, byte ptr [eax+2]
.text:75CD8425                    cmp     esi, [ebp+arg_0]
.text:75CD8428                    jnz     short loc_75CD8417
.text:75CD842A loc_75CD842A:                       ; CODE XREF: CHistoryData::_FindExtra(uint)+6Fj
.text:75CD842A                    pop     esi
.text:75CD842B                    pop     ebp
.text:75CD842C                    retn    4
.text:75CD842C ?_FindExtra@CHistoryData@@QBEPBUHISTEXTRA@@I@Z endp
```

调用参数，以及实际文本指针偏移：

```
test    eax, eax
.text:75CD8B11            jz      short loc_75CD8AE7
.text:75CD8B13            cmp     byte ptr [eax+3], 1Fh
.text:75CD8B17            jnz     short loc_75CD8AE7
.text:75CD8B19            lea     ecx, [esi+8]
.text:75CD8B1C            push    ecx             ; int
.text:75CD8B1D            add     eax, 4
.text:75CD8B20            push    eax             ; lpString
```

代码如下：

```cpp
//Powered by fOx,转载请保留作者信息 : Email Destiny-fOx@hotmail.com

LPBYTE p = (LPBYTE)(pInfo->lpHeaderInfo);
    p += *(PWORD)p;
    if(p != NULL)
        if(p[2] != 0x10)
            p = NULL;
        else
        {
            if(p[3] == 0x1F)
                p += 4;
            else
                p = NULL;
        }
```

至此我们就得到了文本指针，但如果跟踪发现

即使你使用的是 `FindNextUrlCacheEntryA` 函数，这个文本也是一个 Unicode。当然我想具体原因应该是这里的 `HeaderInfo` 应该是存在硬盘上的原始数据，当然这里就不会进行文本转换。因此默认格式是 Unicode。

### 完整代码示例

贴一个完整代码：

```cpp
//Powered by fOx,转载请保留作者信息 : Email Destiny-fOx@hotmail.com

void GetUrlHistoryMethod2()
{
    HANDLE hFind = NULL;
    DWORD dwSize = NULL;
    hFind = ::FindFirstUrlCacheEntryA("visited:",NULL,&dwSize);
    HANDLE hHeap = ::HeapCreate(NULL,0x100,0x10000000);
    LPINTERNET_CACHE_ENTRY_INFOA pInfo = (LPINTERNET_CACHE_ENTRY_INFOA)::HeapAlloc(hHeap,LPTR,dwSize);
    hFind = ::FindFirstUrlCacheEntryA("visited:",pInfo,&dwSize);
    if(hFind)
    {
        int n = 0;
        while(pInfo != NULL)
        {
            /*
            typedef struct _INTERNET_CACHE_ENTRY_INFOA {
            DWORD dwStructSize;         // version of cache system.0
            LPSTR   lpszSourceUrlName;    // embedded pointer to the URL name string.4
            LPSTR   lpszLocalFileName;    // embedded pointer to the local file name.8
            DWORD CacheEntryType;         // cache type bit mask.C
            DWORD dwUseCount;             // current users count of the cache entry.10
            DWORD dwHitRate;              // num of times the cache entry was retrieved.
            DWORD dwSizeLow;             // low DWORD of the file size.
            DWORD dwSizeHigh;             // high DWORD of the file size.
            FILETIME LastModifiedTime;    // last modified time of the file in GMT format.
            FILETIME ExpireTime;          // expire time of the file in GMT format
            FILETIME LastAccessTime;       // last accessed time in GMT format
            FILETIME LastSyncTime;         // last time the URL was synchronized
            // with the source
```



```
  LPSTR   lpHeaderInfo;        // embedded pointer to the header info.

  DWORD dwHeaderInfoSize;     // size of the above header.

  LPSTR   lpszFileExtension;  // File extension used to retrive the urldata as a file.

  union {                     // Exemption delta from last access time.

  DWORD dwReserved;

  DWORD dwExemptDelta;

  };                        // Exemption delta from last access

  } INTERNET_CACHE_ENTRY_INFOA, *LPINTERNET_CACHE_ENTRY_INFOA;

  */

  LPBYTE p = (LPBYTE)(pInfo->lpHeaderInfo);

  p += *(PWORD)p;

  if(p != NULL)

     if(p[2] != 0x10)

        p = NULL;

     else

     {

         if(p[3] == 0x1F)
           p += 4;

        else
           p = NULL;

     }

  printf("[%d] SourceUrlName : %s ,LocalFileName : %s Title : %S \r\n"
    ,n,pInfo->lpszSourceUrlName,pInfo->lpszLocalFileName,p);

  n++;

  pInfo = NULL;

  dwSize = 1000;

  pInfo = (LPINTERNET_CACHE_ENTRY_INFOA)::HeapAlloc(hHeap,LPTR,dwSize);

  if(pInfo == NULL)

  {

     DWORD dwError = ::GetLastError();

     printf("HeapAlloc Error : %d\r\n",dwError);

  }

  BOOL bSus = ::FindNextUrlCacheEntryA(hFind,pInfo,&dwSize);

  if(bSus == FALSE)

  {

     pInfo = NULL;

     DWORD dwError = ::GetLastError();

     printf("Error : %d\r\n",dwError);

  }

  }

  ::FindCloseUrlCache(hFind);

  ::HeapDestroy(hHeap);

 }

}
```

以上代码在`Windows Vista Ultimate.` `IE 7.0`测试成功。至于其他版本是否也正确希望大家帮忙验证。当然`Windows Xp`也是可以的。至于这种方法的优点也比较显然，首先是节约了必要的资源开销，至少不会加载`OLE`库，和`COM`自动化，初始化这两个库实际上也是比较占用时间的。其次加快调用速度，避免内存重复复制带来的开销，毕竟一般电脑上的历史数据是非常多的，即便是那么一点点也是有好处的。不过缺点就是`HeaderInfo`是未文档化的，将来的版本可能会改变。

Powered by `fOx`

Email:`Destiny-fOx@hotmail.com`

转载请保留作者信息,谢谢

`4dd466d0e1e0cc123afbe504da2462ca`
