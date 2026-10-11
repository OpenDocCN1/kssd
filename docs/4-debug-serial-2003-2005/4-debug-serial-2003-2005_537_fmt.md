
# 小小助手 V2.01

> 原文：[`www.pediy.com/kssd/pediy05/pediy50006.htm`](https://www.pediy.com/kssd/pediy05/pediy50006.htm)

### 小小助手 V2.01 的破解过程

作者：`[BCG]`、`[FCG]` 任逍遥

- **软件名称**：`小小助手 V2.01`
- **软件大小**：594 KB
- **软件语言**：简体中文
- **软件类别**：国产软件 / 共享版 / 个人管理
- **应用平台**：`Win95/98/NT/2000`

#### 软件介绍

本软件具有定时提醒、通讯录管理、日记管理三大功能，并设有密码保护，初始密码为：`123`。

1. **定时提醒**：可设置指定的日期、时间提醒您不要忘做某某事，如果该事没有做，还可设置间隔多少分钟重复提醒，并可选择 `WAV`、`MIDI`、以及 `MP3` 格式的文件做为提醒时的音乐。因此，也可以把它作为定时点歌器。日期格式为：`年-月-日` 或 `月-日`，时间格式为：`小时:分钟:秒` 或 `小时:分钟`。要选择 `MP3` 作为提醒音乐，必须安装 `IE5.0` 及以上版本或媒体播放器。
2. **通讯录管理**：设有家庭电话、办公电话、手机、`E-mail`、工作单位等多项内容，模糊查询功能，只需输入姓名、住址或电话前面的几个字或号码，即可查找出相匹配的记录。
3. **日记管理**：可设置日记的背景颜色、字体格式，并可按日期进行查找。
4. **密码更改**：按鼠标右键，选择密码更改，即可进行密码的更改。
5. 如果本软件不能正常运行，请将 `midas.dll` 复制到 `windows\system` 目录下。

**下载地址**：`http://gd.skycn.net/down/little_assistant.zip`

#### 脱壳与分析

用 `FI` 检测到该软件加了壳，用 `Prodump` 将 `"little_assistant.exe"` 脱壳，另存为 `"dd.exe"`，运行 `"dd.exe"`，输入：

- **用户名**：`jack`
- **注册码**：`7878`

运行 `TRW2000`，按 `CTRL+N` 激活调试窗口，输入：

```
bpx hmemcpy
g
```

按`"确定"`后，调试窗口被激活，输入：

```
bc *
pmodule
```

成功进入软件领域，按 `F10` 跟踪：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004F2FD9(C)

|

:004F2FDB 00                    BYTE 0
:004F2FDC 55                    push ebp
:004F2FDD 8BEC                  mov ebp, esp
:004F2FDF 33C9                  xor ecx, ecx
:004F2FE1 51                    push ecx
:004F2FE2 51                    push ecx
:004F2FE3 51                    push ecx
:004F2FE4 51                    push ecx
:004F2FE5 51                    push ecx
:004F2FE6 51                    push ecx
:004F2FE7 51                    push ecx
:004F2FE8 53                    push ebx
:004F2FE9 56                    push esi
:004F2FEA 57                    push edi
:004F2FEB 8945FC                mov dword ptr [ebp-04], eax
:004F2FEE 33C0                  xor eax, eax
:004F2FF0 55                    push ebp
:004F2FF1 68B9314F00            push 004F31B9
:004F2FF6 64FF30                 push dword ptr fs:[eax]
:004F2FF9 648920                 mov dword ptr fs:[eax], esp
:004F2FFC 8D45F4                 lea eax, dword ptr [ebp-0C]
:004F2FFF E8340DF1FF            call 00403D38
:004F3004 8D55F0                 lea edx, dword ptr [ebp-10]
:004F3007 8B45FC                 mov eax, dword ptr [ebp-04]
:004F300A 8B8004050000           mov eax, dword ptr [eax+00000504]
:004F3010 E8A7F7F3FF            call 004327BC
:004F3015 837DF000              cmp dword ptr [ebp-10], 00000000
//用户名不能为空
:004F3019 0F8458010000          je 004F3177
//等于 0 就跳走
:004F301F 8D55F8                 lea edx, dword ptr [ebp-08]
:004F3022 8B45FC                 mov eax, dword ptr [ebp-04]
:004F3025 8B8004050000           mov eax, dword ptr [eax+00000504]
:004F302B E88CF7F3FF            call 004327BC
:004F3030 8B45F8                 mov eax, dword ptr [ebp-08]
//读取用户名长度
:004F3033 E8800FF1FF            call 00403FB8
:004F3038 8BF0                   mov esi, eax
:004F303A 85F6                   test esi, esi
//检查长度是否为 0
:004F303C 7E2A                   jle 004F3068
//小于或等于 0 就跳走
****************************************************************开始计算注册码****************************************************************
:004F303E BF01000000            mov edi, 00000001
//赋值 edi=1
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004F3066(C)
|
:004F3043 8B45F8                 mov eax, dword ptr [ebp-08]
//读取用户名长度
:004F3046 33DB                   xor ebx, ebx
//清 0
:004F3048 8A5C38FF               mov bl, byte ptr [eax+edi-01]
//读取第一个字符的 10 进制 ASCII 码，并将结果赋值给 ebx
:004F304C 83C37B                 add ebx, 0000007B
//将 ebx+123
:004F304F 8D55EC                 lea edx, dword ptr [ebp-14]
:004F3052 8BC3                   mov eax, ebx
//将 ebx 赋值给 eax
:004F3054 E88364F1FF            call 004094DC
:004F3059 8B55EC                 mov edx, dword ptr [ebp-14]
:004F305C 8D45F4                 lea eax, dword ptr [ebp-0C]
:004F305F E85C0FF1FF            call 00403FC0
//这个 CALL 是对用户名中的中文名进行处理，具体就是：如果 ASCII 码+123< 0,就将 edi-1
//（重新读取该字符的 ASCII 码，且第一次读取的是高位 ASCII 码，第二次的是低位 ASCII 码）
//实现代码是：HighAscii=Int((65536 + bl) / 256)
//LowAscii=(65536 + bl) Mod 256
:004F3064 47                     inc edi
//edi+1
:004F3065 4E                     dec esi
//esi-1
:004F3066 75DB                   jne 004F3043
//循环读取 ASCII 码
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004F303C(C)
|
:004F3068 8B45F4                 mov eax, dword ptr [ebp-0C]
:004F306B E8480FF1FF            call 00403FB8
:004F3070 83F80A                 cmp eax, 0000000A
//比较 eax 的长度是否为 10
:004F3073 7E26                   jle 004F309B
//小于或等于 10 则跳走，直接与输入的注册码比较；如果大于就进入下面的注册码处理程序
****************************************************************读取 eax 前 10 位数字****************************************************************
:004F3075 8D45E8                 lea eax, dword ptr [ebp-18]
:004F3078 50                     push eax
:004F3079 B90A000000             mov ecx, 0000000A
:004F307E BA01000000             mov edx, 00000001
:004F3083 8B45F4                 mov eax, dword ptr [ebp-0C]
:004F3086 E83511F1FF            call 004041C0
:004F308B 8B55E8                 mov edx, dword ptr [ebp-18]
:004F308E 8D45F4                 lea eax, dword ptr [ebp-0C]
****************************************************************将上面所得的 10 位数字&121****************************************************************
* Possible StringData Ref from Code Obj ->"121"
                          |
:004F3091 B9D0314F00             mov ecx, 004F31D0
:004F3096 E8690FF1FF            call 00404004
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004F3073(C)
|
:004F309B 8D55E4                 lea edx, dword ptr [ebp-1C]
:004F309E 8B45FC                 mov eax, dword ptr [ebp-04]
:004F30A1 8B8008050000           mov eax, dword ptr [eax+00000508]
:004F30A7 E810F7F3FF            call 004327BC
:004F30AC 8B55E4                 mov edx, dword ptr [ebp-1C]
//错误的注册码
:004F30AF 8B45F4                 mov eax, dword ptr [ebp-0C]
//正确的注册码
:004F30B2 E81110F1FF            call 004040C8
//比较
:004F30B7 0F85AE000000          jne 004F316B
//不等就跳走
```

`OK`，终于获得了正确的注册码：`2292202222121`。输入，按`"确定"`，呵呵，`"您已经注册，感谢使用无超工作室的产品"`。

#### `VB` 注册机（可以支持中文名）

```vb
Private Sub Command1_Click()
Dim a, i As Long
Dim b, c, d, e, f

a = Len(Text1.Text)

If a <= 0 Then GoTo Err1

Label3.Caption = ""

Command2.Enabled = True

For b = 1 To a

c = Mid$(Text1.Text, b, 1)

f = Asc(c)

If f < 0 Then

d = (Int((65536 + f) / 256) + 123) & (((65536 + f) Mod 256) + 123)

Else

d = f + 123

End If

e = e & d

Next b

i = Len(e)

If i <= 10 Then GoTo Err2

Text2.Text = Mid$(e, 1, 10) & "121"

Exit Sub

Err1:

Label3.Caption = "请输入用户名！"

Exit Sub

Err2:

Text2.Text = ""

Text2.Text = e

End Sub

Private Sub Command2_Click()

Clipboard.Clear

Clipboard.SetText (Text2.Text)

Command2.Enabled = False

End Sub
```

`4dd466d0e1e0cc123afbe504da2462ca`
