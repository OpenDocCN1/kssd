
```
00375FC8 |90 nop ;是为了说明一下,这里开始 ANTI 的手段不断
00375FC9 |90 nop ;加强,而且中间不断的夹杂大量有用的指令
00375FCA \90 nop ;高手出手常常是见刀不见人,不留神就中了
00375FCB 90 nop ;毒,不禁想起了一个武侠中的门派----唐门
00375FCC 8D85 E1484090 lea eax,dword ptr ss:[ebp+904048E1] ;唐门暗器独步天下,常常杀人以无形,而现在
00375FD2 EB 02 jmp short 00375FD6 ;开始,我将一一展现这些,孙子云,兵者,诡道也!
00375FD4 90 nop ;其实,作者有的时候,不应该在程序上有什么
00375FD5 90 nop ;重点防护区,应该全程序平衡,否则,你下手越
00375FD6 50 push eax ;重,说明这个区域有重点的东西要出来,肯定会
00375FD7 E8 04000000 call 00375FE0 ;引起人家的注意,一旦小心,那么你的设计就
00375FDC 90 nop ;成了此地无银了.
00375FDD 90 nop
00375FDE 90 nop ;话说多了,375fcc 处取返回地址,并且下面将返回
00375FDF 90 nop ;地址入栈 , 返回地址是:375ff6
00375FE0 83C4 04 add esp,4
00375FE3 8B85 0A114100 mov eax,dword ptr ss:[ebp+41110A] ; GetCurrentThread 地址送 EAX,为 call 传送参数
00375FE9 EB 04 jmp short 00375FEF ；获取当前正在运行的线程，结果放入 EAX
00375FEB 90 nop
00375FEC 90 nop
00375FED 90 nop
00375FEE 90 nop
00375FEF E9 77BF0000 jmp 00381F6B ;去 PUSH API RET 是 call GetCurrentThread 了 ;后面我将不再展开这类 call 模式
```

..........................下面是设线程的

```
00375FF6 6A 02 push 2 ;2 设定线程为极高级别
00375FF8 50 push eax ;GetCurrentThread 返回码 fffffe 入栈，准备下? ；函数来调用。
00376002 8D85 17494000 lea eax,dword ptr ss:[ebp+404917] ;又取 call 返回地址:37602c
0037600D E8 04000000 call 00376016
00376016 83C4 04 add esp,4
00376019 8B85 BA114100 mov eax,dword ptr ss:[ebp+4111BA] ; 取 SetThreadPriority 地址
 00376025 /E9 41BF0000 jmp 00381F6B ;call
```

..........................ANTI-DEBUG

```
0037602C 50 push eax ; ＝1
0037602D 52 push edx ; ＝7c92eb94
0037602E 51 push ecx ; ＝12ff7c 以上三个都入栈
00376032 0F31 rdtsc ; 取时间基准值
00376036 E8 03000000 call 0037603E ;
0037603E 83C4 04 add esp,4 ;
00376041 E8 38000000 call 0037607E ;
00376081 68 C2100000 push 10C2 ;10C2 入栈
0037608C 68 24080E68 push 680E0824 ;代码入栈
00376091 68 90908344 push 44839090 ;代码入栈
00376096 - FFE4 jmp near esp ;去堆栈执行,上趟女厕所
0012FF84 90 nop
0012FF85 90 nop
0012FF86 834424 08 0E add dword ptr ss:[esp+8],0E ; 376099
0012FF8B 68 99603700 push 376099 ; 376099
0012FF90 C2 1000 retn 10
00376098 E8 C3E80300 call 003B4960
0037604F E8 46000000 call 0037609A
0037609A E8 03000000 call 003760A2 ;一路花指令,这个花指令你要是打掉,就 ;破坏了整个代码的观看的完整性
003760A2 58 pop eax ; 0012FF7C
003760A6 83C0 07 add eax,7
003760A9 50 push eax
003760AA C3 retn ;花指令序列,将 call 堆栈平衡,37609a 的 call 平衡
003760AD C3 retn ;37604f call 平衡
00376060 E8 02000000 call 00376067 ;看你有多少耐心
00376067 0F31 rdtsc ;再次取时间戳,
00376069 83C4 04 add esp,4 ;
0037606C 2BC1 sub eax,ecx ;和基准时间戳计算,看过去多少时间
0037606E 3D 00000200 cmp eax,20000 ;对比计算结果,大于 20000,对不起,有人尾随我上女厕所
00376073 76 04 jbe short 00376079 ;来人啦,抓流氓哟,这里必须改造成 jmp,安全了, ;给小娘子带上口罩
00376075 83C4 0C add esp,0C ;
00376078 C3 retn ;
00376079 59 pop ecx ; 将 SetThreadPriority 的结果出堆栈
0037607A 5A pop edx ;
0037607B 58 pop eax ;
```

.。。。。。。。。。。。。。。。。

```
003760AE 8B85 180F4100 mov eax,dword ptr ss:[ebp+410F18] ; 将 00400000 取出来
003760B4 0340 3C add eax,dword ptr ds:[eax+3C] ；＝4000e8
003760B7 68 00FE98C7 push C798FE00 ；
003760BC 50 push eax ; 这里入栈估计有明堂
003760BD E8 5D000000 call 0037611F ？
0037612A FF30 push dword ptr ds:[eax] ;又去堆栈了
0037612C E8 C9000000 call 003761FA ；去堆栈执行代码
00376206 8B40 50 mov eax,dword ptr ds:[eax+50] ;7ccfa
00376209 3385 2C0F4100 xor eax,dword ptr ss:[ebp+410F2C] ;eax=35ff6c
00376215 E8 5D000000 call 00376277 ;去堆栈执行代码
0037635E 8B8D 080F4100 mov ecx,dword ptr ss:[ebp+410F08] ;ecx=35ff6cc
00376364 3BC1 cmp eax,ecx ;
00376366 8DB5 2F144100 lea esi,dword ptr ss:[ebp+41142F] ;
0037636C 46 inc esi ; =382b45
0037636D 68 00FE9A28 push 289AFE00 ;
00376372 50 push eax
00376373 E8 5D000000 call 003763D5 ;??????去堆栈
```

●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●DEBUG 文件检查●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●

```
003764BC 6A 09 push 9
003764BE 59 pop ecx ; cx=9,这里一共要判断 9 个 DEBUG 程序
003764C6 AC lods byte ptr ds:[esi] ; 将 382b45 处的字符送 AL
003764C9 F6D0 not al ; not al 解码
003764CB 8846 FF mov byte ptr ds:[esi-1],al ; 解码后放回
```
