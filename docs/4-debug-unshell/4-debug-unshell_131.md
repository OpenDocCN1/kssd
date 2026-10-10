

# ACProtect V1.1

> 原文：[`www.pediy.com/kssd/pediy06/pediy6411.htm`](https://www.pediy.com/kssd/pediy06/pediy6411.htm)

不用脱衣也强暴 ACProtect   
  
不用脱衣也强暴   
【目    标】:ACProtect V1.1 主程序   
【工    具】:Olydbg1.1(diy 版)、LORDPE、ImportREC1.6F   
【任    务】:不脱壳给它做一个内存修改补丁    
【操作平台】:WINDOWS 2000 ADV   
【作    者】:loveboom\[DFCG\]   
【相关链接】: www.anticrack.cn   
【简要说明】：上次脱了一下 acp1.09 的主程序（伪脱）,没想到不能跨平台，最近看到有人把那个壳加的段也不删除，让壳里的东西也放进去，所以我就想干脆不脱它的衣服来强暴它(别以为你穿了马甲，就不可以“强奸”!)。   
  
【详细过程】:   
运行 ACPROTECT 先,现在打开改装后的 ollydbg，File->Attach 一下 acp 的进程.   
77F9F9E0   C3               RETN        //这里停住了，F9 运行    
77F9F9E1 > CC               INT3   
F9 运行后，点右键 view->module /Acprotec/ 两次   
77DF3892   C3               RETN  //第一次    
77DF3893 > 55               PUSH EBP   
……   
00401000   04 10            ADD AL,10        //第二次    
00401002   40               INC EAX   
第二次后输入 AC 或^A 分析代码,完成之后就清楚很多了，再右键->Search for(查找)->All referenced text strings(查找所有字符串).这样之后会出现参考窗:   
任务一：去除打开后的未注册标题.   
右键查找字符串”Unreg”   
Text strings referenced in ACProtec:CODE, item 6765   
Address=004C628B   
Disassembly=MOV EDX,ACProtec.004C978C   
Text string=ASCII "                         Unregistered Limited Version "   
就在这一栏,双击后，到 cpu 窗口中:   
004C6282   . 83BD 64FFFFFF >CMP DWORD PTR SS:\[EBP\-9C\],0   
004C6289   . 75 18          JNZ SHORT ACProtec.004C62A3   
004C628B   . BA 8C974C00    MOV EDX,ACProtec.004C978C                ;  ASCII "                         Unregistered Limited Version "   
004C6290   . 8B45 FC        MOV EAX,DWORD PTR SS:\[EBP\-4\]   
将 004C6289 改成 EB 18(也就是 jmp 004C62A3).   
  
任务二：去除加壳后的程序提示未注册对话框。   
先加一个程序并运行它，看看它的未注册信息，再在字符串参考窗口中找 by 这样不久找到这里：   
Text strings referenced in ACProtec:CODE, item 6481   
Address=004B272B   
Disassembly=MOV ECX,ACProtec.004B2E94   
Text string=ASCII "by unregistered ACProtect"   
再次双击来到这里：   
004B270D  |. 8B45 A0        MOV EAX,DWORD PTR SS:\[EBP\-60\]   
004B2710  |. BA 782E4B00    MOV EDX,ACProtec.004B2E78   
004B2715  |. E8 BA27F5FF    CALL ACProtec.00404ED4   
004B271A  |. 0F85 AA000000  JNZ ACProtec.004B27CA        //NOP    
004B2720  |> 6A 01          PUSH 1   
004B2722  |. 6A 00          PUSH 0   
004B2724  |. A1 74035100    MOV EAX,DWORD PTR DS:\[510374\]   
004B2729  |. 8B00           MOV EAX,DWORD PTR DS:\[EAX\]   
004B272B  |. B9 942E4B00    MOV ECX,ACProtec.004B2E94                ;  ASCII "by unregistered ACProtect"   
004B2730  |. BA FFE30100    MOV EDX,1E3FF   
004B2735  |. E8 3606FCFF    CALL ACProtec.00472D70   
004B273A  |. 8945 F8        MOV DWORD PTR SS:\[EBP\-8\],EAX   
把 004B271A 处 NOP 掉，这样加壳后的程序就不会有未注册提示。   
  
任务三：更改注册信息和段名，也就是把它的 Register to：改成你的大名：   
更改注册信息：   
在 Dump 栏找 Register,找到这里：   
004C97E6  52 65 67 69 73 74 65 72 20 74 6F 3A              Register to:   
这里改成你的喜欢写上的东西,我改成这样：   
004C97E6  43 72 61 63 6B 69 6E 67 44 46 43 47              CrackingDFCG   
改段名：   
004CB2E0  2E 70 65 72 70 6C 65 78                          .perplex   
004CB2E0  2E 5B 44 46 43 47 5D 20                            .\[DFCG\]   
得到我们要改的东西和改后的东西后再用 keymaker 做一个内存补丁就简单的强暴 ACPR 的主程序了(什么还不会,晕)，现在应该不用考虑跨平台的问题了吧.更多的看官们自己去找吧，我懒了.要睡了。附上我做的内存补丁.希望大家能喜欢。   
  
献给我亲爱的 DFCG，愿它能够蒸蒸日上!   
Thanks:   
Fly 辉仔 yock、jingulong、二哥、所有曾经关心支持或帮助过我的朋友！谢谢您们!   
  
Btw:最近心总是烦燥不安，这是我第一次用 vmware,也许是最后一次，第一次在 DFCG 发破文，也许是很久以后回来发，也许永远不会回来，也许最后一次上网吧，最后一次在网吧发破文，也许以后解密论坛不再有 loveboom 这个人，也许有;也许从此在地球上消失。   
也许上天让我在这也世上本来就是多余的.愿大家一路走好!   
                    by  loveboom\[DFCG\]                              
                    Email:bmd2chen@tom.com                             
2004-3-14  
 4dd466d0e1e0cc123afbe504da2462ca