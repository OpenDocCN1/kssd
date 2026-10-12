
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
