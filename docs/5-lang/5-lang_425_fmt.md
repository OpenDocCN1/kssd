
# 音速启动密码全分析

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-480.htm`](https://www.pediy.com/kssd/pediy08/pediy8-480.htm)

- **【文章标题】**: 音速启动密码全分析
- **【文章作者】**: `WAKU`
- **【作者邮箱】**: `wakuwakuwawaku@163.com`
- **【软件名称】**: 音速启动
- **【软件版本】**: `5.0 Build 2006.9.25`
- **【加壳方式】**: `UPX`
- **【编写语言】**: `VB`
- **【使用工具】**: `FileMon`, `VBExplorer`, `OD` 等
- **【操作平台】**: `XP SP2`
- **【软件介绍】**: 音速启动是 `WAKU` 非常喜欢的软件,可以方便地管理应用程序、文件夹、网址.此处省略 1G 的赞美之词,最重要的是她是一个国产的,免费的软件.
- **【作者声明】**: 本文主要用于技术研究,`WAKU` 并不想因为本文的存在而使音速启动的使用受到影响,请大家尊重他人的隐私.

---

### 详细过程

音速启动的安全性主要体现在下面三个方面:

1. 登录验证
2. 文件夹保护
3. 栏目加密

下面将会分别介绍它们的保护措施及对应的破解方法.

#### 1. 登录验证

- **防**: 登录验证也就是音速启动启动的时候要输入的密码,如密码不正确将不能使用音速启动
- **破**: 用 `FileMon` 可以监视到音速启动启动的时候打开了 `Files\Config.ini` 文件,在`[Config]`组里有 `PassWord` 一项,后面是明文存储的登录密码经过 `MD5` 后的值,如删除掉该值,就移除了启动密码(这个...有点简单哈~)

#### 2. 文件夹保护

- **防**: 文件夹保护是一个比较实用的功能,可以阻止他人浏览你想保密的文件夹.启用文件夹保护后,就会运行一个 `vsEnFolder.exe` 的隐藏进程,用普通双击是不能打开受保护的文件夹的,只能点右键>访问此文件夹,然后输入正确的密码才能进入.
- **破**: 文件夹保护相关的文件是 `User50\enFolder.vsf`,打开看一下,会发现类似 `qEJMtDsFtEwDEMEICMDMtCvBrECJnCJMrCwDxCwCINtEsEELmFCMsCyEtBtCnBm` 等字符,很明显,这些是经过加密的.看一下 `User50` 文件夹下的其他文件,比如在 `WAKU` 的 `User50` 文件夹下就有`"我的程序.vst"`,`"我的文件夹.vst"`等,都是以这种方式加密的.而这些加密的内容肯定会有逆运算还原成明文给用户查看,看来有必要研究一下它的还原算法了~

点击音速启动的菜单>文件夹保护,会弹出一个 `InputBox` 要求输入口令(默认密码为 `12345`),输入错了会弹出口令错误的消息框.

用 `PEID` 探一下音速启动的主程序 `VStart.exe`, `UPX` 的壳,轻松脱掉.再查,`VB6` 写的,讨厌...

用 `OD` 载入脱壳后的文件 `unpacked.exe`,在命令行输入 `bp rtcMsgBox` 下断,再输入口令,结果...没断下来,现在还不知道为啥,请高人指教.

这招不行咱换一招,用 `VBExplorer.exe` 加载 `unpacked.exe`,在工程窗口找到 `frmInputBox` 窗体,在属性窗口找到 `frmInputBox` 窗体的事件,其中 `cmdOK` 有一个 `Click` 事件,记下该事件的地址为 `0x004EE520`,在 `OD` 用 `CTRL+G` 来到该地址,下断.运行程序,再随便输入口令,果然断下来了,我 `F8` 啊 `F8`,`F8` 了半天发现自己进到了 `MSVBVM60.DLL` 里,可能这个确定按钮离验证过程实在是太远了...咱再换一招.

字符串搜索~要注意 `VB` 用的是 `UNICODE`,所以搜索时也要 `FIND UNICODE`,搜索完后翻翻,发现这个字符串 `user50\enFolder.vsf`,这不就是文件夹保护的那个文件么,啥也别说了终于找到组织了,双击,`F2` 下断,附近的代码节选如下:

```
0055DF5A    68 74D04300     push    0043D074                                ; user50\enFolder.vsf
0055DF5F    FF15 98104000   call    [<&MSVBVM60.__vbaStrCat>]               ; MSVBVM60.__vbaStrCat
0055DF65    8BD0            mov     edx, eax
0055DF67    8D4D D0         lea     ecx, [ebp-30]
0055DF6A    FF15 BC134000   call    [<&MSVBVM60.__vbaStrMove>]              ; MSVBVM60.__vbaStrMove
0055DF70    C745 FC 0900000>mov     dword ptr [ebp-4], 9
0055DF77    8B55 D0         mov     edx, [ebp-30]
0055DF7A    52              push    edx
0055DF7B    6A 01           push    1
0055DF7D    6A FF           push    -1
0055DF7F    6A 01           push    1
0055DF81    FF15 F8124000   call    [<&MSVBVM60.__vbaFileOpen>]             ; 读取 enFolder.vsf
0055DF87    C745 FC 0A00000>mov     dword ptr [ebp-4], 0A
0055DF8E    6A 01           push    1
0055DF90    8D45 D4         lea     eax, [ebp-2C]
0055DF93    50              push    eax
0055DF94    FF15 38104000   call    [<&MSVBVM60.__vbaLineInputStr>]         ; 从 enFolder.vsf 中读取一行内容
0055DF9A    C745 FC 0B00000>mov     dword ptr [ebp-4], 0B
0055DFA1    6A 01           push    1
0055DFA3    FF15 A4114000   call    [<&MSVBVM60.__vbaFileClose>]            ; MSVBVM60.__vbaFileClose
0055DFA9    C745 FC 0C00000>mov     dword ptr [ebp-4], 0C
0055DFB0    8B4D D4         mov     ecx, [ebp-2C]                           ; 读取的内容 > ECX
0055DFB3    51              push    ecx
0055DFB4    E8 77C4FDFF     call    <转换成明文>                            ; (Initial CPU selection)
```

从 `0055DF94` 处的 `CALL` 返回的 `EAX` 中就可以看到读取的内容正是 `enFolder.vsf` 里的密文.经过 `0055DF84` 的 `CALL`,该密文就被还原成明文,看来这个 `CALL`,就是还原明文的 `CALL` 了,`F7` 跟进.

先等一下,我先说明几点,众所周知,`VB` 编译的程序有很多垃圾代码,就像这个还原明文的 `CALL` 里,有很多没有用的代码,在分析的时候浪费了我大量精力,本着我不入地狱谁入地狱的精神,为了避免再浪费大家的时间,我只把有用的代码加上了注释,其他的就忽略吧.

```
0053A430 >  55              push    ebp
0053A431    8BEC            mov     ebp, esp
0053A433    83EC 14         sub     esp, 14
0053A436    68 06CB4000     push    <jmp.&MSVBVM60.__vbaExcept>
0053A43B    64:A1 00000000  mov     eax, fs:[0]
0053A441    50              push    eax
0053A442    64:8925 0000000>mov     fs:[0], esp
0053A449    81EC A8000000   sub     esp, 0A8
0053A44F    53              push    ebx
0053A450    56              push    esi
0053A451    57              push    edi
0053A452    8965 EC         mov     [ebp-14], esp
0053A455    C745 F0 806B400>mov     dword ptr [ebp-10], 00406B>
0053A45C    33F6            xor     esi, esi
0053A45E    8975 F4         mov     [ebp-C], esi
0053A461    8975 F8         mov     [ebp-8], esi
0053A464    8975 CC         mov     [ebp-34], esi
0053A467    8975 C8         mov     [ebp-38], esi
0053A46A    8975 C4         mov     [ebp-3C], esi
0053A46D    8975 C0         mov     [ebp-40], esi
0053A470    8975 BC         mov     [ebp-44], esi
0053A473    8975 AC         mov     [ebp-54], esi
0053A476    8975 9C         mov     [ebp-64], esi
0053A479    8975 8C         mov     [ebp-74], esi
0053A47C    8B55 08         mov     edx, [ebp+8]
0053A47F    8D4D C8         lea     ecx, [ebp-38]
0053A482    FF15 2C134000   call    [<&MSVBVM60.__vbaStrCopy>] ; 暂存密文
0053A488    6A 01           push    1
0053A48A    FF15 10114000   call    [<&MSVBVM60.__vbaOnError>] ; MSVBVM60.__vbaOnError
0053A490    8975 D0         mov     [ebp-30], esi
0053A493    8B45 C8         mov     eax, [ebp-38]
0053A496    50              push    eax
0053A497    FF15 40104000   call    [<&MSVBVM60.__vbaLenBstr>] ; 求得密文长度(UNICODE*2)
0053A49D    8985 6CFFFFFF   mov     [ebp-94], eax                 ; 长度 > [EBP-94]
0053A4A3    BB 01000000     mov     ebx, 1
0053A4A8    8B3D 54124000   mov     edi, [<&MSVBVM60.__vbaUI1I>; MSVBVM60.__vbaUI1I4
0053A4AE    3B9D 6CFFFFFF   cmp     ebx, [ebp-94]
0053A4B4    0F8F C2020000   jg      0053A77C                      ; 还有字符吗?
0053A4BA    C745 B4 0100000>mov     dword ptr [ebp-4C], 1
0053A4C1    C745 AC 0200000>mov     dword ptr [ebp-54], 2
0053A4C8    8D4D C8         lea     ecx, [ebp-38]
0053A4CB    894D 94         mov     [ebp-6C], ecx
0053A4CE    C745 8C 0840000>mov     dword ptr [ebp-74], 4008
0053A4D5    8D55 AC         lea     edx, [ebp-54]
0053A4D8    52              push    edx
0053A4D9    53              push    ebx
0053A4DA    8D45 8C         lea     eax, [ebp-74]
0053A4DD    50              push    eax
0053A4DE    8D4D 9C         lea     ecx, [ebp-64]
0053A4E1    51              push    ecx
0053A4E2    FF15 8C114000   call    [<&MSVBVM60.rtcMidCharVar>>; MSVBVM60.rtcMidCharVar
0053A4E8    8D55 9C         lea     edx, [ebp-64]
0053A4EB    52              push    edx
0053A4EC    FF15 44104000   call    [<&MSVBVM60.__vbaStrVarMov>; MSVBVM60.__vbaStrVarMove
0053A4F2    8BD0            mov     edx, eax
0053A4F4    8D4D CC         lea     ecx, [ebp-34]
0053A4F7    8B35 BC134000   mov     esi, [<&MSVBVM60.__vbaStrM>; MSVBVM60.__vbaStrMove
0053A4FD    FFD6            call    esi
0053A4FF    8D45 9C         lea     eax, [ebp-64]
0053A502    50              push    eax
0053A503    8D4D AC         lea     ecx, [ebp-54]
0053A506    51              push    ecx
0053A507    6A 02           push    2
0053A509    FF15 50104000   call    [<&MSVBVM60.__vbaFreeVarLi>; MSVBVM60.__vbaFreeVarList
0053A50F    83C4 0C         add     esp, 0C
0053A512    C745 B4 0100000>mov     dword ptr [ebp-4C], 1
0053A519    C745 AC 0200000>mov     dword ptr [ebp-54], 2
0053A520    8D55 C8         lea     edx, [ebp-38]
0053A523    8955 94         mov     [ebp-6C], edx
0053A526    C745 8C 0840000>mov     dword ptr [ebp-74], 4008
0053A52D    8D45 AC         lea     eax, [ebp-54]
0053A530    50              push    eax
0053A531    8BCB            mov     ecx, ebx
0053A533    83C1 01         add     ecx, 1
```



```
0053A536    0F80 06030000    jo      0053A842  

0053A53C    51               push    ecx  

0053A53D    8D55 8C          lea     edx, [ebp-74]  

0053A540    52               push    edx  

0053A541    8D45 9C          lea     eax, [ebp-64]  

0053A544    50               push    eax  

0053A545    FF15 8C114000    call    [<&MSVBVM60.rtcMidCharVar>>; 2 个字符一组依次取密文中的字符,取得的字符记为 c1 和 c2  

0053A54B    8D4D 9C          lea     ecx, [ebp-64]  

0053A54E    51               push    ecx  

0053A54F    FF15 44104000    call    [<&MSVBVM60.__vbaStrVarMov>; MSVBVM60.__vbaStrVarMove  

0053A555    8BD0             mov     edx, eax  

0053A557    8D4D C0          lea     ecx, [ebp-40]  

0053A55A    FFD6             call    esi  

0053A55C    8D55 9C          lea     edx, [ebp-64]  

0053A55F    52               push    edx  

0053A560    8D45 AC          lea     eax, [ebp-54]  

0053A563    50               push    eax  

0053A564    6A 02            push    2  

0053A566    FF15 50104000    call    [<&MSVBVM60.__vbaFreeVarLi>; MSVBVM60.__vbaFreeVarList  

0053A56C    83C4 0C          add     esp, 0C  

0053A56F    6A 01            push    1  

0053A571    68 080E4300      push    00430E08                 ; ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789abcdefghijklmnopqrstuvwxyz  

0053A576    8B4D CC          mov     ecx, [ebp-34]  

0053A579    51               push    ecx  

0053A57A    6A 00            push    0  

0053A57C    8B35 14134000    mov     esi, [<&MSVBVM60.__vbaInSt>; MSVBVM60.__vbaInStr  

0053A582    FFD6             call    esi                      ; 返回 c1 在上面的字符串中的位置  

0053A584    8BC8             mov     ecx, eax  

0053A586    83E9 01          sub     ecx, 1  

0053A589    0F80 B3020000    jo      0053A842  

0053A58F    FFD7             call    edi                      ; 位置-1,记为 a  

0053A591    8845 D8          mov     [ebp-28], al             ; a 放到[ebp-28]中  

0053A594    6A 01            push    1  

0053A596    68 080E4300      push    00430E08                 ; ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789abcdefghijklmnopqrstuvwxyz  

0053A59B    8B55 C0          mov     edx, [ebp-40]  

0053A59E    52               push    edx  

0053A59F    6A 00            push    0  

0053A5A1    FFD6             call    esi                      ; 返回 c2 在上面的字符串中的位置  

0053A5A3    8BC8             mov     ecx, eax  

0053A5A5    83E9 01          sub     ecx, 1  

0053A5A8    0F80 94020000    jo      0053A842  

0053A5AE    FFD7             call    edi                      ; 位置-1,记为 b  

0053A5B0    8845 D4          mov     [ebp-2C], al             ; b 放到[ebp-2c]中  

0053A5B3    8B75 D4          mov     esi, [ebp-2C]            ; b 再放到 ESI 中  

0053A5B6    81E6 FF000000    and     esi, 0FF                 ; 取低字节,其实还是 b  

0053A5BC    89B5 58FFFFFF    mov     [ebp-A8], esi  

0053A5C2    68 00000840      push    40080000  

0053A5C7    6A 00            push    0  

0053A5C9    68 00000040      push    40000000  

0053A5CE    6A 00            push    0  

0053A5D0    FF15 44134000    call    [<&MSVBVM60.__vbaPowerR8>] ; MSVBVM60.__vbaPowerR8  

0053A5D6    FF15 9C134000    call    [<&MSVBVM60.__vbaFpI4>]    ; MSVBVM60.__vbaFpI4  

0053A5DC    8BC8             mov     ecx, eax                 ; EAX 总是等于 8  

0053A5DE    8BC6             mov     eax, esi                 ; esi 中还是 b > EAX  

0053A5E0    99               cdq  

0053A5E1    F7F9             idiv    ecx                      ; b / 8  

0053A5E3    8BC8             mov     ecx, eax  

0053A5E5    FFD7             call    edi  

0053A5E7    8845 DC          mov     [ebp-24], al             ; 商 > [EBP-24] 记为 d1  

0053A5EA    DB85 58FFFFFF    fild    dword ptr [ebp-A8]  

0053A5F0    DD9D 50FFFFFF    fstp    qword ptr [ebp-B0]  

0053A5F6    68 00000840      push    40080000  

0053A5FB    6A 00            push    0  

0053A5FD    68 00000040      push    40000000  

0053A602    6A 00            push    0  

0053A604    FF15 44134000    call    [<&MSVBVM60.__vbaPowerR8>] ; MSVBVM60.__vbaPowerR8  

0053A60A    8B55 DC          mov     edx, [ebp-24]  

0053A60D    81E2 FF000000    and     edx, 0FF  

0053A613    8995 4CFFFFFF    mov     [ebp-B4], edx            ; 余数 > [EBP-B4]  

0053A619    DB85 4CFFFFFF    fild    dword ptr [ebp-B4]  

0053A61F    DD9D 44FFFFFF    fstp    qword ptr [ebp-BC]  

0053A625    DC8D 44FFFFFF    fmul    qword ptr [ebp-BC]  

0053A62B    DCAD 50FFFFFF    fsubr   qword ptr [ebp-B0]  

0053A631    DFE0             fstsw   ax  

0053A633    A8 0D            test    al, 0D                   ; 经过一顿浮点运算,AL 中又是余数了....  

0053A635    0F85 02020000    jnz     0053A83D  

0053A63B    FF15 F0114000    call    [<&MSVBVM60.__vbaFpUI1>]   ; MSVBVM60.__vbaFpUI1  

0053A641    8845 D4          mov     [ebp-2C], al             ; 余数 > [EBP-2C] 记为 d2  

0053A644    6A 00            push    0  

0053A646    8B75 D0          mov     esi, [ebp-30]  

0053A649    56               push    esi  

0053A64A    6A 01            push    1  

0053A64C    6A 11            push    11  

0053A64E    8D45 BC          lea     eax, [ebp-44]  

0053A651    50               push    eax  

0053A652    6A 01            push    1  

0053A654    68 80000000      push    80  

0053A659    FF15 00124000    call    [<&MSVBVM60.__vbaRedimPres>; MSVBVM60.__vbaRedimPreserve  

0053A65F    83C4 1C          add     esp, 1C  

0053A662    8B45 BC          mov     eax, [ebp-44]  

0053A665    85C0             test    eax, eax  

0053A667    74 2A            je      short 0053A693  

0053A669    66:8338 01       cmp     word ptr [eax], 1  

0053A66D    75 24            jnz     short 0053A693  

0053A66F    8BCE             mov     ecx, esi  

0053A671    2B48 14          sub     ecx, [eax+14]  

0053A674    898D 78FFFFFF    mov     [ebp-88], ecx  

0053A67A    3B48 10          cmp     ecx, [eax+10]  

0053A67D    72 0C            jb      short 0053A68B  

0053A67F    FF15 B0114000    call    [<&MSVBVM60.__vbaGenerateB>; MSVBVM60.__vbaGenerateBoundsError  

0053A685    8B8D 78FFFFFF    mov     ecx, [ebp-88]  

0053A68B    898D 40FFFFFF    mov     [ebp-C0], ecx  

0053A691    EB 0C            jmp     short 0053A69F  

0053A693    FF15 B0114000    call    [<&MSVBVM60.__vbaGenerateB>; MSVBVM60.__vbaGenerateBoundsError  

0053A699    8985 40FFFFFF    mov     [ebp-C0], eax  

0053A69F    66:0FB64D DC     movzx   cx, byte ptr [ebp-24]     ; d1 > CX  

0053A6A4    66:6BC9 3E       imul    cx, cx, 3E                ; CX *= 3Eh  

0053A6A8    0F80 94010000    jo      0053A842  

0053A6AE    66:0FB655 D8    movzx   dx, byte ptr [ebp-28]     ; a > DX  

0053A6B3    66:03CA          add     cx, dx                    ; CX += a,记结果为 e  

0053A6B6    0F80 86010000    jo      0053A842  

0053A6BC    FF15 34124000    call    [<&MSVBVM60.__vbaUI1I2>]   ; MSVBVM60.__vbaUI1I2  

0053A6C2    8B4D BC          mov     ecx, [ebp-44]  

0053A6C5    8B51 0C          mov     edx, [ecx+C]  

0053A6C8    8B8D 40FFFFFF    mov     ecx, [ebp-C0]  

0053A6CE    88040A           mov     [edx+ecx], al  

0053A6D1    8B4D BC          mov     ecx, [ebp-44]  

0053A6D4    85C9             test    ecx, ecx  

0053A6D6    74 2D            je      short 0053A705  

0053A6D8    66:8339 01       cmp     word ptr [ecx], 1  

0053A6DC    75 27            jnz     short 0053A705  

0053A6DE    8BC6             mov     eax, esi  

0053A6E0    2B41 14          sub     eax, [ecx+14]  

0053A6E3    8985 74FFFFFF    mov     [ebp-8C], eax  

0053A6E9    3B41 10          cmp     eax, [ecx+10]  

0053A6EC    72 0F            jb      short 0053A6FD  

0053A6EE    FF15 B0114000    call    [<&MSVBVM60.__vbaGenerateB>; MSVBVM60.__vbaGenerateBoundsError  

0053A6F4    8B4D BC          mov     ecx, [ebp-44]  

0053A6F7    8B85 74FFFFFF    mov     eax, [ebp-8C]  

0053A6FD    8985 3CFFFFFF    mov     [ebp-C4], eax  

0053A703    EB 0F            jmp     short 0053A714
```



```
0053A705    FF15 B0114000   call    [<&MSVBVM60.__vbaGenerateB>; MSVBVM60.__vbaGenerateBoundsError

0053A70B    8985 3CFFFFFF   mov     [ebp-C4], eax

0053A711    8B4D BC         mov     ecx, [ebp-44]

0053A714    85C9            test    ecx, ecx

0053A716    74 27           je      short 0053A73F

0053A718    66:8339 01      cmp     word ptr [ecx], 1

0053A71C    75 21           jnz     short 0053A73F

0053A71E    8BC6            mov     eax, esi

0053A720    2B41 14         sub     eax, [ecx+14]

0053A723    8985 78FFFFFF   mov     [ebp-88], eax

0053A729    3B41 10         cmp     eax, [ecx+10]

0053A72C    72 1A           jb      short 0053A748

0053A72E    FF15 B0114000   call    [<&MSVBVM60.__vbaGenerateB>; MSVBVM60.__vbaGenerateBoundsError

0053A734    8B4D BC         mov     ecx, [ebp-44]

0053A737    8B85 78FFFFFF   mov     eax, [ebp-88]

0053A73D    EB 09           jmp     short 0053A748

0053A73F    FF15 B0114000   call    [<&MSVBVM60.__vbaGenerateB>; MSVBVM60.__vbaGenerateBoundsError

0053A745    8B4D BC         mov     ecx, [ebp-44]

0053A748    8B49 0C         mov     ecx, [ecx+C]

0053A74B    8B95 3CFFFFFF   mov     edx, [ebp-C4]

0053A751    8A1411          mov     dl, [ecx+edx]                ; [ecx+edx]中就是 e

0053A754    3255 D4         xor     dl, [ebp-2C]                  ; e ^ d2

0053A757    881401          mov     [ecx+eax], dl                 ; DL 中得到的结果就是还原后的字符了

0053A75A    83C6 01         add     esi, 1

0053A75D    0F80 DF000000   jo      0053A842

0053A763    8975 D0         mov     [ebp-30], esi

0053A766    B8 02000000     mov     eax, 2

0053A76B    03C3            add     eax, ebx

0053A76D    0F80 CF000000   jo      0053A842

0053A773    8BD8            mov     ebx, eax

0053A775    33F6            xor     esi, esi

0053A777   ^ E9 32FDFFFF    jmp     0053A4AE                      ; 继续取下两个字符

0053A77C    8D45 BC         lea     eax, [ebp-44]                 ; ECX 中就是还原的明文

0053A77F    8945 94         mov     [ebp-6C], eax

0053A782    C745 8C 1160000>mov     dword ptr [ebp-74], 6011

0053A789    56              push    esi

0053A78A    6A 40           push    40

0053A78C    8D4D 8C         lea     ecx, [ebp-74]

0053A78F    51              push    ecx

0053A790    8D55 AC         lea     edx, [ebp-54]

0053A793    52              push    edx

0053A794    FF15 C0124000   call    [<&MSVBVM60.rtcStrConvVar2>; MSVBVM60.rtcStrConvVar2

0053A79A    8D45 AC         lea     eax, [ebp-54]

0053A79D    50              push    eax

0053A79E    FF15 44104000   call    [<&MSVBVM60.__vbaStrVarMov>; MSVBVM60.__vbaStrVarMove

0053A7A4    8BD0            mov     edx, eax

0053A7A6    8D4D C4         lea     ecx, [ebp-3C]

0053A7A9    FF15 BC134000   call    [<&MSVBVM60.__vbaStrMove>] ; MSVBVM60.__vbaStrMove

0053A7AF    8D4D AC         lea     ecx, [ebp-54]

0053A7B2    FF15 30104000   call    [<&MSVBVM60.__vbaFreeVar>] ; MSVBVM60.__vbaFreeVar

0053A7B8    FF15 00114000   call    [<&MSVBVM60.__vbaExitProc>>; MSVBVM60.__vbaExitProc

0053A7BE    9B              wait

0053A7BF    68 27A85300     push    0053A827

0053A7C4    EB 3F           jmp     short 0053A805

0053A7C6    BA B00D4300     mov     edx, 00430DB0

0053A7CB    8D4D C4         lea     ecx, [ebp-3C]

0053A7CE    FF15 2C134000   call    [<&MSVBVM60.__vbaStrCopy>] ; MSVBVM60.__vbaStrCopy

0053A7D4    FF15 00114000   call    [<&MSVBVM60.__vbaExitProc>>; MSVBVM60.__vbaExitProc

0053A7DA    9B              wait

0053A7DB    68 27A85300     push    0053A827

0053A7E0    EB 23           jmp     short 0053A805

0053A7E2    F645 F4 04      test    byte ptr [ebp-C], 4

0053A7E6    74 09           je      short 0053A7F1

0053A7E8    8D4D C4         lea     ecx, [ebp-3C]

0053A7EB    FF15 1C144000   call    [<&MSVBVM60.__vbaFreeStr>] ; MSVBVM60.__vbaFreeStr

0053A7F1    8D4D 9C         lea     ecx, [ebp-64]

0053A7F4    51              push    ecx

0053A7F5    8D55 AC         lea     edx, [ebp-54]

0053A7F8    52              push    edx

0053A7F9    6A 02           push    2

0053A7FB    FF15 50104000   call    [<&MSVBVM60.__vbaFreeVarLi>; MSVBVM60.__vbaFreeVarList

0053A801    83C4 0C         add     esp, 0C

0053A804    C3              retn

0053A805    8D4D CC         lea     ecx, [ebp-34]

0053A808    8B35 1C144000   mov     esi, [<&MSVBVM60.__vbaFree>; MSVBVM60.__vbaFreeStr

0053A80E    FFD6            call    esi

0053A810    8D4D C8         lea     ecx, [ebp-38]

0053A813    FFD6            call    esi

0053A815    8D4D C0         lea     ecx, [ebp-40]

0053A818    FFD6            call    esi

0053A81A    8D45 BC         lea     eax, [ebp-44]

0053A81D    50              push    eax

0053A81E    6A 00           push    0

0053A820    FF15 E0104000   call    [<&MSVBVM60.__vbaAryDestru>; MSVBVM60.__vbaAryDestruct

0053A826    C3              retn
```

上面的还原算法如果用 C++来写,只需要大约 10 行代码:

```
char* str = "GJxB0MJIbKhMcJrI0MIKLLQLaLzL6KHJKKKN8JxJQLaLiLlLLN8JsJiM9KjL8NoKjD9MwIfK";  //密文

int i;

int a, b, d1, d2;

for (i = 0; i < strlen(str); i += 2)
{
  a = GetIndex(str[i]);      //获得 str[i]在 ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789abcdefghijklmnopqrstuvwxyz 中的位置
  b = GetIndex(str[i+1]);

  d1 = b / 8;

  d2 = b % 8;

  cout<<char(d1*0x3e + a ^ d2);  //输出还原后的字符
}
```

经还原后的明文看,前 32 位存放的是密码经过 MD5 加密后的值,之后放的是加密的文件夹路径,之间用`|`隔开.

知道了存放格式,破解就好办了,直接把`enFolder.vsf`的前 64 个字符改为`zFsEqDFIJMIMrFGJDMHMxDFKpEnBrDJMsCJMmDtDDIrEoExDvAqFGKyEsCJKrCDL`,这个密文还原后是`827CCB0EEA8A706C4C34A16891F84E7B`,也就是`12345`经过 MD5 加密后的值.所以这么改后,密码就变成了`12345`,不错吧?

### 3.栏目加密

**防:** 音速启动可以针对每个栏目分别加密,这样切换到加密后的栏目不输入密码是看不到里面有什么程序的.

**破:** 为了分析方便,WAKU 在自己的音速启动"我的程序"类别下新加了一个栏目,名字叫"`test`",打开`user50\我的程序.vst`,最后一行是`+uKeLoLoM`,不算前面的`+`号,`KeLoLoM`经过还原算法正好是`test`.可见`+`号是栏目名称的标记.(注意`test`的密文却可以有多个,比如`tJfKqJvL`还原后也是`test`,密文是随机生成的)

随便往里扔一个东西,比如我把桌面上的`1.exe`扔进`test`栏目,发现`我的程序.vst`最后的内容变成如下模样:

```
+uKeLoLoM

oD,DKxB6KDNmJaJuLmLeLkKpNqJ7BdMkKfLbFTKeLoMsIjKlLdKqJ7LYJDIKLWJ7L10I0M3g07LoDgE9MxJeL,ejbn1nhjlkfmsjgncjllukri6aalmidj8cuj9mpnpnlmjndkol0myjelkltm1ny3k2izh16kmbge9mxj9m,,,qE,nBiA,
```

其中逗号起分隔作用,那么我们把它还原看看,原来是这个样子:

```
+test

1,C:\Documents and Settings\WAKU\桌面\1.exe,C:\Documents and Settings\WAKU\桌面\1.exe,,,0,0,
```

大体上是名称,路径之类的,不必过于研究.

如果我们给栏目加密,密码还用`12345`,之后密文变成这样:

```
+sI8NqJtJ,xDtFtAEJDKFJqEGJILHMvBCJoFpDoEJMmEJMmDsCELmBqCwCwDrEEMxDqAJKsBJN

xDoAqDILoFFJlDjMJL,EJAM5JHJjM9KpMnKeLmIvLvM9DcNjNeKaEUJcJsItJiLiMeJrI5JWLELMJWJ0M2ZI0H0jZ0MnAkAdIxJeL,CLAM6KJLkLeNtIlI8NkKoMoL7BcNjNeKaEYNcJtJoMgJlLcLqJ5JWLDILKXI7LZ2I0N2iY1NpChFfKxJfK,,,oC,oClD,
```

再翻译一下,是这样:

```
+test,827CCB0EEA8A706C4C34A16891F84E7B

827E7B,oD,C:\Documents and Settings\WAKU\桌面\1.exe,C:\Documents and Settings\WAKU\桌面\1.exe,,,0,0,
```

`827CCB0EEA8A706C4C34A16891F84E7B`还是`MD5(12345)`的值.

可见,在栏目名后面加一个逗号,然后放的是`MD5(密码)`.其他都没什么变化,就是名称变成了`827E7B,oD`,这是什么呢?`oD`和没加密前的名称一样,是`1`的密文,只要再经过一次还原就知道原始程序名称了.那前面的`827E7B`呢?

仔细看 MD5 值的前 3 位和后 3 位,发现了吗?哈哈~~~~为了证实这不是一种巧合,我们把密码更改为`54321`,再来看看:

**密文:**

```
+vLfKqJtJ,nBmBFIEMFIJLrBEMtBFJvBtAqDnBEMJKCKJJDLsCnBDKDLuBnBsApEmBnEzFpBsE

rFnACLvBpBpBkCnKIM,EJAM0MCMnIbIvKhM9MjNtJrI9DcNkKfL6AYN9MuKvLiLlLcLpK5JXKGNMJWJ6K01J1H0e25JnAgEfKyK9M,DKuC0MDNlK8LvKlIeLmIsIoL7BaLnJeKaETKeLuKoMlMnJcLrI1NVMDILKYL4IZ2GYH0g01NqFjDdIxJcJ,,,mA,pDeE,
```

**明文:**

```
+test,01CFCD4F6B8770FEBFB40CB906715822

01C822,mB....
```

之后还用我 COPY 吗?哈哈~

再怎么破解我就不多说了,相信看懂了后是件很容易的事,收工.

WAKU

2006-10-13.

`4dd466d0e1e0cc123afbe504da2462ca`
