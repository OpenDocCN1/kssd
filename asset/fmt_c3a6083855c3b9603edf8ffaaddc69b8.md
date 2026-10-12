
:004436F7 6A03                    push 00000003

:004436F9 FFD7                    call edi

:004436FB 83C410                  add esp, 00000010

:004436FE 6639B5E0FEFFFF          cmp word ptr [ebp+FFFFFEE0], si

:00443705 0F85DA050000            jne 00443CE5

====>跳则 OVER！爆破点 ③

…… …… 省 略 …… ……

* Reference To: MSVBVM60.rtcMsgBox, Ord:0253h

|

:00443A49 FF1590104000            Call dword ptr [00401090]

====>呵呵，胜利女神！

————————————————————

### 二、程序重启后又变回未注册了，呵呵，启动时还有验证呀。

用 `RegMon` 知道程序把注册码保存在注册表中，所以在反汇编代码里搜索 `sserialnumber` 下断。

* Possible StringData Ref from Code Obj ->"sserialnumber"

|

:00435A87 68188F4000              push 00408F18

:00435A8C 8D954CFFFFFF            lea edx, dword ptr [ebp+FFFFFF4C]

:00435A92 52                      push edx

* Reference To: MSVBVM60.__vbaStrToAnsi, Ord:0000h

|

:00435A93 FF15D0114000            Call dword ptr [004011D0]

…… …… 省 略 …… ……

* Reference To: MSVBVM60.rtcLeftCharVar, Ord:0269h

|

:00435FCB FF15E8114000            Call dword ptr [004011E8]

====>取 试炼码 前 6 位：135724

…… …… 省 略 …… ……

:0043608D FF512C                  call [ecx+2C]

====>计算 135724 的 MD5 值=4DDCB6075647F0A96811CF0ACB291A93

…… …… 省 略 …… ……

:004360CE 8B955CFFFFFF            mov edx, dword ptr [ebp+FFFFFF5C]

====>EDX=4DDCB6075647F0A96811CF0ACB291A93

:004360D4 52                      push edx

* Possible StringData Ref from Code Obj ->"BB4E92EC6FBA2F7B93CF192D7CB9368DB"

|

:004360D5 68E4904000              push 004090E4

====>004090E4=BB4E92EC6FBA2F7B93CF192D7CB9368DB

* Reference To: MSVBVM60.__vbaStrCmp, Ord:0000h

|

:004360DA FF15E8104000            Call dword ptr [004010E8]

====>比较 CALL！ ④

:004360E0 F7D8                    neg eax

:004360E2 1BC0                    sbb eax, eax

:004360E4 F7D8                    neg eax

:004360E6 F7D8                    neg eax

:004360E8 6689858CFEFFFF          mov word ptr [ebp+FFFFFE8C], ax

:004360EF 8D8D5CFFFFFF            lea ecx, dword ptr [ebp+FFFFFF5C]

* Reference To: MSVBVM60.__vbaFreeStr, Ord:0000h

|

:004360F5 FF1528124000            Call dword ptr [00401228]

:004360FB 0FBF858CFEFFFF          movsx eax, word ptr [ebp+FFFFFE8C]

:00436102 85C0                    test eax, eax

:00436104 0F8480010000            je 0043628A

====>不跳则 OVER！ 爆破点 ④

…… …… 省 略 …… ……

:00436332 8BD0                    mov edx, eax

====>EDX=fly                   用户名

:00436334 8D8D58FFFFFF            lea ecx, dword ptr [ebp+FFFFFF58]

* Reference To: MSVBVM60.__vbaStrMove, Ord:0000h

|

:0043633A FF15F0114000            Call dword ptr [004011F0]

:00436340 8D8D54FFFFFF            lea ecx, dword ptr [ebp+FFFFFF54]

:00436346 51                      push ecx

:00436347 8D9558FFFFFF            lea edx, dword ptr [ebp+FFFFFF58]

:0043634D 52                      push edx

:0043634E 8B8594FEFFFF            mov eax, dword ptr [ebp+FFFFFE94]

:00436354 8B08                    mov ecx, dword ptr [eax]

:00436356 8B9594FEFFFF            mov edx, dword ptr [ebp+FFFFFE94]

:0043635C 52                      push edx

:0043635D FF512C                  call [ecx+2C]

====>计算 fly 的 MD5 值=AF17BC3B4A86A96A0F053A7E5F7C18BA

…… …… 省 略 …… ……

* Reference To: MSVBVM60.rtcMidCharVar, Ord:0278h

|

:0043644A FF15C8104000            Call dword ptr [004010C8]

====>取试炼码 135724689012 的第 7 位至第 10 位：6890

…… …… 省 略 …… ……

* Reference To: MSVBVM60.rtcMidCharVar, Ord:0278h

|

:004364A1 FF15C8104000            Call dword ptr [004010C8]

====>取 AF17BC3B4A86A96A0F053A7E5F7C18BA 的第 16 至 19 位：A0F0

:004364A7 8D8D20FFFFFF            lea ecx, dword ptr [ebp+FFFFFF20]

:004364AD 51                      push ecx

:004364AE 8D9500FFFFFF            lea edx, dword ptr [ebp+FFFFFF00]

:004364B4 52                      push edx

* Reference To: MSVBVM60.__vbaVarTstNe, Ord:0000h

|

:004364B5 FF15B8114000            Call dword ptr [004011B8]

====>比较 CALL！ ⑤

…… …… 省 略 …… ……

:004364FC 85C9                    test ecx, ecx

====> 爆破点  ⑤

:004364FE 0F8480010000            je 00436684

====>不跳则 OVER！

…… …… 省 略 …… ……

* Reference To: MSVBVM60.rtcRightCharVar, Ord:026Bh

|

:004366AE FF1500124000            Call dword ptr [00401200]

====>取 AF17BC3B4A86A96A0F053A7E5F7C18BA 的最后 8 位：5F7C18BA

…… …… 省 略 …… ……

:00436723 FF522C                  call [edx+2C]

====>计算 5F7C18BA 的 MD5 值=3B300D0A6C82BE9E41C9D78365E9E442

…… …… 省 略 …… ……

* Reference To: MSVBVM60.rtcMidCharVar, Ord:0278h

|

:00436803 FF15C8104000            Call dword ptr [004010C8]

====>取试炼码 135724689012 的最后 2 位字符：12

…… …… 省 略 …… ……

* Reference To: MSVBVM60.rtcMidCharVar, Ord:0278h

|

:0043685A FF15C8104000            Call dword ptr [004010C8]

====>取 3B300D0A6C82BE9E41C9D78365E9E442 的 23、24 位：83

:00436860 8D8520FFFFFF            lea eax, dword ptr [ebp+FFFFFF20]

:00436866 50                      push eax

:00436867 8D8D00FFFFFF            lea ecx, dword ptr [ebp+FFFFFF00]

:0043686D 51                      push ecx

* Reference To: MSVBVM60.__vbaVarTstNe, Ord:0000h

|

:0043686E FF15B8114000            Call dword ptr [004011B8]

====>比较 CALL！ ⑥

…… …… 省 略 …… ……

:004368B5 85C0                    test eax, eax

====> 爆破点  ⑥

:004368B7 0F8480010000            je 00436A3D

====>不跳则 OVER！

…… …… 省 略 …… ……

:00436996 FF91B0020000            call dword ptr [ecx+000002B0]

====>要求注册的 NAG！

:00436A14 E893FAFCFF              call 004064AC

====>删除注册表中的用户名

—————————————————————————————————

### 【算 法  总 结】：

1. 前 6 位注册码是作者预置的，`MD5` 值是：`BB4E92EC6FBA2F7B93CF192D7CB9368DB`

   呵呵，穷举？爆破？付款？等待密钥泄露？恐怕只有这几种办法啦

2. 计算用户名 `fly` 的 `MD5` 值=`AF17BC3B4A86A96A0F053A7E5F7C18BA`

   取其第 16 位至 19 位：`A0F0` 作为注册码的 第 7、8、9、10 位

3. 取 `AF17BC3B4A86A96A0F053A7E5F7C18BA` 的最后 8 位：`5F7C18BA`

   计算 `5F7C18BA` 的 `MD5` 值=`3B300D0A6C82BE9E41C9D78365E9E442`

   取其第 23、24 位：`83` 作为注册码最后 2 位

—————————————————————————————————

### 【完 美  爆 破】：

1. `00443480 8B957CFFFFFF            mov edx, dword ptr [ebp+FFFFFF7C]`
   改为：`BAE490400090                mov edx, 004090E4`

2. `004435BB 0F8524070000            jne 00443CE5`
   改为：`909090909090                NOP 掉`

3. `00443705 0F85DA050000            jne 00443CE5`
   改为：`909090909090                NOP 掉`
