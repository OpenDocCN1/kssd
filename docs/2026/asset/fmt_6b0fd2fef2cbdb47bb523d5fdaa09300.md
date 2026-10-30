
```
.text:77F2EBC6                 mov     ebx, [ebp+hdc]
.text:77F2EBC9                 mov     [ebp+var_4], eax
.text:77F2EBCC                 mov     eax, [ebp+lphandletable]
.text:77F2EBCF                 push    esi
.text:77F2EBD0                 push    eax
.text:77F2EBD1                 mov     esi, ecx
.text:77F2EBD3                 mov     [ebp+var_258], ebx
.text:77F2EBD9                 call    MRCOLORMATCHTOTARGET::bCheckRecord(tagHANDLETABLE *) ;这个函数里规定[esi+0x14]+[esi+0x10]+0x1B==[esi+0x4]
.text:77F2EBDE                 test    eax, eax
.text:77F2EBE0                 jz      loc_77F2ED19
.text:77F2EBE6                 mov     eax, [esi+8]
.text:77F2EBE9                 xor     ecx, ecx
.text:77F2EBEB                 inc     ecx
.text:77F2EBEC                 cmp     eax, ecx
.text:77F2EBEE                 push    edi
.text:77F2EBEF                 mov     [ebp+var_254], ecx
.text:77F2EBF5                 jnz     loc_77F2ED03    ; ENHMETARECORD.iType==EMR_HEADER ?
.text:77F2EBFB                 test    [esi+0Ch], cl
.text:77F2EBFE                 jz      loc_77F2ECB2    ; Record size是否是偶数
```

从上面的代码知道要触发漏洞必须符合的几个条件：

1. `ENHMETARECORD.iType==EMR_HEADER`，也就是第一个RECORD。
2. 当`esi`指向上图蓝色的位置时，`([esi+0x14]+[esi+0x10]+0x1B)& 0FFFFFFFCh==[esi+0x4]`。
3. Record size必须是偶数。

```
.text:77F2ECB2 loc_77F2ECB2:                           ; CODE XREF: MRCOLORMATCHTOTARGET::bPlay(void *,tagHANDLETABLE *,uint)+49 j
.text:77F2ECB2                 mov     edi, 10002h
.text:77F2ECB7                 push    edi             ; const_10002
.text:77F2ECB8                 push    4               ; const_4
.text:77F2ECBA                 lea     eax, [esi+18h]
.text:77F2ECBD                 push    eax             ; lp_sz
.text:77F2ECBE                 push    ebx             ; hdc
.text:77F2ECBF                 call    IcmGetColorSpaceByName(x,x,x,x)
.text:77F2ECC4                 mov     ebx, eax        ;必须返回0
.text:77F2ECC6                 test    ebx, ebx
.text:77F2ECC8                 jnz     short loc_77F2ECE2
.text:77F2ECCA                 push    edi             ; int
.text:77F2ECCB                 push    4               ; int
.text:77F2ECCD                 lea     eax, [esi+18h]
.text:77F2ECD0                 push    eax             ; 指向RECORD的内容
.text:77F2ECD1                 push    [ebp+var_258]   ; int
.text:77F2ECD7                 call    IcmCreateColorSpaceByName(x,x,x,x)  ;有漏洞的函数
```

`IcmGetColorSpaceByName`必须返回0。

```
.text:77F0335B __stdcall IcmCreateColorSpaceByName(x, x, x, x) proc near
.text:77F0335B                                         ; CODE XREF: IcmUpdateLocalDCColorSpace(x,x)+1043 p
.text:77F0335B                                         ; IcmUpdateLocalDCColorSpace(x,x)+150B7 p ...
.text:77F0335B
.text:77F0335B var_250         = dword ptr -250h
.text:77F0335B var_24C         = dword ptr -24Ch
.text:77F0335B var_248         = dword ptr -248h
.text:77F0335B var_244         = dword ptr -244h
.text:77F0335B var_240         = dword ptr -240h
.text:77F0335B String1         = word ptr -20Ch
.text:77F0335B var_4           = dword ptr -4
.text:77F0335B arg_0           = dword ptr  8
.text:77F0335B lpString2       = dword ptr  0Ch
.text:77F0335B arg_8           = dword ptr  10h
.text:77F0335B arg_C           = dword ptr  14h
.text:77F0335B
.text:77F0335B                 mov     edi, edi
.text:77F0335D                 push    ebp
.text:77F0335E                 mov     ebp, esp
.text:77F03360                 sub     esp, 250h
.text:77F03366                 mov     eax, ___security_cookie;使用了GS
.text:77F0336B                 mov     edx, [ebp+lpString2]
.text:77F0336E                 push    esi
.text:77F0336F                 mov     esi, [ebp+arg_0]
.text:77F03372                 mov     [ebp+var_4], eax
.text:77F03375                 push    edi
.text:77F03376                 xor     eax, eax
.text:77F03378                 mov     ecx, 93h
.text:77F0337D                 lea     edi, [ebp+var_250]
.text:77F03383                 rep stosd
.text:77F03385                 and     [ebp+var_244], eax
.text:77F0338B                 mov     eax, [ebp+arg_8]
.text:77F0338E                 mov     [ebp+var_240], eax
.text:77F03394                 push    edx             ; lpString2
.text:77F03395                 lea     eax, [ebp+String1]
.text:77F0339B                 push    eax             ; lpString1
.text:77F0339C                 mov     [ebp+var_250], 50534F43h
.text:77F033A6                 mov     [ebp+var_24C], 400h
.text:77F033B0                 mov     [ebp+var_248], 24Ch
.text:77F033BA                 call    ds:lstrcpyW(x,x) ; 如果第一个RECORD的内容足够大就可以溢出。
.text:77F033C0                 push    [ebp+arg_C]
.text:77F033C3                 lea     eax, [ebp+var_250]
.text:77F033C9                 push    0
.text:77F033CB                 push    eax
.text:77F033CC                 push    esi
.text:77F033CD                 call    IcmCreateColorSpaceByColorSpace(x,x,x,x)
.text:77F033D2                 mov     ecx, [ebp+var_4]
.text:77F033D5                 pop     edi
.text:77F033D6                 pop     esi
.text:77F033D7                 call    ___security_check_cookie(x)
.text:77F033DC                 leave
.text:77F033DD                 retn    10h
.text:77F033DD __stdcall IcmCreateColorSpaceByName(x, x, x, x) endp
```

本人菜鸟一只，请指教，谢谢。

[\[招生\]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [emf.jpg](javascript:void(0);) （10.73kb，1013次下载）
- [poc.jpg](javascript:void(0);) （32.71kb，1023次下载）
