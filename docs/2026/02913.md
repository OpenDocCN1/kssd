
# PC 控制台(PC Controler) V1.5.75 破解过程

（5千字）

破解撰写：leeyam[BCG]

首先运行该程序，随意填写注册信息，提示需要重新启动软件验证注册码！

判断该程序先将输入的注册码存放某个位置，然后启动时调用！

用Language发现是用ASPACK加的壳，很容易脱掉。然后利用W32Dasm反编，查找字串，发现程序会调用注册表，`"\Software\MySoftware Studio\UserInfo"`。

从第一个调用入手：

```
* Possible StringData Ref from Code Obj ->"\Software\MySoftware
Studio\UserInfo"
                  |
 :00489B6C BA909E4800              mov edx, 00489E90
:00489B71 8B45F8
mov eax, dword ptr [ebp-08]
 :00489B74 E81B78FDFF      call 00461394
 :00489B79 8845FF          mov byte ptr [ebp-01], al
 :00489B7C 807DFF00              cmp byte ptr [ebp-01], 00
:00489B80 750A
jne 00489B8C
 :00489B82 E841A7F7FF  call 004042C8
 :00489B87 E9C3020000    jmp 00489E4F
 * Referenced by a (U)nconditional or (C)onditional
Jump at Address:
 |:00489B80(C)
 |
 * Possible StringData Ref
from Code Obj ->"UserName"
                    |
 :00489B8C BAC09E4800              mov edx, 00489EC0
:00489B91 8B45F8
mov eax, dword ptr [ebp-08]
 :00489B94 E8A77FFDFF      call 00461B40
 :00489B99 84C0            test al, al
 :00489B9B 0F848D020000          je 00489E2E
 * Possible StringData
Ref from Code Obj ->"UserID"
                    |
 :00489BA1 BAD49E4800              mov edx, 00489ED4
:00489BA6 8B45F8
mov eax, dword ptr [ebp-08]
 :00489BA9 E8927FFDFF      call 00461B40
 :00489BAE 84C0            test al, al
 :00489BB0 0F8478020000          je 00489E2E
 * Possible StringData
Ref from Code Obj ->"UserEmail"
                      |
 :00489BB6 BAE49E4800              mov
edx, 00489EE4
 :00489BBB 8B45F8
    mov eax, dword ptr [ebp-08]
 :00489BBE E87D7FFDFF          call 00461B40
 :00489BC3 84C0
                test al, al
 :00489BC5
0F8463020000            je 00489E2E
 *
Possible StringData Ref from Code Obj ->"10674538806"
    |
 :00489BCB BAF89E4800
  mov edx, 00489EF8
 :00489BD0 8B45F8
        mov eax, dword ptr [ebp-08]
 :00489BD3 E8687FFDFF            call 00461B40
 :00489BD8 84C0
                  test al, al
:00489BDA 0F844E020000            je 00489E2E
:00489BE0 33C0
xor eax, eax
 :00489BE2 55
        push ebp
 :00489BE3 68389C4800
        push 00489C38
 :00489BE8 64FF30
            push dword ptr fs:[eax]
 :00489BEB
648920                  mov dword
ptr fs:[eax], esp
 :00489BEE 8D4DF4
      lea ecx, dword ptr [ebp-0C]
 * Possible StringData
Ref from Code Obj ->"UserName"
                      |
 :00489BF1 BAC09E4800              mov
edx, 00489EC0
 :00489BF6 8B45F8
    mov eax, dword ptr [ebp-08]
 :00489BF9 E8167DFDFF          call 00461914
 :00489BFE 8D4DF0
              lea ecx, dword ptr [ebp-10]
 * Possible StringData Ref from Code Obj ->"UserID"
    |
 :00489C01 BAD49E4800
  mov edx, 00489ED4
 :00489C06 8B45F8
        mov eax, dword ptr [ebp-08]
 :00489C09 E8067DFDFF            call 00461914
 :00489C0E 8D4DEC
                lea ecx, dword ptr [ebp-14]
 * Possible StringData Ref from Code Obj ->"UserEmail"
      |
 :00489C11 BAE49E4800
    mov edx, 00489EE4
 :00489C16 8B45F8
          mov eax, dword ptr [ebp-08]
 :00489C19
E8F67CFDFF              call 00461914
 * Possible StringData Ref from Code Obj ->"10674538806"
      |
 :00489C1E BAF89E4800
    mov edx, 00489EF8
 :00489C23 8D4DE8
          lea ecx, dword ptr [ebp-18]
 :00489C26
8B45F8                  mov eax,
dword ptr [ebp-08]
 :00489C29 E8E67CFDFF
    call 00461914
 :00489C2E 33C0
          xor eax, eax
 :00489C30 5A
                  pop edx
 :00489C31
59
pop ecx
 :00489C32 59
      pop ecx
 :00489C33 648910
          mov dword ptr fs:[eax], edx
 :00489C36
EB1D                    jmp
00489C55
 :00489C38 E9F3A2F7FF
jmp 00403F30
 :00489C3D C645FF00
    mov [ebp-01], 00
 :00489C41 E852A6F7FF
      call 00404298
 :00489C46 E87DA6F7FF
        call 004042C8
 :00489C4B E9FF010000
          jmp 00489E4F
 :00489C50 E843A6F7FF
            call 00404298
```

以上可以很容易发现是在调入注册表里填写的注册信息！相信判断注册码的位置就在下面，再往下看看！发现以下代码想必是判断关键：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00489D06(C)
|
:00489D49 8B45E8
      mov eax, dword ptr [ebp-18]…………………………调入假码
:00489D4C
8B55DC                  mov edx,
dword ptr [ebp-24]…………………………调入真码
:00489D4F E864AEF7FF
        call 00404BB8…………………………判断注册码！
:00489D54 0F9445FF
              sete byte ptr [ebp-01]
:00489D58
807DFF00                cmp byte ptr [ebp-01],
00
:00489D5C 0F84D0000000            je 00489E32
```

于是调用我的TRW2000，直接下中断`bpx 489d4f`，然后运行该程序，顺利被拦截，马上`D edx`得到正确注册码，添入软件后注册成功！
