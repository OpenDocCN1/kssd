
# ThemeEngine 2.9.1

> 原文：[`www.pediy.com/kssd/pediy05/pediy50171.htm`](https://www.pediy.com/kssd/pediy05/pediy50171.htm)

第一次尝试破解 `delphi` 控件，走了些弯路，现将一些心得告诉大家。

`delphi` 控件破解篇----`ThemeEngine` 2.9.1

主页；`http://www.ksdev.com`

介绍：一个 `delphi` 控件，可以使 `delphi` 编译的程序具 `windows` XP 界面风格

限制：`trial` 版本生成的 `delphi` 程序有 NAG 窗；`delphi` 启动时也有 NAG 窗

工具：`delphi`，`wdasm`，`IDA`，`dcucu`，`ultraedit`

说明：以下操作对于 `ThemeEngine` 2.9.1 for `delphi`5 版本

### 步骤 1

`bpl` 文件在 `delphi` 中相当于 `dll` 文件。`delphi` 启动时也用 `loadlibrarya` 载入

`delphi` 启动时出现 NAG 窗与 `bpl` 文件有关

用 `IDA` 反编译 `lib.D5` 目录下文件 `ThemeEngine5.bpl`，利用 `search` 功能，查找 NAG 窗中的提示语句，以定位

`public @Ksthemeforms@initialization$qqrv`

```
|
CODE:0040C058       sub    ds:dword_445108, 1
CODE:0040C05F       jnb    short locret_40C066                //jnb => jmp
CODE:0040C061       call   @Ksthemeversion@ShowVersion2$qqrv  //NAG 窗
```

### 步骤 2

`dcu`、`pas` 等文件保存了事件代码。`delphi` 编译时将调用的 `dcu`、`pas` 等文件编译后生成程序。因此编译后程序出现 NAG 窗应以 `dcu` 文件为目标(why? 商业软件没有提供 `pas` 源代码)。

用 `ultraedit` 打开 `lib.D5` 目录下额 `dcu` 文件，还是利用 NAG 窗提示寻找；一番分析后明确 NAG 窗来自于 `KsThemeVersion.dcu`。

用 `DCUCU` 反编译

```
procedure ShowVersion;

begin
000: ?    |55                   | PUSH EBP
001:       |8B EC                | MOV EBP,ESP
...
030: 笜... |B8 98 00 00 00       | MOV EAX,ShowVersion{0x20}+$00000098
...
046: ?... |E8 00 00 00 00       | CALL MessageDlg{0x10} //NAG
```

为了阻止 NAG 窗出现，

将 `000: ?    |55                   | PUSH EBP` 变为 `000: ?    |C3                   | RET NEAR`

同样，对于

```
procedure ShowVersion2;

begin
000: ?    |55                   | PUSH EBP
```

也是变为

`000: ?    |C3                   | RET NEAR`

上述修改用 `ultraedit` 完成后保存。

再用 `delphi` 编译、测试一下，OK！ 4dd466d0e1e0cc123afbe504da2462ca
