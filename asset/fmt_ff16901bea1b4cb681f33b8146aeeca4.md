
:004AFD50 `8B80E4020000`             `mov eax, dword ptr [eax+000002E4]`   

:004AFD56 `E82926F8FF`             `call 00432384`   

:004AFD5B `8B45E8`                 `mov eax, dword ptr [ebp-18]`   

:004AFD5E `5A`                     `pop edx`   

:004AFD5F `E8843BFCFF`             `call 004738E8`   

:004AFD64 `6A40`                   `push 00000040`   

* Possible StringData Ref from Data Obj ->"Flash Saver Maker"   

                             |   

:004AFD66 `68D0FD4A00`             `push 004AFDD0`   

* Possible StringData Ref from Data Obj ->"Registered Success"--------------->注册成功对话框   

                             |   

:004AFD6B `68E4FD4A00`             `push 004AFDE4`   

:004AFD70 `8B45FC`                 `mov eax, dword ptr [ebp-04]`   

:004AFD73 `E85089F8FF`             `call 004386C8`   

:004AFD78 `50`                     `push eax`   

* Reference To: `user32.MessageBoxA`, Ord:0000h   

                             |   

:004AFD79 `E8227FF5FF`             `Call 00407CA0`   

:004AFD7E `8B45FC`                 `mov eax, dword ptr [ebp-04]`   

:004AFD81 `C7803402000001000000`   `mov dword ptr [ebx+00000234], 00000001`   

:004AFD8B `EB1A`                   `jmp 004AFDA7`   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:004AFD33(C)---------------------------------------------------------------->从这里跳来   

|   

:004AFD8D `6A40`                   `push 00000040`   

* Possible StringData Ref from Data Obj ->"Flash Saver Maker"   

                             |   

:004AFD8F `68D0FD4A00`             `push 004AFDD0`   

* Possible StringData Ref from Data Obj ->"Registered Code Error"------------>注册错误对话框   

                             |   

:004AFD94 `68F8FD4A00`             `push 004AFDF8`   

:004AFD99 `8B45FC`                 `mov eax, dword ptr [ebp-04]`   

:004AFD9C `E82789F8FF`             `call 004386C8`   

:004AFDA1 `50`                     `push eax`   

* Reference To: `user32.MessageBoxA`, Ord:0000h   

                             |   

:004AFDA2 `E8F97EF5FF`             `Call 00407CA0`   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:004AFD8B(U)   

|   

:004AFDA7 `33C0`                   `xor eax, eax`   

:004AFDA9 `5A`                     `pop edx`   

:004AFDAA `59`                     `pop ecx`   

:004AFDAB `59`                     `pop ecx`   

:004AFDAC `648910`                 `mov dword ptr fs:[eax], edx`   

:004AFDAF `68C9FD4A00`             `push 004AFDC9`   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:004AFDC7(U)   

|   

:004AFDB4 `8D45E8`                 `lea eax, dword ptr [ebp-18]`   

:004AFDB7 `BA04000000`             `mov edx, 00000004`   

:004AFDBC `E8CB40F5FF`             `call 00403E8C`   

:004AFDC1 `C3`                     `ret`   

:004AFDC2 `E9B53AF5FF`             `jmp 0040387C`   

:004AFDC7 `EBEB`                   `jmp 004AFDB4`   

:004AFDC9 `8BE5`                   `mov esp, ebp`   

:004AFDCB `5D`                     `pop ebp`   

:004AFDCC `C3`                     `ret`   

******************************************************************************   

******************************************************************************   

### 00473A38 验证◎◎◎◎◎   

『返回时如果 AL 为零则后续会出错』   

`00473A38` `$ 55`             `PUSH EBP`   

`00473A39` `. 8BEC`           `MOV EBP,ESP`   

`00473A3B` `. 83C4 E8`        `ADD ESP,-18`   

`00473A3E` `. 33C9`           `XOR ECX,ECX`   

`00473A40` `. 894D F4`        `MOV DWORD PTR SS:[EBP-C],ECX`   

`00473A43` `. 894D F0`        `MOV DWORD PTR SS:[EBP-10],ECX`   

`00473A46` `. 8955 FC`        `MOV DWORD PTR SS:[EBP-4],EDX`------------>输入的注册码   

`00473A49` `. 8945 E8`        `MOV DWORD PTR SS:[EBP-18],EAX`----------->输入的用户名   

`00473A4C` `. 8B45 E8`        `MOV EAX,DWORD PTR SS:[EBP-18]`   

`00473A4F` `. E8 4808F9FF`    `CALL 0040429C`--------------------------->?   

`00473A54` `. 8B45 FC`        `MOV EAX,DWORD PTR SS:[EBP-4]`------------>输入的注册码   

`00473A57` `. E8 4008F9FF`    `CALL 0040429C`--------------------------->?   

`00473A5C` `. 33C0`           `XOR EAX,EAX`   

`00473A5E` `. 55`             `PUSH EBP`   

`00473A5F` `. 68 4D3B4700`    `PUSH 00473B4D`   

`00473A64` `. 64:FF30`        `PUSH DWORD PTR FS:[EAX]`   

`00473A67` `. 64:8920`        `MOV DWORD PTR FS:[EAX],ESP`   

`00473A6A` `. C645 FB 00`     `MOV BYTE PTR SS:[EBP-5],0`   

`00473A6E` `. 8B45 FC`        `MOV EAX,DWORD PTR SS:[EBP-4]`----------->输入的注册码   

`00473A71` `. E8 7206F9FF`    `CALL 004040E8`-------------------------->取输入注册码的长度   

`00473A76` `. 83F8 10`        `CMP EAX,10`----------------------------->比较长度是否为 10h(即 16)   

`00473A79` `. 0F85 A3000000`  `JNZ 00473B22`--------------------------->注册码长度必须为 10h(即 16)位   

`00473A7F` `. B2 01`          `MOV DL,1`   

`00473A81` `. A1 342E4700`    `MOV EAX,DWORD PTR DS:[472E34]`---------->ASSII 07,"TBase64"   

`00473A86` `. E8 FDF3FFFF`    `CALL 00472E88`   

`00473A8B` `. 8945 EC`        `MOV DWORD PTR SS:[EBP-14],EAX`---------->保存上面 ASCII 的地址   

`00473A8E` `. 33C0`           `XOR EAX,EAX`   

`00473A90` `. 55`             `PUSH EBP`   

`00473A91` `. 68 1B3B4700`    `PUSH 00473B1B`   

`00473A96` `. 64:FF30`        `PUSH DWORD PTR FS:[EAX]`   

`00473A99` `. 64:8920`        `MOV DWORD PTR FS:[EAX],ESP`   

`00473A9C` `. 8D45 F4`        `LEA EAX,DWORD PTR SS:[EBP-C]`---------->新地址   

`00473A9F` `. 8B55 FC`        `MOV EDX,DWORD PTR SS:[EBP-4]`---------->输入的注册码   

`00473AA2` `. E8 5904F9FF`    `CALL 00403F00`------------------------->注册码到上面新地址   

`00473AA7` `. 8D45 F0`        `LEA EAX,DWORD PTR SS:[EBP-10]`--------->新地址   

`00473AAA` `. E8 B903F9FF`    `CALL 00403E68`------------------------->?   

`00473AAF` `. 8D4D F0`        `LEA ECX,DWORD PTR SS:[EBP-10]`--------->00000000   

`00473AB2` `. 8B55 F4`        `MOV EDX,DWORD PTR SS:[EBP-C]`---------->输入的注册码   

`00473AB5` `. 8B45 EC`        `MOV EAX,DWORD PTR SS:[EBP-14]`--------->ASSII 07,"TBase64"   

`00473AB8` `. E8 CFF7FFFF`    `CALL 0047328C`------------------------->关键一◎◎◎◎◎   

`00473ABD` `. 8B55 F0`        `MOV EDX,DWORD PTR SS:[EBP-10]`--------->关键一的结果----------从这里往下共有 4 次运算   

`00473AC0` `. B8 643B4700`    `MOV EAX,00473B64`---------------------->是否程序固定("F")   

`00473AC5` `. E8 0A09F9FF`    `CALL 004043D4`------------------------->关键二◎◎◎◎◎   

`00473ACA` `. 85C0`           `TEST EAX,EAX`-------------------------->EAX 作为标志   

`00473ACC` `. 74 37`          `JE SHORT 00473B05`--------------------->为零则错误   

`00473ACE` `. 8B55 F0`        `MOV EDX,DWORD PTR SS:[EBP-10]`--------->关键一的结果   

`00473AD1` `. B8 703B4700`    `MOV EAX,00473B70`---------------------->是否程序固定("S")   

`00473AD6` `. E8 F908F9FF`    `CALL 004043D4`------------------------->关键二◎◎◎◎◎   

`00473ADB` `. 85C0`           `TEST EAX,EAX`-------------------------->EAX 作为标志   

`00473ADD` `. 74 26`          `JE SHORT 00473B05`--------------------->为零则错误   

`00473ADF` `. 8B55 F0`        `MOV EDX,DWORD PTR SS:[EBP-10]`--------->关键一的结果   

`00473AE2` `. B8 7C3B4700`    `MOV EAX,00473B7C`---------------------->是否程序固定("Z")   

`00473AE7` `. E8 E808F9FF`    `CALL 004043D4`------------------------->关键二◎◎◎◎◎   

`00473AEC` `. 85C0`           `TEST EAX,EAX`-------------------------->EAX 作为标志   

`00473AEE` `. 74 15`          `JE SHORT 00473B05`--------------------->为零则错误   

`00473AF0` `. 8B55 F0`        `MOV EDX,DWORD PTR SS:[EBP-10]`--------->关键一的结果   

`00473AF3` `. B8 883B4700`    `MOV EAX,00473B88`---------------------->是否程序固定("L")   

`00473AF8` `. E8 D708F9FF`    `CALL 004043D4`------------------------->关键二◎◎◎◎◎   

`00473AFD` `. 85C0`           `TEST EAX,EAX`-------------------------->EAX 作为标志   

`00473AFF` `. 74 04`          `JE SHORT 00473B05`--------------------->为零则错误   

`00473B01` `. C645 FB 01`     `MOV BYTE PTR SS:[EBP-5],1`   

`00473B05` `> 33C0`           `XOR EAX,EAX`   

`00473B07` `. 5A`             `POP EDX`   

`00473B08` `. 59`             `POP ECX`   

`00473B09` `. 59`             `POP ECX`   

`00473B0A` `. 64:8910`        `MOV DWORD PTR FS:[EAX],EDX`   
