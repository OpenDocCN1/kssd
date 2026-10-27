
```
004FD7D5     EB 01               jmp short 图书馆管.004FD7D8
004FD7D7     FF61 EB             jmp dword ptr ds:[ecx-15]
004FD7DA     01E8                add eax,ebp
004FD7DC     C3                  retn
004FD7DD     8D85 C4484000       lea eax,dword ptr ss:[ebp+4048C4]
004FD7E3     50                  push eax                              ; 图书馆管.004FDC8D
004FD7E4     C3                  retn
004FD7E5     55                  push ebp
004FD7E6     8BEC                mov ebp,esp
004FD7E8     57                  push edi                              ; 图书馆管.004FD842
004FD7E9     8B45 10             mov eax,dword ptr ss:[ebp+10]
004FD7EC     8BB8 C4000000       mov edi,dword ptr ds:[eax+C4]
004FD7F2     FF37                push dword ptr ds:[edi]
004FD7F4     33FF                xor edi,edi                           ; 图书馆管.004FD842
004FD7F6     64:8F07             pop dword ptr fs:[edi]                ; kernel32.7C800000
004FD7F9     8380 C4000000 08    add dword ptr ds:[eax+C4],8
004FD800     8BB8 A4000000       mov edi,dword ptr ds:[eax+A4]
004FD806     C1C7 07             rol edi,7
004FD809     89B8 B8000000       mov dword ptr ds:[eax+B8],edi         ; 图书馆管.004FD842
004FD80F     B8 00000000         mov eax,0
004FD814     5F                  pop edi                               ; kernel32.7C800000
004FD815     C9                  leave
004FD816     C3                  retn
004FD817     32C0                xor al,al
004FD819     8DBD 0D414000       lea edi,dword ptr ss:[ebp+40410D]
004FD81F     B9 85070000         mov ecx,785
004FD824     AA                  stos byte ptr es:[edi]
004FD825   ^ E2 FD               loopd short 图书馆管.004FD824         <------//循环，到下一行按F4运行
004FD827     8DBD EF484000       lea edi,dword ptr ss:[ebp+4048EF]
004FD82D     B9 5D040000         mov ecx,45D
004FD832     AA                  stos byte ptr es:[edi]
004FD833   ^ E2 FD               loopd short 图书馆管.004FD832         <------//循环，到下一行按F4运行
004FD835     61                  popad
004FD836     50                  push eax                              ; 图书馆管.004FDC8D <------//此壳的标志
004FD837     33C0                xor eax,eax                           ; 图书馆管.004FDC8D
004FD839     64:FF30             push dword ptr fs:[eax]
004FD83C     64:8920             mov dword ptr fs:[eax],esp
004FD83F     EB 01               jmp short 图书馆管.004FD842            <------//异常跳转，一般OEP离此不远了。
004FD841     87EB                xchg ebx,ebp
004FD843     0000                add byte ptr ds:[eax],al
004FD845     0000                add byte ptr ds:[eax],al
............................
来到这里
004FD842     0000                add byte ptr ds:[eax],al <------//出现访问违反异常，按shift+F8跳过。
004FD844     0000                add byte ptr ds:[eax],al
004FD846     0000                add byte ptr ds:[eax],al
004FD848     0000                add byte ptr ds:[eax],al
004FD84A     0000                add byte ptr ds:[eax],al
004FD84C     0000                add byte ptr ds:[eax],al
004FD84E     0000                add byte ptr ds:[eax],al
............................
00402912   - FF25 30114000       jmp dword ptr ds:[401130]             ; MSVBVM60.\_\_vbaVarTstNe
00402918   - FF25 48114000       jmp dword ptr ds:[401148]             ; MSVBVM60.\_\_vbaVarCopy
0040291E   - FF25 18104000       jmp dword ptr ds:[401018]             ; MSVBVM60.\_\_vbaVarMove
00402924   - FF25 6C114000       jmp dword ptr ds:[40116C]             ; MSVBVM60.\_\_vbaVarForNext
0040292A   - FF25 6C104000       jmp dword ptr ds:[40106C]             ; MSVBVM60.\_\_vbaVarForInit
00402930   - FF25 5C114000       jmp dword ptr ds:[40115C]             ; MSVBVM60.\_\_vbaStrVarCopy
00402936   - FF25 B0104000       jmp dword ptr ds:[4010B0]             ; MSVBVM60.\_\_vbaAryConstruct2
0040293C   - FF25 38114000       jmp dword ptr ds:[401138]             ; MSVBVM60.\_\_vbaVarAdd
00402942   - FF25 04104000       jmp dword ptr ds:[401004]             ; MSVBVM60.\_\_vbaVarSub
00402948   - FF25 50104000       jmp dword ptr ds:[401050]             ; MSVBVM60.rtcVarFromFormatVar
0040294E   - FF25 D8104000       jmp dword ptr ds:[4010D8]             ; MSVBVM60.EVENT\_SINK\_QueryInterface
00402954   - FF25 9C104000       jmp dword ptr ds:[40109C]             ; MSVBVM60.EVENT\_SINK\_AddRef
0040295A   - FF25 CC104000       jmp dword ptr ds:[4010CC]             ; MSVBVM60.EVENT\_SINK\_Release
00402960   - FF25 C8104000       jmp dword ptr ds:[4010C8]             ; MSVBVM60.Zombie\_GetTypeInfoCount
00402966   - FF25 64104000       jmp dword ptr ds:[401064]             ; MSVBVM60.Zombie\_GetTypeInfo
0040296C   - FF25 00104000       jmp dword ptr ds:[401000]             ; MSVBVM60.EVENT\_SINK\_GetIDsOfNames
00402972   - FF25 38104000       jmp dword ptr ds:[401038]             ; MSVBVM60.EVENT\_SINK\_Invoke
00402978   - FF25 84104000       jmp dword ptr ds:[401084]             ; MSVBVM60.GetMemStr
0040297E   - FF25 90104000       jmp dword ptr ds:[401090]             ; MSVBVM60.PutMemStr
00402984   - FF25 2C114000       jmp dword ptr ds:[40112C]             ; MSVBVM60.ThunRTMain
0040298A     0000                add byte ptr ds:[eax],al
0040298C     68 68314000         push 图书馆管.00403168                ; ASCII "VB5!6&vb6chs.dll"
00402991     E8 EEFFFFFF         call 图书馆管.00402984                ; jmp to MSVBVM60.ThunRTMain <------//停在这里，提示被调试的程序无法处理异常，进程终止。
```

接下来又重复了几遍，仍然如此。后来，仔细分析了出错地址代码及注释，发现了一个问题：每次都是出现访问违反异常后按F8从004FD842直接跳到00402991，然后进程终止，为什么没有经过上一行呢？于是重新载入主程序，清除所有断点，试着直接在命令行中下硬件中断`HE 0040298C`，F9运行，跳过004FD842处的访问违反异常后，程序断在了0040298C，再按F8单步跟踪，这次在00402991处没有提示无法处理，程序正常运行，又试了几次，都成功运行，所以初步断定0040298C就是OEP。接下来用OD脱壳插件直接在当前调试进程中成功脱壳，程序运行正常。下面发生的事让我百思不得其解：本想正式分析和狗相关代码以便脱狗运行，谁知一个意外却使我到此收工了，去掉狗看看出错提示时，结果没有狗脱壳后的程序也正常运行了。运行未脱壳程序，找到出错提示，在脱壳后的主程序中查找，居然找不到，个人认为这个软件是把狗和壳捆帮在一起了。本人菜鸟一个，对加密狗接触的也不多，请各位大虾多提意见，多批评指正。

【破解总结】：

主要工作是脱壳的过程。查壳得知是yoda's cryptor 1.x / modified，用脱壳机不能正确脱壳，估计做了伪装，手工脱壳注意几个循环和回跳，最后是出错分析，个人感觉，经验比较重要。无狗的情况下也跟了很久，但搞不定，由于已和软件作者联系，待版本升级后再提供此版本供大家练习。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
