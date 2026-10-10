

# Smartsearch

> 原文：[`www.pediy.com/kssd/pediy05/pediy50287.htm`](https://www.pediy.com/kssd/pediy05/pediy50287.htm)

刚吐血完成 diype 那个网聪，现在又来吐血写教程，写完我就要去医院输血了，大家记得免费捐点：）   
文章题目：我的 diype 历程之一（smartsearch 保存功能之添加）   
    作者：greenlemon(菩提！)\[FCG\]   
破解工具：太多。。。。   
破解目的：正在思考：）   
  
diype=Dancing In Your Program Empire(最新解释:)   
  
首先感谢 pll621 大哥为我们提供了 6 篇高质量的 diype 教程，还有许多前辈们为此作的研究，都给了我动力和帮助。diype 的确是一项很累人的工作，其间 upfeed 对我的指点功不可没，我谨以此篇文章献给所有帮助我的朋友们，当然也是对我劳动的总结，同时我也想以此作为给初学者的教程（但是初学者的概念很难界定，我也是初学者，很多人还称自己是菜鸟，虽然对我来说他们已经是肉鸟：），所以如果你觉得还看不懂那么一定是我文笔太差，而不是你水平不够）。   
  
言归正传，让我们磨刀霍霍向网聪（希望原作者不要看到这篇文章：）   
  
写在前面：这里我希望你已经具备了 win32asm 的基本知识，如果没有可以查看一下 win32asm 基础教程。   
          其次你应该对 winapi 有初步的了解，如果没有至少你应该有这个 winapi 帮助手册，这两样看雪里都有！   
  
网聪的这款软件注册机及破解方法我以在以前的文章中说过，但是在注册成功后，最为关键的保存功能居然仍然没有，用 dede 反编译后发现那里只有一个用来显示什么"付费用户才能用此项功能"的 nag 而没有其他的代码，说明作者没有在这个网络下载版里提供这项功能，这有两种可能，1。作者根本没在这个版里写代码，2。他写了但是只有付费后才会提供一个补丁修改为指向这段代码。（事实上在我修改程序时发现的确有一些代码没有使用，可能他有写在里面，但是由于我已经在自己添加了，而且去寻找正确的代码入口也很困难，所以就没在细究）下面我们来分析要手动添加代码所要做的工作：   
  
1。回忆一下传统的保存功能都是怎样的过程，首先在点击保存后，要打开一个对话框要你输入你要保存的文件名，这个可以用 api 里 getsavefilename 其次在你输入名称后点确定，会创建一个新文件，或者覆盖原来的文件，这个用 createfile 来实现，当然 createfile 只是创建文件，你还要将信息保存进去，所以还要用 writefile 来实现，最后别忘了关闭文件 closehandle。这就是保存文件所要做的，当然还有内存的操作，这个我后面再说。   
  
2。接下来看看我们要把什么保存进去，当然是我们搜索到的电邮地址，可以通过 dede 很清楚地找到存放地址的控件是叫做 stringgrid 的东西，现在我们还不清楚他是怎么保存的，也就是说究竟以何种数据结构保存搜索到的结果。这是我们需要分析的。   
  
3。我们要找到程序的空余部分，好添加代码，如果你觉得不够，还要自己加一个 section，或者在原来的 section 上增加一些。这个工作很好做，用 topo 就可以：）（peter 老大教的）   
  
好我们明确了目标，就可以开始了，从最简单的开始，3 最简单：），我们找找程序的空余部分，我用的是 4a9426 开始的空间，我开始以为不够，所以又添加了一个 section，从 4d4000 开始。（好像差不多刚好，没有仔细的统计，反正增加几百字节也无所谓，重要的是能把东西做出来：）   
  
完成了一步很开心吧？没有。。。太简单了？你还要怎样：）好进入第 2，分析一下 stringgrid 的运作方式，开始我很迷惑，因为跟踪后发现他存放的空间不连续，我以为那不是正确的地址，还有一块连续的地方存放，百思不解后在 upfeed 的提醒下，我自己用 delphi 写了一段 stringgrid 的代码，发现我的想法是对的，只不过因为这个程序用的是多线程，所以每个线程都拥有一些结果，线程内是连续的，线程之间就是不连续的分配起始地址了，看来有时候还要自己写代码来分析，要不会被迷惑的，来看看它是怎么存放的：   
  
004A12EA  ||MOV EAX,DWORD PTR DS:\[4ACC14\]   
004A12EF  ||PUSH EAX                        ; /Arg1 => 00BB3DB4 ASCII "sales@eyou.com"   
004A12F0  ||MOV ECX,DWORD PTR DS:\[4AD0C0\]    ; |   
004A12F6  ||XOR EDX,EDX                      ; |   
004A12F8  ||MOV EAX,DWORD PTR DS:\[EBX+2F8\]  ; |   
004A12FE  ||CALL smartsea.00468568          ; \\smartsea.00468568  //这个就是设置 stringgrid 元素值的，跟进去看看    
004A1303  ||INC DWORD PTR DS:\[4AD0C0\]   
  
00468568  /$>PUSH EBP   
00468569  |.>MOV EBP,ESP   
0046856B  |.>PUSH ECX   
0046856C  |.>PUSH EBX   
0046856D  |.>PUSH ESI   
0046856E  |.>PUSH EDI   
0046856F  |.>MOV DWORD PTR SS:\[EBP\-4\],ECX  //当你每次运行到这里就会发现在\[ebp+8\]这个堆栈里存放着你想要的搜索到的邮件地址    
00468572  |.>MOV ESI,EDX                      虽然它不是一个接一个的规律变化，原因刚才我讲了，但是我们可以把这些指针保存起来呀：）   
00468574  |.>MOV EBX,EAX   
00468576  |.>MOV EDX,DWORD PTR SS:\[EBP\-4\]   
00468579  |.>MOV EAX,EBX   
0046857B  |.>CALL smartsea.004684A4   
00468580  |.>MOV ECX,DWORD PTR SS:\[EBP+8\]   
00468583  |.>MOV EDX,ESI   
00468585  |.>MOV EDI,DWORD PTR DS:\[EAX\]   
00468587  |.>CALL DWORD PTR DS:\[EDI+20\]   
0046858A  |.>MOV CL,1   
0046858C  |.>MOV EDX,ESI   
0046858E  |.>MOV EAX,EBX   
00468590  |.>CALL smartsea.00468440   
00468595  |.>XOR ECX,ECX   
00468597  |.>MOV EDX,DWORD PTR SS:\[EBP\-4\]   
0046859A  |.>MOV EAX,EBX   
0046859C  |.>CALL smartsea.00468440   
004685A1  |.>MOV ECX,DWORD PTR SS:\[EBP\-4\]   
004685A4  |.>MOV EDX,ESI   
004685A6  |.>MOV EAX,EBX   
004685A8  |.>CALL smartsea.004683F4   
004685AD  |.>POP EDI   
004685AE  |.>POP ESI   
004685AF  |.>POP EBX   
004685B0  |.>POP ECX   
004685B1  |.>POP EBP   
004685B2  \\.>RETN 4   
  
  
现在我们来做这个保存，我原先的设想把这些指针保存到我开辟的新的 section 里，这样不用做内存操作，省去一些代码，后来想了一想，如果以我们搜索到 1000 个地址为例，因为指针是 dword，也就是说需要 4000 字节来存放这些指针，晕倒，所以看来不能偷懒，还是老老实实开一块内存来吧。   
  
对内存的操作我这样实现，首先 globalalloc 一块内存，然后 globallock，这样我们就可以保存进去，最后要记得 globalunlock，globalfree。（不懂就去查前面两个手册）那么什么时候申请这块内存呢？我是在 formcreate 的时候做的，你可以在其他时候，只要保证你后面要保存指针的时候有内存用。   
  
0049B134  .>PUSH EBP                            //formcreate 开始   
0049B135  .>MOV EBP,ESP   
0049B137  .>ADD ESP,-40   
0049B13A  .>PUSH EBX   
0049B13B  .>PUSH ESI   
0049B13C  .>PUSH EDI   
0049B13D  .>XOR ECX,ECX   
0049B13F  .>MOV DWORD PTR SS:\[EBP\-3C\],ECX   
0049B142  .>MOV DWORD PTR SS:\[EBP\-40\],ECX   
0049B145  .>MOV DWORD PTR SS:\[EBP\-34\],ECX   
0049B148  .>MOV DWORD PTR SS:\[EBP\-38\],ECX   
0049B14B  .>MOV DWORD PTR SS:\[EBP\-30\],ECX   
0049B14E  .>MOV DWORD PTR SS:\[EBP\-28\],ECX   
0049B151  .>MOV DWORD PTR SS:\[EBP\-2C\],ECX   
0049B154  .>MOV DWORD PTR SS:\[EBP\-24\],ECX   
0049B157  .>MOV EBX,EAX   
0049B159  .>MOV ESI,smartsea.004ACC08   
0049B15E  .>XOR EAX,EAX   
0049B160  .>PUSH EBP   
0049B161  .>PUSH smartsea.0049B7FB   
0049B166  .>PUSH DWORD PTR FS:\[EAX\]   
0049B169  .>MOV DWORD PTR FS:\[EAX\],ESP   
0049B16C  .>JMP smartsea.004D4000              //把 mov     eax, \[ebx+$0308\]改成这个，跳到我们的代码   
0049B171    >NOP   
0049B172  .>XOR EDX,EDX   
0049B174  .>CALL smartsea.0042D0F8   
  
为便于理解，给出 globalalloc,globallock 的原型   
  
HGLOBAL GlobalAlloc(   
  
    UINT uFlags,    //  要申请的内存的标志，也就是说这块内存是什么类型的   
    DWORD dwBytes     //  要申请的内存大小   
  );   
LPVOID GlobalLock(   
  
    HGLOBAL hMem     // 你要锁定的那块内存的指针   
  );   
       
004D4000  6>PUSH 0FFFFF                        //我们要这么大的内存：）                         
004D4005  6>PUSH 42                            //GHND，标志具体可以查手册   
004D4007  E>CALL <JMP.&kernel32.GlobalAlloc>    //这个函数的地址可以用 WIN32ASM 查看，我这里是 4069A0   
004D400C  A>MOV DWORD PTR DS:\[4D4540\],EAX      //把返回的地址指针保存起来，我存到了 4D4540   
004D4011  5>PUSH EAX                            //我们要锁定他   
004D4012  E>CALL <JMP.&kernel32.GlobalLock>    //这个函数地址 4069B8   
004D4017  A>MOV DWORD PTR DS:\[4D4544\],EAX      //把指针保存到 4D4544   
004D401C  8>MOV EAX,DWORD PTR DS:\[EBX+308\]      //这是原来的程序，要恢复他（这里他没有用到寄存器，所以我们不用保存他们得值）   
004D4022  -E>JMP smartsea.0049B172              //返回了   
  
有了内存，我们就可以保存了，这么大一块内存，我决定不仅仅保存指针了，把那些字符串全保存起来，以后更便于写入文件不是吗？   
  
就是刚才这段：   
  
00468568  $>PUSH EBP   
00468569  .>MOV EBP,ESP   
0046856B  .>PUSH ECX   
0046856C  .>PUSH EBX   
0046856D  .>PUSH ESI   
0046856E  .>PUSH EDI   
0046856F  .>JMP smartsea.004D4027        //这里改了，跳到我们的程序   
00468574  .>MOV EBX,EAX   
00468576  .>MOV EDX,DWORD PTR SS:\[EBP\-4\]   
00468579  .>MOV EAX,EBX   
  
这里我们也用到两个函数：一个是字符串拷贝，一个是获取字符串长度，本来我想用 lstrcat 这个字符串连接函数，但是程序里没有，要自己构造麻烦，所以自己来实现吧。原型如下   
  
LPTSTR lstrcpy(   
  
    LPTSTR lpString1,    //  目的空间指针   
    LPCTSTR lpString2     // 源字符串指针   
  );   
  
int lstrlen(   
  
    LPCTSTR lpString     // 要计算长度的字符串地址指针    
  );       
  
  
004D4027  PUSH EAX                      //发现他们有使用这些寄存器，而我们的操作会破坏他们的值，所以存起来先   
004D4028  PUSH ECX   
004D4029  PUSH EDX   
004D402A  MOV EBX,DWORD PTR SS:\[EBP+8\]  //记得我刚才说的这里存放着我们要的东西   
004D402D  CMP EBX,0                      //这个比较是因为，你会发现第一次的值是 0，可能是 stringgrid 的初始化，对我们没用，而且会出错    
004D4030  JE SHORT smartsea.004D406B    //如果是 0 就返回去不操作   
004D4032  PUSH EBX                      //源字符串指针，给 lstrcpy 准备的   
004D4033  MOV ESI,DWORD PTR DS:\[4D4544\]  //还记得吗，这就是我们申请的内存空间指针存放处   
004D4039  MOV EBX,DWORD PTR DS:\[4D4548\]  //这个地址用来存放一个变量，这个变量就是字符串的长度，这样，我们可以实现字符串连接   
004D403F  LEA EBX,DWORD PTR DS:\[EBX+ESI\] //上面两个的和就指向我们要存放的目的地址的指针   
004D4042  PUSH EBX                      //压进去给 lstecpy   
004D4043  CALL <JMP.&kernel32.lstrcpyA>  //这个函数地址是 4012F8    
004D4048  PUSH DWORD PTR DS:\[4D4544\]    //来测一下现在我们现在的字符串有多长    
004D404E  CALL <JMP.&kernel32.lstrlenA>  //地址是 401308   
004D4053  MOV DWORD PTR DS:\[4D4548\],EAX  //把返回的长度值存进我们的变量空间   
004D4058  MOV WORD PTR DS:\[EAX+ESI\],0A0D //加入回车换行，目的是在保存的文件里可以一行显示一个 EMAIL   
004D405E  MOV BYTE PTR DS:\[EAX+ESI+2\],0  //再加一个字符串结束标志 NULL   
004D4063  ADD EAX,2                      //由于我们加了两个字节，所以长度变化了   
004D4066  MOV DWORD PTR DS:\[4D4548\],EAX  //再次保存    
004D406B  POP EDX                        //可以返回了   
004D406C  POP ECX   
004D406D  POP EAX   
004D406E  MOV DWORD PTR SS:\[EBP\-4\],ECX  //这个都是源程序的东西，别忘了恢复    
004D4071  MOV ESI,EDX   
004D4073  JMP smartsea.00468574          //回去咯   
  
简单的解释一下，如果你觉得一头雾水的话，我们的目的是保存成这样的形式：   
  
green@sina.com/r/nlemon@sina.com/r/nupfeed@sina.com/r/n......   
这样我们在文件里才可以存成这样：   
green@sina.com   
lemon@sina.com   
upfeed@sina.com   
  
再回去理解一下，不难了吧。   
  
第二步算是搞定了，来搞最后的一步，经过了上面，发现原来也不难是吧？：）事实上这么简单的代码，我调了 n 次，改了 n 次，所以说。。。。不知道说什么好，可能我比较菜吧：）   
  
来看看保存按钮里有什么：   
  
0049E688  push    $00   
0049E68A  mov    cx, word ptr \[$49E6A0\]   
0049E691  mov    dl, $02   
  
\* Possible String Reference to: '付费注册用户才能保存电邮'   
|   
0049E693  mov    eax, $0049E6AC   
  
|   
0049E698  call    004510A4   
0049E69D  ret   
  
没有什么有价值的东西，全部改掉！   
  
这是改后的：利用到 messagebox 原型如下   
  
int MessageBox(   
  
    HWND hWnd,    //  父窗体的句柄   
    LPCTSTR lpText,    // 要显示的字符串指针   
    LPCTSTR lpCaption,    // 标题栏上字符串的指针    
    UINT uType     // 消息框的类型   
  );   
  
注意到需要父窗体句柄,所以要到程序里去找找,把断点下在 createwindowex 很快就可以找到，但是我原先在这里范了一个错误，我直接用的是内存里的地址，忘了这对每台机器都是变动的，现在修正了一下，我把它保存起来了！   
  
所以首先找到这里：   
  
0044A0DE  MOV EDX,EAX                              ;   
0044A0E0  MOV ECX,84CA0000                        ; |   
0044A0E5  MOV EAX,DWORD PTR DS:\[4AAAB8\]            ; |   
0044A0EA  CALL smartsea.0040730C                  ; 这个进去就是 creatwindowex，返回值是句柄存在 eax 里   
0044A0EF  JMP smartsea.004D4078                    //跳到我们的保存代码   
0044A0F4  NOP   
0044A0F5  CALL smartsea.00403B40   
  
004D4078  MOV DWORD PTR DS:\[4D4550\],EAX            //存到这   
004D407D  MOV DWORD PTR DS:\[EBX+24\],EAX            //源程序的东西恢复   
004D4080  LEA EAX,DWORD PTR DS:\[EBX+7C\]   
004D4083  JMP smartsea.0044A0F5   
  
0049E688  call    004A9426                  //到我们做一个函数那   
0049E68D  cmp    eax, +$00                //比较返回值，判断我们是保存还是取消   
0049E690  jz      0049E69C                  //取消的话是 0，就跳   
0049E692  mov    eax, $004D43E9            //这里存放着 success!    
0049E697  jmp    0049E6A1   
0049E69C  mov    eax, $004D43E0            //这里存着 faliure！，   
0049E6A1  push    $00                      //MB\_OK 型   
0049E6A3  push    $004D4370                //这里存着字符串。。我的标志：）   
0049E6A8  push    eax                      //看看是成功还是失败咯   
0049E6A9  push    dword ptr \[$4D4550\]      //这个是句柄，怎么来的？当然是跟踪来的，把断点下在 CREATEWINDOW 很快就可以找到   
  
\* Reference to: user32.MessageBoxA()   
|   
0049E6AF  call    00401288   
0049E6B4  ret   
  
来看看我们的函数（有点长，但很简单：）用到了这几个函数：   
  
BOOL GetSaveFileName(   
  
    LPOPENFILENAME lpofn     //  指向 OPENFILENAME 类型的结构指针   
  );   
  
这个就是结构体，参数很多，查手册吧   
typedef struct tagOFN { // ofn    
    DWORD        lStructSize;   
    HWND          hwndOwner;   
    HINSTANCE    hInstance;   
    LPCTSTR      lpstrFilter;   
    LPTSTR        lpstrCustomFilter;   
    DWORD        nMaxCustFilter;   
    DWORD        nFilterIndex;   
    LPTSTR        lpstrFile;   
    DWORD        nMaxFile;   
    LPTSTR        lpstrFileTitle;   
    DWORD        nMaxFileTitle;   
    LPCTSTR      lpstrInitialDir;   
    LPCTSTR      lpstrTitle;   
    DWORD        Flags;   
    WORD          nFileOffset;   
    WORD          nFileExtension;   
    LPCTSTR      lpstrDefExt;   
    DWORD        lCustData;   
    LPOFNHOOKPROC lpfnHook;   
    LPCTSTR      lpTemplateName;   
} OPENFILENAME;   
  
HANDLE CreateFile(   
  
    LPCTSTR lpFileName,    // 文件名指针   
    DWORD dwDesiredAccess,    //  授权的模式（读，写，还是都有）   
    DWORD dwShareMode,    // 共享模式    
    LPSECURITY\_ATTRIBUTES lpSecurityAttributes,    //  安全属性   
    DWORD dwCreationDistribution,    // 建立模式    
    DWORD dwFlagsAndAttributes,    // 文件属性   
    HANDLE hTemplateFile     // 临时文件    
  );   
BOOL WriteFile(   
  
    HANDLE hFile,    //  要写入的文件句柄，一般由 CREATEFILE 返回的值就是   
    LPCVOID lpBuffer,    //  要写入文件的缓冲区指针   
    DWORD nNumberOfBytesToWrite,    //  写入多少字节   
    LPDWORD lpNumberOfBytesWritten,    //  已写入的字节   
    LPOVERLAPPED lpOverlapped     // 重叠模式指针    
  );   
  
004A9426  push    ebp                            //保存寄存器的值，不知道用了哪些，都存吧保险：）    
004A9427  push    ebx   
004A9428  push    esi   
004A9429  push    edi   
004A942A  mov    dword ptr \[$4D4390\], $0000004C  //下面是结构体的一些参数，这是整个结构体的大小，存放到从 4D4390 开始的一段空间里    
004A9434  push    dword ptr \[$4D4550\]            //这个是窗体句柄，上面提到过   
004A943A  pop    dword ptr \[$4D4394\]            //存到第二个 DWORD   
004A9440  push    dword ptr \[$4AC4D8\]            //这是模块句柄，跟踪程序开始的 GETMODULEHANDLE 就可以找到   
004A9446  pop    dword ptr \[$4D4398\]            //存到第 3 个   
004A944C  mov    dword ptr \[$4D439C\], $004D4350 //这个是文件过滤的参数我们这里是 ALL FILES 和 TEXT  FILES 存在 4D4350 处可以用 16 进制编辑器预先写入   
004A9456  mov    dword ptr \[$4D43AC\], $004D4420  //文件名要存放的地方   
004A9460  mov    dword ptr \[$4D43A8\], $00000002  //缺省状态的过滤参数是 TEXTFILE   
004A946A  mov    dword ptr \[$4D43B0\], $00000104  //文件名最大长度不超过 260 字节   
004A9474  mov    dword ptr \[$4D43C4\], $00282806  //文件的标志，查手册   
004A947E  mov    dword ptr \[$4D43C0\], $004D4370  //标题栏上的字符串，还是我：）   
004A9488  push    $004D4390                      //结构体指针给 getsavefilename   
  
\* Reference to: comdlg32.GetSaveFileNameA()   
|   
004A948D  call    0044C534   
004A9492  cmp    eax, +$01                      //比较一下如果是取消就返回   
004A9495  jnz    004A94FA                          
004A9497  push    $00                            //createfile 的七个参数   
004A9499  push    $20   
004A949B  push    $02   
004A949D  push    $00   
004A949F  push    $03   
004A94A1  push    $C0000000   
004A94A6  push    $004D4420   
  
\* Reference to: kernel32.CreateFileA()   
|   
004A94AB  call    00401210   
004A94B0  mov    dword ptr \[$4D454C\], eax        //文件句柄存起来   
004A94B5  push    dword ptr \[$4D4544\]            //看看现在我们那个内存空间里有多长的字符串，好准备写入   
  
\* Reference to: kernel32.lstrlenA()   
|   
004A94BB  call    00401308   
004A94C0  push    $00                            //写入文件用的 5 个参数   
004A94C2  push    $004D4550   
004A94C7  push    eax   
004A94C8  push    dword ptr \[$4D4544\]   
004A94CE  push    dword ptr \[$4D454C\]   
  
\* Reference to: kernel32.WriteFile()   
|   
004A94D4  call    00401260   
004A94D9  push    dword ptr \[$4D4544\]   
  
  
|   
004A94DF  call    004069D0 我的 diype 历程之一（smartsearch 保存功能之添加）   
我的 diype 历程之一（smartsearch 保存功能之添加）   
我的 diype 历程之一（smartsearch 保存功能之添加）   
  
004A94E4  push    dword ptr \[$4D4540\]   
  
\* Reference to: kernel32.GlobalFree()   
|   
004A94EA  call    004069B0   
004A94EF  push    dword ptr \[$4D454C\]   
  
\* Reference to: kernel32.CloseHandle()   
|   
004A94F5  call    00401208   
004A94FA  pop    edi   
004A94FB  pop    esi   
004A94FC  pop    ebx   
004A94FD  pop    ebp   
004A94FE  ret   
  
后记：似乎看起来没做多少工作，不过我觉得很累了：）也许我的精力比较差把，希望能抛砖引玉。不足之处还望大家多指点！   
转载请注明出处，并保持完整性，谢谢！  4dd466d0e1e0cc123afbe504da2462ca