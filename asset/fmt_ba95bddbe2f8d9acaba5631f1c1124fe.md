
# 去除 ExpressFS Server 时间和 NAG

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-52.htm`](https://www.pediy.com/kssd/pediy08/pediy8-52.htm)

**题目**：去除 `ExpressFS Server` 时间和 NAG

**工具**：`Softice`，`ULTRAEDIT`，`PEID`

**目的**：去除 NAG 和时间限制

**引子**：今天又从 2002 电脑爱好者光盘找了一个小软件，就是这个 FTP 服务器，功能倒是没有试用，因为没有局域网环境。这个软件要求注册，试用期 30 天，而且是 `Keyfile` 形式的注册，注册码 128 位，我跟踪了很久发现没有处理这个文件数据的地方。先爆破了再说吧。拿 `PEID` 查看没有加壳，是 `DELPHI` 写的。

下断点 `bpx CreatefileA`，按 `F5` 退出 `SOFTICE`，启动程序，被拦住。按一阵 20 多次 `F12`，然后换 `F10` 跟踪，来到如下代码处：

```
:004627C4 E8532CFEFF            call 0044541C //这个 CALL 显示主窗口。
:004627C9 E8C600FAFF            call 00402894
:004627CE 85C0                  test eax, eax
:004627D0 7E1B                  jle 004627ED    //这里随其跳。
:004627D2 8D55FC                lea edx, dword ptr [ebp-04]
:004627D5 B801000000            mov eax, 00000001
:004627DA E81501FAFF            call 004028F4
:004627DF 8B55FC                mov edx, dword ptr [ebp-04]
:004627E2 8B83C4020000          mov eax, dword ptr [ebx+000002C4]
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00462773(C)
|
:004627E8 E86FBEFFFF            call 0045E65C       //这个 CALL 关键
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004627D0(C)
:004627ED 80BBB505000001        cmp byte ptr [ebx+000005B5], 01   // [ebx+000005B5]里面的值 1 与 1 比较。
:004627F4 7529                  jne 0046281F       // 这里改为 je, 即 75->74。
:004627F6 A1E0884600            mov eax, dword ptr [004688E0]
:004627FB E87CD1FFFF            call 0045F97C
:00462800 83F81E                cmp eax, 0000001E
:00462803 7C15                  jl 0046281A
:00462805 6A00                  push 00000000
:00462807 668B0D94284600        mov cx, word ptr [00462894]
:0046280E 33D2                  xor edx, edx
* Possible StringData Ref from Code Obj ->"ExpressFS Server has expired!"
                        |
:00462810 B8A0284600            mov eax, 004628A0
:00462815 E8AEB7FEFF            call 0044DFC8
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00462803(C)
:0046281A E8A91C0000            call 004644C8      //这个 CALL 出现 NAG。
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004627F4(C)
|:0046281F 80BBB505000000      cmp byte ptr [ebx+000005B5], 00   // [ebx+000005B5]里面的 1 与 0 比较。
:00462826 7416                  je 0046283E        // 此处改为 jne，即 74->75。
:00462828 A1E0884600            mov eax, dword ptr [004688E0]
:0046282D E84AD1FFFF            call 0045F97C
:00462832 83F81E                cmp eax, 0000001E
:00462835 7C07                  jl 0046283E
:00462837 8BC3                  mov eax, ebx
:00462839 E8262AFEFF            call 00445264
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00462826(C), :00462835(C)
|
:0046283E 8BC3                  mov eax, ebx
:00462840 E8EBF3FFFF            call 00461C30
:00462845 84C0                  test al, al
:00462847 741B                  je 00462864        // 此处改为 jne，即 74->75。
:00462849 8B8378030000          mov eax, dword ptr [ebx+00000378]
:0046284F 8B10                  mov edx, dword ptr [eax]
:00462851 FF92B8000000          call dword ptr [edx+000000B8]
:00462857 84C0                  test al, al
:00462859 7409                  je 00462864        // 此处改为 jne，即 74->75。
:0046285B 8BC3                  mov eax, ebx
:0046285D E816030000            call 00462B78
:00462862 EB0E                  jmp 00462872      //这个 CALL 可以跳过后面（*）那个向导过程。
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00462847(C), :00462859(C)
|
:00462864 E82B00FAFF            call 00402894
:00462869 85C0                  test eax, eax
:0046286B 7F05                  jg 00462872     
:0046286D E852220000            call 00464AC4  //这个 CALL 每次启动时都出现配置向导。（*）
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00462862(U), :0046286B(C)
|
:00462872 33C0                  xor eax, eax
:00462874 5A                    pop edx
:00462875 59                    pop ecx
:00462876 59                    pop ecx
:00462877 648910                mov dword ptr fs:[eax], edx
================================================================
```

经过上面的修改，发现启动程序时，窗口稍纵即逝，原来躲在系统托盘里了，不影响使用，右击其图标即可选择命令，打开窗口。我把时间加快一年，结果启动程序时，出现过期提示。下面也解决一下这个问题好了。下断点 `bpx getlocaltime`，按 `F5` 退出 `SOFTICE`，然后启动程序，被拦截。按一次 `F11` 即可回到主程序空间，换 `F10` 跟踪，经过一段代码跟踪后，我们来到下面代码处了，你会看到这段代码跟上面一段是互补的，重叠的，不同时刻调用不同功能，其他障碍都在前面解决了，可是如果过期则使用下面的代码逻辑：

```
004627FB  |. E8 7CD1FFFF    CALL XFSSVR.0045F97C   //这个 CALL 取系统时间。
00462800  |. 83F8 1E        CMP EAX,1E             //EAX 放着使用的天数。
:00462803   7C15            jl 0046281A            //此处因为超期，不会跳转。直接修改为 jg 462872,机器码是 7F 6D。 
:00462805 6A00                  push 00000000
:00462807 668B0D94284600        mov cx, word ptr [00462894]
0046280E  |. 33D2             XOR EDX,EDX                             ; |
00462810  |. B8 A0284600      MOV EAX,XFSSVR.004628A0                 ; |ASCII "ExpressFS Server has expired!
Please register immediately!"
00462815  |. E8 AEB7FEFF      CALL XFSSVR.0044DFC8   //这里就出过期窗口。
0046281A  |> E8 A91C0000      CALL XFSSVR.004644C8
0046281F  |> 80BB B5050000    >CMP BYTE PTR DS:[EBX+5B5],0
00462826  |. 74 16            JE SHORT XFSSVR.0046283E
00462828  |. A1 E0884600      MOV EAX,DWORD PTR DS:[4688E0]
0046282D  |. E8 4AD1FFFF      CALL XFSSVR.0045F97C    
00462832  |. 83F8 1E          CMP EAX,1E
00462835  |. 7C 07            JL SHORT XFSSVR.0046283E
00462837  |. 8BC3             MOV EAX,EBX
00462839  |. E8 262AFEFF      CALL XFSSVR.00445264
0046283E  |> 8BC3             MOV EAX,EBX
00462840  |. E8 EBF3FFFF      CALL XFSSVR.00461C30
00462845  |. 84C0             TEST AL,AL
00462847  |. 74 1B            JE SHORT XFSSVR.00462864
00462849  |. 8B83 78030000   MOV EAX,DWORD PTR DS:[EBX+378]
```

经过上述修改，启动程序发现立即退出了，肯定还有地方没有改掉。继续下同样断点，按 `F11` 一次，然后 `F10` 跟踪到下面代码处，这段代码就是前面这段的后半部分：

```
00462828  |. A1 E0884600      MOV EAX,DWORD PTR DS:[4688E0]
0046282D  |. E8 4AD1FFFF      CALL XFSSVR.0045F97C    //此 CALL 取使用天数。
00462832  |. 83F8 1E          CMP EAX,1E              //与 1E 比较，即 30 天。
00462835  |. 7C 07            JL SHORT XFSSVR.0046283E   //如果超期，这个不跳。修改为 JMP 无论如何也跳. 机器码 EB。 
00462837  |. 8BC3             MOV EAX,EBX
00462839  |. E8 262AFEFF      CALL XFSSVR.00445264  
0046283E  |> 8BC3             MOV EAX,EBX
00462840  |. E8 EBF3FFFF      CALL XFSSVR.00461C30
00462845  |. 84C0             TEST AL,AL
00462847  |. 74 1B            JE SHORT XFSSVR.00462864
================================================================
```

还有一个地方，就是在点击窗口上的 `Launch` 按钮时，出现跟前面提到的一样的 NAG。下面也一并搞掉。下断点 `bpx enablewindow`，按 `F5` 退出 `SOFTICE`，然后点击 `Launch` 按钮，被拦住。这里如果下 `messageboxa` 则拦不住。如果换 `Createwindow` 函数可以拦住。按一次 `F11`，5 次 `F12` 来到下面代码处：

```
00461C70   . FF51 60          CALL DWORD PTR DS:[ECX+60]
00461C73   . 66:BA EDFF       MOV DX,0FFED          //我们停在这里。
00461C77   . 8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]
00461C7A   . E8 05A2FCFF      CALL XFSSVR.0042BE84
00461C7F   . 33D2             XOR EDX,EDX
00461C81   . 55               PUSH EBP
00461C82   . 68 EF214600      PUSH XFSSVR.004621EF
00461C87   . 64:FF32          PUSH DWORD PTR FS:[EDX]
00461C8A   . 64:8922          MOV DWORD PTR FS:[EDX],ESP
00461C8D   . 80FB 01          CMP BL,1
00461C90   . 0F85 19030000    JNZ XFSSVR.00461FAF
00461C96   . 833D DC884600    >CMP DWORD PTR DS:[4688DC],0
00461C9D   . 0F85 0C030000    JNZ XFSSVR.00461FAF
00461CA3   . A1 D8884600      MOV EAX,DWORD PTR DS:[4688D8]
00461CA8   . 80B8 B5050000    >CMP BYTE PTR DS:[EAX+5B5],0
00461CAF   . 74 05            JE SHORT XFSSVR.00461CB6     //刚好这个跳转可以跳过下面这个 CALL。修改 74->EB。
00461CB1   . E8 12280000      CALL XFSSVR.004644C8          //这个 CALL 出现 NAG。
```
