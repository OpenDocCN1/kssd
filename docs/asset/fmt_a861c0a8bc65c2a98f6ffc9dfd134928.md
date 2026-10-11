
# badcat21

> 原文：[`www.pediy.com/kssd/pediy03/forum1007.htm`](https://www.pediy.com/kssd/pediy03/forum1007.htm)

这个软件极弱，正好作为破解第一课。

运行`badcat21`，选注册，填`121212121`，按`ctrl+n`，下`bpx hmemcpy`，`x`退出点注册，`pmodule`到它领空，按`f12`，一次就退了，弱吧，嘿嘿

原样再来一便，到这里。

按`f10`往下，

```
:00482DD5 3BC6                 cmp eax, esi

:00482DD7 DBE2                 fclex

:00482DD9 7D12                 jge 00482DED

:00482DDB 68A0000000           push 000000A0

* Possible StringData Ref from Code Obj ->"酦?檉??"

|

:00482DE0 681CA04100           push 0041A01C

:00482DE5 53                   push ebx

:00482DE6 50                   push eax

* Reference To: MSVBVM60.__vbaHResultCheckObj, Ord:0000h

|

:00482DE7 FF1558104000         Call dword ptr [00401058]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00482DD9(C)

|

:00482DED 8B45E0               mov eax, dword ptr [ebp-20]

:00482DF0 8D4DBC               lea ecx, dword ptr [ebp-44]

:00482DF3 8945D4               mov dword ptr [ebp-2C], eax

:00482DF6 8D45CC               lea eax, dword ptr [ebp-34]

:00482DF9 50                   push eax

:00482DFA 51                   push ecx

:00482DFB 8975E0               mov dword ptr [ebp-20], esi

:00482DFE C745CC08000000       mov [ebp-34], 00000008

* Reference To: MSVBVM60.rtcTrimVar, Ord:0208h

|

:00482E05 FF15C4104000         Call dword ptr [004010C4]

:00482E0B 8B55E8               mov edx, dword ptr [ebp-18]
```

来到这里。这个`edx`就是算出来的注册码

再往下就判断退出了。

```
:00482E0E B80B000000           mov eax, 0000000B

:00482E13 89857CFFFFFF         mov dword ptr [ebp+FFFFFF7C], eax

:00482E19 89458C               mov dword ptr [ebp-74], eax

:00482E1C 8D45BC               lea eax, dword ptr [ebp-44]

:00482E1F 899564FFFFFF         mov dword ptr [ebp+FFFFFF64], edx

:00482E25 8D8D5CFFFFFF         lea ecx, dword ptr [ebp+FFFFFF5C]

:00482E2B 50                   push eax

:00482E2C 8D55AC               lea edx, dword ptr [ebp-54]

:00482E2F 51                   push ecx

:00482E30 52                   push edx

:00482E31 C74584FFFFFFFF       mov [ebp-7C], FFFFFFFF

:00482E38 897594               mov dword ptr [ebp-6C], esi

:00482E3B C7855CFFFFFF08800000 mov dword ptr [ebp+FFFFFF5C], 00008008

* Reference To: MSVBVM60.__vbaVarCmpNe, Ord:0000h

|

:00482E45 FF154C104000         Call dword ptr [0040104C]

:00482E4B 8BD0                 mov edx, eax

:00482E4D 8D4D9C               lea ecx, dword ptr [ebp-64]

* Reference To: MSVBVM60.__vbaVarMove, Ord:0000h

|

:00482E50 FF1514104000         Call dword ptr [00401014]

:00482E56 8D857CFFFFFF         lea eax, dword ptr [ebp+FFFFFF7C]

:00482E5C 8D4D8C               lea ecx, dword ptr [ebp-74]

:00482E5F 50                   push eax

:00482E60 8D559C               lea edx, dword ptr [ebp-64]

:00482E63 51                   push ecx

:00482E64 8D856CFFFFFF         lea eax, dword ptr [ebp+FFFFFF6C]

:00482E6A 52                   push edx

:00482E6B 50                   push eax

* Reference To: MSVBVM60.rtcImmediateIf, Ord:02A9h

|

:00482E6C FF15E0114000         Call dword ptr [004011E0]

:00482E72 8D8D6CFFFFFF         lea ecx, dword ptr [ebp+FFFFFF6C]

:00482E78 51                   push ecx

* Reference To: MSVBVM60.__vbaBoolVar, Ord:0000h

|

:00482E79 FF15C0104000         Call dword ptr [004010C0]

:00482E7F 8D4DDC               lea ecx, dword ptr [ebp-24]

:00482E82 8945E4               mov dword ptr [ebp-1C], eax

* Reference To: MSVBVM60.__vbaFreeObj, Ord:0000h

|

:00482E85 FF1578124000         Call dword ptr [00401278]

:00482E8B 8D956CFFFFFF         lea edx, dword ptr [ebp+FFFFFF6C]

:00482E91 8D857CFFFFFF         lea eax, dword ptr [ebp+FFFFFF7C]

:00482E97 52                   push edx

:00482E98 8D4D8C               lea ecx, dword ptr [ebp-74]

:00482E9B 50                   push eax

:00482E9C 8D559C               lea edx, dword ptr [ebp-64]

:00482E9F 51                   push ecx

:00482EA0 8D45BC               lea eax, dword ptr [ebp-44]

:00482EA3 52                   push edx

:00482EA4 8D4DCC               lea ecx, dword ptr [ebp-34]

:00482EA7 50                   push eax

:00482EA8 51                   push ecx

:00482EA9 6A06                 push 00000006

* Reference To: MSVBVM60.__vbaFreeVarList, Ord:0000h

|

:00482EAB FF1534104000         Call dword ptr [00401034]

:00482EB1 8B17                 mov edx, dword ptr [edi]

:00482EB3 83C41C               add esp, 0000001C

:00482EB6 8D45E4               lea eax, dword ptr [ebp-1C]

:00482EB9 50                   push eax

:00482EBA 57                   push edi

:00482EBB FF9204070000         call dword ptr [edx+00000704]
```

很简单吧？第一次写破解。

下弦月 `4dd466d0e1e0cc123afbe504da2462ca`
