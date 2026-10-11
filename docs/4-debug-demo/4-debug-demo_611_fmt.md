
# QQ2000C 1230 标准版实现去广告

> 原文：[`www.pediy.com/kssd/pediy05/pediy50515.htm`](https://www.pediy.com/kssd/pediy05/pediy50515.htm)

如何修改 `QQ2000C 1230` 标准版实现去广告、给隐身朋友发送文件。

（首先感谢提供解决方法的朋友们：最弱智、`dRag0nMa`、`lllaaa`、`pll621`）

1. 首先要屏蔽 QQ 自动从服务器上下载广告图片和 `flash` 文件。
2. 在对话框广告位置区域中去掉响应鼠标点击打开广告文件或动画。
3. 给隐身的朋友发送文件。

正文开始 >>>

### 1、屏蔽 QQ 自动从服务器上下载广告图片和 flash 文件

#### 方法一

`dRag0nMa` 建议，以前最弱智也提过，修改 `QQ.exe` 中的 `"AD\"` 的 `ASCII` 跟 `UNICODE` 为 `"A\*\\"` 或者其他让系统认为非法的名称。可以使用 `WINHEX` 替换修改。

#### 方法二

`小皮皮`---`lllaaa` 提供，修改 `QQ.exe` 的程序代码实现不下载广告文件。

```asm
* Reference To: MFC42.Ordinal:1943, Ord:1943h
                      |
:004C1C09 E8F27D0000             Call 004C9A00
:004C1C0E 33C0                   xor eax, eax
:004C1C10 398644010000           cmp dword ptr [esi+00000144], eax
:004C1C16 8945FC                 mov dword ptr [ebp-04], eax
:004C1C19 742B                   je 004C1C46   //NOP 掉就不下载广告了
:004C1C1B 6A01                   push 00000001
:004C1C1D 51                     push ecx
:004C1C1E 8BCC                   mov ecx, esp
:004C1C20 896508                 mov dword ptr [ebp+08], esp

* Possible StringData Ref from Data Obj ->"Downing..."
                      |
:004C1C23 68B0A05500             push 0055A0B0

* Reference To: MFC42.Ordinal:0219, Ord:0219h
                      |
:004C1C28 E85F7F0000             Call 004C9B8C
:004C1C2D 51                     push ecx
:004C1C2E C645FC01               mov [ebp-04], 01
:004C1C32 8BCC                   mov ecx, esp
:004C1C34 89650C                 mov dword ptr [ebp+0C], esp

* Possible StringData Ref from Data Obj ->"adlog"
                      |
:004C1C37 68F89D5500             push 00559DF8

* Reference To: MFC42.Ordinal:0219, Ord:0219h
                      |
:004C1C3C E84B7F0000             Call 004C9B8C
:004C1C41 E926030000             jmp 004C1F6C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C1C19(C)
|
* Possible StringData Ref from Data Obj ->"11tracymutex200211121427"
                      |
:004C1C46 6894A05500             push 0055A094
:004C1C4B 50                     push eax
:004C1C4C 50                     push eax

* Reference To: KERNEL32.CreateMutexA, Ord:003Fh
                      |
:004C1C4D FF1520F14F00           Call dword ptr [004FF120]
:004C1C53 6A01                   push 00000001
:004C1C55 A3CCAD5500             mov dword ptr [0055ADCC], eax
:004C1C5A 5B                     pop ebx
:004C1C5B 53                     push ebx
:004C1C5C 51                     push ecx
:004C1C5D 8BCC                   mov ecx, esp
:004C1C5F 896508                 mov dword ptr [ebp+08], esp

* Possible StringData Ref from Data Obj ->"CreateAdDownMutex"
                      |
:004C1C62 6880A05500             push 0055A080

* Reference To: MFC42.Ordinal:0219, Ord:0219h
```

### 2、在对话框广告位置区域中去掉响应鼠标点击打开广告文件或动画

利用 `createfilea` 断点分析，修改如下代码可实现。

```asm
 * Reference To: KERNEL32.CreateFileA, Ord:0034h
                      |
:1002FBFE FF15D4400610           Call dword ptr [100640D4]
:1002FC04 8BF8                   mov edi, eax
:1002FC06 83FFFF                 cmp edi, FFFFFFFF
:1002FC09 897D08                 mov dword ptr [ebp+08], edi
:1002FC0C 0F84EB000000           je 1002FCFD // JMP 后对鼠标点击不反应
:1002FC12 8D45F4                 lea eax, dword ptr [ebp-0C]
:1002FC15 50                     push eax
:1002FC16 57                     push edi

* Reference To: KERNEL32.GetFileSize, Ord:0112h
                      |
:1002FC17 FF15B0400610           Call dword ptr [100640B0]
```

### 3、给隐身的朋友发送文件

#### 方法一

`老妖怪` `PLL621` 提供，修改程序 `CQQApplication.dll` 代码，让灰色按钮亮起来：`01A1B3CB FF7508 PUSH DWORD [EBP+08]=>` 改为 `push 0`。

#### 方法二

修改程序 `CQQApplication.dll` 代码，让程序始终认为用户在线，利用对话框函数 `MessageboxA` 跟到程序空间，过了 `MFC` 的调用后，返回到 `CQQApplication.dll` 领空，往上分析，跟入判断调用的 `CALL`，修改如下代码可以实现给隐身的朋友发送文件：

```asm
:1003CC71 FF5138                 call [ecx+38]
:1003CC74 837D0814               cmp dword ptr [ebp+08], 00000014 //不在线否？
:1003CC78 8B450C                 mov eax, dword ptr [ebp+0C]
:1003CC7B 885DFC                 mov byte ptr [ebp-04], bl
:1003CC7E 7420                   je 1003CCA0 //NOP 掉即可给隐身朋友发送文件
:1003CC80 3BC3                   cmp eax, ebx
:1003CC82 7406                   je 1003CC8A
:1003CC84 8B08                   mov ecx, dword ptr [eax]
:1003CC86 50                     push eax
:1003CC87 FF5108                 call [ecx+08]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1003CC82(C)
|
:1003CC8A 8B45F0                 mov eax, dword ptr [ebp-10]
:1003CC8D 834DFCFF               or dword ptr [ebp-04], FFFFFFFF
:1003CC91 3BC3                   cmp eax, ebx
:1003CC93 7406                   je 1003CC9B
```

<The End> `4dd466d0e1e0cc123afbe504da2462ca`
