

# 关于 MoleBox 打包文件的一点研究

> 原文：[`www.pediy.com/kssd/pediy11/115123.html`](https://www.pediy.com/kssd/pediy11/115123.html)

经过我的一点很粗浅的研究,MoleBox 将多个文件打包在一起,  
主要是通加密文件操作相关的 IAT..只是替换..  
因此我的方法是.脱壳后,或是找到 OEP 后..  
就可以看到一个  
00547AF8    FF15 1CC25500   CALL NEAR DWORD PTR \[55C21C\]             ; kernel32.GetCommandLineA  
00547AFE    A3 C0796804     MOV DWORD PTR \[46879C0\],EAX  
  
数据窗口查看 55c21c 地址上下范围长地址查看.那都 是 IAT 表了..  
没有表示出 winapi 名字就是被替换了...  
方法一  
在替换的地方可以下硬改写断点...重新调试,F9 三次就可以看到原来的,再一次,就改了..在改成最新的地方下断点如我的是.  
04692666    FF15 40E86904   CALL NEAR DWORD PTR \[469E840\]            ; kernel32.VirtualProtect  
0469266C    85C0            TEST EAX,EAX  
0469266E    75 0A           JNZ SHORT 永恒 Onli.0469267A  
04692670    B9 0B0000EF     MOV ECX,EF00000B  
04692675    E8 9D2F0000     CALL 永恒 Onli.04695617  
0469267A    8B4D 08         MOV ECX,DWORD PTR \[EBP+8\]  
0469267D    8B55 F8         MOV EDX,DWORD PTR \[EBP-8\]  
04692680    8B02            MOV EAX,DWORD PTR \[EDX\]  
04692682    8901            MOV DWORD PTR \[ECX\],EAX  ;这个地方改了 IAT....一会 NOP 掉就可以了  
04692684    8D4D F4         LEA ECX,DWORD PTR \[EBP-C\]  
04692687    51              PUSH ECX  
04692688    8B55 F0         MOV EDX,DWORD PTR \[EBP-10\]  
0469268B    52              PUSH EDX  
0469268C    6A 04           PUSH 4  
0469268E    8B45 08         MOV EAX,DWORD PTR \[EBP+8\]  
04692691    50              PUSH EAX  
04692692    FF15 40E86904   CALL NEAR DWORD PTR \[469E840\]            ; kernel32.VirtualProtect  
  
  
要全恢复  
在 VirtualProtect 下断点.F9 二次,一开始这些地方没有解出来,执行二次刚好解好.这时就可以到刚才改 IAT 为新的地主 04692682 NOP 掉...  
去断点,执行.原来的 IAT 就全出来了...  
  
现在就可很方便的看 readFile.在那个地方了....