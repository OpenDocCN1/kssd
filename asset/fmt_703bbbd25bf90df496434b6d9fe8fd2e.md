
# Guitar Pro v3.0

> 原文：[`www.pediy.com/kssd/pediy04/kanxue022.htm`](https://www.pediy.com/kssd/pediy04/kanxue022.htm)

下载：`http://211.152.134.220/guitarpp/yu/GuitarPro300.zip`

用`dede`找出“确定”按钮的地址`004CEC8C`，用`trw2k`下断。

输入注册信息，用户名：`LANCELOT[CCG]`，注册码：`12345-67890-434343`。

按确定到这里：

```
\=================================================================================================
004CEC8C  55                         push      ebp
          ...... 略 ......
\* Reference to control TFLicence.Cle1 : TEdit
|
004CECC5  8B83E4020000               mov       eax, [ebx+$02E4]
\* Reference to: controls.TControl.GetText(TControl):System.String;<-------读取注册码的第 1 段
|                                                                                   12345
004CECCB  E81C74F6FF                 call      004360EC
004CECD0  FF75F8                     push      dword ptr [ebp-$08]
004CECD3  8D55F4                     lea       edx, [ebp-$0C]
\* Reference to control TFLicence.Cle2 : TEdit
|
004CECD6  8B83EC020000               mov       eax, [ebx+$02EC]
\* Reference to: controls.TControl.GetText(TControl):System.String;<-------读取注册码的第 2 段
|                                                                                   67890
004CECDC  E80B74F6FF                 call      004360EC
004CECE1  FF75F4                     push      dword ptr [ebp-$0C]
004CECE4  8D55F0                     lea       edx, [ebp-$10]
\* Reference to control TFLicence.Cle3 : TEdit
|
004CECE7  8B83F4020000               mov       eax, [ebx+$02F4]
\* Reference to: controls.TControl.GetText(TControl):System.String;<-------读取注册码的第 3 段
|                                                                                   43434
004CECED  E8FA73F6FF                 call      004360EC
004CECF2  FF75F0                     push      dword ptr [ebp-$10]
004CECF5  8D45FC                     lea       eax, [ebp-$04]
004CECF8  BA03000000                 mov       edx, $00000003
\* Reference to: system.@LStrCatN;<----------------------------------------连接在一起
|                                                                                   123456789043434
004CECFD  E8DA52F3FF                 call      00403FDC
004CED02  8B45FC                     mov       eax, [ebp-$04]
004CED05  50                         push      eax
004CED06  8D55F8                     lea       edx, [ebp-$08]
\* Reference to control TFLicence.Nom : TLabel
|
004CED09  8B83CC020000               mov       eax, [ebx+$02CC]
\* Reference to: controls.TControl.GetText(TControl):System.String;<--------读取用户名
|
004CED0F  E8D873F6FF                 call      004360EC
004CED14  8B45F8                     mov       eax, [ebp-$08]
004CED17  B92C010000                 mov       ecx, $0000012C
004CED1C  5A                         pop       edx
|
004CED1D  E8EECC0800                 call      0055BA10<-------------------------核心计算，跟进去
004CED22  84C0                       test      al, al
004CED24  0F8460010000               jz        004CEE8A
\===============================================================================================

0055BA10  55                         push      ebp
          ...... 略 ......
\* Reference to: system.@LStrLen:Integer;
|   or: system.@DynArrayLength;
|   or: system.DynArraySize(Pointer):Integer;
|
0055BA69  E8AE84EAFF                 call      00403F1C
0055BA6E  8BF0                       mov       esi, eax
0055BA70  85F6                       test      esi, esi
0055BA72  7E21                       jle       0055BA95
0055BA74  B901000000                 mov       ecx, $00000001
0055BA79  8B45FC                     mov       eax, [ebp-$04]
0055BA7C  0FB64408FF                 movzx     eax, byte ptr [eax+ecx-$01]<-------用户名的第一个字 L==0x4c
0055BA81  F7E9                       imul      ecx<-------------------------------乘上序号
0055BA83  03F8                       add       edi, eax<--------------------------和加上 edi 中的值
0055BA85  8BC7                       mov       eax, edi
0055BA87  BFE8030000                 mov       edi, $000003E8
0055BA8C  99                         cdq
0055BA8D  F7FF                       idiv      edi<-------------------------------和除去 0x3e8
0055BA8F  8BFA                       mov       edi, edx<--------------------------余数放到 edi
0055BA91  41                         inc       ecx
0055BA92  4E                         dec       esi
0055BA93  75E4                       jnz       0055BA79<--------------------------循环计算，用户名的 13 个字母
0055BA95  85FF                       test      edi, edi
0055BA97  7505                       jnz       0055BA9E
0055BA99  BF01000000                 mov       edi, $00000001
0055BA9E  8B45F4                     mov       eax, [ebp-$0C]<---------------------0x12c
0055BAA1  2DDC000000                 sub       eax, $000000DC<---------------------0x12c-0xdc==0x50
0055BAA6  03C7                       add       eax, edi<---------------------------0x50+0x15==0x65
0055BAA8  40                         inc       eax<--------------------------------0x65+0x1==0x66
0055BAA9  B9E8030000                 mov       ecx, $000003E8
0055BAAE  99                         cdq
0055BAAF  F7F9                       idiv      ecx
0055BAB1  8BFA                       mov       edi, edx<-------------------------- 0x66
0055BAB3  8D45F0                     lea       eax, [ebp-$10]
0055BAB6  8B55F8                     mov       edx, [ebp-$08]
0055BAB9  8A520A                     mov       dl, byte ptr [edx+$0A]<--------------注册码的第 11 位
\* Reference to: system.@LStrFromChar(String;Char);
|   or: system.@LStrFromWChar(String;WideChar);
|   or: system.@WStrFromChar(WideString;Char);
|   or: system.@WStrFromWChar(WideString;WideChar);
|
0055BABC  E88383EAFF                 call      00403E44
0055BAC1  8B45F0                     mov       eax, [ebp-$10]
0055BAC4  50                         push      eax
0055BAC5  8D45EC                     lea       eax, [ebp-$14]
0055BAC8  8B55F8                     mov       edx, [ebp-$08]
0055BACB  8A12                       mov       dl, byte ptr [edx]<--------------注册码的第 1 位
\* Reference to: system.@LStrFromChar(String;Char);
|   or: system.@LStrFromWChar(String;WideChar);
|   or: system.@WStrFromChar(WideString;Char);
|   or: system.@WStrFromWChar(WideString;WideChar);
|
0055BACD  E87283EAFF                 call      00403E44
0055BAD2  8B45EC                     mov       eax, [ebp-$14]
\* Reference to: sysutils.StrToInt(System.AnsiString):System.Integer;
|
0055BAD5  E8CAD4EAFF                 call      00408FA4
0055BADA  8BF0                       mov       esi, eax
0055BADC  8D45EC                     lea       eax, [ebp-$14]
0055BADF  8B55F8                     mov       edx, [ebp-$08]
0055BAE2  8A5201                     mov       dl, byte ptr [edx+$01]<--------------注册码的第 2 位
\* Reference to: system.@LStrFromChar(String;Char);
|   or: system.@LStrFromWChar(String;WideChar);
|   or: system.@WStrFromChar(WideString;Char);
|   or: system.@WStrFromWChar(WideString;WideChar);
|
0055BAE5  E85A83EAFF                 call      00403E44
0055BAEA  8B45EC                     mov       eax, [ebp-$14]
\* Reference to: sysutils.StrToInt(System.AnsiString):System.Integer;
|
0055BAED  E8B2D4EAFF                 call      00408FA4
0055BAF2  03F0                       add       esi, eax<--------------注册码的第 1，2 位之和
0055BAF4  8BC6                       mov       eax, esi
0055BAF6  F7EF                       imul      edi<-------------------乘上 0x66
0055BAF8  B90A000000                 mov       ecx, $0000000A<--------除与 0xa
0055BAFD  99                         cdq
0055BAFE  F7F9                       idiv      ecx
0055BB00  8BC2                       mov       eax, edx<-------------余数放在 eax
0055BB02  8D55EC                     lea       edx, [ebp-$14]
\* Reference to: sysutils.IntToStr(System.Integer):System.AnsiString;overload;
|
0055BB05  E836D4EAFF                 call      00408F40
0055BB0A  8B55EC                     mov       edx, [ebp-$14]
```
