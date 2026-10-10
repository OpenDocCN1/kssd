

# QQ1220 显 IP 补丁

> 原文：[`www.pediy.com/kssd/pediy04/kanxue499.htm`](https://www.pediy.com/kssd/pediy04/kanxue499.htm)

手把手教你修改 QQ1220   
                
声明：本文章仅为教学目的而作，请勿将此用于不法用途。   
      
前言：Oicq 是目前国内使用最广泛的聊天软件，随之也出现了许多 QQ 的补丁，其中最著名和最实用的   
      一个补丁应该算是邹丹作的显示 ip 补丁，我看了邹丹写作的《为 OICQ 820 添加显示 IP 地址和   
      端口号的功能》和《如何为程序打补丁（基础，提高篇）》，然后自己试验了一次。在实验的   
      过程中，发现那两篇文章中存在着一些的不足，特别是《如何为程序打补丁（提高篇）》一文   
      中，完全把 OrignalFirstThunk 和 FirstThunk 两个数据的作用搞反了，对初学者是一个极大的误   
      导，再加上那两篇文章都没有写到具体的实现问题，给很多朋友的学习带来不便，为此，我结合   
      自己修改 QQ 的经验，写作此文，希望能对朋友们有所帮助。   
  
思路：我们首先必须要得到对方的 ip，然后用 ip 作参数，调用 ipsearcher.dll 得到对方地址，然后显   
      示在原来的广告窗口的位置。需要说明的是，这篇文章没有详细说明如何得到 QQ 中在线好友的   
      ip 地址，因为本文重点不在于此，本文只是提出一种稍微高阶一些的补丁方法——调用外部 dll   
      来给程序添加功能。   
  
正文：首先考虑在何处显示 ip 地址的问题。现在所有的显 ip，地址补丁的 QQ 都是利用原来 QQ 的广告   
      窗口位置显示的，这样做是基于如下两点考虑：   
  
  1    现在 QQ 的 Flash 广告的确很烦人，稍不小心，就容易中招，点击到腾讯的网站上去。   
  
  2    广告窗口的 hwnd 值我们可以很方便地得到，这样，就为我们使用 SetWindowText 打下了基础。   
  
      要使用广告窗口来显示我们的信息，当然首先要屏蔽掉广告窗口上面的图片，其次，还要屏蔽掉   
      在广告窗口内部点击时，程序自动启动腾讯浏览器，进入腾讯站点。其中，屏蔽图片我们可以使   
      用 CreateFileA 断点。屏蔽浏览器的启动，我们可以使用 ShellExecuteA 断点。   
  
      使用 CreateFileA 以后，看到如下代码段：   
  
:0043AEFF 895E54                  mov dword ptr \[esi+54\], ebx   
  
\* Reference To: KERNEL32.CreateFileA, Ord:0034h   
                                  |   
:0043AF02 FF1594125800            Call dword ptr \[00581294\]   
:0043AF08 8BF8                    mov edi, eax   
:0043AF0A 83FFFF                  cmp edi, FFFFFFFF   
:0043AF0D 897D08                  mov dword ptr \[ebp+08\], edi   
:0043AF10 0F84EB000000            je 0043B001                  //我们只要修改这个跳转就可以了   
:0043AF16 8D45F4                  lea eax, dword ptr \[ebp\-0C\]   
  
      我们需要修改上述跳转到如下地点：   
  
:0043AFDE 3945FC                  cmp dword ptr \[ebp\-04\], eax   
:0043AFE1 7427                    je 0043B00A   
:0043AFE3 FFB688000000            push dword ptr \[esi+00000088\]  //使上面的跳转跳到这里   
:0043AFE9 E8D7801000              call 005430C5   
:0043AFEE 59                      pop ecx   
  
      使用 ShellExecuteA 断点栏下来以后，往上看，可以看到如下代码：   
  
:00459C7F 50                      push eax   
:00459C80 FFB64C030000            push dword ptr \[esi+0000034C\]  // \[esi+34C\]就是广告窗口 hwnd   
  
\* Reference To: USER32.GetWindowRect, Ord:015Ch   
                                  |   
:00459C86 FF1578175800            Call dword ptr \[00581778\]   
:00459C8C 8D45DC                  lea eax, dword ptr \[ebp\-24\]   
:00459C8F 8BCE                    mov ecx, esi   
:00459C91 50                      push eax   
:00459C92 E8FECE0E00              call 00546B95   
:00459C97 FF7510                  push \[ebp+10\]   
:00459C9A 8D45DC                  lea eax, dword ptr \[ebp\-24\]   
:00459C9D FF750C                  push \[ebp+0C\]   
:00459CA0 50                      push eax   
  
\* Reference To: USER32.PtInRect, Ord:01EAh   
                                  |   
:00459CA1 FF15B0175800            Call dword ptr \[005817B0\]   
:00459CA7 85C0                    test eax, eax      //修改这一句为 xor eax,eax，正好两字节   
:00459CA9 0F844C010000            je 00459DFB            
  
        上面是发送消息的时候的判断代码，接受消息的窗口，也有类似的代码如下：   
  
:004B02CC FF750C                  push \[ebp+0C\]   
:004B02CF 50                      push eax   
  
\* Reference To: USER32.PtInRect, Ord:01EAh   
                                  |   
:004B02D0 FF15B0175800            Call dword ptr \[005817B0\]   
:004B02D6 85C0                    test eax, eax        //同样改为 xor eax,eax 两字节   
:004B02D8 0F8409010000            je 004B03E7   
  
        如此，我们让广告窗口消失了，然后，到了我们如何得到在线好友的 ip，457A4A 处代码如下：   
  
:00457A44 6824010000              push 00000124   
:00457A49 57                      push edi   
:00457A4A E8CB350F00              call 0054B01A   
:00457A4F 8D86E4030000            lea eax, dword ptr \[esi+000003E4\]   
:00457A55 50                      push eax   
:00457A56 53                      push ebx   
  
        当我们执行到 457A4F 处时，健入命令：d \*(\*(esi+414)+358)，就可以看到我们所选择的在线   
     好友的 ip 地址，如何得到 414 和 358 这两个数，需要大家的经验和运气，基本上 QQ 的每一个版本   
     这两个数得值都会不同。我们现在得到了我们需要的 ip 地址，下面来到了本文最核心的地方：   
     如何把这个 ip 转换成为地址数据，显示在广告窗口。   
  
     在任何一个可以显示地址的 QQ 中，都可以看到两个文件是原版的 QQ 所不具有的，1 个是   
     ipsearcher.dll，一个是 wry.dll，其中，wry.dll 是著名的根据 ip 查地址软件“追捕”所使用的   
     数据库，ipsearcher.dll 不过是它的一个界面，让你可以更加方便的使用 wry.dll，ipsearcher.dll   
     的使用方法及其简单（后面可以看到它的使用），它里面只有一个引出函数\_GetAddress，我们   
     需要作的事情就是让 QQ 启动的时候，自动加载 ipsearcher.dll。这里需要 PE 文件格式的知识，而   
     且极易出错，操作的时候务必小心。操作步骤如下：   
  
  1    用 PEditor 打开 QQ2000b.exe，点击 director，可以看到 imp table 的 RVA 是 1CCE28，Size 是 17C。   
          
  2    使用 PEditor 的 FLC 功能，得到 RVA 1CCE28==File offset 1CCE28，好，方便了我们的工作。   
  
  3    用 WinHex 打开 QQ2000b.exe，Alt+G，填入 1CCE28，然后拖动鼠标选择，看着右下方有 Size 标注   
        出你选择的块的大小，当选择的区域达到 17C，停止拖动，Ctrl+C 拷贝选择区域到剪贴版。   
  
  4    再次用 PEditor 打开 QQ2000b.exe，点击 section，看到.data 段的起始地址为：1D1000   
  
  5    回到 WinHex，Alt+G，填入 1D1000，看到上面有很大一篇空间为 0，于是在 1D0150 处 Ctrl+B，把   
        剪贴版上的数据填充到此处（注意是 Ctrl+B，不是 Ctrl+V，Ctrl+V 是拷贝，会增加文件的长度）   
        Ctrl+S 存盘。   
  
  6    PEditor 的 director 中，手动修改 imp table 的 RVA 为：1D0150（因为 QQ 的 File offset==RVA）   
  
  7    在 WinHex 中，按照下图写入新的引入函数项。 

![](img/0ed977651af9a60a1bad54e2b91c9a87.jpg)  
  
        在此，对 OrignalFirstThunk 和 FirstThunk 项作一次说明：   
       
     系统在程序初始化时根据 OrignalFirstThunk 的值找到函数名，然后调用 GetProcAddress 函数根   
     据函数名取得函数的入口地址，然后用函数入口地址取代 FirstThunk 指向的地址串中对应的值。   
     有的程序的 OrignalFirstThunk 的值为 0，则初始化时系统根据 FirstThunk 的值找到指向函数名的   
     地址串，由地址串找到函数名再根据函数名得到入口地址，然后用入口地址取代 FirstThunk 指向   
     的地址串中的原值。   
  
     根据这个说明我们可以看出，在添加新的引入函数的时候，真正重要的是 FirstThunk 处的值，   
     OrignalFirstThunk 填不填写无关紧要，要填的话，就得填入一个真正有效的值，如果嫌麻烦，   
     可以使其为 0。具体到 QQ 中，也就是说 1D02B8 处，我们完全可以不填写任何数据，直接写 0。   
     而 FirstThunk 处的值，一定要填写正确，我们才能调用该 Dll 中的函数，在这里，FirstThunk   
     的指针关系是这样的：   
  
     1D0310-->1D02E0-->(0100)\_GetAddress   
  
     而 RVA=1D0310 在程序执行时的 VA 是 5D0310，所以，我们的代码中，写入 Call dword \[5D0310\]   
     就是写入 CALL IPSEARCHER!\_GetAddress。我们的程序中，还要使用到其它的几个函数，都可   
     以用这种方式引入，现列表如下：   
  
     Call dword \[5D0310\]    ==    Call  IPSEARCHER!\_GetAddress   
  
     Call dword \[5814D4\]    ==    Call  USER32!SetWindowTextA   
  
     Call dword \[5816E4\]    ==    Call  USER32!wsprintfA   
  
     最后就是 QQ 显地址的最后实现代码了，由于我们需要写的代码并不是很多，所以我就直接利用   
     了前面修改 CreateFileA 时，空下来的代码空间，写的时候要注意，最好在 PEditor 里面把代码   
     段的属性改为可执行的（也就是最高一位为 1）   
  
     457A4A 处如此修改：   
  
0167:00457A4A E8CB350F00      CALL    0054B01A   
0167:00457A4F E9C134FEFF      JMP      0043AF15   
0167:00457A54 90              NOP      
  
        跳到 43AF15 后，执行如下代码：   
  
0167:0043AF15 51              PUSH    ECX   
0167:0043AF16 52              PUSH    EDX   
0167:0043AF17 56              PUSH    ESI   
0167:0043AF18 8B8614040000    MOV      EAX,\[ESI+0414\]   
0167:0043AF1E 8B8058030000    MOV      EAX,\[EAX+0358\]   
0167:0043AF24 803800          CMP      BYTE \[EAX\],00   
0167:0043AF27 7436            JZ      0043AF5F   
0167:0043AF29 50              PUSH    EAX   
0167:0043AF2A FF1510035D00    CALL    \`IPSEARCHER!\_GetAddress\`   
0167:0043AF30 83C404          ADD      ESP,BYTE +04   
0167:0043AF33 FF7004          PUSH    DWORD \[EAX+04\]   
0167:0043AF36 FF30            PUSH    DWORD \[EAX\]   
0167:0043AF38 B890B95F00      MOV      EAX,005FB990   
0167:0043AF3D 50              PUSH    EAX   
0167:0043AF3E B8A0B95F00      MOV      EAX,005FB9A0   
0167:0043AF43 50              PUSH    EAX   
0167:0043AF44 FF15E4165800    CALL    \`USER32!wsprintfA\`   
0167:0043AF4A 83C410          ADD      ESP,BYTE +10   
0167:0043AF4D B8A0B95F00      MOV      EAX,005FB9A0   
0167:0043AF52 50              PUSH    EAX   
0167:0043AF53 FFB64C030000    PUSH    DWORD \[ESI+034C\]   
0167:0043AF59 FF15D4145800    CALL    \`USER32!SetWindowTextA\`   
0167:0043AF5F 5E              POP      ESI   
0167:0043AF60 5A              POP      EDX   
0167:0043AF61 59              POP      ECX   
0167:0043AF62 8D86E4030000    LEA      EAX,\[ESI+03E4\]   
0167:0043AF68 E9E8CA0100      JMP      00457A55   
  
          上面这段代码需要注意如下几个问题：   
  
    1    推荐使用 Hiew 写代码，这样修改方便，并且免除动态调试的时候各种不稳定的麻烦。TRW 的 A 命   
          令的确不是很好用。最好的一点是 Hiew 里面写了代码直接存盘就可以了，不象 TRW 里面，还得   
        转几个弯才能写回文件里面。   
  
    2    一定要注意堆栈的平衡问题。调用了\_GetAddress 和 wsprintfA 以后，都有堆栈的调整语句，这   
          些语句处理不好，马上就让程序崩溃了。   
  
    3    要注意自己的代码中，寄存器的值得保存的问题。上面众多的 push，pop 指令就是为此而写。   
  
    4    从自己的代码返回到程序原来的代码中时，原来的程序有一句 LEA  EAX,\[ESI+03E4\]，这里要注   
          意加上。   
  
          修改完代码以后，我们还需要在 eXeScope 中修改广告窗的样式，让其看起来更加美观。   
  
        经过以上几道手续，QQ 的修改工作就算最后完成了。   
  
注意：    这个显示 ip 的东西是调用的著名的“追捕”的数据库 wry.dll，所以，能够分辨出来的 ip 的多少，取决   
          于你的 wry.dll 的版本的新旧。想用得更爽的朋友，自己去找最新的追捕吧！   
  
\========================================================== 

* * *

 

 4dd466d0e1e0cc123afbe504da2462ca