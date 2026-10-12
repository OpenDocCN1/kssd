
# Aurora Media Workshop 3.4.2 注册过程

> 原文：[`www.pediy.com/kssd/pediy10/62749.html`](https://www.pediy.com/kssd/pediy10/62749.html)

【文章标题】: `Aurora Media Workshop 3.4.2` 注册过程
【文章作者】: `thdzhqg`
【软件名称】: `Aurora Media Workshop`
【软件大小】: `4.66 MB`
【下载地址】: 华军软件园
【加壳方式】: 无
【保护方式】: 注册码
【编写语言】: `Microsoft Visual C++ 6.0`
【使用工具】: `od`
【操作平台】: `winxp`
【软件介绍】: 国外一款多媒体编辑软件
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

## 【详细过程】

使用 `od` 载入查找字符串 `'invalid username or registration code'` 来到

```
0045BDA0    .  6A FF            push    -1                                ;  (initial cpu selection)
0045BDA2    .  68 18204700      push    00472018                          ;  SE 处理程序安装
0045BDA7    .  64:A1 0000000>   mov     eax, fs:[0]
0045BDAD    .  50               push    eax
0045BDAE    .  64:8925 00000>   mov     fs:[0], esp
0045BDB5    .  51               push    ecx
0045BDB6    .  53               push    ebx
0045BDB7    .  56               push    esi
0045BDB8    .  8BF1             mov     esi, ecx
0045BDBA    .  57               push    edi
0045BDBB    .  8D4C24 0C        lea     ecx, [esp+C]
0045BDBF    .  E8 64A40000      call    <jmp.&MFC42.#540_CString::CString>
0045BDC4    .  6A 01            push    1                                 ;  在 0045BDC4 处 F2，不然会跳到注册失败，seh？
0045BDC6    .  8BCE             mov     ecx, esi
0045BDC8    .  C74424 1C 000>   mov     dword ptr [esp+1C], 0
0045BDD0    .  E8 D3A80000     call    <jmp.&MFC42.#6334_CWnd::UpdateData>
0045BDD5    .  E8 FCA70000     call    <jmp.&MFC42.#1168_AfxGetModuleState>
0045BDDA    .  8B48 04         mov     ecx, [eax+4]
0045BDDD    .  E8 08A90000     call    <jmp.&MFC42.#1669_CCmdTarget::BeginWaitCu>
0045BDE2    .  68 E8030000     push    3E8                               ; /Timeout = 1000. ms
0045BDE7    .  FF15 3C324700   call    [<&KERNEL32.Sleep>]               ; \\Sleep
0045BDED    .  E8 E4A70000     call    <jmp.&MFC42.#1168_AfxGetModuleState>
0045BDF2    .  8B48 04         mov     ecx, [eax+4]
0045BDF5    .  E8 EAA80000     call    <jmp.&MFC42.#2652_CCmdTarget::EndWaitCurs>
0045BDFA    .  8B46 64         mov     eax, [esi+64]
0045BDFD    .  8B4E 60         mov     ecx, [esi+60]
0045BE00    .  8D5E 64         lea     ebx, [esi+64]
0045BE03    .  8D7E 60         lea     edi, [esi+60]
0045BE06    .  50               push    eax
0045BE07    .  51               push    ecx
0045BE08    .  E8 733F0000     call    0045FD80                          ;  关键 call 跟进
0045BE0D    .  83C4 08         add     esp, 8
0045BE10    .  85C0             test    eax, eax
0045BE12    .  75 1F             jnz     short 0045BE33                  ;  关键比较
0045BE14    .  6A 40              push    40                              ;  (initial cpu selection)
0045BE16    .  68 9C4C4900       push    00494C9C                        ;  sorry
0045BE1B    .  68 704C4900       push    00494C70                        ;  invalid username or registration code
0045BE20    .  8BCE               mov     ecx, esi
0045BE22    .  E8 7DA90000      call    <jmp.&MFC42.#4224_CWnd::MessageBoxA>
0045BE27    .  C705 D8FB4D00>   mov     dword ptr [4DFBD8], 0
0045BE31    .  EB 76               jmp     short 0045BEA9
0045BE33    >  8B07                mov     eax, [edi]
0045BE35    .  8D4C24 0C         lea     ecx, [esp+C]
0045BE39    .  50               push    eax
0045BE3A    .  68 544C4900       push    00494C54                        ;  license to:%s
0045BE3F    .  51               push    ecx
0045BE40    .  E8 97A70000      call    <jmp.&MFC42.#2818_CString::Format>
0045BE45    .  8B5424 18        mov     edx, [esp+18]
0045BE49    .  83C4 0C          add     esp, 0C
0045BE4C    .  8BCE               mov     ecx, esi
0045BE4E    .  6A 40              push    40
0045BE50    .  68 484C4900       push    00494C48                        ;  thank you
0045BE55    .  52               push    edx
0045BE56    .  E8 49A90000      call    <jmp.&MFC42.#4224_CWnd::MessageBoxA>
0045BE5B    .  57               push    edi
0045BE5C    .  B9 D4D24900      mov     ecx, 0049D2D4
0045BE61    .  E8 CEA30000      call    <jmp.&MFC42.#858_CString::operator=>
0045BE66    .  53               push    ebx
0045BE67    .  B9 D8D24900      mov     ecx, 0049D2D8
0045BE6C    .  E8 C3A30000      call    <jmp.&MFC42.#858_CString::operator=>
0045BE71    .  C705 D8FB4D00>   mov     dword ptr [4DFBD8], 1
0045BE7B    .  8B3F               mov     edi, [edi]
0045BE7D    .  57               push    edi
0045BE7E    .  68 A0C24800       push    0048C2A0                        ;  username
0045BE83    .  68 94C24800       push    0048C294                        ;  register
0045BE88    .  B9 C4D14900      mov     ecx, 0049D1C4
0045BE8D    .  E8 5EC7FEFF      call    004485F0
0045BE92    .  8B1B               mov     ebx, [ebx]
0045BE94    .  B9 C4D14900      mov     ecx, 0049D1C4
0045BE99    .  53               push    ebx
0045BE9A    .  68 84C24800       push    0048C284                        ;  registercode
0045BE9F    .  68 94C24800       push    0048C294                        ;  register
0045BEA4    .  E8 47C7FEFF      call    004485F0
0045BEA9    >  8BCE                mov     ecx, esi
0045BEAB    .  E8 04A80000      call    <jmp.&MFC42.#4853_CDialog::OnOK>
0045BEB0    .  8D4C24 0C        lea     ecx, [esp+C]
0045BEB4    .  C74424 18 FFF>   mov     dword ptr [esp+18], -1
0045BEBC    .  E8 1FA30000      call    <jmp.&MFC42.#800_CString::~CString>
0045BEC1    .  8B4C24 10        mov     ecx, [esp+10]
0045BEC5    .  5F               pop     edi
0045BEC6    .  5E               pop     esi
0045BEC7    .  5B               pop     ebx
0045BEC8    .  64:890D 00000>   mov     fs:[0], ecx
0045BECF    .  83C4 10          add     esp, 10
0045BED2    .  C3               retn
```

---

### 进入关键 `call`

```
0045FD80  /$  53              push    ebx
0045FD81  |.  55              push    ebp
0045FD82  |.  8B6C24 0C       mov     ebp, [esp+C]
0045FD86  |.  56              push    esi
0045FD87  |.  57              push    edi
0045FD88  |.  BE 54D14900     mov     esi, 0049D154                   ;  下面判断用户名第一位是不是 0
0045FD8D  |.  8BC5             mov     eax, ebp                       ;  用户名入 eax
0045FD8F  |>  8A10            /mov    dl, [eax]                       ;  用户名第一位入 dl
0045FD91  |.  8A1E            |mov    bl, [esi]                       ;  0 入 bl
0045FD93  |.  8ACA            |mov    cl, dl                          ;  t 和 x 比较
0045FD95  |.  3AD3            |cmp     dl, bl                          ;  t 和 0 比较
0045FD97  |.  75 1E           |jnz     short 0045FDB7
0045FD99  |.  84C9            |test    cl, cl
0045FD9B  |.  74 16           |je      short 0045FDB3
0045FD9D  |.  8A50 01         |mov    dl, [eax+1]
0045FDA0  |.  8A5E 01         |mov    bl, [esi+1]
0045FDA3  |.  8ACA            |mov    cl, dl
0045FDA5  |.  3AD3            |cmp     dl, bl
0045FDA7  |.  75 0E           |jnz     short 0045FDB7
0045FDA9  |.  83C0 02         |add     eax, 2
0045FDAC  |.  83C6 02         |add     esi, 2
0045FDAF  |.  84C9            |test    cl, cl
0045FDB1  |.^ 75 DC           \\jnz    short 0045FD8F
0045FDB3  |>  33C0             xor     eax, eax
0045FDB5  |.  EB 05            jmp     short 0045FDBC
0045FDB7  |>  1BC0             sbb     eax, eax                       ;  借位减=0
0045FDB9  |.  83D8 FF         sbb     eax, -1                          ;  =1
0045FDBC  |>  85C0             test    eax, eax
```
