
```
0043DDDE   |.  8BF0     mov esi,eax                               ；eax为1，正确；为0，错误

0043DDE0   |.  83C4 08  add esp,8

0043DDE3   |.  F7DE     neg esi

0043DDE5   |.  1BF6     sbb esi,esi

0043DDE7   |.  46       inc esi

0043DDE8   |>  8D4C24 0>lea ecx,dword ptr ss:[esp+8]

0043DDEC   |.  C64424 1>mov byte ptr ss:[esp+1C],2

0043DDF1   |.  E8 27570>call USBTrace.0046351D

0043DDF6   |.  8D4C24 0>lea ecx,dword ptr ss:[esp+C]

0043DDFA   |.  C64424 1>mov byte ptr ss:[esp+1C],1

0043DDFF   |.  E8 19570>call USBTrace.0046351D

0043DE04   |.  8D4C24 2>lea ecx,dword ptr ss:[esp+24]

0043DE08   |.  C64424 1>mov byte ptr ss:[esp+1C],0

0043DE0D   |.  E8 0B570>call USBTrace.0046351D

0043DE12   |.  8D4C24 2>lea ecx,dword ptr ss:[esp+28]

0043DE16   |.  C74424 1>mov dword ptr ss:[esp+1C],-1

0043DE1E   |.  E8 FA560>call USBTrace.0046351D

0043DE23   |.  8B4C24 1>mov ecx,dword ptr ss:[esp+14]

0043DE27   |.  8BC6     mov eax,esi

0043DE29   |.  5F       pop edi

0043DE2A   |.  64:890D >mov dword ptr fs:[0],ecx

0043DE31   |.  5E       pop esi

0043DE32   |.  83C4 18  add esp,18

0043DE35   \.  C2 0800  retn 8
```

## 三、0043DE80 生成注册码的 call：

```
0043DE80   /$  6A FF    push -1

0043DE82   |.  68 A4134>push USBTrace.004813A4                   ;  SE handler installation

0043DE87   |.  64:A1 00>mov eax,dword ptr fs:[0]

0043DE8D   |.  50       push eax

0043DE8E   |.  64:8925 >mov dword ptr fs:[0],esp

0043DE95   |.  81EC 300>sub esp,130

0043DE9B   |.  53       push ebx

0043DE9C   |.  55       push ebp

0043DE9D   |.  56       push esi

0043DE9E   |.  57       push edi

0043DE9F   |.  A1 A0DD4>mov eax,dword ptr ds:[4ADDA0]

0043DEA4   |.  33F6     xor esi,esi

0043DEA6   |.  89B424 4>mov dword ptr ss:[esp+148],esi

0043DEAD   |.  33ED     xor ebp,ebp

0043DEAF   |.  894424 3>mov dword ptr ss:[esp+34],eax

0043DEB3   |.  897424 2>mov dword ptr ss:[esp+28],esi

0043DEB7   |.  897424 2>mov dword ptr ss:[esp+24],esi

0043DEBB   |.  897424 3>mov dword ptr ss:[esp+38],esi

0043DEBF   |.  33FF     xor edi,edi

0043DEC1   |.  894424 1>mov dword ptr ss:[esp+18],eax

0043DEC5   |.  894424 1>mov dword ptr ss:[esp+1C],eax

0043DEC9   |.  894424 1>mov dword ptr ss:[esp+14],eax

0043DECD   |.  8B8424 5>mov eax,dword ptr ss:[esp+150]

0043DED4   |.  C68424 4>mov byte ptr ss:[esp+148],4

0043DEDC   |.  8B40 F8  mov eax,dword ptr ds:[eax-8]              ；用户名的长度

0043DEDF   |.  83F8 04  cmp eax,4                                ；长度要大于等于4

0043DEE2   |.  0F8C F40>jl USBTrace.0043E0DC

0043DEE8   |.  E8 03910>call USBTrace.00476FF0

0043DEED   |.  8B40 08  mov eax,dword ptr ds:[eax+8]             ；eax＝00400000，上面call的返回值

0043DEF0   |.  8D4C24 3>lea ecx,dword ptr ss:[esp+3C]

0043DEF4   |.  68 04010>push 104                                 ; /BufSize = 104 (260.)

0043DEF9   |.  51       push ecx                                 ; |PathBuffer＝0012F228

0043DEFA   |.  50       push eax                                 ; |hModule = 00400000 (USBTrace)

0043DEFB   |.  FF15 EC4>call dword ptr ds:[<&KERNEL32.GetModuleF>; \GetModuleFileNameA
```

The `GetModuleFileName` function retrieves the full path and filename for the executable file containing the specified module.

```
0012F228  43 3A 5C 50 72 6F 67 72  C:\Progr

0012F230  61 6D 20 46 69 6C 65 73  am Files

0012F238  5C 55 53 42 54 72 61 63  \USBTrac

0012F240  65 5C 55 53 42 54 72 61  e\USBTra

0012F248  63 65 2E 65 78 65 00     ce.exe.
```

`eax=0026`,是上面的完整路径的长度。

下面的几个 api 我就不解释了，可以对照 "win32 programmer's reference" 去理解。

前面一小段代码的意思是，得到 `USBTrace.exe` 文件的版本信息。

```
0043DF01   |.  85C0     test eax,eax

0043DF03   |.  0F84 D30>je USBTrace.0043E0DC

0043DF09   |.  8D5424 2>lea edx,dword ptr ss:[esp+24]

0043DF0D   |.  8D4424 3>lea eax,dword ptr ss:[esp+3C]

0043DF11   |.  52       push edx                                 ; /pHandle

0043DF12   |.  50       push eax                                 ; |FileName

0043DF13   |.  E8 84700>call <jmp.&VERSION.GetFileVersionInfoSiz>; \GetFileVersionInfoSizeA

0043DF18   |.  8BD8     mov ebx,eax

0043DF1A   |.  85DB     test ebx,ebx

0043DF1C   |.  0F84 BA0>je USBTrace.0043E0DC

0043DF22   |.  53       push ebx

0043DF23   |.  E8 27040>call USBTrace.0044E34F

0043DF28   |.  83C4 04  add esp,4

0043DF2B   |.  894424 2>mov dword ptr ss:[esp+20],eax

0043DF2F   |.  85C0     test eax,eax

0043DF31   |.  0F84 A50>je USBTrace.0043E0DC

0043DF37   |.  8B4C24 2>mov ecx,dword ptr ss:[esp+24]

0043DF3B   |.  50       push eax                                 ; /Buffer＝003F82B0

0043DF3C   |.  53       push ebx                                 ; |BufSize

0043DF3D   |.  8D5424 4>lea edx,dword ptr ss:[esp+44]            ; |

0043DF41   |.  51       push ecx                                 ; |Reserved

0043DF42   |.  52       push edx                                 ; |FileName

0043DF43   |.  E8 4E700>call <jmp.&VERSION.GetFileVersionInfoA>  ; \GetFileVersionInfoA

0043DF48   |.  85C0     test eax,eax

0043DF4A   |.  0F84 7F0>je USBTrace.0043E0CF

0043DF50   |.  8B5424 2>mov edx,dword ptr ss:[esp+20]            ；003F82B0，Buffer

0043DF54   |.  8D4424 3>lea eax,dword ptr ss:[esp+38]

0043DF58   |.  8D4C24 2>lea ecx,dword ptr ss:[esp+28]

0043DF5C   |.  50       push eax                                 ; /pValueSize

0043DF5D   |.  51       push ecx                                 ; |ppValue

0043DF5E   |.  68 00E94>push USBTrace.0049E900                   ; |pSubBlock = "\"

0043DF63   |.  52       push edx                                 ; |pBlock＝003F82B0

0043DF64   |.  E8 27700>call <jmp.&VERSION.VerQueryValueA>       ; \VerQueryValueA
```

`pValueSize` 所指的 size 为 0，可见并未取出数据。

`pBlock＝003F82B0`，指向取出的版本信息。

下面是部分版本信息，可用资源查看工具去看。

```
003F82B0  84 03 34 00 00 00 56 00  ?4...V.

003F82B8  53 00 5F 00 56 00 45 00  S._.V.E.

003F82C0  52 00 53 00 49 00 4F 00  R.S.I.O.

003F82C8  4E 00 5F 00 49 00 4E 00  N._.I.N.

003F82D0  46 00 4F 00 00 00 00 00  F.O.....

003F82D8  BD 04 EF FE 00 00 01 00  ?稔...

003F82E0  00 00 01 00 02 00 00 00  ......

003F82E8  00 00 01 00 02 00 00 00  ......

003F82F0  3F 00 00 00 00 00 00 00  ?.......

003F82F8  04 00 00 00 01 00 00 00  ......

003F8300  00 00 00 00 00 00 00 00  ........

003F8308  00 00 00 00 E4 02 00 00  ....?..
```

下面是版本数据的结构：

```
VS_VERSION_INFO {
    WORD  wLength;
    WORD  wValueLength;
    WORD  wType;
    WCHAR szKey[];
    WORD  Padding1[];
    VS_FIXEDFILEINFO Value;
    WORD  Padding2[];
    WORD  Children[];
};

typedef struct _VS_FIXEDFILEINFO {  // vsffi
    DWORD dwSignature;           //Contains the value 0xFEEFO4BD
    DWORD dwStrucVersion;
    DWORD dwFileVersionMS;
    DWORD dwFileVersionLS;
    DWORD dwProductVersionMS;
    DWORD dwProductVersionLS;
    DWORD dwFileFlagsMask;
    DWORD dwFileFlags;
    DWORD dwFileOS;
    DWORD dwFileType;
    DWORD dwFileSubtype;
    DWORD dwFileDateMS;
    DWORD dwFileDateLS;
} VS_FIXEDFILEINFO;
```

对照上面的数据可以很好的去理解结构的各个字段。

```
0043DF69   |.  85C0     test eax,eax

0043DF6B   |.  0F84 5E0>je USBTrace.0043E0CF
```

下面是关键部分了，取出版本中的信息。

下面一句执行后 `eax` 为 `003F82D8`，指向 `VS_FIXEDFILEINFO` 结构，通过 `dwSignature` 可认出来。
