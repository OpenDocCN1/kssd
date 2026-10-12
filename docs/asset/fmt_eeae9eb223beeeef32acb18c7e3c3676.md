
```
10001ACE  |.  68 15157A25   push    257A1515
10001AD3  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]
10001AD6  |.  E8 E5F7FFFF   call    <GetAPIAddressByHash_2>
10001ADB  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]
10001ADE  |.  8981 00010000 mov     dword ptr [ecx+100], eax
10001AE4  |.  68 A168A759   push    59A768A1
10001AE9  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]
10001AEC  |.  E8 CFF7FFFF   call    <GetAPIAddressByHash_2>
10001AF1  |.  8B55 9C       mov     edx, dword ptr [ebp-64]
10001AF4  |.  8982 08010000 mov     dword ptr [edx+108], eax
10001AFA  |.  68 7BBE2E07   push    72EBE7B
10001AFF  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]
10001B02  |.  E8 B9F7FFFF   call    <GetAPIAddressByHash_2>
10001B07  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]
10001B0A  |.  8981 0C010000 mov     dword ptr [ecx+10C], eax
10001B10  |.  68 40954A19   push    194A9540
10001B15  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]
10001B18  |.  E8 A3F7FFFF   call    <GetAPIAddressByHash_2>
10001B1D  |.  8B55 9C       mov     edx, dword ptr [ebp-64]
10001B20  |.  8982 10010000 mov     dword ptr [edx+110], eax
10001B26  |.  8B45 9C       mov     eax, dword ptr [ebp-64]
10001B29  |.  8BE5          mov     esp, ebp
10001B2B  |.  5D            pop     ebp
10001B2C  \.  C3            retn
```

再看 `GetAPIAddressByHash` 这个 `call`. 这个 `call` 是根据函数名的 Hash 值得到其 API 地址并返回的. 如下:

```
10001350 >/$  55            push    ebp
10001351  |.  8BEC          mov     ebp, esp
10001353  |.  83EC 0C       sub     esp, 0C
10001356  |.  894D F4       mov     dword ptr [ebp-C], ecx
10001359  |.  8B4D F4       mov     ecx, dword ptr [ebp-C]
1000135C  |.  E8 5FFDFFFF   call    <GetKernel32base>
10001361  |.  8945 F8       mov     dword ptr [ebp-8], eax
10001364  |.  8B45 08       mov     eax, dword ptr [ebp+8]
10001367  |.  50            push    eax
10001368  |.  8B4D F8       mov     ecx, dword ptr [ebp-8]
1000136B  |.  51            push    ecx
1000136C  |.  8B4D F4       mov     ecx, dword ptr [ebp-C]
1000136F  |.  E8 ACFEFFFF   call    <Get_API_BY_Hash>
10001374  |.  8945 FC       mov     dword ptr [ebp-4], eax
10001377  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
1000137A  |.  8BE5          mov     esp, ebp
1000137C  |.  5D            pop     ebp
1000137D  \.  C2 0400       retn    4
```

其中分两步得到指定 Hash 的 API 地址, 第一步是得到 `Kernel32.dll` 的基址. 然后利用 `Kernel32` 的基址和相应 API 的函数名的 HASH 值去得到其实际地址.

看一下其得到 `Kernel32` 基址的方法(使用了很多 shellcode 中使用的方法):

```
100010C0 >/$  55            push    ebp
100010C1  |.  8BEC          mov     ebp, esp
100010C3  |.  51            push    ecx
100010C4  |.  53            push    ebx
100010C5  |.  894D FC       mov     dword ptr [ebp-4], ecx
100010C8  |.  53            push    ebx
100010C9  |.  51            push    ecx
100010CA  |.  64:8B1D 30000> mov     ebx, dword ptr fs:[30]                 ;  PEB 地址。
100010D1  |.  8B4B 0C       mov     ecx, dword ptr [ebx+C]                  ;  PEB->Ldr (_PEB_LDR_DATA 结构)
100010D4  |.  8B49 1C       mov     ecx, dword ptr [ecx+1C]                 ;  PEB->Ldr->InInitializeationOrderModulelist  (_LIST_ENTRY 结构)
100010D7  |.  8B09          mov     ecx, dword ptr [ecx]                    ;  第一个节点为 ntdll，这句之后，就为下一个节点 kernel32.dll
100010D9  |.  8B41 08       mov     eax, dword ptr [ecx+8]                  ;  加 8 处即为模块基址。
100010DC  |.  59            pop     ecx
100010DD  |.  5B            pop     ebx
100010DE  |.  5B            pop     ebx
100010DF  |.  8BE5          mov     esp, ebp
100010E1  |.  5D            pop     ebp
100010E2  \.  C3            retn
```

得到了模块基址, 然后就是对其导出表中的函数进行遍历查询, 看是否哪个函数的函数名的 Hash 与传进来的 HASH 一致了.

如下:

`<Get_API_BY_Hash>` (模块基址已得到)

```
10001220 >/$  55            push    ebp
10001221  |.  8BEC          mov     ebp, esp
10001223  |.  83EC 1C       sub     esp, 1C
10001226  |.  894D E4       mov     dword ptr [ebp-1C], ecx
10001229  |.  8B45 08       mov     eax, dword ptr [ebp+8]
1000122C  |.  8945 FC       mov     dword ptr [ebp-4], eax
1000122F  |.  8B4D FC       mov     ecx, dword ptr [ebp-4]
10001232  |.  8B55 08       mov     edx, dword ptr [ebp+8]
10001235  |.  0351 3C       add     edx, dword ptr [ecx+3C]
10001238  |.  8955 F4       mov     dword ptr [ebp-C], edx
1000123B  |.  8B45 F4       mov     eax, dword ptr [ebp-C]
1000123E  |.  8B4D 08       mov     ecx, dword ptr [ebp+8]
10001241  |.  0348 78       add     ecx, dword ptr [eax+78]
10001244  |.  894D F8       mov     dword ptr [ebp-8], ecx
10001247  |.  C745 F0 01000> mov     dword ptr [ebp-10], 1
1000124E  |.  EB 09         jmp     short 10001259
10001250  |>  8B55 F0       /mov     edx, dword ptr [ebp-10]
10001253  |.  83C2 01       |add     edx, 1
10001256  |.  8955 F0       |mov     dword ptr [ebp-10], edx
10001259  |>  8B45 F8        mov     eax, dword ptr [ebp-8]
1000125C  |.  8B4D F0       |mov     ecx, dword ptr [ebp-10]
1000125F  |.  3B48 14       |cmp     ecx, dword ptr [eax+14]
10001262  |.  73 53         |jnb     short 100012B7
10001264  |.  8B55 F8       |mov     edx, dword ptr [ebp-8]
10001267  |.  8B45 08       |mov     eax, dword ptr [ebp+8]
1000126A  |.  0342 20       |add     eax, dword ptr [edx+20]
1000126D  |.  8B4D F0       |mov     ecx, dword ptr [ebp-10]
10001270  |.  8B55 08       |mov     edx, dword ptr [ebp+8]
10001273  |.  031488        |add     edx, dword ptr [eax+ecx*4]
10001276  |.  52            |push    edx
10001277  |.  8B4D E4       |mov     ecx, dword ptr [ebp-1C]
1000127A  |.  E8 71FEFFFF   |call    100010F0
1000127F  |.  8945 EC       |mov     dword ptr [ebp-14], eax
10001282  |.  8B45 EC       |mov     eax, dword ptr [ebp-14]
10001285  |.  3B45 0C       |cmp     eax, dword ptr [ebp+C]
10001288  |.  75 2B         |jnz     short 100012B5
1000128A  |.  8B4D F8       |mov     ecx, dword ptr [ebp-8]
1000128D  |.  8B55 08       |mov     edx, dword ptr [ebp+8]
10001290  |.  0351 24       |add     edx, dword ptr [ecx+24]
10001293  |.  8B45 F0       |mov     eax, dword ptr [ebp-10]
10001296  |.  66:8B0C42     |mov     cx, word ptr [edx+eax*2]
1000129A  |.  66:894D E8    |mov     word ptr [ebp-18], cx
1000129E  |.  8B55 F8       |mov     edx, dword ptr [ebp-8]
100012A1  |.  8B45 08       |mov     eax, dword ptr [ebp+8]
100012A4  |.  0342 1C       |add     eax, dword ptr [edx+1C]
100012A7  |.  0FB74D E8     |movzx   ecx, word ptr [ebp-18]
100012AB  |.  8B55 08       |mov     edx, dword ptr [ebp+8]
100012AE  |.  031488        |add     edx, dword ptr [eax+ecx*4]
100012B1  |.  8BC2          |mov     eax, edx
100012B3  |.  EB 04         |jmp     short 100012B9
100012B5  |>^  EB 99        \jmp     short 10001250
100012B7  |>  33C0          xor     eax, eax
100012B9  |>  8BE5          mov     esp, ebp
100012BB  |.  5D            pop     ebp
100012BC  \.  C2 0800       retn    8
```

其中, 在填充 API 地址的过程中, 还有两个函数, 如下:

```
... ...
100016CD  |.  6A 47         push    47
100016CF  |.  6A 0D         push    0D
100016D1  |.  8D45 BC       lea     eax, dword ptr [ebp-44]
100016D4  |.  50            push    eax
100016D5  |.  8D4D BC       lea     ecx, dword ptr [ebp-44]
100016D8  |.  51            push    ecx
100016D9  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]
100016DC  |.  E8 6FF9FFFF   call    <DecodeString>
100016E1  |.  8D55 A0       lea     edx, dword ptr [ebp-60]
100016E4  |.  52            push    edx
100016E5  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]
100016E8  |.  E8 03FBFFFF   call    <GetApiAddressByName>
... ...
```

其中有个字符串揭秘函数: `<DecodeString>` 如下:
