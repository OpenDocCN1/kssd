
# 又见 TLS-Unpacking read-me's UnpackMe

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-667.htm`](https://www.pediy.com/kssd/pediy07/pediy7-667.htm)

**作者**：来自轻院的狼\[+Immlep+\]
**目标**：`umpakme_by-read-me.`
**加壳方式**：Unknow

注：TLS 泛滥了，这几天心情乱，教程写的也比较乱，大家就将就下了，如果你要脱这个 `UnpackMe` 的话自己动手看看是最好的。

这个壳是在 Crackl@b 上看到的，作者在上面发布了这个 CrackMe，出于习惯，所以就下载来看看，本来也没有想要去脱的，一来，已经很久没有脱壳了，二来，现在的壳要么简单，要么 BT。习惯性的试试用 `OllyDbg` 载入，`F9` 看看，如果可以运行，那么我就会把这个壳丢到一旁去，可是。。。。。当然我用 `OllyDbg` 时，连 `F9` 也没有按，突然间，我机上的程序全部不能动弹了，`ALT`+`Ctrl`+`Del` 也没用了。。。。汗个，重启。。

这个鸟壳，用了什么恶毒的招数？而且没有中断在入口点，估计是用了 TLS，用 `PETools` 看了一下，果然看到 TLS 表，address of Callbacks 指向了 `00464B40`，打开 `OllyDbg`，在设置选项中选择“中断在系统断点（System BreakPoint）”，ok，现在用 `OllyDbg` 载入，命令行中输入 `follow [00464B40]`，来到 TLS 回调函数处：

```
00464000     E8 00000000               CALL umpakme_.00464005          ；TLS 回调函数处，这里下断点
00464005     5D                        POP EBP
00464006     81ED 05000000             SUB EBP, 5
0046400C     E8 D9030000              CALL umpakme_.004643EA          ；这里进去是一个 IsDebuggerPresent 的 anti
00464011     E8 BF020000              CALL umpakme_.004642D5          ；这里进去，发生一个异常，清除硬件断点。
00464016     EB 01                     JMP SHORT umpakme_.00464019
```

在 `00464000` 处下断点后，`F9` 运行，中断在 `00464000` 处，现在开始我们的脱壳之旅。

#### IsDebuggerPresent

```
004643EA     64:A1 18000000            MOV EAX, DWORD PTR FS:[18]
004643F0     8B40 30                   MOV EAX, DWORD PTR [EAX+30]
004643F3     0FB640 02                 MOVZX EAX, BYTE PTR [EAX+2]
004643F7     83F8 01                   CMP EAX, 1
004643FA     74 01                     JE SHORT umpakme_.004643FD
004643FC     C3                        RETN
```

#### anti hardware breakpoint

```
00464306     8B4424 0C                 MOV EAX, DWORD PTR [ESP+C]
0046430A     C780 B0000000 C4F91200    MOV DWORD PTR [EAX+B0], 12F9C4
00464314     C740 04 00000000          MOV DWORD PTR [EAX+4], 0
0046431B     C740 08 00000000          MOV DWORD PTR [EAX+8], 0
00464322     C740 0C 00000000          MOV DWORD PTR [EAX+C], 0
00464329     C740 10 00000000          MOV DWORD PTR [EAX+10], 0
00464330     C740 14 00000000          MOV DWORD PTR [EAX+14], 0
00464337     C740 18 00000000          MOV DWORD PTR [EAX+18], 0
0046433E     B8 00000000               MOV EAX, 0
00464343     C3                        RETN
```

`F7` 慢慢走。。

#### 恶毒代码开始

```
0046444A     E8 52070000               CALL umpakme_.00464BA1           ;取 kernel base
0046444F     8BF8                      MOV EDI, EAX
00464451     8D85 3A090000            LEA EAX, DWORD PTR [EBP+93A]     ;LoadLibraryA
00464457     50                        PUSH EAX                        ;LoadLibraryA
00464458     57                        PUSH EDI
00464459     E8 58070000               CALL umpakme_.00464BB6           ;取 LoadLibraryA 地址
0046445E     8D9D EF060000            LEA EBX, DWORD PTR [EBP+6EF]
00464464     53                        PUSH EBX                       ;ASCII "ntDLL"
00464465     FFD0                      CALL NEAR EAX                  ;加载 ntDLL.dll
00464467     8BF0                      MOV ESI, EAX
00464469     8D9D 5B070000            LEA EBX, DWORD PTR [EBP+75B]     ;ASCII "NtSuspendProcess"
0046446F     53                        PUSH EBX
00464470     56                        PUSH ESI
00464471     E8 40070000               CALL umpakme_.00464BB6           ;取 NtSuspendProcess 的地址
00464476     8D9D 57070000            LEA EBX, DWORD PTR [EBP+757]
0046447C     8903                      MOV DWORD PTR [EBX], EAX
0046447E     8D9D 70070000            LEA EBX, DWORD PTR [EBP+770]     ; ASCII "NtResumeProcess"
00464484     53                        PUSH EBX
00464485     56                        PUSH ESI
00464486     E8 2B070000               CALL umpakme_.00464BB6           ;取 NtResumeProcess 的地址
0046448B     8D9D 6C070000            LEA EBX, DWORD PTR [EBP+76C]
00464491     8903                      MOV DWORD PTR [EBX], EAX
00464493     8D85 09070000            LEA EAX, DWORD PTR [EBP+709]     ;ASCII "CreateToolhelp32Snapshot"
00464499     50                        PUSH EAX
0046449A     57                        PUSH EDI
0046449B     E8 16070000               CALL umpakme_.00464BB6
004644A0     8D9D 05070000            LEA EBX, DWORD PTR [EBP+705]
004644A6     8903                      MOV DWORD PTR [EBX], EAX
004644A8     8D85 26070000            LEA EAX, DWORD PTR [EBP+726]     ;ASCII "Process32First"
004644AE     50                        PUSH EAX
004644AF     57                        PUSH EDI
004644B0     E8 01070000               CALL umpakme_.00464BB6
004644B5     8D9D 22070000            LEA EBX, DWORD PTR [EBP+722]
004644BB     8903                      MOV DWORD PTR [EBX], EAX
004644BD     8D85 4B070000            LEA EAX, DWORD PTR [EBP+74B]     ;ASCII "OpenProcess"
004644C3     50                        PUSH EAX
004644C4     57                        PUSH EDI
004644C5     E8 EC060000               CALL umpakme_.00464BB6
004644CA     8D9D 47070000            LEA EBX, DWORD PTR [EBP+747]
004644D0     8903                      MOV DWORD PTR [EBX], EAX
004644D2     8D85 39070000            LEA EAX, DWORD PTR [EBP+739]     ;ASCII "Process32Next"
004644D8     50                        PUSH EAX
004644D9     57                        PUSH EDI
004644DA     E8 D7060000               CALL umpakme_.00464BB6
004644DF     8D9D 35070000            LEA EBX, DWORD PTR [EBP+735]
004644E5     8903                      MOV DWORD PTR [EBX], EAX
004644E7     8D85 F9060000            LEA EAX, DWORD PTR [EBP+6F9]     ;ASCII "CloseHandle"
004644ED     50                        PUSH EAX
004644EE     57                        PUSH EDI
004644EF     E8 C2060000               CALL umpakme_.00464BB6
004644F4     8D9D F5060000            LEA EBX, DWORD PTR [EBP+6F5]
004644FA     8903                      MOV DWORD PTR [EBX], EAX
004644FC     6A 00                      PUSH 0
004644FE     6A 02                      PUSH 2
00464500     8D85 05070000            LEA EAX, DWORD PTR [EBP+705]
00464506     8B00                      MOV EAX, DWORD PTR [EAX]
00464508     FFD0                      CALL NEAR EAX                   ; CreateToolhelp32Snapshot
0046450A     8D9D BF050000            LEA EBX, DWORD PTR [EBP+5BF]
00464510     8903                      MOV DWORD PTR [EBX], EAX
00464512     8D9D C3050000            LEA EBX, DWORD PTR [EBP+5C3]
00464518     C703 28010000            MOV DWORD PTR [EBX], 128
0046451E     8D85 C3050000            LEA EAX, DWORD PTR [EBP+5C3]
00464524     50                        PUSH EAX
00464525     8D85 BF050000            LEA EAX, DWORD PTR [EBP+5BF]
0046452B     FF30                      PUSH DWORD PTR [EAX]
0046452D     8D85 22070000            LEA EAX, DWORD PTR [EBP+722]
00464533     8B00                      MOV EAX, DWORD PTR [EAX]
00464535     FFD0                      CALL NEAR EAX                   ; Process32First
00464537     8D9D BB050000            LEA EBX, DWORD PTR [EBP+5BB]
0046453D     8903                      MOV DWORD PTR [EBX], EAX
0046453F     8D9D DB050000            LEA EBX, DWORD PTR [EBP+5DB]
00464545     8B03                      MOV EAX, DWORD PTR [EBX]
00464547     64:8B0D 18000000          MOV ECX, DWORD PTR FS:[18]
0046454E     8B49 20                   MOV ECX, DWORD PTR [ECX+20]
00464551     3BC1                      CMP EAX, ECX
00464553     74 1D                     JE SHORT umpakme_.00464572      ;比较是不是本身进程
00464555     50                        PUSH EAX
00464556     6A 01                      PUSH 1
00464558     68 FF0F1F00               PUSH 1F0FFF
```



```
0046455D     8D85 47070000    LEA EAX, DWORD PTR [EBP+747]  

00464563     8B00             MOV EAX, DWORD PTR [EAX]  

00464565     FFD0             CALL NEAR EAX                  ; OpenProcess  

00464567     50               PUSH EAX                       

00464568     8D85 57070000    LEA EAX, DWORD PTR [EBP+757]  

0046456E     8B00             MOV EAX, DWORD PTR [EAX]      

00464570     FFD0             CALL NEAR EAX                  ; NtSuspendProcess，好毒，挂起所有的进程，不死才怪  

00464572     8D85 C3050000    LEA EAX, DWORD PTR [EBP+5C3]  

00464578     50               PUSH EAX                       

00464579     8D85 BF050000    LEA EAX, DWORD PTR [EBP+5BF]  

0046457F     FF30             PUSH DWORD PTR [EAX]  

00464581     8D85 35070000    LEA EAX, DWORD PTR [EBP+735]  

00464587     8B00             MOV EAX, DWORD PTR [EAX]  

00464589     FFD0             CALL NEAR EAX                  ; Process32Next  

0046458B     8D9D BB050000    LEA EBX, DWORD PTR [EBP+5BB]  

00464591     8903             MOV DWORD PTR [EBX], EAX      

00464593     83F8 00          CMP EAX, 0  

00464596   ^ 75 A7            JNZ SHORT umpakme\_.0046453F   ; 取完所有的进程？  

00464598     8D85 BF050000    LEA EAX, DWORD PTR [EBP+5BF]  

0046459E     FF30             PUSH DWORD PTR [EAX]  

004645A0     8D85 F5060000    LEA EAX, DWORD PTR [EBP+6F5]  

004645A6     8B00             MOV EAX, DWORD PTR [EAX]  

004645A8     FFD0             CALL NEAR EAX                  ; CloseHandle  

004645AA     8D85 E5000000    LEA EAX, DWORD PTR [EBP+E5]  

004645B0     8B18             MOV EBX, DWORD PTR [EAX]  

004645B2     81F3 A0860100    XOR EBX, 186A0  

004645B8     8918             MOV DWORD PTR [EAX], EBX  

004645BA     C3               RETN  
```

－－－－－－－－恶毒代码结束－－－－－－－－－－－－－－－

可以看到，上面的一段代码，是把本身进程外的其它进程挂起，本来如果这个壳按照平常那样运行的话，对于一些系统的进程，它没有足够的权限，它是无法挂起的系统进程，这样的话，一旦壳出错的话，问题就不大，至少不用重启，不过如果用 `OllyDbg` 调试的话，这个壳的进程的权限就会提高，这样它就可以打开系统进程，就会把系统进程也挂起，这样搞不好的话就要重启了。。躲个这个 anti 的方法比较简单，在 `00464553` 和 `00464596` 处改标志位让它跳就可以了。。

`f7` 继续。。。。

－－－－－－－RDTSC Anti---------

```
0046425C     8D85 8E020000    LEA EAX, DWORD PTR [EBP+28E]  

00464262     50               PUSH EAX                       

00464263     64:FF35 00000000 PUSH DWORD PTR FS:[0]  

0046426A     64:8925 00000000 MOV DWORD PTR FS:[0], ESP  

00464271     8D85 92020000    LEA EAX, DWORD PTR [EBP+292]  

00464277     8D9D 9F020000    LEA EBX, DWORD PTR [EBP+29F]  

0046427D     8958 06          MOV DWORD PTR [EAX+6], EBX      

00464280     8D8D CD020000    LEA ECX, DWORD PTR [EBP+2CD]  

00464286     0F31             RDTSC  

00464288     8901             MOV DWORD PTR [ECX], EAX        ; RDTSC 保存第一个值  

0046428A     33D2             XOR EDX, EDX  

0046428C     8802             MOV BYTE PTR [EDX], AL          ; 异常  

0046428E     8B7424 0C        MOV ESI, DWORD PTR [ESP+C]  

00464292     C786 B8000000 9F>MOV DWORD PTR [ESI+B8], 29F  

0046429C     33C0             XOR EAX, EAX                    

0046429E     C3               RETN  

0046429F     64:8F05 00000000 POP DWORD PTR FS:[0]            ; 异常恢复后到这里  

004642A6     83C4 04          ADD ESP, 4  

004642A9     8D8D CD020000    LEA ECX, DWORD PTR [EBP+2CD]  

004642AF     0F31             RDTSC                 ; RDTSC 保存第二个值      

004642B1     8B19             MOV EBX, DWORD PTR [ECX]        

004642B3     2BC3             SUB EAX, EBX                

004642B5     8901             MOV DWORD PTR [ECX], EAX        

004642B7     8B01             MOV EAX, DWORD PTR [ECX]       

004642B9     3D 00093D00      CMP EAX, 3D0900  

004642BE     7F 01            JG SHORT umpakme\_.004642C1    ; 计算前后两次 RDTSC 的值差，超过 3D0900 的话就 over 了，这里不要让它跳。  

004642C0     C3               RETN  
```

－－－－－－－RDTSC Anti---------

`F7` 继续。。。

一会儿就来到核心了，看来这个壳似乎还是挺温柔的。。。没有什么小花。。

```
004640A6     E8 5A010000      CALL umpakme\_.00464205       ；OutputDebugStringA。。。。又一个 anti  

004640AB     E8 F0020000      CALL umpakme\_.004643A0       ；里面发生了一个异常，没有什么看头。。。  

004640B0     E8 4F030000      CALL umpakme\_.00464404       ；又一个 anti，调用无用的 OpenServiceA 函数来鸟掉 OllyDbg  

004640B5     E8 10090000      CALL umpakme\_.004649CA       ；这里进去，解压各个区段。。。。  

004640BA     E8 A0090000      CALL umpakme\_.00464A5F       ；这里进去取一个随即数  

004640BF     E8 00000000      CALL umpakme\_.004640C4  

004640C4     5D               POP EBP                      

004640C5     81ED C4000000    SUB EBP, 0C4  

004640CB     E8 86080000      CALL umpakme\_.00464956       ；这里进入处理输入表  

004640D0     E8 6A070000      CALL umpakme\_.0046483F       ；这个 call 可以步过。。  

004640D5     E8 6F090000      CALL umpakme\_.00464A49       ；这个 call 可以步过。。  

004640DA     E8 F1090000      CALL umpakme\_.00464AD0       ；又一个 anti－－ZwSetInformationThread  

004640DF     E8 9C060000      CALL umpakme\_.00464780       ；恢复被挂起的进程  

004640E4     68 5C004500      PUSH umpakme\_.0045005C       ；OEP  

004640E9     C3               RETN  
```

－－－－－－－－－OutputDebugStringA anti－－－－－－－－－－－

```
00464205     E8 97090000      CALL umpakme\_.00464BA1  

0046420A     8BF8             MOV EDI, EAX  

0046420C     8D85 49020000    LEA EAX, DWORD PTR [EBP+249]  

00464212     50               PUSH EAX  

00464213     57               PUSH EDI                    ; kernel32.77E40000  

00464214     E8 9D090000      CALL umpakme\_.00464BB6  

00464219     8D8D 44020000    LEA ECX, DWORD PTR [EBP+244]  

0046421F     51               PUSH ECX                    ; umpakme\_.004642CD  

00464220     FFD0             CALL NEAR EAX               ; kernel32.OutputDebugStringA  

00464222     8D8D 44020000    LEA ECX, DWORD PTR [EBP+244]  

00464228     8139 3A2D2900    CMP DWORD PTR [ECX], 292D3A ; OutputDebugStringA 一下，看是不是笑脸“:-)”  

0046422E     75 13            JNZ SHORT umpakme\_.00464243 ; 看到 OllDbg 笑的话，就 Over 了啊，这里让它跳。。  

00464230     E8 04000000      CALL umpakme\_.00464239  

00464235     33C0             XOR EAX, EAX  

00464237     3100             XOR DWORD PTR [EAX], EAX  

00464239     33C0             XOR EAX, EAX  

0046423B     50               PUSH EAX  

0046423C     64:8920          MOV DWORD PTR FS:[EAX], ESP  

0046423F   - FF6424 04        JMP NEAR DWORD PTR [ESP+4]       

00464243     C3               RETN  
```

－－－－－－－－－OutputDebugStringA anti－－－－－－－－－－－

一下这个 anti 比较少见，可以看到 `OpenServiceA` 这个函数的参数的是乱压栈的，经过分析知道，在 `OllyDbg` 中，因为 `OpenServiceA` 传递的错误的参数，所以 `OllyDbg` 中会异常，无法在执行，而如果按照平常方式运行这个 `CrackMe` 的话，系统就会处理这个这乱搞的函数，程序还是可以继续运行，只不过 `OpenServiceA` 调用返回失败，按照推理的话，应该有不少这样的函数可以用来 Anti `OllyDbg`，对付这种 anti 的方法就是不要让它执行这个函数。

－－－－－－－－－OpenServiceA anti－－－－－－－－－－－－－－

```
00464404     E8 98070000      CALL umpakme\_.00464BA1  

00464409     8BF8             MOV EDI, EAX                 

0046440B     8D85 3A090000    LEA EAX, DWORD PTR [EBP+93A]  

00464411     50               PUSH EAX                      

00464412     57               PUSH EDI                      

00464413     E8 9E070000      CALL umpakme\_.00464BB6  

00464418     8D9D 41040000    LEA EBX, DWORD PTR [EBP+441]  

0046441E     53               PUSH EBX                      

0046441F     FFD0             CALL NEAR EAX                 

00464421     8D9D 34040000    LEA EBX, DWORD PTR [EBP+434]  

00464427     53               PUSH EBX                      

00464428     50               PUSH EAX                      

00464429     E8 88070000      CALL umpakme\_.00464BB6       

0046442E     83EC 0C          SUB ESP, 0C          ; 执行到这里的时候直接跳到 00464433 执行  

00464431     FFD0             CALL NEAR EAX               ; OpenServiceA  
```



### 反汇编代码片段

```
00464433     C3               RETN                 ;当执行到 0046442E 时直接修改 EIP 到这里，Return to 004640B5
```

### OpenServiceA anti 部分

#### 处理输入表

```
00464956     8D8D EF000000    LEA ECX, DWORD PTR [EBP+EF]
0046495C     8B09             MOV ECX, DWORD PTR [ECX]
0046495E     038D 87010000    ADD ECX, DWORD PTR [EBP+187]         ;
00464964     81E9 0A000000    SUB ECX, 0A
0046496A     8B9D 4C0B0000    MOV EBX, DWORD PTR [EBP+B4C]
00464970     039D 87010000    ADD EBX, DWORD PTR [EBP+187]        
00464976     4B               DEC EBX
00464977     43               INC EBX
00464978     8BF3             MOV ESI, EBX
0046497A     43               INC EBX
0046497B     803B 00          CMP BYTE PTR [EBX], 0
0046497E   ^ 75 FA           JNZ SHORT umpakme_.0046497A
00464980     43               INC EBX
00464981     8B13             MOV EDX, DWORD PTR [EBX]
00464983     0395 87010000    ADD EDX, DWORD PTR [EBP+187]        
00464989     83C3 04          ADD EBX, 4
0046498C     8942 04          MOV DWORD PTR [EDX+4], EAX
0046498F     81C1 0A000000    ADD ECX, 0A
00464995     890A             MOV DWORD PTR [EDX], ECX             ;ECX 等于处理后的函数的地址。
00464997     60               PUSHAD
00464998     9C               PUSHFD
00464999     E8 28F8FFFF      CALL umpakme_.004641C6              ;这里要进去，有一个 anti。。。。
0046499E     83F8 00          CMP EAX, 0
004649A1     74 02            JE SHORT umpakme_.004649A5
004649A3   ^ EB D5            JMP SHORT umpakme_.0046497A
004649A5     9D               POPFD
004649A6     61               POPAD
004649A7     43               INC EBX
004649A8     803B 00          CMP BYTE PTR [EBX], 0
004649AB   ^ 75 FA            JNZ SHORT umpakme_.004649A7
004649AD     83C2 04          ADD EDX, 4
004649B0     43               INC EBX
004649B1     807B 01 00       CMP BYTE PTR [EBX+1], 0
004649B5     74 12            JE SHORT umpakme_.004649C9
004649B7     803B 00          CMP BYTE PTR [EBX], 0
004649BA   ^ 74 BB            JE SHORT umpakme_.00464977
004649BC     8942 04          MOV DWORD PTR [EDX+4], EAX
004649BF     81C1 0A000000    ADD ECX, 0A
004649C5     890A             MOV DWORD PTR [EDX], ECX
004649C7   ^ EB DE            JMP SHORT umpakme_.004649A7
004649C9     C3               RETN
```

#### 处理输入表详细分析

下面来看看 `00464999` 处一个 `Call` 的代码：

```
004641C6     64:A1 30000000   MOV EAX, DWORD PTR FS:[30]
004641CC     8B40 0C          MOV EAX, DWORD PTR [EAX+C]        ;指向 Ldr
004641CF     25 0000FFFF      AND EAX, FFFF0000                 ;取高位，基址？？？
004641D4     8B40 10          MOV EAX, DWORD PTR [EAX+10]       ;这个 anti 不太清楚，我看了一下发现，如果被调试的话[EAX+10]的值为 4000060
004641D7     C3               RETN                              ;如果没有被调试的话，[EAX+10] 的值为 000000，所以在这我们转到数据窗口
                                                ;把[EAX+10]指向的地址中的四个字节清为零就可以了,我这里的地址是 00240010
```

### anti ZwSetInformationThread

```
00464AD0     E8 CC000000      CALL umpakme_.00464BA1
00464AD5     8BF8             MOV EDI, EAX                        
00464AD7 <>  8D85 3A090000    LEA EAX, DWORD PTR [EBP+93A]
00464ADD     50               PUSH EAX                            
00464ADE     57               PUSH EDI                            
00464ADF     E8 D2000000      CALL umpakme_.00464BB6
00464AE4     8D8D 1E0B0000    LEA ECX, DWORD PTR [EBP+B1E]
00464AEA     51               PUSH ECX                            
00464AEB     FFD0             CALL NEAR EAX                       
00464AED     8D9D 070B0000    LEA EBX, DWORD PTR [EBP+B07]
00464AF3     53               PUSH EBX                            
00464AF4     50               PUSH EAX                            
00464AF5     E8 BC000000      CALL umpakme_.00464BB6
00464AFA     8BD8             MOV EBX, EAX                      ;执行到这里是直接修改 EIP 到 00464B06 执行。也就是不要让它调用这个函数。
00464AFC     6A 00            PUSH 0
00464AFE     6A 00            PUSH 0
00464B00     6A 11            PUSH 11
00464B02     6A FE            PUSH -2
00464B04     FFD3             CALL NEAR EBX                       ; ntdll.ZwSetInformationThread
00464B06     C3               RETN                              Return to 004640DF
```

### OEP 部分

现在到了 `OEP` 了：

```
0045005C     55               PUSH EBP                            ; umpakme_.00464000
0045005D     8BEC             MOV EBP, ESP
0045005F     83C4 F0          ADD ESP, -10
00450062     B8 7CFE4400      MOV EAX, umpakme_.0044FE7C
00450067     E8 5C5BFBFF      CALL umpakme_.00405BC8
0045006C     A1 30204500      MOV EAX, DWORD PTR [452030]
00450071     8B00             MOV EAX, DWORD PTR [EAX]
00450073     E8 F0E5FFFF      CALL umpakme_.0044E668
00450078     8B0D 10214500    MOV ECX, DWORD PTR [452110]         ; umpakme_.00453BD0
0045007E     A1 30204500      MOV EAX, DWORD PTR [452030]
00450083     8B00             MOV EAX, DWORD PTR [EAX]
00450085     8B15 08FC4400    MOV EDX, DWORD PTR [44FC08]         ; umpakme_.0044FC54
0045008B     E8 F0E5FFFF      CALL umpakme_.0044E680
00450090     A1 30204500      MOV EAX, DWORD PTR [452030]
00450095     8B00             MOV EAX, DWORD PTR [EAX]
00450097     E8 64E6FFFF      CALL umpakme_.0044E700
0045009C     E8 7F3CFBFF      CALL umpakme_.00403D20
```

### 修复输入表

对于输入表，有兴趣的可以看看，这个壳对输入表处理的方法我觉得还是不错，所有的函数名被保存，通过壳中编好的序列号来调用函数。

```
00466167     B8 01000000      MOV EAX, 1
0046616C   ^ E9 26E7FFFF      JMP umpakme_.00464897
00466171     B8 02000000      MOV EAX, 2
00466176   ^ E9 1CE7FFFF      JMP umpakme_.00464897
0046617B     B8 03000000      MOV EAX, 3
00466180   ^ E9 12E7FFFF      JMP umpakme_.00464897
00466185     B8 04000000      MOV EAX, 4
0046618A   ^ E9 08E7FFFF      JMP umpakme_.00464897
0046618F     B8 05000000      MOV EAX, 5
00466194   ^ E9 FEE6FFFF      JMP umpakme_.00464897
00466199     B8 06000000      MOV EAX, 6
0046619E   ^ E9 F4E6FFFF      JMP umpakme_.00464897
004661A3     B8 07000000      MOV EAX, 7
004661A8   ^ E9 EAE6FFFF      JMP umpakme_.00464897
004661AD     B8 08000000      MOV EAX, 8
```

通过查看 `EAX` 里面的编号查看函数的函数名，然后通过 `00464897` 处获取这个函数的地址，调到这个函数执行。我找了个地方写入如下代码来恢复。

```
00464555     60               PUSHAD
00464556     B9 18414500      MOV ECX, umpakme_.00454118          ；00454118 为输入表函数开始地址。
0046455B     51               PUSH ECX                              ; ntdll.77F532FA
0046455C     51               PUSH ECX                              ; ntdll.77F532FA
0046455D     8B01             MOV EAX, DWORD PTR [ECX]
0046455F     FFD0             CALL NEAR EAX                         ; ntdll.RtlDeleteCriticalSection
00464561     59               POP ECX                               ; ntdll.77F532FA
00464562     8B3424           MOV ESI, DWORD PTR [ESP]
00464565     83C4 04          ADD ESP, 4
00464568     8931             MOV DWORD PTR [ECX], ESI              ; umpakme_.00464C52
0046456A     83C1 04          ADD ECX, 4
0046456D     66:8379 02 46    CMP WORD PTR [ECX+2], 46
00464572     74 03            JE SHORT umpakme_.00464577
00464574     83C1 04          ADD ECX, 4
00464577     8339 00          CMP DWORD PTR [ECX], 0
0046457A     74 02            JE SHORT umpakme_.0046457E
0046457C   ^ EB DD            JMP SHORT umpakme_.0046455B
0046457E     61               POPAD
0046457F     90               NOP
```

写完代码后不要执行先，跳到 `00464933` 处：

```
00464933     894424 24        MOV DWORD PTR [ESP+24], EAX
00464937     9D               POPFD
00464938     61               POPAD
00464939     C3               RETN
```

这里还要处理一下，找到地方写补丁代码，修改后的代码为：



## 代码修补与执行流程

```
00464933   ^\E9 4AFCFFFF       JMP umpakme\_.00464582  

00464938    61                POPAD  

00464939    C3                RETN  
```

到`00464582`处，写入代码：

```
00464582    894424 30         MOV DWORD PTR [ESP+30], EAX         

00464586    9D                POPFD  

00464587    61                POPAD  

00464588    83C4 04           ADD ESP, 4  

0046458B    C3                RET  
```

---

写完上面的代码后，将`EIP`改到`00464555`，在`0046457E`下断点，`F9`让它执行完这段补丁代码。执行所有补丁代码后，撤销这些补丁代码，将`eip`转到`OEP=0045005C`处，现在可以`dump`了，然后用`ImpRec`修复输入表，保存为`Unpacked\_.exe`，用`PETools`打开数据目录表中`tls`表，除了`Address of index`和`Address of callbacks`外，其它的按照脱壳前的填写。`Address of callbacks`我们这里需要将它修改为一个指向`000000`的地址。通过看`tls`段，发现开始地址处`457000`处的字节为`00000000`，所以这里将`Address of index`和`Address of callbacks`的值都填上`457000`，保存修改，运行，OK。叫老猫`test`了一下，他说`2003`不能运行～晕～不鸟它了。有兴趣的可以看看。

附件包含了脱壳和未脱壳的文件，脱壳的文件本机`XP`运行没问题。

附件:`umpakme\_by\_read\_me.rar`

## IsDebuggerPresent 检测方法

### 方法一：`IsDebuggerPresent_byLDR_1`

```
IsDebuggerPresent_byLDR_1:  
  mov eax, fs: [30h]  
  mov eax, [eax+0Ch]; PEB.LDR  
  and eax, 0FFFF0000h; the base address of heap LDR  
  mov eax, [eax+10h]; <> 0 - debugging (HeapFlags <> 0)  
  ret  
```

### 方法二：`IsDebuggerPresent_byLDR_2`

```
IsDebuggerPresent_byLDR_2:  
  mov eax, fs: [30h]  
  mov eax, [eax+0Ch]  
  movzx eax, byte [eax-3]; flags of the block of  
  a heap and eax, 2+4;          <> 0 debugging (flags tail_checking=2 and free_checking=4 are established)  
  ret  
```

Except for flags it is possible to check up a tail marker - hardly it will be more complex(difficult):

### 方法三：`IsDebuggerPresent_byLDR_3`

```
IsDebuggerPresent_byLDR_3:  
  mov eax, fs: [30h]  
  mov eax, [eax+0Ch]  
  movzx есx, byte [eax-2]; number of service bytes in the  
  block movzx edx, word [eax-8]; the general(common) size of the block in terms of 8 bytes  
  sub eax, ecx  
  mov eax, [eax+edx*8];      a  
  tail xor eax, 0ABABABABh;       = 0 - debugging (a tail marker);  
  sub eax, 1;  
  sbb eax, eax;             <> 0 - debugging  
  ret  
```

壳中用到两个用`ldr`的`anti`，即上面的`1`和`2`。

见链接：

`http://www.wasm.ru/forum/index.php?action=vthread&forum=4&topic=10118&%20page=1`

第一个`Anti`并不通用。

重复列出`IsDebuggerPresent_byLDR_1`（可能存在排版重复）：

```
IsDebuggerPresent_byLDR_1:  
  mov eax, fs: [30h]  
  mov eax, [eax+0Ch]; PEB.LDR  
  and eax, 0FFFF0000h; the base address of heap LDR  
  mov eax, [eax+10h]; <> 0 - debugging (HeapFlags <> 0)  
  ret  
```

我的系统是`2k3 sp1`，不在调试状态的时候，`eax`仍然不为`0`。

看汇编代码哪有看`C`代码来得舒服：）

## 汇编代码示例

```
7C95B444   56                PUSH ESI  
7C95B445   50                PUSH EAX  
7C95B446   57                PUSH EDI  
7C95B447   68 E4B4957C      PUSH ntdll.7C95B4E4                   ; ASCII "LDR: Tls Callbacks Found. Imagebase %p Tls %p CallBacks %p\n"  
7C95B44C   E8 9F4FFFFF      CALL ntdll.DbgPrint  
7C95B451   83C4 10          ADD ESP,10  
7C95B454   ^E9 BCD0FEFF     JMP ntdll.7C948515  
7C95B459   8945 E4          MOV DWORD PTR SS:[EBP-1C],EAX  
7C95B45C   83C6 04          ADD ESI,4  
7C95B45F   8975 E0          MOV DWORD PTR SS:[EBP-20],ESI  
7C95B462   381D 21C1997C   CMP BYTE PTR DS:[7C99C121],BL  
7C95B468   74 0F            JE SHORT ntdll.7C95B479  
7C95B46A   50               PUSH EAX  
7C95B46B   57               PUSH EDI  
7C95B46C   68 20B5957C      PUSH ntdll.7C95B520                   ; ASCII "LDR: Calling Tls Callback Imagebase %p Function %p\n"  
7C95B471   E8 7A4FFFFF      CALL ntdll.DbgPrint  
7C95B476   83C4 0C          ADD ESP,0C  
7C95B479   53               PUSH EBX  
7C95B47A   FF75 0C          PUSH DWORD PTR SS:[EBP+C]  
7C95B47D   57               PUSH EDI  
7C95B47E   FF75 E4          PUSH DWORD PTR SS:[EBP-1C]  
7C95B481   E8 0D5DFCFF      CALL ntdll.7C921193  
7C95B486   ^E9 8AD0FEFF     JMP ntdll.7C948515  
```

---

## NT4.0 中 TLS 处理相关源代码

摘自`nt4\private\ntos\dll\ldrinit.c`：

```c
VOID  
LdrpCallTlsInitializers(  
    PVOID DllBase,  
    ULONG Reason  
    )  
{  
    PIMAGE_TLS_DIRECTORY TlsImage;  
    ULONG TlsSize;  
    PIMAGE_TLS_CALLBACK *CallBackArray;  
    PIMAGE_TLS_CALLBACK InitRoutine;  
    TlsImage = (PIMAGE_TLS_DIRECTORY)RtlImageDirectoryEntryToData(  
                   DllBase,  
                   TRUE,  
                   IMAGE_DIRECTORY_ENTRY_TLS,  
                   &TlsSize  
                   );  
    try {  
        if ( TlsImage ) {  
            CallBackArray = TlsImage->AddressOfCallBacks;  
            if ( CallBackArray ) {  
                if (ShowSnaps) {  
                    DbgPrint( "LDR: Tls Callbacks Found. Imagebase %lx Tls %lx CallBacks %lx\n",  
                               DllBase,  
                               TlsImage,  
                               CallBackArray  
                        );  
                    }  
                while(*CallBackArray){  
                    InitRoutine = *CallBackArray++;  
                    if (ShowSnaps) {  
                        DbgPrint( "LDR: Calling Tls Callback Imagebase %lx Function %lx\n",  
                                   DllBase,  
                                   InitRoutine  
                        );  
                    }  
#if defined (WX86)  
                    if (!Wx86CurrentTib() ||  
                        LdrpRunWx86DllEntryPoint(  
                            (PDLL_INIT_ROUTINE)InitRoutine,  
                            NULL,  
                            DllBase,  
                            Reason,  
                            NULL  
                        ) ==  STATUS_IMAGE_MACHINE_TYPE_MISMATCH)  
#endif  
                   {  
                    (InitRoutine)(DllBase,Reason,0);  
                    }  
                }  
            }  
        }  
    }  
    except (EXCEPTION_EXECUTE_HANDLER) {  
        ;  
    }  
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
