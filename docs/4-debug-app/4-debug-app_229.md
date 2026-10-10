

# 获取 EVE Online 游戏中 Python 脚本代码的方法 \[祝大家元旦快乐\]

> 原文：[`www.pediy.com/kssd/pediy10/79827.html`](https://www.pediy.com/kssd/pediy10/79827.html)

庆祝元旦快乐！从看雪论坛学到很多，也回报点。  
  
获取 EVE Online 游戏中 Python 脚本代码的方法  
  
  
首先，通过观察游戏文件不难发现 decompiled.code 文件中可能包含的就是 Python 脚本代码。但是如何才能得到解密的原始字节码呢？因此我们的思路首先围绕这个问题展开。  
既然代码已经加密保存，那么程序运行中自然需要解密，因此如果我们能够跟踪到解密代码的部分，自然容易得到 Python 代码的字节码。为此，我们不妨从 Python 运行引擎的源代码中开始分析，可能更容易找到线索。  
首先，我们先用 IDA 反汇编一下游戏的主程序文件 Exefile.exe ，可以发现程序中有 Python 的不少 API 函数调用。并且调用位于 blue.dll 模块中，通过动态跟踪我们可以进一步确认 blue.dll 模块即为 Python23.dll 换名而得。具体过程这里不多叙述，我们只把注意力集中在 Python 源代码分析上。  
我们可以猜想一下那些 API 是游戏运行时必须调用的？可能这会有个更好的开始。我猜想字节码的版本识别函数 PyImport\_GetMagicNumber 可能是必须调用的。因此可以从这里入手看看。为此我们通过动态跟踪来确认一下，并且从动态跟踪的堆栈中了解函数的调用流程。  
通过跟踪我们发现 PyImport\_GetMagicNumber 函数由 unmarshal\_code 函数调用。我们来了解一下这个函数的流程：  
  
static PyObject \*  
unmarshal\_code(char \*pathname, PyObject \*data, time\_t mtime)  
{  
    ... ( 略 )   
  if (get\_long((unsigned char \*)buf) != PyImport\_GetMagicNumber()) {  
        ... ( 错误处理，略 )  
  }  
  
  if (mtime != 0 && !eq\_mtime(get\_long((unsigned char \*)buf + 4),  
            mtime)) {  
        ... ( 错误处理，略 )  
  }  
  
  code = PyMarshal\_ReadObjectFromString(buf + 8, size - 8);  
   ... ( 略 )   
}  
  
   为了让我们只关注关键部分代码，因此一些不太重要的代码这里都略过。从上面的代码中我们可以看到程序首先进行获取 MagicNumber 的工作和时间戳的认证，然后就通过 PyMarshal\_ReadObjectFromString 函数把缓冲区中的代码数据转换为 code 代码对象。因此我们只要能够在这一行代码处进行拦截即可以得到每个 Python 模块的原始字节码。不过这个方法并不能得到那些没有运行的模块的字节码，因此这个方法仍有缺陷。而我们最好的方法是能够知道 decompiled.code 文件的加密方式，然后编写解密代码解密整个文件得到所有 Python 模块的字节码，为此我们继续追踪下去，希望能够找到解密的源头。  
    我们向上追溯跟踪，可以发现 unmarshal\_code 函数由 get\_code\_from\_data 函数调用，我们来分析一下源代码：  
   
static PyObject \*  
get\_code\_from\_data(ZipImporter \*self, int ispackage, int isbytecode,  
       time\_t mtime, PyObject \*toc\_entry)  
{  
  PyObject \*data, \*code;  
  char \*modpath;  
  char \*archive = PyString\_AsString(self->archive);  
  
  if (archive == NULL)  
    return NULL;  
  
  data = get\_data(archive, toc\_entry);  
  if (data == NULL)  
    return NULL;  
  
  modpath = PyString\_AsString(PyTuple\_GetItem(toc\_entry, 0));  
  
  if (isbytecode) {  
    code = unmarshal\_code(modpath, data, mtime);  
  }  
  else {  
    code = compile\_source(modpath, data);  
  }  
  Py\_DECREF(data);  
  return code;  
}  
    我们可以看到程序实际上是通过 ZipImporter 来管理数据，首先通过这个接口来提取原始数据流（即文件中保存的字节码），然后通过 unmarshal\_code 函数把数据流转换成 code 对象并作为结果返回。这时我们如果通过 IDA 逆向 blue.dll 这段函数的汇编代码就会发现 Stackless Python 源代码与 blue.dll 中的汇编代码稍有不同（当然实际上我是走了不少弯路才发现这点的），我们来看一下汇编代码：  
.text:100F5C30 get\_code\_from\_data proc near            ;   
.... ( 略 )  
.text:100F5C47                 push    esi  
.text:100F5C48                 push    ebp  
.text:100F5C49                 call    get\_data  
.text:100F5C4E                 mov     esi, eax  
.text:100F5C50                 add     esp, 4  
.text:100F5C53                 test    esi, esi  
.text:100F5C55                 jnz     short loc\_100F5C5A  
.text:100F5C57                 pop     esi  
.text:100F5C58                 pop     ebp  
.text:100F5C59                 retn  
.text:100F5C5A ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:100F5C5A  
.text:100F5C5A loc\_100F5C5A:                           ; CODE XREF: get\_code\_from\_data+25 j  
.text:100F5C5A                 cmp     \[esp+8+arg\_4\], 0  
.text:100F5C5F                 push    ebx  
.text:100F5C60                 push    edi  
.text:100F5C61                 jz      short loc\_100F5CBC  
.text:100F5C63                 push    offset aBlue\_crypto ; "blue.crypto"  
.text:100F5C68                 call    PyImport\_ImportModule  
.text:100F5C6D                 mov     edi, eax  
.text:100F5C6F                 add     esp, 4  
.text:100F5C72                 test    edi, edi  
.text:100F5C74                 jz      short loc\_100F5CA4  
.text:100F5C76                 push    offset \_Py\_TrueStruct  
.text:100F5C7B                 push    esi  
.text:100F5C7C                 push    offset aOo\_1    ; "OO"  
.text:100F5C81                 push    offset aUnjumblestring ; "UnjumbleString"  
.text:100F5C86                 push    edi  
.text:100F5C87                 call    PyObject\_CallMethod  
.text:100F5C8C                 add     esp, 14h  
.text:100F5C8F                 add     dword ptr \[edi\], 0FFFFFFFFh  
.text:100F5C92                 mov     ebx, eax  
.text:100F5C94                 jnz     short loc\_100F5CA0  
.text:100F5C96                 mov     edx, \[edi+4\]  
.text:100F5C99                 push    edi  
.text:100F5C9A                 call    dword ptr \[edx+18h\]  
.text:100F5C9D                 add     esp, 4  
    ....( 略 )  
get\_code\_from\_data+C6 j  
.text:100F5D02                 mov     eax, edi  
.text:100F5D04                 pop     edi  
.text:100F5D05                 pop     ebx  
.text:100F5D06                 pop     esi  
.text:100F5D07                 pop     ebp  
.text:100F5D08                 retn  
.text:100F5D08 get\_code\_from\_data endp  
  
    为了便于分析，同样我略去了很多无关紧要的汇编代码。从上面的代码中我们可以看到在 get\_data 函数获取了数据流后到传入 unmarshal\_code 函数之间多了一段对数据流进行进一步处理的转换代码。通过跟踪我们发现这段代码实现的功能正是解密加密的字节码的功能。这一段代码如果用 Python 表示即为 blue.crypto. UnjumbleString(data) 因此，我们需要进一步找到这个解密函数的位置。  
通过搜索字符串我们不难发现这个函数的位置：  
  
.text:10044680 UnjumbleString  proc near               ; DATA XREF: .data:101EC254 o  
    ( 略 )  
.text:100446AF                 push    esi  
.text:100446B0                 push    edi  
.text:100446B1                 push    0  
.text:100446B3                 call    PyTuple\_New  
.text:100446B8                 mov     esi, eax  
.text:100446BA                 push    esi  
.text:100446BB                 push    offset \_Py\_NoneStruct  
.text:100446C0                 mov     \[esp+34h+var\_8\], esi  
.text:100446C4                 call    \_GetVerCryptKey  
.text:100446C9                 mov     edi, eax  
    ( 略 )  
.text:100446F1                 mov     ecx, \[esp+28h+var\_4\]  
.text:100446F5                 push    ebx  
.text:100446F6                 push    ecx  
.text:100446F7                 push    0  
.text:100446F9                 push    offset \_Py\_TrueStruct  
.text:100446FE                 push    offset \_Py\_NoneStruct  
.text:10044703                 push    edi  
.text:10044704                 push    offset aOooio   ; "OOOiO"  
.text:10044709                 call    Py\_BuildValue  
.text:1004470E                 mov     ebx, eax  
    ( 略 )  
.text:10044746                 push    ebp  
.text:10044747                 push    ebx  
.text:10044748                 push    offset \_Py\_NoneStruct  
.text:1004474D                 call    \_CryptDecrypt  
.text:10044752                 mov     ebp, eax  
    ( 略 )  
.text:1004479A                 mov     ecx, \[esp+30h+var\_1C\]  
.text:1004479E                 push    ecx  
.text:1004479F                 call    PyObject\_IsTrue  
.text:100447A4                 add     esp, 4  
.text:100447A7                 test    eax, eax  
.text:100447A9                 jz      loc\_10044837  
.text:100447AF                 push    offset aZlib    ; "zlib"  
.text:100447B4                 call    PyImport\_ImportModule  
.text:100447B9                 add     esp, 4  
.text:100447BC                 test    eax, eax  
.text:100447BE                 mov     \[esp+30h+var\_14\], eax  
    ( 略 )  
.text:100447FB                 push    ebp  
.text:100447FC                 push    offset aO\_2     ; "O"  
.text:10044801                 push    offset aDecompress ; "decompress"  
.text:10044806                 push    eax  
.text:10044807                 call    PyObject\_CallMethod  
    ( 略 )  
.text:1004486C UnjumbleString  endp  
  
    从以上代码中我们来分析一下 UnjumbleString 函数的解密流程，首先程序通过\_GetVerCryptKey 函数获取一个解密密钥，然后通过 \_CryptDecrypt 函数对加密数据进行解密操作，最后通过导入 zlib 模块，并调用其方法 decompress 对解密后的数据进行解压缩。到这里我们只需要了解密钥如何产生以及如何利用密钥进行解密操作即可以实现整个解密过程。因此我们进一步需要分析的目标是 \_GetVerCryptKey 以及 \_CryptDecrypt 函数的具体实现。  
    分析 \_CryptDecrypt 函数我们可以发现其只是利用 Windows API 中的 CryptDecrypt 函数对数据进行解密，\_GetVerCryptKey 函数也只是简单地获取内存中的密钥，而函数内并不具体产生密钥。从这些信息中我们初步可以判断程序是通过 Windows 的 Crypto API 函数实现加解密操作。由于 Crypto API 规范基本一致，因此最终我们的目标锁定在密钥是如何产生的问题上？  
    通过查找对密钥地址进行引用的函数，我们可以得知密钥的产生位于 sub\_10042520 函数的代码中：  
  
.text:10042520 ; 圹圹圹圹圹圹圹?S U B R O U T I N E 圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹?  
.text:10042520  
.text:10042520  
.text:10042520 sub\_10042520    proc near               ; CODE XREF: sub\_10012900+150 p  
.text:10042520  
.text:10042520 var\_18          = dword ptr -18h  
.text:10042520 hKey            = dword ptr -0Ch  
.text:10042520 hPubKey         = dword ptr -8  
.text:10042520 dwDataLen       = dword ptr -4  
.text:10042520  
.text:10042520                 sub     esp, 0Ch  
.text:10042523                 push    0F0000000h      ; dwFlags  
.text:10042528                 push    1               ; dwProvType  
.text:1004252A                 push    offset pszProvider ; "Microsoft Enhanced Cryptographic Provid"...  
.text:1004252F                 push    0               ; pszContainer  
.text:10042531                 push    offset phProv   ; phProv  
.text:10042536                 call    ds:CryptAcquireContextA ; Acquire a handle to a particular  
.text:10042536                                         ; key container within a particular CSP  
.text:1004253C                 test    eax, eax  
.text:1004253E                 jz      short loc\_10042564  
.text:10042540                 mov     edx, phProv  
.text:10042546                 push    offset phKey    ; phKey  
.text:1004254B                 push    0               ; dwFlags  
.text:1004254D                 push    0               ; hPubKey  
.text:1004254F                 push    94h             ; dwDataLen  
.text:10042554                 push    offset pbData   ; pbData  
.text:10042559                 push    edx             ; hProv  
.text:1004255A                 call    ds:CryptImportKey ; Transfer a cryptographic key  
.text:1004255A                                         ; from a key blob to the CSP  
.text:10042560                 test    eax, eax  
.text:10042562                 jnz     short loc\_10042581  
.text:10042564  
.text:10042564 loc\_10042564:                           ; CODE XREF: sub\_10042520+1E j  
.text:10042564                 mov     eax, BeOS  
.text:10042569                 mov     ecx, \[eax\]  
.text:1004256B                 push    offset byte\_101A8F68  
.text:10042570                 push    0  
.text:10042572                 push    0FFFFFFFEh  
.text:10042574                 push    eax  
.text:10042575                 call    dword ptr \[ecx+34h\]  
.text:10042578                 add     esp, 10h  
.text:1004257B                 xor     al, al  
.text:1004257D                 add     esp, 0Ch  
.text:10042580                 retn  
.text:10042581 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:10042581  
.text:10042581 loc\_10042581:                           ; CODE XREF: sub\_10042520+42 j  
.text:10042581                 mov     eax, phProv  
.text:10042586                 push    ebp  
.text:10042587                 push    esi  
.text:10042588                 push    edi  
.text:10042589                 lea     edx, \[esp+18h+hKey\]  
.text:1004258D                 push    edx             ; phKey  
.text:1004258E                 push    1               ; dwFlags  
.text:10042590                 push    1               ; Algid  
.text:10042592                 push    eax             ; hProv  
.text:10042593                 mov     \[esp+28h+hPubKey\], 0  
.text:1004259B                 mov     ebp, eax  
.text:1004259D                 mov     \[esp+28h+hKey\], 0  
.text:100425A5                 call    ds:CryptGenKey  ; Generate random cryptographic keys  
.text:100425A5                                         ; for use with the CSP module  
.text:100425AB                 test    eax, eax  
.text:100425AD                 jz      short loc\_100425FE  
.text:100425AF                 mov     ecx, \[esp+18h+hKey\]  
.text:100425B3                 mov     esi, ds:CryptExportKey ; Export cryptographic keys out of  
.text:100425B3                                         ; a cryptographic service provider  
.text:100425B3                                         ; in a secure manner  
.text:100425B9                 lea     eax, \[esp+18h+dwDataLen\]  
.text:100425BD                 push    eax             ; pdwDataLen  
.text:100425BE                 push    0               ; pbData  
.text:100425C0                 push    0               ; dwFlags  
.text:100425C2                 push    7               ; dwBlobType  
.text:100425C4                 push    0               ; hExpKey  
.text:100425C6                 push    ecx             ; hKey  
.text:100425C7                 call    esi ; CryptExportKey ; Export cryptographic keys out of  
.text:100425C7                                         ; a cryptographic service provider  
.text:100425C7                                         ; in a secure manner  
.text:100425C9                 test    eax, eax  
.text:100425CB                 jz      short loc\_100425FE  
.text:100425CD                 mov     edx, \[esp+18h+dwDataLen\]  
.text:100425D1                 push    edx             ; uBytes  
.text:100425D2                 push    40h             ; uFlags  
.text:100425D4                 call    ds:LocalAlloc  
.text:100425DA                 mov     edi, eax  
.text:100425DC                 test    edi, edi  
.text:100425DE                 jz      short loc\_100425FE  
.text:100425E0                 mov     ecx, \[esp+18h+hKey\]  
.text:100425E4                 lea     eax, \[esp+18h+dwDataLen\]  
.text:100425E8                 push    eax             ; pdwDataLen  
.text:100425E9                 push    edi             ; pbData  
.text:100425EA                 push    0               ; dwFlags  
.text:100425EC                 push    7               ; dwBlobType  
.text:100425EE                 push    0               ; hExpKey  
.text:100425F0                 push    ecx             ; hKey  
.text:100425F1                 call    esi ; CryptExportKey ; Export cryptographic keys out of  
.text:100425F1                                         ; a cryptographic service provider  
.text:100425F1                                         ; in a secure manner  
.text:100425F3                 test    eax, eax  
.text:100425F5                 jnz     short loc\_10042616  
.text:100425F7                 push    edi             ; hMem  
.text:100425F8                 call    ds:LocalFree  
.text:100425FE  
.text:100425FE loc\_100425FE:                           ; CODE XREF: sub\_10042520+8D j  
.text:100425FE                                         ; sub\_10042520+AB j ...  
.text:100425FE                 mov     eax, \[esp+18h+hKey\]  
.text:10042602                 test    eax, eax  
.text:10042604                 jz      loc\_100426E8  
.text:1004260A                 push    eax             ; hKey  
.text:1004260B                 call    ds:CryptDestroyKey  
.text:10042611                 jmp     loc\_100426E8  
.text:10042616 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:10042616  
.text:10042616 loc\_10042616:                           ; CODE XREF: sub\_10042520+D5 j  
.text:10042616                 mov     eax, \[esp+18h+hKey\]  
.text:1004261A                 test    eax, eax  
.text:1004261C                 jz      short loc\_10042625  
.text:1004261E                 push    eax             ; hKey  
.text:1004261F                 call    ds:CryptDestroyKey  
.text:10042625  
.text:10042625 loc\_10042625:                           ; CODE XREF: sub\_10042520+FC j  
.text:10042625                 mov     \[esp+18h+hKey\], 0  
.text:1004262D                 mov     edx, \[edi+0Ch\]  
.text:10042630                 lea     ecx, \[edi+10h\]  
.text:10042633                 xor     eax, eax  
.text:10042635  
.text:10042635 loc\_10042635:                           ; CODE XREF: sub\_10042520+128 j  
.text:10042635                 test    eax, eax  
.text:10042637                 jnz     short loc\_1004263E  
.text:10042639                 mov     byte ptr \[ecx\], 1  
.text:1004263C                 jmp     short loc\_10042642  
.text:1004263E ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:1004263E  
.text:1004263E loc\_1004263E:                           ; CODE XREF: sub\_10042520+117 j  
.text:1004263E                 mov     byte ptr \[eax+ecx\], 0  
.text:10042642  
.text:10042642 loc\_10042642:                           ; CODE XREF: sub\_10042520+11C j  
.text:10042642                 add     eax, 1  
.text:10042645                 cmp     eax, 4  
.text:10042648                 jb      short loc\_10042635  
.text:1004264A                 mov     eax, edx  
.text:1004264C                 mov     esi, edx  
.text:1004264E                 shr     eax, 4  
.text:10042651                 shr     esi, 3  
.text:10042654                 lea     edx, \[esi+eax\*2+4\]  
.text:10042658                 add     ecx, edx  
.text:1004265A                 xor     edx, edx  
.text:1004265C                 test    eax, eax  
.text:1004265E                 jbe     short loc\_10042674  
.text:10042660  
.text:10042660 loc\_10042660:                           ; CODE XREF: sub\_10042520+152 j  
.text:10042660                 test    edx, edx  
.text:10042662                 jnz     short loc\_10042669  
.text:10042664                 mov     byte ptr \[ecx\], 1  
.text:10042667                 jmp     short loc\_1004266D  
.text:10042669 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:10042669  
.text:10042669 loc\_10042669:                           ; CODE XREF: sub\_10042520+142 j  
.text:10042669                 mov     byte ptr \[edx+ecx\], 0  
.text:1004266D  
.text:1004266D loc\_1004266D:                           ; CODE XREF: sub\_10042520+147 j  
.text:1004266D                 add     edx, 1  
.text:10042670                 cmp     edx, eax  
.text:10042672                 jb      short loc\_10042660  
.text:10042674  
.text:10042674 loc\_10042674:                           ; CODE XREF: sub\_10042520+13E j  
.text:10042674                 add     ecx, eax  
.text:10042676                 xor     edx, edx  
.text:10042678                 test    eax, eax  
.text:1004267A                 jbe     short loc\_10042694  
.text:1004267C                 lea     esp, \[esp+0\]  
.text:10042680  
.text:10042680 loc\_10042680:                           ; CODE XREF: sub\_10042520+172 j  
.text:10042680                 test    edx, edx  
.text:10042682                 jnz     short loc\_10042689  
.text:10042684                 mov     byte ptr \[ecx\], 1  
.text:10042687                 jmp     short loc\_1004268D  
.text:10042689 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:10042689  
.text:10042689 loc\_10042689:                           ; CODE XREF: sub\_10042520+162 j  
.text:10042689                 mov     byte ptr \[edx+ecx\], 0  
.text:1004268D  
.text:1004268D loc\_1004268D:                           ; CODE XREF: sub\_10042520+167 j  
.text:1004268D                 add     edx, 1  
.text:10042690                 cmp     edx, eax  
.text:10042692                 jb      short loc\_10042680  
.text:10042694  
.text:10042694 loc\_10042694:                           ; CODE XREF: sub\_10042520+15A j  
.text:10042694                 lea     ecx, \[ecx+eax\*2\]  
.text:10042697                 xor     eax, eax  
.text:10042699                 test    esi, esi  
.text:1004269B                 jbe     short loc\_100426B4  
.text:1004269D                 lea     ecx, \[ecx+0\]  
.text:100426A0  
.text:100426A0 loc\_100426A0:                           ; CODE XREF: sub\_10042520+192 j  
.text:100426A0                 test    eax, eax  
.text:100426A2                 jnz     short loc\_100426A9  
.text:100426A4                 mov     byte ptr \[ecx\], 1  
.text:100426A7                 jmp     short loc\_100426AD  
.text:100426A9 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:100426A9  
.text:100426A9 loc\_100426A9:                           ; CODE XREF: sub\_10042520+182 j  
.text:100426A9                 mov     byte ptr \[eax+ecx\], 0  
.text:100426AD  
.text:100426AD loc\_100426AD:                           ; CODE XREF: sub\_10042520+187 j  
.text:100426AD                 add     eax, 1  
.text:100426B0                 cmp     eax, esi  
.text:100426B2                 jb      short loc\_100426A0  
.text:100426B4  
.text:100426B4 loc\_100426B4:                           ; CODE XREF: sub\_10042520+17B j  
.text:100426B4                 mov     ecx, \[esp+18h+dwDataLen\]  
.text:100426B8                 lea     eax, \[esp+18h+hPubKey\]  
.text:100426BC                 push    eax             ; phKey  
.text:100426BD                 push    0               ; dwFlags  
.text:100426BF                 push    0               ; hPubKey  
.text:100426C1                 push    ecx             ; dwDataLen  
.text:100426C2                 push    edi             ; pbData  
.text:100426C3                 push    ebp             ; hProv  
.text:100426C4                 mov     ebp, ds:CryptImportKey ; Transfer a cryptographic key  
.text:100426C4                                         ; from a key blob to the CSP  
.text:100426CA                 call    ebp ; CryptImportKey ; Transfer a cryptographic key  
.text:100426CA                                         ; from a key blob to the CSP  
.text:100426CC                 push    edi             ; hMem  
.text:100426CD                 mov     esi, eax  
.text:100426CF                 call    ds:LocalFree  
.text:100426D5                 mov     eax, \[esp+18h+hKey\]  
.text:100426D9                 test    eax, eax  
.text:100426DB                 jz      short loc\_100426E4  
.text:100426DD                 push    eax             ; hKey  
.text:100426DE                 call    ds:CryptDestroyKey  
.text:100426E4  
.text:100426E4 loc\_100426E4:                           ; CODE XREF: sub\_10042520+1BB j  
.text:100426E4                 test    esi, esi  
.text:100426E6                 jnz     short loc\_10042717  
.text:100426E8  
.text:100426E8 loc\_100426E8:                           ; CODE XREF: sub\_10042520+E4 j  
.text:100426E8                                         ; sub\_10042520+F1 j ...  
.text:100426E8                 mov     eax, BeOS  
.text:100426ED                 mov     edx, \[eax\]  
.text:100426EF                 push    offset byte\_101A8F68  
.text:100426F4                 push    0  
.text:100426F6                 push    0FFFFFFFEh  
.text:100426F8                 push    eax  
.text:100426F9                 call    dword ptr \[edx+34h\]  
.text:100426FC                 mov     eax, \[esp+28h+hPubKey\]  
.text:10042700                 add     esp, 10h  
.text:10042703                 test    eax, eax  
.text:10042705                 jz      short loc\_1004270E  
.text:10042707                 push    eax             ; hKey  
.text:10042708                 call    ds:CryptDestroyKey  
.text:1004270E  
.text:1004270E loc\_1004270E:                           ; CODE XREF: sub\_10042520+1E5 j  
.text:1004270E                 pop     edi  
.text:1004270F                 pop     esi  
.text:10042710                 xor     al, al  
.text:10042712                 pop     ebp  
.text:10042713                 add     esp, 0Ch  
.text:10042716                 retn  
.text:10042717 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:10042717  
.text:10042717 loc\_10042717:                           ; CODE XREF: sub\_10042520+1C6 j  
.text:10042717                 mov     eax, \[esp+18h+hPubKey\]  
.text:1004271B                 mov     ecx, phProv  
.text:10042721                 push    offset hKey     ; phKey  
.text:10042726                 push    0               ; dwFlags  
.text:10042728                 push    eax             ; hPubKey  
.text:10042729                 push    8Ch             ; dwDataLen  
.text:1004272E                 push    offset byte\_101AE848 ; pbData  
.text:10042733                 push    ecx             ; hProv  
.text:10042734                 call    ebp ; CryptImportKey ; Transfer a cryptographic key  
.text:10042734                                         ; from a key blob to the CSP  
.text:10042736                 test    eax, eax  
.text:10042738                 jz      short loc\_100426E8  
.text:1004273A                 mov     eax, \[esp+18h+hPubKey\]  
.text:1004273E                 test    eax, eax  
.text:10042740                 jz      short loc\_10042749  
.text:10042742                 push    eax             ; hKey  
.text:10042743                 call    ds:CryptDestroyKey  
.text:10042749  
.text:10042749 loc\_10042749:                           ; CODE XREF: sub\_10042520+220 j  
.text:10042749                 pop     edi  
.text:1004274A                 pop     esi  
.text:1004274B                 mov     al, 1  
.text:1004274D                 pop     ebp  
.text:1004274E                 add     esp, 0Ch  
.text:10042751                 retn  
.text:10042751 sub\_10042520    endp  
  
    以上代码虽然稍长，我们分析后还是很容易了解其功能实现。大体上它可以分为以下几个步骤：  
  
1)  通过 CryptAcquireContextA 函数向 CSP 申请一个新的密钥容器。  
2)  通过 CryptImportKey 函数把一段 blob 数据转换成密钥放入容器中。  
3)  通过 CryptGenKey 函数生成一个随机的会话密钥或者是公/私密钥对。  
4)  通过 CryptExportKey 函数导出密钥到申请的内存中。  
5)  对内存中的密钥数据进行多重变换。  
6)  重新通过 CryptImportKey 函数导入内存中的数据到密钥容器中。  
  
到此，我们只要详细分析并实现这些步骤即可以得到一个合法的密钥，然后通过调用相应的 Crypto API 函数对加密数据进行解密，最后用 zlib.decompress 解压缩即可以得到我们需要的原始字节码。