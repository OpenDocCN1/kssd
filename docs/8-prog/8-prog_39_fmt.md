
# 实用补丁:VC6 IDE 字体设置增强

> 原文：[`www.pediy.com/kssd/pediy10/78351.html`](https://www.pediy.com/kssd/pediy10/78351.html)

**VC6 IDE 字体设置增强**

by `morning`

`VC6` 只枚举系统缺省字符集一致的字体,这导致选择字体时甚为不便.虽然说可以通过修改注册表,直接设置字体,但是总觉得有些遗憾,今天有点时间,就小小 `patch` 了一下.

## 函数原型:

```
int EnumFontFamilies(
  HDC hdc,                      // handle to DC
  LPCTSTR lpszFamily,          // font family
  FONTENUMPROC lpEnumFontFamProc, // callback function
  LPARAM lParam               // additional data
);
```

根据 `PSDK` 的解释,显然 `lpszFamily=NULL` 时的枚举值得我们关注

用 `OD` 启动 `VC6`,下断点:`EnumFontFamilies`

点击菜单 `Tools->Options`,切换到 `format` 标签,出现我们所需的中断

### 堆栈情况:

```
0013EC9C   50089D1F  /CALL to EnumFontFamiliesA from devshl.50089D19
0013ECA0   0101005B  |hDC = 0101005B
0013ECA4   00000000  |FamilyName = NULL
0013ECA8   50089D96  |CallbackFunc = devshl.#1969
0013ECAC   00D20720  \lParam = 00D20720
```

前往 `devshl.#1969`

### 代码:

```
50089D96 >   B8 8D510D50     mov     eax, 500D518D
50089D9B    E8 CA89F7FF     call    5000276A
50089DA0    56              push    esi
50089DA1    57              push    edi
50089DA2    E8 BD89F7FF     call    <jmp.&MFC42.#1168_AfxGetModuleState>
50089DA7    8B40 04         mov     eax, dword ptr [eax+4]
50089DAA    6A 00           push    0
50089DAC    8BC8            mov     ecx, eax
50089DAE    E8 ED53F8FF     call    #2291
50089DB3    8B4D 0C         mov     ecx, dword ptr [ebp+C]
50089DB6    83F8 01         cmp     eax, 1
50089DB9    74 13           je      short 50089DCE
50089DBB    85C0            test    eax, eax
50089DBD    74 0F           je      short 50089DCE
50089DBF    8A51 34         mov     dl, byte ptr [ecx+34]  //这里是字符集
50089DC2    0FB6F2          movzx   esi, dl
50089DC5    3BC6            cmp     eax, esi
50089DC7    74 05           je      short 50089DCE    //改成 jmp 即可
50089DC9    80FA FF         cmp     dl, 0FF
50089DCC    75 58           jnz     short 50089E26
50089DCE    8B7D 08         mov     edi, dword ptr [ebp+8]
50089DD1    807F 1C 40      cmp     byte ptr [edi+1C], 40
50089DD5    8D57 1C         lea     edx, dword ptr [edi+1C]
50089DD8    74 4C           je      short 50089E26
50089DDA    8A41 33         mov     al, byte ptr [ecx+33]
50089DDD    8B75 14         mov     esi, dword ptr [ebp+14]
50089DE0    24 01           and     al, 1
50089DE2    74 06           je      short 50089DEA
50089DE4    837E 6C 00      cmp     dword ptr [esi+6C], 0
50089DE8    75 0A           jnz     short 50089DF4
50089DEA    84C0            test    al, al
50089DEC    75 38           jnz     short 50089E26
50089DEE    837E 6C 00      cmp     dword ptr [esi+6C], 0
50089DF2    75 32           jnz     short 50089E26
50089DF4    52              push    edx
50089DF5    8D4D 0C         lea     ecx, dword ptr [ebp+C]
50089DF8    E8 938EF7FF     call    <jmp.&MFC42.#537_CString::CString>
50089DFD    0FB647 1B       movzx   eax, byte ptr [edi+1B]
50089E01    0FB74D 10       movzx   ecx, word ptr [ebp+10]
50089E05    8365 FC 00      and     dword ptr [ebp-4], 0
50089E09    C1E0 10         shl     eax, 10
50089E0C    0BC1            or      eax, ecx
50089E0E    8BCE            mov     ecx, esi
50089E10    50              push    eax
50089E11    8D45 0C         lea     eax, dword ptr [ebp+C]
50089E14    50              push    eax
50089E15    E8 47FEFFFF     call    #984
50089E1A    834D FC FF      or      dword ptr [ebp-4], FFFFFFFF
50089E1E    8D4D 0C         lea     ecx, dword ptr [ebp+C]
50089E21    E8 4C8EF7FF     call    <jmp.&MFC42.#800_CString::~CString>
50089E26    8B4D F4         mov     ecx, dword ptr [ebp-C]
50089E29    6A 01           push    1
50089E2B    58              pop     eax
50089E2C    5F              pop     edi
50089E2D    5E              pop     esi
50089E2E    64:890D 0000000>mov     dword ptr fs:[0], ecx
50089E35    C9              leave
50089E36    C2 1000         retn    10
```

注意 `50089DC7` 附近的注释,改成 `jmp`,跳过字符集检查.

结论:修改文件偏移 `8ADC7` 处,把 `74` 改成 `EB`(此偏移仅适合 `SP6`,其他版本请自行调试)

收工~
