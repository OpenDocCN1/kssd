
# 豪杰大眼睛 Hero Photo ShowV2.1

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-41.htm`](https://www.pediy.com/kssd/pediy08/pediy8-41.htm)

题目：注册图像浏览器“豪杰大眼睛 Hero Photo ShowV2.1”

工具：Softice，PEID

功能简介：独创超级图网，轻松点击看遍互联网；图像处理简单快捷，电子相册，屏保制作；支持图片格式 60 余种，动画图象随意抽取；序列播放，进度随意更改，任意角度看图。

引子：这是继分析“豪杰超级解霸”和“金山影霸”以来的又一个豪杰产品的注册码分析。尽管 Acdsee 是图像浏览器的大腕，尽管大眼睛还有些缺陷，但是我们应该支持我们的民族软件产业。PEID 检查没有加壳，VC 开发。启动程序，输入用户名 `wanggang`，输入注册码 `1111-2222-3333-4444`，打开 softice，下断点 `bpx getwindowtexta`，F5 退出，点击“确定”，被拦住。按一次 F12 来到下面代码处。

```
00402480  /$ 83EC 40         SUB ESP,40
00402483  |. 8B0D 38CC4000 MOV ECX,DWORD PTR DS:[40CC38]
00402489  |. 56             PUSH ESI
0040248A  |. 8B35 98914000 MOV ESI,DWORD PTR DS:[<&USER32.GetWindow>
00402490  |. 8D4424 04     LEA EAX,DWORD PTR SS:[ESP+4]
00402494  |. 6A 08          PUSH 8
00402496  |. 50             PUSH EAX
00402497  |. 51             PUSH ECX
00402498  |. FFD6           CALL ESI             //取注册码第一段。
0040249A  |. A1 34CC4000    MOV EAX,DWORD PTR DS:[40CC34]
0040249F  |. 8D5424 09     LEA EDX,DWORD PTR SS:[ESP+9]
004024A3  |. 6A 08          PUSH 8
004024A5  |. 52             PUSH EDX
004024A6  |. 50             PUSH EAX
004024A7  |. FFD6           CALL ESI             //取注册码第二段。
004024A9  |. 8B15 40CC4000 MOV EDX,DWORD PTR DS:[40CC40]
004024AF  |. 8D4C24 0E     LEA ECX,DWORD PTR SS:[ESP+E]
004024B3  |. 6A 08          PUSH 8
004024B5  |. 51             PUSH ECX
004024B6  |. 52             PUSH EDX
004024B7  |. FFD6           CALL ESI            //取注册码第三段。
004024B9  |. 8B0D 3CCC4000 MOV ECX,DWORD PTR DS:[40CC3C]
004024BF  |. 8D4424 13     LEA EAX,DWORD PTR SS:[ESP+13]
004024C3  |. 6A 08          PUSH 8
004024C5  |. 50             PUSH EAX
004024C6  |. 51             PUSH ECX
004024C7  |. FFD6           CALL ESI           //取注册码第四段。
004024C9  |. 8B15 30CC4000 MOV EDX,DWORD PTR DS:[40CC30]
004024CF  |. 68 00010000    PUSH 100
004024D4  |. B0 2D          MOV AL,2D          //2D 就是减号'-'。
004024D6  |. 68 80CD4000    PUSH AUTHREG.0040CD80
004024DB  |. 52             PUSH EDX
004024DC  |. 884424 1E      MOV BYTE PTR SS:[ESP+1E],AL   //减号分别放到输入的注册码内。隔 4 位一个。
004024E0  |. 884424 19      MOV BYTE PTR SS:[ESP+19],AL
004024E4  |. 884424 14      MOV BYTE PTR SS:[ESP+14],AL
004024E8  |. C64424 23 00   MOV BYTE PTR SS:[ESP+23],0
004024ED  |. FFD6           CALL ESI           //取用户名。
004024EF  |. A1 58C74000    MOV EAX,DWORD PTR DS:[40C758]
004024F4  |. 5E             POP ESI
004024F5  |. 85C0           TEST EAX,EAX
004024F7  |. 74 0E          JE SHORT AUTHREG.00402507
004024F9  |. 8D4C24 00     LEA ECX,DWORD PTR SS:[ESP]
004024FD  |. 51             PUSH ECX
004024FE  |. 68 80CD4000    PUSH AUTHREG.0040CD80
00402503  |. FFD0           CALL EAX                   //这个函数就是计算注册码的地方。（*）
00402505  |. EB 0F          JMP SHORT AUTHREG.00402516   //这里跳走。
00402507  |> 8D5424 00     LEA EDX,DWORD PTR SS:[ESP]
0040250B  |. 52             PUSH EDX
0040250C  |. 68 80CD4000    PUSH AUTHREG.0040CD80
00402511  |. E8 DA140000    CALL AUTHREG.004039F0
00402516  |> 33C9           XOR ECX,ECX
00402518  |. 8D5424 00     LEA EDX,DWORD PTR SS:[ESP]
0040251C  |. 85C0           TEST EAX,EAX                 //如果注册码正确则 EAX 返回 1。这里测试 EAX 是否非 0。
0040251E  |. 0F95C1         SETNE CL                     //如果 EAX 非 0，让 CL=1。
00402521  |. 52             PUSH EDX
00402522  |. 68 80CD4000    PUSH AUTHREG.0040CD80
00402527  |. 890D 40F84000  MOV DWORD PTR DS:[40F840],ECX
0040252D  |. E8 2E000000    CALL AUTHREG.00402560        //加密注册码。
00402532  |. 8B4C24 4C      MOV ECX,DWORD PTR SS:[ESP+4C]
00402536  |. 8B15 64CD4000 MOV EDX,DWORD PTR DS:[40CD64]
0040253C  |. 83C4 08        ADD ESP,8
0040253F  |. 8D4424 00     LEA EAX,DWORD PTR SS:[ESP]
00402543  |. 50             PUSH EAX
00402544  |. 68 E0234000    PUSH AUTHREG.004023E0
00402549  |. 51             PUSH ECX
0040254A  |. 6A 69          PUSH 69
0040254C  |. 52             PUSH EDX
0040254D  |. FF15 C4914000  CALL DWORD PTR DS:[<&USER32.DialogBoxPar>  //显示成功注册信息。
00402553  |. A1 40F84000    MOV EAX,DWORD PTR DS:[40F840]
00402558  |. 83C4 40        ADD ESP,40
0040255B  \. C3             RETN
```

======================================================================

下面进入（*）处的函数分析：

```
:100010A0 83EC20              sub esp, 00000020
:100010A3 56                  push esi
:100010A4 57                  push edi
:100010A5 B908000000          mov ecx, 00000008
:100010AA 33C0                xor eax, eax
:100010AC 8D7C2408            lea edi, dword ptr [esp+08]
:100010B0 F3                  repz
:100010B1 AB                  stosd
:100010B2 8B44242C            mov eax, dword ptr [esp+2C]
:100010B6 50                  push eax
:100010B7 E884010000          call 10001240             //这个函数处理用户名 （**）
:100010BC 83C404              add esp, 00000004
:100010BF 89442408            mov dword ptr [esp+08], eax
:100010C3 33F6                xor esi, esi
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:100010E8(C)
|
:100010C5 0FBE443408          movsx eax, byte ptr [esp+esi+08]   //依次取前面最后一次得到的 4 位用户名字符送 EAX。
:100010CA 83F841              cmp eax, 00000041                 //与 41 比较。
:100010CD 7C08                jl 100010D7                       //如果小于 41，则进行处理。
:100010CF 83F85A              cmp eax, 0000005A                 //与 5A 比较
:100010D2 7F03                jg 100010D7                       //如果大于 5A，则进行处理。
:100010D4 83C020              add eax, 00000020                 //否则加 20，把大写字符变为小写字符。
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:100010CD(C), :100010D2(C)
|
:100010D7 50                  push eax
:100010D8 E813020000          call 100012F0       //这个函数把处理过的用户名转换为可见字符。（***）
:100010DD 83C404              add esp, 00000004
:100010E0 88443408            mov byte ptr [esp+esi+08], al
:100010E4 46                  inc esi
:100010E5 83FE04              cmp esi, 00000004
:100010E8 7CDB                jl 100010C5
:100010EA 8B7C2430            mov edi, dword ptr [esp+30]    //注册码地址送 EDI。
:100010EE 8D4C2408            lea ecx, dword ptr [esp+08]    //变换后的注册码地址送 ECX。
:100010F2 8BF7                mov esi, edi
:100010F4 33D2                xor edx, edx
:100010F6 2BF1                sub esi, ecx
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1000111C(C)
|
:100010F8 8D4C1408            lea ecx, dword ptr [esp+edx+08]
:100010FC 0FBE040E            movsx eax, byte ptr [esi+ecx]//注册码逐位送 EAX。
:10001100 83F841              cmp eax, 00000041
:10001103 7C08                jl 1000110D
:10001105 83F85A              cmp eax, 0000005A
:10001108 7F03                jg 1000110D
:1000110A 83C020              add eax, 00000020
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:10001103(C), :10001108(C)
|
:1000110D 0FBE09              movsx ecx, byte ptr [ecx]   //第一组真码依次送 ECX。
:10001110 3BC1                cmp eax, ecx                //真假比较。这里就应该用 E 命令修改你的假码为真码了。以下类推。
:10001112 0F8514010000        jne 1000122C                //不等则 OVER。
:10001118 42                  inc edx
:10001119 83FA04              cmp edx, 00000004
:1000111C 7CDA                jl 100010F8                 //没有比较完，则继续。
:1000111E 8B442408            mov eax, dword ptr [esp+08]  //变换后的用户名 4 位送 EAX。
:10001122 8D5008              lea edx, dword ptr [eax+08]
```



:10001125 `0FAFD0` `imul edx, eax` //EDX=EDX\*EAX。  

:10001128 `8954240C` `mov dword ptr [esp+0C], edx` //结果送到[ESP+C]处。  

:1000112C `33F6` `xor esi, esi`  

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

| :1000114F(C)  

|  

:1000112E `8A44340C` `mov al, byte ptr [esp+esi+0C]`  

:10001132 `50` `push eax`  

:10001133 `56` `push esi`  

:10001134 `E847030000` `call 10001480` //这个函数把变换得到的新 4 位注册码进一步变换。（\*\*\*\*）  

:10001139 `25FF000000` `and eax, 000000FF`  

:1000113E `50` `push eax`  

:1000113F `E8AC010000` `call 100012F0` //变为可见字符。  

:10001144 `83C40C` `add esp, 0000000C`  

:10001147 `8844340C` `mov byte ptr [esp+esi+0C], al`  

:1000114B `46` `inc esi`  

:1000114C `83FE04` `cmp esi, 00000004`  

:1000114F `7CDD` `jl 1000112E`  

:10001151 `33C9` `xor ecx, ecx`  

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

| :10001176(C)  

|  

:10001153 `0FBE440F05` `movsx eax, byte ptr [edi+ecx+05]` //下面是比较真假注册码的循环。  

:10001158 `83F841` `cmp eax, 00000041`  

:1000115B `7C08` `jl 10001165`  

:1000115D `83F85A` `cmp eax, 0000005A`  

:10001160 `7F03` `jg 10001165`  

:10001162 `83C020` `add eax, 00000020`  

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  

| :1000115B(C), :10001160(C)  

|  

:10001165 `0FBE540C0C` `movsx edx, byte ptr [esp+ecx+0C]`  

:1000116A `3BC2` `cmp eax, edx`  

:1000116C `0F85BA000000` `jne 1000122C`  

:10001172 `41` `inc ecx`  

:10001173 `83F904` `cmp ecx, 00000004`  

:10001176 `7CDB` `jl 10001153`  

:10001178 `8B44240C` `mov eax, dword ptr [esp+0C]` //第一段注册码送 EAX。  

:1000117C `8B4C2408` `mov ecx, dword ptr [esp+08]` //第二段注册码送 ECX。  

:10001180 `8BD0` `mov edx, eax`  

:10001182 `33D1` `xor edx, ecx` //异或运算  

:10001184 `42` `inc edx` //加 1。  

:10001185 `0FAFD1` `imul edx, ecx` //EDX=EDX\*ECX  

:10001188 `03D0` `add edx, eax` //EDX=EDX+EAX。  

:1000118A `33F6` `xor esi, esi`  

:1000118C `89542410` `mov dword ptr [esp+10], edx` //保存结果。  

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

| :100011A6(C)  

|  

:10001190 `0FBE443410` `movsx eax, byte ptr [esp+esi+10]`  

:10001195 `50` `push eax`  

:10001196 `E855010000` `call 100012F0` //处理刚刚得到的 4 位结果。  

:1000119B `83C404` `add esp, 00000004`  

:1000119E `88443410` `mov byte ptr [esp+esi+10], al` //保存  

:100011A2 `46` `inc esi`  

:100011A3 `83FE04` `cmp esi, 00000004`  

:100011A6 `7CE8` `jl 10001190`  

:100011A8 `33C9` `xor ecx, ecx`  

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

| :100011C9(C)  

|  

:100011AA `0FBE440F0A` `movsx eax, byte ptr [edi+ecx+0A]` //下面是比较第三段注册码的循环。  

:100011AF `83F841` `cmp eax, 00000041`  

:100011B2 `7C08` `jl 100011BC`  

:100011B4 `83F85A` `cmp eax, 0000005A`  

:100011B7 `7F03` `jg 100011BC`  

:100011B9 `83C020` `add eax, 00000020`  

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  

| :100011B2(C), :100011B7(C)  

|  

:100011BC `0FBE540C10` `movsx edx, byte ptr [esp+ecx+10]`  

:100011C1 `3BC2` `cmp eax, edx`  

:100011C3 `7567` `jne 1000122C`  

:100011C5 `41` `inc ecx`  

:100011C6 `83F904` `cmp ecx, 00000004`  

:100011C9 `7CDF` `jl 100011AA`  

:100011CB `8B4C240C` `mov ecx, dword ptr [esp+0C]` //第二段注册码送 ECX。  

:100011CF `8B442408` `mov eax, dword ptr [esp+08]` //第一段注册码送 EAX。  

:100011D3 `0FAFC8` `imul ecx, eax` //相乘  

:100011D6 `41` `inc ecx` //ECX 增一。  

:100011D7 `0FAF4C2410` `imul ecx, dword ptr [esp+10]` //ECX 与第三段相乘。  

:100011DC `03C8` `add ecx, eax` //ECX 加 EAX。  

:100011DE `33F6` `xor esi, esi`  

:100011E0 `894C2414` `mov dword ptr [esp+14], ecx` //保存。  

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

| :100011FA(C)  

|  

:100011E4 `0FBE543414` `movsx edx, byte ptr [esp+esi+14]` //处理刚刚得到的第四段注册码。  

:100011E9 `52` `push edx`  

:100011EA `E801010000` `call 100012F0`  

:100011EF `83C404` `add esp, 00000004`  

:100011F2 `88443414` `mov byte ptr [esp+esi+14], al`  

:100011F6 `46` `inc esi`  

:100011F7 `83FE04` `cmp esi, 00000004`  

:100011FA `7CE8` `jl 100011E4`  

:100011FC `33C9` `xor ecx, ecx`  

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

| :1000121D(C)  

|  

:100011FE `0FBE440F0F` `movsx eax, byte ptr [edi+ecx+0F]` //下面是第四段注册码的比较循环。  

:10001203 `83F841` `cmp eax, 00000041`  

:10001206 `7C08` `jl 10001210`  

:10001208 `83F85A` `cmp eax, 0000005A`  

:1000120B `7F03` `jg 10001210`  

:1000120D `83C020` `add eax, 00000020`  

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  

| :10001206(C), :1000120B(C)  

|  

:10001210 `0FBE540C14` `movsx edx, byte ptr [esp+ecx+14]`  

:10001215 `3BC2` `cmp eax, edx`  

:10001217 `7513` `jne 1000122C`  

:10001219 `41` `inc ecx`  

:1000121A `83F904` `cmp ecx, 00000004`  

:1000121D `7CDF` `jl 100011FE`  

:1000121F `5F` `pop edi`  

:10001220 `B801000000` `mov eax, 00000001` //注册码正确则让 EAX=1。  

:10001225 `5E` `pop esi`  

:10001226 `83C420` `add esp, 00000020`  

:10001229 `C20800` `ret 0008`  

======================================================================  

下面分析 :100010B7 处的函数 `call 10001240`：  

| :100010B7  

|  

:10001240 `81EC00020000` `sub esp, 00000200`  

:10001246 `B980000000` `mov ecx, 00000080`  

:1000124B `33C0` `xor eax, eax`  

:1000124D `53` `push ebx`  

:1000124E `55` `push ebp`  

:1000124F `56` `push esi`  

:10001250 `8BB42410020000` `mov esi, dword ptr [esp+00000210]`  

:10001257 `57` `push edi`  

:10001258 `8D7C2410` `lea edi, dword ptr [esp+10]`  

:1000125C `F3` `repz`  

:1000125D `AB` `stosd`  

:1000125E `56` `push esi`  

:1000125F `33ED` `xor ebp, ebp`  

\* Reference To: `KERNEL32.lstrlenA`, Ord:0308h  

|  

:10001261 `FF1504600010` `Call dword ptr [10006004]` //求串长。  

:10001267 `8BD8` `mov ebx, eax`  

:10001269 `81FB00020000` `cmp ebx, 00000200`  

:1000126F `7612` `jbe 10001283`  

:10001271 `B980000000` `mov ecx, 00000080`  

:10001276 `8D7C2410` `lea edi, dword ptr [esp+10]`  

:1000127A `BB00020000` `mov ebx, 00000200`  

:1000127F `F3` `repz`  

:10001280 `A5` `movsd`  

:10001281 `EB0C` `jmp 1000128F`  

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

| :1000126F(C)  

|  



### 汇编代码分析

```asm
:10001283 8D442410          lea eax, dword ptr [esp+10]

:10001287 56                push esi

:10001288 50                push eax

* Reference To: KERNEL32.lstrcpyA, Ord:0302h

                        |

:10001289 FF1584600010      Call dword ptr [10006084]   //串拷贝。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001281(U)
|

:1000128F 8BC3              mov eax, ebx                //用户名串长送 EAX。

:10001291 99                cdq

:10001292 83E203            and edx, 00000003
:10001295 03C2              add eax, edx

:10001297 8BF8              mov edi, eax                //串长送 EDI。

:10001299 C1FF02            sar edi, 02                 //右循环 2 次，相当于除以 4。

:1000129C F6C303            test bl, 03

:1000129F 7401              je 100012A2                 //此处跳走。

:100012A1 47                inc edi

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1000129F(C)
|

:100012A2 33C9              xor ecx, ecx
:100012A4 85DB              test ebx, ebx

:100012A6 7E17              jle 100012BF

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:100012BD(C)
|

:100012A8 8A540C10          mov dl, byte ptr [esp+ecx+10]  //依次取用户名送 DL。

:100012AC 52                push edx
:100012AD 51                push ecx
:100012AE E8CD010000        call 10001480                   //这个函数变换每位字符。

:100012B3 83C408            add esp, 00000008
:100012B6 88440C10          mov byte ptr [esp+ecx+10], al //转换后的字符送原来位置。

:100012BA 41                inc ecx
:100012BB 3BCB              cmp ecx, ebx

:100012BD 7CE9              jl 100012A8                   //循环直至处理完用户名。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:100012A6(C)
|

:100012BF 33F6              xor esi, esi
:100012C1 85FF              test edi, edi
:100012C3 7E1C              jle 100012E1

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:100012DF(C)
|

:100012C5 8B5CB410          mov ebx, dword ptr [esp+4*esi+10]  //转换完的前 4 位送 EBX。

:100012C9 8BC6              mov eax, esi                    //ESI 值送 EAX。ESI 初始值是 0。

:100012CB 83E01F            and eax, 0000001F
:100012CE 03EB              add ebp, ebx
:100012D0 50                push eax                    //循环移位次数参数进栈。
:100012D1 55                push ebp                    //四位用户名字符参数进栈。
:100012D2 E879020000        call 10001550                   //把用户名循环移位 EAX 次。

:100012D7 83C408            add esp, 00000008
:100012DA 46                inc esi
:100012DB 3BF7              cmp esi, edi                  //EDI 为循环次数。
:100012DD 8BE8              mov ebp, eax                  //循环移位后的结果送 EBP。

:100012DF 7CE4              jl 100012C5                   //没有结束，则继续循环。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:100012C3(C)
|

:100012E1 5F                pop edi
:100012E2 8BC5              mov eax, ebp        //把最后一次得到的 4 位用户名送 EAX 返回主调函数。

:100012E4 5E                pop esi
:100012E5 5D                pop ebp
:100012E6 5B                pop ebx
:100012E7 81C400020000      add esp, 00000200
:100012ED C3                ret
```

下面是 `:100010D8` 处的函数 `call 100012F0` 的代码（***）：

```asm
:100012F0 8B442404          mov eax, dword ptr [esp+04]

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:10001312(U), :10001323(U), :1000133E(U)
|

:100012F4 83E07F            and eax, 0000007F
:100012F7 83F841            cmp eax, 00000041   //判断是否是在字符'A'与'Z'之间。

:100012FA 7C07              jl 10001303

:100012FC 83F85A            cmp eax, 0000005A
:100012FF 7F02              jg 10001303          //如果不在 A-Z 之间，则跳。

:10001301 0C20              or al, 20              //如果在 A-Z 之间，则变为小写字符。

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:100012FA(C), :100012FF(C)
|

:10001303 83F86F            cmp eax, 0000006F  //判断是否是字符'o'。

:10001306 750C              jne 10001314
:10001308 B890000000        mov eax, 00000090
:1000130D 83F00E            xor eax, 0000000E
:10001310 0C31              or al, 31
:10001312 EBE0              jmp 100012F4

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001306(C)
|

:10001314 83F830            cmp eax, 00000030  //判断是否是 0。

:10001317 750C              jne 10001325       //如果不是则跳。
:10001319 B8CF000000        mov eax, 000000CF
:1000131E 83F00E            xor eax, 0000000E
:10001321 0C31              or al, 31
:10001323 EBCF              jmp 100012F4

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001317(C)
|

:10001325 83F861            cmp eax, 00000061  //判断是否在 a-z 之间。

:10001328 7C05              jl 1000132F        //如果小于 a 则跳。
:1000132A 83F87A            cmp eax, 0000007A
:1000132D 7E11              jle 10001340       //如果在 a-z 之间则 OK。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001328(C)
|

:1000132F 83F831            cmp eax, 00000031 //判断是否在 1-9 之间。

:10001332 7C05              jl 10001339
:10001334 83F839            cmp eax, 00000039
:10001337 7E07              jle 10001340       //如果在 1-9 之间也 OK 。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001332(C)
|

:10001339 83F00E            xor eax, 0000000E  //如果小于 1 则与 E 异或。

:1000133C 0C31              or al, 31          //加 31。
:1000133E EBB4              jmp 100012F4

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:1000132D(C), :10001337(C)
|

:10001340 C3                ret
```

下面是（*****）处的函数调用分析：`:10001134 call 10001480`

```asm
:10001480 55                push ebp
:10001481 8BEC              mov ebp, esp
:10001483 53                push ebx
:10001484 8A4508            mov al, byte ptr [ebp+08]
:10001487 8A5D0C            mov bl, byte ptr [ebp+0C] //取每位注册码送 BL。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001494(C)
|

:1000148A F6C3C3            test bl, C3               //与 C3（1100 0011）进行"与"运算。

:1000148D 7A01              jpe 10001490              //如果为偶数个 1 则跳。
:1000148F F9                stc                       //否则置进位标志为 1。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1000148D(C)
|

:10001490 D0DB              rcr bl, 1                 //BL 带进位 C 右循环一次。

:10001492 FEC8              dec al                    //计数器减一。
:10001494 75F4              jne 1000148A              //继续循环。

:10001496 885D0C            mov byte ptr [ebp+0C], bl //循环完毕，BL 保存。

:10001499 8A450C            mov al, byte ptr [ebp+0C] //送 AL 返回主调函数。

:1000149C 5B                pop ebx
:1000149D 5D                pop ebp
:1000149E C3                ret
```

### 后记

我不知道豪杰是否一直采用这样的注册码生成算法，不过这些老软件倒是基本一样的思路，他们的新版本我没有得到，暂时无法尝试。本文只针对菜鸟，大侠切莫见笑！！恭祝各位坛友新春愉快！万事如意！

### 结论

*   用户名：`wanggang`
*   注册码：`7zq3-d53t-pht3-gxva`

完稿日期：2006/1/28 Ehiopia Qduwg



`4dd466d0e1e0cc123afbe504da2462ca`
