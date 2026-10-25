
# 奇迹时代

> 原文：`www.pediy.com/kssd/pediy03/forum527.htm`

一个类似于英雄无敌三的游戏,它的光盘校验不是主程序`AoW.exe`，而是`AoWEPACK.dpl`实际上是一个 DLL 文件,`AoW.exe`里同样有校验但只是虚设,我开始就在它里面转圈子,这一点让我耽误了好几分钟时间。

以下是`AoWEPACK.dpl`中无聊的光盘校验代码:

```
* Referenced by a CALL at Address:
|:557063ED

Exported fn(): AoWReg.AoWCDInserted@82678C4D - Ord:0004h
:55706374 53                       push ebx
:55706375 56                       push esi
:55706376 57                       push edi
:55706377 51                       push ecx
:55706378 33DB                     xor ebx, ebx
:5570637A 33F6                     xor esi, esi

* Reference To: kernel32.GetLogicalDrives, Ord:0000h
:5570637C E803B1FFFF               Call 55701484
:55706381 8BF8                     mov edi, eax
:55706383 EB24                     jmp 557063A9

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:557063B0(C)
:55706385 8BCE                     mov ecx, esi
:55706387 B801000000               mov eax, 00000001
:5570638C D3E0                     shl eax, cl
:5570638E 85F8                     test eax, edi
:55706390 7416                     je 557063A8
:55706392 8BC6                     mov eax, esi
:55706394 0461                     add al, 61
:55706396 880424                   mov byte ptr [esp], al
:55706399 8A0424                   mov al, byte ptr [esp]

* Reference To: AoWEPACK.AoWReg.ValidAoWCD@BF94463D
:5570639C E8D3FEFFFF               call 55706274  -->检查光盘的 CALL,想省事的话
:557063A1 84C0                     test al, al    -->改为 B801000000(MOV EAX,00000001)
:557063A3 7403                     je 557063A8
:557063A5 8A1C24                   mov bl, byte ptr [esp]

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:55706390(C), :557063A3(C)
:557063A8 46                       inc esi

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:55706383(U)
:557063A9 84DB                     test bl, bl
:557063AB 7505                     jne 557063B2
:557063AD 83FE1A                   cmp esi, 0000001A
:557063B0 75D3                     jne 55706385

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:557063AB(C)
:557063B2 8BC3                     mov eax, ebx
:557063B4 5A                       pop edx
:557063B5 5F                       pop edi
:557063B6 5E                       pop esi
:557063B7 5B                       pop ebx
:557063B8 C3                       ret
```

`call 55706274` 的内容:

```
* Referenced by a CALL at Address:
|:5570639C

Exported fn(): AoWReg.ValidAoWCD@BF94463D - Ord:0005h
:55706274 55                       push ebp
:55706275 8BEC                     mov ebp, esp
:55706277 6A00                     push 00000000
:55706279 6A00                     push 00000000
:5570627B 6A00                     push 00000000
:5570627D 53                       push ebx
:5570627E 8845FF                   mov byte ptr [ebp-01], al
:55706281 33C0                     xor eax, eax
:55706283 55                       push ebp
:55706284 682E637055               push 5570632E
:55706289 64FF30                   push dword ptr fs:[eax]
:5570628C 648920                   mov dword ptr fs:[eax], esp
:5570628F 8D45F4                   lea eax, dword ptr [ebp-0C]
:55706292 8A55FF                   mov dl, byte ptr [ebp-01]

* Reference To: VCL30.System.@LStrFromChar@001FB870, Ord:0000h
:55706295 E8C6AEFFFF               Call 55701160
:5570629A 8B55F4                   mov edx, dword ptr [ebp-0C]
:5570629D 8D45F8                   lea eax, dword ptr [ebp-08]
:557062A0 B944637055               mov ecx, 55706344

* Reference To: VCL30.System.@LStrCat3@51F89FF7, Ord:0000h
:557062A5 E8E6AEFFFF               Call 55701190
:557062AA 8B45F8                   mov eax, dword ptr [ebp-08]

* Reference To: VCL30.System.@LStrToPChar@51F89FF7, Ord:0000h
:557062AD E8FEAEFFFF               Call 557011B0
:557062B2 50                       push eax

* Reference To: kernel32.GetDriveTypeA, Ord:0000h
:557062B3 E8BCB1FFFF               Call 55701474
:557062B8 83F805                   cmp eax, 00000005
:557062BB 7554                     jne 55706311
:557062BD 8D45F4                   lea eax, dword ptr [ebp-0C]
:557062C0 8A55FF                   mov dl, byte ptr [ebp-01]

* Reference To: VCL30.System.@LStrFromChar@001FB870, Ord:0000h
:557062C3 E898AEFFFF               Call 55701160
:557062C8 8D45F4                   lea eax, dword ptr [ebp-0C]

* Possible StringData Ref from Code Obj ->":\AoWSetup.exe" -->读取光盘中根目录下
                                                  -->的 AoWSetup.exe 文件
:557062CB BA50637055               mov edx, 55706350

* Reference To: VCL30.System.@LStrCat@51F89FF7, Ord:0000h
:557062D0 E8B3AEFFFF               Call 55701188

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:55706261(C)
:557062D5 8B45F4                   mov eax, dword ptr [ebp-0C]

* Reference To: VCL30.SysUtils.FileExists@3C9BDBFB, Ord:0000h
:557062D8 E8C7B2FFFF               Call 557015A4
:557062DD 8BD8                     mov ebx, eax
:557062DF 84DB                     test bl, bl
:557062E1 7424                     je 55706307
:557062E3 8D45F4                   lea eax, dword ptr [ebp-0C]
:557062E6 8A55FF                   mov dl, byte ptr [ebp-01]

* Reference To: VCL30.System.@LStrFromChar@001FB870, Ord:0000h
:557062E9 E872AEFFFF               Call 55701160
:557062EE 8D45F4                   lea eax, dword ptr [ebp-0C]

* Possible StringData Ref from Code Obj ->":\setup.exe"-->再读 SETUP.EXE 文件
:557062F1 BA68637O55               mov edx, 55706368

* Reference To: VCL30.System.@LStrCat@51F89FF7, Ord:0000h
:557062F6 E88DAEFFFF               Call 55701188
:557062FB 8B45F4                   mov eax, dword ptr [ebp-0C]

* Reference To: VCL30.SysUtils.FileExists@3C9BDBFB, Ord:0000h
:557062FE E8A1B2FFFF               Call 557015A4
:55706303 84C0                     test al, al
:55706305 7504                     jne 5570630B

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:557062E1(C)
:55706307 33C0                     xor eax, eax
:55706309 EB02                     jmp 5570630D

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:55706305(C)
:5570630B B001                     mov al, 01

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:55706309(U)
:5570630D 8BD8                     mov ebx, eax
:5570630F EB02                     jmp 55706313

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:557062BB(C)
:55706311 33DB                     xor ebx, ebx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:5570630F(U)
:55706313 33C0                     xor eax, eax
:55706315 5A                       pop edx
:55706316 59                       pop ecx
:55706317 59                       pop ecx
:55706318 648910                   mov dword ptr fs:[eax], edx
```



:5570631B 6835637055             push 55706335

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`:55706333(U)`

`|`

:55706320 8D45F4                  lea eax, dword ptr [ebp-0C]

:55706323 BA02000000             mov edx, 00000002

`* Reference To: VCL30.System.@LStrArrayClr@51F89FF7, Ord:0000h`

`                          |`

:55706328 E81BAEFFFF             Call 55701148

:5570632D C3                     ret  4dd466d0e1e0cc123afbe504da2462ca
