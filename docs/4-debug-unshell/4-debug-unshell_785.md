

# Obsidium1.0.0.69

> 原文：[`www.pediy.com/kssd/pediy06/pediy6968.htm`](https://www.pediy.com/kssd/pediy06/pediy6968.htm)

Obsidium1.0.0.69 未注册版之 VB 脱壳篇   
  
  
【目    标】:我自己写一个小 VB 小程序(要有行楷字体,才能显示,不过看不看到不要紧，最主要是脱壳)   
【工    具】:Olydbg1.1(diy 版)、LordPE、ImportREC1.6F   
【操作平台】:WINDOWS 2000 ad server   
【任    务】:简单的脱 obs 加的 vb 的程序+修复   
【作    者】:loveboom   
【相关链接】:点击下载  
【简要说明】:为什么说是 VB 篇呢?因为若是其它类的，它会改载入段地址的.不能直接修复(可能是我水平太菜了)   
【详细过程】:   
  
设置:内存异常项不要选.   
  
任务一:找到入口   
  
00407000   EB 02            JMP SHORT Obsidium.00407004    ；载入时所在位置   
00407002   881D E8A31C00    MOV BYTE PTR DS:\[1CA3E8\],BL   
  
F9 运行一次出现异常:   
(按 SHIFT+F9 继续)   
  
0041154C   CD 01            INT 1    ；第一次异常   
0041154E   40               INC EAX   
……   
004119E3   A5               MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]  ；第二次异常   
004119E4   EB 02            JMP SHORT Obsidium.004119E8   
……   
  
这次后再按 SHIFT+F9 就出现未注册提示，点确定到这里：   
  
00870C31   CD 01            INT 1        ；停在这里 SHIFT+F9 继续   
00870C33   40               INC EAX       
……   
00870FB8   800406 00        ADD BYTE PTR DS:\[ESI+EAX\],0    ；从这里开始算异常，这里第一次   
00870FBC   EB 03            JMP SHORT 00870FC1   
……   
008713D2   8B00             MOV EAX,DWORD PTR DS:\[EAX\]    ；第二次异常   
008713D4   EB 01            JMP SHORT 008713D7   
……   
008719AE   8B00             MOV EAX,DWORD PTR DS:\[EAX\]    ；第三次异常   
008719B0   EB 01            JMP SHORT 008719B3   
……   
00871BF6   CD 68            INT 68              ；第四次异常   
00871BF8   EB 03            JMP SHORT 00871BFD   
……   
008720D6   8B02             MOV EAX,DWORD PTR DS:\[EDX\]    ；第五次异常   
008720D8   EB 01            JMP SHORT 008720DB   
……   
0087386B   0FB753 06        MOVZX EDX,WORD PTR DS:\[EBX+6\]  ；第六次异常   
0087386F   EB 02            JMP SHORT 00873873   
……   
00873F81   800406 00        ADD BYTE PTR DS:\[ESI+EAX\],0    ；第七次异常   
00873F85   EB 02            JMP SHORT 00873F89   
……   
0087510A   A5               MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]  ；第八次   
0087510B   EB 02            JMP SHORT 0087510F   
……   
0087543D   CD 01            INT 1      ；第九次   
0087543F   40               INC EAX   
  
第九次后，再按一次程序就运行了，真的是最后的异常吗?非也！到这里后取消全部异常前的钩，再 SHIFT+F9，这样又会多回两次异常:   
  
00408140   0F0B             UD2      ；第 1 次   
00408142   EB 02            JMP SHORT Obsidium.00408146   
……   
004085BC   F7F0             DIV EAX    ；第 2 次，这才是最后一次   
004085BE   EB 04            JMP SHORT Obsidium.004085C4   
  
现在再按 shift+f9 程序就真的运行了.   
好了，关闭程序再设置并重来，为什么不一开始就取消全部异常呢？因为那样可以少记异常次数呀(J 又在“懒”)了。   
  
在最后一次异常 005085BC 后在异常句柄处下断:   
  
0012FF98   0012FFE0  Pointer to next SEH record   
0012FF9C   004085ED  SE handler      ；下 MR 4085ED，记住了是内存断，如果不是的话，有时后面的操作对它无效。   
  
下内存断后，SHIFT+F9 会中断在 005085ED 处   
  
004085ED   EB 04            JMP SHORT Obsidium.004085F3    ；断在这里   
004085EF   6965 26 F7C80000 IMUL ESP,DWORD PTR SS:\[EBP+26\],0C8F7   
  
中断后，下 Command is one of  “POPFD”àCTRL+F11 这样不用多久就到了这里   
  
00875C2A   9D               POPFD      ；断在 POPFD 这里   
00875C2B   EB 02            JMP SHORT 00875C2F   
00875C2D   FFA6 EB044B24    JMP DWORD PTR DS:\[ESI+244B04EB\]   
00875C33   26:61            POPAD       ；不能下 POPAD 哦，这里是骗人的   
00875C35   68 F8134000      PUSH 4013F8    ；这里就是壳所抽的代码。   
  
再次就 F8 就到了程序的“地盘”：   
  
00401165   E8 F0FFFFFF      CALL Obsidium.0040115A    ；取消分析就会看到这个样子的   
0040116A   0000             ADD BYTE PTR DS:\[EAX\],AL   
  
VC、DELPHI 等其它类型的程序也可以用 POPFD 的方法找到壳所抽的代码。当然你想用记住的各种程序的入口去改的话，你就不用这么弄了，可以简单一点，在第九次异常后，下内存页断就直接会到入口，再补上你自己记的代码，不过我觉得自己找到代码比较好。如果是 VB 程序的话也可以偷懒的,下内存段断点后，到了程序的“地盘”。那个 PUSH 值可以在这里找到   
  
0012FFC0   004013F8  Obsidium.004013F8    ；这就是 PUSH 的值   
0012FFC4   77E77D08  RETURN to KERNEL32.77E77D08   
  
找到入口（入口为 1160）和抽的代码后，补上并 DUMP 之，程序是 DUMP 下来了，但能用吗？用 ImportREC 找一下，晕，没有一个有效的，所以也就有了我们的任务二了。   
任务二：跳过加密，得到有效的输入表。   
跳过加密找到有效的输入表，其实全靠辉仔 YOCK 指点，才让我搞定。非常感谢辉仔 YOCK。   
我的找输入表的方法和辉仔 YOCK 的有点不同，他是根据 IAT 来下断，我的方法如下:   
在第七次异常后:下断 BP LoadLibraryExA 再 SHIFT+F9 这样，就会在系统 API 入口处中断.   
  
77E6A234   FF7424 04        PUSH DWORD PTR SS:\[ESP+4\]  ；系统中断   
77E6A238   E8 0FCDFFFF      CALL KERNEL32.77E66F4C   
  
ALT+F9 返回到用户代码:   
  
0087430F   EB 04            JMP SHORT 00874315      ；返回到这里   
00874311   84AE 760FEB03    TEST BYTE PTR DS:\[ESI+3EB0F76\],CH   
  
到这里后慢慢按 F7 不久就到这里（也就是辉仔 YOCK 所说的地方）：   
  
00874564   57               PUSH EDI   
00874565   8B5D 14          MOV EBX,DWORD PTR SS:\[EBP+14\]      
00874568   8B75 10          MOV ESI,DWORD PTR SS:\[EBP+10\]   
0087456B   8B7D 0C          MOV EDI,DWORD PTR SS:\[EBP+C\]   
0087456E   8B5B 04          MOV EBX,DWORD PTR DS:\[EBX+4\]   
00874571   66:F706 2000     TEST WORD PTR DS:\[ESI\],20   
  
;这里改成 TEST WORD PTR DS：\[ESI\]，8   
  
00874576   74 46            JE SHORT 008745BE   
  
;这里改成 JNE SHORT 008745BE   
  
00874578   66:F706 0200     TEST WORD PTR DS:\[ESI\],2   
0087457D   75 1F            JNZ SHORT 0087459E   
0087457F   66:C706 0400     MOV WORD PTR DS:\[ESI\],4   
00874584   8B45 14          MOV EAX,DWORD PTR SS:\[EBP+14\]   
00874587   6A 01            PUSH 1   
00874589   6A 00            PUSH 0   
0087458B   FF76 04          PUSH DWORD PTR DS:\[ESI+4\]   
0087458E   6A 00            PUSH 0   
00874590   FF75 18          PUSH DWORD PTR SS:\[EBP+18\]   
00874593   FF50 40          CALL DWORD PTR DS:\[EAX+40\]   
00874596   85C0             TEST EAX,EAX   
00874598   74 39            JE SHORT 008745D3   
  
;这里改成 JE SHORT 008745BE   
  
0087459A   8907             MOV DWORD PTR DS:\[EDI\],EAX   
0087459C   EB 20            JMP SHORT 008745BE   
0087459E   66:C706 0400     MOV WORD PTR DS:\[ESI\],4   
008745A3   8B45 14          MOV EAX,DWORD PTR SS:\[EBP+14\]   
008745A6   0FB756 02        MOVZX EDX,WORD PTR DS:\[ESI+2\]   
008745AA   6A 01            PUSH 1   
008745AC   52               PUSH EDX   
008745AD   6A 00            PUSH 0   
008745AF   FF76 04          PUSH DWORD PTR DS:\[ESI+4\]   
008745B2   FF75 18          PUSH DWORD PTR SS:\[EBP+18\]   
008745B5   FF50 40          CALL DWORD PTR DS:\[EAX+40\]   
008745B8   85C0             TEST EAX,EAX   
008745BA   74 17            JE SHORT 008745D3   
  
;这里改成 JE SHORT 008745BE   
  
008745BC   8907             MOV DWORD PTR DS:\[EDI\],EAX   
008745BE   83C6 08          ADD ESI,8   
008745C1   83C7 04          ADD EDI,4   
008745C4   FF4D 08          DEC DWORD PTR SS:\[EBP+8\]   
008745C7  ⁷⁵ A8            JNZ SHORT 00874571   
008745C9   33C0             XOR EAX,EAX   
008745CB   40               INC EAX   
  
改完之后不用再像辉仔 YOCK 那篇那样继续改下去。   
改完后 F9 运行一次，会到第八次异常处，这时直接用 ImportREC 写入 OEP（1160）修复，会有一个无效的，CUT 之。再 FIXDUMP 就 OK 了。再运行就没有问题了。   
总结：   
这种修复法只能用于 VB 类的壳，因为如果是其它的，会改段地址我还没找到方法修复。诚请高手们指点一下。找跳过输入表方面也是别人的功劳，自我感觉这次脱壳没有多少是自己的。   
  
  
Thanks:   
  
fly、辉仔 yock、二哥、jinggulong、所有曾经关心支持及帮助过我的朋友和您!   
  
BTW：PELOCK1.06 破解版加的壳比演示版的好玩多了，这样下去，脱壳真的会走到绝路了。大家看看这样排版好不好看   
  
bmd2chen@tom.net  
 4dd466d0e1e0cc123afbe504da2462ca