
# [原创]MS08-021漏洞分析

Author:dge

我从分析这个漏洞的过程中重新认识到一些东西，写下来分享给像我一样的人们。

在分析文件格式漏洞时只用调试工具跟踪有时候很费劲。如果POC可以顺利执行shellcode，跟踪比较简单，但如果POC执行不到shellcode，就很难去解决，当然我们可以通过补丁比较确定出几个可疑函数进行重点跟踪，当然前提是你得能确定出来，如果补丁对代码的改动很大，要确定出可疑函数实非易事，即使能确定出几个来，对他们跟踪也是非常麻烦的事情，让复杂的问题变的简单，那肯定是最理想的。

认识PaiMei这个工具是在failwest的书里，虽然读完那本书已经很久了，但一直没用过这个工具，直到前几天分析这个漏洞，在分析未果的情况下，我想起了PaiMei,这才体会到了它的强大，这也是我想写这篇文章的原因。

MS08-021有两个漏洞，这是其中的一个，这个漏洞的POC在很久之前就有人公布了，这是POC的地址：`585K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3#2A6L8s2M7H3M7X3#2Q4x3X3g2U0L8$3#2Q4x3V1k6W2P5s2m8D9L8$3W2@1M7#2)9J5c8U0f1@1y4o6u0Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0k6Q4z5f1y4Q4z5o6W2Q4c8e0c8Q4b7V1q4Q4z5o6k6b7e0@1y4Q4c8e0g2Q4z5e0u0Q4z5p5y4b7j5h3W2y4k6h3W2Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0g2Q4z5o6S2Q4z5o6k6Q4c8e0k6Q4z5f1g2Q4z5e0m8Q4c8e0S2Q4b7V1k6Q4z5e0W2Q4c8e0c8Q4b7U0S2Q4b7f1q4Q4c8e0k6Q4b7V1y4Q4z5p5k6Q4c8e0k6Q4b7U0c8Q4z5f1g2Q4c8e0g2Q4b7U0m8Q4b7U0q4Q4c8e0g2Q4z5p5k6Q4z5e0S2Q4c8e0g2Q4b7V1g2Q4z5e0N6Q4c8e0N6Q4b7f1g2Q4z5o6m8Q4c8e0g2Q4z5p5c8Q4z5e0g2Q4c8e0c8Q4b7V1q4Q4z5o6k6Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0S2Q4b7V1k6Q4z5e0W2Q4c8e0c8Q4b7U0S2Q4b7f1q4Q4c8e0k6Q4b7V1y4Q4z5p5k6Q4c8e0k6Q4b7U0c8Q4z5f1g2Q4c8e0g2Q4z5p5k6Q4z5e0q4Q4c8e0N6Q4z5e0c8Q4z5f1k6Q4c8e0g2Q4z5f1y4Q4b7e0S2Y4k6r3V1K6x3W2)9J5k6h3c8D9L8q4!0q4y4q4!0n7z5q4!0m8c8q4!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4W2)9&6z5q4!0m8c8W2!0q4y4g2)9^5y4g2!0n7z5q4!0q4y4g2)9&6c8g2)9^5b7W2!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4W2!0m8x3q4)9^5z5q4!0q4y4W2!0n7b7g2!0m8x3W2!0q4y4g2)9^5y4#2!0n7b7g2!0q4x3#2)9^5x3q4)9^5x3R3`.`.

用PaiMei和od我们可以轻松地确定出漏洞触发路径。

`PlayEnhMetaFileRecord`

  |__`MRCOLORMATCHTOTARGET::bPlay`

       |__`IcmCreateColorSpaceByName`

            |__`lstrcpyW`

先简单了解下EMF文件格式

![](https://bbs.kanxue.com/upload/attach/200812/105705_t5z5nnp9mo0q4cz.jpg)

## Header结构

```c
typedef struct tagENHMETAHEADER {
  DWORD iType;
  DWORD nSize;
  RECTL rclBounds;
  RECTL rclFrame;
  DWORD dSignature;
  DWORD nVersion;
  DWORD nBytes;
  DWORD nRecords;
  WORD  nHandles;
  WORD  sReserved;
  DWORD nDescription;
  DWORD offDescription;
  DWORD nPalEntries;
  SIZEL szlDevice;
  SIZEL szlMillimeters;
#if (WINVER >= 0x0400)
  DWORD cbPixelFormat;
  DWORD offPixelFormat;
  DWORD bOpenGL;
#endif /* WINVER >= 0x0400 */
#if (WINVER >= 0x0500)
  SIZEL szlMicrometers;
#endif /* WINVER >= 0x0500 */
} ENHMETAHEADER, *PENHMETAHEADER;
```

## RECORD结构

```c
typedef struct tagENHMETARECORD
{
    DWORD   iType;              // Record type EMR_XXX
    DWORD   nSize;              // Record size in bytes
    DWORD   dParm[1];           // DWORD Array of parameters
} ENHMETARECORD;
```

## 函数原型

```c
BOOL PlayEnhMetaFileRecord(
  HDC hdc,                              // handle to DC
  LPHANDLETABLE lpHandletable,          // metafile handle table
  CONST ENHMETARECORD *lpEnhMetaRecord, // metafile record
  UINT nHandles                         // count of handles
);
```

POC截图：

![](https://bbs.kanxue.com/upload/attach/200812/105705_8vcispv4mkoxl8o.jpg)

下边分析下这个漏洞：

```asm
.text:77EFEF66 __stdcall PlayEnhMetaFileRecord(x, x, x, x) proc near
.text:77EFEF66                                         ; CODE XREF: bInternalPlayEMF(x,x,x,x,x)+6A2 p
.text:77EFEF66                                         ; bInternalPlayEMF(x,x,x,x,x)+13B2 p ...
.text:77EFEF66
.text:77EFEF66 var_58          = dword ptr -58h
.text:77EFEF66 var_54          = dword ptr -54h
.text:77EFEF66 var_50          = dword ptr -50h
.text:77EFEF66 var_4C          = dword ptr -4Ch
.text:77EFEF66 var_48          = dword ptr -48h
.text:77EFEF66 var_44          = dword ptr -44h
.text:77EFEF66 var_40          = dword ptr -40h
.text:77EFEF66 var_3C          = dword ptr -3Ch
.text:77EFEF66 var_38          = dword ptr -38h
.text:77EFEF66 var_34          = dword ptr -34h
.text:77EFEF66 var_30          = dword ptr -30h
.text:77EFEF66 var_2C          = dword ptr -2Ch
.text:77EFEF66 var_28          = dword ptr -28h
.text:77EFEF66 var_24          = dword ptr -24h
.text:77EFEF66 var_20          = dword ptr -20h
.text:77EFEF66 var_1C          = dword ptr -1Ch
.text:77EFEF66 var_18          = dword ptr -18h
.text:77EFEF66 var_14          = dword ptr -14h
.text:77EFEF66 var_10          = dword ptr -10h
.text:77EFEF66 var_C           = dword ptr -0Ch
.text:77EFEF66 var_8           = dword ptr -8
.text:77EFEF66 var_4           = dword ptr -4
.text:77EFEF66 hdc             = dword ptr  8
.text:77EFEF66 lphandletable   = dword ptr  0Ch
.text:77EFEF66 lpenhmetarecord = dword ptr  10h
.text:77EFEF66 nhandles        = dword ptr  14h
.text:77EFEF66
.text:77EFEF66 ; FUNCTION CHUNK AT .text:77F18A61 SIZE 0000028A BYTES
.text:77EFEF66
.text:77EFEF66                 mov     edi, edi
.text:77EFEF68                 push    ebp
.text:77EFEF69                 mov     ebp, esp
.text:77EFEF6B                 sub     esp, 58h
.text:77EFEF6E                 push    ebx
.text:77EFEF6F                 mov     ebx, [ebp+lpenhmetarecord]
.text:77EFEF72                 mov     eax, [ebx]
.text:77EFEF74                 cmp     eax, 1
.text:77EFEF77                 push    esi
.text:77EFEF78                 push    edi
.text:77EFEF79                 jb      short loc_77EFEF99
.text:77EFEF7B                 cmp     eax, 122
.text:77EFEF7E                 ja      short loc_77EFEF99
.text:77EFEF80                 push    [ebp+nhandles]
.text:77EFEF83                 mov     ecx, ebx
.text:77EFEF85                 push    [ebp+lphandletable]
.text:77EFEF88                 push    [ebp+hdc]
.text:77EFEF8B                 call    _fpStartDocDlgW[eax*4] ; eax必须是79,这样才可以调用虚函数MRCOLORMATCHTOTARGET::bPlay
.text:77EFEF92
.text:77EFEF92 loc_77EFEF92:                           ; CODE XREF: PlayEnhMetaFileRecord(x,x,x,x)+19D72 j
.text:77EFEF92                                         ; PlayEnhMetaFileRecord(x,x,x,x)+19D80 j
.text:77EFEF92                 pop     edi
.text:77EFEF93                 pop     esi
.text:77EFEF94                 pop     ebx
.text:77EFEF95                 leave
.text:77EFEF96                 retn    10h
.text:77EFEF99 ; ---------------------------------------------------------------------------
.text:77EFEF99
.text:77EFEF99 loc_77EFEF99:                           ; CODE XREF: PlayEnhMetaFileRecord(x,x,x,x)+13 j
.text:77EFEF99                                         ; PlayEnhMetaFileRecord(x,x,x,x)+18 j
.text:77EFEF99                 test    eax, eax
.text:77EFEF9B                 js      loc_77F18CD5
.text:77EFEFA1                 jmp     loc_77F18A61
.text:77EFEFA1 __stdcall PlayEnhMetaFileRecord(x, x, x, x) endp
```

上图中的6Ch偏移处的四字节必须是`79000000`，这样才可以让数据流到达`MRCOLORMATCHTOTARGET::bPlay`。

```asm
text:77F2EBB5 public: int __thiscall MRCOLORMATCHTOTARGET::bPlay(void *, struct tagHANDLETABLE *, unsigned int) proc near
.text:77F2EBB5                                         ; CODE XREF: PlayEnhMetaFileRecord(x,x,x,x)+25 P
.text:77F2EBB5                                         ; DATA XREF: .data:77F327C0 o
.text:77F2EBB5
.text:77F2EBB5 var_264         = dword ptr -264h
.text:77F2EBB5 var_260         = dword ptr -260h
.text:77F2EBB5 var_25C         = dword ptr -25Ch
.text:77F2EBB5 var_258         = dword ptr -258h
.text:77F2EBB5 var_254         = dword ptr -254h
.text:77F2EBB5 var_250         = dword ptr -250h
.text:77F2EBB5 var_24C         = dword ptr -24Ch
.text:77F2EBB5 var_248         = dword ptr -248h
.text:77F2EBB5 var_244         = dword ptr -244h
.text:77F2EBB5 var_240         = dword ptr -240h
.text:77F2EBB5 lp_buff         = dword ptr -20Ch
.text:77F2EBB5 var_4           = dword ptr -4
.text:77F2EBB5 hdc             = dword ptr  8
.text:77F2EBB5 lphandletable   = dword ptr  0Ch
.text:77F2EBB5
.text:77F2EBB5                 mov     edi, edi
.text:77F2EBB7                 push    ebp
.text:77F2EBB8                 mov     ebp, esp
.text:77F2EBBA                 sub     esp, 264h
.text:77F2EBC0                 mov     eax, ___security_cookie
.text:77F2EBC5                 push    ebx
```
