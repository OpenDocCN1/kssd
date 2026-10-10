

# HTMLZip

> 原文：[`www.pediy.com/kssd/pediy03/forum615.htm`](https://www.pediy.com/kssd/pediy03/forum615.htm)

HTMLZip 脱壳后的 Import Table 的修复   
  
启动 HTMLZip, 然后启动 imp\_list.exe。   
  
找到 HTMLzip.exe，去掉 autodetect 前的框。RVA 输入 57000，长度 3000。   
  
Rebuilder 后有三个文件，Import0.bin, Import1.bin, Import2.bin。   
  
  
一、适用 trw2000 的 pedump 脱壳后的文件   
（1）打开 Import0.bin，全选，选中 2C00 个字节，复制   
Hex Workshop 打开脱壳后文件，Go to 57000，选中 2C00 个字节，   
粘贴。存盘。   
（2）用 PE-editor 打开修改后的文件，按 directory 按钮，将 IMPORT Table RVA 改为 57000, 长度为 3000 就可以了。   
  
二、适用与 ProcDump 脱壳后的文件   
（1）用 pe-editor 看 section，发现有 57000 的 section 的 Raw Offset 为 54A00。   
    改 Entry Point 为 542E4。存盘退出。   
（2）打开 Import1.bin，全选，共 924 个字节，copy。   
（3）打开脱壳后的文件，goto 54A00，选 924 个字节，paste。存盘就可以了。不用其他改动。

手动脱 HTMLZip 的壳(asprotect)   
  
有幸认识了 hying 兄，结果 hying 兄教了我如下方法进行 Asprotect1.1 版本以上的脱壳。   
hying 兄要我成功后贴个贴子，我的文笔不好，只好迎着头皮上了。   
  
软件：    HTMLZip  1.0  Beta  (下载地址为：)   
工具：    TRW2000   
  
TRW2000 装入程序，一直到下面的连续五个 Call 应该如何跟，还是等 Hying 兄的文章吧。   
告诉大家一个偷懒的办法，如果前面跟了很久也没看到这 5 个 call 的典型地方，就按 F12，   
一次一次地按，直到按下 F12 后要等将近 10 秒甚至更多时间才回到 trw 时，就按 F8，   
这就到了。:)   
  
(我的系统是 WinMe，其他系统会有不同的 xxxx:xxxxxxxx)   
016F:01110684  PUSH    EBP   
016F:01110685  MOV      EBP,ESP   
016F:01110687  ADD      ESP,BYTE -0C   
016F:0111068A  CALL    01102F7C   
016F:0111068F  JNZ      NEAR 01103C58   
016F:01110695  CALL    01104114   
016F:0111069A  CALL    01108894   
016F:0111069F  CALL    01109224   
016F:011106A4  CALL    0110B584   
016F:011106A9  CALL    01103C58    <-------- 这个 Call 进去。   
016F:011106AE  MOV      ESP,EBP   
016F:011106B0  POP      EBP   
进去后按 F10 跟，期间能够遇到 2 个 Call，都用 F10 带过，当走过第二个 RET 后，   
(懒人可按 F12, F8, F12, F8)   
代码会变成：   
016F:0110F4CA  MOV      EAX,\[ESP+0C\]   
016F:0110F4CE  JMP      SHORT 0110F4D1   
016F:0110F4D1  ADD      DWORD \[EAX+B8\],BYTE +02   
016F:0110F4D8  JMP      SHORT 0110F4F4   
下面两行动态生成：   
016F:0110F4F4  XOR      EAX,EAX   
016F:0110F4F6  RET      <--------停，看 stack 窗口，会看到 xxxxxxxx:BFFxxxxxx   
016F:0110FC4F  CALL    0110D304   
016F:0110FC54  CMP      DWORD \[01112578\],BYTE +00   
然后下 u 命令翻页，在最近的地方设断点。   
此时能看到如下代码：(110F4F4，110F4F6 为动态生成，翻页后就不见了。)   
016F:0110F51F  PUSH    DWORD 0110F600   
016F:0110F524  PUSH    DWORD 0110EF20   
016F:0110F529  PUSH    DWORD 0110EBC8   
016F:0110F52E  PUSH    DWORD 0110E58C   
016F:0110F533  PUSH    DWORD 0110E118   
016F:0110F538  PUSH    DWORD 0110F27C   
016F:0110F53D  RET    <------- 在这里下断点，然后 G 可到这里。   
bd 这个断点，再 F8。以下用 F10 走，看见类似于   
016F:0110F4CA  MOV      EAX,\[ESP+0C\]   
016F:0110F4CE  JMP      SHORT 0110F4D1   
016F:0110F4D1  ADD      DWORD \[EAX+B8\],BYTE +02   
016F:0110F4D8  JMP      SHORT 0110F4F4   
016F:0110F4F4  XOR      EAX,EAX   
016F:0110F4F6  RET      <---------------停，看 stack 窗口，会看到 xxxxxxxx:BFFxxxxxx   
就翻页找最近的 RET，重复下断点，F5, bd 断点，F8 继续。   
如此大约我进行了 7 次。   
  
\======================以上就是 hying 兄教我的方法=============================   
  
  
再接着往下说之前先讲个跟踪原则：   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
\*    xxxx:AAAAAAAA  CALL BBBBBBBB        \*   
\*    当跟踪时看到一个 Call 的时候，要看看    \*   
\*    BBBBBBBB 是不是比 AAAAAAAA 大，如果是，    \*   
\*    用 F8；如果不是，用 F10 带过。        \*   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
（当然，原则会有例外，下面会讲到什么情况下原则不适用。）   
  
现在开始，利用上面说的原则，用 F8 或者 F10 追吧。   
到了这里需要注意了，下面这段代码细看和我们前面遇上的有些不同。   
016F:0110F263  RET      
016F:0110F264  MOV      EAX,0111290C   
016F:0110F269  MOV      EDX,0A   
016F:0110F26E  CALL    0110B8D4  <------- 离 110F26E 很远   
016F:0110F273  CALL    0110F0E4  <------- 离 110F273 很近，这个 Call 一定用 F8 进去，否则 GameOver。   
016F:0110F278  RET      
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
\*    这就是原则例外(1). 当看见两个 Call 并在一起，    \*   
\*    BBBBBBBB 都<AAAAAAAA，但是第二个 CALL 的        \*   
\*    BBBBBBBB 和 AAAAAAAA 相差不如第一个悬殊，用 F8。    \*   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
很快会走到这里，看到 USER32!MessageBoxA，标志着离终点不远了。   
016F:0110F1EE  PUSH    BYTE +00   
016F:0110F1F0  CALL    \`USER32!MessageBoxA\`  <--------  注意!!!!!!!!!!!!!!!!   
016F:0110F1F5  CALL    01102CDC   
016F:0110F1FA  XOR      EAX,EAX   
016F:0110F1FC  POP      EDX   
016F:0110F1FD  POP      ECX   
016F:0110F1FE  POP      ECX   
016F:0110F1FF  MOV      \[FS:EAX\],EDX   
016F:0110F202  JMP      SHORT 0110F20E   
016F:0110F204  JMP      01102B70   
  
往下慢慢走，继续遵循我们的原则，来到这里，这是 1126C1 执行以前的代码：   
016F:01126C61  CALL    01126C7E  <----  观察下面代码，是不是很“乱”？   
                     这里一定用 F10 带，F8 就 GameOver 了。   
016F:01126C66  NOP   
016F:01126C67  MOV      BH,86   
016F:01126C69  MOV      EDI,GS   
016F:01126C6B  CALL    1A22A1A4    
016F:01126C70  DB      FF   
016F:01126C71  JNG      01126C3D   
016F:01126C73  ROR      DWORD \[ECX+EDX\],E2   
016F:01126C77  HLT      
016F:01126C78  MOV      BH,1A   
016F:01126C7A  REPNE POP  EDI   
016F:01126C7C  IDIV    DWORD \[EBX\]   
016F:01126C7E  AND      EAX,BYTE -68   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
\*    这里是原则例外(2)，            \*       
\*    走过 USER32!MessageBoxA 以后，        \*   
\*    看到类似上面这个 Call，下面代码很“乱”    \*   
\*    一定用 F10 带过。                \*   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
这就是 F10 执行 1126C1 以后的代码：   
016F:01126C61  CALL    01126C7E   
016F:01126C66  POP      EBX   
016F:01126C67  POP      EAX   
016F:01126C68  ADD      EAX,33371CB8   
016F:01126C6D  POP      ESP   
016F:01126C6E  OR      ECX,ECX   
016F:01126C70  JZ      01126C55   
016F:01126C72  MOV      \[ECX\],EAX   
016F:01126C74  ADD      EAX,EBX   
016F:01126C76  MOV      \[ESP+1C\],EAX   
016F:01126C7A  POPA      
016F:01126C7B  JMP      EAX  <------- 终于到家了，注意 Eax 的值，这就是入口值。   
  
以上这个原则其实是 hying 兄方法的复杂操作，但是有时遇上   
xor eax, eax   
ret   
最近的 ret 只能断一次时，就有用了。比如 PictureToExe 这个东东的 asprotect 脱壳。   
用上述原则，就能成功脱掉 Asprotect。   
  
想作练习的话，可以找 Advanced System Agent，前面有人在论坛上要求脱壳的，虽然   
可以用冲击波，不过练习一下纯手动脱壳也不失为一个好教材。  4dd466d0e1e0cc123afbe504da2462ca