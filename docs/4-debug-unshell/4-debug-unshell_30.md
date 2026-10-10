

# 广义 ESP 定律

> 原文：[`www.pediy.com/kssd/pediy06/pediy6083.htm`](https://www.pediy.com/kssd/pediy06/pediy6083.htm)

寻找真正的入口（OEP）--广义 ESP 定律  
  
作者：Lenus  
FROM: poptown.gamewan.com/bbs  
E-MAIL：Lenus\_M@163.com  
  
1.前言  
    
  在论坛上看到很多朋友，不知道什么是 ESP 定律，ESP 的适用范围是什么，ESP 定律的原理是什么，如何使用 ESP 定律？看到了我在  
  
http://poptown.gamewan.com/dispbbs.asp?boardID=5&ID=54&page=1  
  
调查结果发现,大家对 ESP 定律很感兴趣，当然因为实在是太好用了，现在我就来告诉大家什么是 ESP 定律，它的原理是什么！！  
  
BTW：在看完了手动脱壳入门十八篇了以后，再看这篇文章也许会对你更有帮助！  
  
在下面地址下载：  
  
http://soft.winzheng.com/SoftView/SoftView\_23125.htm  
  
2.准备知识  
  
  在我们开始讨论 ESP 定律之前，我先给你讲解一下一些简单的汇编知识。  
  1.call  
  这个命令是访问子程序的一个汇编基本指令。也许你说，这个我早就知道了！别急请继续看完。  
  call 真正的意义是什么呢？我们可以这样来理解：1.向堆栈中压入下一行程序的地址；2.JMP 到 call 的子程序地址处。例如：  
  
00401029    .  E8 DA240A00    call 004A3508  
0040102E    .  5A             pop edx  
在执行了 00401029 以后，程序会将 0040102E 压入堆栈，然后 JMP 到 004A3508 地址处！  
     
  2.RET  
  与 call 对应的就是 RET 了。对于 RET 我们可以这样来理解：1.将当前的 ESP 中指向的地址出栈；2.JMP 到这个地址。  
    
  这个就完成了一次调用子程序的过程。在这里关键的地方是：如果我们要返回父程序，则当我们在堆栈中进行堆栈的操作的时候，一定要保证在 RET 这条指令之前，ESP 指向的是我们压入栈中的地址。这也就是著名的“堆栈平衡”原理！  
  
3.狭义 ESP 定律  
    
  ESP 定律的原理就是“堆栈平衡”原理。  
    
  让我们来到程序的入口处看看吧！  
    
  1.这个是加了 UPX 壳的入口时各个寄存器的值！  
EAX 00000000  
ECX 0012FFB0  
EDX 7FFE0304  
EBX 7FFDF000  
ESP 0012FFC4  
EBP 0012FFF0  
ESI 77F51778 ntdll.77F51778  
EDI 77F517E6 ntdll.77F517E6  
EIP 0040EC90 note-upx.<ModuleEntryPoint>  
C 0  ES 0023 32bit 0(FFFFFFFF)  
P 1  CS 001B 32bit 0(FFFFFFFF)  
A 0  SS 0023 32bit 0(FFFFFFFF)  
Z 0  DS 0023 32bit 0(FFFFFFFF)  
S 1  FS 0038 32bit 7FFDE000(FFF)  
T 0  GS 0000 NULL  
D 0  
O 0  LastErr ERROR\_MOD\_NOT\_FOUND (0000007E)  
  
  2.这个是 UPX 壳 JMP 到 OEP 后的寄存器的值！  
EAX 00000000  
ECX 0012FFB0  
EDX 7FFE0304  
EBX 7FFDF000  
ESP 0012FFC4  
EBP 0012FFF0  
ESI 77F51778 ntdll.77F51778  
EDI 77F517E6 ntdll.77F517E6  
EIP 004010CC note-upx.004010CC  
C 0  ES 0023 32bit 0(FFFFFFFF)  
P 1  CS 001B 32bit 0(FFFFFFFF)  
A 0  SS 0023 32bit 0(FFFFFFFF)  
Z 1  DS 0023 32bit 0(FFFFFFFF)  
S 0  FS 0038 32bit 7FFDE000(FFF)  
T 0  GS 0000 NULL  
D 0  
O 0  LastErr ERROR\_MOD\_NOT\_FOUND (0000007E)  
  
呵呵~是不是除了 EIP 不同以外，其他都一模一样啊！  
  
为什么会这样呢？  
我们来看看 UPX 的壳的第一行：  
  
0040EC90 n>  60               pushad      //\*\*\*\*注意这里\*\*\*\*\*  
0040EC91     BE 15B04000      mov esi,note-upx.0040B015  
    
PUSHAD 就是把所有寄存器压栈！我们在到壳的最后看看：  
  
0040EE0F     61               popad      //\*\*\*\*注意这里\*\*\*\*\*  
0040EE10   - E9 B722FFFF      jmp note-upx.004010CC   //JMP 到 OEP  
  
POP 就是将所有寄存器出栈！  
  
  
而当我们 PUSHAD 的时候，ESP 将寄存器压入了 0012FFC0--0012FFA4 的堆栈中！如下：  
  
0012FFA4   77F517E6  返回到 ntdll.77F517E6 来自 ntdll.77F78C4E           //EDI   
0012FFA8   77F51778  返回到 ntdll.77F51778 来自 ntdll.77F517B5          //ESI  
0012FFAC   0012FFF0                                                    //EBP  
0012FFB0   0012FFC4                                                   //ESP  
0012FFB4   7FFDF000                                                  //EBX  
0012FFB8   7FFE0304                                                 //EDX  
0012FFBC   0012FFB0                                                //ECX  
0012FFC0   00000000                                               //EAX  
  
所以这个时候，在教程上面就告诉我们对 ESP 的 0012FFA4 下硬件访问断点。也就是说当程序要访问这些堆栈，从而恢复原来寄存器的值，准备跳向苦苦寻觅的 OEP 的时候，OD 帮助我们中断下来。  
  
于是我们停在 0040EE10 这一行！  
    
总结：我们可以把壳假设为一个子程序，当壳把代码解压前和解压后，他必须要做的是遵循堆栈平衡的原理，让 ESP 执行到 OEP 的时候，使 ESP=0012FFC4。  
  
4.广义 ESP 定律  
  
  很多人看完了教程就会问：ESP 定律是不是就是 0012FFA4，ESP 定律的适用范围是不是只能是压缩壳！  
    
  我的回答是：NO！  
  
  看完了上面你就知道你如果用 0012FFA8 也是可以的，ESP 定律不仅用于压缩壳他也可以用于加密壳！！！  
  
  首先，告诉你一条经验也是事实---当 PE 文件运行开始的时候，也就是进入壳的第一行代码的时候。寄存器的值总是上面的那些值，不信你自己去试试！而当到达 OEP 后，绝大多的程序都第一句都是压栈！（除了 BC 编写的程序，BC 一般是在下面几句压栈）  
  
  现在，根据上面的 ESP 原理，我们知道多数壳在运行到 OEP 的时候 ESP=0012FFC4。这就是说程序的第一句是对 0012FFC0 进行写入操作！  
    
  最后我们得到了广义的 ESP 定律，对只要在 0012FFC0 下，硬件写入断点，我们就能停在 OEP 的第二句处！！  
  
下面我们来举个例子,就脱壳进阶第一篇吧！  
  
  载入 OD 后，来到这里：  
  
0040D042 N>  B8 00D04000      mov eax,Notepad.0040D000 //停在这里  
0040D047     68 4C584000      push Notepad.0040584C  
0040D04C     64:FF35 00000000 push dword ptr fs:\[0\]    //第一次硬件中断，F9  
0040D053     64:8925 00000000 mov dword ptr fs:\[0\],esp  
0040D05A     66:9C            pushfw  
0040D05C     60               pushad  
0040D05D     50               push eax  
  
直接对 0012FFC0 下硬件写入断点，F9 运行。（注意硬件中断）  
  
在 0040D04C 第一次硬件中断，F9 继续！  
  
0040D135     A4               movs byte ptr es:\[edi\],byte ptr ds:\[esi\] //访问异常，不管他 shift+F9 继续  
0040D136     33C9             xor ecx,ecx  
0040D138     83FB 00          cmp ebx,0  
0040D13B   ^ 7E A4            jle short Notepad.0040D0E1  
  
第二次硬件中断。  
  
004058B5       64             db 64                                 //断在这里  
004058B6       89             db 89  
004058B7       1D             db 1D  
004058B8       00             db 00  
004058B9       00             db 00  
  
这里也不是，F9 继续！  
  
004010CC   /.  55             push ebp  
004010CD   |.  8BEC           mov ebp,esp  //断在这里，哈哈，到了！（如果发现有花指令，用 ctrl+A 分析一下就能显示出来）  
004010CF   |.  83EC 44        sub esp,44  
004010D2   |.  56             push esi  
  
   快吧！还不过瘾，在来一个例子。  
  
   脱壳进阶第二篇  
  
   如果按上面的方法断不下来，程序直接运行了！没什么，我们在用另一种方法！  
    
   载入后停在这里，用插件把 OD 隐藏！  
  
0040DBD6 N>^\\E9 25E4FFFF      jmp Note\_tEl.0040C000                  //停在这里  
0040DBDB     0000             add byte ptr ds:\[eax\],al  
0040DBDD     0038             add byte ptr ds:\[eax\],bh  
0040DBDF     A4               movs byte ptr es:\[edi\],byte ptr ds:\[esi\]  
0040DBE0     54               push esp  
  
   F9 运行，然后用 SHIFT+F9 跳过异常来到这里：  
  
0040D817   ^\\73 DC            jnb short Note\_tEl.0040D7F5       //到这里  
0040D819     CD20 64678F06    vxdcall 68F6764  
0040D81F     0000             add byte ptr ds:\[eax\],al  
0040D821     58               pop eax  
  
   在这里对 0012FFC0 下硬件写入断点！（命令行里键入 HW 12FFC0）SHIFT+F9 跳过异常，就来到 OEP 的第二行处：（用 CTRL+A 分析一下）  
  
004010CC   /.  55             push ebp  
004010CD   |.  8BEC           mov ebp,esp                       //断在这里  
004010CF   |.  83EC 44        sub esp,44  
004010D2   |.  56             push esi  
004010D3   |.  FF15 E4634000  call dword ptr ds:\[4063E4\]  
004010D9   |.  8BF0           mov esi,eax  
004010DB   |.  8A00           mov al,byte ptr ds:\[eax\]  
004010DD   |.  3C 22          cmp al,22  
  
   就这样我们轻松搞定了两个加密壳的找 OEP 问题！  
  
5.总结  
  
  现在我们可以轻松的回答一些问题了。  
    
  1.ESP 定律的原理是什么？  
  
  堆栈平衡原理。  
    
  2.ESP 定律的适用范围是什么？  
  
  几乎全部的压缩壳，部分加密壳。只要是在 JMP 到 OEP 后，ESP=0012FFC4 的壳，理论上我们都可以使用。但是在何时下断点避开校验，何时下断 OD 才能断下来，这还需要多多总结和多多积累。欢迎你将你的经验和我们分享。  
  
  3.是不是只能下断 12FFA4 的访问断点？  
  
  当然不是，那只是 ESP 定律的一个体现，我们运用的是 ESP 定律的原理，而不应该是他的具体数值，不能说 12FFA4，或者 12FFC0 就是 ESP 定律，他们只是 ESP 定律的一个应用罢了！  
  
  4.对于 STOLEN CODE 我们怎么办？  
  
  哈哈，这正是寻找 STOLEN CODE 最好的办法！当我们断下时，正好断在了壳处理 STOLEN CODE 的地方，在 F8 一会就到 OEP 了！  
  
6.后话  
    
    
  以上的方法原理都是我自己总结，自己的经验，如果有什么不对的地方，有什么没解释清楚的地方。还请海涵！但是如果觉得我很厉害，那就大可不必，因为 ESP 定律也是别人教我的，不是我第一个提出来的！我只是个比你们早飞一点的菜鸟罢了^-^  
  
  看了上面的文字希望能对你在寻找 OEP 的时候有帮助，但是别忘了一句话：菜鸟认为找 OEP 很难，高手认为修复才是最难！ 好了，下一篇应该写 IAT 的修复原理了！让我们共同努力吧！  
  
  最后如果转载注明作者并保持文章的完整, 谢谢你看完

 4dd466d0e1e0cc123afbe504da2462ca