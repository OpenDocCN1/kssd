

# Aspack 2.12

> 原文：[`www.pediy.com/kssd/pediy05/pediy50134.htm`](https://www.pediy.com/kssd/pediy05/pediy50134.htm)

用 ollydbg 脱 aspack2.12 的壳   
  
\================================================================================   
作者      : CoolWolF/81n.net   
目标      : 用 aspack2.12 加了壳的记事本(版本号:5.0.2140.1)   
使用工具  : aspack2.12, Ollydbg1.7b, LordPE, Import REConstructor1.42, PEiD0,7b   
环境      : Windows 2000 Server SP2   
\================================================================================   
本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!   
\================================================================================   
  
  
    这是小弟第一篇脱壳的文章, 水平有限, 请大家多指点. (说来惭愧, 2002 年 5 月之前, 我还没有手动脱壳的经验.)   
  
    首先, 用 aspack2.12 把记事本加壳, 用 PEiD 看了一下, 说是"ASPack 2.x modified - DAMN". 呵呵.   
    打开 Ollydbg, 加载 notepad.exe, 弹出一个对话窗. 不用管他, 按确定. 接着弹出另一个对话窗告诉你程序可能被压缩过, 是否继续, 还是按确定按钮.   
    程序停在入口处, 按 F8 单步执行:   
  
01010001 > 60              PUSHAD //入口    
01010002  E8 03000000      CALL notepad.0101000A //进去,  过了这个 Call 程序就执行了   
01010007  -E9 EB045D45      JMP 465E04F7   
0101000C  55              PUSH EBP   
0101000D  C3              RETN   
0101000E  E8 01000000      CALL notepad.01010014   
  
0101000A  5D              POP EBP   
0101000B  45              INC EBP                                  ; notepad.01010007   
0101000C  55              PUSH EBP   
0101000D  C3              RETN   
0101000E  E8 01000000      CALL notepad.01010014 //进去~   
  
    一路 F8, 直到:   
  
0101012D  0BC9            OR ECX,ECX   
0101012F  74 2E            JE SHORT notepad.0101015F   
01010131  78 2C            JS SHORT notepad.0101015F   
01010133  AC              LODS BYTE PTR DS:\[ESI\]   
01010134  3C E8            CMP AL,0E8   
01010136  74 0A            JE SHORT notepad.01010142   
01010138  EB 00            JMP SHORT notepad.0101013A   
0101013A  3C E9            CMP AL,0E9   
0101013C  74 04            JE SHORT notepad.01010142   
0101013E  43              INC EBX   
0101013F  49              DEC ECX   
01010140  ^EB EB            JMP SHORT notepad.0101012D    
01010142  8B06            MOV EAX,DWORD PTR DS:\[ESI\]   
01010144  EB 00            JMP SHORT notepad.01010146   
01010146  803E 07          CMP BYTE PTR DS:\[ESI\],7   
01010149  ⁷⁵ F3            JNZ SHORT notepad.0101013E   
0101014B  24 00            AND AL,0   
0101014D  C1C0 18          ROL EAX,18   
01010150  2BC3            SUB EAX,EBX   
  
  
    程序在这里循坏, 我们可没时间等, 看来最远的地方就是 0101015F, 在 0101015F 按 F2 下断, F9 跳过后继续疯狂的 F8:   
  
010102B6  8B95 22040000    MOV EDX,DWORD PTR SS:\[EBP+422\]          ; notepad.01000000   
010102BC  8B06            MOV EAX,DWORD PTR DS:\[ESI\]   
010102BE  85C0            TEST EAX,EAX   
010102C0  75 03            JNZ SHORT notepad.010102C5   
010102C2  8B46 10          MOV EAX,DWORD PTR DS:\[ESI+10\]   
010102C5  03C2            ADD EAX,EDX                              ; notepad.01000000   
010102C7  0385 49050000    ADD EAX,DWORD PTR SS:\[EBP+549\]   
010102CD  8B18            MOV EBX,DWORD PTR DS:\[EAX\]   
010102CF  8B7E 10          MOV EDI,DWORD PTR DS:\[ESI+10\]   
010102D2  03FA            ADD EDI,EDX   
010102D4  03BD 49050000    ADD EDI,DWORD PTR SS:\[EBP+549\]   
010102DA  85DB            TEST EBX,EBX   
010102DC  0F84 A2000000    JE notepad.01010384   
010102E2  F7C3 00000080    TEST EBX,80000000   
010102E8  75 04            JNZ SHORT notepad.010102EE   
010102EA  03DA            ADD EBX,EDX   
010102EC  43              INC EBX   
010102ED  43              INC EBX   
010102EE  53              PUSH EBX   
010102EF  81E3 FFFFFF7F    AND EBX,7FFFFFFF   
010102F5  53              PUSH EBX   
010102F6  FFB5 45050000    PUSH DWORD PTR SS:\[EBP+545\]   
010102FC  FF95 490F0000    CALL DWORD PTR SS:\[EBP+F49\]   
01010302  85C0            TEST EAX,EAX   
01010304  5B              POP EBX   
01010305  75 6F            JNZ SHORT notepad.01010376 //下去   
01010307  F7C3 00000080    TEST EBX,80000000   
0101030D  75 19            JNZ SHORT notepad.01010328   
0101030F  57              PUSH EDI   
01010376  8907            MOV DWORD PTR DS:\[EDI\],EAX              ; msvcrt.\_wtol   
01010378  8385 49050000 04 ADD DWORD PTR SS:\[EBP+549\],4   
0101037F  ^E9 32FFFFFF      JMP notepad.010102B6 // 又回去了   
  
    看看这次是到什么地方? 可以走到 01010384 :   
  
01010384  8906            MOV DWORD PTR DS:\[ESI\],EAX   
01010386  8946 0C          MOV DWORD PTR DS:\[ESI+C\],EAX   
01010389  8946 10          MOV DWORD PTR DS:\[ESI+10\],EAX   
0101038C  83C6 14          ADD ESI,14   
0101038F  8B95 22040000    MOV EDX,DWORD PTR SS:\[EBP+422\]   
01010395  ^E9 EBFEFFFF      JMP notepad.01010285 //还是要回去继续循环   
0101039A  B8 20640000      MOV EAX,6420   
  
    那我们就大胆一点, 在 0101039A 下断, F9..... 拦住了. 继续 F8(这里就不能像刚才那么疯狂了, 因为马上我们就要到达真正的入口)   
  
0101039A  B8 20640000      MOV EAX,6420   
0101039F  50              PUSH EAX   
010103A0  0385 22040000    ADD EAX,DWORD PTR SS:\[EBP+422\]   
010103A6  59              POP ECX   
010103A7  0BC9            OR ECX,ECX   
010103A9  8985 A8030000    MOV DWORD PTR SS:\[EBP+3A8\],EAX   
010103AF  61              POPAD   
010103B0  75 08            JNZ SHORT notepad.010103BA   
010103B2  B8 01000000      MOV EAX,1   
010103B7  C2 0C00          RETN 0C   
010103BA  68 20640001      PUSH notepad.01006420 //这是什么?   
010103BF  C3              RETN  //过了这里, 你会看到....   
  
    好了~~ 当执行到 010103BF 的时候, 我们就可以打开 LoadPe, 找到 notepad.exe 的进程, 选择 dump full, 保存为 dumped.exe, 再用 PEiD 看看: 还是"ASPack 2.x modified - DAMN". 恩...? 要对自己有信心. 不要怀疑!   
    把 Ollydbg 关闭掉, 运行加了壳的 notepad.exe, 然后打开 Import REConstructor, 选择 notepad.exe 进程. 这时候显示程序的 OEP 是 00010001, 填入我们刚才找到的 00006420(因为 ImageBase 是 01000000, 所以要减去). 按 IAT AutoSearch 按钮, 出现对话窗 Found Something, 确定后再按 Get Imports 修复. 看来我们运气不错, 都自动完成了. 那么就按 Fix Dump, 选择刚才保存的 dumped.exe. 然后就可以退出 Import REConstructor 了.   
    再用 PEiD 一看 dumped\_.exe: "Microsoft Visual C++ 5.0". 哈哈! 运行一下呢? 也很正常. 最后用 LoadPe 的 Rebuild PE 重建一下, 这时候文件的大小是 69.6K, 算是完美脱壳了!   
    本来是想写新版 FSG 1.2 的脱壳, 但整理起来很耗时间, 所以就拿一个好对付的来说了, 哈哈.   
  
    感谢众多前辈的精彩文章   
    献给所有和我一样的脱壳菜鸟   
\=========================================================================   
(完)  4dd466d0e1e0cc123afbe504da2462ca