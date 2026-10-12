
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
