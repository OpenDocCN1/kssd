

# qq 反汇编日志 1

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-201.htm`](https://www.pediy.com/kssd/pediy08/pediy8-201.htm)

2006 年 6 月 15 日 9：30  
oo，前两天的反汇编记录下来的东西都丢了（昨天重做系统时，忘了），不过还好，丢的东西大部分都是我的曲折错误分析  
现在从新开始：（为写远程暴力破解 qq 密码程序而反汇编 qq）  
反汇编工具：OllyICE  
现在开始：  
用 OllyICE 加载 qq.exe，qq 登录对话框出现，随便输入密码：123，运行，一个错误对话框出现，  
错误  
输入密码与上次成功登录得密码不一致，  
是否到服务器验证？  
  
好，打开 OllyICE 的模块窗口，然后找到 user32 模块，打开右键菜单选中查看名称，OllyICE 的名称窗口出现了，这里的函数都是  
User32 模块中的，好，找到 MessageBoxA，打开右键菜单选中反汇编窗口中跟随，这样就到了 MessageBoxA 函数的入口点，按 f2 键设下  
端点，设好后点刚才 qq 跳出来的错误对话框上的否，qq 用户登录又出现了，再输入 123  
OllyICE 在我们刚才设置的 MessageBoxA 函数的入口点停了下来，察看堆栈窗口  
  
0012FC2C   60B5C8E7  /CALL 到 MessageBoxA 来自 MFC42.60B5C8E1  
  
说明调用是 MFC42 库中的 60b5c8e1 处的指令的前一条指令调用了 MessageBoxA，在 CALL 到 MessageBoxA 来自 MFC42.60B5C8E1 上  
右键，打开右键菜单，选中反汇编窗口跟随，就到 mfc42 地址空间的 60b5c8e7 指令位置（不同的电脑，可能值不一样）,上一条指令  
就是调用 MessageBoxA 函数的，如下所示  
  
60B5C8DC  |.  FF7424 10     push    dword ptr \[esp+10\]               ; |Text  
60B5C8E0  |.  51            push    ecx                              ; |hOwner  
60B5C8E1  |.  FF15 D0B5B960 call    \[<&USER32.MessageBoxA>\]          ; \\MessageBoxA  
60B5C8E7  |.  5E            pop     esi  
60B5C8E8  |.  C2 0C00       retn    0C  
  
在 60b5c8e7 pop esi 上设置端点，然后点运行,刚才的那个错误对话框又出现了，点否，好，现在程序停在了我们刚才设置端点的位  
置 605c8e7 处，现在在 MFC42 地址空间中，不管它，我们想到的位置是 qq 空间，按两下 f8 键，到了 00415c35 cmp eax,6 指令处，看一  
下 OllyICE 的标题栏 OllyICE - QQ.exe -\[cpu -主线程，模块 - qq\]  
好了这就是我们的目的地。  
  
反汇编就从这里开始，第一个分析的函数就是：包含 指令地址 00415c35 的函数  
第一步：找出 包含指令地址 00415c35 的 函数入口点  
        向上滚动反汇编窗口，找到 00415b55 地址  
  00415B52  \\.  C2 0400       retn    4  
  00415B55  /$  B8 5CD24C00   mov     eax, 004CD25C  
  00415B5A  |.  E8 41A10500   call    0046FCA0  
  00415B5F  |.  83EC 48       sub     esp, 48  
  
  OllyICE 分析出这个是函数的入口点，因为上面紧跟的是 retn 指令，不过一般的函数入口处：应该是  
  push  ebp  
  mov  ebp,esp  
  才对，如果是罗函数的话，经过优化编译的话就不是这个样子了，我们先在反汇编的是 qq，qq 的软件工程师水平应该是可以  
  的,有可能经过特殊处理，  
  
  004CD25C=004CD25C  
  eax=0012FCBC  
  本地调用来自 00414FE8, 00416FEE  
  这个交叉引用说明，的确是函数的入口点  
  
  这样函数入口点就确定了：004b5b55  
第二步：确定函数结束地址，按 OllyICE 的提示 结束地址应为：00415c9b retn 14，一个函数有可能有多个出口  
        我们来做进一步的鉴定，看看从 004b5b55 到 00415c9b 之间的指令有无交叉引用到 00415c9b 以后的指令  
  答：没有，因此 00415c9b 即为函数的结束地址，至此我们确定了函数的入口点和结束地址  
  004b5b55--00415c9b  
现在我们开始还原 函数 qq004b55 的源代码  
  
第三步：确定参数个数和参数类型  
  找到调用 004b5b55 函数 指令 ：00414fe8 ------------------------------------  
                                                                                  |  
00414FD6  |.  C645 FC 05    mov     byte ptr \[ebp-4\], 5                           |  
00414FDA  |.  E8 A1A60500   call    <jmp.&MFC42.#535\_CString::CString>            |  
00414FDF  |.  FF75 EC       push    dword ptr \[ebp-14\]                            |  
00414FE2  |.  8BCB          mov     ecx, ebx                                      |  
00414FE4  |.  C645 FC 05    mov     byte ptr \[ebp-4\], 5                           |  
00414FE8  |.  E8 680B0000   call    00415B55      ---------------------------------                     
00414FED  |.  FFD6          call    esi    
  
  00414FE2  |.  8BCB          mov     ecx, ebx --- ecx 应该是个寄存器参数，要不这条指令就是个垃圾指令  
  函数 004b5b55 可能有一个寄存器参数，说可能，是因为编译器生成的垃圾指令到处可见，要对 ecx 进一步鉴定，要  
  分析 004b5b55 函数的代码  
00415B55  /$  B8 5CD24C00   mov     eax, 004CD25C  
00415B5A  |.  E8 41A10500   call    0046FCA0  
00415B5F  |.  83EC 48       sub     esp, 48  
00415B62  |.  53            push    ebx  
00415B63  |.  56            push    esi  
00415B64  |.  33DB          xor     ebx, ebx  
00415B66  |.  895D FC       mov     \[ebp-4\], ebx  
00415B69  |.  895D F0       mov     \[ebp-10\], ebx  
00415B6C  |.  8B81 84000000 mov     eax, \[ecx+84\]  
  在函数的开始处，调用了一个函数 0046fca0,察看一下它的代码，看看和 ecx 有染没有  
0046FCA0  /$  6A FF         push    -1  
0046FCA2  |.  50            push    eax  
0046FCA3  |.  64:A1 0000000>mov     eax, fs:\[0\]  
0046FCA9  |.  50            push    eax  
0046FCAA  |.  8B4424 0C     mov     eax, \[esp+C\]  
0046FCAE  |.  64:8925 00000>mov     fs:\[0\], esp  
0046FCB5  |.  896C24 0C     mov     \[esp+C\], ebp  
0046FCB9  |.  8D6C24 0C     lea     ebp, \[esp+C\]  
0046FCBD  |.  50            push    eax  
0046FCBE  \\.  C3            retn  
  哈哈，这个函数挺简练的，和 ecx 无关，这就好，再看看 00415b6c 地址 以上的指令都与 ecx 无关，而在 00415b6c 处引用了  
  ecx，因此断定 ecx 就是一个寄存器参数，不错，good  
  下面来看一下 004b5b55 函数 有多少个堆栈参数  
  从函数结束地址 00415C9B  \\.  C2 1400       retn    14  
  可知 堆栈参数个数为 14h / 4 = 5  
  到此我们确定了参数的个数：一个寄存器参数 + 5 个堆栈参数 = 6（说明函数采用的是 fastcall 调用方式）  
  给他们编号分别为：arg1,arg2,arg3,arg4,arg5,arg6  
    
  下面我们来确定参数类型  
  在指令中识别参数  
  函数 004b5b55 每有标准的函数头 即 push ebp ; mov ebp,esp,如果是采用优化编译的话，参数应该用 esp 寄存器来寻址  
  可在函数过程中只在分配局部变量时，用了一次 esp  
00415B5F  |.  83EC 48       sub     esp, 48  
  其他的地方都没有，令人奇怪的是到处都是用 ebp 寄存器寻址的，对了，函数入口点，调用了一个函数：0046fca0  
  看看它都实现了什么功能  
    
0046FCA0  /$  6A FF         push    -1  
执行指令后堆栈数据  
相对 esp 的地址    数据  
esp      -1  
esp+4      ret addr  
0046FCA2  |.  50            push    eax  
执行指令后堆栈数据  
相对 esp 的地址    数据  
esp      eax (通过寄存器 eax 传递过来的参数进栈）  
esp+4      -1  
esp+8      ret addr  
0046FCA3  |.  64:A1 0000000>mov     eax, fs:\[0\]    ；eax 指向 seh （结构化异常处理）  
0046FCA9  |.  50            push    eax  
执行指令后堆栈数据  
相对 esp 的地址    数据  
esp      eax (seh 指针）  
esp+4      eax (通过寄存器 eax 传递过来的参数进栈）  
esp+8      -1  
esp+c      ret addr  
0046FCAA  |.  8B4424 0C     mov     eax, \[esp+C\]  ；把 ret addr 传递给 eax  
执行指令后堆栈数据  
相对 esp 的地址    数据  
esp      eax (seh 指针）  
esp+4      eax (通过寄存器 eax 传递过来的参数进栈）  
esp+8      -1  
esp+c      ret addr  
0046FCAE  |.  64:8925 00000>mov     fs:\[0\], esp    ；此时 esp 指向一个 EXCEPTION\_REGISTRATION 结构  
              ；表明同过寄存器 eax 传递给函数的是异常回调函数地址  
              ；-1 则是 seh 的附加数据  
执行指令后堆栈数据  
相对 esp 的地址    数据  
esp      eax (seh 指针）<-------------------------------fs:\[0\]  
esp+4      eax (通过寄存器 eax 传递过来的参数进栈）  
esp+8      -1  
esp+c      ret addr  
  
0046FCB5  |.  896C24 0C     mov     \[esp+C\], ebp  ；保存 ebp 到 原来 ret addr 所占堆栈位置  
执行指令后堆栈数据  
相对 esp 的地址    数据  
esp      eax (seh 指针）<-------------------------------fs:\[0\]  
esp+4      eax (通过寄存器 eax 传递过来的参数进栈）  
esp+8      -1  
esp+c      ebp  ----- 注意发生变化了  
  
0046FCB9  |.  8D6C24 0C     lea     ebp, \[esp+C\]  ；ebp 指向 保存 ebp 的位置（也即原来 ret addr 的位置)  
执行指令后堆栈数据  
相对 esp 的地址    数据  
esp      eax (seh 指针）<-------------------------------fs:\[0\]  
esp+4      eax (通过寄存器 eax 传递过来的参数进栈）  
esp+8      -1  
esp+c      ebp  ----- 注意发生变化了<------------------------ ebp  
0046FCBD  |.  50            push    eax  
执行指令后堆栈数据  
相对 ebp 的地址    相对 esp 的地址    数据  
ebp-10      esp      eax(函数的返回地址)  
ebp-c      esp+4      eax (seh 指针）<-------------------------------fs:\[0\]  
ebp-8      esp+8      eax (通过寄存器 eax 传递过来的参数进栈）  
ebp-4      esp+c      -1  
ebp      esp+10      ebp  ----- 注意发生变化了<------------------------ ebp  
0046FCBE  \\.  C3            retn  
执行指令后堆栈数据  
相对 ebp 的地址    相对 esp 的地址    数据  
ebp-c      esp      eax (seh 指针）<-------------------------------fs:\[0\]  
ebp-8      esp+4      eax (通过寄存器 eax 传递过来的参数进栈）  
ebp-4      esp+8      -1  
ebp      esp+c      ebp  ----- 注意发生变化了<------------------------ ebp  
  
因为 函数 0046fca0 没有堆栈参数，所以 ebp+4 指向的是它的父函数即 004b5b55 结束时要返回的地址  
  
相对 ebp 的地址    相对 esp 的地址    数据  
ebp-c      esp      eax (seh 指针）<-------------------------------fs:\[0\]  
ebp-8      esp+4      eax (通过寄存器 eax 传递过来的参数进栈）  
ebp-4      esp+8      -1  
ebp      esp+c      ebp  ----- 注意发生变化了<------------------------ ebp  
ebp +4      esp+10      004b5b55 函数 结束时要返回的地址  
  
综上所述：0046fca0 实现的功能为：《1》注册异常回调函数  
         《2》实现了和 push  ebp  
                 mov  ebp,esp  
          查不多的功能，调用 0046fca0 函数的函数的第一个堆栈参数地址为 ebp +8  
          这和 push ebp ;move ebp,esp 是一样的，但是局部变量的寻址就不一样了  
          调用 0046fca0 函数的函数局部变量是从 ebp-d 开始的，而不是从 ebp-4 开始的  
          搞定……^\_^  
现在我们就可以很轻松的识别出参数了,开始吧  
识别参数类型  
首先我用 32 位汇编语言来实现 00415b55，我给他起个名字叫 \_lastStep 吧,我把寄存器传参也改成堆栈方式  
  
\_LastStep  proc  \_arg1,\_arg2,\_arg3,\_arg4,\_arg5,\_arg6  
    mov  eax,004cd25c  
    call  0046fca0  
    sub  esp,48h  
    push  ebx  
    push  esi  
    xor  ebx,ebx  
    mov  \[ebp-4\],ebx  ;\[ebp-4\] 是 seh 附加数据的地址，原来值为-1,现在要把它置零了  
    mov  @dwVar1,ebx  
    mov  ecx,\_arg1  
    mov  eax,\[ecx+84\]  ；可知\_arg1 是一个指针  
    lea  edx,@dwVar1  
    push  edx  
    push  004e7460  
    mov  ecx,\[eax\]  
    push  eax  
    mov  byte ptr\[ebp-4\],1  ;\[ebp-4\] 是 seh 附加数据的地址  
    call  \[ecx+1c\]    ;表明\_arg1 是一个函数指针的指针的指针  
    test  eax,eax  
    jnz  lable1  
    mov  eax,@dwVar1  
    lea  edx,@dwVar2  
    push  edx  
    push  004e8940    ；ascii "ewh.db"  
    mov  ecx,\[eax\]  
    push  eax  
    call  \[ecx+14\]  
    test  eax,eax  
    je  label2  
  lable1:  
    mov  eax,\_arg4    ;可知\_arg4 是一个 int\*  
    mov  dword ptr\[eax\],2  
    jmp  lable3  
  lable2:  
    mov  eax,\_arg3  
    push  edi  
    mov  edx,@dwVar1  
    push  1  
    mov  ecx,\[eax-8\]  
    pop  esi  
    mov  edi,\[edx\]  
    push  esi  
    push  eax  
    push  ecx  
    push  ebx  
    push  dword ptr \_arg2  
    push  edx  
    call  \[edi+1c\]  
    test  eax,eax  
    pop  edi  
    je  lable4    
    cmp  \_arg6,esi  
    je  lable5  
    mov  eax,\_arg4  
    mov  \[eax\],ebx  ;可知\_arg4 是个指针  
    jmp  lable3  
  lable5:  
    lea  ecx,\_arg6  
    call  CString::CString  
    lea  ecx,\_arg2  
    mov  byte ptr \[ebp-4\],2  
    call  CString::CString  
    mov  esi,BasicCtrDll.BasicLoadStr  
    lea  eax,\_arg6  
    push  281  
    push  eax  
    mov  byte prt\[ebp-4\],3  
    call  esi  
    lea  eax,\_arg2  
    push  28d  
    push  eax  
    call  esi  
    add  esp,10  
    lea  ecx,\[ebp-54\]  ;ebp-54 指向一个结构  
    call  CWNd::CWnd  
    push  114  
    lea  ecx,\[ebp-54\]  
    push  dword ptr \_arg2  
    mov  byte prt\[ebp-4\],4  
    push  dword ptr \_arg6  
    call  CWnd::MessageBoxA  
    cmp  eax,6  
    mov  eax,\_arg4  
    jnz  Lable6  
    mov  dword prt\[eax\],2  
    jmp  lable7  
  lable6:  
    mov  \[eax\],ebx  
  lable7:  
    lea  ecx,\[ebp-54\]  
    mov  byte ptr\[ebp-4\],3  
    call  CWnd::~Cwnd  
    lea  ecx,\_arg2  
    mov  byte ptr\[ebp-4\],2  
    call  CString::~CString  
    lea  ecx,\_arg6  
    mov  byte ptr\[ebp-4\],1  
    call  CString::~Cstring  
    jmp  lable3  
  lable4:  
    mov  eax,\_arg4  
    mov  \[eax\],esi  
  lable3:  
    mov  eax,@dwVar1  
    mov  \[ebp-4\],bl  
    cmp  eax,ebx  
    je  lable8  
    mov  ecx,\[eax\]  
    push  eax  
    call  \[ecx+8\]  
  lable8:  
    or  dwword ptr\[ebp-4\],0ffffffffh  
    lea  ecx,\_arg3  
    call  CString::~CString  
    mov  ecx,\[ebp-c\]  ; pre seh handler 指针  
    pop  esi  
    pop  ebx  
    mov  fs:\[0\],ecx  
    leave  
    retn  14  
\_LastSetp  enp  
  
今天就到这里，明天继续  
2006-6-15 13:20  
  
  
欢迎访问我的博客：http://dreamcometotrue.blog.ccidnet.com 

识别函数中的参数和局部变量  
经过上面的分析我们已经知道在 qq00405b55 函数中参数和局部变量都是用 ebp 寄存器来表示的,现在我们来找出函数中的参数，和局部变量  
首先：我们把函数中使用 ebp 寄存器寻找的变量找出来，删除重复的并按从低地址到高地址排序，得到如下结果：  
ebp-54h  
ebp-14h  
ebp-10h  
ebp-ch  
ebp-4h  
ebp+8h  
ebp+ch  
ebp+10h  
ebp+18h  
我们知道堆栈参数是：从 ebp+8 开始向高地址方向的，局部变量是从 ebp-d 开始向低地址方向的  
现在给他们分组：  
堆栈参数有：ebp+8h,ebp+ch,ebp+10h,ebp+18h,堆栈参数名我们规定从低地址开始编号\_arg1,\_arg2,\_arg3,....,所以就有 ：  
ebp+8h = \_arg1  
ebp+ch = \_arg2  
ebp+10h = \_arg3  
ebp+18h = \_arg4 少了一个 怎么没有 ebp+14h 呢，\_arg3 占了 8 个字节的地盘？？？先不管它，呆会分析参数类型的时候在看  
局部变量有：ebp-10h,ebp-14h,ebp-54h,局部变量我们规定从高地址开始编号@var1,@var2,@var3,.....所以有:  
ebp-10h = @var1  因为局部变量是从 ebp-d 开始的所以可知这个局部变量的大小为(10h-d)+1 4 byte  
ebp-14h = @var2 同理：这个紧挨着@var1 可知他也为(14h-11h)+1 4 byte   
ebp-54h = @var3 这是最后一个变量，它的大小为：(54h-15h)+1 = 40h byte  
还有几个孤儿，没有组的，他们是 ebp-ch,和 ebp-4h，日至 1 中已经知道他们分别是 seh 指针和 seh 的附加数据  
第二步：我们将参数名、局部变量名替换到反汇编代码中去，还有跳转地址我们也用相应的标号替换掉就得到了  
  
  mov     eax, 004CD25C  
  call    0046FCA0  
  sub     esp, 48  
  push    ebx  
  push    esi  
  xor     ebx, ebx  
  mov     \[ebp-4\], ebx  
  mov     @var1, ebx  
  mov     eax, \[ecx+84\]  
  lea     edx, @var1  
  push    edx  
  push    004E7460  
  mov     ecx, \[eax\]  
  push    eax  
  mov     byte ptr \[ebp-4\], 1  
  call    \[ecx+1C\]  
  test    eax, eax  
  jnz     lable1    ;short 00415B9F  
  mov     eax, @var1  
  lea     edx, @var2  
  push    edx  
  push    004E8940  
  mov     ecx, \[eax\]  
  push    eax  
  call    \[ecx+14\]  
  test    eax, eax  
  je      lable2    ;short 00415BAD  
lable1:  
  mov     eax, \_arg3  
  mov     dword ptr \[eax\], 2  
  jmp     lable3    ;00415C72  
lable2:  
  mov     eax, \_arg2  
  push    edi  
  mov     edx, @var1  
  push    1  
  mov     ecx, \[eax-8\]  
  pop     esi  
  mov     edi, \[edx\]  
  push    esi  
  push    eax  
  push    ecx  
  push    ebx  
  push    dword ptr \_arg1  
  push    edx  
  call    \[edi+1C\]  
  test    eax, eax  
  pop     edi  
  je      lable4    ;00415C6D  
  cmp     \_arg4, esi  
  je      lable5    ;short 00415BDF  
  mov     eax, \_arg3  
  mov     \[eax\], ebx  
  jmp     lable3    ;00415C72  
lable5:  
  lea     ecx, \_arg4  
  call    <jmp.&MFC42.#540\_CString::CStrin>  
  lea     ecx, \_arg1  
  mov     byte ptr \[ebp-4\], 2  
  call    <jmp.&MFC42.#540\_CString::CStrin>  
  mov     esi, \[<&BasicCtrlDll.BasicLoadSt>;  BasicCtr.BasicLoadStr  
  lea     eax, \_arg4  
  push    281  
  push    eax  
  mov     byte ptr \[ebp-4\], 3  
  call    esi                              ;  <&BasicCtrlDll.BasicLoadStr>  
  lea     eax, \_arg1  
  push    28D  
  push    eax  
  call    esi  
  add     esp, 10  
  lea     ecx, @var3  
  call    <jmp.&MFC42.#567\_CWnd::CWnd>  
  push    114  
  lea     ecx, @var3  
  push    dword ptr \_arg1  
  mov     byte ptr \[ebp-4\], 4  
  push    dword ptr \_arg4  
  call    <jmp.&MFC42.#4224\_CWnd::MessageB>  
  cmp     eax, 6  
  mov     eax, \_arg3  
  jnz     label6        ;short 00415C45  
  mov     dword ptr \[eax\], 2               ;  \[eax\]可能保存的是某个标志，2 代表本地登陆失败，到服务器上去验证去  
  jmp     lable7        ;short 00415C47  
lable6:  
  mov     \[eax\], ebx  
lable7:  
  lea     ecx, @var3  
  mov     byte ptr \[ebp-4\], 3              ;  \[12fcac\]  
  call    <jmp.&MFC42.#818\_CWnd::~CWnd>  
  lea     ecx, \_arg1  
  mov     byte ptr \[ebp-4\], 2  
  call    <jmp.&MFC42.#800\_CString::~CStri>  
  lea     ecx, \_arg4  
  mov     byte ptr \[ebp-4\], 1  
  call    <jmp.&MFC42.#800\_CString::~CStri>  
  jmp     lable3        ;short 00415C72  
lable4:  
  mov     eax, \_arg3  
  mov     \[eax\], esi  
lable3:  
  mov     eax, \_arg3  
  mov     \[ebp-4\], bl  
  cmp     eax, ebx  
  je      lable8        ;short 00415C82  
  mov     ecx, \[eax\]  
  push    eax  
  call    \[ecx+8\]  
lable8:  
  or      dword ptr \[ebp-4\], FFFFFFFF  
  lea     ecx, \_arg2  
  call    <jmp.&MFC42.#800\_CString::~CStri>  
  mov     ecx, \[ebp-C\]                     ;  seh 指针  
  pop     esi  
  pop     ebx  
  mov     fs:\[0\], ecx  
  leave  
  retn    14                               ;  父函数传递了 5 个参数  
  
靠，终于整完了  
  
下一步：去掉编译器自己添加的或与函数功能无关的，异常处理东东  
删掉  
mov     eax, 004CD25C  
  call    0046FCA0  
  注册异常处理函数，打开堆栈页面  
  sub     esp, 48    ；分配局部变量空间  
删掉  
  mov     ecx, \[ebp-C\]                     ;  seh 指针  
删掉  
  mov     fs:\[0\], ecx  
  leave  
  retn    14    
  将注册的异常处理函数删除掉，堆栈平衡  
  
  
剩下的就是裸函数了  
  
  push    ebx  
  push    esi  
  xor     ebx, ebx  
  mov     \[ebp-4\], ebx  
  mov     @var1, ebx  
  mov     eax, \[ecx+84\]  
  lea     edx, @var1  
  push    edx  
  push    004E7460  
  mov     ecx, \[eax\]  
  push    eax  
  mov     byte ptr \[ebp-4\], 1  
  call    \[ecx+1C\]  
  test    eax, eax  
  jnz     lable1  
  mov     eax, @var1  
  lea     edx, @var2  
  push    edx  
  push    004E8940  
  mov     ecx, \[eax\]  
  push    eax  
  call    \[ecx+14\]  
  test    eax, eax  
  je      lable2  
lable1:  
  mov     eax, \_arg3  
  mov     dword ptr \[eax\], 2  
  jmp     lable3  
lable2:  
  mov     eax, \_arg2  
  push    edi  
  mov     edx, @var1  
  push    1  
  mov     ecx, \[eax-8\]  
  pop     esi  
  mov     edi, \[edx\]  
  push    esi  
  push    eax  
  push    ecx  
  push    ebx  
  push    dword ptr \_arg1  
  push    edx  
  call    \[edi+1C\]  
  test    eax, eax  
  pop     edi  
  je      lable4  
  cmp     \_arg4, esi  
  je      lable5  
  mov     eax, \_arg3  
  mov     \[eax\], ebx  
  jmp     lable3  
lable5:  
  lea     ecx, \_arg4  
  call    <jmp.&MFC42.#540\_CString::CStrin>  
  lea     ecx, \_arg1  
  mov     byte ptr \[ebp-4\], 2  
  call    <jmp.&MFC42.#540\_CString::CStrin>  
  mov     esi, \[<&BasicCtrlDll.BasicLoadSt>;  BasicCtr.BasicLoadStr  
  lea     eax, \_arg4  
  push    281  
  push    eax  
  mov     byte ptr \[ebp-4\], 3  
  call    esi                              ;  <&BasicCtrlDll.BasicLoadStr>  
  lea     eax, \_arg1  
  push    28D  
  push    eax  
  call    esi  
  add     esp, 10  
  lea     ecx, @var3  
  call    <jmp.&MFC42.#567\_CWnd::CWnd>  
  push    114  
  lea     ecx, @var3  
  push    dword ptr \_arg1  
  mov     byte ptr \[ebp-4\], 4  
  push    dword ptr \_arg4  
  call    <jmp.&MFC42.#4224\_CWnd::MessageB>  
  cmp     eax, 6  
  mov     eax, \_arg3  
  jnz     label6  
  mov     dword ptr \[eax\], 2  
  jmp     lable7  
lable6:  
  mov     \[eax\], ebx  
lable7:  
  lea     ecx, @var3  
  mov     byte ptr \[ebp-4\], 3  
  call    <jmp.&MFC42.#818\_CWnd::~CWnd>  
  lea     ecx, \_arg1  
  mov     byte ptr \[ebp-4\], 2  
  call    <jmp.&MFC42.#800\_CString::~CStri>  
  lea     ecx, \_arg4  
  mov     byte ptr \[ebp-4\], 1  
  call    <jmp.&MFC42.#800\_CString::~CStri>  
  jmp     lable3  
lable4:  
  mov     eax, \_arg3  
  mov     \[eax\], esi  
lable3:  
  mov     eax, \_arg3  
  mov     \[ebp-4\], bl  
  cmp     eax, ebx  
  je      lable8  
  mov     ecx, \[eax\]  
  push    eax  
  call    \[ecx+8\]  
lable8:  
  or      dword ptr \[ebp-4\], FFFFFFFF  
  lea     ecx, \_arg2  
  call    <jmp.&MFC42.#800\_CString::~CStri>  
  pop     esi  
  pop     ebx  
  
让函数光着身子不好，现在给它穿上衣服吧  
我先给它穿上 win32asm 这套衣服，以后再给她穿 c++这套衣服,给这个过程起个名字：叫 \_TempName 吧，现在还不知道她是做什么用的,知道了,再换  
寄存器参数我用 \_regArg1,\_regArg2,...表示  
  
\_TempName  Proc  \_regArg1,\_arg1,\_arg2,\_arg3,\_arg?,\_arg4  (因为不知道倒数第二个为什么没有在函数中引用过，所以先用？表示)  
  push    ebx  
  push    esi  
  xor     ebx, ebx  
  mov     \[ebp-4\], ebx  
  mov     @var1, ebx  
  mov     eax, \[ecx+84\]  
  lea     edx, @var1  
  push    edx  
  push    004E7460  
  mov     ecx, \[eax\]  
  push    eax  
  mov     byte ptr \[ebp-4\], 1  
  call    \[ecx+1C\]  
  test    eax, eax  
  jnz     lable1  
  mov     eax, @var1  
  lea     edx, @var2  
  push    edx  
  push    004E8940  
  mov     ecx, \[eax\]  
  push    eax  
  call    \[ecx+14\]  
  test    eax, eax  
  je      lable2  
lable1:  
  mov     eax, \_arg3  
  mov     dword ptr \[eax\], 2  
  jmp     lable3  
lable2:  
  mov     eax, \_arg2  
  push    edi  
  mov     edx, @var1  
  push    1  
  mov     ecx, \[eax-8\]  
  pop     esi  
  mov     edi, \[edx\]  
  push    esi  
  push    eax  
  push    ecx  
  push    ebx  
  push    dword ptr \_arg1  
  push    edx  
  call    \[edi+1C\]  
  test    eax, eax  
  pop     edi  
  je      lable4  
  cmp     \_arg4, esi  
  je      lable5  
  mov     eax, \_arg3  
  mov     \[eax\], ebx  
  jmp     lable3  
lable5:  
  lea     ecx, \_arg4  
  call    <jmp.&MFC42.#540\_CString::CStrin>  
  lea     ecx, \_arg1  
  mov     byte ptr \[ebp-4\], 2  
  call    <jmp.&MFC42.#540\_CString::CStrin>  
  mov     esi, \[<&BasicCtrlDll.BasicLoadSt>;  BasicCtr.BasicLoadStr  
  lea     eax, \_arg4  
  push    281  
  push    eax  
  mov     byte ptr \[ebp-4\], 3  
  call    esi                              ;  <&BasicCtrlDll.BasicLoadStr>  
  lea     eax, \_arg1  
  push    28D  
  push    eax  
  call    esi  
  add     esp, 10  
  lea     ecx, @var3  
  call    <jmp.&MFC42.#567\_CWnd::CWnd>  
  push    114  
  lea     ecx, @var3  
  push    dword ptr \_arg1  
  mov     byte ptr \[ebp-4\], 4  
  push    dword ptr \_arg4  
  call    <jmp.&MFC42.#4224\_CWnd::MessageB>  
  cmp     eax, 6  
  mov     eax, \_arg3  
  jnz     label6  
  mov     dword ptr \[eax\], 2  
  jmp     lable7  
lable6:  
  mov     \[eax\], ebx  
lable7:  
  lea     ecx, @var3  
  mov     byte ptr \[ebp-4\], 3  
  call    <jmp.&MFC42.#818\_CWnd::~CWnd>  
  lea     ecx, \_arg1  
  mov     byte ptr \[ebp-4\], 2  
  call    <jmp.&MFC42.#800\_CString::~CStri>  
  lea     ecx, \_arg4  
  mov     byte ptr \[ebp-4\], 1  
  call    <jmp.&MFC42.#800\_CString::~CStri>  
  jmp     lable3  
lable4:  
  mov     eax, \_arg3  
  mov     \[eax\], esi  
lable3:  
  mov     eax, \_arg3  
  mov     \[ebp-4\], bl  
  cmp     eax, ebx  
  je      lable8  
  mov     ecx, \[eax\]  
  push    eax  
  call    \[ecx+8\]  
lable8:  
  or      dword ptr \[ebp-4\], FFFFFFFF  
  lea     ecx, \_arg2  
  call    <jmp.&MFC42.#800\_CString::~CStri>  
  pop     esi  
  pop     ebx  
  ret  
\_TempName  endp  
  
下一步分析 qq00415b55 调用的函数  
  
第一个  call    \[ecx+1C\] oo 是个间接调用指令，那来动态跟踪以下调用的是什么？00410ee2  
  
00410EE2   .  B8 CCC84C00   mov     eax, 004CC8CC  
00410EE7   .  E8 B4ED0500   call    0046FCA0  
00410EEC   .  51            push    ecx  
00410EED   .  51            push    ecx  
00410EEE   .  8B45 08       mov     eax, \[ebp+8\]  
00410EF1   .  56            push    esi  
00410EF2   .  8D4D EC       lea     ecx, \[ebp-14\]  
00410EF5   .  FF70 D8       push    dword ptr \[eax-28\]  
00410EF8   .  8D70 BC       lea     esi, \[eax-44\]  
00410EFB   .  E8 94E80500   call    <jmp.&MFC42.#6467\_AFX\_MAINTAIN\_S>  
00410F00   .  FF75 10       push    dword ptr \[ebp+10\]               ; /Arg2  
00410F03   .  8365 FC 00    and     dword ptr \[ebp-4\], 0             ; |  
00410F07   .  8BCE          mov     ecx, esi                         ; |  
00410F09   .  FF75 0C       push    dword ptr \[ebp+C\]                ; |Arg1  
00410F0C   .  E8 71040000   call    00411382                         ; \\QQ.00411382  
00410F11   .  8B4D F0       mov     ecx, \[ebp-10\]  
00410F14   .  8B55 EC       mov     edx, \[ebp-14\]  
00410F17   .  834D FC FF    or      dword ptr \[ebp-4\], FFFFFFFF  
00410F1B   .  5E            pop     esi  
00410F1C   .  8951 04       mov     \[ecx+4\], edx  
00410F1F   .  8B4D F4       mov     ecx, \[ebp-C\]  
00410F22   .  64:890D 00000>mov     fs:\[0\], ecx  
00410F29   .  C9            leave  
00410F2A   .  C2 0C00       retn    0C  
  
分析过程和 分析 qq00415b55 一样  
经分析可知 函数没有寄存器参数，c/4=3 个堆栈参数，这里要注意的是  
00410EEC   .  51            push    ecx  
00410EED   .  51            push    ecx  
这两条指令是分配局部变量空间的，因为函数中有两个局部变量，这是编译器快速分配局部变量而使得花招  
00410ee2 的函数原型为：  
  
00410ee2  proto  \_arg1,\_arg2,\_arg3  
  
下一个 qq00415b55 调用的函数是 call \[ecx+14\] = 0044c62b  
0044C62B   .  B8 84424D00   mov     eax, 004D4284  
0044C630   .  E8 6B360200   call    0046FCA0  
0044C635   .  51            push    ecx  
0044C636   .  51            push    ecx  
0044C637   .  8B45 08       mov     eax, \[ebp+8\]  
0044C63A   .  56            push    esi  
0044C63B   .  8D4D EC       lea     ecx, \[ebp-14\]  
0044C63E   .  FF70 FC       push    dword ptr \[eax-4\]  
0044C641   .  8D70 E0       lea     esi, \[eax-20\]  
0044C644   .  E8 4B310200   call    <jmp.&MFC42.#6467\_AFX\_MAINTAIN\_STATE2::AFX\_MAINTAIN\_STATE2>  
0044C649   .  FF75 10       push    dword ptr \[ebp+10\]  
0044C64C   .  8B06          mov     eax, \[esi\]  
0044C64E   .  8365 FC 00    and     dword ptr \[ebp-4\], 0  
0044C652   .  FF75 0C       push    dword ptr \[ebp+C\]  
0044C655   .  56            push    esi  
0044C656   .  FF50 64       call    \[eax+64\]  
0044C659   .  8B4D F0       mov     ecx, \[ebp-10\]  
0044C65C   .  8B55 EC       mov     edx, \[ebp-14\]  
0044C65F   .  834D FC FF    or      dword ptr \[ebp-4\], FFFFFFFF  
0044C663   .  5E            pop     esi  
0044C664   .  8951 04       mov     \[ecx+4\], edx  
0044C667   .  8B4D F4       mov     ecx, \[ebp-C\]  
0044C66A   .  64:890D 00000>mov     fs:\[0\], ecx  
0044C671   .  C9            leave  
0044C672   .  C2 0C00       retn    0C  
看起来好像和上个函数长得很像  
  
函数原型为：  
  
0044c62b  proto  \_arg1,\_arg2,\_arg3  
  
next function: 0044c6c5  
函数原型为：  
0044c6c5  proto  \_arg1,\_arg2,\_arg3,\_arg4,\_arg5,\_arg6  
下一个函数是 BasicCtr.BasicLoadStr  
60092057 >/$  55            push    ebp  
60092058  |.  8BEC          mov     ebp, esp  
6009205A  |.  56            push    esi  
6009205B  |.  57            push    edi  
6009205C  |.  E8 5FCD0000   call    <jmp.&MFC42.#1168\_AfxGetModuleState>  
60092061  |.  8B70 0C       mov     esi, \[eax+C\]  
60092064  |.  8B3D BC8B0B60 mov     edi, \[600B8BBC\]                                                 ;  BasicCtr.60090000  
6009206A  |.  E8 51CD0000   call    <jmp.&MFC42.#1168\_AfxGetModuleState>  
6009206F  |.  FF75 0C       push    dword ptr \[ebp+C\]  
60092072  |.  8B4D 08       mov     ecx, \[ebp+8\]  
60092075  |.  8978 0C       mov     \[eax+C\], edi  
60092078  |.  E8 91CD0000   call    <jmp.&MFC42.#4160\_CString::LoadStringA>  
6009207D  |.  E8 3ECD0000   call    <jmp.&MFC42.#1168\_AfxGetModuleState>  
60092082  |.  8970 0C       mov     \[eax+C\], esi  
60092085  |.  5F            pop     edi  
60092086  |.  5E            pop     esi  
60092087  |.  5D            pop     ebp  
60092088  \\.  C3            retn  
  
可算见到一个正常函数，函数有两个堆栈参数参数，没有寄存器参数，这个函数调用方式属于\_cdel 方式，平很堆栈交给了  
qq00415b55，qq00415b55 调用完这个函数后并没有做平衡堆栈，而是在第二次调用完这个函数后一起做了堆栈平衡，这可能是  
优化编译的结果，这样节省一条指令  
我们从 basicCtr.BasicLoadStr 中也可以找出参数，\[ebp+8\],\[ebp+c\]这两个  
  
在 qq00415b55 中平衡堆栈代码  
00415C06  |.  FFD6          call    esi                                                             ;  BasicCtr.BasicLoadStr; <&BasicCtrlDll.BasicLoadStr>  
00415C08  |.  8D45 08       lea     eax, \[ebp+8\]                                                    ;  \_arg2  
00415C0B  |.  68 8D020000   push    28D  
00415C10  |.  50            push    eax  
00415C11  |.  FFD6          call    esi  
00415C13  |.  83C4 10       add     esp, 10 ---------------------平衡了两次调用,一个函数的参数个数=10h/2/4=2 个  
  
所以这个函数的原型应为：  
BasicCtr.BasicLoadStr  proto  \_arg1,\_arg2  
  
下一个，也是 qq00415b55 调用 qq 自定义函数中的最后一个 0044cf97 函数只有一个堆栈参数，没有寄存器参数，函数原型为：  
0044cf97  proc  \_arg1  
  
经过上面的分析，qq00415b55 的 win32asm 源代码应为：  
BasicCtr.BasicLoadStr  proto  \_arg1,\_arg2  
0044cf97  proto  \_arg1  
  
\_TempName  Proc  \_regArg1,\_arg1,\_arg2,\_arg3,\_arg?,\_arg4  (因为不知道倒数第二个为什么没有在函数中引用过，所以先用？表示)  
  local  @dwVar1,@dwVar2  
  local  @wnd:CWnd  (这个以后再解释)  
  push    ebx  
  push    esi  
  xor     ebx, ebx  
  mov     \[ebp-4\], ebx  
  mov     @var1, ebx  
  mov     eax, \[ecx+84\]  
  mov     ecx, \[eax\]  
  mov     byte ptr \[ebp-4\], 1  
  invoke  \[ecx+1c\],eax,004e7460,addr @var1  (这里不管 addr 会覆盖掉 eax 的值)  
  test    eax, eax  
  jnz     lable1  
  mov     eax, @var1  
  mov     ecx, \[eax\]  
  invoke  \[ecx+14\],eax,004e8940,addr @var2(这里不管 addr 会覆盖掉 eax 的值)  
  test    eax, eax  
  je      lable2  
lable1:  
  mov     eax, \_arg3  
  mov     dword ptr \[eax\], 2  
  jmp     lable3  
lable2:  
  mov     eax, \_arg2  
  push    edi  
  mov     edx, @var1  
  push    1  
  mov     ecx, \[eax-8\]  
  pop     esi  
  mov     edi, \[edx\]  
  invoke  \[edi+1c\],edx,\_arg1,ebx,ecx,eax,esi  
  test    eax, eax  
  pop     edi  
  je      lable4  
  cmp     \_arg4, esi  
  je      lable5  
  mov     eax, \_arg3  
  mov     \[eax\], ebx  
  jmp     lable3  
lable5:  
  invoke  CString::CString,\_arg4  
  mov     byte ptr \[ebp-4\], 2  
  invoke  CString::CString>,\_arg1  
  mov     byte ptr \[ebp-4\], 3  
  invoke  BasicCtrlDll.BasicLoadStr,addr \_arg4,281h  
  invoke  BasicCtrlDll.BasicLoadStr,addr \_arg1,28dh  
  invoke  CWnd::CWnd,addr @wnd  
  mov     byte ptr \[ebp-4\], 4  
  invoke  CWnd::MessageBox,addr @wnd,\_arg4,\_arg1,MB\_YESNO or MB\_DEFBUTTON2 or MB\_ICONHAND (114h)  ;这个就是登录失败的时候的标题和信息  
  cmp     eax, 6  
  mov     eax, \_arg3  
  jnz     label6  
  mov     dword ptr \[eax\], 2  
  jmp     lable7  
lable6:  
  mov     \[eax\], ebx  
lable7:  
  mov     byte ptr \[ebp-4\], 3  
  invoke  CWnd::~CWnd,addr @wnd  
  mov     byte ptr \[ebp-4\], 2  
  invoke  CString::~CString,addr \_arg1  
  mov     byte ptr \[ebp-4\], 1  
  invoke  CString::~CString,\_arg4  
  jmp     lable3  
lable4:  
  mov     eax, \_arg3  
  mov     \[eax\], esi  
lable3:  
  mov     eax, \_arg3  
  mov     \[ebp-4\], bl  
  cmp     eax, ebx  
  je      lable8  
  mov     ecx, \[eax\]  
  invoke  \[ecx+8\],eax  
lable8:  
  or      dword ptr \[ebp-4\], FFFFFFFF  
  invoke  CString::~CString,\_arg2  
  pop     esi  
  pop     ebx  
  ret  
\_TempName  endp  
  
有点累了，今天就到这里了，明天继续.......  
  
2006-06-17 20：00 

2006-6-18 8:28  
识别参数类型和局部变量类型  
  xor     ebx, ebx  
  mov     \[ebp-4\], ebx  
  mov     @var1, ebx  
由这三天指令中的上下两条指令：@var1 应该是个整数类型的 改名为@intVar1  
  mov  ecx,\_regArg1  
  mov     eax, \[ecx+84\]  
  mov     ecx, \[eax\]  
  mov     byte ptr \[ebp-4\], 1  
  invoke  \[ecx+1c\],eax,004e7460,addr @var1  (这里不管 addr 会覆盖掉 eax 的值)  
第一次反汇编，最少面那条指令在日至 2 中漏掉了  
由上面可知：\[ecx+1c\]=\[\[eax\]+1c\]=\[\[\[\_regArg1+1c\]\]\]=\*\*\*(\_regArg1+1c) 可知 \_regArg1 是函数指针的指针的指针  
  
@var2 的类型现在还不能判断，80% 他和@intVar1 类型一样，只有跟进\[ecx+14\]函数中才知道  
  
0044C62B   .  B8 84424D00   mov     eax, 004D4284  
0044C630   .  E8 6B360200   call    0046FCA0  
0044C635   .  51            push    ecx  
0044C636   .  51            push    ecx  
0044C637   .  8B45 08       mov     eax, \[ebp+8\]  
0044C63A   .  56            push    esi  
0044C63B   .  8D4D EC       lea     ecx, \[ebp-14\]  
0044C63E   .  FF70 FC       push    dword ptr \[eax-4\]  
0044C641   .  8D70 E0       lea     esi, \[eax-20\]  
0044C644   .  E8 4B310200   call    <jmp.&MFC42.#6467\_AFX\_MAINTAIN\_S>  
0044C649   .  FF75 10       push    dword ptr \[ebp+10\]  
0044C64C   .  8B06          mov     eax, \[esi\]  
0044C64E   .  8365 FC 00    and     dword ptr \[ebp-4\], 0  
0044C652   .  FF75 0C       push    dword ptr \[ebp+C\]  
0044C655   .  56            push    esi  
0044C656   .  FF50 64       call    \[eax+64\]  
0044C659   .  8B4D F0       mov     ecx, \[ebp-10\]  
0044C65C   .  8B55 EC       mov     edx, \[ebp-14\]  
0044C65F   .  834D FC FF    or      dword ptr \[ebp-4\], FFFFFFFF  
0044C663   .  5E            pop     esi  
0044C664   .  8951 04       mov     \[ecx+4\], edx  
0044C667   .  8B4D F4       mov     ecx, \[ebp-C\]  
0044C66A   .  64:890D 00000>mov     fs:\[0\], ecx  
0044C671   .  C9            leave  
0044C672   .  C2 0C00       retn    0C  
  
@var2 对应着第三个参数即\[ebp+10\]   
0044C649   .  FF75 10       push    dword ptr \[ebp+10\]  
0044C64C   .  8B06          mov     eax, \[esi\]  
0044C64E   .  8365 FC 00    and     dword ptr \[ebp-4\], 0  
0044C652   .  FF75 0C       push    dword ptr \[ebp+C\]  
0044C655   .  56            push    esi  
0044C656   .  FF50 64       call    \[eax+64\]  
看来还要再跟进去  
0044D23C   .  B8 B4454D00   mov     eax, 004D45B4  
0044D241   .  E8 5A2A0200   call    0046FCA0  
0044D246   .  83EC 10       sub     esp, 10  
0044D249   .  8B45 10       mov     eax, \[ebp+10\]  
0044D24C   .  53            push    ebx  
0044D24D   .  56            push    esi  
0044D24E   .  57            push    edi  
0044D24F   .  C700 01000000 mov     dword ptr \[eax\], 1  
0044D255   .  E8 32240200   call    <jmp.&MFC42.#1154\_AfxGetAppModul>  
0044D25A   .  8B40 04       mov     eax, \[eax+4\]  
0044D25D   .  33DB          xor     ebx, ebx  
0044D25F   .  8B80 D4000000 mov     eax, \[eax+D4\]  
0044D265   .  3BC3          cmp     eax, ebx  
0044D267   .  0F84 4C010000 je      0044D3B9  
0044D26D   .  895D EC       mov     \[ebp-14\], ebx  
0044D270   .  8B08          mov     ecx, \[eax\]  
0044D272   .  8D55 EC       lea     edx, \[ebp-14\]  
0044D275   .  52            push    edx  
0044D276   .  68 E06F4E00   push    004E6FE0  
0044D27B   .  50            push    eax  
0044D27C   .  895D FC       mov     \[ebp-4\], ebx  
0044D27F   .  FF51 1C       call    \[ecx+1C\]  
0044D282   .  85C0          test    eax, eax  
0044D284   .  0F85 1E010000 jnz     0044D3A8  
0044D28A   .  895D F0       mov     \[ebp-10\], ebx  
0044D28D   .  8B45 EC       mov     eax, \[ebp-14\]  
0044D290   .  8D55 F0       lea     edx, \[ebp-10\]  
0044D293   .  52            push    edx  
0044D294   .  53            push    ebx  
0044D295   .  FF75 0C       push    dword ptr \[ebp+C\]  
0044D298   .  8B08          mov     ecx, \[eax\]  
0044D29A   .  C645 FC 01    mov     byte ptr \[ebp-4\], 1  
0044D29E   .  50            push    eax  
0044D29F   .  FF51 20       call    \[ecx+20\]  
0044D2A2   .  85C0          test    eax, eax  
0044D2A4   .  0F85 EE000000 jnz     0044D398  
0044D2AA   .  8B45 F0       mov     eax, \[ebp-10\]  
0044D2AD   .  8D55 E8       lea     edx, \[ebp-18\]  
0044D2B0   .  895D E8       mov     \[ebp-18\], ebx  
0044D2B3   .  52            push    edx  
0044D2B4   .  8B08          mov     ecx, \[eax\]  
0044D2B6   .  50            push    eax  
0044D2B7   .  FF51 1C       call    \[ecx+1C\]  
0044D2BA   .  85C0          test    eax, eax  
0044D2BC   .  0F85 D6000000 jnz     0044D398  
0044D2C2   .  8B45 E8       mov     eax, \[ebp-18\]  
0044D2C5   .  40            inc     eax  
0044D2C6   .  50            push    eax  
0044D2C7   .  E8 5C240200   call    <jmp.&MFC42.#823\_operator new>  
0044D2CC   .  8BF0          mov     esi, eax  
0044D2CE   .  8B45 E8       mov     eax, \[ebp-18\]  
0044D2D1   .  40            inc     eax  
0044D2D2   .  8975 0C       mov     \[ebp+C\], esi  
0044D2D5   .  50            push    eax                              ; /n  
0044D2D6   .  53            push    ebx                              ; |c  
0044D2D7   .  56            push    esi                              ; |s  
0044D2D8   .  E8 B5290200   call    <jmp.&MSVCRT.memset>             ; \\memset  
0044D2DD   .  8B45 F0       mov     eax, \[ebp-10\]  
0044D2E0   .  83C4 10       add     esp, 10  
0044D2E3   .  8D55 E4       lea     edx, \[ebp-1C\]  
0044D2E6   .  8B08          mov     ecx, \[eax\]  
0044D2E8   .  52            push    edx  
0044D2E9   .  56            push    esi  
0044D2EA   .  FF75 E8       push    dword ptr \[ebp-18\]  
0044D2ED   .  50            push    eax  
0044D2EE   .  FF51 0C       call    \[ecx+C\]  
0044D2F1   .  85C0          test    eax, eax  
0044D2F3   .  74 0C         je      short 0044D301  
0044D2F5   .  56            push    esi  
0044D2F6   .  E8 8B230200   call    <jmp.&MFC42.#825\_operator delete>  
0044D2FB   .  59            pop     ecx  
0044D2FC   .  E9 97000000   jmp     0044D398  
0044D301   >  8B7D 08       mov     edi, \[ebp+8\]  
0044D304   .  53            push    ebx                              ; /Arg1  
0044D305   .  8D77 70       lea     esi, \[edi+70\]                    ; |  
0044D308   .  8BCE          mov     ecx, esi                         ; |  
0044D30A   .  E8 A49C0500   call    004A6FB3                         ; \\QQ.004A6FB3  
0044D30F   .  8B06          mov     eax, \[esi\]  
0044D311   .  3BC3          cmp     eax, ebx  
0044D313   .  74 08         je      short 0044D31D  
0044D315   .  8B08          mov     ecx, \[eax\]  
0044D317   .  50            push    eax  
0044D318   .  FF51 08       call    \[ecx+8\]  
0044D31B   .  891E          mov     \[esi\], ebx  
0044D31D   >  56            push    esi  
0044D31E   .  E8 136F0700   call    004C4236  
0044D323   .  8B06          mov     eax, \[esi\]  
0044D325   .  59            pop     ecx  
0044D326   .  FF75 0C       push    dword ptr \[ebp+C\]  
0044D329   .  8B08          mov     ecx, \[eax\]  
0044D32B   .  FF75 E4       push    dword ptr \[ebp-1C\]  
0044D32E   .  50            push    eax  
0044D32F   .  FF51 78       call    \[ecx+78\]  
0044D332   .  85C0          test    eax, eax  
0044D334   .  74 10         je      short 0044D346  
0044D336   .  395D 0C       cmp     \[ebp+C\], ebx  
0044D339   .  74 58         je      short 0044D393  
0044D33B   .  FF75 0C       push    dword ptr \[ebp+C\]  
0044D33E   .  E8 43230200   call    <jmp.&MFC42.#825\_operator delete>  
0044D343   .  59            pop     ecx  
0044D344   .  EB 4D         jmp     short 0044D393  
0044D346   >  395D 0C       cmp     \[ebp+C\], ebx  
0044D349   .  74 09         je      short 0044D354  
0044D34B   .  FF75 0C       push    dword ptr \[ebp+C\]  
0044D34E   .  E8 33230200   call    <jmp.&MFC42.#825\_operator delete>  
0044D353   .  59            pop     ecx  
0044D354   >  8B06          mov     eax, \[esi\]  
0044D356   .  8D57 58       lea     edx, \[edi+58\]  
0044D359   .  52            push    edx  
0044D35A   .  68 9CD75200   push    0052D79C                         ;  ASCII "AST"  
0044D35F   .  8B08          mov     ecx, \[eax\]  
0044D361   .  50            push    eax  
0044D362   .  FF51 34       call    \[ecx+34\]  
0044D365   .  85C0          test    eax, eax  
0044D367   .  75 2A         jnz     short 0044D393  
0044D369   .  8B06          mov     eax, \[esi\]  
0044D36B   .  8D57 5C       lea     edx, \[edi+5C\]  
0044D36E   .  52            push    edx  
0044D36F   .  68 98D75200   push    0052D798                         ;  ASCII "EMH"  
0044D374   .  8B08          mov     ecx, \[eax\]  
0044D376   .  50            push    eax  
0044D377   .  FF51 44       call    \[ecx+44\]  
0044D37A   .  85C0          test    eax, eax  
0044D37C   .  75 15         jnz     short 0044D393  
0044D37E   .  8B36          mov     esi, \[esi\]  
0044D380   .  83C7 6C       add     edi, 6C  
0044D383   .  57            push    edi  
0044D384   .  68 94D75200   push    0052D794                         ;  ASCII "UIN"  
0044D389   .  8B06          mov     eax, \[esi\]  
0044D38B   .  56            push    esi  
0044D38C   .  FF50 34       call    \[eax+34\]  
0044D38F   .  85C0          test    eax, eax  
0044D391   .  74 2D         je      short 0044D3C0  
0044D393   >  8B45 10       mov     eax, \[ebp+10\]  
0044D396   .  8918          mov     \[eax\], ebx  
0044D398   >  8B45 F0       mov     eax, \[ebp-10\]  
0044D39B   .  885D FC       mov     \[ebp-4\], bl  
0044D39E   .  3BC3          cmp     eax, ebx  
0044D3A0   .  74 06         je      short 0044D3A8  
0044D3A2   .  8B08          mov     ecx, \[eax\]  
0044D3A4   .  50            push    eax  
0044D3A5   .  FF51 08       call    \[ecx+8\]  
0044D3A8   >  8B45 EC       mov     eax, \[ebp-14\]  
0044D3AB   .  834D FC FF    or      dword ptr \[ebp-4\], FFFFFFFF  
0044D3AF   .  3BC3          cmp     eax, ebx  
0044D3B1   .  74 06         je      short 0044D3B9  
0044D3B3   .  8B08          mov     ecx, \[eax\]  
0044D3B5   .  50            push    eax  
0044D3B6   .  FF51 08       call    \[ecx+8\]  
0044D3B9   >  B8 05400080   mov     eax, 80004005  
0044D3BE   .  EB 23         jmp     short 0044D3E3  
0044D3C0   >  8B45 F0       mov     eax, \[ebp-10\]  
0044D3C3   .  885D FC       mov     \[ebp-4\], bl  
0044D3C6   .  3BC3          cmp     eax, ebx  
0044D3C8   .  74 06         je      short 0044D3D0  
0044D3CA   .  8B08          mov     ecx, \[eax\]  
0044D3CC   .  50            push    eax  
0044D3CD   .  FF51 08       call    \[ecx+8\]  
0044D3D0   >  8B45 EC       mov     eax, \[ebp-14\]  
0044D3D3   .  834D FC FF    or      dword ptr \[ebp-4\], FFFFFFFF  
0044D3D7   .  3BC3          cmp     eax, ebx  
0044D3D9   .  74 06         je      short 0044D3E1  
0044D3DB   .  8B08          mov     ecx, \[eax\]  
0044D3DD   .  50            push    eax  
0044D3DE   .  FF51 08       call    \[ecx+8\]  
0044D3E1   >  33C0          xor     eax, eax  
0044D3E3   >  8B4D F4       mov     ecx, \[ebp-C\]  
0044D3E6   .  5F            pop     edi  
0044D3E7   .  5E            pop     esi  
0044D3E8   .  5B            pop     ebx  
0044D3E9   .  64:890D 00000>mov     fs:\[0\], ecx  
0044D3F0   .  C9            leave  
0044D3F1   .  C2 0C00       retn    0C  
  
函数有 3 个堆栈参数，没有寄存器参数  
我们要找的是第三个参数即\[ebp+10\]  
  
0044D249   .  8B45 10       mov     eax, \[ebp+10\]  
0044D24C   .  53            push    ebx  
0044D24D   .  56            push    esi  
0044D24E   .  57            push    edi  
0044D24F   .  C700 01000000 mov     dword ptr \[eax\], 1  
  
由上面的指令可知: \[ebp+10\] 是一个整形数的地址,即 我们要确定的@var2 是个整形类型，改名为@intVar2  
返回\_TempName 继续分析  
lable1:  
  mov     eax, \_arg3  
  mov     dword ptr \[eax\], 2  
可知：\_arg3 是个整形指针  改名为\_pIntArg3  
  mov     eax, \_arg2  
  push    edi  
  mov     edx, @var1  
  push    1  
  mov     ecx, \[eax-8\]  
可知：\_arg2 是一个指针    改名为\_pArg2  
  invoke  CString::CString,\_arg4  => \_arg4 是个字符串指针  改名为 \_lpszArg4  
  invoke  CString::CString,\_arg1 => \_arg1 是个字符串指针 改名为 \_lpszArg1  
  invoke  CWnd::CWnd,addr @var3  
可知：@var3 是个隐含的 this 指针，函数中原来的指令为  
00415C16  |.  8D4D AC       lea     ecx, \[ebp-54\]  
00415C19  |.  E8 569A0500   call    <jmp.&MFC42.#567\_CWnd::CWnd>  
visual c++ 编译器使用 ecx 用作 this 指针   
所以 @var3 是个 CWnd 的实例 改名为 @wnd  
  
invoke  CWnd::MessageBox,addr @wnd,\_arg4,\_arg1,MB\_YESNO or MB\_DEFBUTTON2 or MB\_ICONHAND (114h)  ;这个就是登录失败的时候的标题和信息  
由上面可知：\_arg4 为：内容 "输入密码与上次成功登陆的密码不一致，(回车符）是否到服务器验证?"  
      \_arg1 为：标题 "错误"  
invoke  CString::~CString,\_arg2 => \_arg2 是个字符串指针 改名为\_lpszArg2  
看看现在都确定了那些  
\_regArg1 是个函数指针的指针的指针  改名为:\_lpRegArg1 吧 ，他终归是个指针  
\_arg1 是个字符串指针       改名为：\_lpszArg1  
\_arg2 是个字符串指针       改名为\_lpszArg2  
\_arg3 是个整形指针      改名为\_pIntArg3  
\_arg4 是个字符串指针      改名为 \_lpszArg4  
  
@var1 是个整形数      改名为 @intVar1  
@var2 是个整形数      改名为 @intVar2  
@var3 是个 CWnd 对象      改名为 @wndVar3  
  
函数现在可以写成：  
  
\_TempName  Proc  \_lpRegArg1,\_lpszArg1,\_lpszArg2,\_pIntArg3,\_arg?,\_lpszArg4  (因为不知道倒数第二个为什么没有在函数中引用过，所以先用？表示)  
  local  @intVar1,@intVar2  
  local  @wndVar3:CWnd  
  push    ebx  
  push    esi  
  xor     ebx, ebx  
  mov     \[ebp-4\], ebx  
  mov     @intVar1, ebx  
  mov  ecx,\_lpRegArg1  
  mov     eax, \[ecx+84\]  
  mov     ecx, \[eax\]  
  mov     byte ptr \[ebp-4\], 1  
  invoke  \[ecx+1c\],eax,004e7460,addr @intVar1  (这里不管 addr 会覆盖掉 eax 的值)  
  test    eax, eax  
  jnz     lable1  
  mov     eax, @intVar1  
  mov     ecx, \[eax\]  
  invoke  \[ecx+14\],eax,004e8940,addr @intVar2(这里不管 addr 会覆盖掉 eax 的值)  
  test    eax, eax  
  je      lable2  
lable1:  
  mov     eax, \_lpIntArg3  
  mov     dword ptr \[eax\], 2  
  jmp     lable3  
lable2:  
  mov     eax, \_arg2  
  push    edi  
  mov     edx, @intVar1  
  push    1  
  mov     ecx, \[eax-8\]  
  pop     esi  
  mov     edi, \[edx\]  
  invoke  \[edi+1c\],edx,\_arg1,ebx,ecx,eax,esi  
  test    eax, eax  
  pop     edi  
  je      lable4  
  cmp     \_lpszArg4, esi  
  je      lable5  
  mov     eax, \_lpIntArg3  
  mov     \[eax\], ebx  
  jmp     lable3  
lable5:  
  invoke  CString::CString,\_lpszArg4  
  mov     byte ptr \[ebp-4\], 2  
  invoke  CString::CString>,\_lpszArg1  
  mov     byte ptr \[ebp-4\], 3  
  invoke  BasicCtrlDll.BasicLoadStr,addr \_lpszArg4,281h  
  invoke  BasicCtrlDll.BasicLoadStr,addr \_lpszArg1,28dh  
  invoke  CWnd::CWnd,addr @wnd  
  mov     byte ptr \[ebp-4\], 4  
  invoke  CWnd::MessageBox,addr @wnd,\_lpszArg4,\_lpszArg1,MB\_YESNO or MB\_DEFBUTTON2 or MB\_ICONHAND (114h)  ;这个就是登录失败的时候的标题和信息  
  cmp     eax, 6  
  mov     eax, \_lpIntArg3  
  jnz     label6  
  mov     dword ptr \[eax\], 2  
  jmp     lable7  
lable6:  
  mov     \[eax\], ebx  
lable7:  
  mov     byte ptr \[ebp-4\], 3  
  invoke  CWnd::~CWnd,addr @wndVar3  
  mov     byte ptr \[ebp-4\], 2  
  invoke  CString::~CString,addr \_lpszArg1  
  mov     byte ptr \[ebp-4\], 1  
  invoke  CString::~CString,\_lpszArg4  
  jmp     lable3  
lable4:  
  mov     eax, \_lpIntArg3  
  mov     \[eax\], esi  
lable3:  
  mov     eax, \_lpIntArg3  
  mov     \[ebp-4\], bl  
  cmp     eax, ebx  
  je      lable8  
  mov     ecx, \[eax\]  
  invoke  \[ecx+8\],eax  
lable8:  
  or      dword ptr \[ebp-4\], FFFFFFFF  
  invoke  CString::~CString,\_lpszArg2  
  pop     esi  
  pop     ebx  
  ret  
\_TempName  endp  
  
今天到这里了，进度有点慢.....  
  
2006-06-18 10:00 

2006-06-21 7:10  
找出\_TempName 函数的返回值，这里的返回值，只考虑高级语言中的 return 语句中的值，不考虑其他引用返回值、等等情况  
\_TempName 函数在结尾时，没有显示对 eax，edx 进行赋值，可以判断\_TempName 函数的返回值为空，但也有可能\_TempName 返回她调用函数的值  
比如：return func(arg1,..),为了进一步确定，看看调用\_TempName 函数的函数有没有在 call 00415b55 指令后紧跟着使用 eax、edx 的值  
00414FE8  |.  E8 680B0000   call    00415B55                           
00414FED  |.  FFD6          call    esi                              ;  Kernel32.GetTickCount  
  
结果表明：没有使用 eax，edx ，所以\_TempName 的返回值应为 void  
  
c++表示的函数圆形应为：  
void  TempName(void\* regArg1,char\* lpszArg1,char\* lpszArg2,int\* lpIntArg3,?,char\* lpszArg4);  
  
下面将汇编语言指令还原成高级语言的表达式  
  
从函数调用语句或比较指令向上倒序还原，例如：  
  push    ebx  
  push    esi  
  xor     ebx, ebx  
  mov     \[ebp-4\], ebx  
  mov     @intVar1, ebx  
  mov  ecx,\_lpRegArg1  
  mov     eax, \[ecx+84\]  
  mov     ecx, \[eax\]  
  mov     byte ptr \[ebp-4\], 1  
  invoke  \[ecx+1c\],eax,004e7460,addr @intVar1  (这里不管 addr 会覆盖掉 eax 的值)  
  test    eax, eax  
就从 invoke \[ecx+1c\],eax,004e7460,addr @intVar1 开始，用局部变量，和参数替换掉寄存器，这样我们就去掉了临时的寄存器  
变量得到了高级语言表达式。(在高级语言中，我把局部变量名前面的@去掉，参数名前面的\_ 去掉)  
ecx=\[eax\]=\[\[ecx+84\]\]=\[\[lpRegArg1+84h\]\]=\*(\*(lpRegArg1+84h))  
eax=\[ecx+84\]=\[lpRegArg1+84\]=\*(lpRegArg1+0x84)  
invoke \[ecx+1c\],eax,004e7460,addr @intVar1 用 c++ 表示为：  
\*(\*(\*(lpRegArg1+0x84))+0x1c)(\*(lpRegArg1+0x84),0x004e7460,&intVar1);  
删掉  
  mov  ecx,\_lpRegArg1  
  mov     eax, \[ecx+84\]  
  mov     ecx, \[eax\]  
  这几个临时的寄存器变量  
上面还剩的指令有：  
  push    ebx  
  push    esi  
  ；这是保存寄存器值，与高级语言没什么关系删掉  
  xor     ebx, ebx  
  mov     \[ebp-4\], ebx  
  mov     @intVar1, ebx  
  mov     byte ptr \[ebp-4\], 1  
可以写成：  
  \[ebp-4\]=0;  
  intVar1=0;  
        byte ptr\[ebp-4\]=1;  
  
  以上的指令还原为：  
  
  \[ebp-4\]=0;  
  intVar1=0;  
        byte ptr\[ebp-4\]=1;  
  \*(\*(\*(lpRegArg1+0x84))+0x1c)(\*(lpRegArg1+0x84),0x004e7460,&intVar1);  
  
  跳转指令使用条件表达式和 goto 替换  
  
依次类推函数可写为  
void TempName(void\* lpRegArg1,char\* lpszArg1,char\* lpszArg2,int\* lpIntArg3,?,lpszArg4)  
{  
  int intVar1,intVar2;  
  CWnd wndVar3;  
  mov  \[ebp-4\],0  
  intVar1=0;  
  \*(\*(\*(lpRegArg1+0x84))+0x1c)(\*(lpRegArg1+0x84),0x004e7460,&intVar1);  
  if(eax) goto lable1  
  \*(\*(intVar1)+0x14)(intVar1,0x004e8940,&intVar2);  
  // 看来 var1 不是个整形变量，而是个指针 改名为 lpVar1,函数要改写了  
  intVar1=0;这条语句可能是空指针初始化    
}  
  
void TempName(void\* lpRegArg1,char\* lpszArg1,char\* lpszArg2,int\* lpIntArg3,?,lpszArg4)  
{  
  void\* lpVar1=NULL;  
  int intVar2;  
  CWnd wndVar3;  
  mov  \[ebp-4\],0  
  \*(\*(\*(lpRegArg1+0x84))+0x1c)(\*(lpRegArg1+0x84),0x004e7460,&lpVar1);  
  if(eax) goto lable1  
  \*(\*(lpVar1)+0x14)(intVar1,0x004e8940,&intVar2);  
  if(!eax) goto lable2;  
lable1:  
  \*lpIntArg3=2;  
  goto lable3;  
lable2:  
  \*(\*lpVar1+0x1c)(lpVar1,0,\*(lpszArg2-8),lpszArg2,1);  
  if(!eax) goto lable4;  
  if(lpszArg4==1) goto lable5;  
  \*lpIntArg3=0;  
  goto lable3;  
lable5:  
  CString(lpszArg4);  
  mov  byte ptr\[ebp-4\],2;  
  CString(lpszArg1);  
  mov  byte ptr\[ebp-4\],3;  
  BasicCrtDll.BasicLoadStr(lpszArg4,0x281);  
  BasicCrtDll.BasicLoadStr(lpszArg1,0x28d);  
  wndVar3.MessageBox(lpszArg4,lpszArg1,MB\_YESNO or MB\_DEFBUTTON2 or MB\_ICONHAND);  
  if (eax!=6) goto lable6  
  \*lpIntArg3=2;  
  goto lable7;  
lable6:  
  \*lpIntArg3=0;  
lable7:  
  mov  byte ptr\[ebp-4\],3  
  wndVar3.~CWnd();  
  mov  byte ptr\[ebp-4\],2  
  ~CString(lpszArg1);  
  mov  byte ptr\[ebp-4\],1  
  ~CString(lpszArg4);  
  goto lable3;  
lable4:  
  \*lpIntArg3=1;  
lable3:  
  mov  byte prt\[ebp-4\],0  
  if(lpIntArg3==0) goto lable8;  
  \*(\*lpIntArg3+8)(lpIntArg3);  
lable8:  
  or  dword ptr\[ebp-4\],-1  
  ~CString(lpszArg2);    
}  
  
下一步还原控制语句：将控制语句，从函数中看到只有向下跳转的，没有向上跳转的，可知函数中只有循环结构  
给 if ... 条件编号：  
给代码快标号  
void TempName(void\* lpRegArg1,char\* lpszArg1,char\* lpszArg2,int\* lpIntArg3,?,lpszArg4)  
{  
  void\* lpVar1=NULL;  
  int intVar2;  
  CWnd wndVar3;  
  mov  \[ebp-4\],0  
  \*(\*(\*(lpRegArg1+0x84))+0x1c)(\*(lpRegArg1+0x84),0x004e7460,&lpVar1);  
  if(eax) goto lable1---------------------------------------------------------------------------c1  
  \*(\*(lpVar1)+0x14)(intVar1,0x004e8940,&intVar2);  
  if(!eax) goto lable2;-------------------------------------------------------------------------c2  
lable1:  
  \*lpIntArg3=2;-------------------------------code1  
  goto lable3;  
lable2:  
  \*(\*lpVar1+0x1c)(lpVar1,0,\*(lpszArg2-8),lpszArg2,1);  
  if(!eax) goto lable4;-------------------------------------------------------------------------c3  
  if(\*lpszArg4==1) goto lable5;------------------------------------------------------------------c4  
  \*lpIntArg3=0;-------------------------------code2  
  goto lable3;  
lable5:  
  CString(lpszArg4);---------------------------------------------------code3  
  mov  byte ptr\[ebp-4\],2;  
  CString(lpszArg1);  
  mov  byte ptr\[ebp-4\],3;  
  BasicCrtDll.BasicLoadStr(lpszArg4,0x281);  
  BasicCrtDll.BasicLoadStr(lpszArg1,0x28d);  
  wndVar3.MessageBox(lpszArg4,lpszArg1,MB\_YESNO or MB\_DEFBUTTON2 or MB\_ICONHAND);  
  if (eax!=6) goto lable6------------------------------------------------------------------------c5  
  \*lpIntArg3=2;-------------------------------------------------------code4  
  goto lable7;  
lable6:  
  \*lpIntArg3=0;-------------------------------------------------------code5  
lable7:  
  mov  byte ptr\[ebp-4\],3-------------------------------------------code6  
  wndVar3.~CWnd();  
  mov  byte ptr\[ebp-4\],2  
  ~CString(lpszArg1);  
  mov  byte ptr\[ebp-4\],1  
  ~CString(lpszArg4);  
  goto lable3;  
lable4:  
  \*lpIntArg3=1;--------------------------------------------------------code7  
lable3:  
  mov  byte prt\[ebp-4\],0---------------------------------------------code8  
  if(lpIntArg3==0) goto lable8;-----------------------------------------------------------------c6  
  \*(\*lpIntArg3+8)(lpIntArg3);-------------------------------------------code9  
lable8:  
  or  dword ptr\[ebp-4\],-1-------------------------------------------code10  
  ~CString(lpszArg2);    
}  
  
第二步：构造逻辑二叉树，当条件为假时，执行二叉树的左边，当条件为真时，执行二叉树的右边，无条件跳转和顺序执行的用直线表示  
如过用原来的条件画数不好画，有交叉线，就对原来的条件取反，可得到如下所示的二叉树：  
  
if(c1 || !c2)   
{  
  code1  
}  
elseif( c3 )  
{  
  code7;  
}  
elseif (c4)  
{  
  code3;  
  if(c5)  
  {  
    code5;  
  }  
  else  
  {  
    code4;  
  }  
  code6;  
}  
else  
{  
  code2;  
}  
  
code8;  
if(!c6)  
{  
  code9;  
}  
code10;  
  
将条件编号和代码快编号用相应的语句替换可得：  
c1 的条件为 测试 eax！=0 ,而 eax 就是上面函数调用的返回值  
所以 TempName 函数可写为：  
void TempName(void\* lpRegArg1,char\* lpszArg1,char\* lpszArg2,int\* lpIntArg3,?,lpszArg4)  
{  
  void\* lpVar1=NULL;  
  int intVar2;  
  CWnd wndVar3;  
  mov  \[ebp-4\],0  
  if(\*(\*(\*(lpRegArg1+0x84))+0x1c)(\*(lpRegArg1+0x84),0x004e7460,&lpVar1) || !\*(\*(lpVar1)+0x14)(intVar1,0x004e8940,&intVar2))  
  {  
    \*pIntArg3=2;--------------------------------------------------------code1  
  }  
  elseif (!\*(\*lpVar1+0x1c)(lpVar1,0,\*(lpszArg2-8),lpszArg2,1))  
  {  
    \*lpIntArg3=1;--------------------------------------------------------code7  
  }  
  elseif (\*lpszArg4==1)  
  {  
    CString(lpszArg4);---------------------------------------------------code3  
    mov  byte ptr\[ebp-4\],2;  
    CString(lpszArg1);  
    mov  byte ptr\[ebp-4\],3;  
    BasicCrtDll.BasicLoadStr(lpszArg4,0x281);  
    BasicCrtDll.BasicLoadStr(lpszArg1,0x28d);  
    wndVar3.MessageBox(lpszArg4,lpszArg1,MB\_YESNO or MB\_DEFBUTTON2 or MB\_ICONHAND);  
    if(wndVar3.MessageBox(lpszArg4,lpszArg1,MB\_YESNO or MB\_DEFBUTTON2 or MB\_ICONHAND)!=6)  
    {  
    \*lpIntArg3=0  
    }  
    else  
    {  
    \*lpIntArg3=2  
          }  
    mov  byte ptr\[ebp-4\],3-------------------------------------------code6  
    wndVar3.~CWnd();  
    mov  byte ptr\[ebp-4\],2  
    ~CString(lpszArg1);  
    mov  byte ptr\[ebp-4\],1  
    ~CString(lpszArg4);  
  }  
  else  
  {  
    \*lpIntArg3=0--------------------------------code2  
  }  
  mov  byte prt\[ebp-4\],0---------------------------code8  
  if(!(lpIntArg3==0))  
  {  
    \*(\*lpIntArg3+8)(lpIntArg3);-------------------------------------------code9  
  }  
  or  dword ptr\[ebp-4\],-1-------------------------------------------code10  
  ~CString(lpszArg2);  
}  
  
今天就到这里，这里可能会出好多错误，希望大家能不吝赐教，我对ｃ＋＋，ｍｆｃ　不熟，只对汇编比较熟，所以上面有些代码，也不知道什么样的源程序可以编译成这个  
样子，上面的代码中带有汇编指令，不过程序大致思路是这样的，还有如果谁有还原分支语句的好方法，讲出来，大家分享一下。  
  
2006-06-21 10:47 

出现严重错误 误将\[ebp-10\] lpVar1 看作了 \[ebp+10\] lpIntArg3  
  if(!(lpIntArg3==0))  
  {  
    \*(\*lpIntArg3+8)(lpIntArg3);-------------------------------------------code9  
  }  
应该为：  
  if(!(lpVar1==0))  
  {  
    \*(\*lpVar1+8)(lpVar1);-------------------------------------------code  
  }  
下面的事情就是来分析函数的功能，如果不是想完全彻底的分析函数功能，就不用分析所有调用的函数，只分析感兴趣的就 ok 了。  
  
好了，现在来总结一下：分析一个函数的一般步骤  
  
<1>找到函数的入口点和函数尾  
<2>找出函数中的参数个数  
<3>识别函数中的参数和局部变量  
<4>去掉编译器自己添加的或与函数功能无关的，异常处理,加上函数头  
<5>分析调用的函数,得到被调用函数原型  
<6>识别参数类型和局部变量类型  
<7>找出函数的返回值  
<8>将汇编语言指令还原成高级语言的表达式  
<9>还原控制语句  
<10>递归分析子函数  
<11>得出函数功能  
  
在分析函数参数类型，和局部变量类型时，可能分析的类型不正确，在以后的步骤中可以把错误纠正过来  
上面的 11 步，有可能有问题，让实践来检验她吧，并来逐步修正和完善她，使她越来越成熟越来越美丽......。