
# Crystal Button 1.31A

> 原文：[`www.pediy.com/kssd/pediy05/pediy50329.htm`](https://www.pediy.com/kssd/pediy05/pediy50329.htm)

暴力破解 `Crystal Button` `1.31A`

破解者: `AK47[FCG]`

使用工具：`Trw1.23`、`SuperBPM`、`Fi`、`Hiew`、`PE Explorer 1.8`、`ImportRECv1.4.2+`、`loader`、`PEditor`

软件说明：`Crystal Button` “水晶按钮”是一个用来创建专业品质的用于网络的图形的工具。您可以创建多种三维的玻璃、金属、塑料甚至 XP 样式的按钮。您可以创作导航栏，用不同的鼠标掠过效果来创建按钮——突出显示、按压和其它。“水晶按钮”能轻松创建 3D 图像。您可以修改图像材料和文字，照明，边框类型，形状以及更多！

开工啦！！！！！！！！

## 第一步

用 `FI` 检测显示为 “ASProtect v1.2? Alexey”。好说，请出 `loader` 找到它的入口点：`00432b00`

运行 `Trw1.23` 还有 `SuperBPM`

下断点：`bpx 017f:00432b00`

运行 `Crystal Button` 被拦截，下命令：`suspend` 返回，用 `PEditor` 的 `tasks` 来抓取内存，保存文件为 `DUMPED.EXE`。

下一步进行修复，运行 `ImportRECv1.4.2+`，选取 `Crystal Button.exe` 后在 OEP 中输入

得到的入口点：`00032b00`。点击 `IAT AutoSearch` 再点击 `get imports` 和

`show invalid`。用右键单击被选中的部分用 `trace level1` (`disasm`) 修复，

发现还有一部分没有被修复，用右键菜单中的 `pulgin trace(asprotect 1.2x emul)` 就可以全部修复了。

好!! 点击 `Fix dump` 在对话框中选中刚才得到的 `DUMPED.EXE`，得到 `Dumped\_.exe`。这个就是被脱了壳的 `Crystal Button`，运行试一下没有问题。到此脱壳成功。

## 进行第二步

先看一下它需要修改的地方：

1. 无法保存方案和方案另存为
2. 运行程序有提示注册的对话框
3. 导出的图像带有 `trial` 字样的图片

### 先解决 1. 无法保存方案和方案另存为

经过跟踪来找到这里：

```
:00409D70 64A100000000             mov eax, dword ptr fs:[00000000]
:00409D76 6AFF                     push FFFFFFFF
:00409D78 68CC8B4800              push 00488BCC
:00409D7D 50                       push eax
:00409D7E 64892500000000           mov dword ptr fs:[00000000], esp
:00409D85 81ECA8000000             sub esp, 000000A8
:00409D8B 56                       push esi
:00409D8C 8BF1                     mov esi, ecx
:00409D8E E8E98D0700              call 00482B7C
:00409D93 8B4004                   mov eax, dword ptr [eax+04]
:00409D96 8B80C0000000             mov eax, dword ptr [eax+000000C0]
:00409D9C 8B48F8                   mov ecx, dword ptr [eax-08]
:00409D9F 85C9                     test ecx, ecx
:00409DA1 0F85A6020000             jne 0040A04D————》》》这里是关键
:00409DA7 6A02                     push 00000002
:00409DA9 8D4C2414                 lea ecx, dword ptr [esp+14]
:00409DAD E81EC70100               call 004264D0
:00409DB2 8D4C2410                 lea ecx, dword ptr [esp+10]
:00409DB6 C78424B400000000000000   mov dword ptr [esp+000000B4], 00000000
:00409DC1 E8A45D0600               call 0046FB6A
:00409DC6 83F801                   cmp eax, 00000001
:00409DC9 0F854F020000             jne 0040A01E
:00409DCF 53                       push ebx
:00409DD0 57                       push edi
:00409DD1 E8A68D0700               call 00482B7C
:00409DD6 8B7804                   mov edi, dword ptr [eax+04]
:00409DD9 687CD24B00               push 004BD27C
:00409DDE 8D4C2414                 lea ecx, dword ptr [esp+14]
:00409DE2 E8F2470600               call 0046E5D9
:00409DE7 6874D24B00               push 004BD274
:00409DEC 8D4C2410                 lea ecx, dword ptr [esp+10]
:00409DF0 C68424C000000001         mov byte ptr [esp+000000C0], 01
:00409DF8 E8DC470600               call 0046E5D9
:00409DFD 8B00                     mov eax, dword ptr [eax]
:00409DFF 81C7386E0000             add edi, 00006E38
:00409E05 8BF0                     mov esi, eax
:00409E07 8B07                     mov eax, dword ptr [edi]
```

用 `HIEW` 将 `00409DA1 jne 0040A04D` 改为 `Je` 就可以成功地使用保存方案和方案另存为的功能。

### 解决 2. 运行程序有提示注册的对话框

每次程序运行的时候就会提示注册，现在就是要去掉它：

经跟踪来到：

```
:0046FB6A B8E4B64800               mov eax, 0048B6E4
:0046FB6F E88434FCFF               call 00432FF8
:0046FB74 83EC18                   sub esp, 00000018
:0046FB77 53                       push ebx
:0046FB78 56                       push esi
:0046FB79 8BF1                     mov esi, ecx
:0046FB7B 57                       push edi
:0046FB7C 8965F0                   mov dword ptr [ebp-10], esp
:0046FB7F 8975E4                   mov dword ptr [ebp-1C], esi
:0046FB82 8B4648                   mov eax, dword ptr [esi+48]
:0046FB85 8B7E44                   mov edi, dword ptr [esi+44]
:0046FB88 8945E8                   mov dword ptr [ebp-18], eax
:0046FB8B E8EC2F0100               call 00482B7C
:0046FB90 837E4000                 cmp dword ptr [esi+40], 00000000
:0046FB94 8B580C                   mov ebx, dword ptr [eax+0C]
:0046FB97 741E                     je 0046FBB7------》》》这时是关键
:0046FB99 E8DE2F0100               call 00482B7C
:0046FB9E 8B580C                   mov ebx, dword ptr [eax+0C]
:0046FBA1 6A05                     push 00000005
:0046FBA3 FF7640                   push [esi+40]
:0046FBA6 53                       push ebx
```

将 `0046FB97 je 0046FBB7` 中的 `je` 改为 `jne` 就可以完成去掉提示注册的对话框的任务。

### 下面进行最后一项：解决 3. 导出的图像带有 trial 字样的图片

用 `PE Explorer 1.8` 打开 `Dumped\_.exe`，在资源的位图中的 `292` 项可以看到带有 `trial` 字样的图片。

就是它，程序每次在使用导出的图像功能时，就是将它放在导出的图像上面。

好！咱们就将它删除，一定要注意在删除它之前得先选择“XP 可视方式显示向导”并下一步下一步，一直到完成，然后才将它删除并保存为一个新的文件，我将它取名为：`Button.exe`。

运行 `Button.exe` 看看能不能用，发现问题：在选用“导出的图像”功能时整个程序就时入了死循环。

好说，运行 `TRW`，并下断点：`pbx 0046fb6a`。然后使用“导出的图像”功能被拦截，一直小心跟踪看看它是在哪儿死循环，一直到：

```
:0040B75C(C)

:0040B745 85C9                     test ecx, ecx
:0040B747 7C0D                     jl 0040B756
:0040B749 85D2                     test edx, edx
:0040B74B 7C09                     jl 0040B756
:0040B74D 0FAFC2                   imul eax, edx
:0040B750 0306                     add eax, dword ptr [esi]
:0040B752 C60408FF                 mov byte ptr [eax+ecx], FF

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:0040B747(C), :0040B74B(C)

:0040B756 8B4318                   mov eax, dword ptr [ebx+18]
:0040B759 41                       inc ecx
:0040B75A 3BC8                     cmp ecx, eax
:0040B75C 7CE7                     jl 0040B745————》》》》注意这里

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0040B743(C)

:0040B75E 8B431C                   mov eax, dword ptr [ebx+1C]
:0040B761 42                       inc edx
:0040B762 3BD0                     cmp edx, eax
:0040B764 7CD1                     jl 0040B737————》》》还有这里

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0040B732(C)

:0040B766 8D4D98                   lea ecx, dword ptr [ebp-68]
:0040B769 C645FC01                 mov [ebp-04], 01
:0040B76D E83A710600               call 004728AC
:0040B772 C745D0C40C4900            mov [ebp-30], 00490CC4
:0040B779 8D4DD0                   lea ecx, dword ptr [ebp-30]
:0040B77C C745FC04000000            mov [ebp-04], 00000004
:0040B783 E8E37B0600               call 0047336B
```

跟踪发现问题就在：

`0040B75C jl 0040B745`

和

`0040B764 jl 0040B737`

分别将它们的 `JL` 改为 `JNL` 就可以了。

运行一下完全正常，导出的图像看一下: 那个讨厌的带有 `trial` 字样的图片不见了!!!!!

到此破解完成。由于没有仔细测试，也不知有什么方有错，但有一点可以肯定，导出的文件有那个讨厌的标志了。

`4dd466d0e1e0cc123afbe504da2462ca`
