
# Aspack 2.2 简易分析

> 原文：`www.pediy.com/kssd/pediy11/106616.html`

**文章标题**：Aspack 2.2 简易分析

**文章作者**：`wuhanqi`

**下载地址**：自己搜索下载

**作者声明**：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

## 详细过程

这次分析用的 `ASPACK` 是 `Regkiller` 工具包里面的那个。

随便找了一个 `Delphi` 的加壳...

```
0041B001  >  60            PUSHAD                     ; EP
0041B002    E8 03000000     CALL 0041B00A
0041B007    90            NOP                      ; 花指令 nop
0041B008    EB 04          JMP SHORT 0041B00E
0041B00A    5D            POP EBP
0041B00B    45            INC EBP
0041B00C    55            PUSH EBP
0041B00D    C3            RETN
0041B00E    E8 01000000     CALL 0041B014
0041B013    90            NOP                      ; 花指令 nop
0041B014    5D            POP EBP                  ; 0041b013 出栈到 EBP
0041B015    BB EDFFFFFF     MOV EBX,-13                  ; EBX=-13
0041B01A    03DD          ADD EBX,EBP                  ; EBX=EBP+EBX=0041B000
0041B01C    81EB 00B00100   SUB EBX,1B000                ; EBX=EBX-1B000=00400000 获取基址
0041B022    83BD 7D040000 0> CMP DWORD PTR SS:[EBP+47D],0     ; 比较 EBP+47D 处数据是否为 0
0041B029    899D 7D040000   MOV DWORD PTR SS:[EBP+47D],EBX    ; 把基址放到 EBP+47D 处
0041B02F    0F85 C0030000   JNZ 0041B3F5                 ; EBP+47D 不为零的话就去 OEP！
0041B035    8D85 89040000   LEA EAX,DWORD PTR SS:[EBP+489]   ; 获取 kernel32.dll 放到 eax
0041B03B    50            PUSH EAX                   ; kernel32.dll 压栈
0041B03C    FF95 090F0000   CALL DWORD PTR SS:[EBP+F09]      ; 调用 GetModuleHandleA 获取 kernel32 的基址
0041B042    8985 81040000   MOV DWORD PTR SS:[EBP+481],EAX    ; kernel32 的基址到 EBP+481
0041B048    8BF0          MOV ESI,EAX                  ; ESI=EAX
0041B04A    8D7D 51       LEA EDI,DWORD PTR SS:[EBP+51]     ; 获取 VirtualAlloc 放到 EDI
0041B04D    57            PUSH EDI                   ; EDI 压栈
0041B04E    56            PUSH ESI                   ; kernel32 的基址压栈
0041B04F    FF95 050F0000   CALL DWORD PTR SS:[EBP+F05]      ; 调用 GetProcAddress 获取 EDI 的地址
0041B055    AB            STOSD                    ; 把获取到的地址放到原来字符串所在的位置
0041B056    B0 00         MOV AL,0
0041B058    AE            SCASB
0041B059  ^ 75 FD        JNZ SHORT 0041B058             ; 上面几句比较难读，作用是获取下一个函数名称
0041B05B    3807          CMP BYTE PTR DS:[EDI],AL
0041B05D  ^ 75 EE        JNZ SHORT 0041B04D             ; 一共获取了这三个函数：VirtualAlloc VirtualFree VirtualProtect
0041B05F    8D45 7A       LEA EAX,DWORD PTR SS:[EBP+7A]     ; EAX=0041b013+7a
0041B062    FFE0          JMP EAX                    ; 去 EAX 所指向的地址
```

继续：

```
0041B08D  .  8B9D 8D050000 MOV EBX,DWORD PTR SS:[EBP+58D]    ;  41b013+58d 的 DWORD 到 EBX
0041B093  .  0BDB        OR EBX,EBX                   ;  看看是不是零
0041B095  .  74 0A       JE SHORT 0041B0A1              ;  是就跳
0041B097  .  8B03        MOV EAX,DWORD PTR DS:[EBX]
0041B099  .  8785 91050000 XCHG DWORD PTR SS:[EBP+591],EAX
0041B09F  .  8903        MOV DWORD PTR DS:[EBX],EAX
0041B0A1  >  8DB5 BD050000 LEA ESI,DWORD PTR SS:[EBP+5BD]    ;  获取保存第一个区段的偏移 1000 的地址 放到 ESI
0041B0A7  .  833E 00      CMP DWORD PTR DS:[ESI],0          ;  看看偏移的值是不是 0
0041B0AA  .  0F84 15010000 JE 0041B1C5                  ;  是就跳
0041B0B0  .  6A 04       PUSH 4
0041B0B2  .  68 00100000 PUSH 1000
0041B0B7  .  68 00180000 PUSH 1800
0041B0BC  .  6A 00       PUSH 0
0041B0BE  .  FF55 51     CALL DWORD PTR SS:[EBP+51]        ;  调用了 VirtualAlloc 申请了一段 1800 大小的空间
0041B0C1  .  8985 53010000 MOV DWORD PTR SS:[EBP+153],EAX    ;  把获取到的地址放到 EBP+153
0041B0C7  >  8B46 04      MOV EAX,DWORD PTR DS:[ESI+4]      ;  源程序获取第一个区段的大小到 EAX
0041B0CA  .  05 0E010000  ADD EAX,10E                    ;  EAX=EAX+10E
0041B0CF  .  6A 04       PUSH 4
0041B0D1  .  68 00100000 PUSH 1000
0041B0D6  .  50          PUSH EAX
0041B0D7  .  6A 00       PUSH 0
0041B0D9  .  FF55 51     CALL DWORD PTR SS:[EBP+51]        ;  调用了 VirtualAlloc 申请了一段 EAX 大小的空间
0041B0DC  .  8985 4F010000 MOV DWORD PTR SS:[EBP+14F],EAX    ;  申请的地址到 EBP+14F
0041B0E2  .  56          PUSH ESI                   ;  ESI 压栈
0041B0E3  .  8B1E        MOV EBX,DWORD PTR DS:[ESI]       ;  第一个区段的偏移到 ebx
0041B0E5  .  039D 7D040000 ADD EBX,DWORD PTR SS:[EBP+47D]   ;  偏移与基址相加 00401000
0041B0EB  .  FFB5 53010000 PUSH DWORD PTR SS:[EBP+153]     ;  第一次申请的空间地址入栈
0041B0F1  .  FF76 04     PUSH DWORD PTR DS:[ESI+4]        ;  第一个区段大小入栈
0041B0F4  .  50          PUSH EAX                   ;  第二个空间地址入栈
0041B0F5  .  53          PUSH EBX                   ;  00401000 入栈
0041B0F6  .  E8 2D050000 CALL 0041B628                ;  开始解码
0041B0FB  .  B3 01       MOV BL,1                     ;  bl=0
0041B0FD  .  80FB 00      CMP BL,0                     ;  比较是否为 0
0041B100  .  75 5E       JNZ SHORT 0041B160               ;  不为零就跳
0041B102  .  FE85 E9000000 INC BYTE PTR SS:[EBP+E9]       ;  更改上面的 mov bl,0 为 mov bl,1 不太懂这几个指令干嘛的..
0041B108  .  8B3E        MOV EDI,DWORD PTR DS:[ESI]       ;  EDI 等于第一个区段的偏移
0041B10A  .  03BD 7D040000 ADD EDI,DWORD PTR SS:[EBP+47D]   ;  EDI=EDI+基址=00401000
0041B110  .  FF37        PUSH DWORD PTR DS:[EDI]           ;  把 00401000 处的 Dword 压栈
0041B112  .  C607 C3      MOV BYTE PTR DS:[EDI],0C3        ;  修改第一个字节为 C3
0041B115  .  FFD7        CALL EDI                     ;  然后 call 00401000 被刚刚修改的 c3 给退出来了...
0041B117  .  8F07        POP DWORD PTR DS:[EDI]           ;  再恢复数据 晕 作孽啊
0041B119  .  50          PUSH EAX                   ;  第一个区段大小入栈
0041B11A  .  51          PUSH ECX                   ;  ECX 入栈 此时为 1
0041B11B  .  56          PUSH ESI                   ;  把存放偏移的地址压栈
0041B11C  .  53          PUSH EBX                   ;  00401000 入栈
0041B11D  .  8BC8        MOV ECX,EAX                  ;  ECX=EAX
0041B11F  .  83E9 06      SUB ECX,6                    ;  ECX=ECX-6
0041B122  .  8BB5 4F010000 MOV ESI,DWORD PTR SS:[EBP+14F]   ;  第二次申请的地址到 ESI
0041B128  .  33DB        XOR EBX,EBX                  ;  清零 EBX
0041B12A  >  0BC9        OR ECX,ECX                   ;  看看 ecx 是不是 0
0041B12C  .  74 2E       JE SHORT 0041B15C              ;  是就跑
0041B12E  .  78 2C       JS SHORT 0041B15C
0041B130  .  AC          LODSB                      ;  逐个获取第二次申请的空间里面的字节 放到 al
0041B131  .  3C E8       CMP AL,0E8                   ;  看看是不是 E8
0041B133  .  74 0A       JE SHORT 0041B13F              ;  是就跳转
0041B135  .  EB 00       JMP SHORT 0041B137
0041B137  >  3C E9       CMP AL,0E9                   ;  看看是不是 E9
0041B139  .  74 04       JE SHORT 0041B13F              ;  是就跳转
0041B13B  >  43          INC EBX                      ;  否则 EBX+1
```



```
0041B13C   .  49             DEC ECX                                       ;  ECX-1  
0041B13D   .^ EB EB          JMP SHORT 0041B12A  
0041B13F   >  8B06           MOV EAX,DWORD PTR DS:[ESI]                   ;  把 E8/E9 后面的 DWORD 放到 EAX  
0041B141   .  EB 00         JMP SHORT 0041B143  
0041B143   >  803E 00        CMP BYTE PTR DS:[ESI],0                       ;  看看 E8/E9 后第一个字节是不是 0  
0041B146   .^ 75 F3         JNZ SHORT 0041B13B                              ;  不是就跳了  
0041B148   .  24 00         AND AL,0                                      ;  AL=AL and 0  
0041B14A   .  C1C0 18       ROL EAX,18                                     ;  eax=eax rol 18  
0041B14D   .  2BC3          SUB EAX,EBX                                    ;  eax=eax-ebx  
0041B14F   .  8906          MOV DWORD PTR DS:[ESI],EAX                     ;  把运算后的 EAX 放回去  
0041B151   .  83C3 05      ADD EBX,5                                      ;  EBX=EBX+5  
0041B154   .  83C6 04      ADD ESI,4                                      ;  ESI=ESI+4  
0041B157   .  83E9 05      SUB ECX,5                                      ;  ECX=ECX-4  
0041B15A   .^ EB CE         JMP SHORT 0041B12A                              ;  跳回去，敢情上面在修复跳转和 call  
0041B15C   >  \5B            POP EBX                                        ;  把之前压进去的东西都弹出来  
0041B15D   .  5E            POP ESI  
0041B15E   .  59            POP ECX  
0041B15F   .  58            POP EAX  
0041B160   >  EB 08         JMP SHORT 0041B16A  
继续：  
0041B16A   >  \8BC8          MOV ECX,EAX                                    ;  第一个区段大小到 ECX  
0041B16C   .  8B3E          MOV EDI,DWORD PTR DS:[ESI]                     ;  偏移到 EDI  
0041B16E   .  03BD 7D040000 ADD EDI,DWORD PTR SS:[EBP+47D]                 ;  偏移加上基址=00401000  
0041B174   .  8BB5 4F010000 MOV ESI,DWORD PTR SS:[EBP+14F]                 ;  第二次申请的空间地址到 ESI  
0041B17A   .  C1F9 02       SAR ECX,2                                      ;  ECX 算术右移两位  
0041B17D   .  F3:A5         REP MOVSD                                      ;  转移数据咯  
0041B17F   .  8BC8          MOV ECX,EAX                                    ;  ECX=EAX  
0041B181   .  83E1 03      AND ECX,3                                      ;  ECX=ECX AND 3    
0041B184   .  F3:A4         REP MOVSB                                      ;  再转移数据..  
0041B186   .  5E            POP ESI                                        ;  存放偏移的地址出栈  
0041B187   .  68 00800000   PUSH 8000  
0041B18C   .  6A 00         PUSH 0  
0041B18E   .  FFB5 4F010000 PUSH DWORD PTR SS:[EBP+14F]  
0041B194   .  FF55 5E       CALL DWORD PTR SS:[EBP+5E]                     ;  释放刚刚申请的空间  
0041B197   .  83C6 0C      ADD ESI,0C                                       ;  ESI=ESI+0C  
0041B19A   .  833E 00      CMP DWORD PTR DS:[ESI],0                        ;  看看 ESI 处 dword 东西是不是 0  
0041B19D   .^ 0F85 24FFFFFF JNZ 0041B0C7                                    ;  这边回去循环了 就第一个区段变第二个区段..  
0041B1A3   .  68 00800000   PUSH 8000  
0041B1A8   .  6A 00         PUSH 0  
0041B1AA   .  FFB5 53010000 PUSH DWORD PTR SS:[EBP+153]  
0041B1B0   .  FF55 5E       CALL DWORD PTR SS:[EBP+5E]                     ;  把第一次申请的空间释放掉  
…………………… 个人感觉中间这部分代码无意义 程序都跳过去了 我就不分析了 在此略过 ……………………  
0041B26F     BE 00A00000     MOV ESI,0A000                                  ; ESI=A000  
0041B274     8B95 7D040000   MOV EDX,DWORD PTR SS:[EBP+47D]               ; 基址到 EDX  
0041B27A     03F2            ADD ESI,EDX                                    ; ESI=ESI+EDX=0040A000  
0041B27C     8B46 0C         MOV EAX,DWORD PTR DS:[ESI+C]                 ; esi+c 的数据到 EAX  
0041B27F     85C0            TEST EAX,EAX                                   ; 看看是不是 0  
0041B281     0F84 0D010000   JE 0041B394                                    ; 是就跳  
0041B287     03C2            ADD EAX,EDX                                    ; EAX=EAX+基址 获取了 DLL 名称  
0041B289     8BD8            MOV EBX,EAX                                    ; EBX=EAX  
0041B28B     50             PUSH EAX                                       ; EAX 压栈  
0041B28C     FF95 090F0000   CALL DWORD PTR SS:[EBP+F09]                   ; 调用 `GetModuleHandleA` 获取 DLL 基址  
0041B292     85C0            TEST EAX,EAX                                   ; 看看获取到了没  
0041B294     75 07           JNZ SHORT 0041B29D                              ; 获取到了就跳  
0041B296     53             PUSH EBX                                       ; 否则会载入这个 DLL  
0041B297     FF95 0D0F0000   CALL DWORD PTR SS:[EBP+F0D]  
0041B29D     8985 A1050000   MOV DWORD PTR SS:[EBP+5A1],EAX               ; KERNEL32 的基址放到 EBP+5A1  
0041B2A3     C785 A5050000 0>MOV DWORD PTR SS:[EBP+5A5],0                 ; EBP+5A5=0 清零了存放 KERNEL32 基址后的那个 DWORD  
0041B2AD     8B95 7D040000   MOV EDX,DWORD PTR SS:[EBP+47D]               ; 程序基址到 EDX  
0041B2B3     8B06            MOV EAX,DWORD PTR DS:[ESI]                   ; ESI 的数据到 EAX  
0041B2B5     85C0            TEST EAX,EAX                                   ; 看看是不是 0  
0041B2B7     75 03           JNZ SHORT 0041B2BC                              ; 是就跳  
0041B2B9     8B46 10         MOV EAX,DWORD PTR DS:[ESI+10]                ; ESI+10 的数据放到 EAX  
0041B2BC     03C2            ADD EAX,EDX                                    ; EAX=EAX+基址=0040a1d0  
0041B2BE     0385 A5050000   ADD EAX,DWORD PTR SS:[EBP+5A5]               ; EAX = EAX 加上 EBP+5A5 的数据  
0041B2C4     8B18            MOV EBX,DWORD PTR DS:[EAX]                   ; 把 EAX 的数据放到 EBX  
0041B2C6     8B7E 10         MOV EDI,DWORD PTR DS:[ESI+10]                ; ESI+10 的数据放到 EAX  
0041B2C9     03FA            ADD EDI,EDX                                    ; EDI 加上基址  
0041B2CB     03BD A5050000   ADD EDI,DWORD PTR SS:[EBP+5A5]               ; EDI 与 EBP+5A5 的数据做加法运算 此时 EDI=EAX  
0041B2D1     85DB            TEST EBX,EBX                                   ; 看看 EBX 是不是零 此时 EBX 是从 eax 获取到的数据  
0041B2D3   /0F84 A5000000   JE 0041B37E                                    ; 是就跳转 结束填充 IAT  
0041B2D9     F7C3 00000080   TEST EBX,80000000                             ; 看看等不等于 80000000  
0041B2DF     75 04           JNZ SHORT 0041B2E5                              ; 相等就跳  
0041B2E1     03DA            ADD EBX,EDX                                    ; EBX=EBX+基址  
0041B2E3     43             INC EBX                                        ; EBX+1  
0041B2E4     43             INC EBX                                        ; EBX+1  
0041B2E5     53             PUSH EBX                                       ; EBX 压栈 此时获取到了函数名称  
0041B2E6     81E3 FFFFFF7F   AND EBX,7FFFFFFF                               ; EBX = EBX AND 7FFFFFFF  
0041B2EC     53             PUSH EBX                                       ; EBX 再压栈  
0041B2ED     FFB5 A1050000   PUSH DWORD PTR SS:[EBP+5A1]                   ; 把 kernel32 的基址压栈  
0041B2F3     FF95 050F0000   CALL DWORD PTR SS:[EBP+F05]                   ; `GetProcAddress` 获取函数地址  
0041B2F9     85C0            TEST EAX,EAX                                   ; 看看获取到没  
0041B2FB     5B             POP EBX                                        ; 之前压了两次函数名 出栈一次 数据放到 EBX  
0041B2FC     75 72          JNZ SHORT 0041B370                               ; 获取到了就跳转  
跳转来到：  
0041B370   >  \8907           MOV DWORD PTR DS:[EDI],EAX                   ;  填充数据到 EDI 即是填充 IAT 修复输入表  
0041B372   .  8385 A5050000>ADD DWORD PTR SS:[EBP+5A5],4                 ;  [EBP+5A5]+4  
0041B379   .^ E9 2FFFFFFF    JMP 0041B2AD                                   ;  跳回去  
…………………………中间还调用了几次 `VirtualProtect` 不知道干吗的 没看懂 不过感觉壳该干完的好像差不多了 遂没分析 ……………………  
0041B3F5     B8 5C670000     MOV EAX,675C                                   ; EAX=OEP 的 VA  
0041B3FA     50             PUSH EAX                                       ; 压栈  
0041B3FB     0385 7D040000   ADD EAX,DWORD PTR SS:[EBP+47D]               ; EAX=EAX 加基址  
0041B401     59             POP ECX                                        ; ECX 出栈 此时 ecx 为 OEP 的 VA  
0041B402     0BC9           OR ECX,ECX                                     ; 看看是不是 0  
0041B404     8985 03040000   MOV DWORD PTR SS:[EBP+403],EAX               ; 把 OEP 放到 EBP+403 即修改下面的 push 方便我们去 OEP  
0041B40A     61             POPAD  
```



```
  0041B40B    75 08           JNZ SHORT 0041B415                   ; 不是 0 就跳

  0041B40D    B8 01000000     MOV EAX,1

  0041B412    C2 0C00         RETN 0C

  0041B415    68 5C674000     PUSH 0040675C                         ; OEP 压栈

  0041B41A    C3              RETN                                  ; ...然后就该干嘛干嘛
```

--------------------------------------------------------------------------------

**【经验总结】**

第一次分析压缩壳...不知道哪里会有错，希望大牛们能指出

能有基础分析这个要感谢天草的矛与盾的趣味，讲的非常非常细.. 还有网上做过分析的前辈们，或多或少的我可能都看过

文章，呵呵~

夜深人静脑子不好使，错了大家千万别骂我，希望你能指出，觉得好的，不吝啬的，就加点`UB.UF`啦，嘿嘿~

--------------------------------------------------------------------------------

**【版权声明】**: 依旧没版权..学习学习再学习！

2010 年 02 月 03 日 23:46:38

本代码的着色效果由`xTiNt`自动完成
