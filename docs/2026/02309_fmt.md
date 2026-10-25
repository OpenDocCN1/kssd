
# ThemeFreak V1.6破解 (8千字)

软件名：`ThemeFreak V1.6`（桌面屏保）

下载：URL `ftp://ftp.cq.ilike.myrice.com/soft/soft_desktool/TFreak.exe`

限制：不注册只能用十天

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A2E25(C)
|
:004A2E94 33C0
  xor eax, eax
:004A2E96 55
  push ebp
:004A2E97 6859304A00              push 004A3059
:004A2E9C 64FF30
  push dword ptr fs:[eax]
:004A2E9F 648920
  mov dword ptr fs:[eax], esp
:004A2EA2 8D55FC
  lea edx, dword ptr [ebp-04]
:004A2EA5 8B83D0020000            mov eax, dword ptr [ebx+000002D0]
:004A2EAB E878FFF8FF              call 00432E28
:004A2EB0 8B45FC
  mov eax, dword ptr [ebp-04]
:004A2EB3 E80C10F6FF              call 00403EC4
:004A2EB8 83F808
  cmp eax, 00000008    <-----比较注册名与 8个字节
:004A2EBB 7D0F
  jge 004A2ECC          <-----大于等于就跳，不跳就BYE-BYE
* Possible StringData Ref from Code Obj ->"Your registration name has a minimun
"
                  ->"of 8 characters. Please copy the "
                  ->"Registration name and code exactly "
                  ->"as they are in your registration "
                  ->"email!"
            |
:004A2EBD B870304A00              mov eax, 004A3070
:004A2EC2 E84156FBFF              call 00458508
:004A2EC7 E95D010000              jmp 004A3029
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A2EBB(C)
|
:004A2ECC 8D55F8
  lea edx, dword ptr [ebp-08]
:004A2ECF 8B83D0020000            mov eax, dword ptr [ebx+000002D0]
:004A2ED5 E84EFFF8FF              call 00432E28
:004A2EDA 8B55F8
  mov edx, dword ptr [ebp-08]
:004A2EDD 8B83F4020000            mov eax, dword ptr [ebx+000002F4]
:004A2EE3 83C024
  add eax, 00000024
:004A2EE6 E8AD0DF6FF              call 00403C98
:004A2EEB 8D55F4
  lea edx, dword ptr [ebp-0C]
:004A2EEE 8B83DC020000            mov eax, dword ptr [ebx+000002DC]
:004A2EF4 E82FFFF8FF              call 00432E28
:004A2EF9 8B45F4
  mov eax, dword ptr [ebp-0C]
:004A2EFC 50
  push eax
:004A2EFD 8D55F0
  lea edx, dword ptr [ebp-10]
:004A2F00 8B83F4020000            mov eax, dword ptr [ebx+000002F4]
:004A2F06 E89D4AFEFF              call 004879A8
:004A2F0B 8B55F0
  mov edx, dword ptr [ebp-10]
:004A2F0E 58
  pop eax
:004A2F0F E8C010F6FF              call 00403FD4    <-------关键的CALL，进去看看
:004A2F14 0F8505010000            jne 004A301F
  <-------跳就BYE-BYE了，出现“Invalid Registration name and/or code! Please
copy the Registration name and code exactly as they are in your registration
email!”窗口
:004A2F1A B201
  mov dl, 01
:004A2F1C A184844600              mov eax, dword ptr [00468484]
:004A2F21 E85E56FCFF              call 00468584
:004A2F26 8BF0
  mov esi, eax
:004A2F28 BA02000080              mov edx, 80000002
:004A2F2D 8BC6
  mov eax, esi
:004A2F2F E8F056FCFF              call 00468624
:004A2F34 B101
  mov cl, 01
* Possible StringData Ref from Code Obj ->"Software\Cwhere\ThemeFreak"
            |
:004A2F36 BA0C314A00              mov edx, 004A310C
:004A2F3B 8BC6
  mov eax, esi
:004A2F3D E84A57FCFF              call 0046868C
* Possible StringData Ref from Code Obj ->"Data"
            |
:004A2F42 BA30314A00              mov edx, 004A3130
:004A2F47 8BC6
  mov eax, esi
:004A2F49 E8DA59FCFF              call 00468928
:004A2F4E 8D55EC
  lea edx, dword ptr [ebp-14]
:004A2F51 8B83D0020000            mov eax, dword ptr [ebx+000002D0]
:004A2F57 E8CCFEF8FF              call 00432E28
:004A2F5C 8B4DEC
  mov ecx, dword ptr [ebp-14]
* Possible StringData Ref from Code Obj ->"RegName"
            |
:004A2F5F BA40314A00              mov edx, 004A3140
:004A2F64 8BC6
  mov eax, esi
:004A2F66 E8BD5AFCFF              call 00468A28
:004A2F6B 8D55E8
  lea edx, dword ptr [ebp-18]
:004A2F6E 8B83DC020000            mov eax, dword ptr [ebx+000002DC]
:004A2F74 E8AFFEF8FF              call 00432E28
:004A2F79 8B4DE8
  mov ecx, dword ptr [ebp-18]
* Possible StringData Ref from Code Obj ->"RegNum"
            |
:004A2F7C BA50314A00              mov edx, 004A3150
:004A2F81 8BC6
  mov eax, esi
:004A2F83 E8A05AFCFF              call 00468A28
:004A2F88 8BC6
  mov eax, esi
:004A2F8A E86556FCFF              call 004685F4
:004A2F8F 8BC6
  mov eax, esi
:004A2F91 E8E6FFF5FF              call 00402F7C
:004A2F96 A1008B4B00              mov eax, dword ptr [004B8B00]
:004A2F9B 8B00
  mov eax, dword ptr [eax]
:004A2F9D 8B80F0020000            mov eax, dword ptr [eax+000002F0]
:004A2FA3 33D2
  xor edx, edx
:004A2FA5 E896FDF8FF              call 00432D40
:004A2FAA 8D55E4
  lea edx, dword ptr [ebp-1C]
:004A2FAD 8B83D0020000            mov eax, dword ptr [ebx+000002D0]
:004A2FB3 E870FEF8FF              call 00432E28
:004A2FB8 8B55E4
  mov edx, dword ptr [ebp-1C]
:004A2FBB A1008B4B00              mov eax, dword ptr [004B8B00]
:004A2FC0 8B00
  mov eax, dword ptr [eax]
:004A2FC2 8B80EC020000            mov eax, dword ptr [eax+000002EC]
:004A2FC8 E88BFEF8FF              call 00432E58
:004A2FCD A1008B4B00              mov eax, dword ptr [004B8B00]
:004A2FD2 8B00
  mov eax, dword ptr [eax]
:004A2FD4 8B80EC020000            mov eax, dword ptr [eax+000002EC]
:004A2FDA B201
  mov dl, 01
:004A2FDC E85FFDF8FF              call 00432D40
:004A2FE1 A1008B4B00              mov eax, dword ptr [004B8B00]
:004A2FE6 8B00
  mov eax, dword ptr [eax]
:004A2FE8 8B80E8020000            mov eax, dword ptr [eax+000002E8]
:004A2FEE B201
  mov dl, 01
:004A2FF0 E84BFDF8FF              call 00432D40
:004A2FF5 A1D88B4B00              mov eax, dword ptr [004B8BD8]
:004A2FFA 8B00
  mov eax, dword ptr [eax]
:004A2FFC 8B8014050000            mov eax, dword ptr [eax+00000514]
:004A3002 33D2
  xor edx, edx
:004A3004 E8C705FAFF              call 004435D0
* Possible StringData Ref from Code Obj ->"Thankyou for registering ThemeFreak!"
            |
:004A3009 B860314A00              mov eax, 004A3160
:004A300E E8F554FBFF              call 00458508
:004A3013 A170994B00              mov eax, dword ptr [004B9970]
:004A3018 E81BAEFAFF              call 0044DE38
:004A301D EB0A
  jmp 004A3029
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A2F14(C)      <----原来是从这儿来的
|
* Possible StringData Ref from Code Obj ->"Invalid Registration name and/or
"
                  ->"code! Please copy the Registration "
                  ->"name and code exactly as they "
                  ->"are in your registration email!"
****************************************************************略****************************************************************
            |
:004A301F B890314A00              mov eax, 004A3190
:004A3024 E8DF54FBFF              call 00458508
:00403FD4 53
  push ebx
:00403FD5 56
  push esi
:00403FD6 57
  push edi
:00403FD7 89C6
  mov esi, eax    <--------下D EAX 你输入的注册码
:00403FD9 89D7
  mov edi, edx    <--------下D EDX 真正的注册码
:00403FDB 39D0
  cmp eax, edx
:00403FDD 0F848F000000            je 00404072
:00403FE3 85F6
  test esi, esi
:00403FE5 7468
  je 0040404F
:00403FE7 85FF
  test edi, edi
:00403FE9 746B
  je 00404056
:00403FEB 8B46FC
  mov eax, dword ptr [esi-04]
:00403FEE 8B57FC
  mov edx, dword ptr [edi-04]
:00403FF1 29D0
  sub eax, edx
:00403FF3 7702
  ja 00403FF7
:00403FF5 01C2
  add edx, eax
```

```
注册名：调试软件
注册码：35405055
```

哇！写破解教学比破解软件累多了，呵呵！在此谨向广大为初学者不辞辛劳写破解教学的前辈致以崇高的敬意！
