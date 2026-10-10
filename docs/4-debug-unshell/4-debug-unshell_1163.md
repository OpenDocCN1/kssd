

# Visual Protect

> 原文：[`www.pediy.com/kssd/pediy03/forum1041.htm`](https://www.pediy.com/kssd/pediy03/forum1041.htm)

Bye bye Visage Software's Visual Protect   
v2.31 \[2-August-2000\] filesize:4.04mb   
http://www.visagesoft.com   
by meRlin 6-August 2000   
  
  
我没见过任何软件用它来做保护，希望将来也不会碰到，不过破解它实在很有意思。   
写这一小篇东西是为了：   
1 - 记住自己是怎么做的。   
2 - 希望告诉其他破解者如何处理这种保护。   
  
用到的工具：   
SoftIce 4.xx   
Wdasm   
Procdump   
Hexeditor   
  
  
作者的话：   
Visage Software 发布这个很专业的工具其对象是那些希望发布试用版软件的开发者。   
  
Visual Protect 是计算机软件电子商务的有效和实用的解决办法。它有着优美的界面，允许可能的用户在购买前试用你完整的软件。Visual Protect 是一种强有力的工具，提供 64 位，有安全加密的出口许可，可在世界范围内销售。   
  
它不和你程序通常功能发生关系，只是集中于维护一个安全的发布环境。Visual Protect 的使用者可完全控制如何将程序提供给最终用户。   
一旦文件或者程序由 Visual Protect 来保护并发布到您的用户手中，这种保护不能被移除。   
  
好了好了！   
我们中所有人都听说过这样的话：“现成的保护都是没用的!!”   
  
对于 VBox 类的保护也是一样，我将显示如何修改它，使得任何被这种方法保护的程序可以获得全部功能，就算保护依旧存在，而"\*.vpl"文件不存在时。第一步我将 unpack/dump 这个文件。   
  
Visage Software 显然相信自己的工作，Visual Protect 自身用它自己来保护。（如果有人 dump 了它，Visual Protect 自身就不会完全工作。）   
当你启动了被保护的程序以后，Filemon 或者 Regmon 就不能运行了。   
  
提示！你可以在保护一个新的程序的同时运行他们。   
  
用 Notepad.exe 来测试这个加壳的程序（先拷贝 notepad!!）   
运行 Visual protect，你现在可以看见"Try/Buy"nag 是如何查找它保护的程序的，按下"Try"   
\-生成新项目->下一步   
在 Trial Options   
    -check number of days, fill with any encryptionkey you like ->next   
    -you do not need to change anything here! ->next   
    -Select notepad.exe and push "Apply"   
      
  
加壳完毕后可见一些信息：   
  
Protection applied:   
1.The Protected application file(s):   
\-C:\\Program Files\\Visage Software\\Visual   
Protect\\Notepad.exe   
2.The backup file(s):   
\-C:\\Program Files\\Visage Software\\Visual   
Protect\\Notepad.bak   
  
Some more text and then:   
To distribute the protected application, please make   
The following changes to your installer:   
  
1.replace your original executable(s) with:   
  -C:\\Program Files\\Visage Software\\Visual   
  Protect\\Notepad.exe   
  
2.Add the Visual Protect file(s):   
2\. 加入 Visual Protect 文件到安装程序中：   
  -C:\\Program Files\\Visage Software\\Visual   
  Protect\\Notepad.vpl   
  -C:\\PROGRAM FILES\\VISAGE SOFTWARE\\VISUALPROTECT\\VP.DLL   
  To your installer   
  
啊哈! 监视程序需要 VP.DLL。   
  
在 Softice 的 symbol loader 里面加载 notepad.exe，按下"load exports"，然后查找 VP.DLL，选择并按"load"....哦，入口处没停!   
运行 Procdump 并观察 notepad.exe 的 Sections，   
按下"Sections"   
除了.rsrc 以外所有的 sections 都不见了，加入了两个新的：   
.VDATA   
.VCODE   
  
我们应该将.VDATA 的"characteristics"由 C00000040 改为 E00000020，如此它就可以在入口处停下了(别被.VDATA 的名字骗了)，改好后存盘退出。   
再次加载 symbol loader。进入第四个 call，在其后 0040FFA1 处可见 REPZ STOSD，STOSW 和下面的几行->两个 call(解壳/解密 calls)和下面 25-30 多行有个 CALL \[USER32!CharUpperA\]，把断点设在这个 call 后面的一个 call，继续运行（F5） 知道它被断下，F10 单步，你完成后会看到下面的东西：   
:00410032    push dword ptr \[00414A90\]   
        push eax   
        lea eax, \[ebp-0C\]   
        push eax   
        CALL \[VP!\[NONAME\]\] -intresting call!!   
        mov eax, \[ebp+08\]   
        pop edi   
        pop esi   
        pop ebx   
        test eax, eax   
        jz 0041005D   
        mov ecx, \[00414A90\]   
        add eax, ecx   
        pop ebx   
        leave   
        ret   
  
在 call 后面一行设断点，运行它! (F5)   
我们的"nag"就在这里!   
  
按下"Try"，softice 将在 call 后的这行断下，看看什么东西放进了 eax(000010CC=真正的程序入口)，下面几行放入 ecx(00400000=Imagebase)，再下面几行 add eax, ecx =004010CC。再往下走到 reg，停下来做这些：   
  
A EIP (return)   
JMP EIP (return)   
按下 Esc 退出汇编模式。   
  
现在我们可以 dump Notepad.exe 了。按 F5 回到 Windows 运行 procdump，找到 notepad.exe 一行，选中它，用鼠标右键 dump (Full)，保存文件为 Dumped.exe.   
  
现在我们再一次右击 notepad 行，点"Kill task"退出 notepad。按 PE Editor 找到 dumped.exe，选中它，改入口点为 100CC，点"OK"保存，点"Exit"退出 Procdump.   
  
我建议你将 Dumped.exe 放到一个空文件夹然后运行它，会有一个消息框告诉你 VP.DLL 不见了!拷贝 VP.DLL 进同一个目录，再次运行 Dumped.exe，它可以工作了，也没了恼人的 nag，只是仍旧说它需要 DLL，谁还在乎呢？反正我不会! dumping 后的一个缺点就是它的大小。   
我不知道是否可能将其恢复到原来状态。   
(我建议你也 dump 出 VisualProtect.exe，如果你的计算机上日期被移走了。)   
  
\--第二步--   
\--找到我们的“破解处”--   
  
正如我们以前看到的，如果软件是个 Demo 或者正式版，那个神秘的 call 里面有一些东西会被擦掉，我们跟进去看看。   
直接到 offset 00499BEC，或许你的机器上会不一样!   
(检查最后偏移地址最后三位，如果一样，你应该也在这个位置了。)   
:00499BEC    add eax, 64   
        mov ecx, 00001000   
        call 00432890   
        mov eax, \[0049E6EC\] -------- put a bpx here, then run with F5 until it break   
        mov edx, \[ebp-04\]   
        mov \[eax\], edx   
        mov eax, \[0049E6EC\]   
        cmp dword ptr \[eax\], 0C   
        jz 00499DF4   
        mov eax, \[0049E6EC\]   
        cmp dword ptr \[eax\], 02   
        jz 00499E50   
        mov eax, \[0049E6EC\]   
        cmp dword ptr \[eax\], 03   
        jz 00499EAC   
        mov eax, \[0049E6EC\]   
        cmp dword ptr \[eax\], 0A   
        jz 00499C61   
        mov eax, \[0049E6EC\]   
        cmp dword ptr \[eax\], 0B   
        jz 00499FB55   
        mov eax, \[0049E6EC\]   
        cmp dword ptr \[eax\], 04   
        jz 00499F08   
        mov eax, \[0049E6EC\]   
        cmp dword ptr \[eax\], 01   
        jz 0049A00E   
        call 0049872C   
        mov eax, \[0049E6EC\]   
        cmp dword ptr \[eax\], 00   
        jz 00499C88   
  
\--more code--   
      
我敢说我们已经很接近我们的破解处了。   
实际上所有的指针对在和 eax 指的内存位置进行比较，dump 它(内存位置 0049E204)，将它的值改为 A，然后按 F5；P)   
这个程序过期了!   
  
再回到那里，将字节改为 1(!!你一定要等到所有的那些 ffffffff 都被改写后再改动)，然后再运行，!!破解了 :)   
正如你看到的，你可以试验很多选择-> A,B,C,1,2,3,4,0.   
关闭 notepad，清楚所有的断点。   
  
我们把它找出来!   
首先，我们需要找到何时一些字节会写到 offset 00??E204 ok! (就是 EAX 的内存地址)   
With notepad.exe it will be 0049E204 (InMyComputer)   
对 notepad.exe 而言，它将是 0049E204 (在我的计算机上)   
在 softice 里这样做：BPMB DS:0049E204 RW   
再次运行 notepad.exe，softice 会在这里中断：   
（你按下第二次 F5 以后）   
  
        jb    004B02BC   
:004B0293    repz movsd          ----------land here   
        jmp    \[edx\*4+004B03A8\]   
        mov eax, edi   
        mov edx, 00000003   
        sub ecx, 04   
  
你会看见内存地址被填满 FFFFFFFF，不过最终我们需要它成为 00000001=Registered OK!   
下一步，找到所有这些 FFFFFFFF 从哪里来的，按 F5 一次，当再次中断时，你会看到这些：   
        push dword ptr FS:\[eax\]   
        mov FS:\[eax\], esp   
        mov eax,\[0049E6EC\] <---------------this points to 0049E204   
:0049A910    mov dword ptr \[eax\], FFFFFFFF <--- fill with this   
        push 00002000      ---------------you land here   
  
在 softice 里下 A CS:0049A910 (return)   
写入 mov dword ptr \[eax\], 00000001(Esc 退出汇编模式)   
将 eip 改回到 offset 0049A910 (R EIP)   
然后运行它，会中断几次来进行合法检查，跳过他们，就像它已经注册了(也是欺骗注册文件)   
我们的破解只是改了上面提到的代码，我可以告诉你你不会在 VP.DLL 里面找到这些代码(它被加壳了在运行时候被 load 进来?)   
  
mov byte ptr \[eax\], FFFFFFFF 和 0xC700FFFFFFFFh 等价，如果你在 0049A910 处（这里是我们看到代码的地方）dump，你会看到它们。0049A910 只有在我们的 notepad 运行时候可以看到，如果你保护的是其他软件，offset 会改变，不过最后四位 xxxxA910 总是一样的。   
这种补丁在内存中很容易做到，不过我们的目标是对 VP.DLL 补丁，这样就可以让它在运行时自己补自己。   
  
\---第三部分---   
\---我们如何做---   
  
首先，我们需要一些地方来放我们的新代码。用 hexeditor 打开 VP.DLL，往下卷一点，你会看见很多 00，这里可以放我们的代码。   
我决定用 offset 00000360 作为起始点，在这个 offset 写入"here"。保存文件。   
为什么？当我们需要定位的时候很容易找到它（它可以成为一个通用 patch，我们有时要找到它，这是就需要 search 它一下。）   
从哪里来调用我们的代码呢？我们要在 VP.DLL 里面找一些地方，那里的代码不会改变他们自己(就是真正的 offset)   
这次我们很幸运，这个文件没有 CRC 校验。   
  
再次运行 notepad，再一次停在这里：   
        jb    004B02BC   
:004B0293        repz Movsd            
        jmp    \[edx\*4+004B03A8\]   
        mov eax, edi   
        mov edx, 00000003   
        sub ecx, 04   
  
从这里单步 F10，直到下一个 ret，越过 call \[KERNEL32!GlobalFree\]，最终越过下一个 ret，出现下面代码：   
:004AFC44    pop ecx ------------------you are here   
        mov eax, \[004B4A28\]   
        mov ecx, \[eax+3C\]   
        add esi, 28   
        inc edi   
        movzx ecx, word ptr \[eax+ecx+06\]   
        cmp edi, ecx   
        jl    004AFC1F   
    5F    pop edi    -------------   
    5E    pop esi                |   
    C3    ret                    - we can use the space here to jump to our patch!   
    55    push ebp                - 9 bytes is enough to do it.   
    8BEC    mov ebp, esp            |   
    83EC10    sub esp, 10  ------------   
  
我们需要查查这些代码是否用了很多次，如果这样的话，就要一个计数器了，或许第一件事是在 hexeditor 里面找找它。查找 5F5EC3558BEC83EC10，你会在 offset 0003DC5A 处找到。(VP.DLL)。   
在 POP EDI 处下 bpx，按 F5。我们真运气！这个代码只用了一次。在停在这里后，你会在我们需要打补丁的地方停下了（如果你仍旧还有 R/W bpx 在）   
Ok! 这就是我们（应该说我）用来跳到我们补丁代码的地方了。   
如何找到我们应该跳到何处呢？   
如果你做了我刚才说的（在 vp.dll offset 0000360 处写入"here!"），这就不成问题了。找到它。   
  
下一步在我们的断点周围(POP EDI)看看，是否能找到当运行不同程序的时候的保护模式。   
停掉所有的 R/W 断点，留下其他的"on"，当 softice 在 bpx 处中断时，寄存器看起来象这样：   
  
    this values is for notepad.exe   
    EAX 00430000  EBX 81747B68  ECX 00000003  EDX 005C0008  ESI 00430270   
    EDI 00000003  EBP 006EFC24  ESP 006EFC10  EIP 004AFC5A   
  
    and this is for Visual Protect itself (VisualProtect.exe)   
    EAX 007E0000  EBX 81747B68  ECX 00000003  EDX 00970328  ESI 007e0270   
    EDI 00000003  EBP 00A9FC24  ESP 00A9FC10  EIP 0085FC5A   
  
    finaly I did create a protected calc.exe and this is the values for it   
    EAX 00400000  EBX 81747B68  ECX 00000003  EDX 00590008  ESI 00400270   
    EDI 00000003  EBP 005FFC24  ESP 005FFC10  EIP 0047FC5A   
  
如果看看 eax 指向的内存位置，我们就能得到 VP.DLL 的入口点(dump 它，你会看到的)。好吧，现在我们从这里开始，如果我们用 eax 来找我们的补丁代码，就可以不管被保护文件装载时的内存位置了，我们来找到它。   
  
如果用 eax，我们可以找到我们的代码，唯一要记住的是加上 360，（文件 offset 360），这样就指向了我们补丁代码的第一位。   
单步(F8)直到 ret，我们该下手了! 看看发生了什么   
edi 为 0，ecx=esi=eax，这是要保证我们没有破坏重要的东西。   
用 edi 来补丁是安全的，(ECX 也不错，不过最好在离开时恢复它)要做这样的补丁，我们需要找到当我们补丁成功时如何回来的路。   
  
我确实找到了一个“聪明”的方法来做这些，我看看周围，发现内存 esp-10 处是最适位置，那里的值一直不变（惟一不同的地方是 offset 的前 4 位，比如 00??FC37）。它指向一个 eip 地址(和我们的一样)，这就是我们回来的路!（当然我们需要加上 0x23h 在我们用它之前，为什么？从 00??FC37 到 00??FC5A 我们跳到补丁的地方相差 0x23h）   
  
我们在 00??A910 处得到补丁代码了。   
现在运行另外一个被保护的程序，看看这些代码在那里被加载，如果我们在此行下断点，就能看到不同了。   
  
看起来代码始终（从我们的观点来看）被加载于低于我们 eip（从这点我们跳向我们的新代码）10000 个 byte 的地方，记住这点! 我们要用这个值来在我们的补丁中找到它。   
  
那么是什么呢？   
汇编 POP EDI 行，（写下老的代码）   
  
        add eax, 0360 -to file offset for first byte in our patch   
        jmp eax      - jump to it   
        nop          - to fill   
        nop          - to fill   
  
写下新代码 (0560030000FFE09090)   
  
打开 VP.DLL 查找老代码，将他们替换为新代码，保存文件为 patched.dll！将原来的 VP.DLL 改名，然后将 patched.dll 改为 VP.DLL.   
  
现在我们有些事情要记住!   
  
首先要从我们的代码处回来需要加上 23h，这个值储存在\[esp-10\]，它等于 eip=00??FC5A。   
  
其次，我们要打补丁的偏移地址要减掉 10000(如果我们比较它和\[esp-10\]处的 eip)，比如 004AFC37，首先清掉 FC37 为 004A0000，然后减掉 10000->00490000，最后加上 A910 成为 0049A910，这是要 patch 的偏移地址。   
  
运行带有打过补丁的 dll 的 notepad，停在跳往 004AFC5A 的这一行，准备跳向（00???360），汇编如下：   
  
  
File offset:   
00000360   
        mov eax, \[esp-10\]          - grab eip ->00??FC37   
        XOR AH, AH                - zero ah   
        XOR AL, AL                - zero al   
        DEC EAX                    - decrease eax   
        SUB EAX, FFFF              - substract with FFFF   
        ADD EAX, 0A910            - EAX gets memorylocation to patch   
        ADD EAX, 02                - add 02 to move past the C7 byte   
        MOV DWORD PTR \[EAX\], 00000001    - patch it   
        MOV EAX, ESI   
            ^^^^|^^^                  
    eax 需要回到我们跳之前的值，esi 保存剩余的值。   
                            --------   
        XOR AH, AH                  - zero ah      |--just in case   
        XOR AL, AL                  - zero al      |--something is added   
                            --------   
        MOV EDI, \[ESP-10\]            - get back the eip   
        ADD EDI, 023                - add 23h to correct it   
        MOV DWORD PTR \[EDI\], 55C35E5F   
                    ^^^^|^^^        
    改回原来的代码，这在内存 00??FC5A。   
  
        ADD EDI, 04                  - move forward   
        MOV DWORD PTR \[EDI\], EC83EC8B    - change some more bytes back   
        ADD EDI, 04                - move forward   
        MOV BYTE PTR \[EDI\], 10        - the last byte to change back   
        MOV EDI, \[ESP-10\]            - get back the eip   
        ADD EDI, 023                - add 23h to correct it   
        JMP EDI                      - all done! jump back   
  
当你回到那里时，还是原来的代码（在你打补丁之前），看起来就像什么对没发生过一样。  4dd466d0e1e0cc123afbe504da2462ca