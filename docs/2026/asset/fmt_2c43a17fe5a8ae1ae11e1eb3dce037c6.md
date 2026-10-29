
# [原创]某 DRM 解密流程简单分析

## 环境及工具

- 手机：小米手机 MI 2A
- 系统版本：Android 4.1.1
- 工具：IDA Pro 6.6、C32Asm、VS2005

帮朋友写了一个解密 DRM 视频的小软件，我把分析该播放器的调试步骤记录下来与大家一起学习。高手莫要见笑，仅供小菜玩乐，有不对或不足的地方还请多多指教，不胜感激！

## 一：第一次打开加密视频会出现如下验证

![](https://bbs.kanxue.com/upload/attach/201501/536985_turl6p0nj7yoa2x.png)

输入用户名与密码登录成功后如下图：

![](https://bbs.kanxue.com/upload/attach/201501/536985_o7zss0gjos5y0mm.png)

点击“支付并获取许可证”成功后就可以播放加密的视频了，并在安装包目录下生成了两个重要的文件：`indiv3.key` 与 `V3.lic`。

## 二：加密后视频大致结构如下

![](https://bbs.kanxue.com/upload/attach/201501/536985_eag3bbnivge1b15.jpg)

1. 加密视频头中有加密标志、密钥等信息。
2. 32 字节检验码是判断视频是否有被篡改。
3. WMA 文件的头 16 个字节是固定的，为十六进制的“30 26 B2 75 8E 66 CF 11 A6 D9 00 AA 00 62 CE 6C”，用来标识这个是否为 WMA 文件。
4. 接下来的 8 个字节为一个整数，表示整个 WMA 文件头部的大小（定位到加密数时会用到）。

## 三：动态调试播放器分析解密流程，分为如下步骤

通过反编译分析 APK 包发现，主要逻辑是在 `.so` 中实现的，所以我们主要是调试分析 `.so`。

### 第 1 步

运行播放器，调试器附加上，对如下函数进行下断点：

```text
open_stream

demux_open

av_base64_decode

av_sha_init

av_rc4_init

av_aes_init
```

### 第 2 步

F9 运行，打开一个加密的视频程序，断在 `open_stream` 函数。打开成功后接着来到 `demux_open` 函数，这一步中主要是做打开并读取加密视频头 `0x800` 字节，并判断是否有 `"XDRM"`、`"SIGN"` 标志。

```asm
libmplayer.so:5EB594C4

libmplayer.so:5EB594C4 loc_5EB594C4                            ; CODE XREF: sub_5EB5922C+93Cj

libmplayer.so:5EB594C4 LDR             LR, =(aHarrydebug6666 - 0x5EB594D8)

libmplayer.so:5EB594C8 MOV             R0, #1

libmplayer.so:5EB594CC MOV             R1, #0

libmplayer.so:5EB594D0 ADD             R2, PC, LR              ; "harrydebug6666.\n"

libmplayer.so:5EB594D4 BL              mp_msg

libmplayer.so:5EB594D8 LDRB            R2, [SP,#0xDF0+var_D10]

libmplayer.so:5EB594DC CMP             R2, #'X'                ; 判断是否有 XDRM 头标志

libmplayer.so:5EB594E0 BEQ             loc_5EB5995C

libmplayer.so:5EB5995C loc_5EB5995C                            ; CODE XREF: sub_5EB5922C+2B4j

libmplayer.so:5EB5995C LDRB            R0, [SP,#0xDF0+var_D0F]

libmplayer.so:5EB59960 CMP             R0, #'D'

libmplayer.so:5EB59964 BNE             loc_5EB594E4

libmplayer.so:5EB59968 LDRB            R3, [SP,#0xDF0+var_D0E]

libmplayer.so:5EB5996C CMP             R3, #'R'

libmplayer.so:5EB59970 BNE             loc_5EB594E4

libmplayer.so:5EB59974 LDRB            R1, [SP,#0xDF0+var_D0D]

libmplayer.so:5EB59978 CMP             R1, #'M'

libmplayer.so:5EB5997C BNE             loc_5EB594E4
```

### 第 3 步

解码加密视频头文件中的 Base64 字符（`yJ3pPh4RItrcgckEtqjS+Q==`）：

```asm
libmplayer.so:5EB5A704 STRB            R6, [R4],#1

libmplayer.so:5EB5A708 BL              av_base64_decode        ; 解码 yJ3pPh4RItrcgckEtqjS+Q==
```

解码后为：

```text
C8 9D E9 3E 1E 11 22 DA  DC 81 C9 04 B6 A8 D2 F9
```

### 第 4 步

RC4 解密“第 3 步”得到的值。

密钥为固定的一个字符串：`Tiasfy!!`

解密前：

```text
C8 9D E9 3E 1E 11 22 DA  DC 81 C9 04 B6 A8 D2 F9
```

解密后：

```text
65 35 30 65 66 63 65 31  33 36 63 35 34 31 38 38（字符形式 e50efce136c54188）
```

### 第 5 步

拷贝加密视频中的加密头下面 32 字节 SHA-1 值，后面做检验用。

```text
39 34 53 B6 DE B8 AF B9  D7 2B 14 25 85 1C B9 D2

D2 79 B3 B2 28 B7 BA 82  F0 49 07 9E 71 6A 5E BF
```

### 第 6 步

打开并读取 `/data/data/com.huplayer.neon/files/V3.lic` 文件。

```asm
libmplayer.so:5EB5ABB0 ADD             R1, PC, R4 ; unk_5F591AD4

libmplayer.so:5EB5ABB4 BL              fopen ; 打开 /data/data/com.huplayer.neon/files/V3.lic

libmplayer.so:5EB5ABB8 SUBS            R5, R0, #0

libmplayer.so:5EB5ABBC BEQ             loc_5EB5BE04

libmplayer.so:5EB5ABC0 LDR             R4, =(aHarrydebug805_ - 0x5EB5ABD8)

libmplayer.so:5EB5ABC4 MOV             R0, R8

libmplayer.so:5EB5ABC8 BL              free

libmplayer.so:5EB5ABCC MOV             R0, #1

libmplayer.so:5EB5ABD0 ADD             R2, PC, R4              ; "harrydebug805.\n"

libmplayer.so:5EB5ABD4 MOV             R1, R6

libmplayer.so:5EB5ABD8 BL              mp_msg

libmplayer.so:5EB5ABDC MOV             R1, R6

libmplayer.so:5EB5ABE0 MOV             R2, #2

libmplayer.so:5EB5ABE4 MOV             R0, R5

libmplayer.so:5EB5ABE8 BL              fseek

libmplayer.so:5EB5ABEC MOV             R0, R5

libmplayer.so:5EB5ABF0 BL              ftell

libmplayer.so:5EB5ABF4 LDR             R7, =(aHarrydebug807_ - 0x5EB5AC5C)

libmplayer.so:5EB5ABF8 STR             R0, [SP,#0xDF0+var_D20]

libmplayer.so:5EB5ABFC BL              malloc

libmplayer.so:5EB5AC00 LDR             R12, =(aHarrydebug806_ - 0x5EB5AC10)

libmplayer.so:5EB5AC04 MOV             R1, R6

libmplayer.so:5EB5AC08 ADD             R2, PC, R12             ; "harrydebug806.\n"

libmplayer.so:5EB5AC0C STR             R0, [SP,#0xDF0+var_D84]

libmplayer.so:5EB5AC10 MOV             R0, #1

libmplayer.so:5EB5AC14 BL              mp_msg

libmplayer.so:5EB5AC18 MOV             R1, R6

libmplayer.so:5EB5AC1C LDR             R2, [SP,#0xDF0+var_D20]

libmplayer.so:5EB5AC20 LDR             R0, [SP,#0xDF0+var_D84]

libmplayer.so:5EB5AC24 BL              memset_0

libmplayer.so:5EB5AC28 MOV             R0, R5

libmplayer.so:5EB5AC2C MOV             R1, R6

libmplayer.so:5EB5AC30 MOV             R2, R6

libmplayer.so:5EB5AC34 BL              fseek

libmplayer.so:5EB5AC38 MOV             R3, R5

libmplayer.so:5EB5AC3C MOV             R1, #1

libmplayer.so:5EB5AC40 LDR             R2, [SP,#0xDF0+var_D20]

libmplayer.so:5EB5AC44 LDR             R0, [SP,#0xDF0+var_D84]

libmplayer.so:5EB5AC48 BL              fread

libmplayer.so:5EB5AC4C MOV             R0, R5

libmplayer.so:5EB5AC50 BL              fclose

libmplayer.so:5EB5AC54 ADD             R2, PC, R7              ; "harrydebug807。
```

### 第 7 步

获取加密视频头文件中的两个 Base64 码，并在 `V3.lic` 中查找是否存在。

`<KID>yJ3pPh4RItrcgckEtqjS+Q==</KID>` 与 `<CID>nZjpa0hCd9vfh5oAt6nf9A==</CID>`：

```asm
libmplayer.so:5EB5ACDC ADD             R1, PC, R1 ; unk_5F5AAFA4

libmplayer.so:5EB5ACE0 BL              sprintf

libmplayer.so:5EB5ACE4 MOV             R1, R4

libmplayer.so:5EB5ACE8 LDR             R0, [SP,#0xDF0+var_D84]

libmplayer.so:5EB5ACEC BL              strstr                  ; 查找 <KID>yJ3pPh4RItrcgckEtqjS+Q==</KID>

libmplayer.so:5EB5ACF0 LDR             R2, =(aHarrydebug808_ - 0x5EB5AD00)

libmplayer.so:5EB5ACF4 MOV             R1, R6

libmplayer.so:5EB5ACF8 ADD             R2, PC, R2              ; "harrydebug808.\n"

libmplayer.so:5EB5ACFC STR             R0, [SP,#0xDF0+var_D1C]

libmplayer.so:5EB5AD00 MOV             R0, #1

libmplayer.so:5EB5AD04 BL              mp_msg

libmplayer.so:5EB5AD08 MOV             R1, R5

libmplayer.so:5EB5AD0C LDR             R0, [SP,#0xDF0+var_D84]

libmplayer.so:5EB5AD10 BL              strstr                  ; 查找 <CID>nZjpa0hCd9vfh5oAt6nf9A==</CID>

libmplayer.so:5EB5AD14 MOV             R7, R0

libmplayer.so:5EB5AD18 MOV             R0, R4

libmplayer.so:5EB5AD1C LDR             R4, =(aHarrydebug809_ - 0x5EB5AD38)

libmplayer.so:5EB5AD20 BL              free

libmplayer.so:5EB5AD24 MOV             R0, R5

libmplayer.so:5EB5AD28 BL              free
```

### 第 8 步

继续查找 `V3.lic` 中的 `<CONTENT>` 与 `</CONTENT>`，并获得 `<CONTENT>` 与 `</CONTENT>` 中间的 Base64 字符。

在 `V3.lic` 获得的 Base64 字符如下：
