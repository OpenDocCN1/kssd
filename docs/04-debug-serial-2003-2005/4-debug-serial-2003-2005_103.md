
# Dr. Hardware 2002 Build 3.5.0e

> 原文：[`www.pediy.com/kssd/pediy05/pediy50265.htm`](https://www.pediy.com/kssd/pediy05/pediy50265.htm)

Dr. Hardware 2002 破文

- **软件名称**：Dr. Hardware 2002
- **发布公司**：Gebhard Software
- **软件分类**：系统工具
- **发布日期**：2002.07.19
- **文件大小**：2258K
- **版本**：Build 3.5.0e
- **语言**：english
- **软件大小**：
- **软件简介**：好用的，界面友好的系统诊测软件，让你对自己的电脑配置一目了然：硬件系统，各种设备，操作系统，串口与并口信息，鼠标，游戏杆，驱动器，主板，多媒体配置，`CMOS`，以及内存的详细信息一览无余。PC Test & Benchmark Program for Windows 2000/NT4/XP/95/98/ME
- **下载网址**：http://www.dr-hardware.com
- **注册金额**：分不同的版本，最低$19，最高$49（好贵啊）
- **pj 人**：胖大兵[DFCG]
- **使用工具**：
    - 侦测壳类：`fi2.5`, `PEid0.8` 中文版
    - 反编译：`w32dasm` 黄金版中文版
    - 动态调试：`Ollydbg` 中文版 v1.04
    - 特殊工具：笔和纸   :-)

## 过程

首先安装软件，安装程序是一个文件，正常安装。

执行安装程序，程序很好用，对计算机的详细的情况都有很详细的说明！就是不支持打印！好象是！打开注册的信息，注册的部分有两个过程。一个是注册（`register`)，一个是解锁（`unlock`）。前一个是各种注册的途径，没有实际的用途。后一个才是我们要用的，注册的窗体有两个部分，难道注册码是以 `XXXXXX-XXXXXXX` 的形式的吗？前填上一个简单的数字，`9898989898-98989898`，点注册以后，报错！点确定以后，前面的 `98989898` 没有了，后面的一段还在！没有改变。重新用不同的数字和字符串填写，总是后面的不变！分析以后，后面的很有可能是注册的名字！所以不变！主要是前面的数字是注册码！

每一段的数字的限制长度为 14 位。

程序在没有注册的时候只能使用 10 天。

好了！已经知道了这么多！现在开始！`go,go,go.`

先看看程序有没有家壳，使用 `fi2.5`，`PEid0.8` 中文版检查没有加壳！（真是太爽了，作者就是好。这两天一直在手动脱壳，我都快崩溃了），程序是 `C++` 编的！（这个还是很难的，要是加 `aspr` 的壳）

好了！没有壳的话，就可以直接做的！使用 `w32dasm` 黄金版中文版进行反编译，因为主程序很大，要 4M 多，编译需要时间。我们在 `QQ` 聊天一会！哈哈！。。。。。。。。。。。。。。。。。

经过的不长的时间，我的机器相对快一点了！呵呵！程序反编译，点（串式参考），查找我们刚才 `unlock` 时候报错的那个信息，如果你用的是这一版的 `w32dasm` 黄金版中文版的话！会很麻烦，因为他的串式很多，不过我建议大家可以使用 `kW32Dsm(20020825)` 这个修改过的，在他的里面增加了一个搜索的控件真是!好用！呵呵！

"ok"

"okay"

"OKI"

"ole32.dll"

"OleCreateFontIndirect"

"OleCreatePictureIndirect"

"OleCreatePropertyFrame"

"OleLoadPicture"

"olepro32.dll"

"on"

"Once you have received your key "

"Online registration failed."《===========在这一句上双击来到反汇编的地方！

"Online via Internet"

"online"

"Ope&n report or other file..."

"Open file"

"open"

### 双击后的汇编代码

```
:004121C6 90                   nop
:004121C7 90                   nop
:004121C8 55                   push ebp
:004121C9 8BEC                 mov ebp, esp
:004121CB 83C4D8               add esp, FFFFFFD8
:004121CE 53                   push ebx
:004121CF 56                   push esi
:004121D0 57                   push edi
:004121D1 8BD8                 mov ebx, eax
:004121D3 B80C3C6300           mov eax, 00633C0C
:004121D8 E8076F1E00           call 005F90E4〈===========关键比较（A）
:004121DD 66C745E80800         mov [ebp-18], 0008
:004121E3 E89C9E0400           call 0045C084
:004121E8 84C0                 test al, al
:004121EA 752F                 jne 0041221B〈=================关键跳转，暴力 pj 更改。
:004121EC 66C745E81400         mov [ebp-18], 0014

* Possible StringData Ref from Data Obj ->"Online registration failed."
                          |
:004121F2 BAE2816600           mov edx, 006681E2
:004121F7 8D45FC               lea eax, dword ptr [ebp-04]
:004121FA E8DD321F00           call 006054DC
:004121FF FF45F4               inc [ebp-0C]
:00412202 8B00                 mov eax, dword ptr [eax]
:00412204 E80BCD1900           call 005AEF14
:00412209 FF4DF4               dec [ebp-0C]
:0041220C 8D45FC               lea eax, dword ptr [ebp-04]
:0041220F BA02000000           mov edx, 00000002
:00412214 E80F341F00           call 00605628
:00412219 EB1A                 jmp 00412235
```

### 跟进关键 CALL

```
:0045C084 55                   push ebp
:0045C085 8BEC                 mov ebp, esp
:0045C087 81C4BCFEFFFF         add esp, FFFFFEBC
:0045C08D 53                   push ebx
:0045C08E 56                   push esi
:0045C08F 57                   push edi
:0045C090 BB08E26700           mov ebx, 0067E208
:0045C095 BE1C2D6800           mov esi, 00682D1C
:0045C09A B8CCAF6400           mov eax, 0064AFCC
:0045C09F E840D01900           call 005F90E4
:0045C0A4 8B1500496400         mov edx, dword ptr [00644900]
:0045C0AA 8995BCFEFFFF         mov dword ptr [ebp+FFFFFEBC], edx
:0045C0B0 8B1504496400         mov edx, dword ptr [00644904]
:0045C0B6 8995C0FEFFFF         mov dword ptr [ebp+FFFFFEC0], edx
:0045C0BC 668B1508496400       mov dx, word ptr [00644908]
:0045C0C3 668995C4FEFFFF       mov word ptr [ebp+FFFFFEC4], dx
:0045C0CA 8A150A496400         mov dl, byte ptr [0064490A]
:0045C0D0 8895C6FEFFFF         mov byte ptr [ebp+FFFFFEC6], dl
:0045C0D6 8B0D0B496400         mov ecx, dword ptr [0064490B]
:0045C0DC 898D4CFFFFFF         mov dword ptr [ebp+FFFFFF4C], ecx

* Possible StringData Ref from Data Obj ->"9X"
                          |
:0045C0E2 8B0510496400         mov eax, dword ptr [00644910]
:0045C0E8 898540FFFFFF         mov dword ptr [ebp+FFFFFF40], eax

* Possible StringData Ref from Data Obj ->"NT"
                          |
:0045C0EE 8B0514496400         mov eax, dword ptr [00644914]
:0045C0F4 898544FFFFFF         mov dword ptr [ebp+FFFFFF44], eax

* Possible StringData Ref from Data Obj ->"PR"
                          |
:0045C0FA 8B0518496400         mov eax, dword ptr [00644918]
:0045C100 898548FFFFFF         mov dword ptr [ebp+FFFFFF48], eax
:0045C106 C68537FFFFFF00       mov byte ptr [ebp+FFFFFF37], 00
:0045C10D 33D2                 xor edx, edx
:0045C10F 89952CFFFFFF         mov dword ptr [ebp+FFFFFF2C], edx
:0045C115 899530FFFFFF         mov dword ptr [ebp+FFFFFF30], edx
:0045C11B 33C9                 xor ecx, ecx
:0045C11D 8A4B1F               mov cl, byte ptr [ebx+1F]
:0045C120 49                   dec ecx
:0045C121 752C                 jne 0045C14F
:0045C123 33C0                 xor eax, eax
:0045C125 8A4320               mov al, byte ptr [ebx+20]
:0045C128 48                   dec eax
:0045C129 7524                 jne 0045C14F
:0045C12B 33D2                 xor edx, edx
:0045C12D 8A5321               mov dl, byte ptr [ebx+21]
:0045C130 4A                   dec edx
:0045C131 751C                 jne 0045C14F
:0045C133 33C9                 xor ecx, ecx
:0045C135 8A4B22               mov cl, byte ptr [ebx+22]
:0045C138 49                   dec ecx
:0045C139 7514                 jne 0045C14F
:0045C13B B001                 mov al, 01
:0045C13D 8B9550FFFFFF         mov edx, dword ptr [ebp+FFFFFF50]
:0045C143 64891500000000       mov dword ptr fs:[00000000], edx
:0045C14A E9360F0000           jmp 0045D085

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0045C121(C), :0045C129(C), :0045C131(C), :0045C139(C)
|
:0045C14F 66C78560FFFFFF0800   mov word ptr [ebp+FFFFFF60], 0008
:0045C158 8B0D04E26700         mov ecx, dword ptr [0067E204]
:0045C15E 8B81A4060000         mov eax, dword ptr [ecx+000006A4]
:0045C164 B201                 mov dl, 01
:0045C166 8B08                 mov ecx, dword ptr [eax]
:0045C168 FF9120010000         call dword ptr [ecx+00000120]
:0045C16E A104E26700           mov eax, dword ptr [0067E204]
:0045C173 8B80A4060000         mov eax, dword ptr [eax+000006A4]
:0045C179 E872A01000           call 005661F0
:0045C17E 66C78560FFFFFF1400   mov word ptr [ebp+FFFFFF60], 0014

* Possible StringData Ref from Data Obj ->"REGKEY"
                          |
:0045C187 BA0A686400           mov edx, 0064680A
:0045C18C 8D45FC               lea eax, dword ptr [ebp-04]
:0045C18F E848931A00           call 006054DC
:0045C194 FF856CFFFFFF         inc dword ptr [ebp+FFFFFF6C]
:0045C19A 8B10                 mov edx, dword ptr [eax]
:0045C19C 8B0D04E26700         mov ecx, dword ptr [0067E204]
:0045C1A2 8B81A4060000         mov eax, dword ptr [ecx+000006A4]
```



```
:0045C1A8 E8EB901000           call 00565298  
:0045C1AD 898514FFFFFF        mov dword ptr [ebp+FFFFFF14], eax  
:0045C1B3 33D2                xor edx, edx  
:0045C1B5 8955F8              mov dword ptr [ebp-08], edx  
:0045C1B8 8D55F8              lea edx, dword ptr [ebp-08]  
:0045C1BB FF856CFFFFFF        inc dword ptr [ebp+FFFFFF6C]  
:0045C1C1 8B8514FFFFFF        mov eax, dword ptr [ebp+FFFFFF14]  
:0045C1C7 8B08                mov ecx, dword ptr [eax]  
:0045C1C9 FF5158              call [ecx+58]  
:0045C1CC 837DF800            cmp dword ptr [ebp-08], 00000000  
:0045C1D0 7405                je 0045C1D7  
:0045C1D2 8B45F8              mov eax, dword ptr [ebp-08]  
:0045C1D5 EB05                jmp 0045C1DC
* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0045C1D0(C)  
|  
:0045C1D7 B811686400          mov eax, 00646811
* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0045C1D5(U)  
|  
:0045C1DC 85C0                test eax, eax  
:0045C1DE 0F94C2              sete dl  
:0045C1E1 83E201             and edx, 00000001  
:0045C1E4 52                  push edx  
:0045C1E5 FF8D6CFFFFFF        dec dword ptr [ebp+FFFFFF6C]  
:0045C1EB 8D45F8              lea eax, dword ptr [ebp-08]  
:0045C1EE BA02000000          mov edx, 00000002  
:0045C1F3 E830941A00          call 00605628  
:0045C1F8 FF8D6CFFFFFF        dec dword ptr [ebp+FFFFFF6C]  
:0045C1FE 8D45FC              lea eax, dword ptr [ebp-04]  
:0045C201 BA02000000          mov edx, 00000002  
:0045C206 E81D941A00          call 00605628  
:0045C20B 59                  pop ecx  
:0045C20C 84C9                test cl, cl  
:0045C20E 0F856A060000        jne 0045C87E  
:0045C214 66C78560FFFFFF2000  mov word ptr [ebp+FFFFFF60], 0020
* Possible StringData Ref from Data Obj ->"REGKEY"  
                            |  
:0045C21D BA12686400          mov edx, 00646812  
:0045C222 8D45F4              lea eax, dword ptr [ebp-0C]  
:0045C225 E8B2921A00          call 006054DC  
:0045C22A FF856CFFFFFF        inc dword ptr [ebp+FFFFFF6C]  
:0045C230 8B10                mov edx, dword ptr [eax]  
:0045C232 A104E26700          mov eax, dword ptr [0067E204]  
:0045C237 8B80A4060000        mov eax, dword ptr [eax+000006A4]  
:0045C23D E856901000          call 00565298  
:0045C242 898510FFFFFF        mov dword ptr [ebp+FFFFFF10], eax  
:0045C248 33D2                xor edx, edx  
:0045C24A 8955F0              mov dword ptr [ebp-10], edx  
:0045C24D 8D55F0              lea edx, dword ptr [ebp-10]  
:0045C250 FF856CFFFFFF        inc dword ptr [ebp+FFFFFF6C]  
:0045C256 8B8510FFFFFF        mov eax, dword ptr [ebp+FFFFFF10]  
:0045C25C 8B08                mov ecx, dword ptr [eax]  
:0045C25E FF5158              call [ecx+58]  
:0045C261 837DF000            cmp dword ptr [ebp-10], 00000000  
:0045C265 7405                je 0045C26C  
:0045C267 8B45F0              mov eax, dword ptr [ebp-10]  
:0045C26A EB05                jmp 0045C271  
...
```

还有很多！我一直看了个莫名其妙！换动态调试（这个方法也适合其他的 `pj`，`2000` 下不能使用 `trw`，所以用这个方法会比较简单一点）

执行 `Ollydbg` 中文版 `v1.04`，加载 `Dr. Hardware 2002`，我们刚刚看到了关键跳转的地方，在他的前面的几行设置断点。

经验：从跳转的语句向上，找到第一个 `nop` 或者跳转等，设置中断。

```
:004121C6 90                  nop  
:004121C7 90                  nop  
:004121C8 55                  push ebp〈==============设置断点  
:004121C9 8BEC                mov ebp, esp  
:004121CB 83C4D8              add esp, FFFFFFD8  
:004121CE 53                  push ebx  
:004121CF 56                  push esi  
:004121D0 57                  push edi  
:004121D1 8BD8                mov ebx, eax  
:004121D3 B80C3C6300          mov eax, 00633C0C  
:004121D8 E8076F1E00          call 005F90E4  
:004121DD 66C745E80800        mov [ebp-18], 0008  
:004121E3 E89C9E0400          call 0045C084〈==========关键 call  
:004121E8 84C0                test al, al  
:004121EA 752F                jne 0041221B〈===============关键跳转  
:004121EC 66C745E81400        mov [ebp-18], 0014
```

执行程序，点注册，填好注册信息，点注册（`unlock`）

程序被中断在断点

点 `f8`，一步一步执行，遇到 `call`，点 `f7` 进入，然后继续点 `f8`，这个时候就比较简单了。

```
005F90E4      /$ 53           PUSH EBX  
005F90E5      |. 8BDD        MOV EBX,EBP  
005F90E7      |. 0358 04     ADD EBX,DWORD PTR DS:[EAX+4]  
005F90EA      |. 8943 08     MOV DWORD PTR DS:[EBX+8],EAX  
005F90ED      |. 8D4424 08   LEA EAX,DWORD PTR SS:[ESP+8]  
005F90F1      |. 8943 0C     MOV DWORD PTR DS:[EBX+C],EAX  
005F90F4      |. C743 04 AF>MOV DWORD PTR DS:[EBX+4],DRHAR>  
005F90FB      |. C743 10 00>MOV DWORD PTR DS:[EBX+10],0  
005F9102      |. C743 12 00>MOV DWORD PTR DS:[EBX+12],0  
005F9109      |. C743 1C 00>MOV DWORD PTR DS:[EBX+1C],0  
005F9110      |. 64:67:A1 0>MOV EAX,DWORD PTR FS:[0]  
005F9115      |. 8903        MOV DWORD PTR DS:[EBX],EAX  
005F9117      |. 64:67:891E>MOV DWORD PTR FS:[0],EBX  
005F911D      |. 5B          POP EBX  
005F911E      \\. C3         RETN〈=========程序从这里返回  
005F911F        . 90         NOP
```

返回到这个地方

```
004121DD        . 66:C745 E8>MOV WORD PTR SS:[EBP-18],8  
004121E3        . E8 9C9E040>CALL DRHARD.0045C084〈===================关键 f7 跟进（B）  
004121E8        . 84C0        TEST AL,AL  
004121EA        . 75 2F       JNZ SHORT DRHARD.0041221B  
004121EC        . 66:C745 E8>MOV WORD PTR SS:[EBP-18],14  
004121F2        . BA E281660>MOV EDX,DRHARD.006681E2       ;  ASCII "Online registration failed."  
004121F7        . 8D45 FC     LEA EAX,DWORD PTR SS:[EBP-4]  
004121FA        . E8 DD321F0>CALL DRHARD.006054DC  
004121FF        . FF45 F4     INC DWORD PTR SS:[EBP-C]  
00412202        . 8B00        MOV EAX,DWORD PTR DS:[EAX]  
```

从（B）跟进以后，就一直点 `f8` 就可以了！

在 `30` 多行以后程序把输入的假注册吗放在 `ESI` 中，不断的有字符串在 `EAX` 中进行比较！

这些在 `EAX` 中比较的字符串就是我们要的注册吗了！有很多，大概 `15` 个以上吧！有兴趣的都找出来好了！

`Ollydbg` 中文版 `v1.04` 的窗体，左上角是程序代码，右上角是寄存器的内容，右下角是堆栈的信息！所有的信息一目了然，这也是我喜欢用 `Ollydbg` 中文版 `v1.04` 的缘故。

点 `f8` 一步一步跟踪，我们发现程序使用一些长度在 `10---14` 的字符串和我们输入的注册号码的前一部分的信息进行比较。

记下所有的字符串。这些就是程序的注册码。

`DRHNT2002PROFK`

`PREMIUM2002DRH`

`STDKEYDR2002`

`PROFKEY2002WIN`

`PROFKEY2002NT`

`PREMKEYDRH2002`

以上是 `2002` 这个版本下的注册吗，使用于不同的操作系统。

小结：程序使用内存中固定的字符串和输入的注册信息进行比较，比较简单！但要注意，我们要使用注册码中有 `2002` 字样的注册码，因为这个是这一版本的注册吗，其他的是以前版本的注册吗。还有要多试一下，不同的注册吗是给不同的操作系统使用的！

问题：注册后，在关于的窗体里面有单用户的信息！不知道是不是注册吗还不完全，请那位有时间再看一下！谢谢！ `4dd466d0e1e0cc123afbe504da2462ca`
