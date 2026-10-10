
# R 命令

> 原文：[`www.pediy.com/kssd/pediy03/forum738.htm`](https://www.pediy.com/kssd/pediy03/forum738.htm)

## 从轻松试卷 v4.03 的破解看 r fl z 的妙用

**作者**：PaulYoung

**E-mail**：paulyoung@yeah.net

**软件简介**：顾名思义啦。

**URL**：`http://easypaper.yeah.net` （最新版本是 `v4.04`，相信也差不了多少）

**破解工具**：`SOFT-ICE V4.0.5`

二○○一年三月二十四日凌晨

==============================================================================================================

首先感谢★小牧童★大师的指点，才能使我能用 `W32DASM V8.93` 反汇编轻松试卷。虽然我是用动态跟踪破解的，但要写破解心得，首选还是 `w32DASM`。

==============================================================================================================

好了，首先自我介绍一番，我是名初学者，学了个把两个月破解吧，因此，汇编知识自然是鸟之又鸟的了，但熟能生巧嘛，而且我又是看雪破解论坛的常客，一来二往，真的学了不少东西。

最近我学了一招 `r fl z`，功能大概跟 `a` 指令差不多吧，不过我觉得它按起来顺手多了。`TRW2000` 和 `SOFT-ICE` 均可使用，灵活运用，对破解能起到事半功倍的作用。

下面，我就以轻松试卷为例，看看 `r fl z` 有何妙处吧。

1. 运行轻松试卷，在注册框的学校/单位、用户名、注册号处填写，随你便啦。

2. `Ctrl+D` 激活 `SOFT-ICE`，下 `BPX HMEMCPY`。`F5` 退出，点击确定，被拦截。

3. 下 `BD *`，中断所有断点。

4. 按 12 次 `F12` （因为 13 次出错），开始看：

```
:004019A5 8D4DF8                 lea ecx, dword ptr [ebp-08]       //程序入口
:004019A8 51                   push ecx
:004019A9 8BD7                 mov edx, edi
:004019AB 8D45F4               lea eax, dword ptr [ebp-0C]
:004019AE E8D5721200           call 00528C88
:004019B3 FF431C               inc [ebx+1C]
:004019B6 8D55F4               lea edx, dword ptr [ebp-0C]
:004019B9 58                   pop eax
:004019BA E80D751200           call 00528ECC
:004019BF 50                   push eax
:004019C0 FF4B1C               dec [ebx+1C]
:004019C3 8D45F4               lea eax, dword ptr [ebp-0C]
:004019C6 BA02000000           mov edx, 00000002
:004019CB E818741200           call 00528DE8
:004019D0 FF4B1C               dec [ebx+1C]
:004019D3 8D45F8               lea eax, dword ptr [ebp-08]
:004019D6 BA02000000           mov edx, 00000002
:004019DB E808741200           call 00528DE8                   //学校名、单位名不能为空（我掉头就溜……）
:004019E0 59                   pop ecx
:004019E1 84C9                 test cl, cl
:004019E3 745F                 je 00401A44                  //下 r fl z ,F5 退出，可看到上面提示
:004019E5 66C743101400         mov [ebx+10], 0014
:004019EB 8D5701               lea edx, dword ptr [edi+01]
:004019EE 8D45F0               lea eax, dword ptr [ebp-10]
:004019F1 E892721200           call 00528C88
:004019F6 FF431C               inc [ebx+1C]
:004019F9 8D45F0               lea eax, dword ptr [ebp-10]
:004019FC 33D2                 xor edx, edx
:004019FE 8955EC               mov dword ptr [ebp-14], edx
:00401A01 8D55EC               lea edx, dword ptr [ebp-14]
:00401A04 FF431C               inc [ebx+1C]
:00401A07 E8E8A90100           call 0041C3F4
:00401A0C 8D45EC               lea eax, dword ptr [ebp-14]
:00401A0F 8B00                 mov eax, dword ptr [eax]
:00401A11 E886420100           call 00415C9C
:00401A16 FF4B1C               dec [ebx+1C]
:00401A19 8D45EC               lea eax, dword ptr [ebp-14]
:00401A1C BA02000000           mov edx, 00000002
:00401A21 E8C2731200           call 00528DE8
:00401A26 FF4B1C               dec [ebx+1C]
:00401A29 8D45F0               lea eax, dword ptr [ebp-10]
:00401A2C BA02000000           mov edx, 00000002
:00401A31 E8B2731200           call 00528DE8
:00401A36 8B0B                 mov ecx, dword ptr [ebx]
:00401A38 64890D00000000       mov dword ptr fs:[00000000], ecx
:00401A3F E972030000           jmp 00401DB6

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004019E3(C)
|

:00401A44 66C743102000         mov [ebx+10], 0020
:00401A4A 33C0                 xor eax, eax
:00401A4C 8945E8               mov dword ptr [ebp-18], eax
:00401A4F 8D55E8               lea edx, dword ptr [ebp-18]
:00401A52 FF431C               inc [ebx+1C]
:00401A55 8B86DC020000         mov eax, dword ptr [esi+000002DC]
:00401A5B E8F8A60D00           call 004DC158
:00401A60 8D55E8               lea edx, dword ptr [ebp-18]
:00401A63 52                   push edx
:00401A64 8D5711               lea edx, dword ptr [edi+11]
:00401A67 8D45E4               lea eax, dword ptr [ebp-1C]
:00401A6A E819721200           call 00528C88
:00401A6F FF431C               inc [ebx+1C]
:00401A72 8D55E4               lea edx, dword ptr [ebp-1C]
:00401A75 58                   pop eax
:00401A76 E851741200           call 00528ECC
:00401A7B 50                   push eax
:00401A7C FF4B1C               dec [ebx+1C]
:00401A7F 8D45E4               lea eax, dword ptr [ebp-1C]
:00401A82 BA02000000           mov edx, 00000002
:00401A87 E85C731200           call 00528DE8
:00401A8C FF4B1C               dec [ebx+1C]
:00401A8F 8D45E8               lea eax, dword ptr [ebp-18]
:00401A92 BA02000000           mov edx, 00000002
:00401A97 E84C731200           call 00528DE8                   //用户名不能为空（傻子才会跟进去）
:00401A9C 59                   pop ecx
:00401A9D 84C9                 test cl, cl
:00401A9F 745F                 je 00401B00                   //下 r fl z ,F5 退出，可看到上面提示
:00401AA1 66C743102C00         mov [ebx+10], 002C
:00401AA7 8D5712               lea edx, dword ptr [edi+12]
:00401AAA 8D45E0               lea eax, dword ptr [ebp-20]
:00401AAD E8D6711200           call 00528C88
:00401AB2 FF431C               inc [ebx+1C]
:00401AB5 8D45E0               lea eax, dword ptr [ebp-20]
:00401AB8 33D2                 xor edx, edx
:00401ABA 8955DC               mov dword ptr [ebp-24], edx
:00401ABD 8D55DC               lea edx, dword ptr [ebp-24]
:00401AC0 FF431C               inc [ebx+1C]
:00401AC3 E82CA90100           call 0041C3F4
:00401AC8 8D45DC               lea eax, dword ptr [ebp-24]
:00401ACB 8B00                 mov eax, dword ptr [eax]
:00401ACD E8CA410100           call 00415C9C
:00401AD2 FF4B1C               dec [ebx+1C]
:00401AD5 8D45DC               lea eax, dword ptr [ebp-24]
:00401AD8 BA02000000           mov edx, 00000002
:00401ADD E806731200           call 00528DE8
:00401AE2 FF4B1C               dec [ebx+1C]
:00401AE5 8D45E0               lea eax, dword ptr [ebp-20]
:00401AE8 BA02000000           mov edx, 00000002
:00401AED E8F6721200           call 00528DE8
:00401AF2 8B0B                 mov ecx, dword ptr [ebx]
:00401AF4 64890D00000000       mov dword ptr fs:[00000000], ecx
:00401AFB E9B6020000           jmp 00401DB6

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401A9F(C)
|

:00401B00 66C743103800         mov [ebx+10], 0038
:00401B06 33C0                 xor eax, eax
:00401B08 8945D8               mov dword ptr [ebp-28], eax
:00401B0B 8D55D8               lea edx, dword ptr [ebp-28]
:00401B0E FF431C               inc [ebx+1C]
:00401B11 8B86E0020000         mov eax, dword ptr [esi+000002E0]
:00401B17 E83CA60D00           call 004DC158
:00401B1C 8D55D8               lea edx, dword ptr [ebp-28]
:00401B1F 52                   push edx
:00401B20 8D5724               lea edx, dword ptr [edi+24]
:00401B23 8D45D4               lea eax, dword ptr [ebp-2C]
:00401B26 E85D711200           call 00528C88
:00401B2B FF431C               inc [ebx+1C]
:00401B2E 8D55D4               lea edx, dword ptr [ebp-2C]
```
