
# [原创]破解 iTunes的登录协议。

- **软件**：iTunes
- **作者**：fosom
- **日期**：2013.4.15
- **前提**：OD，IDA，iTunes

最近很多看雪的朋友找我看看iTunes怎么分析，今天又恰好没事，就看了看。

## iTunes登录协议-描述

```
<plist version="1.0">

<dict>

<key>appleId</key>

<string>asdfasdf@asdf.com</string>

<key>attempt</key>

<integer>1</integer>

<key>createSession</key>

<string>true</string>

<key>guid</key>

<string>4AEF365B.CF847F94.08E11EF5.FDD88C5B.09DA8172.65FAFEFB.6A193139</string>

<key>machineName</key>

<string>2012-20130221HH</string>

<key>password</key>

<string>qweqweqwe</string>

<key>why</key>

<string>signIn</string>

</dict>

</plist>
```

## 登录过程分析

在此，只提供关键Call的分析过程：（也就是登录协议的xml的组合过程）

```
.text:1081B6F0 LoginServer     proc near               ; CODE XREF: sub_1082A6A0+3F7p
.text:1081B6F0
.text:1081B6F0 var_244         = dword ptr -244h
.text:1081B6F0 var_240         = dword ptr -240h
.text:1081B6F0 var_23C         = dword ptr -23Ch
.text:1081B6F0 var_238         = dword ptr -238h
.text:1081B6F0 var_234         = dword ptr -234h
.text:1081B6F0 var_230         = dword ptr -230h
.text:1081B6F0 var_229         = byte ptr -229h
.text:1081B6F0 var_228         = dword ptr -228h
.text:1081B6F0 var_221         = byte ptr -221h
.text:1081B6F0 var_220         = dword ptr -220h
.text:1081B6F0 var_21C         = dword ptr -21Ch
.text:1081B6F0 var_218         = word ptr -218h
.text:1081B6F0 var_18          = word ptr -18h
.text:1081B6F0 var_16          = byte ptr -16h
.text:1081B6F0 var_10          = dword ptr -10h
.text:1081B6F0 var_C           = dword ptr -0Ch
.text:1081B6F0 var_4           = dword ptr -4
.text:1081B6F0
.text:1081B6F0                 push    ebp
.text:1081B6F1                 mov     ebp, esp
.text:1081B6F3                 push    0FFFFFFFFh
.text:1081B6F5                 push    offset loc_10FC8DDB
.text:1081B6FA                 mov     eax, large fs:0
.text:1081B700                 push    eax
.text:1081B701                 sub     esp, 23Ch
.text:1081B707                 mov     eax, dword_1131C190
.text:1081B70C                 xor     eax, ebp
.text:1081B70E                 mov     [ebp+var_10], eax
.text:1081B711                 push    ebx
.text:1081B712                 push    esi
.text:1081B713                 push    edi
.text:1081B714                 push    eax
.text:1081B715                 lea     eax, [ebp+var_C]
.text:1081B718                 mov     large fs:0, eax
.text:1081B71E                 xor     edi, edi
.text:1081B720                 xor     ebx, ebx
.text:1081B722                 mov     [ebp+var_244], ecx
.text:1081B728                 mov     esi, edx
.text:1081B72A                 mov     [ebp+var_240], edi
.text:1081B730                 mov     [ebp+var_23C], ebx
.text:1081B736                 mov     [ebp+var_4], edi
.text:1081B739                 mov     al, [esi+0CB8h]
.text:1081B73F                 and     al, 1
.text:1081B741                 mov     [ebp+var_221], al
.text:1081B747                 mov     eax, [esi+0CB8h]
.text:1081B74D                 push    edi
.text:1081B74E                 shr     eax, 1
.text:1081B750                 push    esi
.text:1081B751                 and     al, 1
.text:1081B753                 push    ecx
.text:1081B754                 mov     ecx, dword_1156CA4C
.text:1081B75A                 mov     [ebp+var_229], al
.text:1081B760                 call    sub_108186D0
.text:1081B765                 mov     [ebp+var_238], eax
.text:1081B76B                 cmp     eax, edi
.text:1081B76D                 jnz     short loc_1081B77A
.text:1081B76F                 mov     edi, [esi+8A0h]
.text:1081B775                 jmp     loc_1081BBD0
.text:1081B77A ; ---------------------------------------------------------------------------
.text:1081B77A
.text:1081B77A loc_1081B77A:                           ; CODE XREF: LoginServer+7Dj
.text:1081B77A                 mov     ecx, ds:kCFTypeDictionaryValueCallBacks
.text:1081B780                 mov     edx, ds:kCFTypeDictionaryKeyCallBacks
.text:1081B786                 push    ecx
.text:1081B787                 push    edx
.text:1081B788                 push    edi
.text:1081B789                 push    edi
.text:1081B78A                 call    ds:CFDictionaryCreateMutable
.text:1081B790                 add     esp, 10h
.text:1081B793                 mov     [ebp+var_228], eax
.text:1081B799                 cmp     eax, edi
.text:1081B79B                 jnz     short loc_1081B7A7
.text:1081B79D                 mov     edi, 0FFFFFF94h
.text:1081B7A2                 jmp     loc_1081BBAE
.text:1081B7A7 ; ---------------------------------------------------------------------------
.text:1081B7A7
.text:1081B7A7 loc_1081B7A7:                           ; CODE XREF: LoginServer+ABj
.text:1081B7A7                 mov     eax, [esi+8]
.text:1081B7AA                 mov     ecx, [esi+0Ch]
.text:1081B7AD                 push    offset aDsid    ; "dsid"
.text:1081B7B2                 mov     [ebp+var_234], eax
.text:1081B7B8                 mov     [ebp+var_230], ecx
.text:1081B7BE                 call    ds:__CFStringMakeConstantString
.text:1081B7C4                 mov     ebx, eax
.text:1081B7C6                 add     esp, 4
.text:1081B7C9                 cmp     ebx, edi
.text:1081B7CB                 jz      short loc_1081B816
.text:1081B7CD                 mov     edx, [ebp+var_234]
.text:1081B7D3                 or      edx, [ebp+var_230]
.text:1081B7D9                 jz      short loc_1081B816
.text:1081B7DB                 mov     ecx, dword_11565448
.text:1081B7E1                 lea     eax, [ebp+var_234]
.text:1081B7E7                 push    eax
.text:1081B7E8                 push    4
.text:1081B7EA                 push    ecx
.text:1081B7EB                 call    ds:CFNumberCreate
.text:1081B7F1                 mov     edi, eax
.text:1081B7F3                 add     esp, 0Ch
.text:1081B7F6                 test    edi, edi
.text:1081B7F8                 jz      short loc_1081B816
.text:1081B7FA                 mov     edx, [ebp+var_228]
.text:1081B800                 push    edi
.text:1081B801                 push    ebx
.text:1081B802                 push    edx
.text:1081B803                 call    ds:CFDictionarySetValue
.text:1081B809                 add     esp, 0Ch
.text:1081B80C                 push    edi
.text:1081B80D                 call    ds:CFRelease
.text:1081B813                 add     esp, 4
.text:1081B816
.text:1081B816 loc_1081B816:                           ; CODE XREF: LoginServer+DBj
.text:1081B816                                         ; LoginServer+E9j ...
.text:1081B816                 push    offset aAppleid_0 ; 这里，就是帐号： asdfasdf@asdf.com
.text:1081B81B                 lea     edi, [esi+8B0h]
.text:1081B821                 call    ds:__CFStringMakeConstantString
.text:1081B827                 mov     ebx, eax
.text:1081B829                 add     esp, 4
.text:1081B82C                 test    ebx, ebx
.text:1081B82E                 jz      short loc_1081B864
.text:1081B830                 test    edi, edi
.text:1081B832                 jz      short loc_1081B864
.text:1081B834                 cmp     word ptr [edi], 0
.text:1081B838                 jz      short loc_1081B864
.text:1081B83A                 mov     eax, edi
.text:1081B83C                 call    GetCFString     ; ；宽字符转字符；Widestring --> char*
.text:1081B841                 mov     edi, eax
.text:1081B843                 test    edi, edi
.text:1081B845                 jz      short loc_1081B864
.text:1081B847                 mov     eax, [ebp+pXMLNode] ; 042B1170  E8 E6 78 01 80 12 00 01 09 00 00 00 00 00 00 00
.text:1081B847                                         ; 042B1180  00 00 00 00 00 00 01 00 50 02 76 01 00 00 00 00
.text:1081B84D                 push    eax             ; pXMLNode
```
