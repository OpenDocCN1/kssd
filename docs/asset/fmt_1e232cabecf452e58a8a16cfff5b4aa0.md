
```
:004189EF 81ECD8000000             sub esp, 000000D8

:004189F5 6A00                     push 00000000

:004189F7 8D8D1CFFFFFF             lea ecx, dword ptr [ebp+FFFFFF1C]

:004189FD E863F6FEFF               call 00408065

//调用格式化对话框（代码在下面）

:00418A02 8365FC00                 and dword ptr [ebp-04], 00000000

:00418A06 8D8D1CFFFFFF             lea ecx, dword ptr [ebp+FFFFFF1C]

:00418A0C E88AF70100               call 0043819B

:00418A11 6A00                     push 00000000

* Possible StringData Ref from Data Obj

//嘻嘻

:00418A13 68806C4600               push 00466C80

* Possible StringData Ref from Data Obj

//不要害怕,只是和你开个小玩笑。本软件不会破坏任...?

:00418A18 68486C4600               push 00466C48

:00418A1D 6A00                     push 00000000

* Reference To: USER32.MessageBoxA, Ord:01BEh

//函数说明：建立消息框窗体

:00418A1F FF1530264500             Call dword ptr [00452630]

:00418A25 834DFCFF                 or dword ptr [ebp-04], FFFFFFFF

:00418A29 8D8D1CFFFFFF             lea ecx, dword ptr [ebp+FFFFFF1C]

:00418A2F E8B6F6FEFF               call 004080EA

:00418A34 8B4DF4                   mov ecx, dword ptr [ebp-0C]

:00418A37 64890D00000000           mov dword ptr fs:[00000000], ecx

:00418A3E C9                       leave

:00418A3F C3                       ret

* Referenced by a CALL at Address:

|:004189FD

|

:00408065 B85FF34400               mov eax, 0044F35F

:0040806A E841D90100               call 004259B0

:0040806F 51                       push ecx

:00408070 56                       push esi

:00408071 57                       push edi

:00408072 FF7508                   push [ebp+08]

:00408075 8BF1                     mov esi, ecx

:00408077 8975F0                   mov dword ptr [ebp-10], esi

* Possible Reference to Dialog: DialogID_00C2

|

:0040807A 68C2000000               push 000000C2

//对话框 ID：194（十六进制 C2 为十进制 194，你可用 Exescope 等资源编辑软件看一下

 主程序资源对话框 194 是不是我们熟悉的 windows 格式化窗口）

:0040807F E862000300               call 004380E6

//调出格式化窗口

:00408084 8365FC00                 and dword ptr [ebp-04], 00000000

:00408088 8D7E5C                   lea edi, dword ptr [esi+5C]

:0040808B 8BCF                     mov ecx, edi

:0040808D E807060300               call 00438699

:00408092 C707747A4500             mov dword ptr [edi], 00457A74

:00408098 8DBE98000000             lea edi, dword ptr [esi+00000098]

:0040809E C645FC01                 mov [ebp-04], 01

:004080A2 8BCF                     mov ecx, edi

:004080A4 E8F0050300               call 00438699

:004080A9 8B4DF4                   mov ecx, dword ptr [ebp-0C]

:004080AC C7078C764500             mov dword ptr [edi], 0045768C

:004080B2 83A6D400000000           and dword ptr [esi+000000D4], 00000000

:004080B9 C706A03D4500             mov dword ptr [esi], 00453DA0

:004080BF 8BC6                     mov eax, esi

:004080C1 5F                       pop edi

:004080C2 5E                       pop esi

:004080C3 64890D00000000           mov dword ptr fs:[00000000], ecx

:004080CA C9                       leave

:004080CB C20400                   ret 0004

//鼠标不停的向“开始”格式化按钮上移动   4dd466d0e1e0cc123afbe504da2462ca
```
