
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
