

# Themida 的另类破解

> 原文：[`www.pediy.com/kssd/pediy10/72152.html`](https://www.pediy.com/kssd/pediy10/72152.html)

【文章标题】: Themida 的另类破解  
【下载地址】: 自己搜索下载  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
      大家都知道 Themida 的强大保护功能，几乎无人能完整的破解它的 VM 中的 handler（最新进展如何？）。我也试图跟踪过它的几个 handler，它为一个简单的入栈操作竟有长达数千行的垃圾代码和无穷的转跳，如果要把它 168（？）个 handler 都清理出来，他一定会疯掉！或累死掉？我曾经下决心不碰 Themida 了，不知为什么鬼使神差的我又跟踪了几个 Themida 加密软件，竟然还有新的发现！  
      附件提供的一个程序是用汇编写的 PE 信息小工具，用 Themida1855 加密，加密选择 RISC-64 processor，全选加密可选项，并对部分 API 函数使用 VM 加密，入口虚拟系数选择 15。应该说对该程序用了较强的加密措施。现在介绍用我的脚本，一步步对它破解(绝不暗中使用对原程序已知的信息)，居然还原了所有的代码，请看下面详细过程。  
    
      一、介绍一个万能脚本  
      1.下面的脚本适用于任何用 Themida 1.8.5.5 版加密的程序，操作十分简单。  
  --------------------------------------------------------------  
  //本脚本适用于 Thmida 1.8.5.5 版本加密的任何程序  
    
    bphwc  
    bc  
    jmp Second     //第一次运行后,将本行注释掉后保存一次(本)脚本  
    
  data:  
    var mem  
    var mem1  
    var temIAT  
    var temESI  
    var temAPI  
    var APIstr  
    var Addr\_0  
    var Addr\_1  
    var Addr\_2  
    var Addr\_3  
    var Addr\_4  
    var Addr\_5  
    var Rva  
  Init:  
    mov Rva,65b970       //用对话框中的值修改该值,并将第一行的 jmp Second 注释掉  
    mov Addr\_0,Rva-997  
    mov Addr\_1,Rva-809  
    mov Addr\_2,Rva-ea  
    mov Addr\_3,Rva-9d  
    mov Addr\_4,Rva  
    mov Addr\_5,Rva+74c  
      
  start:    
    esto  
    esto  
    bphws Addr\_0,"x"   
    esto    
    bphwc Addr\_0  
    mov \[Addr\_0\],0062e990  //废除监视断点的代码  
      
    bp Addr\_1       
    bp Addr\_2      
    bp Addr\_3      
    bp Addr\_4      
    bp Addr\_5      
    
  First:  
    run  
    cmp eip,Addr\_1  
    jz  A1  
    cmp eip,Addr\_2          
    jz  A2  
    cmp eip,Addr\_3  
    jz  A3  
    cmp eip,Addr\_4  
    jz  A4  
    cmp eip,Addr\_5  
    jz  A5  
    jmp  First  
    
  A1:      
    mov temAPI,eax     //eax 是 API 函数地址  
    gn temAPI         //显示函数名  
    log $RESULT   
    add APIstr,"  "  
    add APIstr,$RESULT\_2  //恢复 API 函数名(INT 表)  
    jmp First  
  A2:      
    mov temIAT,eax  
    cmp  edx,10000  
    ja  next  
    xor  edx,80000000  
    mov \[eax\],edx        //修复 IAT 表(写入序列号)  
    jmp First  
  next:   
    mov \[eax\],temAPI  
    jmp First  
    
  A3:  
    mov mem1,eax          //eax 是内存中呼叫 API 地址  
    mov temESI,\[esi\]      //获取转跳标记  
    jmp First  
    
  A4:  
    cmp temESI,AAAAAAAA  
    jnz step3  
    mov \[mem1\],#FF25#      //修复代码中转跳地址  
    mov \[mem1+2\],temIAT     
    jmp First  
    
  step3:  
    mov \[mem1\],#FF15#      //修复代码中呼叫地址  
    mov \[mem1+2\],temIAT  
    jmp First  
    
  A5:  
    bc       
    pause  
    
  Second:                //查找写 IAT 地址的一条指令  
    var memory  
    var tmp  
    var temp  
    var tmpbp  
    var Str  
    
    gmi eip,CODEBASE  
          mov  memory,$RESULT    
    find memory,#0000000000000000000000000000000000000000000000#  
    mov tmp,$RESULT  
    bphws tmp,"w"  
          esto  
    esto  
    esto  
    bphwc tmp  
    sto  
          find memory,#909090909090909090909090909090909090#  
    mov temp,$RESULT+2  
    bphws temp,"w"  
    mov  tmpbp,eip  
    run  
    cmp  eip,tmpbp+2  
          jnz  next2  
          run  
    next2:  
    itoa eip  
    mov  Str,"请用下列数值："  
          add  Str,$RESULT   
    add  Str," 修改脚本中的 Rva."  
    bphwc  
       msg Str  
    pause                       //（脚本完）  
    
      本脚本的使用方法是：用 OD 打开程序后，如果第一次加载本脚本，则会弹出一个对话框，用对话框中的数据修改脚本中的 Rva 值（第 20 行，且只有一处），并一定将第 3 行的 jmp Second 注释掉，一定将脚本保存一次！这时脚本就是该程序的专用脚本了。重新用 OD 加载程序后，可以无须任何更改，直接调用本脚本了！  
      第 2 次运行脚本，屏幕快速闪动，一会就出现了神奇的效果了。如果想应用于新的程序，则只须将注释掉的 jmp Second 恢复即可。  
  ----------------------------------------------------------------------------  
    
      2.挖掉 Themida 的一只监视眼，蒙上它的另一只眼  
      本脚本的运行原理，请参考我原先的一篇文章《对 Themida 加密 VC++程序的完美脱壳》，当时我初次接触 Themida，那个脚本有很多不完善的地方，如不容易进行到底，移植困难等。Themida 有很多监视断点的 SEH，当时没有采用相应措施，所以被 Themida 频频发现。它的“监视”方法很多，用得最多的大约有(1)int 1 中断，(2)内存访问异常中断，(3)代码扫描等。对于(1)(2)没有任何办法摘除，但脚本的一个循环可与它周旋；对于(3)在我跟踪的代码段中，用脚本将它摘除了。所以脚本运行得非常顺利(使用在其它被 Themida 加密的程序中也一样)。  
      (1)脚本中的 mov \[Addr\_0\],0062e990 就是将该地址的“jb”改成了“jmp”，跳过了代码扫描监视。---相当于挖掉了它的一只眼睛---  
      (2)Thmida 的另一(多)只眼睛实在难于发现，不管你是什么类型的断点，很快被发现并单步异常后弹出对话框，然后退出运行。请注意，Themida 设置的是单步中断，如果让它自行处理这个中断，就是前面的结果。如果将它拦截下来，自已处理就可以绕过了它的 SEH。“拦截”方法就是打开 OD“选项--调试设置--异常页”只选“非法访问内存”一项，其余全部不能选取。这样除非法访问内存异常让它自行处理外，其余全部交由 OD 处理。这样当 Themida 发现断点后产生的单步异常就被 OD 拦截了。自行处理的方式就是脚本中“First：”标签部分。(用脚本处理 OD 拦截下来的中断是非常容易的)  
       脚本中的 First 部分(断点循环)就是判断中断地址是不是自己设置的，是则进入相应操作；若不是，则一定是 Thmida 发现断点后的 SEH 中断地址(单步中断)，处理方式就是用“run“。OD 的这个 run，就是让程序在 eip 中断地址上继续运行，绕过了 Themida 的 SEH 处理程序(陷阱)。---相当于蒙上了它的另一只眼睛----      
       脚本中的其余部分无须多说，请参考我的前一篇文章。  
    
      3.运行脚本后的神奇效果  
      用 OD 打开程序后，加载这个脚本，程序运行得很快。当它暂停后 Themida 外壳已经被除掉。转到代码段 401000，让 OD 分析代码一次。如果 Themida 在 code 段中没有虚拟 API 函数，则显示在你面前的是一个非常清晰的可读的反汇编程序，各个 API 函数调用和名称显露无遗。如果你仅仅是想了解原程序的基本结构和思路，这就够了。如果 Themida 虚拟了部分 API，请往下看。如果你一定要 dump 出来，并去掉垃圾代码，也请往下看。  
    
      二、恢复程序的入口代码  
      如果想进入它的 VM(虚拟机)部分，清理它的 handler，还原出原代码，我是绝对无能为力了，下决心不碰它了。  
      那么，我是怎样恢复入口代码的呢？跟踪 Themida 发现，它对解压出来的代码段并不重视，居然没有设置断点监视？你可以任意设断和修改，我巧妙地利用了它这一弱点。  
    
      1.设断技巧（以提供的 PeInfo\_them.exe 为例）  
      用脚本脱壳后，转到 401000 代码区，让 OD 对代码分析一次，来到程序的尾部，API 函数都集中在这里(称它为 API 表)(VC++程序的函数表分散在代码中,相对集中)。  
        
  004015FC   $- FF25 5820400>jmp     dword ptr \[402058\]  ;  USER32.wsprintfA  
  00401602   .- FF25 5420400>jmp     dword ptr \[402054\]  ;  USER32.DialogBoxParamA          <---F2 设断  
  00401608   $- FF25 5020400>jmp     dword ptr \[402050\]  ;  USER32.EndDialog  
  0040160E   $- FF25 4C20400>jmp     dword ptr \[40204C\]  ;  USER32.GetDlgItem  
  00401614   .- FF25 4820400>jmp     dword ptr \[402048\]  ;  USER32.GetWindowTextLengthA     <---F2 设断  
  0040161A   $- FF25 4420400>jmp     dword ptr \[402044\]  ;  USER32.LoadIconA  
  00401620   .- FF25 4020400>jmp     dword ptr \[402040\]  ;  USER32.MessageBoxA              <---F2 设断  
  00401626   .- FF25 3C20400>jmp     dword ptr \[40203C\]  ;  USER32.SendMessageA             <---F2 设断  
  0040162C   $- FF25 3820400>jmp     dword ptr \[402038\]  ;  USER32.SetWindowTextA  
  00401632   $- FF25 2420400>jmp     dword ptr \[402024\]  ;  kernel32.CloseHandle  
  00401638   $- FF25 2820400>jmp     dword ptr \[402028\]  ;  kernel32.CreateFileA  
  0040163E   $- FF25 2C20400>jmp     dword ptr \[40202C\]  ;  kernel32.CreateFileMappingA  
  00401644   .- FF25 3020400>jmp     dword ptr \[402030\]  ;  kernel32.ExitProcess            <---F2 设断  
  0040164A   .- FF25 2020400>jmp     dword ptr \[402020\]  ;  kernel32.FreeLibrary            <---F2 设断  
  00401650   $- FF25 1C20400>jmp     dword ptr \[40201C\]  ;  kernel32.GetFileSize  
  00401656   .- FF25 1820400>jmp     dword ptr \[402018\]  ;  kernel32.GetModuleHandleA       <---F2 设断  
  0040165C   .- FF25 1420400>jmp     dword ptr \[402014\]  ;  kernel32.LoadLibraryA           <---F2 设断  
  00401662   $- FF25 1020400>jmp     dword ptr \[402010\]  ;  kernel32.MapViewOfFile  
  00401668   $- FF25 0C20400>jmp     dword ptr \[40200C\]  ;  kernel32.UnmapViewOfFile  
  0040166E   $- FF25 0820400>jmp     dword ptr \[402008\]  ;  kernel32.lstrcpyA  
  00401674   $- FF25 0020400>jmp     dword ptr \[402000\]  ;  comdlg32.GetOpenFileNameA  
    
      (1)注意到 API 表中有“$”的地方(如果全都没有,则再分析一次)，$表示程序中有 call 在呼叫，若没有则表示没有关联(VC++编写的程序有很多无关联的垃圾函数)。在这里没有关联的函数肯定是被 Themida 虚拟了 call 代码。在没有“$”的地址上设 F2 中断。  
       明眼一看，本例是一个对话框窗口，它只有 DialogBoxParamA，没有 CreateWindowEx、消息循环等函数。若是 Window 窗口，则不能在消息循环函数上设断，否则 OD 将进入假死状态。    
      (2)在解压后代码段中浏览，发现了一段连续的乱码，用 OD 分析也无用，它一般就是入口代码段了。本例是 4015B6--4015FB 这一段(被虚拟了)。  
      (3)从 401000 开始寻找跳进 006xxxxx 段的 jmp，这些 jmp 都是跳到被 VM 虚拟后的 API 代码中(代码 E9 前面都有一个“-”号)，本例中跳进 VM 中的 8 个 jmp 地址如下：  
    
  0040100D   .- E9 E0532300  jmp     006363F2  
  00401029   .- E9 408F2300  jmp     00639F6E  
  0040103E   .- E9 95CC2300  jmp     0063DCD8  
  00401371   .- E9 4A092400  jmp     00641CC0  
  00401385   .- E9 F0422400  jmp     0064567A  
  004013C9   .- E9 B2802400  jmp     00649480  
  004013DD   .- E9 61B52400  jmp     0064C943  
  0040151B   .- E9 39F32400  jmp     00650859  
      在以上的每个地址都用 F2 设断。（放心设断,Themida 是不来过问的。）  
    
      2.操作技巧(以后的操作基本上都是 F9 或 alt-F9，无须 shift-F9)  
      (1)当设置完成后，按 F9 运行一次，程序中断在  
  0040165C   .- FF25 1420400>jmp     dword ptr \[402014\]  ;  kernel32.LoadLibraryA  
      且在堆栈中出现  
  0013FF90   00654697  /CALL 到 LoadLibraryA  
  0013FF94   00402060  \\FileName = "RichEd20.dll"  
  0013FF98   004001C0  ASCII "   "                <--这是应用程序开始的栈顶，dump 堆栈平衡时弹出至此  
  0013FF9C   0013FFE0  指向下一个 SEH 记录的指针  
  0013FFA0   005E0DB2  SE 处理程序  
  0013FFA4   7C930738  ntdll.7C930738  
      堆栈中 0013FF98 是进入应用程序后的栈顶，13FF94 的(00402060)是压入的参数，0013FF90 的(00654697)是 VM 中的呼叫地址。  
      记录下堆栈中的数据，并将 4015B6 开始的乱码改写为：(它就是解密出来的入口第一组代码)  
      push  00402060  
      call  0040165C  <---虚拟机中怎样操作不知道，而还原后的代码就是 call LoadLibraryA，即 call 0040165C。  
    
      (2)继续 F9 一次，中断在  
  00401656   .- FF25 1820400>jmp     dword ptr \[402018\]  ;  kernel32.GetModuleHandleA  
    
      堆栈中出现  
  0013FF90   006546A3    /CALL 到 GetModuleHandleA  
  0013FF94   00000000    \\pModule = NULL  
      这里记下堆栈，接着前面如下修改乱码；  
      push  0  
      call  00401656   <---入口的第二组数据  
    
      (3)继续一次 F9，(记录每一次中断后堆栈中的数据)中断在  
  00401602   .- FF25 5420400>jmp     dword ptr \[402054\]  ;  USER32.DialogBoxParamA  
      堆栈是  
  0013FF80   006546BB    /CALL 到 DialogBoxParamA  
  0013FF84   00400000    |hInst = 00400000            <--GetModuleHandleA 的返回值  
  0013FF88   000003E8    |pTemplate = 3E8             <--资源 ID  
  0013FF8C   00000000    |hOwner = NULL  
  0013FF90   00401544    |DlgProc = PeInfo\_t.00401544 <--窗口程序地址  
  0013FF94   00000000    \\lParam = NULL  
    
    
      因为程序进入了 DialogBoxParamA 后，不会退出，但初始化窗口时还会调用其它函数，只有关闭窗口后才退出。  
      这里将前面 3 次中断后堆栈中的值，加上必须保存的句柄，将入口处的乱码改写如下：  
      push  402060  
      call  0040165C  ;LoadLibraryA  
      mov   \[xxxxx\],eax                  <---返回 RichEdit20A 句柄  
      push  0  
      call  00401656  ;GetModuleHandleA  
      mov   \[xxxxx\],eax                  <---返回 GetModuleHandleA 句柄  
      push  0  
      push  401544  
      push  0  
      push  3E8  
      push  \[xxxxx\]                      <---它一定是 GetModuleHandleA 返回的句柄  
      call  00401602  ;DialogBoxParamA  
    
     打开 403000 全局变量区，每中断一次就会写入一些数据。你会发现 LoadLibraryA 句柄存入了\[403004\]，GetModuleHandleA 句柄存入了\[403000\]，这样前面的三个\[xxxxx\]都解决了(需要一些 API 函数知识)：即  
    
      push  402060                        <---查 402060 是字串 RichEd20.dll，原来是装载 RichEd20 库  
      call  0040165C       ;call LoadLibraryA  
      mov   \[403004\],eax                  <---返回 RichEdit20A 句柄  
      push  0  
      call  00401656       ;call GetModuleHandleA  
      mov   \[403000\],eax                  <---返回 GetModuleHandleA 句柄  
      push  0  
      push  401544    <--DialogBoxParamA 函数的 Proc 地址  
      push  0  
      push  3E8       <--窗口资源 ID  
      push  \[403000\]                      <---它是 GetModuleHandleA 返回的句柄  
      call  00401602       ;call DialogBoxParamA  
      这就是入口的开局部分。  
  
     提问：如果程序中有如：mov eax,12345678； mov ecx,eax；mov \[403040\],ecx；add eax,ecx 等代码，怎么办？  
     应该说，每种语言编写的程序入口都有一种相对固定的格式，在窗口未建立之前一般不会有 mov ecx,eax；add eax,ecx 等代码，熟习了各种语言的程序入口代码对破解大有帮助。如果在脚本中设置了 bpmw \[40x000\],ff（\[40x000\]是数据区），则对找回如:mov \[xxxxx\],eax 等代码有一定的帮助，但断点在 VM 中，要防止 Themida 的监视。退一步说，即使漏掉了一些这样的代码，dump 后的程序不能运行，但对程序的结构和思路已经完全掌握了，目的已经达到了。  
        
     (4)继续一次 F9，中断在地址 401371，(窗口初始化调用的函数)  
  00401371   .- E9 4A092400  jmp     00641CC0  
     这是 code 中被虚拟了的 API 函数，将该地址设置的中断 F2 关闭(因为程序可能反复调用它，以后每当在前面的 8 个 jmp 上中断一次，就关闭该断点)，再 F9 一次，中断在 API 表中的  
  00401626   .- FF25 3C20400>jmp     dword ptr \[40203C\]  ;  USER32.SendMessageA  
     堆栈中是  
  0013FC14   00641CD7    /CALL 到 SendMessageA  
  0013FC18   004600B2    |hWnd = 4600B2            
  0013FC1C   00000080    |Message = WM\_SETICON  
  0013FC20   00000001    |wParam = 1  
  0013FC24   0C1A027D    \\lParam = C1A027D  
     原来 401371 加密前应该是 call 401625(即 call SendMessageA)，查看地址 401371，参数非常清晰，你不必费力去研究它的参数来源，它是装载图标。  
     注意，这个 call SendMessageA 不是接在开局的 call DialogBoxParamA 后面，因为程序还没有退出 DialogBoxParamA。  
     记下 00401371 地址，将要修改为 call 401625(参数修复前不要修改，可能程序会反复调用它)。  
     ……………………………………  
     这样反复 F9，反复记下中断地址和中断函数，(每中断一个 jmp 的地址，就关闭它的断点(API 表中的 F2 不能关闭)。这样凡是与初始化有关的被虚拟的 API 都现身了！若发现再 F9，老在 API 表中的 SendMessageA(或其它函数)上中断，这是初始化中的调用，运行时总要进入系统领空，可按 alt-F9 让其返回(在 VM 中)，再 F9 回到用户领空。窗口未出现前可以先关闭 SendMessageA 的 F2 断点，但窗口出现后一定要恢复其断点。  
      
    (5)如果 F9 后，OD 没有反应，但状态栏提示读取\[FFFFFFFF\]异常，不管它，这时，窗口已经出现了(看桌面状态条），程序进入了 DialogBoxParamA 中的消息循环。  
     到此为止，代码段中还有 5 个断点没有被访问，它们是窗口运行后才要访问的地址。  
    
    (6)按普通程序一样运行窗口，让它随便打开一个.exe 文件，它立即中断在某 jmp 虚拟函数上，按前面同样的方法操作，在 API 函数表中就发现了它调用的 API 是谁了，这样就一个个地把 Themida 虚拟的函数清理完了。当代码段中的 F2 中断地址关闭完后。被 VM 虚拟的全部函数都现身了。（如何恢复参数，后面再说）  
       
    (7)反复运行窗口发现在代码中设置的断点 0040151B9(jmp 00650859)始终没有被调用，而 API 表中的 MessageBoxA 也没有被调用，可以肯定 0040151B9(jmp 00650859)是虚拟了 call MessageBoxA。原来它是一个出错对话框，当你打开一个非 exe 文件时，对话框就出现了。在堆栈中也可以找到它全部参数。  
      
    (8)入口的后半部  
    当清理完代码段中的所有 jmp 断点后(被虚拟的 API)，可以关闭窗口了，按下关闭钮，立即中断在  
  0040164A   .- FF25 2020400>jmp     dword ptr \[402020\]  ;  kernel32.FreeLibrary  
    堆栈是  
  0013FF90   006546CA    /CALL 到 FreeLibrary  
  0013FF94   74D90000    \\hLibModule = 74D90000      <--LoadLibraryA 的句柄  
    即：接着 call DialogBoxParamA 后面，按下面代码修改  
      push  \[403004\]  
      call  40164A       (call FreeLibrary)  
    再 F9，中断在  
  0013FF90   004015FC    (CALL 到 ExitProcess)  
    再 F9,就退出了。那么，完整的入口代码是：  
      push  402060                    
      call  0040165C    
      mov   \[403004\],eax                    
      push  0  
      call  00401656    
      mov   \[403000\],eax                    
      push  0  
      push  00401544      
      push  0  
      push  3E8        
      push  \[403000\]                        
      call  00401602    
      push  \[403004\]  
      call  0040164A  
      push  0  
      call  004015FC  
    
      三、参数的恢复  
      API 参数的设置，需要一些 API 知识，以本例为例简要说明，回到前面，401000 这里有 3 个被虚拟的函数。前面的操作知道，先中断在一个 jmp 上，接着中断在 API 表中对应的 API 函数上。这样都清楚了它们是 call xxxxxx。  
  
  00401000   $  55            push    ebp  
  00401001   .  8BEC          mov     ebp, esp  
  00401003   .  83C4 F8       add     esp, -8  
  00401006   .  60            pushad  
  00401007   .  FF35 0C304000 push    dword ptr \[40300C\]  
\->0040100D   .- E9 E0532300   jmp     006363F2            <--应该是 call GetWindowTextLengthA  
  00401012   .  D3F4          sal     esp, cl  
  00401014   .  0D B38ACE8D   or      eax, 8DCE8AB3  
  00401019   .  45            inc     ebp  
  0040101A   .  F8            clc  
  0040101B   .  50            push    eax  
  0040101C   .  6A 00         push    0  
  0040101E   .  68 37040000   push    437  
  00401023   .  FF35 0C304000 push    dword ptr \[40300C\]  
\->00401029   .- E9 408F2300   jmp     00639F6E              <--应该是 call SendMessageA  
  0040102E   .  0B37          or      esi, dword ptr \[edi\]  
  00401030   .  B0 F7         mov     al, 0F7  
  00401032   .  F5            cmc  
  00401033   .  68 C2000000   push    0C2  
  00401038   .  FF35 0C304000 push    dword ptr \[40300C\]  
\->0040103E   .- E9 95CC2300   jmp     0063DCD8              <--应该是 call SendMessageA  
  00401043   .  46            inc     esi  
  00401044   .  6F            outs    dx, dword ptr es:\[edi\]  
  00401045   .  C2 0400       retn    4  
    
       (1)第一个 call GetWindowTextLengthA，参数只有一个 push \[40300C\]，前面已知这是 RichEdit20A 的句柄，显然它是获取编辑软件 RichEdit20A 中的字符数，结果应该暂存，eax 怎样操作？  
       (2)第 2 个是 call SendMessageA，消息发往 RichEdit20A，堆栈中的表示是：  
  0013F798   00639F81  /CALL 到 SendMessageA  
  0013F79C   00450292  |hWnd = 450292  
  0013F7A0   00000437  |Message = MSG(437)  
  0013F7A4   00000000  |wParam = 0  
  0013F7A8   0013F7CC  \\lParam = 13F7CC  <--移动光标或选择范围的操作是该参数指向一个 CHARRANGE 结构(8 字节)  
  
       这下难了，必须弄清 RichEdit20A 是干什么的？原来它是一个大型文本编辑软件，操作十分复杂，这里是大才小用，只用它来显示数据，并不对数据编辑。对 RichEdit20A 的操作主要是靠 SendMessageA 发送消息。它操作前，一定要确定光标位，确定光标位消息中的 lParam 参数指向一个 CHARRANGE 结构(2 个 dword)，这两个值相等，其值是光标位置；若不等则表示选择一个区间。  
       清楚了，GetWindowTextLengthA 获取屏幕上字符长度，第 1 个 SendMessageA 应该将光标定于尾部，好让第 2 个 SendMessageA 向它添加字符。那么第 1 个 SendMessageA 的参数 13F7CC 就一定是指向 CHARRANGE 结构，从 ebp 的值知 13F7CC 是指向\[ebp-8\]，\[ebp-4\]即一个 CHARRANGE 结构，两个 dword 值应该相等且就是当前的字串长度。  
       这样，显然应该有：mov \[ebp-8\],eax；mov \[ebp-4\],eax。所以，代码应该如下：  
       call GetWindowTextLengthA  
       mov \[ebp-8\],eax  
       mov \[ebp-4\],eax  
       lea eax,\[ebp-8\]  
       push  eax         <---取 CHARRANGE 结构地址  
       push  0  
       push  437  
       push \[40300C\]  
       call SendMessageA  
  
       第 2 个 SendMessageA 一定是将 buffer 中的字串添加上去。它的参数 lParam 应该指向一个 buffer。堆栈中参数是   
  0013F798   0063DCE9  /CALL 到 SendMessageA  
  0013F79C   00E20202  |hWnd = 450292  
  0013F7A0   000000C2  |Message = EM\_REPLACESEL    <-- EM\_REPLACESEL=C2  
  0013F7A4   00000000  |wParam = 0  
  0013F7A8   0040222F  \\Text = "--------------------------------------  
      MSG 消息 EM\_REPLACESEL 也表示向 RichEdit20A 添加字符，参数 lParam 指向 40222F，是直接 push 40222F 还是由调用程序带来？查调用 401000 的呼叫，知道它有多个地方调用，它的参数一定得由调用程序带来。  
      这样，接着的代码是  
      push  \[ebp+8\]    <--唯一的带来参数   
      push  0  
      push  0C2        <-- 0C2=EM\_REPLACESEL  
      push  \[40300C\]  
      call  SendMessageA  
    
      (4)SendMessageA 没有返回值，最后还有乱码 46 6f 的原码是什么？观察程序，前面有 pushad，后面一定要有 popad，又汇编程序是堆栈自动平衡方式，最后一般都有 leave，最后这个 C2 0400 (retn 4)是否是原码呢？应该说是，Themida 一般不去惹 call ebx、jmp eax、retn 等代码，因为它的转跳是不定的，themida 操作起来很复杂。  
      至此，这段代码被还原了。有的地方说法可能很勉强(因为有原码参考，所以我说得很肯定)，用在其它地方可能就不正确了。不管怎么说，分析起来虽然很困难，但肯定比从 VM 虚拟代码中还原要容易得多了。  
      本方法有个致命的弱点是，一些与被虚拟的函数无关的代码，如 add eax,6，mov ecx,edi 等是永远找不回来的。好在是已经编译过的.exe 文件再用 Themida 加密，如果不提供原编译文件，则只能对 API 虚拟了，包含它附近的几个代码。有关 API 的参数代码相对简单一些，很少有前面说到的那些代码形式，退后一步说，即使有，且被漏掉了，但所有的 API 都被还原了，关系清楚了，虽然 dump 后的程序不能运行，但破解的欲望达到了！   
    
      四、可能出现的问题  
      1.如果你用 OD 打开文件后加载脚本，OD 象没有反应一样直接就运行到了窗口出现。  
        结论：你可能是第 1 次将脚本用到其它程序上而又忘记了将脚本中的 jmp Second 恢复过来，或者你忘记了存盘。    
      2.用脚本第 1 次弹出对话框没问题，第 2 次按要求修改也存了盘，但就是不出现任何结果。  
        结论：Themida 版本可能不是 1.8.5.5 版。或者“调试设置--异常页”的选项选得太多，特别是选择了“忽略特权指令或无效指令”。  
      3.脚本不能运行到底，Themida 弹出警告框后退出。  
        结论：“调试设置--异常页”的选项选得太多，特别是选择了“忽略单步异常”。  
    
        我说话很索，谢谢你耐心读完。   
    
 --------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2008 年 09 月 05 日 10:41:32

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || temp.rar \[附件请到论坛下载：[`bbs.pediy.com/showthread.php?t=72152`](http://bbs.pediy.com/showthread.php?t=72152%20) \]   
 |