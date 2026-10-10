

# Acprotect 1.21 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6431.htm`](https://www.pediy.com/kssd/pediy06/pediy6431.htm)

这几天比较忙，看雪这里也来得少了。看见偶成为“斑竹”，恐慌之余想想，唯有努力吧。   
看到本版关于 Acpr 的帖子不少，也把自己的一点心得拿出来献丑，也算祝贺“脱壳论坛”的成立。   
  
本文计划分四部分：一. dump、二. IAT 修复（不添加段）、三. 程序修复、四. 跨系统问题，由于时间的关系，打算一段一段的贴出。   
  
（系统 WIN2K）   
  
一、dump   
   1.躲过 acpr 的检查   
     用 od(1.1b)载入程序，Command 处输入:d fs:\[30\]+2，回车后将 Hex dump 窗口中左上角的 1 改成 0（这就相当于插件 IsDebugPresent 选   
  
hide 啦）；在 Command 处又输入 bp Process32First+1，回车。F9 执行程序，断点处中断后去掉断点，Alt+F9 返回程序“领空”，F8 两下后来到   
006E1A4C   3B95 B3D44000    CMP EDX,DWORD PTR SS:\[EBP+40D4B3\]   
这里\[EBP+40D4B3\]处存放的是程序的 ProcessId，我们把它改成其父进程（OllDbg）的 ProcessId，（用工具 pupe）找到 OllDbg 的 Id 后替换该处   
  
数据。   
（当然，如果使用 Fly 的修改版这一步就省了，不过知其所以不是更好吗！再加上 od 升级后 Fly 的新版还在加工中吧。）   
  
   2.到伪 oep   
     这步就更快了，按下 Alt+M，在地址 401000 (size 00CC000)处下 F2，F9 执行程序！   
     中断了，呵呵，在 406EDC 处！这里是非常熟悉的代码！！我们正处于 Delphi 程序的第一个 call 之中！！！   
00406EDC   PUSH EBX   
00406EDD   MOV EBX,EAX   
00406EDF   XOR EAX,EAX   
00406EE1   MOV DWORD PTR DS:\[4F370C\],EAX   
00406EE6   PUSH 0   
00406EE8   CALL ACProtec.00406E18   
........   .........   
   3.修复 oep   
     Acpr 1.21 主程序这次把几乎整个初始化主干代码都偷掉了，不过找回这部分代码也很容易。   
     停在 406edc 后看看 stack 窗口，返回到 6ed109 ？！鼠标在 stack 窗口里选中这行，回车，我们看见了经 Acpr 处理后“   
  
主干”代码：   
006ED0E3  PUSH ESI   
006ED0E4  MOV DWORD PTR SS:\[ESP\],ECX   
006ED0E7  PUSH EAX   
006ED0E8  MOV EAX,ACProtec.00406EDC   
006ED0ED  MOV ECX,EAX   
006ED0EF  POP EAX   
006ED0F0  PUSH ECX   
006ED0F1  POP DWORD PTR DS:\[6D4F0F\]                            ; ACProtec.00406EDC   
006ED0F7  POP DWORD PTR DS:\[6D4EFF\]   
006ED0FD  MOV ECX,DWORD PTR DS:\[6D4EFF\]   
006ED103  CALL DWORD PTR DS:\[6D4F0F\]                           ; ACProtec.00406EDC   
006ED109  MOV EAX,DWORD PTR DS:\[4F258C\]   
006ED10E  MOV EAX,DWORD PTR DS:\[EAX\]   
006ED110  MOV DWORD PTR DS:\[6D4EFB\],ACProtec.004622F8   
006ED11A  PUSH DWORD PTR DS:\[6D4EFB\]   
006ED120  POP DWORD PTR DS:\[6D4F0B\]   
006ED126  CALL DWORD PTR DS:\[6D4F0B\]   
006ED12C  MOV DWORD PTR DS:\[6D4EF7\],EDI   
........   ......   
006ED1AA  MOV DWORD PTR SS:\[ESP\],ACProtec.00462310   
006ED1B1  POP DWORD PTR DS:\[6D4F07\]   
006ED1B7  CALL DWORD PTR DS:\[6D4F07\]   
006ED1BD  PUSH DWORD PTR DS:\[4F228C\]                           ; ACProtec.004F45E8   
006ED1C3  POP DWORD PTR DS:\[6D4EE3\]   
006ED1C9  MOV ECX,DWORD PTR DS:\[6D4EE3\]   
006ED1CF  MOV EAX,DWORD PTR DS:\[4F258C\]   
006ED1D4  MOV EAX,DWORD PTR DS:\[EAX\]   
006ED1D6  PUSH DWORD PTR DS:\[47F7CC\]                           ; ACProtec.0047F818   
006ED1DC  POP EDX   
006ED1DD  PUSH ACProtec.00462310   
006ED1E2  POP DWORD PTR DS:\[6D4F03\]                            ; ACProtec.004CBFF8   
006ED1E8  CALL DWORD PTR DS:\[6D4F03\]                           ; ACProtec.004CBFF8   
006ED1EE  MOV EAX,DWORD PTR DS:\[4F258C\]   
006ED1F3  MOV EAX,DWORD PTR DS:\[EAX\]   
  
从这段代码容易分析出被“偷”的代码如下：   
  
PUSH EBP   
MOV EBP,ESP   
ADD ESP,-0C   
XOR EAX,EAX   
PUSH EAX     ;   这条语句可改成 push 0 等   
MOV \[EBP-4\],EAX   
MOV EAX,004CBFF8   
CALL 00406EDC   
MOV EAX,\[4F258C\]   
MOV EAX,\[EAX\]   
CALL 004622F8   
MOV ECX,\[4F2354\]   
MOV EAX,\[4F258C\]   
MOV EAX,\[EAX\]   
MOV EDX,\[47FB5C\]   
CALL 00462310   
MOV ECX,\[4F228C\]   
MOV EAX,\[4F258C\]   
MOV EAX,\[EAX\]   
MOV EDX,\[47F7CC\]   
CALL 00462310   
MOV EAX,\[4F258C\]   
MOV EAX,\[EAX\]   
  
在代码段尾部的空闲处键入上面的代码，再加上 Acpr 未偷的两条语句，Oep 就算修复了。这两条语句是：   
004CC263  CALL ACProtec.00462390   
004CC268  CALL ACProtec.004049D8   
  
现在可以 dump 了。   
  
    后面将看到，这段“标准”的 oep 代码还要根据我们的需要加以修改。   
  
（时间问题，今天就这一段吧）。

由于 Acpr 把 ImageSize 作为“敏感”数据，因此这一步就不能完全交给 ImportRec 了，当然，这么好的工具还是要用的。这里分为两步，第一是常规修复 IAT；第二就是移植 Import Table。   
   这其中第一步的确很简单，除了一个地方，Acpr 对原来的入口地址仅仅分解为两个值的异或运算结果。找 IAT 也很容易，因为是 Delphi 程序，所以 406EE8 这行就应该是调用 GetModuleHandle 函数了，鼠标选中该行后回车，见到：   
00406E18  JMP DWORD PTR DS:\[691234\]                ; ACProtec.006D42A7   
再回车，又见：   
  
006D42A7  PUSH 66D2D09B   
006D42AC  XOR DWORD PTR SS:\[ESP\],11358640   
006D42B3  RETN   
呵呵，异或后转到 api（这也算加密？有点晕）   
  
OK，根据这里的\[691234\]，知道这应该是 Thunk 值表的位置，将 Dump 窗口转到这里稍作调整看到这个表:   
  
00691140  10 40 6D 00 1D 40 6D 00 2A 40 6D 00 37 40 6D 00      始于 691140   
00691150  44 40 6D 00 51 40 6D 00 5E 40 6D 00 6B 40 6D 00   
00691160  78 40 6D 00 85 40 6D 00 92 40 6D 00 9F 40 6D 00   
........  ......   
00691800  28 26 79 71 5A 8A 79 71 B7 42 79 71 E9 15 78 71   
00691810  00 00 00 00 72 C3 5F 77 00 00 00 00 E6 A4 AF 76   
00691820  CD A5 AF 76 00 00 00 00 00 00 00 00 00 00 00 00      止于 691824   
  
利用异或还原数据吧，程序中空闲处键入：   
  
           pusha   
           mov esi,691140   
           mov edi,esi   
again：    lodsd                ;这里的行标在 od 中是地址，下面有行标处一样   
           test eax,eax   
           je OK   
           cmp eax,70000000   
           ja OK   
           mov edx,eax   
           mov eax,\[edx+1\]      ;\[edx+1\]指向 push 的数据   
           xor eax,\[edx+8\]      ;\[edx+8\]处是原来 xor 的数据   
           cmp eax,70000000     ;应该大于 70000000   
           jb check\_it   
OK:        stosd   
           cmp esi,691824   
           jbe again                
           jmp over   
check\_it:  nop                  ; <-- 这里设断，若停在这里，记下 edi 和 eax 的值。并在 eax 处设断。   
over    :  popa                 ; <-- 这里设断   
  
以上代码与 tDasm 的不同之处在于 1.还原的是 Thunk 表以保证所有的 api 无遗漏；2.有一个是 Acpr 借用了的 Api，必须找出来。   
好，执行这程序段吧。过程中两次停在 check\_it，其 edi 和 eax 的值分别是：   
           edi         eax   
        6911FC      6E084D   
        691588      6E084D   
记下这两组数据（注意 eax 都一样）修复程序时使用。   
下面轮到 ImportRec 上场了，启动它选中目标后 RVA 处设为 291140，Size 处设为 700，GetImports！   
乖乖，除了有两处外全部 OK！这两处就是刚才记下的啦，这个函数无论你用追踪层次几都无济于事。看看吧，都位于 user32 组中，特别是 291588 处附近的 Api 函数以“姓氏笔画为序”，很整齐的。这个未解析出来的函数排在 MessageBeep 的上方哦，是不是...它？一个商品化的程序不可能没有它！把结果存为文本文件放到记事本中搜索一下，果然没有它－－－MessageBoxA 啊！！！多半是了。也许你要说“又在猜了！”，好吧，刚才不是设断了吗，跟一下那个断点就清楚了，如果这样你会发现这个函数有四个参数（当然咯），跟到最后看到函数有两个出口 1.retn 2.jmp eax，这个时候 eax 的值正好指向 MessageBox，至此恢复完毕。（注意，这个就是 Acpr 借用的“Api”入口，运行时还要填入 6E084D 这个值的。）这时用 ImportRec 以通常方式修复 IAT（修复后的程序丧失加密功能），结果就是 dumped\_.exe 了。   
第二步，利用上面的 dumped\_.exe 移植 Impotr Table。这一步操作起来有点考耐心的。   
用 LordPe 查一下 Impotr Table 的 RVA---2F6000，记下后关闭 LordPe，od 载入 dumped\_.exe，把 Hex dump 窗口转到 6F6000，把 6F6000～6F6130 的数据复制到 691000～691130，再把 6F6140～6F7EF0 处的数据复制到 691840～6935F0，记下位置差 6F6140-691840=64900，用 Winhex 打开 dumped\_exe 把 291140～291830 的数据块复制下来，用 Winhex 的 RamEditor 把它（以 ASCII-Hex 方式）粘贴到 od 载入的 dumped\_.exe，位置：691140～691830。（希望不要说昏了）   
关闭 Winhex，在 od 中找“空地”键入：   
           pusha   
           mov esi,691000   
           mov edi,esi   
again：    lodsd             ;这里的行标在 od 中是地址，下面有行标处一样   
           cmp eax,2F0000   
           jb OK   
           sub eax,64900     ;64900 是位置差哦   
OK:        stosd   
           cmp esi,691824   
           jb again   
           popa     
over:      nop               ;<- 这里设断，执行这段代码   
  
呵呵，移植完毕。   
现在，你可以把内存中的 dumped\_.exe 再 dump 一次就啦，（dump 前最好清除自己的工作代码），然后用 LordPe 打开新 dump 的文件把 Import Table 的 RVA 改成 291000，Save 后检查 Import Table 的内容，会发现最后一个 api“看不见”，这是因为文件头中定义该段的 RSize 小了一点的缘故，用 LordPe 修改一下就好，RSize 和 VSize 都从 25D4 改成 2600 吧。再次 Save 后检查，allright！   
最后轮到去掉 ImportRec 添加的段了，这用 LordPe 就可简单搞定。用 OD 载入试试，是不是有点爽！   
（老板叫出差，看来第三篇要耽搁几天了。见谅） 

呵呵，又回家了。各位兄弟好！   
该说说这第三个问题了。前有 shinegood 的脱壳版，后有 tDasm 的修复文章，我这个不会又...？管他的，一则说过的话总要兑现，二则各有各的方法，更何况他们的不是都还有点问题吗。（二者使用了相同的方法来修复）   
修复中主要要解决的是 Replace Code 的地址，一般情况下，程序把 Replace Code 放到堆中执行，堆的结构由壳代码初   
  
始化时建立，这样，去了壳的程序就会出错。这里采用的方法就是修改放置 Replace Code 的地址。   
od 载入已修复 iat 的程序，隐藏 od、忽略所有异常、eip 定位到修复的 oep、在 6d540c 处设断，执行程序。断下光标移到 6b55b7，按 F4，看到：   
006D55B7  CALL Acprotect.006E1460   
006D55BC  MOV EAX,DWORD PTR SS:\[ESP+20\]                 ;获得返回地址   
006D55C0  XOR ECX,ECX   
006D55C2  MOV EBX,DWORD PTR SS:\[EBP+ECX\*4+4026E2\]       ;取表中元素   
006D55C9  ADD EBX,DWORD PTR SS:\[EBP+40D2AA\]             ;加 Image Base   
006D55CF  CMP EAX,EBX                                   ;   
006D55D1  JE SHORT Acprotect.006D55DA                   ;查表   
006D55D3  NOP   
006D55D4  NOP   
006D55D5  NOP   
006D55D6  NOP   
006D55D7  INC ECX   
006D55D8  JMP SHORT Acprotect.006D55C2                  ;再找   
006D55DA  MOV DWORD PTR SS:\[EBP+ECX\*4+4026E2\],0         ;找到后把这个元素从表中删除   
006D55E5  LEA ESI,DWORD PTR SS:\[EBP+4055C2\]             ;取（变形）Replace Code 表基地址   
006D55EB  MOV EAX,0A   
006D55F0  MUL ECX   
006D55F2  ADD ESI,EAX                                   ;得到本次应取（变形）Replace Code 地址   
006D55F4  PUSH ESI   
006D55F5  PUSH ECX   
006D55F6  MOV AL,BYTE PTR SS:\[EBP+4023FC\]               ;取还原 Replace Code 的操作数   
006D55FC  OR AL,AL   
006D55FE  JNZ SHORT Acprotect.006D5628                  ;跳，则操作数已经存在   
006D5600  NOP   
006D5601  NOP   
006D5602  NOP   
006D5603  NOP   
006D5604  MOV EAX,DWORD PTR SS:\[EBP+40D2AA\]             ;   
006D560A  MOV ESI,DWORD PTR DS:\[EAX+3C\]                 ;   
006D560D  ADD ESI,DWORD PTR SS:\[EBP+40D2AA\]             ;   
006D5613  ADD ESI,28                                    ;   
006D5616  LODS DWORD PTR DS:\[ESI\]                       ;得到 Entry Point 值   
006D5617  MOV BL,AL   
006D5619  ADD BL,AH   
006D561B  SHR EAX,10   
006D561E  ADD BL,AL   
006D5620  ADD BL,AH   
006D5622  MOV BYTE PTR SS:\[EBP+4023FC\],BL               ;BL 中为还原 Replace Code 的操作数   
006D5628  POP ECX   
006D5629  POP ESI   
006D562A  PUSHAD   
006D562B  MOV EAX,2   
006D5630  CALL Acprotect.006E11EA   
006D5635  OR EAX,EAX                                    ;   
006D5637  JNZ SHORT Acprotect.006D565D                  ;跳，表示把 Replace Code 放在原来变形码的位置   
006D5639  NOP                                           ;    .   
006D563A  NOP                                           ;    .    
006D563B  NOP                                           ;    .    
006D563C  NOP                                           ;    .    
006D563D  POPAD                                         ;    .    
006D563E  MOV EDI,DWORD PTR SS:\[EBP+40D2AE\]             ;否则放到位于\[ebp+40d2ae\]的指针指向的位置   
006D5644  MOV EAX,0A                                    ;一般情况下该指针指向堆中   
006D5649  MUL ECX   
006D564B  ADD EDI,EAX   
006D564D  MOV ECX,0A   
006D5652  MOV BL,BYTE PTR SS:\[EBP+4023FC\]               ;取还原 Replace Code 的操作数   
006D5658  JMP SHORT Acprotect.006D566B   
006D565A  NOP   
006D565B  NOP   
006D565C  NOP   
006D565D  POPAD   
006D565E  MOV EDI,ESI                                      
006D5660  MOV ECX,0A   
006D5665  MOV BL,BYTE PTR SS:\[EBP+4023FC\]   
006D566B  LODS BYTE PTR DS:\[ESI\]                        ;取变形 Replace Code   
006D566C  XOR AL,BL                                     ;还原   
006D566E  STOS BYTE PTR ES:\[EDI\]                        ;放置   
006D566F  LOOPD SHORT Acprotect.006D566B   
006D5671  SUB EDI,0A                                    ;得本次 Replace Code 入口地址   
006D5674  PUSH EDI   
006D5675  MOV ESI,DWORD PTR SS:\[ESP+24\]   
006D5679  SUB ESI,4   
006D567C  LODS DWORD PTR DS:\[ESI\]   
006D567D  SUB EDI,Acprotect.0040240C   
006D5683  SUB EDI,EBP   
006D5685  ADD EAX,EDI   
006D5687  MOV DWORD PTR DS:\[ESI-4\],EAX   
006D568A  POP EDI   
006D568B  PUSH EDI   
006D568C  XOR ECX,ECX   
006D568E  CMP ECX,8   
006D5691  JE SHORT Acprotect.006D56A1   
006D5693  NOP   
006D5694  NOP   
006D5695  NOP   
006D5696  NOP   
006D5697  MOV EAX,DWORD PTR SS:\[ESP+ECX\*4+4\]   
006D569B  MOV DWORD PTR SS:\[ESP+ECX\*4\],EAX   
006D569E  INC ECX   
006D569F  JMP SHORT Acprotect.006D568E   
006D56A1  MOV DWORD PTR SS:\[ESP+ECX\*4\],EDI   
006D56A4  PUSHAD                                         ;以下是把以上代码变形   
006D56A5  CALL Acprotect.006D56AA   
006D56AA  POP ESI   
006D56AB  SUB ESI,6   
006D56AE  MOV ECX,0ED   
006D56B3  SUB ESI,ECX   
006D56B5  MOV EDX,4923BF69   
006D56BA  SHR ECX,2   
006D56BD  SUB ECX,2   
006D56C0  CMP ECX,0   
006D56C3  JL SHORT Acprotect.006D56DF   
006D56C5  MOV EAX,DWORD PTR DS:\[ESI+ECX\*4\]   
006D56C8  MOV EBX,DWORD PTR DS:\[ESI+ECX\*4+4\]   
006D56CC  ADD EAX,EBX   
006D56CE  ROR EAX,15   
006D56D1  XOR EAX,EDX   
006D56D3  ADD EDX,3BC3E0C   
006D56D9  MOV DWORD PTR DS:\[ESI+ECX\*4\],EAX   
006D56DC  DEC ECX   
006D56DD  JMP SHORT Acprotect.006D56C0   
006D56DF  POPAD   
006D56E0  POPAD   
006D56E1  RETN                                            ;“回”到 Replace Code 入口地址   
  
从以上注解可以看出，为了避免程序把 Replace Code 放到堆中，修改 6D5637 的 jnz 为 jmp 就可以了，但是这段代码是变了形的，所以必须作如下处理:   
1.6D5637 的 jnz 为 jmp   
2.改 6D56A4 的代码为  jmp 6d56e0，禁止程序作代码变形   
3.把这个函数入口处第二条语句改成  jmp 6d55b7   
保存以上修改。   
F9 执行程序，我这里看到程序出错！F12 停下，检查后知是程序试图向堆中不存在的地址作串传输，于是重新载入，修改程序入口处代码，以加大堆的 size   
PUSH 5EA20   
PUSH 40   
CALL DWORD PTR DS:\[691230\]     ; kernel32.LocalAlloc   
执行上面三条语句后再转到修复的 oep 处执行，正常了，加壳，测试都没有问题。再加另一个壳，在压缩程序时程序不动了！这个现象和 shinegood 兄的一样。   
想到在修复 iat 时碰到的 MessageBox 问题（至今我还这么笨，就没去看看看看 Acpr 的文档！），想想应该还原这两个函数吧？于是在上面的语句后又加上：   
MOV EAX,Acprotect.006E084D   
MOV DWORD PTR DS:\[6911fc\],EAX   
MOV DWORD PTR DS:\[691588\],EAX   
再试一试，没问题啦！（就是不能跨系统）   
（注意：shinegood 脱壳版不能按此修改）   
  
  

 4dd466d0e1e0cc123afbe504da2462ca