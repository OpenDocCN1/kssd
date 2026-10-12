
# Windows 密码管理大师 2.1 简单算法分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-161.htm`](https://www.pediy.com/kssd/pediy07/pediy7-161.htm)

软件名称: `Windows 密码管理大师 2.1`

软件性质: `[共享软件]`

文件大小: `3.76M`

最近版本: `2.1`

操作系统: `9X/ME/2K/XP`

语言界面: `中文`

下载地址: `http://download.pchome.net/utility/codeman/19360.html`

软件介绍:

你是否经常出入各大论坛，并且有各种不同的用户名和密码。你是否一个或多个 QQ 号码，

当你登录论坛或 QQ 的时候是否将密码忘记，那么用《Windows 密码管理大师》来管理你的密码吧，它可以帮你管理你在各论坛的注册资料，包括用户名，注册邮箱，注册 ID，注册密码，注册网址等，当你选中一项资料时便可轻易访问该网站。本软件启动时用密码保护，可以防止非法用户查看你的注册资料，非常安全方便。

Cracker: `游上岸的鱼` `[DFCG]` `[FCG]` `[BCG]` `[DCM]`

`2005.04.09`

前言: 这软件的破解很简单只是简单的加法运算,我想高手就没有必要看了,不过要给我顶哦!

开工: 工具 `OllyDbg.V1.10` , `PEID0.93`, `UPX ShellEx`

用 `PEID0.93` 检测为 `UPX 0.89.6 - 1.02 / 1.05 - 1.24 -> Markus & Laszlo`

用 `UPX ShellEx` 顺利脱壳,再用 `PEID0.93` 检测为 `Borland Delphi 4.0 - 5.0`

好了,用 `WIN32DASM` 反汇编,查找字符串 `"注册码不正确,无法注册"` 来到这里

```
0053BC2D    E8 C2CDFFFF      call Windows?005389F4             ; 关键 CALL F7 跟进

0053BC32    84C0             test al,al                         ; 检测 AL 是否为 0

0053BC34    74 09            je short Windows?0053BC3F          ; 不是那么就不跳,那么我们就注册成功

0053BC36    8BC3             mov eax,ebx

0053BC38    E8 03CBFFFF      call Windows?00538740

0053BC3D    5B               pop ebx

0053BC3E    C3               retn

0053BC3F    B8 54BC5300      mov eax,Windows?0053BC54

0053BC44    E8 9B32F2FF      call Windows?0045EEE4

0053BC49    5B               pop ebx

0053BC4A    C3               retn

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0053BC34(C)

|

* Possible StringData Ref from Data Obj -> "注册码不正确,无法注册"

                                  |

:0053BC3F B854BC5300              mov eax, 0053BC54

:0053BC44 E89B32F2FF              call 0045EEE4

:0053BC49 5B                      pop ebx

:0053BC4A C3                      ret
```

`0053BC2D` F7 进去后 F8 往下走,期待机器码的出现来到这里

```
005389FF    56               push esi

00538A00    8945 FC          mov dword ptr ss:[ebp-4],eax

00538A03    33C0             xor eax,eax

00538A05    55               push ebp

00538A06    68 D08A5300      push Windows?00538AD0

00538A0B    64:FF30          push dword ptr fs:[eax]

00538A0E    64:8920          mov dword ptr fs:[eax],esp

00538A11    33C0             xor eax,eax

00538A13    8945 F4          mov dword ptr ss:[ebp-C],eax

00538A16    8D55 F8          lea edx,dword ptr ss:[ebp-8]

00538A19    8B45 FC          mov eax,dword ptr ss:[ebp-4]

00538A1C    8B80 E4030000    mov eax,dword ptr ds:[eax+3E4]

00538A22    E8 7DDBEFFF      call Windows?004365A4

00538A27    8B45 F8          mov eax,dword ptr ss:[ebp-8]

00538A2A    E8 29B7ECFF      call Windows?00404158

00538A2F    8BD8             mov ebx,eax

00538A31    85DB             test ebx,ebx

00538A33    7E 2E            jle short Windows?00538A63         ; 检测机器码是否为空

00538A35    BE 01000000      mov esi,1

00538A3A    8D45 F0          lea eax,dword ptr ss:[ebp-10]

00538A3D    50               push eax

00538A3E    B9 01000000      mov ecx,1

00538A43    8BD6             mov edx,esi

00538A45    8B45 F8          mov eax,dword ptr ss:[ebp-8]

00538A48    E8 13B9ECFF      call Windows?00404360

00538A4D    8B45 F0          mov eax,dword ptr ss:[ebp-10]

00538A50    E8 C7B8ECFF      call Windows?0040431C             ; 取机器码的 ACSII 值

00538A55    8A00             mov al,byte ptr ds:[eax]

00538A57    25 FF000000      and eax,0FF                        ; 清 0 其他

00538A5C    0145 F4          add dword ptr ss:[ebp-C],eax       ; 机器码的 ACSII 值累加,[EBP-C]是典型的 delphi 程序寄存器

00538A5F    46               inc esi

00538A60    4B               dec ebx                              ; 计数器

00538A61    ^ 75 D7         jnz short Windows?00538A3A
```

说明:

`00538A3A` 到 `00538A61` 的循环用 delphi 表示就是

```
For i:= 1 To Length(jiqima) Do
  begin
  temp1:=ord(jiqima[i]);
  sn1:=temp1+sn1;
  end;
```

```
00538A63    8D55 EC          lea edx,dword ptr ss:[ebp-14]

00538A66    8B45 FC          mov eax,dword ptr ss:[ebp-4]

00538A69    8B80 E8030000    mov eax,dword ptr ds:[eax+3E8]

00538A6F    E8 30DBEFFF      call Windows?004365A4

00538A74    8B45 EC          mov eax,dword ptr ss:[ebp-14]

00538A77    E8 0C15EDFF      call Windows?00409F88

00538A7C    8B55 F4          mov edx,dword ptr ss:[ebp-C]

00538A7F    81C2 FC7E1200    add edx,127EFC                     ; SN1 加上常数$127EFC

00538A85    81C2 9FE46400    add edx,Windows?0064E49F           ; 再加上常数$0064E49F

00538A8B    3BC2             cmp eax,edx                          ; 注册码比较

00538A8D    75 19            jnz short Windows?00538AA8          ; 不等就跳转

00538A8F    B3 01            mov bl,1                             ; 等就置 BL=1

00538A91    B8 18D75400      mov eax,Windows?0054D718

00538A96    8B55 F8          mov edx,dword ptr ss:[ebp-8]

00538A99    E8 8EB4ECFF      call Windows?00403F2C

00538A9E    8B45 F4          mov eax,dword ptr ss:[ebp-C]

00538AA1    A3 1CD75400      mov dword ptr ds:[54D71C],eax

00538AA6    EB 02            jmp short Windows?00538AAA

00538AA8    33DB             xor ebx,ebx                          ; 清空

00538AAA    33C0             xor eax,eax                            ; 清空置 AL=0

00538AAC    5A               pop edx

00538AAD    59               pop ecx

00538AAE    59               pop ecx

00538AAF    64:8910          mov dword ptr fs:[eax],edx

00538AB2    68 D78A5300      push Windows?00538AD7

00538AB7    8D45 EC          lea eax,dword ptr ss:[ebp-14]

00538ABA    E8 19B4ECFF      call Windows?00403ED8

00538ABF    8D45 F0          lea eax,dword ptr ss:[ebp-10]

00538AC2    E8 11B4ECFF      call Windows?00403ED8

00538AC7    8D45 F8          lea eax,dword ptr ss:[ebp-8]

00538ACA    E8 09B4ECFF      call Windows?00403ED8

00538ACF    C3               retn
```

算法总结:

1. 依次取机器码的 `ASCII` 码并且相加得到 `SN1`

2. 在把 `SN1` + `$127EFC` + `$0064E49F` 得到 `SN2`

3. 再把 `SN2` 转化为 `10` 进制就是我们需要的注册码

用 `keymaker` 写内存注册机

中断地址: `00538A8B`

中断次数: `1`

第一字节: `3B`

指令长度: `2`

寄存器方式:

`EDX`

十进制

算法注册机源代码 `delphi7.0+winxp SP1` 测试通过

```delphi
procedure TForm1.bsSkinButton1Click(Sender: TObject);

var

jiqima: String;

i: Integer;

sn1: Int64;

temp1: Int64;

begin

  jiqima:=bsSkinEdit1.text;

  sn1:=0;

 For i:= 1 To Length(jiqima) Do

  begin

  temp1:=ord(jiqima[i]);

  sn1:=temp1+sn1;

  end;

  sn1:=sn1+$127EFC+$64E49F;

  bsSkinEdit2.Text:=inttostr(sn1);

  end;

end.
```

小弟第一次在看雪发帖子``给点面子别丢鸡蛋哦``![](img/b16bec246e75299c87cf57e1fb808ac9.jpg)

`4dd466d0e1e0cc123afbe504da2462ca`
