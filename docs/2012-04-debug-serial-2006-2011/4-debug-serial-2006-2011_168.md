
# [ASProtect 2.1x SKE]-脱壳并破解－海医病案统计管理系统 V4.61

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-154.htm`](https://www.pediy.com/kssd/pediy08/pediy8-154.htm)

## xx 病案统计管理系统 V4.61

第一次碰到加了 `ASProtect` 的壳并有自己本身的注册程序，心痒解之。

`ASProtect` 2.1x SKE -> Alexey Solodovnikov

保护方式：Advanced Import protection

### 1. 脱壳

#### 定位 OEP+IAT

```
code：地址=00401000  大小=00217000 (2191360.)
```

**最后一次异常：**

```
0102FAA5      C700 EFCA5C85       mov dword ptr ds:[eax],855CCAEF
0102FAAB      67:64:8F06 0000     pop dword ptr fs:[0]
0102FAB1      83C4 04             add esp,4
0102FAB4      83E8 AF             sub eax,-51
```

**OEP：**

```
00616C7C      55                  push ebp
00616C7D      8BEC                mov ebp,esp
00616C7F      83C4 F0             add esp,-10
00616C82      53                  push ebx
00616C83      B8 74666100         mov eax,main.00616674
00616C88      E8 2FFDDEFF         call main.004069BC
00616C8D      8B1D 54816200       mov ebx,dword ptr ds:[628154]         ; main.00629C0C
```

无 stolen oep。

**IAT：** `RVA=6301A0-00630998`, `SIZE=7F8`

```
00406908    - FF25 70026300       jmp dword ptr ds:[630270]              ; kernel32.TlsGetValue

0063019C  00000000
006301A0  00000000
006301A4  77F87BF9  ntdll.RtlDeleteCriticalSection
006301A8  77F89134  ntdll.RtlLeaveCriticalSection
006301AC  77F89103  ntdll.RtlEnterCriticalSection
...
006301F4  B5CCB129                                  // 好多加密
006301F8  4E33759B
006301FC  77E6E6A9  kernel32.GetProcAddress
00630200  00B5580F
00630204  3BD08651
...
00630954  717185C1  comctl32.ImageList_Destroy
00630958  7171884C  comctl32.ImageList_Create
0063095C  BFCE6B2A
00630960  00000000
00630964  777C440B  winspool.OpenPrinterA
00630968  777C5F7C  winspool.EnumPrintersA
0063096C  777C583B  winspool.DocumentPropertiesA
00630970  777C4964  winspool.ClosePrinter
00630974  00000000
00630978  27BBEA5D
0063097C  00000000
00630980  CA10037F
00630984  67490C24
00630988  3583A4E8
0063098C  C5054AA6
00630990  00000000
00630994  77E69BBA  kernel32.MulDiv
00630998  00000000
```

基本情况已经清楚了，开始 patch：

#### 第一次 patch 前的异常：特征码 ASCII "85"

```
0102EA55      C601 B6        mov byte ptr ds:[ecx],0B6
0102EA58      6323           arpl word ptr ds:[ebx],sp
0102EA5A      DEAA B86B9C6B  fisubr word ptr ds:[edx+6B9C6BB8]
0102EA60      75 11          jnz short 0102EA73

0102EBF8      68 E8F40201    push 102F4E8                       ; ASCII "85"
0102EBFD      E8 2A62FEFF    call 01014E2C
0102EC02      A1 1C370301    mov eax,dword ptr ds:[103371C]
0102EC07      8B00           mov eax,dword ptr ds:[eax]
0102EC09      E8 0A8CFFFF    call 01027818
0102EC0E      84C0           test al,al
0102EC10      75 0A          jnz short 0102EC1C
0102EC12      68 E8F40201    push 102F4E8                       ; ASCII "85"
```

搜索：`3BF0 75 5E EB 01`

```
010275EC      6933 C08A433B   imul esi,dword ptr ds:[ebx],3B438A>
010275F2      3BF0             cmp esi,eax            //在此下断
010275F4      75 5E            jnz short 01027654
010275F6      EB 01            jmp short 010275F9
```

下断后观察 `esi` 值，注意数据窗口的 IAT 变化。

`eax=B8`；`esi=B8`，`6F`，`6B`； 当 `esi=6F` 加密，好 patch 代码：

```
010275F4    - E9 078A2300     jmp 01260000       // 申请的空间
```

```
je 010275F9
cmp esi,06B
je 01027654
mov esi,06B
jmp 01027654
```

patch 后 F9 运行停在下一个异常，撤销所有修改，在 `010275F4` 新建 eip。

运行到最后一个异常，定位第二次 patch 的位置：

搜索：`89,45,F0,B8,00,07,00,00`

```
01027188      8945 F0          mov dword ptr ss:[ebp-10],eax
0102718B      B8 00070000      mov eax,700
01027190      E8 B7B3FDFF      call 0100254C           //第二次 patch 的位置
```

运行到 OEP 后往下搜索 `E8`：

```
00401338      E8 C3ECE700      call 01280000
```

修改第二次 patch 的位置：

```
01027190    - E9 A28E2900      jmp 012C0037       // 申请的空间
```

**patch 代码：**

```
BA 00104000 803A E8 75 12 8B42 01 03C2 83C0 05 3D 00002801 75 03 EB 0C 90 42 81FA 00806100 72 E0 EB FE 8915 00012C01 60 FFE2 90 90 90 90 90 90 90 60 B8 A0016300 90 3910 75 20 8B0D 00012C01 C701 FF250000 8941 02 61 90 8B15 00012C01 90 90 90 EB BE 90 90 90 90 83C0 04 3D 98096300 7E D2 EB E3 90 90
```

```
012C0000      BA 00104000      mov edx,main.<ModuleEntryPoint>    ; CODE 开始
012C0005      803A E8          cmp byte ptr ds:[edx],0E8
012C0008      75 12            jnz short 012C001C
012C000A      8B42 01          mov eax,dword ptr ds:[edx+1]
012C000D      03C2             add eax,edx
012C000F      83C0 05          add eax,5
012C0012      3D 00002801      cmp eax,1280000                    ; call xxxxxxxx
012C0017      75 03            jnz short 012C001C
012C0019      EB 0C            jmp short 012C0027
012C001B      90               nop
012C001C      42               inc edx
012C001D      81FA 00806100    cmp edx,618000                     ; code 结束
012C0023    ^ 72 E0            jb short 012C0005
012C0025    - EB FE            jmp short 012C0025
012C0027      8915 00012C01    mov dword ptr ds:[12C0100],edx
012C002D      60               pushad
012C002E      FFE2             jmp edx
012C0030      90               nop
012C0031      90               nop
012C0032      90               nop
012C0033      90               nop
012C0034      90               nop
012C0035      90               nop
012C0036      90               nop
012C0037      60               pushad
012C0038      B8 A0016300      mov eax,6301A0                     ; IAT 起始
012C003D      90               nop
012C003E      3910             cmp dword ptr ds:[eax],edx
012C0040      75 20            jnz short 012C0062
012C0042      8B0D 00012C01    mov ecx,dword ptr ds:[12C0100]
012C0048      C701 FF250000    mov dword ptr ds:[ecx],25FF
012C004E      8941 02          mov dword ptr ds:[ecx+2],eax
012C0051      61               popad
012C0052      90               nop
012C0053      8B15 00012C01    mov edx,dword ptr ds:[12C0100]
012C0059      90               nop
012C005A      90               nop
012C005B      90               nop
012C005C    ^ EB BE            jmp short 012C001C
012C005E      90               nop
012C005F      90               nop
012C0060      90               nop
012C0061      90               nop
012C0062      83C0 04          add eax,4
012C0065      3D 98096300      cmp eax,630998                     ; IAT 结束
012C006A    ^ 7E D2            jle short 012C003E
012C006C    ^ EB E3            jmp short 012C0051
012C006E      90               nop
012C006F      90               nop
```

在 `012C0000` 新起 eip，F9 运行。

`lordpe` 选择进程 dump，`ImportREC` 选择进程，`OEP=00216C7C`, 自动搜索，没有无效函数，修复。

运行 ok，`Borland Delphi 6.0 - 7.0`。

### 2. 破解

#### 获得注册码

单位名称：MeiLun
负责人：God
机器码：WD-WCAD13523066
注册码：87654321
错误提示：注册码错误。

算法查询：Sha－1，Base64

`MessageBoxExA`，断下后返回，往上找源头：

```
005BA0FC      55               push ebp
```

初始化一串数据：

```
005BA133      C705 00F76200 0>mov dword ptr ds:[62F700],4
005BA13D      C705 04F76200 0>mov dword ptr ds:[62F704],2
005BA147      C705 08F76200 0>mov dword ptr ds:[62F708],5
005BA151      C705 0CF76200 0>mov dword ptr ds:[62F70C],7
005BA15B      C705 10F76200 0>mov dword ptr ds:[62F710],8
005BA165      C705 14F76200 0>mov dword ptr ds:[62F714],3
005BA16F      C705 18F76200 0>mov dword ptr ds:[62F718],1
005BA179      C705 1CF76200 0>mov dword ptr ds:[62F71C],6
```

取得机器码与上面的数据相加：

```
005BA193      C703 01000000    mov dword ptr ds:[ebx],1
005BA199      8D55 F8          lea edx,dword ptr ss:[ebp-8]
005BA19C      A1 ECF66200      mov eax,dword ptr ds:[62F6EC]
005BA1A1      E8 E2ECE4FF      call dumped\_.00408E88             ; 机器码
005BA1A6      8B45 F8          mov eax,dword ptr ss:[ebp-8]
005BA1A9      8B13             mov edx,dword ptr ds:[ebx]
005BA1AB      8A4410 FF        mov al,byte ptr ds:[eax+edx-1]
005BA1AF      8B13             mov edx,dword ptr ds:[ebx]
005BA1B1      8882 28F76200    mov byte ptr ds:[edx+62F728],al
```



# 逆向工程分析记录

## 1. 机器码生成逻辑

以下汇编代码展示了机器码的计算过程：

```asm
005BA1B7      8B03             mov eax,dword ptr ds:[ebx]
005BA1B9      0FB680 28F76200 movzx eax,byte ptr ds:[eax+62F728]
005BA1C0      8B13             mov edx,dword ptr ds:[ebx]
005BA1C2      030495 FCF66200 add eax,dword ptr ds:[edx*4+62F6FC]    ; 相加
005BA1C9      8D55 F4          lea edx,dword ptr ss:[ebp-C]
005BA1CC      E8 43F1E4FF      call dumped_.00409314                  ; 转换成十进制
005BA1D1      8B55 F4          mov edx,dword ptr ss:[ebp-C]
005BA1D4      8B03             mov eax,dword ptr ds:[ebx]
005BA1D6      8D0485 34F76200 lea eax,dword ptr ds:[eax*4+62F734]
005BA1DD      E8 92A2E4FF      call dumped_.00404474
005BA1E2      FF03             inc dword ptr ds:[ebx]
005BA1E4      4E               dec esi
005BA1E5    ^ 75 B2           jnz short dumped_.005BA199
```

**相关数据参数：**

```text
0062F700  04 00 00 00 02 00 00 00 05 00 00 00 07 00 00 00  ........
0062F710  08 00 00 00 03 00 00 00 01 00 00 00 06 00 00 00  ........
0062F720  00 00 00 00 00 00 00 00 00 57 44 2D 57 43 41 44  .......WD-WCAD
0062F730  31 33 35 32 33 30 36 36 34                       135230664
```

通过提取机器码片段（每次 4 字节），并将累加值转换为十进制，共计算 15 次。每次计算结果取特定位（第 2 位）进行连接，最终得到注册码。

**计算过程及取值映射：**

- 堆栈 `ss:[0012F0AC]=010745F4` (ASCII "91")：值为 8，取第 1 位 -> `W`
- 堆栈 `ss:[0012F0AC]=01076C34` (ASCII "70")：值为 4，取第 0 位 -> `D`
- 堆栈 `ss:[0012F0AC]=00FC30D4` (ASCII "50")：值为 3，取第 0 位 -> `-`
- 堆栈 `ss:[0012F0AC]=00FC744C` (ASCII "94")：值为 1，取第 4 位 -> `W`
- 堆栈 `ss:[0012F0AC]=080FD264` (ASCII "75")：值为 5，取第 5 位 -> `C`
- 堆栈 `ss:[0012F0AC]=080FD274` (ASCII "68")：值为 6，取第 8 位 -> `A`
- 堆栈 `ss:[0012F0AC]=080FD284` (ASCII "69")：值为 2，取第 9 位 -> `D`
- 堆栈 `ss:[0012F0AC]=080FD294` (ASCII "55")：值为 7，取第 5 位 -> `1`
- 堆栈 `ss:[0012F0AC]=080FD2A4` (ASCII "51")：取值 3
- 堆栈 `ss:[0012F0AC]=080FD2B4` (ASCII "53")：取值 5
- 堆栈 `ss:[0012F0AC]=080CC9B4` (ASCII "759453490")：取值 2
- 堆栈 `ss:[0012F0AC]=07EE4D84` (ASCII "1145127818")：取值 3
- 堆栈 `ss:[0012F0AC]=080D49F8` (ASCII "842347361")：取值 0
- 堆栈 `ss:[0012F0AC]=01025FF4` (ASCII "3551337")：取值 6
- 堆栈 `ss:[0012F0AC]=00FD15A4` (ASCII "17253930")：取值 6

### 生成注册码片段

```asm
005BA2B5      B8 7CF76200     mov eax,dumped_.0062F77C
005BA2BA      8B13            mov edx,dword ptr ds:[ebx]
005BA2BC      8B1495 58F76200 mov edx,dword ptr ds:[edx*4+62F758]
005BA2C3      E8 20A4E4FF     call dumped_.004046E8
005BA2C8      FF03            inc dword ptr ds:[ebx]
005BA2CA      833B 09         cmp dword ptr ds:[ebx],9
005BA2CD    ^ 75 E6          jnz short dumped_.005BA2B5
```

最终计算结果为 `49005851`。

## 2. 注册码验证

程序将生成的 `49005851` 与输入的注册码进行比较：

```asm
005BA2D8      E8 0FFEEAFF     call dumped_.0046A0EC          ; 取注册码
005BA2DD      8B55 F0         mov edx,dword ptr ss:[ebp-10]  ; ZCM
005BA2E0      A1 7CF76200     mov eax,dword ptr ds:[62F77C]  ; 49005851
005BA2E5      E8 42A5E4FF     call dumped_.0040482C          ; 比较
005BA2EA      74 1E          je short dumped_.005BA30A
```

若相等则提示注册成功。观察发现并未直接写入注册表或本地注册文件，推测数据可能写入数据库文件 `dataset.mdb`。

## 3. 错误信息分析与定位

重启程序后若注册信息错误，会弹出 `MessageBoxExA` 提示：

```text
0012F044   77E180BC    /CALL to MessageBoxExA from user32.77E180B7
0012F048   002801FE    |hOwner = 002801FE ('xx 病案统计管理系统',class='TApplication')
0012F04C   0058D7F8    |Text = "注册信息错误 - NO.1"
0012F048   0058D798    |Title = "提示"
0012F054   00000000    |Style = MB_OK|MB_APPLMODAL
0012F058   00000000    \LanguageID = 0 (LANG_NEUTRAL)
```

回溯调用链至 `dumped_.0058CFD3`，通过 `W32Dasm` 查找源头：

```asm
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0058CCC5(C)
|
:0058CCD7 8B06             mov eax, dword ptr [esi]
```

后续代码中检测到可能的字符串引用：
- "单位名称"
- "注册码"
- "单位负责人"

继续向下追踪，发现程序读取配置文件 `hytj.ini` 的内容：

```asm
0058CE68      B9 28D75800   mov ecx,dumped_.0058D728       ; ASCII "hytj.ini"
0058CE6D      E8 BA78E7FF   call dumped_.0040472C
0058CE72      8B4D DC       mov ecx,dword ptr ss:[ebp-24]
0058CE75      B2 01         mov dl,1
0058CE77      A1 00424400   mov eax,dword ptr ds:[444200]
0058CE7C      E8 2F74EBFF   call dumped_.004442B0
0058CE81      A3 D4F56200   mov dword ptr ds:[62F5D4],eax
0058CE86      6A 00         push 0
0058CE88      8D45 D8       lea eax,dword ptr ss:[ebp-28]
0058CE8B      50            push eax
0058CE8C      B9 3CD75800   mov ecx,dumped_.0058D73C       ; ASCII "PKCGH"
0058CE91      BA 4CD75800   mov edx,dumped_.0058D74C       ; ASCII "SET"
0058CE96      A1 D4F56200   mov eax,dword ptr ds:[62F5D4]
0058CE9B      8B38          mov edi,dword ptr ds:[eax]
0058CE9D      FF17          call dword ptr ds:[edi]        ; 读取文件 hytj.ini 的 PKCGH
```

进入该调用后，实际执行了 `GetPrivateProfileStringA`：

```text
00444C1C      E8 EB21FCFF     call <jmp.&kernel32.GetPrivateProfileStringA>

0012E8B8   00444C21    /CALL to GetPrivateProfileStringA from dumped_.00444C1C
0012E8BC   0058D74C    |Section = "SET"                      // 读取 SET 部分
0012E8C0   0058D73C    |Key = "PKCGH"                        // 读取 PKCGH 段内容
0012E8C4   004048E5    |Default = ""
0012E8C8   0012E8E0    |ReturnBuffer = 0012E8E0
0012E8CC   00000800    |BufSize = 800 (2048.)
0012E8D0   080FD0A4    \IniFileName = "E:\xx 病案统计管理系统\hytj.ini"
```

## 4. 关键逻辑判断

读取后进行检查：

```asm
0058CEFC      837D D8 00      cmp dword ptr ss:[ebp-28],0         ; 数值来源待确认
0058CF00      74 11          je short dumped_.0058CF13           ; 是否为空？
0058CF02      8B45 D8        mov eax,dword ptr ss:[ebp-28]
0058CF05      8B55 CC        mov edx,dword ptr ss:[ebp-34]       ; (ASCII "D59AW8fl")
0058CF08      E8 1F79E7FF    call dumped_.0040482C               ; 比较
0058CF0D      0F84 C5000000  je dumped_.0058CFD8                 ; 关键跳转
```

若 `[ebp-28]` 内容为空，则触发错误信息。若要绕过此检查，需将 `[ebp-28]` 的内容修改为字符串 `D59AW8fl`。

## 5. 修复与注册配置

检查原始 `hytj.ini` 文件，发现缺少 `PKCGH` 键值：

```ini
[set]
database =database\database.mdb
babase   =database\mzba.mdb
ime      =王码五笔型输入法 86 版
datainput=2
```

在文件中添加 `PKCGH` 部分并保持对齐格式：

```ini
[set]
database =database\database.mdb
babase   =database\mzba.mdb
ime      =王码五笔型输入法 86 版
datainput=2
PKCGH    =D59AW8fl
```

重新加载程序后，`[ebp-28]` 读取到 `D59AW8fl`，通过比较检查，跳过错误提示，进入主界面。注册菜单变为灰色（已注册状态），流程完成。

## 6. 注册信息摘要

- **单位名称**：`MeiLun`
- **负责人**：`God`
- **机器码**：`WD-WCAD13523066`
- **注册码**：`49005851`
- **配置文件添加项**：`PKCGH = D59AW8fl`

## 7. 后记

追踪 `D59AW8fl` 的生成过程发现其复杂性，涉及 `SHA-1` 加密、`Base64` 编码以及作者自定义的加密算法。该算法对“注册单位名称”与“注册码”的计算结果进行加密，循环过程较长，但最终还原为明文进行比较。因此，注册信息强依赖于单位名称和机器码。

`4dd466d0e1e0cc123afbe504da2462ca`
