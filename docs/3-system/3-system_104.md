
# 坑灰未冷山东乱,loader 原来不读书

> 原文：[`www.pediy.com/kssd/pediy10/102708.html`](https://www.pediy.com/kssd/pediy10/102708.html)

自己转自己的![](img/3e9367e5826886294bccbb08463f3e34.jpg "酷")

`http://www.team509.com/modules.php?name=News&file=article&sid=80`

谢谢 7 楼的兄弟帮我写注！

支持，很好的内容。

LZ 的文档和测试程序，我本地一份吧![](img/3e9367e5826886294bccbb08463f3e34.jpg "酷")

> 引用: `IDA` 和 `DumpBin` 等工具在检测 `Tls Callback` 函数时存在的一个问题，及解决方案

详见 `http://www.team509.com/loader.rar`

| 附件 |
| :--- |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `loader.rar` |

应该说楼主言简意赅，一语说出了关键所在：

系统对 `PE` 文件的 `Loader` 在检测是否有 `TLS Callback` 存在并调用函数时，读取记录 `TLS` 的 `IMAGE_DATA_DIRECTORY` 结构，从其 `VirtualAddress` 成员中得到 `IMAGE_TLS_DIRECTORY` 结构的偏移（加上基址变成指针），但并未检测其 `Size` 成员。因此即使此处 `Size` 成员置 `0`，依然可以读取并调用 `TLS Callback` 函数。

但 `IDA`、`DumpBin` 等工具在检测 `TLS Callback` 时，则检测了相应 `IMAGE_DATA_DIRECTORY` 结构的 `Size` 成员。

因此，当编写者有意将相应 `IMAGE_DATA_DIRECTORY` 结构的 `Size` 成员置 `0` 时，`IDA` 和 `DumpBin` 等工具将认为这个 `TLS Callback` 无效，然而 `PE` 的 `Loader` 却不检测 `Size` 成员而依然认为其有效并正常执行 `TLS Callback`，从而导致 `IDA` 等对此 `TLS Callback` 的误判。

接下来我验证一下楼主所提到的内容。

`PE` 的 `Loader` 对 `TLS Callback` 的读取和调用在 `ntdll!LdrpCallTlsInitializers`

> 引用: `.text:7C948578`

```
.text:7C948578 ; ********************************************************
.text:7C948578 ;
.text:7C948578 ; Attributes: bp-based frame
.text:7C948578 ;
.text:7C948578 ; int __stdcall LdrpCallTlsInitializers(int ImageBase,int InitTls)
.text:7C948578 __stdcall LdrpCallTlsInitializers(x, x) proc near
.text:7C948578 ; CODE XREF: LdrpInitializeThread(x)+EDCDp
.text:7C948578 ; LdrShutdownThread()+16941p
.text:7C948578 ; LdrpRunInitializeRoutines(x)+13FDFp
.text:7C948578 ; LdrShutdownThread()+2231Cp
.text:7C948578 ; LdrpInitializeThread(x)+1AD39p
.text:7C948578 ; LdrpRunInitializeRoutines(x)+19B86p ...
.text:7C948578
.text:7C948578 pSize= dword ptr -28h
.text:7C948578 AddressOfCallBacks= dword ptr -20h
.text:7C948578 CallBackFunction= dword ptr -1Ch
.text:7C948578 ms_exc = CPPEH_RECORD ptr -18h
.text:7C948578 ImageBase = dword ptr 8
.text:7C948578 InitTls = dword ptr 0Ch
.text:7C948578
.text:7C948578 push 18h
.text:7C94857A push offset dword_7C9485D0
.text:7C94857F call __SEH_prolog
.text:7C94857F
.text:7C948584 lea eax, [ebp+pSize]
.text:7C948587 push eax
.text:7C948588 push IMAGE_DIRECTORY_ENTRY_TLS
.text:7C94858A push 1
.text:7C94858C mov edi, [ebp+ImageBase]
.text:7C94858F push edi
.text:7C948590 call RtlImageDirectoryEntryToData(x,x,x,x) ; 取 IMAGE_TLS_DIRECTORY 结构指针
.text:7C948590
.text:7C948595 xor ebx, ebx
.text:7C948597 mov [ebp+ms_exc.disabled], ebx
.text:7C94859A cmp eax, ebx ; 非 0 即为成功，下面读取其中的 AddressOfCallBacks 数组，循环调用其中的每个 CallBack 函数。注意这里没有检查相应的 DataDirectory 中的 Size。
.text:7C94859C jz short loc_7C9485BE
.text:7C94859C
.text:7C94859E mov esi, [eax+IMAGE_TLS_DIRECTORY.AddressOfCallBacks]
.text:7C9485A1 mov [ebp+AddressOfCallBacks], esi
.text:7C9485A4 cmp esi, ebx
.text:7C9485A6 jz short loc_7C9485BE
.text:7C9485A6
.text:7C9485A8 cmp _ShowSnaps, bl
.text:7C9485AE jnz loc_7C95CA9E
.text:7C9485AE
.text:7C9485B4
.text:7C9485B4 loc_7C9485B4: ; CODE XREF: LdrpGenericExceptionFilter(x,x)+4589j
.text:7C9485B4 ; LdrpGenericExceptionFilter(x,x)+45BBj
.text:7C9485B4 mov eax, [esi]
.text:7C9485B6 cmp eax, ebx
.text:7C9485B8 jnz loc_7C95CAB3
.text:7C9485B8
.text:7C9485BE
.text:7C9485BE loc_7C9485BE: ; CODE XREF: LdrpCallTlsInitializers(x,x)+24j
.text:7C9485BE ; LdrpCallTlsInitializers(x,x)+2Ej
.text:7C9485BE ; LdrpGenericExceptionFilter(x,x)+4614j
.text:7C9485BE or [ebp+ms_exc.disabled], 0FFFFFFFFh
.text:7C9485C2 call __SEH_epilog
.text:7C9485C2
.text:7C9485C7 retn 8
.text:7C9485C7
.text:7C9485C7 __stdcall LdrpCallTlsInitializers(x, x) endp
.text:7C9485C7
.text:7C9485C7 ; ---------------------------------------------------------------------------
.text:7C95CA9E ; ---------------------------------------------------------------------------
.text:7C95CA9E
.text:7C95CA9E loc_7C95CA9E: ; CODE XREF: LdrpCallTlsInitializers(x,x)+36j
.text:7C95CA9E push esi
.text:7C95CA9F push eax
.text:7C95CAA0 push edi
.text:7C95CAA1 push offset s_LdrTlsCallbac ; "LDR: Tls Callbacks Found. Imagebase %p" ...
.text:7C95CAA6 call _DbgPrint
.text:7C95CAA6
.text:7C95CAAB add esp, 10h
.text:7C95CAAE jmp loc_7C9485B4
.text:7C95CAAE
.text:7C95CAB3 ; ---------------------------------------------------------------------------
.text:7C95CAB3
.text:7C95CAB3 loc_7C95CAB3: ; CODE XREF: LdrpCallTlsInitializers(x,x)+40j
.text:7C95CAB3 mov [ebp-1Ch], eax
.text:7C95CAB6 add esi, 4
.text:7C95CAB9 mov [ebp+AddressOfCallBacks], esi
.text:7C95CABC cmp _ShowSnaps, bl
.text:7C95CAC2 jz short loc_7C95CAD3
.text:7C95CAC2
.text:7C95CAC4 push eax
.text:7C95CAC5 push edi
.text:7C95CAC6 push offset s_LdrCallingTls ; "LDR: Calling Tls Callback Imagebase %p" ...
.text:7C95CACB call _DbgPrint
.text:7C95CACB
.text:7C95CAD0 add esp, 0Ch
.text:7C95CAD0
.text:7C95CAD3
.text:7C95CAD3 loc_7C95CAD3: ; CODE XREF: LdrpGenericExceptionFilter(x,x)+459Dj
.text:7C95CAD3 push ebx ; Reserved
.text:7C95CAD4 push dword ptr [ebp+0Ch] ; Reason
.text:7C95CAD7 push edi ; ImageBase
.text:7C95CAD8 push dword ptr [ebp-1Ch] ; CallBackFunction
.text:7C95CADB call LdrpCallInitRoutine(x,x,x,x)
.text:7C95CADB
.text:7C95CAE0 jmp loc_7C9485B4
.text:7C95CAE0
.text:7C95CAE0 ; ---------------------------------------------------------------------------
```

可以看到，在调用 `RtlImageDirectoryEntryToData` 得到 `TLS` 对应的 `IMAGE_TLS_DIRECTORY` 结构的指针后，并没有检查相应 `IMAGE_DATA_DIRECTORY` 结构的 `Size` 成员（其实 `RtlImageDirectoryEntryToData` 除了返回结构指针外，`IMAGE_DATA_DIRECTORY` 结构的 `Size` 成员同样写入最后一个参数提供的地址中返回给调用者了，但是这里没有检查这个值）就直接查找 `IMAGE_TLS_DIRECTORY` 结构中的函数指针并循环调用了。
