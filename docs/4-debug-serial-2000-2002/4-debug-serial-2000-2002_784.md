
# DrawingHand v4.0

> 原文：[`www.pediy.com/kssd/pediy04/kanxue045.htm`](https://www.pediy.com/kssd/pediy04/kanxue045.htm)

一个屏幕保护程序，挺好玩的，不妨一试

下载：`http://software.wx88.net/down/drawinghand.zip`

难得碰到这么好破的软件，拿上来与大家共享，该软件声称自己是在线注册，云云

结果被我用 `W32Dasm` 两分钟搞定。

## 第一步

搜索 `string data reference` 找 `registered` 来到

```asm
* Possible StringData Ref from Data Obj ->"Configuration"
                     |
:00403408 68D8124200        push 004212D8
:0040340D FFD7                call edi
:0040340F 6A00                push 00000000
:00403411 50                  push eax
:00403412 68F1000000          push 000000F1
:00403417 681C040000          push 0000041C
:0040341C 56                  push esi
:0040341D A3C0384200          mov dword ptr [004238C0], eax
:00403422 FFD3                call ebx
:00403424 A14CC04300          mov eax, dword ptr [0043C04C]<--------注意这里，如果[0043c04c]
:00403429 85C0                test eax, eax
:0040342B 7443                je 00403470                       的值为 1，就为注册版

* Possible StringData Ref from Data Obj ->"Registered
"
                     |
:0040342D 684C134200          push 0042134C
:00403432 E8290B0000          call 00403F60
:00403437 83C404              add esp, 00000004

* Possible StringData Ref from Data Obj ->"Registered"
                     |
:0040343A 6840134200          push 00421340
:0040343F 68F9000000          push 000003F9
:00403444 56                  push esi

* Reference To: USER32.SetDlgItemTextA, Ord:022Ch
                     |
:00403445 FF15B0084400        Call dword ptr [004408B0]

* Reference To: USER32.GetDlgItem, Ord:0102h
                     |
:0040344B 8B1D94084400        mov ebx, dword ptr [00440894]

:00403451 68F9000000          push 000003F9
:00403456 56                  push esi
:00403457 FFD3                call ebx

* Reference To: USER32.EnableWindow, Ord:00B7h
                     |
:00403459 8B3D98084400        mov edi, dword ptr [00440898]

:0040345F 6A00                push 00000000
:00403461 50                  push eax
:00403462 FFD7                call edi
:00403464 68F8000000          push 000003F8
:00403469 56                  push esi
:0040346A FFD3                call ebx
:0040346C 6A00                push 00000000
:0040346E EB30                jmp 004034A0

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040342B(C)
|

* Possible StringData Ref from Data Obj ->"NOT Registered
"
                     |
:00403470 6830134200          push 00421330
:00403475 E8E60A0000          call 00403F60

* Reference To: USER32.GetDlgItem, Ord:0102h
                     |
:0040347A 8B1D94084400        mov ebx, dword ptr [00440894]

:00403480 83C404              add esp, 00000004
:00403483 68F9000000          push 000003F9
:00403488 56                  push esi
:00403489 FFD3                call ebx

* Reference To: USER32.EnableWindow, Ord:00B7h
                     |
:0040348B 8B3D98084400        mov edi, dword ptr [00440898]

* Possible Reference to String Resource ID=00001: "Drawing Hand 4.0"
                     |
:00403491 6A01                push 00000001
:00403493 50                  push eax
:00403494 FFD7                call edi
:00403496 68F8000000          push 000003F8
:0040349B 56                  push esi
:0040349C FFD3                call ebx

* Possible Reference to String Resource ID=00001: "Drawing Hand 4.0"
                     |
:0040349E 6A01                push 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040346E(U)
|
:004034A0 50                  push eax
:004034A1 FFD7                call edi
:004034A3 8D4C2410            lea ecx, dword ptr [esp+10]
:004034A7 E86D1A0100          call 00414F19
:004034AC 8D4C2414            lea ecx, dword ptr [esp+14]
:004034B0 E8641A0100          call 00414F19

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00402A61(C)
|

* Possible StringData Ref from Data Obj ->"ScreenSaverConfigureDialog = WM_DESTROY
"
                     |
:004034B5 6804134200          push 00421304
:004034BA E8A10A0000          call 00403F60
:004034BF 83C404              add esp, 00000004
:004034C2 E8190C0000          call 004040E0

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00402A73(C), :00402AA4(C), :00402E71(C), :00402EEC(C), :00402F2E(C)
|:00403077(C), :004030F2(C)
|
:004034C7 5F                  pop edi
:004034C8 5E                  pop esi
:004034C9 5D                  pop ebp
:004034CA 33C0                xor eax, eax
:004034CC 5B                  pop ebx
:004034CD 81C4740B0000        add esp, 00000B74
:004034D3 C21000              ret 0010
```

## 第二步

搜索字符串 `[0043c04c]`，来到这里

```asm
* Reference To: KERNEL32.GetPrivateProfileStringA, Ord:013Ah
                     |
:00404F2E FF1560074400        Call dword ptr [00440760]
:00404F34 8A442404            mov al, byte ptr [esp+04]
:00404F38 84C0                test al, al
:00404F3A 7448                je 00404F84
:00404F3C 8D4C2404            lea ecx, dword ptr [esp+04]
:00404F40 51                  push ecx
:00404F41 8D4C2404            lea ecx, dword ptr [esp+04]
:00404F45 E8F9FF0000          call 00414F43
:00404F4A 8B442400            mov eax, dword ptr [esp]
:00404F4E 803830              cmp byte ptr [eax], 30<------------0
:00404F51 7528                jne 00404F7B
:00404F53 80780131            cmp byte ptr [eax+01], 31<---------1
:00404F57 7522                jne 00404F7B
:00404F59 80780232            cmp byte ptr [eax+02], 32<---------2
:00404F5D 751C                jne 00404F7B
:00404F5F 80780338            cmp byte ptr [eax+03], 38<---------8
:00404F63 7516                jne 00404F7B
:00404F65 80780436            cmp byte ptr [eax+04], 36<---------6
:00404F69 7510                jne 00404F7B
:00404F6B 80780535            cmp byte ptr [eax+05], 35<---------5
:00404F6F 750A                jne 00404F7B

* Possible Reference to String Resource ID=00001: "Drawing Hand 4.0"
                     |
:00404F71 C7054CC0430001000000  mov dword ptr [0043C04C], 00000001<--------看到了没,往上看看
```

## 注册码

`012865`

## 签名

```text
              ,;~;,
                /\__
              ( /
              (()      //)
              | \\ ,;;'\
            __ _( )m=(lancelot(================--------
         /' ' '()/~' '.(, |
      ,;(      )||   | ~
    ,;' \  /-(.;, )              兰斯洛特[CCG][FCG]
          /      /
        //      ||
        )_\\      )_\\

\=======================================================  4dd466d0e1e0cc123afbe504da2462ca
```
