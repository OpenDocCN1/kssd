

# OLLYDBG 快速脱 tElock v0.98

> 原文：[`www.pediy.com/kssd/pediy05/pediy50608.htm`](https://www.pediy.com/kssd/pediy05/pediy50608.htm)

现在的 ASPR 都有脱壳机了，tElock 还没有，还要手脱，真是累，一直想找一个较快的方法，呵   
发现用 OLLYDBG 脱速度很快，而且还是傻瓜式的：－）   
目标：\*\*投注大师 2003 黄金版   
工具：ollydbg1.09,winhex,loadpe,peditor.   
一、找入口点   
用 OLLYDBG 找 tElock 的入口点完全不用动脑，跟我来：   
1。用 OD 载入目标程序，先在 OD 调试设置中把异常卡中的“INT3 中断”和“单步中断”打钩。.   
2 运行程序，出现异常用 SHIFT＋F9 过，记住数（下回有用），几后程序运行，这我们看 OD 的窗口   
004766F1    0000            ADD BYTE PTR DS:\[EAX\],AL   
004766F3    0000            ADD BYTE PTR DS:\[EAX\],AL   
004766F5    0000            ADD BYTE PTR DS:\[EAX\],AL   
004766F7    0000            ADD BYTE PTR DS:\[EAX\],AL   
004766F9    0000            ADD BYTE PTR DS:\[EAX\],AL   
004766FB    0000            ADD BYTE PTR DS:\[EAX\],AL   
004766FD    0000            ADD BYTE PTR DS:\[EAX\],AL   
004766FF    0000            ADD BYTE PTR DS:\[EAX\],AL   
00476701    0000            ADD BYTE PTR DS:\[EAX\],AL   
00476703    0000            ADD BYTE PTR DS:\[EAX\],AL   
00476705    0000            ADD BYTE PTR DS:\[EAX\],AL   
都是这样的代码，但我们滚动条一直向上拉，到最上面，我们看到了如下码   
00475000  - E9 DE61FBFF    JMP 智能投注.0042B1E3   
00475005    0000            ADD BYTE PTR DS:\[EAX\],AL   
00475007    0000            ADD BYTE PTR DS:\[EAX\],AL   
00475009    0000            ADD BYTE PTR DS:\[EAX\],AL   
0047500B    0000            ADD BYTE PTR DS:\[EAX\],AL   
0047500D    0000            ADD BYTE PTR DS:\[EAX\],AL   
0047500F    0000            ADD BYTE PTR DS:\[EAX\],AL   
哈哈，这唯一特别的一行看到了吧，0042b1e3,这就是程序的 oep 了。要记做你按了多少次   
SHIFT＋F9，我这是 6 次，   
2、DUMP 出 IT 和程序   
重新开始程序，运行，然后再按 6-2=4 次 SHIFT+f9,这时 shift+f 搜索命令：AND  DWORD PTR \[ESI+0C\],00   
只有一处：   
00476210    8B95 62D34000  MOV EDX,DWORD PTR SS:\[EBP+40D362\]   
00476216    8BB5 52D34000  MOV ESI,DWORD PTR SS:\[EBP+40D352\]   
0047621C    85F6            TEST ESI,ESI　　　　　　　　　　　　　　，在这里下断   
0047621E    0F84 06040000  JE 智能投注.0047662A   
00476224    03F2            ADD ESI,EDX   
00476226    83A5 52D44000 0>AND DWORD PTR SS:\[EBP+40D452\],0　0047622D    8B46 0C        MOV EAX,DWORD PTR DS:\[ESI+C\]   
00476230    8366 0C 00      AND DWORD PTR DS:\[ESI+C\],0   
        ,这是搜索到的   
00476234    85C0            TEST EAX,EAX   
00476236    0F84 EE030000  JE 智能投注.0047662A   
0047623C    03C2            ADD EAX,EDX   
0047623E    8BD8            MOV EBX,EAX   
00476240    50              PUSH EAX   
00476241    FF95 D0D24000  CALL DWORD PTR SS:\[EBP+40D2D0\]   
我们在 47621c 下断，按 shift+f9 断了下来，看看 esi 的值：00059cc0,是什么，呵，这就是   
it 的位置了，然后 d 459cc0,我们看看左下角，IT 就躺在这里，一直向下拉，看看有多大，   
45B9E8-459CC0=1D28,这是时你可以打开 loadpe,部分脱壳 JJ.DMP，位置：00459cc0.大小:1d28,   
回到 OD，清断点，按 1 次 shift+f9, 然后下断:bp 0042B1E3(可别说你忘了这是什么），再   
按 SHIFT＋f9，来到 oep 了，再用 loadpe 完全脱壳 DUMPED.EXE。呵，快完了。   
3、扫尾工作   
用 winhex 把 jj.dmp 的代码粘到 dumped.exe 的相应位置保存，再用 peditor 打开 dumped.exe,   
修改入口点为 2B1E3(0042B1E3-00400000),引入表地址为：59CC0，最后按 rebuider,呵呵，   
大功告成。看看时间，2 分钟不到。   
测试了几个 tElock 的壳，都可以用这种方法脱之。  4dd466d0e1e0cc123afbe504da2462ca