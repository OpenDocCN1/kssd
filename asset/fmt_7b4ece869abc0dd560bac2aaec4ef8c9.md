
# IrfanView

> 原文：[`www.pediy.com/kssd/pediy06/pediy6234.htm`](https://www.pediy.com/kssd/pediy06/pediy6234.htm)

`IrfanView`

* `ASP` 加壳
* `pe-scan` 脱壳

这个软件的算法好复杂，头都大了，哪位老大帮忙给解释解释了，明码是比较，我想写注册机，可是算法没有分析透……

* `/Name:dnpf`
* `Code:283481102`

\* Reference To: `KERNEL32.GetPrivateProfileStringA`, Ord:013Ah

```
:00455039 8B35C0814C00 mov esi, dword ptr [004C81C0]
:0045503F . 68 C0DC4F00 PUSH UNIRFANV.004FDCC0 ; /IniFileName = "E:PROGRAM FILESIRFANVIEWi_view32.ini"
:00455044 . 8D5424 4C LEA EDX,DWORD PTR SS:[ESP+4C] ; |
:00455048 . 68 00010000 PUSH 100 ; |BufSize = 100 (256.)
:0045504D . 52 PUSH EDX ; |ReturnBuffer
:0045504E . 68 98204F00 PUSH UNIRFANV.004F2098 ; |Default = ""
:00455053 . 68 64814E00 PUSH UNIRFANV.004E8164 ; |Key = "Name"
:00455058 . 68 54814E00 PUSH UNIRFANV.004E8154 ; |Section = "Registration"
:0045505D . 33DB XOR EBX,EBX ; |
:0045505F . FFD6 CALL ESI ; GetPrivateProfileStringA
:00455061 . 68 C0DC4F00 PUSH UNIRFANV.004FDCC0 ; /IniFileName = "E:PROGRAM FILESIRFANVIEWi_view32.ini"
:00455066 . 8D8424 500100>LEA EAX,DWORD PTR SS:[ESP+150] ; |
:0045506D . 68 00010000 PUSH 100 ; |BufSize = 100 (256.)
:00455072 . 50 PUSH EAX ; |ReturnBuffer
:00455073 . 68 98204F00 PUSH UNIRFANV.004F2098 ; |Default = ""
:00455078 . 68 4C814E00 PUSH UNIRFANV.004E814C ; |Key = "Code"
:0045507D . 68 54814E00 PUSH UNIRFANV.004E8154 ; |Section = "Registration"
:00455082 . FFD6 CALL ESI ; GetPrivateProfileStringA
:00455084 8A442448 mov al, byte ptr [esp+48]
:00455088 84C0 test al, al
:0045508A 746A je 004550F6-------------------------------------->看有没有注册名
:0045508C 8A84244C010000 mov al, byte ptr [esp+0000014C]
:00455093 84C0 test al, al
:00455095 745F je 004550F6-------------------------------------->看有没有注册码
:00455097 8D8C244C010000 lea ecx, dword ptr [esp+0000014C]---------------->注册码
:0045509E 8D542448 lea edx, dword ptr [esp+48]---------------------->注册名
:004550A2 51 push ecx
:004550A3 52 push edx
:004550A4 E81718FEFF call 004368C0------------------------------------->验证是否合法(关键)
:004550A9 83C408 add esp, 00000008
:004550AC 85C0 test eax, eax
:004550AE 7446 je 004550F6--------------------------------------->不合法到下面要求输入
:004550B0 A14C394F00 mov eax, dword ptr [004F394C]
:004550B5 6804010000 push 00000104
:004550BA 6860C84F00 push 004FC860
\* Possible Reference to String Resource ID=01237: "IrfanView is already registered !"------------->注册成功后的提示
:004550BF 68D5040000 push 000004D5
:004550C4 50 push eax
\* Reference To: USER32.LoadStringA, Ord:01ABh
:004550C5 FF15A0834C00 Call dword ptr [004C83A0]
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0045567B(U)
:004550CB 8B0D88CB4F00 mov ecx, dword ptr [004FCB88]
:004550D1 6840200000 push 00002040
:004550D6 6800DF4F00 push 004FDF00
:004550DB 6860C84F00 push 004FC860
:004550E0 51 push ecx
\* Reference To: USER32.MessageBoxA, Ord:01BEh
:004550E1 FF15A4834C00 Call dword ptr [004C83A4]
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0044F65C(C), :004518BA(C), :00451A86(C), :00453AE7(C), :00453E22(C)
|:004540D7(C)
:004550E7 33C0 xor eax, eax
:004550E9 5F pop edi
:004550EA 5E pop esi
:004550EB 5D pop ebp
:004550EC 5B pop ebx
:004550ED 81C4580B0000 add esp, 00000B58
:004550F3 C21000 ret 0010
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0045508A(C), :00455095(C), :004550AE(C)
:004550F6 8D94245C050000 lea edx, dword ptr [esp+0000055C]---------------------------->没有注册时到这里
:004550FD 52 push edx\----------------------------------------
:004550FE 68606B4300 push 00436B60 |
\* Reference To: USER32.GetActiveWindow, Ord:00DDh
:00455103 FF1578834C00 Call dword ptr [004C8378] |
:00455109 50 push eax |
:0045510A A1F4E54F00 mov eax, dword ptr [004FE5F4] |
\* Possible Reference to Dialog: DialogID_0409
:0045510F 6809040000 push 00000409 |
:00455114 50 push eax |
\* Reference To: USER32.DialogBoxParamA, Ord:0093h
:00455115 FF15F4844C00 Call dword ptr [004C84F4]------------------------------------->到这里出输入框
:0045511B 85C0 test eax, eax
:0045511D 0F84AD000000 je 004551D0
:00455123 8DBC245C050000 lea edi, dword ptr [esp+0000055C]
:0045512A 83C9FF or ecx, FFFFFFFF
:0045512D 33C0 xor eax, eax
:0045512F F2 repnz
:00455130 AE scasb
:00455131 F7D1 not ecx
:00455133 49 dec ecx
:00455134 83F902 cmp ecx, 00000002
:00455137 0F825C010000 jb 00455299------------------------------->到错误
:0045513D 8DBC245C050000 lea edi, dword ptr [esp+0000055C]
:00455144 83C9FF or ecx, FFFFFFFF
:00455147 F2 repnz
:00455148 AE scasb
:00455149 F7D1 not ecx
:0045514B 49 dec ecx
:0045514C 83F955 cmp ecx, 00000055
:0045514F 0F8744010000 ja 00455299------------------------------->到错误
:00455155 8DBC245C060000 lea edi, dword ptr [esp+0000065C]
:0045515C 83C9FF or ecx, FFFFFFFF
:0045515F 33D2 xor edx, edx
:00455161 F2 repnz
:00455162 AE scasb
:00455163 F7D1 not ecx
:00455165 49 dec ecx
:00455166 85C9 test ecx, ecx
:00455168 7E75 jle 004551DF
```
