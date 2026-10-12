
# SDProtector 1.2 脱壳

> 原文：[`www.pediy.com/kssd/pediy11/114776.html`](https://www.pediy.com/kssd/pediy11/114776.html)

```
00903000 >   55               push ebp                 //程序载入停在这里
00903001    8BEC             mov ebp,esp              //由于壳是双进程的，所以我们需要先干掉双进程
00903003    6A FF            push -0x1                //至于为什么是双进程你 OD 运行下就知道啦
00903005    68 1D321305      push 0x513321D           //壳比较**，网上搜索了下都是最后一次异常法
0090300A    68 88888808      push 0x8888888           //但是 SOD 太强啦，减少保护他也一样无法异常
0090300F    64:A1 00000000   mov eax,dword ptr fs:[0] //使用其他隐藏插件也未果，总被检测出调试器
00903015    50               push eax                 //于是便有了此法，具体请看下文
00903016    64:8925 0000000> mov dword ptr fs:[0],esp //Ctrl+g 查找 CreateFileA，F2 在函数末尾下断免检测
0090301D    58               pop eax                  //此壳异常 N 多，还有效验初始化时间，检测调试器，诸多检测
0090301E    64:A3 00000000   mov dword ptr fs:[0],eax //Die64 查编写是 DELPHI，好啦，我们就以**制**吧
```

```
7C801A28 >   8BFF             mov edi,edi                  //查找到了这里
7C801A2A     55               push ebp
7C801A2B     8BEC             mov ebp,esp
7C801A2D     FF75 08          push dword ptr ss:[ebp+0x8]
7C801A30     E8 8FC60000      call kernel32.7C80E0C4
7C801A35     85C0             test eax,eax
7C801A37     74 1E            je short kernel32.7C801A57
7C801A39     FF75 20          push dword ptr ss:[ebp+0x20]
7C801A3C     FF75 1C          push dword ptr ss:[ebp+0x1C]
7C801A3F     FF75 18          push dword ptr ss:[ebp+0x18]
7C801A42     FF75 14          push dword ptr ss:[ebp+0x14]
7C801A45     FF75 10          push dword ptr ss:[ebp+0x10]
7C801A48     FF75 0C          push dword ptr ss:[ebp+0xC]
7C801A4B     FF70 04          push dword ptr ds:[eax+0x4]
7C801A4E     E8 5DED0000      call kernel32.CreateFileW
7C801A53     5D               pop ebp
7C801A54     C2 1C00          retn 0x1C                    //在这里 F2 下断
7C801A57     83C8 FF          or eax,-0x1                  //shift+f9 运行程序
7C801A5A    ^ EB F7          jmp short kernel32.7C801A53   //停下后 ait+f9 返回,记住返回前取消断点
```

```
0090F554     8BF0             mov esi,eax                    //返回到了这里
0090F556     83FE FF          cmp esi,-0x1                   //F8 单步小心跟踪
0090F559     0F84 72010000    je SparkNot.0090F6D1
0090F55F     8D4C24 1C        lea ecx,dword ptr ss:[esp+0x1C]
0090F563     6A 00            push 0x0
0090F565     51               push ecx
0090F566     8D5424 18        lea edx,dword ptr ss:[esp+0x18]
0090F56A     6A 0C            push 0xC
```

```
0090F593     837C24 1C 0C     cmp dword ptr ss:[esp+0x1C],0xC //下面的 jnb 修改 C 标志位让他跳
0090F598     73 76            jnb short SparkNot.0090F610     //这里让他跳不跳就死啦
0090F59A     E8 F7FDFFFF      call SparkNot.0090F396          //不跳就死
0090F59F     894424 10        mov dword ptr ss:[esp+0x10],eax
0090F5A3     8D4424 10        lea eax,dword ptr ss:[esp+0x10]
0090F5A7     50               push eax
```

```
0012FE84   00 00 00 00 EA FE 12 00 7E 74 65 6D 70 30 2D 31  ....掰.~temp0-1      //上面一句时就可以看见分离子进程
0012FE94   35 34 31 31 31 30 37 30 31 2E 74 6D 70 00 90 00  541110701.tmp.?
0012FEA4   10 00 00 00 00 FE 12 00 54 0E 90 61 10 35 1A 5F  ....?.T5_
0012FEB4   07 1D B2 E4 58 D6 4B 15 43 3A 5C 44 4F 43 55 4D  蹭 XC:\\DOCUM
0012FEC4   45 7E 31 5C 66 61 6E 66 61 6E 5C 4C 4F 43 41 4C  E~1\\fanfan\\LOCAL
0012FED4   53 7E 31 5C 54 65 6D 70 5C 7E 74 65 6D 70 30 2D  S~1\\Temp\\~temp0-
0012FEE4   31 35 34 31 31 31 30 37 30 31 2E 74 6D 70 00 90  1541110701.tmp.
```

```
0090F642     81FF 803E0000    cmp edi,0x3E80                  //这里判断是否需要分离进程
0090F648     897C24 20        mov dword ptr ss:[esp+0x20],edi //下面的 jbe 修改 C 标志位让他跳
0090F64C     76 08            jbe short SparkNot.0090F656     //这里也让他跳
0090F64E     83FB 05          cmp ebx,0x5                       //不然跳到 JMP 下去又是死循环
0090F651     74 7E            je short SparkNot.0090F6D1
0090F653     43               inc ebx
0090F654     EB 57            jmp short SparkNot.0090F6AD
0090F656     85FF             test edi,edi
0090F658     0F85 B0000000    jnz SparkNot.0090F70E            //通过修改跳转后我们的 jnz 实现了跳转
0090F65E     E8 33FDFFFF      call SparkNot.0090F396           //我们让 jnz 跳过分离进程
0090F663     8BF8             mov edi,eax                      //单步 F8 跟
0090F665     8B4424 10        mov eax,dword ptr ss:[esp+0x10]
```

```
0090F70E     5F               pop edi
0090F70F     5E               pop esi                   //jnz 跳转实现后来到这里
0090F710     5D               pop ebp                   //我们 Ctrl+g 查找 GetModuleHandleA，F2 在函数末尾下断免检测
0090F711     B8 08000000      mov eax,0x8
0090F716     5B               pop ebx
0090F717     81C4 48010000    add esp,0x148
0090F71D     C3               retn
```

```
7C80B6F1 >   8BFF             mov edi,edi               //我们查找到了这里
7C80B6F3     55               push ebp                  //在函数尾下断吧
7C80B6F4     8BEC             mov ebp,esp
7C80B6F6     837D 08 00       cmp dword ptr ss:[ebp+0x8],0x0
7C80B6FA     74 18            je short kernel32.7C80B714
7C80B6FC     FF75 08          push dword ptr ss:[ebp+0x8]
7C80B6FF     E8 C0290000      call kernel32.7C80E0C4
7C80B704     85C0             test eax,eax
7C80B706     74 08            je short kernel32.7C80B710
7C80B708     FF70 04          push dword ptr ds:[eax+0x4]
7C80B70B     E8 7D2D0000      call kernel32.GetModuleHandleW
7C80B710     5D               pop ebp
7C80B711     C2 0400          retn 0x4                   //在这里 F2 下断
7C80B714     64:A1 18000000   mov eax,dword ptr fs:[0x18] //shift+f9 运行
7C80B71A     8B40 30          mov eax,dword ptr ds:[eax+0x30] //注意堆栈和寄存器
7C80B71D     8B40 08          mov eax,dword ptr ds:[eax+0x8]  //我们需要找返回时机
7C80B720    ^ EB EE          jmp short kernel32.7C80B710
```

```
0012DB80   0090DFE8  返回到 SparkNot.0090DFE8 来自 SparkNot.009129D2
0012DB84   00000000                             //当堆栈窗口这样显示时就是我们的返回时机啦
0012DB88   0000020B                             //此时可以在上面的返回到右键跟随过去也可以 ait+f9
0012DB8C   00903100  SparkNot.00903100
0012DB90   7C800000  kernel32.7C800000
0012DB94   7C80262C  kernel32.7C80262C
0012DB98   00911264  ASCII "IsDebuggerPresent"
0012DB9C   0090C7E9  返回到 SparkNot.0090C7E9 来自 SparkNot.00908D62
0012DBA0   7C8074C7  ASCII "IsDebuggerPresent"
0012DBA4   00911264  ASCII "IsDebuggerPresent"
```

```
0090DFE8     50               push eax                 //返回到了这里，我们看看程序是否解码
0090DFE9     E8 C7B1FFFF      call SparkNot.009091B5    //ait+e 双击程序进程
0090DFEE     E8 CBA3FFFF      call SparkNot.009083BE
0090DFF3     8BF0             mov esi,eax
0090DFF5     81EE 00010000    sub esi,0x100
0090DFFB     E8 DFABFFFF      call SparkNot.00908BDF
0090E000     8BF8             mov edi,eax
0090E002     8D4424 08        lea eax,dword ptr ss:[esp+0x8]
0090E006     50               push eax
0090E007     E8 E0B2FFFF      call SparkNot.009092EC
```

```
007BD000     3243 6C          xor al,byte ptr ds:[ebx+0x6C]        //双击进来后
007BD003     6963 6B 0952657> imul esp,dword ptr ds:[ebx+0x6B],0x73655> //倒程序代码乱的
007BD00A     48               dec eax                               //我们来看看我们的基址是多少
007BD00B     61               popad
007BD00C     6E               outs dx,byte ptr es:[edi]
007BD00D     64:6C            ins byte ptr es:[edi],dx
007BD00F     65:0200          add al,byte ptr gs:[eax]
007BD012     0000             add byte ptr ds:[eax],al
007BD014     0000             add byte ptr ds:[eax],al
```



# Memory map，条目 19

//我们程序的基址是 `00400000`

```
地址=00400000 //而我们上面对应的地址却是 00700000 的地址
大小=00001000 (4096.) //看来我们是跑进父进程的代码啦
属主=SparkNot 00400000 (自身) //Ctrl+g 查找 00401000 看看
区段=
包含=PE 文件头
类型=Imag 01001002
访问=R
初始访问=RWE
```

━━━━━━━━━━━━━━━━━━━━━━━━━━

```
00401000    04 10           add al,0x10              //DELPHI 的特征之一
00401002    40              inc eax                  //一般 DELPHI 在 00401000 处显示的都是这样几句
00401003    0003            add byte ptr ds:[ebx],al  //现在到了程序本身的代码啦，现在就差找 OEP 啦
00401005    07              pop es                   //现在我就来说说我对 DELPHI 特征的一点定义吧
00401006    42              inc edx                  //右键查找 ASCII 码
00401007    6F              outs dx,dword ptr es:[edi] //把滚动条一直向下拉，拉到最底部
00401008    6F              outs dx,dword ptr es:[edi]  //通常我们 DELPHI 的程序就会有标题在那里显示
00401009    6C              ins byte ptr es:[edi],dx    //当然是指 OEP 没有被抽也没有被 VM 的情况下
0040100A    65:61           popad                   //具体的请看下图
0040100C    6E              outs dx,byte ptr es:[edi]
```

━━━━━━━━━━━━━━━━━━━━━━━━━━

```
005D95FE    BA D8965D00     mov edx,SparkNot.005D96D8          ; 星火记事本       //双击后来到这里
005D9603    E8 F483ECFF     call SparkNot.004A19FC             //现在我们找段首吧
005D9608    8B0D 6CDF5D00   mov ecx,dword ptr ds:[0x5DDF6C]    ; SparkNot.005DFC20  //段首就是我们的 OEP 啦
005D960E    8B09            mov ecx,dword ptr ds:[ecx]

005D9610    B2 01           mov dl,0x1

005D9612    A1 54ED5900     mov eax,dword ptr ds:[0x59ED54]    ; prH
005D9617    E8 C40EECFF     call SparkNot.0049A4E0

005D961C    8B15 80D95D00   mov edx,dword ptr ds:[0x5DD980]    ; SparkNot.005F85E8
005D9622    8902            mov dword ptr ds:[edx],eax
005D9624    A1 80D95D00     mov eax,dword ptr ds:[0x5DD980]
```

━━━━━━━━━━━━━━━━━━━━━━━━━━

```
005D9564    55              push ebp                 //OEP 地址
005D9565    8BEC            mov ebp,esp               //我们新建一下 EIP，然后脱壳
005D9567    83C4 EC         add esp,-0x14             //Ctrl+b 查找 FF25 随便找个 IAT 调用看看我们的 IAT 指针
005D956A    53              push ebx
005D956B    56              push esi
005D956C    33C0            xor eax,eax
005D956E    8945 EC         mov dword ptr ss:[ebp-0x14],eax
005D9571    B8 848E5D00     mov eax,SparkNot.005D8E84
005D9576    E8 99D9E2FF     call SparkNot.00406F14
```

━━━━━━━━━━━━━━━━━━━━━━━━━━

```
004012CC  - FF25 08F35F00   jmp dword ptr ds:[0x5FF308]      //右键数据窗口跟随看看我们的 IAT 被加密成什么形式啦
004012D2    8BC0            mov eax,eax
004012D4  - FF25 04F35F00   jmp dword ptr ds:[0x5FF304]
004012DA    8BC0            mov eax,eax
004012DC  - FF25 00F35F00   jmp dword ptr ds:[0x5FF300]
004012E2    8BC0            mov eax,eax
004012E4  - FF25 FCF25F00   jmp dword ptr ds:[0x5FF2FC]
004012EA    8BC0            mov eax,eax
004012EC  - FF25 F8F25F00   jmp dword ptr ds:[0x5FF2F8]
```

━━━━━━━━━━━━━━━━━━━━━━━━━━

```
005FF308  001116FA          //我们的指针全部被加密啦，没有一个好的
005FF30C  001116C8          //现在我们记录下 OEP 然后记录第一个被加密指针的地址
005FF310  00000000          //从载程序后寻找我们的 IAT 指针吧
005FF314  001117E2
005FF318  00111734
005FF31C  00111702
005FF320  00111750
005FF324  00000000
```

`OEP：005D9564` //得到这三句之后就好办啦

`IAT 起始：005FF258 001113AE` //从载程序

`IAT 结束：005FFBB8 0016BC76`

━━━━━━━━━━━━━━━━━━━━━━━━━━

```
00903000 >  55              push ebp               //od 从新载入程序
00903001    8BEC            mov ebp,esp            //现在我们在数据窗口找到 005FF258 下硬件写入
00903003    6A FF           push -0x1              //Ctrl+g 查找 CreateFileA，F2 在函数末尾下断免检测
00903005    68 1D321305     push 0x513321D
0090300A    68 88888808     push 0x8888888
0090300F    64:A1 00000000  mov eax,dword ptr fs:[0]
00903015    50              push eax
00903016    64:8925 0000000>mov dword ptr fs:[0],esp
0090301D    58              pop eax
0090301E    64:A3 00000000  mov dword ptr fs:[0],eax
```

━━━━━━━━━━━━━━━━━━━━━━━━━━

```
0090F593    837C24 1C 0C    cmp dword ptr ss:[esp+0x1C],0xC      //下面就不分析具体为什么跳过上面有讲啦
0090F598    73 76           jnb short SparkNot.0090F610          //修改标志位跳过
0090F59A    E8 F7FDFFFF     call SparkNot.0090F396
0090F59F    894424 10       mov dword ptr ss:[esp+0x10],eax
```

━━━━━━━━━━━━━━━━━━━━━━━━━━

```
0090F648    897C24 20       mov dword ptr ss:[esp+0x20],edi
0090F64C    76 08           jbe short SparkNot.0090F656          //修改标志位跳过
0090F64E    83FB 05         cmp ebx,0x5
0090F651    74 7E           je short SparkNot.0090F6D1
```

━━━━━━━━━━━━━━━━━━━━━━━━━━

```
0090F70E    5F              pop edi            //我们 Ctrl+g 查找 GetModuleHandleA，F2 在函数末尾下断免检测
0090F70F    5E              pop esi
0090F710    5D              pop ebp
0090F711    B8 08000000     mov eax,0x8
0090F716    5B              pop ebx
0090F717    81C4 48010000   add esp,0x148
0090F71D    C3              retn
```

━━━━━━━━━━━━━━━━━━━━━━━━━━

```
005FF258  001113AE         //程序在我们下段后第 12 次运行，写入第一个加密指针
005FF25C  00000000         //现在反回回去看看吧
005FF260  00000000
005FF264  00000000
005FF268  00000000
005FF26C  00000000
```

━━━━━━━━━━━━━━━━━━━━━━━━━━

这里通过下面这样的语句进程为程序指针完成加密操作，但是这里并未出现了任何我们程序的指针部分。所以我们可以在看看，程序是通过几句比较来判断指针属于那一个模块。

```
008FAD17    8917            mov dword ptr ds:[edi],edx
```

━━━━━━━━━━━━━━━━━━━━━━━━━━

```
008FAE57    2B7C24 30       sub edi,dword ptr ss:[esp+0x30]      ; SparkNot.00401000
008FAE5B    8B5424 34       mov edx,dword ptr ss:[esp+0x34]      //在这里时我们出现了代码段地址
008FAE5F    893A            mov dword ptr ds:[edx],edi           //我们在继续像后面走走看看我们的程序流程
008FAE61    F7D8            neg eax                //继续从新载入程序
008FAE63    83C4 0C         add esp,0xC            //同时我们在硬件断点窗口点跟随我们下的断点，或者直接数据窗口中查找
```

━━━━━━━━━━━━━━━━━━━━━━━━━━

```
005FF258  7C9313B1  ntdll.RtlDeleteCriticalSection   //又运行十几次之后发现我们的指针居然出现解密啦
005FF25C  001113C6                               //此时我们的程序
005FF260  0011131E                               //此时我们可以判断得出我们前面加密的地址在后面还会解密
005FF264  00111436                               //此时正是处理指针的大好时机
005FF268  00111452
005FF26C  00111420
005FF270  00111470
005FF274  0011147C
```

━━━━━━━━━━━━━━━━━━━━━━━━━━

```
009105E7  /EB 1E          jmp short SparkNot.00910607    //此时我们程序到停在了这里
009105E9  |8B5424 10      mov edx,dword ptr ss:[esp+0x10] //我们从新载入程序
009105ED  |57             push edi
009105EE  |52             push edx
009105EF  |E8 94C1FFFF    call SparkNot.0090C788
009105F4  |8906          mov dword ptr ds:[esi],eax
```

━━━━━━━━━━━━━━━━━━━━━━━━━━

```
00903000 >  55              push ebp                   //又一次从新载入程序
00903001    8BEC            mov ebp,esp                //继续吧上面的步骤
00903003    6A FF           push -0x1                  //同样是先解决分离的进程然后下段继续
00903005    68 1D321305     push 0x513321D             //我就不多做介绍，直接到处理指针处
0090300A    68 88888808     push 0x8888888
0090300F    64:A1 00000000  mov eax,dword ptr fs:[0]
```

━━━━━━━━━━━━━━━━━━━━━━━━━━

```
0090BA33    8B4424 14       mov eax,dword ptr ss:[esp+0x14]    //这里时我们指针开始执行写入
0090BA37    40              inc eax                      //在执行几步会发现我们的第一个指针开始写入
0090BA38    0FBFE8         movsx ebp,ax
0090BA3B    894424 14       mov dword ptr ss:[esp+0x14],eax
0090BA3F    8B4424 28       mov eax,dword ptr ss:[esp+0x28]
0090BA43    3BE8            cmp ebp,eax
0090BA45  ^ 0F8C 53FFFFFF  jl SparkNot.0090B99E
```

━━━━━━━━━━━━━━━━━━━━━━━━━━

直接下着写入段点后发现指针目前只加密了一部分，这里我们就先锁定加密的那一部分。

`\---------------------------------------------------------省略一部分指针`

```
005FF30C  7C809B97  kernel32.CloseHandle
005FF310  00000000              //这下可以清晰看见我们指针加密的是那一部分啦
```



# 脱壳过程记录

## 1. IAT 指针分析

```
005FF314  00154A14              //此时把写入段点改为下 005FF30C 处，因为那里是最后一个未加密指针  
005FF318  00154A4C              //这样我们就可以快速定位到我们加密指针的地址  
005FF31C  0090EAB3  SparkNot.0090EAB3     //继续从新载入程序现在完成我们脱壳的整个步骤  
005FF320  00154A84  
005FF324  00000000  
005FF328  77DA7ABB  advapi32.RegQueryValueExA  
005FF32C  77DA7852  advapi32.RegOpenKeyExA  
005FF330  77DA6C27  advapi32.RegCloseKey  
005FF334  00000000  
005FF338  770F4880  oleaut32.SysFreeString  
005FF33C  77124505  oleaut32.SysReAllocStringLen  
005FF340  770F4BA7  oleaut32.SysAllocStringLen  
005FF344  00000000  
005FF348  7C809C15  kernel32.TlsSetValue  
005FF34C  7C809790  kernel32.TlsGetValue  
005FF350  7C8099DD  kernel32.LocalAlloc  
005FF354  7C80B6F1  kernel32.GetModuleHandleA  
005FF358  00000000  
```

\---------------------------------------------------------省略一部分指针

```
005FF6C0  00000000  
005FF6C4  00154ABC  
005FF6C8  00154AF4  
005FF6CC  00154B2C  
005FF6D0  00154B64  
005FF6D4  00154B9C  
005FF6D8  00154BD4  
```

\---------------------------------------------------------省略一部分指针

━━━━━━━━━━━━━━━━━━━━━━━━━━

## 2. 代码断点与修改

```
0090CB85   /74 0A           je short SparkNot.0090CB91       //这里需要修改成 JMP 跳过效验  
0090CB87   |E8 43B8FFFF     call SparkNot.009083CF  
0090CB8C   |E8 DEF BFFFF    call SparkNot.0090C76F  
0090CB91   \\B8 01000000    mov eax,0x1  
0090CB96    83C4 68          add esp,0x68  
```

━━━━━━━━━━━━━━━━━━━━━━━━━━

```
00910352    8B4424 34       mov eax,dword ptr ss:[esp+0x34]  
00910356    85C0            test eax,eax  
00910358    0F84 36020000   je SparkNot.00910594              //这句 JE 就是判断是否加密我们的指针，如果加密则跳转不实现  
0091035E    68 56DC4000     push SparkNot.0040DC56            //修改上面的 JE 为 JMP 就可以达到我们想到的目的啦  
00910363    E8 3D80FFFF     call SparkNot.009083A5            //但是获取 IAT 后发现程序还特殊处理了一个指针  
00910368    50              push eax  
00910369    57              push edi  
```

━━━━━━━━━━━━━━━━━━━━━━━━━━

## 3. 特殊加密指针处理

```
005FF314  77D311CB  user32.GetKeyboardType  
005FF318  77D20FC8  user32.LoadStringA             //我们对特殊的加密指针地址下硬件写入断点  
005FF31C  0090EAB3  SparkNot.0090EAB3              //根据判断应该是 USER32 模块里的指针  
005FF320  77D20F70  user32.CharNextA              //好我们从新来一次，这次我只负责找出这个指针加密，完成我们的脱壳  
```

━━━━━━━━━━━━━━━━━━━━━━━━━━

```
00910358   /0F84 36020000   je SparkNot.00910594     //先把我们的加密指针的跳转给处理啦  
0091035E   |68 56DC4000     push SparkNot.0040DC56    //然后在单步跟我们的特殊处理的指针  
00910363   |E8 3D80FFFF     call SparkNot.009083A5  
00910368   |50               push eax  
00910369   |57               push edi  
0091036A   |E8 F389FFFF     call SparkNot.00908D62  
```

━━━━━━━━━━━━━━━━━━━━━━━━━━

```
0091059E    50               push eax                   //在这里时我们发现我们的指针的 ASCII 码出现  
0091059F    57               push edi                   //我们可以看见信息窗口 MessageBoxA  
009105A0    E8 BD87FFFF      call SparkNot.00908D62  
009105A5    85C0            test eax,eax  
009105A7    75 0E           jnz short SparkNot.009105B7  
```

━━━━━━━━━━━━━━━━━━━━━━━━━━

`eax=0090FC4A` (`SparkNot.0090FC4A`), `ASCII` `"MessageBoxA"`       //信息窗口显示如下

━━━━━━━━━━━━━━━━━━━━━━━━━━

```
009105A7   /75 0E           jnz short SparkNot.009105B7     //这里修改 JMP 跳过加密  
009105A9   |68 B3CA4000     push SparkNot.0040CAB3          //修改后我们特殊处理的指针出现 MessageBoxA  
009105AE   |E8 F27DFFFF     call SparkNot.009083A5          //继续 shift+f9 把程序运行起来  
009105B3   |8906            mov dword ptr ds:[esi],eax  
009105B5   |EB 50           jmp short SparkNot.00910607  
```

━━━━━━━━━━━━━━━━━━━━━━━━━━

## 4. OEP 计算

此时填上我们事先第一次找来的 `OEP` 地址自动查找即可完成我们的脱壳

`OEP`：`005D9564` - 程序基址 `00400000` = `1d9564`

脱壳时发现我们的程序还有三个无效指针，不管他直接给剪掉即可

运行程序发现我们的程序有效验，载入我们脱壳后的程序

既然有效验，我们就逐一排除效验

首先肯定是排除文件名效验，因为简单又方便，把原程序文件名修改一下，原程序文件名改动就出现了效验

好我们把我们脱壳后的文件名改成程序原文件的文件名

试了下功能，发现点关于时程序会出现错误，点退出时程序也会出现错误

现在我们来解决这两处错误吧，用到 `DELPHI` 按钮事件

## 5. Delphi 按钮事件脚本

```
var DelphiAddr

mov DelphiAddr,401000

Delphiloop:

find DelphiAddr,#740E8BD38B83????????FF93????????#

cmp $RESULT,0

je Exit

add $RESULT,0A

bp $RESULT

add $RESULT,1

mov DelphiAddr,$RESULT

jmp Delphiloop

Exit:

eval "按钮事件断点设置完毕!"

msg $RESULT

ret
```

━━━━━━━━━━━━━━━━━━━━━━━━━━

## 6. 退出与错误修复

```
005B6E80    55              push ebp   //退出在下面就发生错误，既然是退出就好办多啦  
005B6E81    8BEC            mov ebp,esp    //直接让 EAX 为 0 然后把跳转指向 ExitProcess  
005B6E83    83C4 F0         add esp,-0x10  //修改如下  
```

修改后：

```
005B6E80   .  B8 00000000   mov eax,0x0  
005B6E85   .^ E9 63D9E4FF   jmp SparkNot.004047ED  
```

━━━━━━━━━━━━━━━━━━━━━━━━━━

```
0058F9B4    55              push ebp             //这里调用错误，我们直接把段首给 RETN 掉  
0058F9B5    8BEC            mov ebp,esp          //修改如下  
0058F9B7    83C4 C0         add esp,-0x40  
0058F9BA    53              push ebx  
0058F9BB    33D2            xor edx,edx  
0058F9BD    8955 C0         mov dword ptr ss:[ebp-0x40],edx  
```

修改后：

```
0058F9B4    C3              retn  
```

━━━━━━━━━━━━━━━━━━━━━━━━━━

## 7. 脱壳后程序

`http://www.dbank.com/download.action?t=40&k=MzI0MjY3Nzc=&pcode=LCw4OTkyMTksODk5MjE5&rnd=4`
