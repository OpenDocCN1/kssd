
# Registering Trojan Remover 4.2.1

(14千字)

1、下载地址：`http://www.simplysup.com/tremover/`

2、大小：1430Kb

3、破解工具：`SoftICE 4.05`、`TRW2000`、`Win32dasm 8.93`、`Hiew 6.40`

4、软件简介：是一款查杀木马的相当流行的工具，这是最新版本。

主程序用`ASPack`压缩过，程序是用`Dephi 3.0`写成，主程序只有197K，脱壳后居然有1.08Mb。

用`Win32dasm`反汇编后看到了一组：`"Temporary registration Code "`

```
* Possible StringData Ref from Code Obj ->"419246"
* Possible StringData Ref from Code Obj ->"387192"
* Possible StringData Ref from Code Obj ->"388028"
* Possible StringData Ref from Code Obj ->"422199"
```

试了一下还是可以的，出现了这样的提示窗：`"Temporary registration has been applied successfully. This registration will be valid for the next 72 hours."`

只能用72个小时，我#%^&$#^，也太小气了。于是决定操刀解之而后快:)

5、破解过程：运行程序输入

```
Username: CoolBob
Organisation: China Cracker Group
Serial Number: 26313818 (随机产生的)
Registration Key: 12345
```

一开始用`TRW2000`，发现上当。该程序注册验证时用到`float`运算。所以，换`SoftICE`上场，下`BPX hmemcpy`，F5跳出来后，点OK按钮被SoftICE拦截，12次F12来到这里---->

```
Copyright ?1999-2001 Simply Super Software

* Reference To: VCL30.Controls.TControl.GetText@23EDC2EF, Ord:0000h
            |
:0044591A E811C2FBFF              Call 00401B30
:0044591F 8B45F0 mov eax, dword ptr [ebp-10] <----Here we come
:00445922 50                      push eax  <----Save Serial Number
:00445923 8D55EC lea edx, dword ptr [ebp-14]
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004458BF(C)
|
:00445926 8B45FC mov eax, dword ptr [ebp-04]
:00445929 8B80E8010000            mov eax, dword ptr [eax+000001E8]
* Reference To: VCL30.Controls.TControl.GetText@23EDC2EF, Ord:0000h
            |
:0044592F E8FCC1FBFF              Call 00401B30
:00445934 8B55EC mov edx, dword ptr [ebp-14] <----Get Name
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004458D4(C)
|
:00445937 A1B86B4C00              mov eax, dword ptr [004C6BB8]
:0044593C 8B00                    mov eax, dword ptr [eax]
:0044593E 59                      pop ecx
:0044593F E80C3B0700              call 004B9450
:00445944 8D55F0 lea edx, dword ptr [ebp-10]
:00445947 8B45FC mov eax, dword ptr [ebp-04]
:0044594A 8B8008020000            mov eax, dword ptr [eax+00000208]
* Reference To: VCL30.Controls.TControl.GetText@23EDC2EF, Ord:0000h
            |
:00445950 E8DBC1FBFF              Call 00401B30
:00445955 8B55F0 mov edx, dword ptr [ebp-10] <----Real Registration Key
:00445958 8B45F8 mov eax, dword ptr [ebp-08] <----12345
* Reference To: VCL30.System.@LStrCmp@51F89FF7, Ord:0000h
            |
:0044595B E848B8FBFF              Call 004011A8              <---Hmmm,What about this call?
:00445960 0F856B020000            jne 00445BD1                <---If jump,bad guy
:00445966 A19C6B4C00              mov eax, dword ptr [004C6B9C]
:0044596B C60001 mov byte ptr [eax], 01
:0044596E A1D46B4C00              mov eax, dword ptr [004C6BD4]
:00445973 C60000 mov byte ptr [eax], 00
:00445976 B201                    mov dl, 01
:00445978 A11C8B4E00              mov eax, dword ptr [004E8B1C]
* Reference To: VCL30.Registry.TRegistry.Create@23EDC2EF, Ord:0000h
            |
:0044597D E83EC6FBFF              Call 00401FC0
:00445982 8945F4 mov dword ptr [ebp-0C], eax
:00445985 BA02000080              mov edx, 80000002
:0044598A 8B45F4 mov eax, dword ptr [ebp-0C]
* Reference To: VCL30.Registry.TRegistry.SetRootKey@23EDC2EF, Ord:0000h
            |
:0044598D E83EC6FBFF              Call 00401FD0
:00445992 B101                    mov cl, 01
* Possible StringData Ref from Code Obj ->"SOFTWARE\Simply Super Software\Trojan
"
                  ->"Remover\User"
            |
:00445994 BA385F4400              mov edx, 00445F38
:00445999 8B45F4 mov eax, dword ptr [ebp-0C]
* Reference To: VCL30.Registry.TRegistry.OpenKey@23EDC2EF, Ord:0000h
            |
:0044599C E837C6FBFF              Call 00401FD8
:004459A1 84C0 test al, al
:004459A3 0F84DC000000            je 00445A85
:004459A9 33C0                    xor eax, eax
:004459AB 55                      push ebp
:004459AC 685D5A4400              push 00445A5D
:004459B1 64FF30 push dword ptr fs:[eax]
:004459B4 648920 mov dword ptr fs:[eax], esp
:004459B7 8D55F0 lea edx, dword ptr [ebp-10]
:004459BA 8B45FC mov eax, dword ptr [ebp-04]
:004459BD 8B80E8010000            mov eax, dword ptr [eax+000001E8]
```

如果只是简单的找注册码，到这里应该结束了，但作为Cracker我们应该有一种一追到底的精神，就像追MM一样:)要追到她们感动为止，否则，不要轻易放弃。情圣守则第一条。

又扯远了^O^

让我们来分析一下注册码生成过程，在这里的时候：

```
:00445922 50              push eax    <----Save Serial Number
```

下`BPR eax eax+7 r`，按一下F5。

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004B943B(C)
|
:004B94A6 8B55FC mov edx, dword ptr [ebp-04]
:004B94A9 8A543AFF                mov dl, byte ptr [edx+edi-01]      <----Here we come
* Reference To: VCL30.System.@LStrFromChar@001FB870, Ord:0000h
            |
:004B94AD E8B67CF4FF              Call 00401168
:004B94B2 8B45E8 mov eax, dword ptr [ebp-18]
* Reference To: VCL30.SysUtils.StrToInt@0F6FDFF6, Ord:0000h
  <---StrToInt(Serial Number)
            |
:004B94B5 E8EE80F4FF              Call 004015A8
:004B94BA 83F802                  cmp eax, 00000002            <---if SN[i]>=2 jump
:004B94BD 7D44                    jge 004B9503
:004B94BF 8D45E8 lea eax, dword ptr [ebp-18]
:004B94C2 8B55FC mov edx, dword ptr [ebp-04]
:004B94C5 8A543AFF                mov dl, byte ptr [edx+edi-01]
* Reference To: VCL30.System.@LStrFromChar@001FB870, Ord:0000h
            |
:004B94C9 E89A7CF4FF              Call 00401168
:004B94CE 8B45E8 mov eax, dword ptr [ebp-18]
* Reference To: VCL30.SysUtils.StrToInt@0F6FDFF6, Ord:0000h    <----StrToInt(SN[i])
            |
:004B94D1 E8D280F4FF              Call 004015A8
:004B94D6 83C003                  add eax, 00000003        <----SN[i]=SN[i]+3
:004B94D9 7105                    jno 004B94E0
* Reference To: VCL30.System.@IntOver@51F89FF7, Ord:0000h
            |
:004B94DB E8C07BF4FF              Call 004010A0
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004B94D9(C)
|
:004B94E0 6BC051                  imul eax, 00000051          <----SN[i]=SN[i]*0x51
:004B94E3 7105                    jno 004B94EA
* Reference To: VCL30.System.@IntOver@51F89FF7, Ord:0000h
            |
:004B94E5 E8B67BF4FF              Call 004010A0
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004B94E3(C)
|
:004B94EA 8BF8                    mov edi, eax
:004B94EC 8D55E8 lea edx, dword ptr [ebp-18]
:004B94EF 8BC7                    mov eax, edi
* Reference To: VCL30.SysUtils.IntToStr@0F6FDFF6, Ord:0000h
            |
:004B94F1 E8AA80F4FF              Call 004015A0              <---IntToStr
:004B94F6 8B55E8 mov edx, dword ptr [ebp-18]
:004B94F9 8D45F8 lea eax, dword ptr [ebp-08]
* Reference To: VCL30.System.@LStrCat@51F89FF7, Ord:0000h
            |
:004B94FC E88F7CF4FF              Call 00401190             <---StrCat
:004B9501 EB38                    jmp 004B953B
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004B94BD(C)
|
:004B9503 8D45E8 lea eax, dword ptr [ebp-18]
:004B9506 8B55FC mov edx, dword ptr [ebp-04]
:004B9509 8A543AFF                mov dl, byte ptr [edx+edi-01]
* Reference To: VCL30.System.@LStrFromChar@001FB870, Ord:0000h
            |
:004B950D E8567CF4FF              Call 00401168
:004B9512 8B45E8 mov eax, dword ptr [ebp-18]
* Reference To: VCL30.SysUtils.StrToInt@0F6FDFF6, Ord:0000h
            |
:004B9515 E88E80F4FF              Call 004015A8
:004B951A 6BC02F                  imul eax, 0000002F        <---eax=eax*2F
:004B951D 7105                    jno 004B9524
* Reference To: VCL30.System.@IntOver@51F89FF7, Ord:0000h
            |
:004B951F E87C7BF4FF              Call 004010A0
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004B951D(C)
|
:004B9524 8BF8                    mov edi, eax
:004B9526 8D55E8 lea edx, dword ptr [ebp-18]
:004B9529 8BC7                    mov eax, edi
* Reference To: VCL30.SysUtils.IntToStr@0F6FDFF6, Ord:0000h
            |
:004B952B E87080F4FF              Call 004015A0              <----IntToStr
:004B9530 8B55E8 mov edx, dword ptr [ebp-18]
:004B9533 8D45F8 lea eax, dword ptr [ebp-08]
* Reference To: VCL30.System.@LStrCat@51F89FF7, Ord:0000h
            |
:004B9538 E8537CF4FF              Call 00401190             <---StrCat
```
