
```
(a34.5ac): Break instruction exception - code 80000003 (first chance)
eax=7ff9b000 ebx=00000000 ecx=00000000 edx=76faec83 esi=00000000 edi=00000000
eip=76f43c48 esp=0736ff5c ebp=0736ff88 iopl=0 nv up ei pl zr na pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000246
ntdll!DbgBreakPoint:
76f43c48 cc int 3
0:016> bp 00465730
\*\*\* ERROR: Module load completed but symbols could not be loaded for C:\Program Files\CyberLink\LabelPrint\LabelPrint.exe
0:016> g
ModLoad: 6da60000 6dab8000 C:\Program Files\Common Files\microsoft shared\ink\tiptsf.dll
ModLoad: 6e710000 6e741000 C:\Windows\system32\EhStorShell.dll
ModLoad: 6e6a0000 6e70a000 C:\Windows\System32\cscui.dll
ModLoad: 6e7e0000 6e7e9000 C:\Windows\System32\CSCDLL.dll
ModLoad: 6fa00000 6fa0b000 C:\Windows\system32\CSCAPI.dll
ModLoad: 6e630000 6e6a0000 C:\Windows\system32\ntshrui.dll
ModLoad: 74cb0000 74cc9000 C:\Windows\system32\srvcli.dll
ModLoad: 73620000 7362a000 C:\Windows\system32\slc.dll
ModLoad: 6e620000 6e626000 C:\Windows\system32\IconCodecService.dll
ModLoad: 71590000 715ec000 C:\Windows\System32\StructuredQuery.dll
ModLoad: 6e300000 6e34e000 C:\Windows\system32\actxprxy.dll
ModLoad: 6acf0000 6ad1b000 C:\Program Files\Internet Explorer\ieproxy.dll
ModLoad: 72430000 72446000 C:\Windows\system32\thumbcache.dll
ModLoad: 6dfc0000 6dfee000 C:\Windows\system32\SHDOCVW.dll
ModLoad: 697f0000 69890000 C:\Windows\system32\SearchFolder.dll
ModLoad: 6ca60000 6cbf8000 C:\Windows\system32\NetworkExplorer.dll
ModLoad: 6f4d0000 6f4d9000 C:\Windows\system32\LINKINFO.dll
ModLoad: 73120000 7312f000 C:\Windows\system32\samcli.dll
ModLoad: 73f80000 73f92000 C:\Windows\system32\SAMLIB.dll
ModLoad: 732c0000 732c9000 C:\Windows\system32\netutils.dll
Breakpoint 0 hit
eax=0710591c ebx=01651fb0 ecx=0012e1b8 edx=00000002 esi=75c13e59 edi=00000000
eip=00465730 esp=0012e19c ebp=0012eee4 iopl=0 nv up ei pl nz na po nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000202
LabelPrint+0x65730:
00465730 6aff push 0FFFFFFFFh
```

断下后运行会发现再次断下，我们在第5次断下的时候再运行才触发异常

```
Breakpoint 0 hit
eax=0710591c ebx=01651fb0 ecx=0012e1b8 edx=00000002 esi=75c13e59 edi=00000000
eip=00465730 esp=0012e19c ebp=0012eee4 iopl=0 nv up ei pl nz na po nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000202
LabelPrint+0x65730:
00465730 6aff push 0FFFFFFFFh
0:000> g
Breakpoint 0 hit
eax=0710578c ebx=01651fb0 ecx=0012e1b8 edx=00000000 esi=75c13e59 edi=00000000
eip=00465730 esp=0012e19c ebp=0012eee4 iopl=0 nv up ei pl nz na po nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000202
LabelPrint+0x65730:
00465730 6aff push 0FFFFFFFFh
0:000> g
Breakpoint 0 hit
eax=0710591c ebx=01651fb0 ecx=0012e1b8 edx=00000000 esi=75c13e59 edi=00000000
eip=00465730 esp=0012e19c ebp=0012eee4 iopl=0 nv up ei pl nz na po nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000202
LabelPrint+0x65730:
00465730 6aff push 0FFFFFFFFh
0:000> g
Breakpoint 0 hit
eax=0710578c ebx=01651fb0 ecx=0012e1b8 edx=00000000 esi=75c13e59 edi=0012e6a0
eip=00465730 esp=0012e19c ebp=0012eee4 iopl=0 nv up ei pl nz na po nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000202
LabelPrint+0x65730:
00465730 6aff push 0FFFFFFFFh
0:000> g
Breakpoint 0 hit
eax=0710591c ebx=01651fb0 ecx=0012e1b8 edx=0012e8a8 esi=75c13e59 edi=00000000
eip=00465730 esp=0012e19c ebp=0012eee4 iopl=0 nv up ei pl nz na po nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000202
LabelPrint+0x65730:
00465730 6aff push 0FFFFFFFFh
0:000> g
(a34.238): Access violation - code c0000005 (first chance)
First chance exceptions are reported before any exception handling.
This exception may be expected and handled.
eax=07190042 ebx=01651fb0 ecx=00130000 edx=07193bec esi=01651ae8 edi=00000000
eip=7c37042b esp=0012e16c ebp=0012eee4 iopl=0 nv up ei pl nz na pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00010206
\*\*\* ERROR: Symbol file could not be found. Defaulted to export symbols for C:\Program Files\CyberLink\LabelPrint\MSVCR71.dll -
MSVCR71!wcscpy+0xb:
7c37042b 668901 mov word ptr [ecx],ax ds:0023:00130000=6341
```

重新加载在第5次断下时，查看此时`esp`，确定函数调用过程：

```
Breakpoint 0 hit
eax=06eafea4 ebx=01581fb0 ecx=0012e1b8 edx=0012e8a8 esi=75c13e59 edi=00000000
eip=00465730 esp=0012e19c ebp=0012eee4 iopl=0 nv up ei pl nz na po nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000202
LabelPrint+0x65730:
00465730 6aff push 0FFFFFFFFh
0:000> dd esp
0012e19c 00410c26 06eafea4 0012e8a8 0012f360
0012e1ac 01581fb0 00000000 06dcd8c0 01afa490
0012e1bc 01afa4ac 01afabd0 06e26658 0012e234
0012e1cc 01afa458 00000000 00000000 01afa3c8
0012e1dc 06e0d464 760dc744 0012e204 75db3a0c
0012e1ec 06e389fe 00000000 0012e234 00000000
0012e1fc 00000000 06eafea4 0012e220 06eafef4
0012e20c 06e389fe 06eafea4 0012e234 06eafef4
0:000> ub 00410c26
LabelPrint+0x10c05:
00410c05 e836b6ffff call LabelPrint+0xc240 (0040c240)
00410c0a 8b00 mov eax,dword ptr [eax]
00410c0c 8d942400070000 lea edx,[esp+700h]
00410c13 52 push edx
00410c14 50 push eax
00410c15 8d4c2418 lea ecx,[esp+18h]
00410c19 c684243c0d000024 mov byte ptr [esp+0D3Ch],24h
00410c21 e80a4b0500 call LabelPrint+0x65730 (00465730) 调用vulfunc函数
```

函数在`00410c21`地址处调用了`vulfunc`函数。再用ida查看此函数：

```
....
v123 = \*(\_DWORD \*\*)sub\_40C240((OLECHAR \*)L"title");
LOBYTE(STACK[0xD30]) = 32;
vulfunc(&p\_bstrString, v123, (wchar\_t \*)(a1 + 5876)); //1
v151 = a18;
LOBYTE(STACK[0xD30]) = 0;
a2(v151);
v124 = \*(\_DWORD \*\*)sub\_40C240((OLECHAR \*)L"author");
LOBYTE(STACK[0xD30]) = 33;
vulfunc(&p\_bstrString, v124, (wchar\_t \*)(a1 + 6396)); //2
v152 = a20;
LOBYTE(STACK[0xD30]) = 0;
a2(v152);
v125 = \*(\_DWORD \*\*)sub\_40C240((OLECHAR \*)L"date");
LOBYTE(STACK[0xD30]) = 34;
vulfunc(&p\_bstrString, v125, (wchar\_t \*)(a1 + 6916)); //3
v153 = a22;
LOBYTE(STACK[0xD30]) = 0;
a2(v153);
LOWORD(STACK[0x2EC]) = 0;
memset(&STACK[0x2EE], 0, 0x204u);
LOWORD(STACK[0x4F2]) = 0;
v126 = \*(\_DWORD \*\*)sub\_40C240((OLECHAR \*)L"SystemTime");
LOBYTE(STACK[0xD30]) = 35;
vulfunc(&p\_bstrString, v126, (wchar\_t \*)&STACK[0x2EC]); //4
......
v106 = \*(\_DWORD \*\*)sub\_40C240((OLECHAR \*)L"name");
LOBYTE(STACK[0xD28]) = 36;
vulfunc(&v156, v106, (wchar\_t \*)&STACK[0x6F4]); //5
```

现在我们知道为什么`vulfunc`要调用5次了。而且这些属性正好和exploit中一一对应。但是在`vulfunc`中仍进行了两次if判断才最终执行`wcscpy`。所以想要继续确定执行流程，还得分析`vulfunc`函数。重新加载在`vulfunc`函数头下断，在第5次断下时单步跟踪确定执行流程：
