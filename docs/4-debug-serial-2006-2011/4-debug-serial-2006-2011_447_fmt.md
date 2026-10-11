
# Smart PC Suite 算法分析+dll 类型破解通用方法+源码

> 原文：[`www.pediy.com/kssd/pediy10/76063.html`](https://www.pediy.com/kssd/pediy10/76063.html)

【破文标题】Smart PC Suite 算法分析+dll 类型破解通用方法+源码

【破文作者】Playboysen

【作者邮箱】`playboysen@126.com`

【破解工具】`PEiD`, `OD`

【破解平台】`Windows XP`

【软件名称】Smart PC Suite

【软件大小】10 MB

【软件授权】共享版($59.95)

【软件语言】英文

【原版下载】`http://smartpctools.com/products/`

【保护方式】用户名、注册码

【软件简介】使您的电脑运行速度更快，保护您的安全和恢复丢失的数据的唯一软件套件您最需要的！修复您的计算机自己并使其加载和运行速度更快。清理注册表错误和照顾病毒和间谍软件，确保您的个人隐私，并恢复已删除的档案

【破解声明】一点心得，愿与大家分享 o(∩\_\∩)o 版权所有，转载注明作者！

【破解内容】

自从上次写完“MP3 WAV OGG WMA AC3 to CD Burner 算法分析+简单逆向汇编”至今已经快两个月了，临近毕业，每天为生计奔波，找份工作好难，前途未卜，也没心情搞分析，不过偶尔练习一下，不致手生而已，一点心得，与君分享。

今天我们来研究一个国外公司的几款商业是软件，入手点是 `Smart PC Suite`(我专门挑选的售价较贵的 o(∩\_\∩)o)，总的感觉，这个公司的程序员很傻很天真，看完此文章你就明白我的话了~~~

主程序查壳 `Armadillo 1.xx - 2.xx` -> `Silicon Realms Toolworks`，用 `Arma Find Protected` 检测

```
<------- 05-11-2008 06:56:49 ------->
C:\\Documents and Settings\\Administrator\\桌面\\{app}\\SmartPCSuite.exe
!- Protected Armadillo
Protection system (Basic)
!- <Protection Options>
Standard protection or Minimum protection
!- <Backup Key Options>
Fixed Backup Keys
!- <Compression Options>
Better/Slower Compression
!- <Other Options>
!- Version 4.48 14August2006
```

脱壳 `So easy`，不多说，脚本搞定（`Armadillo.V4.0-V5.X.eXe.Standard.Protection.By.fly[CUG].oSc`），不过要提醒大家，虽然是标准壳但是不要用脱壳机，因为此程序加了双层壳，脱壳机太傻，搞不定的

```
013FE001    60              pushad                      ; This is the OEP!  Found By: fly[CUG]
013FE002    E8 03000000     call    013FE00A            ；看到没？脚本停在这里，Aspack，第二层         
013FE007   - E9 EB045D45     jmp     469CE4F7
013FE00C    55              push    ebp
013FE00D    C3              retn
013FE00E    E8 01000000     call    013FE014
013FE013    EB 5D           jmp     short 013FE072
013FE015    BB EDFFFFFF     mov     ebx, -13
```

`ESP` 定律后 `dump`，修复搞定，掠过~~~

试运行注册有错误提示，`OD` 加载脱壳程序搜索相关字符到这里



```
004AF962    E8 6552F5FF     call    00404BCC
004AF967    83F8 03         cmp     eax, 3
004AF96A    7D 47          jge     short 004AF9B3
004AF96C    68 18FC4A00    push    004AFC18                  ; name must have no less than three symbols!
004AF971    8D45 F0        lea     eax, dword ptr [ebp-10]
004AF974    50             push    eax
004AF975    A1 2CDA4E00    mov     eax, dword ptr [4EDA2C]
004AF97A    8B00           mov     eax, dword ptr [eax]
004AF97C    B9 4CFC4A00    mov     ecx, 004AFC4C               ; message9
004AF981    BA 60FC4A00    mov     edx, 004AFC60               ; evaluation
004AF986    8B18           mov     ebx, dword ptr [eax]
004AF988    FF13           call    dword ptr [ebx]
004AF98A    8B55 F0        mov     edx, dword ptr [ebp-10]
004AF98D    A1 04DB4E00    mov     eax, dword ptr [4EDB04]
004AF992    8B00           mov     eax, dword ptr [eax]
004AF994    8B80 04030000  mov     eax, dword ptr [eax+304]
004AF99A    E8 3D2BFBFF    call    004624DC
004AF99F    A1 04DB4E00    mov     eax, dword ptr [4EDB04]
004AF9A4    8B00           mov     eax, dword ptr [eax]
004AF9A6    8B10           mov     edx, dword ptr [eax]
004AF9A8    FF92 EC000000  call    dword ptr [edx+EC]
004AF9AE    E9 F0010000    jmp     004AFBA3
004AF9B3    8D55 EC        lea     edx, dword ptr [ebp-14]
004AF9B6    8B83 20030000  mov     eax, dword ptr [ebx+320]
004AF9BC    E8 EB2AFBFF    call    004624AC
004AF9C1    8B45 EC        mov     eax, dword ptr [ebp-14]
004AF9C4    E8 0352F5FF    call    00404BCC
004AF9C9    85C0           test    eax, eax
004AF9CB    75 47          jnz     short 004AFA14
004AF9CD    68 74FC4A00    push    004AFC74                  ; fields for license key must not be less three symbols!
004AF9D2    8D45 E8        lea     eax, dword ptr [ebp-18]
004AF9D5    50             push    eax
004AF9D6    A1 2CDA4E00    mov     eax, dword ptr [4EDA2C]
004AF9DB    8B00           mov     eax, dword ptr [eax]
004AF9DD    B9 B4FC4A00    mov     ecx, 004AFCB4               ; message10
004AF9E2    BA 60FC4A00    mov     edx, 004AFC60               ; evaluation
004AF9E7    8B18           mov     ebx, dword ptr [eax]
004AF9E9    FF13           call    dword ptr [ebx]
004AF9EB    8B55 E8        mov     edx, dword ptr [ebp-18]
004AF9EE    A1 04DB4E00    mov     eax, dword ptr [4EDB04]
004AF9F3    8B00           mov     eax, dword ptr [eax]
004AF9F5    8B80 04030000  mov     eax, dword ptr [eax+304]
004AF9FB    E8 DC2AFBFF    call    004624DC
004AFA00    A1 04DB4E00    mov     eax, dword ptr [4EDB04]
004AFA05    8B00           mov     eax, dword ptr [eax]
004AFA07    8B10           mov     edx, dword ptr [eax]
004AFA09    FF92 EC000000  call    dword ptr [edx+EC]
004AFA0F    E9 8F010000    jmp     004AFBA3
004AFA14    8D55 E0        lea     edx, dword ptr [ebp-20]
004AFA17    8B83 1C030000  mov     eax, dword ptr [ebx+31C]
004AFA1D    E8 8A2AFBFF    call    004624AC
004AFA22    8B45 E0        mov     eax, dword ptr [ebp-20]
004AFA25    8D55 E4        lea     edx, dword ptr [ebp-1C]
004AFA28    E8 AF9BF5FF    call    004095DC
004AFA2D    8B55 E4        mov     edx, dword ptr [ebp-1C]
004AFA30    A1 00D74E00    mov     eax, dword ptr [4ED700]
004AFA35    E8 164FF5FF    call    00404950
004AFA3A    8D55 D8        lea     edx, dword ptr [ebp-28]
004AFA3D    8B83 20030000  mov     eax, dword ptr [ebx+320]
004AFA43    E8 642AFBFF    call    004624AC
004AFA48    8B45 D8        mov     eax, dword ptr [ebp-28]
004AFA4B    8D55 DC        lea     edx, dword ptr [ebp-24]
004AFA4E    E8 899BF5FF    call    004095DC
004AFA53    8B55 DC        mov     edx, dword ptr [ebp-24]
004AFA56    A1 4CD54E00    mov     eax, dword ptr [4ED54C]
004AFA5B    E8 F04EF5FF    call    00404950
004AFA60    A1 00D74E00    mov     eax, dword ptr [4ED700]
004AFA65    8B00           mov     eax, dword ptr [eax]
004AFA67    50             push    eax
004AFA68    B9 C8FC4A00    mov     ecx, 004AFCC8               ; licuser
004AFA6D    BA D8FC4A00    mov     edx, 004AFCD8               ; 

software\microsoft\windows\currentversion\settings\smart pc suite
004AFA72    B8 01000080    mov     eax, 80000001
004AFA77    E8 BC46FFFF    call    004A4138
004AFA7C    A1 4CD54E00    mov     eax, dword ptr [4ED54C]
004AFA81    8B00           mov     eax, dword ptr [eax]
004AFA83    50             push    eax
004AFA84    BA D8FC4A00    mov     edx, 004AFCD8               ; 

software\microsoft\windows\currentversion\settings\smart pc suite
004AFA89    B9 24FD4A00    mov     ecx, 004AFD24               ; lickey
004AFA8E    B8 01000080    mov     eax, 80000001
004AFA93    E8 A046FFFF    call    004A4138
004AFA98    E8 FB6BFFFF    call    004A6698                  ; 绝对是关键 call 啊!!!!!
004AFA9D    8B15 CCD94E00  mov     edx, dword ptr [4ED9CC]       
004AFAA3    8802           mov     byte ptr [edx], al
004AFAA5    A1 CCD94E00    mov     eax, dword ptr [4ED9CC]
004AFAAA    8038 00        cmp     byte ptr [eax], 0
004AFAAD    74 75          je      short 004AFB24             ; 关键跳!爆破点
004AFAAF    E8 54C3F5FF    call    0040BE08
004AFAB4    83C4 F8        add     esp, -8
004AFAB7    DD1C24         fstp    qword ptr [esp]
004AFABA    9B             wait
004AFABB    BA D8FC4A00    mov     edx, 004AFCD8               ; 

software\microsoft\windows\currentversion\settings\smart pc suite
004AFAC0    B9 34FD4A00    mov     ecx, 004AFD34               ; timefactor1
004AFAC5    B8 01000080    mov     eax, 80000001
004AFACA    E8 4D48FFFF    call    004A431C
004AFACF    E8 34C3F5FF    call    0040BE08
004AFAD4    83C4 F8        add     esp, -8
004AFAD7    DD1C24         fstp    qword ptr [esp]
004AFADA    9B             wait
004AFADB    BA D8FC4A00    mov     edx, 004AFCD8               ; 

software\microsoft\windows\currentversion\settings\smart pc suite
004AFAE0    B9 48FD4A00    mov     ecx, 004AFD48               ; timefactor2
004AFAE5    B8 01000080    mov     eax, 80000001
004AFAEA    E8 2D48FFFF    call    004A431C
004AFAEF    68 5CFD4A00    push    004AFD5C                  ; thank you for registering smart pc suite!
004AFAF4    8D45 D4        lea     eax, dword ptr [ebp-2C]
004AFAF7    50             push    eax
004AFAF8    A1 2CDA4E00    mov     eax, dword ptr [4EDA2C]
004AFAFD    8B00           mov     eax, dword ptr [eax]
004AFAFF    B9 90FD4A00    mov     ecx, 004AFD90               ; message11
004AFB04    BA 60FC4A00    mov     edx, 004AFC60               ; evaluation
004AFB09    8B30           mov     esi, dword ptr [eax]
004AFB0B    FF16           call    dword ptr [esi]
004AFB0D    8B55 D4        mov     edx, dword ptr [ebp-2C]
004AFB10    A1 04DB4E00    mov     eax, dword ptr [4EDB04]
004AFB15    8B00           mov     eax, dword ptr [eax]
004AFB17    8B80 04030000  mov     eax, dword ptr [eax+304]
004AFB1D    E8 BA29FBFF    call    004624DC
004AFB22    EB 69          jmp     short 004AFB8D
004AFB24    68 A4FD4A00    push    004AFDA4                  ; the registration info is incorrect!
004AFB29    8D45 CC        lea     eax, dword ptr [ebp-34]
004AFB2C    50             push    eax
```

毫无疑问，关键跳和关键 `call` 精确定位，进入 `004AFA98` 的 `call` 分析算法。

代码:



## 汇编代码分析

```asm
004A66A0    57                 push    edi
004A66A1    33C0               xor     eax, eax
004A66A3    8945 F0            mov     dword ptr [ebp-10], eax
004A66A6    8945 F4            mov     dword ptr [ebp-C], eax
004A66A9    33C0               xor     eax, eax
004A66AB    55                 push    ebp
004A66AC    68 14684A00        push    004A6814
004A66B1    64:FF30            push    dword ptr fs:[eax]
004A66B4    64:8920            mov     dword ptr fs:[eax], esp
004A66B7    33DB               xor     ebx, ebx
004A66B9    A1 E0F24E00        mov     eax, dword ptr [4EF2E0]
004A66BE    E8 09E5F5FF        call    00404BCC                         ; 用户名长度和 4 比较
004A66C3    83F8 04            cmp     eax, 4
004A66C6    0F8C 2D010000      jl      004A67F9
004A66CC    A1 E4F24E00        mov     eax, dword ptr [4EF2E4]
004A66D1    E8 F6E4F5FF        call    00404BCC                         ; 假码长度和 4 比较
004A66D6    83F8 04            cmp     eax, 4
004A66D9    0F8C 1A010000      jl      004A67F9
004A66DF    8D45 F4            lea     eax, dword ptr [ebp-C]
004A66E2    B9 2C684A00        mov     ecx, 004A682C                    ; ASCII "SmartPCSuite.dll"
004A66E7    8B15 54F24E00      mov     edx, dword ptr [4EF254]          ; 软件根目录地址放入
004A66ED    E8 26E5F5FF        call    00404C18
004A66F2    8B45 F4            mov     eax, dword ptr [ebp-C]           ; 得到 SmartPCSrute.dll 的绝对路径
004A66F5    E8 4E38F6FF        call    00409F48
004A66FA    84C0               test    al, al
004A66FC    0F84 F7000000      je      004A67F9
004A6702    B9 48684A00        mov     ecx, 004A6848                    ; ASCII "TimeFactor2"
004A6707    BA 5C684A00        mov     edx, 004A685C                    ; ASCII "SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Settings\\Smart PC Suite"
004A670C    B8 01000080        mov     eax, 80000001
004A6711    E8 B2DDFFFF        call    004A44C8                         ; 上面这里在检测软件是否已注册，爆破点
004A6716    84C0               test    al, al
004A6718    74 3E              je      short 004A6758
004A671A    BA 5C684A00        mov     edx, 004A685C                    ; ASCII "SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Settings\\Smart PC Suite"
004A671F    B9 48684A00        mov     ecx, 004A6848                    ; ASCII "TimeFactor2"
004A6724    B8 01000080        mov     eax, 80000001
004A6729    E8 42D9FFFF        call    004A4070
004A672E    DD5D F8            fstp    qword ptr [ebp-8]
004A6731    9B                 wait
004A6732    FF75 FC            push    dword ptr [ebp-4]
004A6735    FF75 F8            push    dword ptr [ebp-8]
004A6738    FF35 F0F24E00      push    dword ptr [4EF2F0]
004A673E    FF35 ECF24E00      push    dword ptr [4EF2EC]
004A6744    E8 F3CBFEFF        call    0049333C
004A6749    85C0               test    eax, eax
004A674B    0F8C A8000000      jl      004A67F9
004A6751    B3 01              mov     bl, 1
004A6753    E9 A1000000        jmp     004A67F9
004A6758    8D45 F0            lea     eax, dword ptr [ebp-10]
004A675B    B9 2C684A00        mov     ecx, 004A682C                    ; ASCII "SmartPCSuite.dll"
004A6760    8B15 54F24E00      mov     edx, dword ptr [4EF254]
004A6766    E8 ADE4F5FF        call    00404C18
004A676B    8B45 F0            mov     eax, dword ptr [ebp-10]          ; 得到 SmartPCSrute.dll 的绝对路径
004A676E    E8 59E6F5FF        call    00404DCC
004A6773    50                 push    eax
004A6774    E8 970CF6FF        call    <jmp.&kernel32.LoadLibraryA>      ; 加载 SmartPCSrute.dll
004A6779    8BF8               mov     edi, eax
004A677B    68 F4010000        push    1F4
004A6780    E8 178BF6FF        call    <jmp.&kernel32.Sleep>
004A6785    85FF               test    edi, edi
004A6787    74 70              je      short 004A67F9
004A6789    68 A0684A00        push    004A68A0                           ; ASCII "InstallKey"
004A678E    57                 push    edi
004A678F    E8 C40BF6FF        call    <jmp.&kernel32.GetProcAddress>    ; 得到函数地址
004A6794    89C6               mov     esi, eax
004A6796    68 F4010000        push    1F4
004A679B    E8 FC8AF6FF        call    <jmp.&kernel32.Sleep>
004A67A0    85F6               test    esi, esi
004A67A2    74 1A              je      short 004A67BE
004A67A4    A1 E4F24E00        mov     eax, dword ptr [4EF2E4]           ; 假码的存放地址
004A67A9    E8 1EE6F5FF        call    00404DCC
004A67AE    50                 push    eax
004A67AF    A1 E0F24E00        mov     eax, dword ptr [4EF2E0]           ; 用户名的存放地址
004A67B4    E8 13E6F5FF        call    00404DCC
004A67B9    50                 push    eax
004A67BA    FFD6               call    esi                                ; 调用 Installkey 函数
004A67BC    8BD8               mov     ebx, eax
004A67BE    68 AC684A00        push    004A68AC                           ; ASCII "CheckCode"
004A67C3    57                 push    edi
004A67C4    E8 8F0BF6FF        call    <jmp.&kernel32.GetProcAddress>    ; 得到函数地址
004A67C9    89C6               mov     esi, eax
004A67CB    68 F4010000        push    1F4
004A67D0    E8 C78AF6FF        call    <jmp.&kernel32.Sleep>
004A67D5    85F6               test    esi, esi
004A67D7    74 1A              je      short 004A67F3
004A67D9    A1 E4F24E00        mov     eax, dword ptr [4EF2E4]           ; 假码的存放地址
004A67DE    E8 E9E5F5FF        call    00404DCC
004A67E3    50                 push    eax
004A67E4    A1 E0F24E00        mov     eax, dword ptr [4EF2E0]           ; 用户名的存放地址
004A67E9    E8 DEE5F5FF        call    00404DCC
004A67EE    50                 push    eax
004A67EF    FFD6               call    esi                                ; 调用 CheckCode 函数
004A67F1    8BD8               mov     ebx, eax                           ; 如果错误则 eax 返回 0
004A67F3    57                 push    edi
004A67F4    E8 7F0AF6FF        call    <jmp.&kernel32.FreeLibrary>        ; 释放 SmartPCSuite.dll
004A67F9    33C0               xor     eax, eax
004A67FB    5A                 pop     edx
004A67FC    59                 pop     ecx
004A67FD    59                 pop     ecx
004A67FE    64:8910            mov     dword ptr fs:[eax], edx
004A6801    68 1B684A00        push    004A681B
004A6806    8D45 F0            lea     eax, dword ptr [ebp-10]
004A6809    BA 02000000        mov     edx, 2
004A680E    E8 0DE1F5FF        call    00404920
004A6813    C3                 retn
004A6814   ^ E9 27DAF5FF       jmp     00404240
004A6819   ^ EB EB             jmp     short 004A6806
004A681B    8BC3               mov     eax, ebx                           ;经典爆破点
004A681D    5F                 pop     edi
004A681E    5E                 pop     esi
004A681F    5B                 pop     ebx
004A6820    8BE5               mov     esp, ebp
004A6822    5D                 pop     ebp
004A6823    C3                 retn
```

## 分析总结

1. 程序启动时检测注册表相关键值或者调用 `SmartPCSuite.dll` 来判断注册与否。
2. 调用软件根目录的 `SmartPCSuite.dll`，使用其中的 `InstallKey` 和 `CheckCode` 函数进行注册验证。
3. 如果注册正确，`InstallKey` 和 `CheckCode` 返回 `1`。

## 解决方案

我们见招拆招，有几种方法：

1. **爆破**：相关爆破点已经明确标出，此法是小鸟所为，对其提高分析能力不利，不提倡，不多说。
2. **修改关键跳**：修改 `004AFAAD` 关键跳，生成注册信息后，将相关键值提取出来，需要时在别机导入即可。
3. **伪造 DLL**：程序需要 `SmartPCSuite.dll`，我们伪造一个 `SmartPCSuite.dll`。此方法用处多多，是破解 dll 文件注册验证类型的经典狠招，推荐！

### 伪造 DLL 实现

用 `Delphi 7` 构建一个 dll 文件，源码如下：

```pascal
library Project2;
uses
  SysUtils,
  Classes;

{$R \*.res}

function InstallKey(str1,str2:PChar):Boolean;Stdcall;
begin
  result:=True;
end;

function CheckCode(str1,str2:PChar):Boolean;Stdcall;
begin
  result:=True;
end;

exports
   InstallKey,CheckCode;

begin
end.
```

然后把生成的 dll 改名为 `SmartPCSuite.dll` 替换软件根目录的 `SmartPCSuite.dll`，发现程序正常运行，已注册！！

## 程序不足与改进建议

最后再说一下这个程序的几点不足，但愿其它程序员能引以为戒：

1. `Armadillo` 当压缩壳用，几乎没有发挥强壳的任何作用，还使得主程序体积暴增，不智！
2. 出现注册错误提示，而且相关字符并未加密，给别人以可乘之机，大大缩减了程序破解时间，大愚！
3. 注册名和注册码没什么用，修改关键跳后生成的两个键值的注册信息才是关键，而注册名可以随便改，汗~~
4. 没有重启验证，没有在关键函数处验证注册信息。
5. 没有自校验主程序和关键 dll。
6. 过于看重软件界面，功能一般，启动缓慢，有些华而不实~~
7. ……

再说下去，程序员就哭了，算了，只是给软件公司提点改进建议而已，没别的意思~~

## 附件与测试验证

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | 伪造的 `SmartPCSuite.rar` |
| :--- | :--- |

经测试我们伪造的 dll 文件几乎能搞定此公司所有的商业软件。拿 `MagicSpeed` 来说，软件根目录下有一个 `MagicSpeed.dll`，我们把伪造的 dll 改名为 `MagicSpeed.dll` 替代根目录中的文件，打开软件后提示注册，随便输入用户名和注册码都可以正确注册，而且以后再打开则不需重新注册即为正版，其它软件大同小异。

同理 `FixMyRegistry` 则替代 `FixMyRegistry.dll` 等等，脱壳什么的都不需要。

原理很简单，此公司所有软件的验证 dll 中的验证函数都是 `InstallKey` 和 `CheckCode`，而且注册码正确都是返回 `True`，所有我们就顺水推舟，直接搞定他们了，哎~~



本篇文章和前段时间写的两篇文章呼应，都是针对 `dll` 验证类型的分析破解，总结如下：

## 类型一
调用 `dll` 文件压入相关参数然后返回注册码的破解分析

`[`bbs.pediy.com/showthread.php?...ght=playboysen`](http://bbs.pediy.com/showthread.php?t=72809&highlight=playboysen)`

## 类型二
纯粹 `dll` 验证，根据注册信息的正确与否返回相关的值，破解方法如本篇文章

## 类型三
`dll` 插件型的破解，如千千静听插件等等

`[`bbs.pediy.com/showthread.php?...ght=playboysen`](http://bbs.pediy.com/showthread.php?t=66796&highlight=playboysen)`

当然如果软件校验主程序和 `dll` 文件的话，免不了要先解除校验的。

至此 `dll` 验证类型的破解分析告一段落，再碰到相关的软件再进行补充，希望能给大家提供一点思路，以开拓视野。
