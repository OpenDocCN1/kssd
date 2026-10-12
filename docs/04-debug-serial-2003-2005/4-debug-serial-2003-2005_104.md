
# DVDChapter V1.2

> 原文：[`www.pediy.com/kssd/pediy06/pediy6010.htm`](https://www.pediy.com/kssd/pediy06/pediy6010.htm)

下载页面：  [`www.skycn.com/soft/6680.html`](http://www.skycn.com/soft/6680.html)

软件大小：  632 KB

软件语言：  简体中文

软件类别：  国产软件 / 共享版 / 视频处理

应用平台：  `Win9x/NT/2000/XP`

加入时间：  2001-10-06 15:01:05

下载次数：  977

推荐等级：  *****

开 发 商：  [`www.boilsoft.com/`](http://www.boilsoft.com/)

【软件简介】：`DVDChapter` 是一个独立的 DVD 光盘工具，用来抽取 DVD 光盘上的章节，并把数据保存在硬盘上。本软件已广泛使用于视频点歌系统中，用于抽取 Karaoke 和 MTV 光盘上歌曲。支持 `Microsoft Windows NT / 2000`、`95/98/Me`。经过一年多的发展，`DVDChapter` 已成为一套成熟、稳定的工具软件。

【软件限制】：10 次试用。

【作者声明】：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

【破解工具】：`TRW2000` 娃娃修改版、`Ollydbg1.09`、`PEiD`、`W32Dasm` `9.0` 白金版

—————————————————————————————————

【过 程】：

呵呵，坛子上有朋友提出这个东东，我半个月前就看了看，找到了核心，却水平太浅而无法求逆。

幸得 `afanty` 老师的指教，终于完成了这篇笔记。感谢 `afanty` 老师！

如果有错误当是我分析的原因。

`DVDChapter.exe` 无壳。`Visual C++` `6.0` 编写。

注册码格式为 `X-Y-Z` 共 3 部分。简单假设 `X=1357` `Y=2468` `Z=9012`

试炼码：`1357-2468-9012`

—————————————————————————————————

```
:004040D0 6AFF                            push FFFFFFFF
:004040D2 68F0C14100                     push 0041C1F0
:004040D7 64A100000000                   mov eax, dword ptr fs:[00000000]
:004040DD 50                             push eax
:004040DE 64892500000000                 mov dword ptr fs:[00000000], esp
:004040E5 83EC08                         sub esp, 00000008
:004040E8 56                             push esi
:004040E9 8BF1                           mov esi, ecx
:004040EB 6A01                           push 00000001
:004040ED E81A1F0100                     call 0041600C
:004040F2 A1E4484200                     mov eax, dword ptr [004248E4]
:004040F7 89442408                       mov dword ptr [esp+08], eax
:004040FB C744241400000000               mov [esp+14], 00000000
:00404103 89442404                       mov dword ptr [esp+04], eax
* Possible Reference to String Resource ID=00106: "o ?
                                |
:00404107 6A6A                           push 0000006A
:00404109 8D4C240C                       lea ecx, dword ptr [esp+0C]
:0040410D C644241801                     mov [esp+18], 01
:00404112 E828450100                     call 0041863F
:00404117 8B465C                         mov eax, dword ptr [esi+5C]
                                ==== >EAX=1357-2468-9012     试炼码
:0040411A 50                             push eax
:0040411B E860FBFFFF                     call 00403C80
                                ==== >关键 CALL！进入！
:00404120 83C404                         add esp, 00000004
:00404123 85C0                           test eax, eax
:00404125 7436                           je 0040415D
                                ==== >跳则 OVER！
:00404127 8B465C                         mov eax, dword ptr [esi+5C]
:0040412A 50                             push eax
:0040412B E820FCFFFF                     call 00403D50
:00404130 83C404                         add esp, 00000004
:00404133 8D4C2404                       lea ecx, dword ptr [esp+04]
* Possible Reference to String Resource ID=00104: "??
?(腹 o?
                                |
:00404137 6A68                           push 00000068
:00404139 E801450100                     call 0041863F
:0040413E 8B442408                       mov eax, dword ptr [esp+08]
:00404142 8B4C2404                       mov ecx, dword ptr [esp+04]
:00404146 6A00                           push 00000000
:00404148 50                             push eax
:00404149 51                             push ecx
:0040414A 8BCE                           mov ecx, esi
:0040414C E85F170100                     call 004158B0
                                ==== >呵呵，胜利女神！
:00404151 8B16                           mov edx, dword ptr [esi]
:00404153 8BCE                           mov ecx, esi
:00404155 FF92C4000000                   call dword ptr [edx+000000C4]
:0040415B EB1E                           jmp 0040417B
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00404125(C)
|
* Possible Reference to String Resource ID=00105: "???
?腹 o 鯰?
                                |
:0040415D 6A69                           push 00000069
:0040415F 8D4C2408                       lea ecx, dword ptr [esp+08]
:00404163 E8D7440100                     call 0041863F
:00404168 8B442408                       mov eax, dword ptr [esp+08]
:0040416C 8B4C2404                       mov ecx, dword ptr [esp+04]
:00404170 6A10                           push 00000010
:00404172 50                             push eax
:00404173 51                             push ecx
:00404174 8BCE                           mov ecx, esi
:00404176 E835170100                     call 004158B0
                                ==== >BAD BOY！
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040415B(U)
|
:0040417B 8D4C2404                       lea ecx, dword ptr [esp+04]
:0040417F C644241400                     mov [esp+14], 00
:00404184 E852310100                     call 004172DB
:00404189 8D4C2408                       lea ecx, dword ptr [esp+08]
:0040418D C7442414FFFFFFFF               mov [esp+14], FFFFFFFF
:00404195 E841310100                     call 004172DB
:0040419A 8B4C240C                       mov ecx, dword ptr [esp+0C]
:0040419E 5E                             pop esi
:0040419F 64890D00000000                 mov dword ptr fs:[00000000], ecx
:004041A6 83C414                         add esp, 00000014
:004041A9 C3                             ret
```

—————————————————————————————————

进入关键 CALL：`40411B` `call` `00403C80`

* Referenced by a CALL at Addresses:

|:00401F4E , :0040411B |

```
:00403C80 53                             push ebx
:00403C81 55                             push ebp
:00403C82 56                             push esi
:00403C83 8B742410                       mov esi, dword ptr [esp+10]
                                ==== >ESI=1357-2468-9012     试炼码
:00403C87 33ED                           xor ebp, ebp
:00403C89 57                             push edi
:00403C8A 85F6                           test esi, esi
:00403C8C 0F84B0000000                   je 00403D42
:00403C92 803E00                         cmp byte ptr [esi], 00
:00403C95 0F84A7000000                   je 00403D42
:00403C9B 8BFE                           mov edi, esi
:00403C9D 83C9FF                         or ecx, FFFFFFFF
:00403CA0 33C0                           xor eax, eax
:00403CA2 F2                             repnz
:00403CA3 AE                             scasb
:00403CA4 F7D1                           not ecx
:00403CA6 51                             push ecx
                                ==== >取试炼码位数 ECX=F
:00403CA7 6A40                           push 00000040
* Reference To: KERNEL32.LocalAlloc, Ord:01C8h
                                |
:00403CA9 FF1550D24100                   Call dword ptr [0041D250]
:00403CAF 8BD8                           mov ebx, eax
:00403CB1 85DB                           test ebx, ebx
:00403CB3 0F8489000000                   je 00403D42
:00403CB9 8BFE                           mov edi, esi
:00403CBB 83C9FF                         or ecx, FFFFFFFF
:00403CBE 33C0                           xor eax, eax
:00403CC0 6A2D                           push 0000002D
:00403CC2 F2                             repnz
:00403CC3 AE                             scasb
:00403CC4 F7D1                           not ecx
:00403CC6 2BF9                           sub edi, ecx
:00403CC8 53                             push ebx
:00403CC9 8BC1                           mov eax, ecx
:00403CCB 8BF7                           mov esi, edi
:00403CCD 8BFB                           mov edi, ebx
:00403CCF C1E902                         shr ecx, 02
:00403CD2 F3                             repz
:00403CD3 A5                             movsd
:00403CD4 8BC8                           mov ecx, eax
:00403CD6 83E103                         and ecx, 00000003
:00403CD9 F3                             repz
:00403CDA A4                             movsb
:00403CDB 8BF3                           mov esi, ebx
                                ==== >ESI=1357-2468-9012     试炼码
:00403CDD E80E2E0000                     call 00406AF0
                                ==== >检测 CALL！看看注册码是否是 X-Y 格式
:00403CE2 83C408                         add esp, 00000008
```



## 核心 CALL：`403D23 call 00403C30`

* Referenced by a CALL at Address:
`|:00403D23`

```assembly
:00403C30 8B442404        mov eax, dword ptr [esp+04]
                          ====>EAX=1357

:00403C34 53              push ebx

:00403C35 85C0            test eax, eax

:00403C37 57              push edi

:00403C38 7440            je 00403C7A

:00403C3A 8B7C2410        mov edi, dword ptr [esp+10]
                          ====>EDI=9012

:00403C3E 85FF            test edi, edi

:00403C40 7438            je 00403C7A

:00403C42 8B5C2414        mov ebx, dword ptr [esp+14]
                          ====>EBX=2468

:00403C46 85DB            test ebx, ebx

:00403C48 7430            je 00403C7A

:00403C4A 56              push esi

:00403C4B 50              push eax

:00403C4C E83FFFFFFF      call 00403B90
                          一、
                          ====>对 1357 进行运算得出 C62B

:00403C51 57              push edi

:00403C52 8BF0            mov esi, eax
                          ====>ESI=EAX=C62B

:00403C54 E837FFFFFF     call 00403B90
                          二、
                          ====>对 9012 进行运算得出 66866

:00403C59 53              push ebx

:00403C5A 8BF8            mov edi, eax
                          ====>EDI=EAX=66866

:00403C5C E82FFFFFFF     call 00403B90
                          三、
                          ====>对 2468 进行运算得出 181A0

:00403C61 57              push edi

:00403C62 56              push esi

:00403C63 8BD8            mov ebx, eax
                          ====>EBX=EAX=181A0

:00403C65 E886FFFFFF     call 00403BF0
                          四、
                          ====>运算 CALL!进入!

:00403C6A 83C414         add esp, 00000014

:00403C6D 33C9           xor ecx, ecx

:00403C6F 3BC3           cmp eax, ebx
                          ====>比较了！如果这 2 者相等则 OK！爆破点②！
                          ====>EAX=498545E
                          ====>EBX=181A0

:00403C71 0F94C1        sete cl

:00403C74 5E              pop esi

:00403C75 5F              pop edi

:00403C76 8BC1            mov eax, ecx

:00403C78 5B              pop ebx

:00403C79 C3              ret
```

## 进入运算一、二、三：`call 00403B90`

* Referenced by a CALL at Addresses:
`|:00403C4C, :00403C54, :00403C5C`

```assembly
:00403B90 8B542404        mov edx, dword ptr [esp+04]

:00403B94 33C0            xor eax, eax

:00403B96 85D2            test edx, edx

:00403B98 744F            je 00403BE9

:00403B9A 8A0A            mov cl, byte ptr [edx]
                          一、
                          ====>CL=依次取 1357 字符的 HEX 值
                          —————————————————————————
                          二、
                          ====>CL=依次取 9012 字符的 HEX 值
                          —————————————————————————
                          三、
                          ====>CL=依次取 2468 字符的 HEX 值

:00403B9C 84C9            test cl, cl

:00403B9E 744C            je 00403BEC

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00403BE6(C)

:00403BA0 80F930          cmp cl, 30

:00403BA3 7C11            jl 00403BB6

:00403BA5 80F939          cmp cl, 39

:00403BA8 7F0C            jg 00403BB6

:00403BAA 0FBEC9          movsx ecx, cl

:00403BAD 8D04C0          lea eax, dword ptr [eax+8*eax]
                          一、
                          1、 ====>EAX=00   * 9=0
                          2、 ====>EAX=01   * 9=9
                          3、 ====>EAX=27   * 9=15F
                          4、 ====>EAX=581  * 9=3189
                          —————————————————————————
                          二、
                          1、 ====>EAX=00   * 9=0
                          2、 ====>EAX=09   * 9=51
                          3、 ====>EAX=144  * 9=B64
                          4、 ====>EAX=2D91 * 9=19A19
```



# 汇编分析与算法还原

## 一、 分支逻辑分析

### 分支 1：字符 '9'
```
:00403BB0 8D4481D0          lea eax, dword ptr [ecx+4*eax-30]
```
运算逻辑：
1. `EAX = 00`，`0 * 9 = 0`
2. `EAX = 02`，`2 * 9 = 12`
3. `EAX = 4C`，`4 * 9 = 2AC`
4. `EAX = AB6`，`AB * 9 = 6066`

### 分支 2：字符处理
```
:00403BB0 8D4481D0          lea eax, dword ptr [ecx+4*eax-30]
```
运算逻辑：
1. `EAX = 31 + 4*0 - 30 = 1`
2. `EAX = 33 + 4*9 - 30 = 27`
3. `EAX = 35 + 4*15F - 30 = 581`
4. `EAX = 37 + 4*3189 - 30 = C62B`

### 分支 3：字符处理
运算逻辑：
1. `EAX = 39 + 4*0 - 30 = 9`
2. `EAX = 30 + 4*51 - 30 = 144`
3. `EAX = 31 + 4*B64 - 30 = 2D91`
4. `EAX = 32 + 4*19A19 - 30 = 66866`

### 分支 4：字符处理
运算逻辑：
1. `EAX = 32 + 4*0 - 30 = 2`
2. `EAX = 34 + 4*12 - 30 = 4C`
3. `EAX = 36 + 4*2AC - 30 = AB6`
4. `EAX = 38 + 4*6066 - 30 = 181A0`

## 二、 核心循环代码

```asm
:00403BB4 EB2A              jmp 00403BE0
                            ; 如果是字母则进行后续运算

:00403BB6 80F941            cmp cl, 41
:00403BB9 7C11              jl 00403BCC
:00403BBB 80F95A            cmp cl, 5A
:00403BBE 7F0C              jg 00403BCC
:00403BC0 0FBEC9            movsx ecx, cl
:00403BC3 8D04C0            lea eax, dword ptr [eax+8*eax]
:00403BC6 8D4481C9          lea eax, dword ptr [ecx+4*eax-37]
:00403BCA EB14              jmp 00403BE0

:00403BCC 80F961            cmp cl, 61
:00403BCF 7C18              jl 00403BE9
:00403BD1 80F97A            cmp cl, 7A
:00403BD4 7F13              jg 00403BE9
:00403BD6 0FBEC9            movsx ecx, cl
:00403BD9 8D04C0            lea eax, dword ptr [eax+8*eax]
:00403BDC 8D4481A9          lea eax, dword ptr [ecx+4*eax-57]

:00403BE0 8A4A01            mov cl, byte ptr [edx+01]
:00403BE3 42                inc edx
:00403BE4 84C9              test cl, cl
:00403BE6 75B8              jne 00403BA0
:00403BE8 C3                ret
```

## 三、 算法解析

根据 `afanty` 的指教，该代码段将 `1357`、`2468`、`9012` 视为 **36进制数**，并转换为 **16进制数**：
- `X = 1357` (Base 36) = `C62B` (Hex)
- `Y = 2468` (Base 36) = `181A0` (Hex)
- `Z = 9012` (Base 36) = `66866` (Hex)

### CALL 四运算细节
入口地址：`00403C65` 调用 `00403BF0`

```asm
:00403BF0 8B4C2404          mov ecx, dword ptr [esp+04]
                            ; ECX = [esp+04] = C62B

:00403BF4 56                push esi
:00403BF5 8BC1              mov eax, ecx
                            ; EAX = ECX = C62B

:00403BF7 33D2              xor edx, edx
:00403BF9 BE40B60000        mov esi, 0000B640
                            ; ESI = B640

:00403BFE F7F6              div esi
                            ; EDX = C62B % B640 = FEB

:00403C00 8D0492            lea eax, dword ptr [edx+4*edx]
                            ; EAX = FEB * 5 = 4F97

:00403C03 8D04C2            lea eax, dword ptr [edx+8*eax]
                            ; EAX = FEB + 8*4F97 = 28CA3

:00403C06 8D04C2            lea eax, dword ptr [edx+8*eax]
                            ; EAX = FEB + 8*28CA3 = 147503

:00403C09 8D3440            lea esi, dword ptr [eax+2*eax]
                            ; ESI = 147503 * 3 = 3D5F09

:00403C0C B88303E659        mov eax, 59E60383
                            ; EAX = 59E60383

:00403C11 F7E1              mul ecx
                            ; 59E60383 * C62B = 45970059E901
                            ; EAX = 0059E901 (低32位)
                            ; EDX = 4597 (高32位，进位)

:00403C13 8B44240C          mov eax, dword ptr [esp+0C]
                            ; EAX = [esp+0C] = 66866

:00403C17 C1EA0E            shr edx, 0E
                            ; EDX = 4597 SHR 0E = 00000001

:00403C1A 8D0CC500000000    lea ecx, dword ptr [8*eax+00000000]
                            ; ECX = 8 * 66866 = 334330

:00403C21 03F2              add esi, edx
                            ; ESI = 3D5F09 + 1 = 3D5F0A

:00403C23 2BC8              sub ecx, eax
                            ; ECX = 334330 - 66866 = 002CDACA

:00403C25 8D0488            lea eax, dword ptr [eax+4*ecx]
                            ; EAX = 66866 + 4 * 2CDACA = 00B9D38E

:00403C28 8D1440            lea edx, dword ptr [eax+2*eax]
                            ; EDX = B9D38E * 3 = 22D7AAA

:00403C2B 8D0456            lea eax, dword ptr [esi+2*edx]
                            ; EAX = 3D5F0A + 2 * 22D7AAA = 498545E

:00403C2E 5E                pop esi
:00403C2F C3                ret
```

**数学表达式总结**：
上述汇编逻辑等价于公式：
`(X % 0xb640) * 987 + (X * 0x59E60383) >> 46 + Z * 174`
*注：987、46、174 为十进制值。*

## 四、 算法总结

1. **注册码格式**：`X-Y-Z`，由三部分组成，每部分仅限数字或字母。
2. **进制转换**：将 `X`、`Y`、`Z` 视为 **36进制数**，计算其 **16进制值**。
   - 示例：`X=1357` (Base 36) -> `C62B` (Hex)；`Y=2468` (Base 36) -> `181A0` (Hex)；`Z=9012` (Base 36) -> `66866` (Hex)。
3. **校验运算**：利用 `X` 和 `Z` 的十六进制值计算中间结果 `498545E`。
   - 公式：`(0xC62B % 0xb640) * 987 + (0xC62B * 0x59E60383) >> 46 + 0x66866 * 174 = 498545E`
4. **验证条件**：`Y` 的十六进制值必须等于计算出的结果 `498545E`。
5. **生成注册码**：
   - 目标值 `498545E` (Hex) 对应 **36进制** 的 `19WCIM`。
   - 示例注册码：`1357-19WCIM-9012`。

> 提示：可以尝试假设 `X=1`、`Y=2`、`Z=3` 进行简化计算。

## 五、 完美爆破（NOP 修改）

1. 修改 `00403D45`：
   - 原指令：`33C0 xor eax, eax`
   - 改为：`B001 mov al, 01`

2. 修改 `00403C6F`：
   - 原指令：`3BC3 cmp eax, ebx`
   - 改为：`3BC0 cmp eax, eax`

*修改后输入任意注册码均可通过验证。*

## 六、 注册信息保存

```regedit
REGEDIT4

[HKEY_LOCAL_MACHINE\Software\BoilSoft\DvdChapter\SN]
"sn"="1357-19WCIM-9012"
```

## 七、 可用注册码示例

`1357-19WCIM-9012`

## 八、 VC 注册机源码片段

作者：`afanty`
说明：`afanty` 大哥限制了输入的第 1、3 部分为 4 位字符，但足以使用。

```cpp
#include "stdafx.h"
#include "test01.h"
#include "test01Dlg.h"

CTest01Dlg::CTest01Dlg(CWnd* pParent /*=NULL*/)
	: CDialog(CTest01Dlg::IDD, pParent)
{
	//{{AFX_DATA_INIT(CTest01Dlg)
	m_code1 = _T("");
	m_code2 = _T("");
	m_code3 = _T("");
	//}}AFX_DATA_INIT
}

void CTest01Dlg::DoDataExchange(CDataExchange* pDX)
{
	CDialog::DoDataExchange(pDX);
	//{{AFX_DATA_MAP(CTest01Dlg)
	DDX_Control(pDX, IDC_EDIT_3, m_edit3);
	DDX_Control(pDX, IDC_EDIT_2, m_edit2);
	DDX_Control(pDX, IDC_EDIT_1, m_edit1);
//}}AFX_DATA_MAP
}
```

---
Cracked By `afanty fly`
2003-05-01 16:30



```cpp
DDX_Text(pDX, IDC_EDIT_1, m_code1);

DDX_Text(pDX, IDC_EDIT_2, m_code2);

DDX_Text(pDX, IDC_EDIT_3, m_code3);

//}}AFX_DATA_MAP

}

BEGIN_MESSAGE_MAP(CTest01Dlg, CDialog)

//{{AFX_MSG_MAP(CTest01Dlg)

ON_BN_CLICKED(IDC_BUTTON, OnButton)

//}}AFX_MSG_MAP

END_MESSAGE_MAP()

BOOL CTest01Dlg::OnInitDialog()

{

CDialog::OnInitDialog();

m_edit1.SetLimitText(4);

m_edit3.SetLimitText(4);

m_edit2.SetReadOnly();

return TRUE;

}

void CTest01Dlg::OnButton()

{

UpdateData(TRUE);

int i, num;

unsigned long data1 = 0;

unsigned long data2 = 0;

unsigned long data3 = 0;

for (i = 0; i < m_code1.GetLength(); i++)

{

num = m_code1[i];

if ((num >= '0') && (num <= '9')) num = num - 0x30;

else if ((num >= 'A') && (num <= 'Z')) num = num - 0x37;

else if ((num >= 'a') && (num <= 'z')) num = num - 0x57;

else return;

data1 = data1 * 36 + num;

}

for (i = 0; i < m_code3.GetLength(); i++)

{

num = m_code3[i];

if ((num >= '0') && (num <= '9')) num = num - 0x30;

else if ((num >= 'A') && (num <= 'Z')) num = num - 0x37;

else if ((num >= 'a') && (num <= 'z')) num = num - 0x57;

else return;

data3 = data3 * 36 + num;

}

__int64 tmp = 0x59E60383;

tmp = tmp * data1;

tmp = tmp >> 46;

data2 = (data1 % 0xB640) * 987 + (unsigned long)tmp + (data3 * 174);

char ch;

m_code2 = "";

while (data2 > 0)

{

num = data2 % 36;

if ((num >= 0) && (num <= 9)) num = num + 0x30;

else num = num + 0x37;

ch = num;

m_code2.Insert(0, ch);

data2 = data2 / 36;

}

UpdateData(FALSE);

}
```

`4dd466d0e1e0cc123afbe504da2462ca`
