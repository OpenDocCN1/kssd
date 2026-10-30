
# [原创]Windows Vista/7 内核提权`NtGdiEnableEudc` 0day漏洞分析

作者：pcasa of Code Audit Labs , 8efK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6g2k6#2L8r3&6t1N6h3&6@1i4K6u0W2j5$3!0E0

## 1 Affected Products

Windows Vista, Window 7

## 2 Vulnerability Details

用户在启用自定义字符字体时，内核模块从注册表读取数据时，未对注册表中值的类型做检查，直接把数据读取到栈中，产出栈溢出，覆盖函数的返回地址，导致在内核中执行任意代码。

## 3 Analysis

以Windows7 7600版本中的`win32k.sys` 6.1.7600.16667为例,64位和Vista略有不同，可参考POC

首先将溢出的缓冲区字符写入到注册表中

- 项：`[HKEY_CURRENT_USER\EUDC\<当前代码页>]`
- 键：`SystemDefaultEUDCFont`
- 值的类型：`REG_BIN`
- 数据长度：`0x28`

然后调用 `GDI32.DLL` 的 `EnableEUDC(TRUE)`; 进而调用内核中 `win32k!NtGdiEnableEudc` 函数

经过一系列的函数调用，到达读取注册表的数据的地方

```
.text:BF81BA0B     sub_BF81BA0B    proc near               ; CODE XREF: sub_BF81B3B4+B2p
.text:BF81BA0B
.text:BF81BA0B     var_20          = LSA_UNICODE_STRING ptr -20h
.text:BF81BA0B     var_18          = dword ptr -18h
.text:BF81BA0B     var_14          = dword ptr -14h
.text:BF81BA0B     Handle          = dword ptr -10h
.text:BF81BA0B     var_C           = dword ptr -0Ch
.text:BF81BA0B     var_8           = dword ptr -8
.text:BF81BA0B     Path            = dword ptr -4
.text:BF81BA0B     arg_0           = dword ptr  8
.text:BF81BA0B     arg_4           = word ptr  0Ch
.text:BF81BA0B
.text:BF81BA0B 000                 mov     edi, edi
.text:BF81BA0D 000                 push    ebp
.text:BF81BA0E 004                 mov     ebp, esp
.text:BF81BA10 004                 sub     esp, 20h
.text:BF81BA13 024                 push    ebx
.text:BF81BA14 028                 push    esi
.text:BF81BA15 02C                 push    edi
.text:BF81BA16 030                 mov     esi, 208h
.text:BF81BA1B 030                 push    esi
.text:BF81BA1C 034                 lea     ecx, [ebp+var_C]
.text:BF81BA1F 034                 call    ??0MALLOCOBJ@@QAE@K@Z ; MALLOCOBJ::MALLOCOBJ(ulong)
.text:BF81BA24 030                 push    esi
.text:BF81BA25 034                 lea     ecx, [ebp+Path]
.text:BF81BA28 034                 call    ??0MALLOCOBJ@@QAE@K@Z ; MALLOCOBJ::MALLOCOBJ(ulong)
.text:BF81BA2D 030                 mov     ecx, [ebp+var_C]
.text:BF81BA30 030                 xor     esi, esi
.text:BF81BA32 030                 cmp     ecx, esi
.text:BF81BA34 030                 jz      loc_BF81BB20
.text:BF81BA3A 030                 mov     edi, [ebp+Path]
.text:BF81BA3D 030                 cmp     edi, esi
.text:BF81BA3F 030                 jz      loc_BF81BB20
.text:BF81BA45 030                 xor     eax, eax
.text:BF81BA47 030                 mov     [ebp+Handle], esi
.text:BF81BA4A 030                 mov     [ebp+var_14], esi
.text:BF81BA4D 030                 mov     [ecx], ax
.text:BF81BA50 030                 mov     [edi], ax
.text:BF81BA53 030                 mov     [ebp+var_20.Length], ax
.text:BF81BA57 030                 mov     eax, 104h
.text:BF81BA5C 030                 push    eax
.text:BF81BA5D 034                 mov     edx, eax
.text:BF81BA5F 034                 push    edi
.text:BF81BA60 038                 mov     [ebp+var_18], esi
.text:BF81BA63 038                 mov     [ebp+var_20.MaximumLength], dx
.text:BF81BA67 038                 mov     [ebp+var_20.Buffer], ecx
.text:BF81BA6A 038                 call    sub_BF81B946
.text:BF81BA6F 030                 cmp     eax, esi
.text:BF81BA71 030                 mov     [ebp+var_8], eax
.text:BF81BA74 030                 jl      short loc_BF81BAF2
.text:BF81BA76 030                 lea     eax, [ebp+var_18]
.text:BF81BA79 030                 push    eax
.text:BF81BA7A 034                 lea     eax, [ebp+var_14]
.text:BF81BA7D 034                 push    eax
.text:BF81BA7E 038                 lea     eax, [ebp+Handle]
.text:BF81BA81 038                 push    eax
.text:BF81BA82 03C                 push    edi
.text:BF81BA83 040                 call    sub_BF81BBD8
.text:BF81BA88 030                 test    eax, eax
.text:BF81BA8A 030                 jz      short loc_BF81BAEB
.text:BF81BA8C 030                 cmp     [ebp+var_18], esi
.text:BF81BA8F 030                 jz      short loc_BF81BAEB
.text:BF81BA91 030                 push    esi             ; Environment
.text:BF81BA92 034                 push    esi             ; Context
.text:BF81BA93 038                 push    offset ?SharedQueryTable@@3PAU_RTL_QUERY_REGISTRY_TABLE@@A ; QueryTable
.text:BF81BA98 03C                 push    edi             ; Path
.text:BF81BA99 040                 lea     eax, [ebp+var_20] ; 从注册表读取数据的BUFFFER地址，读出的数据保存在栈中
.text:BF81BA9C 040                 push    esi             ; RelativeTo
.text:BF81BA9D 044                 mov     ?SharedQueryTable@@3PAU_RTL_QUERY_REGISTRY_TABLE@@A.QueryRoutine, esi ; _RTL_QUERY_REGISTRY_TABLE * SharedQueryTable
.text:BF81BAA3 044                 mov     ?SharedQueryTable@@3PAU_RTL_QUERY_REGISTRY_TABLE@@A.Flags, RTL_QUERY_REGISTRY_REQUIRED or RTL_QUERY_REGISTRY_DIRECT
.text:BF81BAAD 044                 mov     ?SharedQueryTable@@3PAU_RTL_QUERY_REGISTRY_TABLE@@A.Name, offset aSystemdefaulte ; "SystemDefaultEUDCFont"
.text:BF81BAB7 044                 mov     ?SharedQueryTable@@3PAU_RTL_QUERY_REGISTRY_TABLE@@A.EntryContext, eax
.text:BF81BABC 044                 mov     ?SharedQueryTable@@3PAU_RTL_QUERY_REGISTRY_TABLE@@A.DefaultType, esi
.text:BF81BAC2 044                 mov     ?SharedQueryTable@@3PAU_RTL_QUERY_REGISTRY_TABLE@@A.DefaultData, esi
.text:BF81BAC8 044                 mov     ?SharedQueryTable@@3PAU_RTL_QUERY_REGISTRY_TABLE@@A.DefaultLength, esi
.text:BF81BACE 044                 mov     dword_BFA198FC, esi
.text:BF81BAD4 044                 mov     dword_BFA19900, esi
.text:BF81BADA 044                 mov     dword_BFA19904, esi
.text:BF81BAE0 044                 call    ds:__imp__RtlQueryRegistryValues@20 ; 成功读取后，EntryContext中的前两个DWORD是系统定义的，紧跟是从注册表中读取出来的值
.text:BF81BAE6 030                 mov     [ebp+var_8], eax
.text:BF81BAE9 030                 jmp     short loc_BF81BAF2
```

### 调用`RtlQueryRegistryValues()`前后变量的变化

```
         EBP         返回地址    参数1       参数2

94eb3cd4 14 3d eb 94 6b b4 21 94 90 32 1d fe 04 01 00 00  .=..k.!..2......

kd> r

eax=94eb3cb4 ebx=fe1d3290 ecx=00000001 edx=8b86ff00 esi=00000000 edi=fe721008
eip=9421bae0 esp=94eb3c94 ebp=94eb3cd4 iopl=0         nv up ei pl nz na po nc
cs=0008  ss=0010  ds=0023  es=0023  fs=0030  gs=0000             efl=00000202

win32k!bAppendSysDirectory+0x2de:

9421bae0 ff1504013f94    call    dword ptr [win32k!_imp__RtlQueryRegistryValues (943f0104)] ds:0023:943f0104={nt!RtlQueryRegistryValues (83e41ddb)}

94eb3cb4 00 00 04 01 c0 12 ec fd 01 00 00 00 ac 10 00 80  ................
94eb3cc4 b4 10 00 80 c0 12 ec fd 00 00 00 00 08 10 72 fe  ..............r.
94eb3cd4 14 3d eb 94 6b b4 21 94 90 32 1d fe 04 01 00 00  .=..k.!..2......
94eb3ce4 01 00 00 00 01 00 00 00 36 b9 21 94 00 00 00 00  ........6.!.....

call __imp__RtlQueryRegistryValues@20

kd> r

eax=00000000 ebx=fe1d3290 ecx=00000003 edx=00000002 esi=00000000 edi=fe721008
eip=9421bae6 esp=94eb3ca8 ebp=94eb3cd4 iopl=0         nv up ei pl zr na pe nc
cs=0008  ss=0010  ds=0023  es=0023  fs=0030  gs=0000             efl=00000246

win32k!bAppendSysDirectory+0x2e4:

9421bae6 8945f8          mov     dword ptr [ebp-8],eax ss:0010:94eb3ccc=00000000

94f63cb4 28 00 00 00 03 00 00 00 00 00 00 00 00 00 00 00  (...............
94f63cc4 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
94f63cd4 00 00 00 00 00 00 3c 00 00 00 00 00 00 00 00 00  ......<.........
94f63ce4 01 00 00 00 01 00 00 00 36 b9 21 94 00 00 00 00  ........6.!.....
```

注册表中键的默认类型为`REG_SZ`，故参数类型为`PUNICODE_STRING`，而当前键设置的类型为`REG_BIN`，参数的类型为一个结构，即第一个`DWORD`表示当前缓冲区的长度；同时由于`UNICODE_STRING`结构中的前4个字节一定不为0且符合需要复制的数据空间要求，故在`RtlQueryRegistryValues()`函数中，会错误地把数据直接复制到当前栈中。此时，函数的返回地址被覆盖为 Ring3 中分配的 `Buffer` 地址，同时覆盖了函数的参数值，
