
# 两个简单的手机游戏分析

> 原文：[`www.pediy.com/kssd/pediy10/83758.html`](https://www.pediy.com/kssd/pediy10/83758.html)

【文章标题】: 第一个-手机游戏爆破

【文章作者】: `wangdell`

【软件名称】: `TextTwist.exe`

【下载地址】: 自己搜索下载

【编写语言】: `arm`

【使用工具】: `IDA` `WM6SDK` `UE`

【操作平台】: `wince`

--------------------------------------------------------------------------------

【详细过程】

老婆手机中很多小游戏，都需要注册。一直想看看 `wince` 到底是怎么回事。弄了两天，终于搞懂一些。刚开始看指令，好象不太难，以为很简单，但里面是非明码比较，无法追码，算法看了半天也没看懂，可能是用了什么现成的算法库。有时间再细研究吧，手机游戏的处女破，就不和她缠绵了，先爆破一下。hiahia.

1、准备

1.1 `IDA` 打开 `Texttwister.exe`。静态分析。

1.2 开 `Activesync`。文件=>连接设置 中 选择 `DMA`。

1.3 开启 `WM6SDK` 带的设仿真器管理器，选 `Pocket pc 20003 SE` 仿真程序，右键“连接”。

1.4 选 `Pocket pc 20003 SE` 仿真程序，右键“插入底座”。等待 `activysync` 和仿真器连接和同步。

1.5 真机中试运行，在程序底部 `tools->register` 菜单中，可进入注册对话框，提示输入用户名和密码。

2、调试

在 `IDA` 的 `imports` 中浏览 `api`,看到 `DialogBoxIndirectParamW`.按 `x` 查调用处，并在其上尝试设断。最终找到

```
.text:00015B28 MOV R3, #0
.text:00015B2C MOV R1, R0 ; hDialogTemplate
.text:00015B30 STR R3, [SP,#0x29C+dwInitParam]
.text:00015B34 LDR R3, =regdlgporc ; lpDialogFunc
.text:00015B38 MOV R2, R6 ; hWndParent
.text:00015B3C LDR R0, [R4] ; hInstance
.text:00015B40 BL DialogBoxIndirectParamW
```

为注册对话框调用处。

进入 `regdlgproc` 中看到

```
.text:00015988 SUB SP, SP, #0x35C
.text:0001598C MOV R3, #WM_INITDIALOG
.text:00015990 MOV R4, R0
.text:00015994 CMP R1, R3
.text:00015998 BEQ loc_15A18
.text:0001599C MOVL R3, WM_COMMAND
.text:000159A4 CMP R1, R3
.text:000159A8 BNE loc_159C4
.text:000159AC MOV R3, R2,LSL#16
.text:000159B0 MOV R5, R3,LSR#16
.text:000159B4 CMP R5, #1
.text:000159B8 BEQ loc_159CC
.text:000159BC CMP R5, #2
.text:000159C0 BEQ loc_159CC
.text:000159C4
.text:000159C4 loc_159C4 ; CODE XREF: regdlgporc+24j
.text:000159C4 MOV R0, #0
.text:000159C8 B loc_15AA8
.text:000159CC ; ---------------------------------------------------------------------------
.text:000159CC
.text:000159CC loc_159CC ; CODE XREF: regdlgporc+34j
; regdlgporc+3Cj
.text:000159CC LDR R2, =name ; lpString
.text:000159D0 MOV R3, #0x64 ; 'd' ; nMaxCount
.text:000159D4 MOV R1, #0x65 ; 'e' ; nIDDlgItem
.text:000159D8 MOV R0, R4 ; hDlg
.text:000159DC BL GetDlgItemTextW
.text:000159E0 LDR R2, =code ; lpString
.text:000159E4 MOV R3, #0x64 ; 'd' ; nMaxCount
.text:000159E8 MOV R1, #0x66 ; 'f' ; nIDDlgItem
.text:000159EC MOV R0, R4 ; hDlg
.text:000159F0 BL GetDlgItemTextW
.text:000159F4 MOV R1, R5 ; nResult
.text:000159F8 MOV R0, R4 ; hDlg
.text:000159FC BL EndDialog
.text:00015A00 MOV R1, #1 ; st
.text:00015A04 MOV R0, R4 ; hwnd
.text:00015A08 BL SHSipPreference
.text:00015A0C B loc_15AA4
```

用 `ida` 的 `grahview` 较易看出，其根据消息是 `#WM_INITDIALOG` 或 `WM_COMMAND` 分别处理。

在其中的两个 `GetDlgItemTextW` 分别是获得 `name` 和 `code` 处。

在相应位置设断。

`f9` 运行，`ida` 提示与移动设备连接，并提示需下载到移动设备中，确定后，程序开始运行，中断后，单步跟踪来到

```
.text:00015B78 MOV R3, R0,LSL#16
.text:00015B7C MOVS R5, R3,ASR#16
.text:00015B80 NOP
.text:00015B84 MOV R1, #1
.text:00015B88 MOV R0, R7
.text:00015B8C BL codecpy
.text:00015B90 B loc_15BB4
.text:00015B94 ; ---------------------------------------------------------------------------
.text:00015B94 MOVL R3, 0xFFFFFFEA
.text:00015B98 CMP R5, R3
.text:00015B9C BNE loc_15BB0
.text:00015BA0 MOVL R0, 0x1B6C
.text:00015BA8 BL sub_155D0
.text:00015BAC B loc_15BB4
.text:00015BB0 ; ---------------------------------------------------------------------------
.text:00015BB0
.text:00015BB0 loc_15BB0 ; CODE XREF: sub_15AB8+E4j
.text:00015BB0 BL sub_15F8C
.text:00015BB4
.text:00015BB4 loc_15BB4 ; CODE XREF: sub_15AB8+D8j
; sub_15AB8+F4j
.text:00015BB4 CMP R5, #0
.text:00015BB8 BNE loc_15B04
.text:00015BBC BL sub_1876C
.text:00015BC0
.text:00015BC0 loc_15BC0 ; CODE XREF: sub_15AB8+98j
.text:00015BC0 MOV R0, #0
.text:00015BC4
.text:00015BC4 loc_15BC4 ; CODE XREF: sub_15AB8+40j
.text:00015BC4 ADD SP, SP, #0x288
.text:00015BC8 LDMFD SP!, {R4-R7,PC}
```

`00015BB4` 处是关键比较，如不等，则跳回再次调用注册对话框。

向上翻

```
.text:00015B40 BL DialogBoxIndirectParamW
.text:00015B44 MOV R3, R0,LSL#16
.text:00015B48 MOV R3, R3,ASR#16
.text:00015B4C CMP R3, #1
.text:00015B50 BNE loc_15BC0
.text:00015B54 CMP R7, #0
.text:00015B58 BEQ loc_15B70
.text:00015B5C MOV R0, R7
.text:00015B60 BL codecheck_0
```

看到 `15b60` 处，应为关键函数 跟进 `codecheck_0` 函数。

```
.text:0001A6A8 codecheck_0 ; CODE XREF: sub_15AB8+A8p
; sub_19B38+14p ...
.text:0001A6A8
.text:0001A6A8 name1 = -0x30C
.text:0001A6A8 var_28C = -0x28C
.text:0001A6A8 name = -0x288
.text:0001A6A8
.text:0001A6A8 STMFD SP!, {R4,LR}
.text:0001A6AC SUB SP, SP, #0x304
.text:0001A6B0 MOV R4, R0
.text:0001A6B4 MOV R3, #0x284
.text:0001A6B8 ADD R0, SP, #0x30C+var_28C
.text:0001A6BC STR R3, [SP,#0x30C+var_28C]
.text:0001A6C0 BL regkey
.text:0001A6C4 ADD R1, SP, #0x30C+name ; wchar_t *
.text:0001A6C8 ADD R0, SP, #0x30C+name1 ; wchar_t *
.text:0001A6CC BL wcscpy
.text:0001A6D0 MOV R2, #1
.text:0001A6D4 ADD R1, SP, #0x30C+name1
.text:0001A6D8 MOV R0, R4
.text:0001A6DC BL codecheck
.text:0001A6E0 ADD SP, SP, #0x304
.text:0001A6E4 LDMFD SP!, {R4,PC}
```

看到 `1a6dc` 处为关键函数。

这里爆破吧，更改

```
.text:0001A6DC 4A FD FF EB BL codecheck
```

为

```
.text:0001A6DC 00 00 A0 E3 MOV R0, #0
```

重新运行，任意即可注册成功。

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2009 年 03 月 14 日 1:13:24

【文章标题】: 第二个-手机游戏追码

【文章作者】: `wangdell`

【软件名称】: `美女麻将.exe`

【软件大小】: `421K`

【下载地址】: 自己搜索下载

【保护方式】: 无

【编写语言】: `MFC?`

【使用工具】: `IDA` `ActiveSync` `WM6SDK`

【操作平台】: `Wince`

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

【详细过程】

又看了一个手机游戏，这个比较简单，内存中明码比较，适合我等新手。

1.准备

准备方法与上一篇相同。

1.1 `IDA` 打开"美女麻将.exe"。静态分析。

1.2 开 `Activesync`。文件=>连接设置 中 选择 `DMA`。

1.3 开启 `WM6SDK` 带的设仿真器管理器，选 `Pocket pc 20003 SE` 仿真程序，右键“连接”。

1.4 选 `Pocket pc 20003 SE` 仿真程序，右键“插入底座”。等待 `activysync` 和仿真器连接和同步。

1.5 真机中试运行，注意到程序启动有提示注册，时间限制为 14 天。进入注册对话框后，输入假码，会弹出 `messagebox`, 提示错误。

2.分析

在 `IDA` 中 `imports` 中浏览 `api`，看到几个可疑函数

```



``` 
  .idata:0002B0A4  ; int __stdcall lstrcmpiW(LPCWSTR lpString1, LPCWSTR lpString2)
  .idata:0002B0A4                  IMPORT __imp_lstrcmpiW  ; DATA XREF: .text:off_28F70o
  
  .idata:0002B258  ; __declspec(dllimport) int __cdecl AfxMessageBox(unsigned int, unsigned int, unsigned int)
  .idata:0002B258                  IMPORT __imp__AfxMessageBox__YAHIII_Z
  
  .idata:0002B034  ; int __stdcall MessageBoxW(HWND hWnd, LPCWSTR lpText, LPCWSTR lpCaption, UINT uType)
  .idata:0002B034                  IMPORT __imp_MessageBoxW ; DATA XREF: .text:off_28E20o

```

按 `x`,在相关调用处翻找代码。发现下面的可以代码：

代码:
```
  .text:00027FA0                  LDR     R1, =a105d088a0595  ; lpString2
  .text:00027FA4                  ADD     R0, SP, #0x32C+String1  ; lpString1
  .text:00027FA8                  BL      lstrcmpiW
  .text:00027FAC                  CMP     R0, #0
  .text:00027FB0                  BNE     loc_28008
  .text:00027FB4                  MOV     R0, #0xE1
  .text:00027FB8                  BL      sub_11374
  .text:00027FBC                  MOV     R1, R0          ; lpFormat
  .text:00027FC0                  ADD     R2, SP, #0x32C+String1
  .text:00027FC4                  ADD     R0, SP, #0x32C+Buffer  ; lpBuffer
  .text:00027FC8                  BL      wsprintfW
  .text:00027FCC                  MOV     R1, #0x810
  .text:00027FD0                  ADD     R2, SP, #0x32C+Buffer
  .text:00027FD4                  ORR     R1, R1, #6
  .text:00027FD8                  MOV     R0, R5
  .text:00027FDC                  BL      _SetDlgItemText_CWnd__QAAXHPBG_Z  ; CWnd::SetDlgItemText(int,ushort const *)
  .text:00027FE0                  MOVL    R1, 0x814
  .text:00027FE8                  MOV     R0, R5
  .text:00027FEC                  BL      _GetDlgItem_CWnd__QBAPAV1_H_Z  ; CWnd::GetDlgItem(int)
  .text:00027FF0                  MOV     R1, #0
  .text:00027FF4                  BL      _ShowWindow_CWnd__QAAHH_Z  ; CWnd::ShowWindow(int)
  .text:00027FF8                  ADD     R0, SP, #0x32C+var_28C
  .text:00027FFC                  BL      __1CDialog_tbc__UAA_XZ  ; CDialog_tbc::~CDialog_tbc(void)
  .text:00028000                  ADD     SP, SP, #0x320
  .text:00028004                  LDMFD   SP!, {R4,R5,PC}
  .text:00028008 ; ---------------------------------------------------------------------------
  .text:00028008
  .text:00028008 loc_28008                       ; CODE XREF: sub_27F4C+64j
  .text:00028008                  MOVL    R2, 0xFFFFFFFF  ; unsigned int
  .text:0002800C                  MOV     R1, #0          ; unsigned int
  .text:00028010                  MOV     R0, #0xE2       ; unsigned int
  .text:00028014                  BL      _AfxMessageBox__YAHIII_Z  ; AfxMessageBox(uint,uint,uint)

```

注意到此处有用到 `strcmp` 和 `AfxMessageBox` 这两个函数。

在 `00027FA0` 代码中的 `=a105d088a0595` 处双击跟随来到

代码:
```
.data:0002B340 a105d088a0595   unicode 0, <105D-088A-0595>,0  ; DATA XREF: .text:lpString2o

```

这里就是真码"`105D-088A-0595`"

在真机中输入这个注册码后，注册成功。

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2009 年 03 月 14 日 16:25:52

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || TextTwist.rar [附件请到论坛下载：[`bbs.pediy.com/showthread.php?t=83758`](http://bbs.pediy.com/showthread.php?t=83758 ) ] |

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || 美女麻将.rar |

