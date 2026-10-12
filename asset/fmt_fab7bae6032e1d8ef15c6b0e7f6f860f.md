
# MS08-021 漏洞分析

> 原文：[`www.pediy.com/kssd/pediy10/78412.html`](https://www.pediy.com/kssd/pediy10/78412.html)

Author:dge  

我从分析这个漏洞的过程中重新认识到一些东西，写下来分享给像我一样的人们。

在分析文件格式漏洞时只用调试工具跟踪有时候很费劲。如果 POC 可以顺利执行 shellcode，跟踪比较简单，但如果 POC 执行不到 shellcode，就很难去解决，当然我们可以通过补丁比较确定出几个可疑函数进行重点跟踪，当然前提是你得能确定出来，如果补丁对代码的改动很大，要确定出可疑函数实非易事，即使能确定出几个来，对他们跟踪也是非常麻烦的事情，让复杂的问题变的简单，那肯定是最理想的。

认识 `PaiMei` 这个工具是在 `failwest` 的书里，虽然读完那本书已经很久了，但一直没用过这个工具，直到前几天分析这个漏洞，在分析未果的情况下，我想起了 `PaiMei`,这才体会到了它的强大，这也是我想写这篇文章的原因。

`MS08-021` 有两个漏洞，这是其中的一个，这个漏洞的 POC 在很久之前就有人公布了，这是 POC 的地址：http://www.milw0rm.com/exploits/5442，有了 POC 和 `PaiMei`，分析这个漏洞就变得简单了，这个漏洞发生在 `gdi32.dll` 中，是典型的栈溢出。

用 `PaiMei` 和 `od` 我们可以轻松地确定出漏洞触发路径。

`PlayEnhMetaFileRecord`  
  |\_\_MRCOLORMATCHTOTARGET::bPlay  
       |\_\_IcmCreateColorSpaceByName  
            |\_\_lstrcpyW  

先简单了解下 EMF 文件格式

![](img/2359b438f25b1fac46761270eec03f48.jpg)

Header 结构：

```c
typedef struct tagENHMETAHEADER {  
  DWORD iType;  
  DWORD nSize;  
  RECTl rclBounds;  
  RECTl rclFrame;  
  DWORD dSignature;  
  DWORD nVersion;  
  DWORD nBytes;  
  DWORD nRecords;  
  WORD  nHandles;  
  WORD  sReserved;  
  DWORD nDescription;  
  DWORD offDescription;  
  DWORD nPalEntries;  
  SIZEl szlDevice;  
  SIZEl szlMillimeters;  
#if (WINVER >= 0x0400)  
  DWORD cbPixelFormat;  
  DWORD offPixelFormat;  
  DWORD bOpenGL;  
#endif /* WINVER >= 0x0400 */  
#if (WINVER >= 0x0500)  
  SIZEl szlMicrometers;  
#endif /* WINVER >= 0x0500 */  
} ENHMETAHEADER, *PENHMETAHEADER;  
```

RECORD 结构

```c
typedef struct tagENHMETARECORD  
{  
    DWORD   iType;              // Record type EMR_XXX  
    DWORD   nSize;              // Record size in bytes  
    DWORD   dParm[1];           // DWORD Array of parameters  
} ENHMETARECORD;  
```

函数原型：

```c
BOOL PlayEnhMetaFileRecord(
  HDC hdc,                              // handle to DC  
  LPHANDLETABLE lpHandletable,          // metafile handle table  
  CONST ENHMETARECORD *lpEnhMetaRecord, // metafile record  
  UINT nHandles                         // count of handles  
);
```

POC 截图：

![](img/224df13b9bf8eb7841b89e6db990d017.jpg)

下边分析下这个漏洞：

```asm
.text:77EFEF66 __stdcall PlayEnhMetaFileRecord(x, x, x, x) proc near
.text:77EFEF66                                        ; CODE XREF: bInternalPlayEMF(x,x,x,x,x)+6A2 p
.text:77EFEF66                                        ; bInternalPlayEMF(x,x,x,x,x)+13B2 p ...
.text:77EFEF66

.text:77EFEF66 var_58          = dword ptr -58h
.text:77EFEF66 var_54          = dword ptr -54h
...
.text:77EFEF66 hdc             = dword ptr  8
.text:77EFEF66 lphandletable   = dword ptr  0Ch
.text:77EFEF66 lpenhmetarecord = dword ptr  10h
.text:77EFEF66 nhandles        = dword ptr  14h

.text:77EFEF66 ; FUNCTION CHUNK AT .text:77F18A61 SIZE 0000028A BYTES

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
.text:77EFEF8B                 call    _fpStartDocDlgW[eax*4] ; eax 必须是 79,这样才可以调用虚函数 MRCOLORMATCHTOTARGET::bPlay
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

上图中的 6Ch 偏移处的四字节必须是 `79000000`，这样才可以让数据流到达 `MRCOLORMATCHTOTARGET::bPlay`。

```asm
text:77F2EBB5 public: int __thiscall MRCOLORMATCHTOTARGET::bPlay(void *, struct tagHANDLETABLE *, unsigned int) proc near
.text:77F2EBB5                                        ; CODE XREF: PlayEnhMetaFileRecord(x,x,x,x)+25 P
.text:77F2EBB5                                        ; DATA XREF: .data:77F327C0 o
.text:77F2EBB5

.text:77F2EBB5 var_264         = dword ptr -264h
...
.text:77F2EBB5 hdc             = dword ptr  8
.text:77F2EBB5 lphandletable   = dword ptr  0Ch

.text:77F2EBB5                 mov     edi, edi
.text:77F2EBB7                 push    ebp
.text:77F2EBB8                 mov     ebp, esp
.text:77F2EBBA                 sub     esp, 264h
.text:77F2EBC0                 mov     eax, ___security_cookie
.text:77F2EBC5                 push    ebx
.text:77F2EBC6                 mov     ebx, [ebp+hdc]
.text:77F2EBC9                 mov     [ebp+var_4], eax
.text:77F2EBCC                 mov     eax, [ebp+lphandletable]
.text:77F2EBCF                 push    esi
.text:77F2EBD0                 push    eax
.text:77F2EBD1                 mov     esi, ecx
.text:77F2EBD3                 mov     [ebp+var_258], ebx
.text:77F2EBD9                 call    MRCOLORMATCHTOTARGET::bCheckRecord(tagHANDLETABLE *) ;这个函数里规定[esi+0x14]+[esi+0x10]+0x1B==[esi+0x4]
```
