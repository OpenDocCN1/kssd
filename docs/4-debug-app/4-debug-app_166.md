

# 都不够给力,我来个给力的,手把手一起写现在某热门游戏的跑钱外挂(一)(二)(三)(四)

> 原文：[`www.pediy.com/kssd/pediy11/123841.html`](https://www.pediy.com/kssd/pediy11/123841.html)

2010 年 10 月 30 日..  
  
先声明,此为技术研究,不要拿来当非法用途.面向中高级的逆向与程序编写程序员.  
此游戏为某公司的某个热门回合制游戏,我会从一开始入手调试到分析,再到成品分批写文放出.....  
  
我所用的写代码语言是 masm32 就是汇编啦,哈哈...用 C 的高手们就不好意思了.因为地址定位用 C 太麻烦,又不想用 C 内嵌汇编,太难看..  
  
技术层面从基本的调试分析思路,到内存搜索定位,注入,SEH 搜索错误处理来定点搜索内存,内存地址位置转跳修改和修改挟持(系统 API 也有),由于是内存外挂,所以要处理游戏程序的自校验,过自校验方法不放出(我可不想收律师信),请自行思索..不过自校验也只是提示,还是能执行的.  
  
外挂程序成品是该游戏的一个分支的自动挣游戏币,里面包括战斗,走路,NPC 对话,地图转换,物品检测.  
  
OK,废话完了,我们开始动手.从理论开始做..  
  
先搞清楚我们要做的,就是利用游戏的内存定位修改来得到游戏数据,接着按照得到的数据利用鼠标键盘模仿操作(SendMessage,或者直接压参数 call 也行,随你喜欢),不要搞变态功能(因为为了跑钱的),实现人物走路加速和战斗加速只要修改一下断点内存就能做到,我会在后面的文章加以说明.  
  
首先,就是 OD 载入(又废话了..),收集数据开始...  
  
0051171E > $  6A 60         push    60  
00511720   .  68 F0975700   push    005797F0  
00511725   .  E8 9A520000   call    005169C4  
0051172A   .  BF 94000000   mov     edi, 94  
0051172F   .  8BC7          mov     eax, edi  
00511731   .  E8 3ADBFFFF   call    0050F270  
00511736   .  8965 E8       mov     dword ptr \[ebp-18\], esp  
00511739   .  8BF4          mov     esi, esp  
0051173B   .  893E          mov     dword ptr \[esi\], edi  
0051173D   .  56            push    esi                              ; /pVersionInformation  
0051173E   .  FF15 58715500 call    dword ptr \[<&KERNEL32.GetVersion>; \\GetVersionExA  
  
标准的 C++,,我暴汗,以前我调试时那游戏可是加了 License 的,那时过检测和分析代码累死我,API 全是乱序的.现在居然把壳放掉? OK,那就更简单了.  
Shift+F9 直接就跑起来了,汗.还真不检测了...  
  
开始,注入方式考虑,由于是外挂,不是木马,所以是用户配合程序,所以注入方式可以有很多,DLL 替换或者远线程注入,既然用户配合,那就不用 DLL 替换,我们来远线程注入(无视杀毒)..  
  
下面代码是我自己写的远线程 DLL 加载,自己看代码处理方式,使用方法是对目标程序使用 VirtualAllocEx 申请内存,之后将下面代码的 DllNameA 填充好我们的外挂 DLL 的路径,用 WriteProcessMemory 写入目标程序,之后用 CreateRemoteThread 建立远线程  
下面代码会在目标程序加载我们指定的 DLL 文件,我们在 DLL 文件的入口,DLL\_PROCESS\_ATTACH 时对游戏程序做内存修改与挟持就可以了.  
此代码可通用 XP,2003,VISTA,WIN7.  
assume   fs:nothing  
mov   eax,fs:\[30h\]  
mov   eax,\[eax + 0ch\]  
mov   esi,\[eax + 1ch\]  
lodsd  
mov   eax,\[eax + 8h\]  
mov  edi,eax  
call  @f  
loadlib  db  'LoadLibraryExA',0  ;为什么是 ExA?不用 A? 因为啊,要兼容 WIN7.所以是 ExA 再问就自己去调试 WIN7 的处理方式- -!..  
@@:  
pop  edx  
mov  esi,edx  
call  \_test  
add  edx,143          ;这里编译后的代码大小,用来定位 DllNameA 路径数据的 没修改代码的情况下别改  
push  0  
push  0  
push  edx  
call  esi  
add  eax,1000h  ;无视掉.懒得编译修改,修改的话上面的 143 大小也要修改.  
mov  ecx,1234  ;同上  
nop      ;同上  
ret  
assume  esi:nothing  
\_test    proc  uses eax ebx ecx edx edi    ;这段代码用来搜索函数的.  
      
    add     eax, \[eax + 3Ch\]  
    assume    eax: ptr IMAGE\_NT\_HEADERS  
    mov  ebx, edi  
    add  ebx, \[eax\].OptionalHeader.DataDirectory.VirtualAddress    ;IMAGE\_EXPORT\_DIRECTORY 引出表入口  
    assume    eax: nothing  
    mov        ecx, ebx  
    mov  ebx, \[ecx + 1Ch\]                ;AddressOfFunctions 的地址  
    mov  edx, ecx  
    mov  edx, \[edx + 18h\]                ;NumberOfNames 地址  
    push  edx  
    mov  edx,esi  
    mov  ebp, ecx      ;IMAGE\_EXPORT\_DIRECTORY 引出表入口  
    mov  ebp, \[ebp + 20h\]    ;AddressOfNames 地址  
      
    mov  esi, ecx  
    mov  esi, \[esi + 24h\]    ;AddressOfNameOrdinals  
    push    ebx  
    push    esi  
    xor  ebx, ebx  
    xor  ecx, ecx  
    xor  eax, eax  
    add  ebp, edi  
      
  getApiGetModuleHandleA:  
    mov  esi, edi  
    add  esi, \[ebp + ecx \* 4\]  
  cmpAPI:  
    mov  al, \[edx + ebx\]  
    cmp  al, \[esi + ebx\]  
    jne  getNext  
    inc  ebx  
    mov  al, 0  
    cmp  al, \[esi + ebx\]  
    jne  cmpAPI  
    je  getOK  
  getNext:  
    xor  ebx, ebx  
    inc  ecx  
    cmp  ecx, \[esp + 8\]  
    jne  getApiGetModuleHandleA  
  getOK:  
    pop  esi     
    pop  ebx  
    add  ebx, edi  
    mov  esi, \[ebx + ecx \* 4\]  
    add  esi, edi      ;esi 为 API 地址  
    pop  eax  
  ret  
\_test endp  
  
DllNameA  db  260 dup (0)  
  
面向中高手,代码我就不一一说明了,只是用来加载 DLL 到目标进程的东西,自己看,看不懂的话下面我写的游戏调试和其它的处理估计也会看得不太懂,努力吧.  
基础的远线程载入就好了,现在我们建立我们的 DLL 文件,从空白的开始.masm32 汇编,自己看着办  
  
    .586  
    .model flat, stdcall  
    option casemap :none  
  
include    windows.inc  
include    kernel32.inc  
includelib  kernel32.lib  
  
    .code  
\_Init    proc  uses ebx esi edi  
  
          ;这里写入初始化的数据,用来写游戏进程的内存.先用 messagebox 来先测试代码的可用性吧.  
    ret  
  
\_Init endp  
  
DllEntry  proc  \_hInstance,\_dwReason,\_dwReserved  
      
    cmp  \_dwReason,DLL\_PROCESS\_ATTACH  
    jnz  @f  
    push  eax  
    invoke  CreateThread,0,0,offset \_Init,0,0,esp  
    invoke  CloseHandle,eax  
    pop  eax  
    @@:  
    mov  eax,TRUE  
    ret  
  
DllEntry  Endp  
    End  DllEntry  
  
DLL 文件加载和处理基本就上面那些,当然细节要自己搞了,哈哈  ..准备工作做好了,我们来对游戏进程调试了...GO\`\`\`\`\`  
再用 OD 载入,为了写个程序开始的执行点,以前代码是一开始就加密了的,现在不用,我们还是定位个程序开始执行后的点吧,以后会有用的.  
断点:CreateWindowExA  Shift+F9,断下来了..  
77D2E4A9 >  8BFF            mov     edi, edi                ;CreateWindowExA 入口  
77D2E4AB    55              push    ebp  
77D2E4AC    8BEC            mov     ebp, esp  
  
断到第 1 次窗口建立,它有建立窗口,利用价值有了,我们利用它能知道代码已经全部解密(对已加壳的程序来说),在这里开始对游戏内存进行修改了,写一次性代码写入它,再用代码还原覆盖回去再 JMP 跳回 CreateWindowExA 让它继续跑就行了.  
注入条件齐备,下一篇是游戏调试分析,每天晚上一篇不同技术,到说完出成品,呵呵.  
  
说点题外话,什么防外挂木马的,要处理我觉得还是可行吧?即使没游戏的源代码,最可行的是写驱动进行游戏数据加密与发送数据加密,驱动进行本进程的数据校对(不让你修改看你怎么搞,干掉驱动我让你玩不了,没驱动加密的异常数据发送过来了,嘻嘻,写段代码自动封号..),其实驱动模版我写了 1 个,嘻嘻,通用 32 位的 WIN7,VISTA,2003,XP  
主要功能就是保护指定进程内存不被修改(一发现任何 1 字节内存修改马上退出)当然驱动也有自校对保护了,哈哈,功能能嘛,是用驱动限制程序启动,也就是每打开进程的话先提示进程路径,用户选择是否启动该进程(所有用户级的进程,不包含服务进程,呵呵,因为驱动 HOOK 的是 createprocess,某种意义上说我 HOOK 的是系统所有内核调用- -!). 

2010 年 10 月 31 日..  
  
有人问到几可以看到成品,可以告诉你,,到七.等你看完就可以自己写出成品了.  
  
接下来是调试篇,这个有点个人化和模糊,我只说重点.  
  
还是那句,打开游戏,Shift+f9 直接跑,接着干嘛? 找有利用价值的字符串....  
  
由于 OD 本来的搜索对汉字支持不太好,所以这里借助个插件,Ultra String Reference 看雪有下,嘻嘻  
  
我们用插件,FIND ASCII 出来一大堆字符串,开始分析 有人说我了,你 TM 是不是调试啊,不从程序开始就分析? 我明确告诉你.我们写的是外挂,只要达到目的就行,要不要给你写个比源代码还详细的分析日志?有空没事做?  
  
我下滚下滚下滚,发现了这么一些东西,这.不是脚本的合并串么? 猜测没用,直接断两个来试试.  
Ultra String Reference, 条目 2011  
 Address=00474C65  
 Disassembly=push    0055B0A0  
 Text String=类名=动画 脚本对象=%s 脚本参数=%s x=%d y=%d 图片=%s %s  
  
Ultra String Reference, 条目 2014  
 Address=00475195  
 Disassembly=push    0055B110  
 Text String=类名=复合动画 脚本对象=%s 脚本参数=%s x=%d y=%d 资源路径=%s %s  
  
Ultra String Reference, 条目 2017  
 Address=00475686  
 Disassembly=push    0055B168  
 Text String=类名=按钮 鼠标=3 脚本对象=%s 脚本参数=%s x=%d y=%d 图片=%s 说明=%s %s  
  
Ultra String Reference, 条目 2019  
 Address=00475A64  
 Disassembly=push    0055B1B0  
 Text String=类名=t 按钮 鼠标=3 脚本对象=%s 脚本参数=%s x=%d y=%d 图片='%s' 颜色=%s 文字='%s' 尺寸=%d 粗细=%d alpha=%d %s  
  
Ultra String Reference, 条目 2022  
 Address=00475F8E  
 Disassembly=push    0055B220  
 Text String=类名=大话牌  脚本对象=%s 脚本参数=%s x=%d y=%d 编号=%d 图号=%d 边属性=%s %s  
  
这个游戏嘛,打开时需要点击"进入游戏"的,也就是按钮,按照上面的合并串,类名=按钮 或 类名=t 按钮 其中一个了,下断.  
00475682  |.  8D5424 2C     lea     edx, dword ptr \[esp+2C\]  
00475686  |.  68 68B15500   push    0055B168  
0047568B  |.  52            push    edx  
0047568C  |.  E8 2F900400   call    004BE6C0  
断 CALL 那里,为什么我不用说了吧?  
OD 把程序重新加载,跑起来,哦,断了..  
注意堆栈,资源路径,当然是被打包过的,呵呵..  
0012ED1C   0012ED50  
0012ED20   0055B168    
0012ED24   046789A8  ASCII "qaaYyU4.jY0EGS5"  
0012ED28   01C61078  
0012ED2C   0000278A  
0012ED30   00000178  
0012ED34   03F3BB18  ASCII "gires2/button/autoscroll2.tca"  
0012ED38   021B41E0  
  
dword ptr \[0012ED50\]=047AF27C  
047AF27C  C0 E0 C3 FB 3D B0 B4 C5 A5 20 CA F3 B1 EA 3D 33  类名=按钮 鼠标=3  
047AF28C  20 BD C5 B1 BE B6 D4 CF F3 3D 71 61 61 59 79 55   脚本对象=qaaYyU  
047AF29C  34 2E 6A 59 30 45 47 53 35 20 BD C5 B1 BE B2 CE  4.jY0EGS5 脚本参  
047AF2AC  CA FD 3D 31 20 78 3D 31 30 31 32 32 20 79 3D 33  数=1 x=10122 y=3  
047AF2BC  37 36 20 CD BC C6 AC 3D 67 69 72 65 73 32 2F 62  76 图片=gires2/b  
047AF2CC  75 74 74 6F 6E 2F 61 75 74 6F 73 63 72 6F 6C 6C  utton/autoscroll  
047AF2DC  32 2E 74 63 61 20 CB B5 C3 F7 3D 20 20 B1 D8 D0  2.tca 说明=  必  
047AF2EC  EB BB AD 3D 31 00                                牖?1..  
  
这里返回值,有点与普通的字符合并有点不同,自己单步调试进去就知道,返回值为 dword ptr \[\[esp\]\] 必须是调用后,dword ptr \[esp\]指向合并后字符串地址  
既然得到了资源开始处理,要得到资源 ID 就是直接跟着 F8 就行,等它压入返回的字符串的 CALL..  
  
004756A2  |.  8D4424 38     lea     eax, dword ptr \[esp+38\]  
004756A6  |.  50            push    eax  
004756A7  |.  8D4C24 14     lea     ecx, dword ptr \[esp+14\]  
004756AB  |.  51            push    ecx  
004756AC  |.  51            push    ecx  
004756AD  |.  8BC4          mov     eax, esp  
004756AF  |.  896424 34     mov     dword ptr \[esp+34\], esp  
004756B3  |.  C700 C9D9278D mov     dword ptr \[eax\], 8D27D9C9  
004756B9  |.  8B5424 40     mov     edx, dword ptr \[esp+40\]  
004756BD  |.  51            push    ecx  
004756BE  |.  8BC4          mov     eax, esp  
004756C0  |.  8910          mov     dword ptr \[eax\], edx  
004756C2  |.  8B4C24 24     mov     ecx, dword ptr \[esp+24\]  
004756C6  |.  896424 38     mov     dword ptr \[esp+38\], esp  
004756CA  |.  E8 C1E8FFFF   call    00473F90  
来了,一个 CALL 之后,接着的代码,在第 3 个参数压入了  
0012ED30   FFFFFFFE  
0012ED34   8D27D9C9  
0012ED38   0012ED50  
对比上面的数据,0012ED50 就是这个了.里面放了字符串的值.也就是说函数里面解析这个脚本,OK,单步进去看看啥情况,要是懒人的话直接对字符串下内存读取断,接着在 004756CA  |.  E8 C1E8FFFF   call    00473F90 下面的返回下一个执行断,直接 Shift+F9.  
万一不是解析 CALL,那内存读取断不生效,也就说不是这个函数,呵呵.要确定是不是的简单办法就是直接下内存读取断,又没人不让你多次重新载入程序.  
我承认我是懒人,OK,下读取断,Shift+f9,ok,断了.  
0046D2B5  |.  8A02          mov     al, byte ptr \[edx\]   ;在这里断了,猜一下,是不是解析点,我们下个条件记录断.  
0046D2B7  |.  3C 20         cmp     al, 20  
0046D2B9  |.  0F84 6F020000 je      0046D52E  
0046D2BF  |.  3C 0A         cmp     al, 0A  
0046D2C1  |.  0F84 67020000 je      0046D52E  
0046D2C7  |.  3C 0D         cmp     al, 0D  
0046D2C9  |.  0F84 5F020000 je      0046D52E  
0046D2CF  |.  84C0          test    al, al  
0046D2D1  |.  57            push    edi  
断记录所有,表达式 EDX,选 ASCII,我们跑起来看看啥情况...注意看记录  
看记录,应该偷笑了,全部脚本解析都在这里进行.  
Log data, 条目 126  
 地址=0046D2B5  
 消息=COND: 045BD8C8 "x=-1000 y=-1000 ",BD,"疟?,B6,"韵?ip7Y3e0  ",BD,"疟?,B2,"问? 宽=0 ",B8,"?0 颜?,AB,"= 边框='' ",BB,"",AD,"",B2,"?0 模?,BD,"=0 "  
Log data, 条目 112  
 地址=0046D2B5  
 消息=COND: 047551B4 "速",B6,"?7 n=26 ?,B7,"?,B6,"='gires/cursor/'"  
Log data, 条目 68  
 地址=0046D2B5  
 消息=COND: 047AF27C "类名=",B0,"",B4,"钮 鼠标=3 ",BD,"疟?,B6,"韵?oNcakL2.qPnBKE2 ",BD,"疟?,B2,"问?1 x=122 y=376 图?,AC,"=gires2/button/autoscroll.tca 说?,F7,"=  必须",BB,"",AD,"=1"  
Log data, 条目 34  
 地址=0046D2B5  
 消息=COND: 047551B4 "类名=图?,AC," x=52 y=215 图?,AC,"='gires2/login/stick.tcp'  z=4"  
  
虽然汉字显示不太正常,不过按照我们习惯,很容易看出那是什么字吧?呵呵....  
接着我们要做的,得到它建立出来的 ID,这个 ID 在后面调试鼠标点击时用的.有人说了,为什么不直接发送 MOVE 消息给鼠标让它定点再发 wm\_lbuttonup,呵呵,因为这个游戏对鼠标捕获做了些处理,所以不能 100%的得到想要的像素坐标.最后的文章我将说明怎么打内存补丁让鼠标恢复正常.  
好了,拿到解析点,只要在解析点打个补丁,就能知道游戏内的所有提示资源或者程序将要做什么.走路或者战斗或者点击 NPC  
不过我做的是不再全部解析点做,记得上面的按钮建立吗?只要跟着那里做,可以实现自动登录了,拿 ID 去,怎么拿?单步一直跟到我们之前的断点,004756CA  |.  E8 C1E8FFFF   call    00473F90,发现了.  
嘛,看见了没?压成浮点,将 ID 保存到内存中,有兴趣的可以跟进去看它保存到哪里.不过对我们作用不大.知道这里是按钮的 ID 就行了.  
004756CA  |.  E8 C1E8FFFF   call    00473F90  
004756CF  |.  DB4424 38     fild    dword ptr \[esp+38\]  ;这里就是该按钮的 ID 了,要知道详细的还是自己单步下,这是什么游戏?呵呵..SO 呢.最后我会给提示的.  
004756D3  |.  83EC 08       sub     esp, 8  
004756D6  |.  DD5C24 30     fstp    qword ptr \[esp+30\]  
004756DA  |.  DD4424 30     fld     qword ptr \[esp+30\]  
004756DE  |.  DD1C24        fstp    qword ptr \[esp\]  
004756E1  |.  E8 DA300300   call    004A87C0  
004756E6  |.  50            push    eax  
知道资源名,和 ID,它是什么按钮也知道了吧?呵呵,接下来我们开始做坏事...  
接下来就是看看鼠标左键点击处理了什么东西了,我们看窗口界面,对主窗口下断,当然就是下 wm\_lbuttonup 了.  
断到主窗口的消息处理那里了.  
004C4960 > .  8B4424 04     mov     eax, dword ptr \[esp+4\]  
004C4964   .  53            push    ebx  
004C4965   .  55            push    ebp  
004C4966   .  56            push    esi  
004C4967   .  57            push    edi  
004C4968   .  6A EB         push    -15                              ; /Index = GWL\_USERDATA  
004C496A   .  50            push    eax                              ; |hWnd  
004C496B   .  FF15 04745500 call    dword ptr \[<&USER32.GetWindowLon>; \\GetWindowLongA  
我们 F8 单步来到他的点击处理位置  
004C49AC   > \\8D0440        lea     eax, dword ptr \[eax+eax\*2\]  
004C49AF   .  8B4CC6 0C     mov     ecx, dword ptr \[esi+eax\*8+C\]  
004C49B3   .  8D04C6        lea     eax, dword ptr \[esi+eax\*8\]  
004C49B6   .  53            push    ebx  
004C49B7   .  55            push    ebp  
004C49B8   .  03CF          add     ecx, edi  
004C49BA   .  FF50 08       call    dword ptr \[eax+8\]                ;  xy2.00467FE0  
就是这个 CALL 了,点击弹起按钮的位置.F7 进去,  
00467FE0   .  56            push    esi                              ;  xy2.00593DC8  
00467FE1   .  8BF1          mov     esi, ecx  
00467FE3   .  8A46 49       mov     al, byte ptr \[esi+49\]  
00467FE6   .  84C0          test    al, al  
00467FE8   .  75 0B         jnz     short 00467FF5  
00467FEA   .  8A46 3E       mov     al, byte ptr \[esi+3E\]  
00467FED   .  84C0          test    al, al  
00467FEF   .  0F85 90000000 jnz     00468085  
00467FF5   >  8A46 3D       mov     al, byte ptr \[esi+3D\]  
00467FF8   .  84C0          test    al, al  
00467FFA   .  74 50         je      short 0046804C      ;单步发现到这里直接跳到下面去了,可以判断这个游戏的处理方式是按照标记点击进行,而不是在这个函数内处理的.  
00467FFC   .  8B0D DC396000 mov     ecx, dword ptr \[6039DC\]  
00468002   .  85C9          test    ecx, ecx  
00468004   .  74 46         je      short 0046804C  
00468006   .  8079 04 01    cmp     byte ptr \[ecx+4\], 1  
0046800A   .  75 19         jnz     short 00468025  
0046800C   .  A1 24365E00   mov     eax, dword ptr \[5E3624\]  
00468011   .  8A50 3D       mov     dl, byte ptr \[eax+3D\]  
00468014   .  84D2          test    dl, dl  
00468016   .  74 0D         je      short 00468025  
00468018   .  8B11          mov     edx, dword ptr \[ecx\]  
0046801A   .  6A 00         push    0  
0046801C   .  FF52 4C       call    dword ptr \[edx+4C\]  
0046801F   .  8B0D DC396000 mov     ecx, dword ptr \[6039DC\]  
00468025   >  8B01          mov     eax, dword ptr \[ecx\]  
00468027   .  6A 03         push    3  
00468029   .  FF50 4C       call    dword ptr \[eax+4C\]  
0046802C   .  8B0D DC396000 mov     ecx, dword ptr \[6039DC\]  
00468032   .  8B4424 08     mov     eax, dword ptr \[esp+8\]  
00468036   .  8B11          mov     edx, dword ptr \[ecx\]  
00468038   .  50            push    eax  
00468039   .  FF52 54       call    dword ptr \[edx+54\]  
0046803C   .  8B0D DC396000 mov     ecx, dword ptr \[6039DC\]  
00468042   .  8B4424 0C     mov     eax, dword ptr \[esp+C\]  
00468046   .  8B11          mov     edx, dword ptr \[ecx\]  
00468048   .  50            push    eax  
00468049   .  FF52 54       call    dword ptr \[edx+54\]  
0046804C   >  FF86 80000000 inc     dword ptr \[esi+80\]   ;跳到这里了.看,下面有利用价值的只有 mov     dword ptr \[esi+560\], edx 和 and     dword ptr \[esi+A7E4\], eax 它们影响了全局变量,也就说明都在这里判断,我们写入后,下硬件读取断点  
00468052   .  8B96 60050000 mov     edx, dword ptr \[esi+560\]  
00468058   .  B8 FEFFFFFF   mov     eax, -2  
0046805D   .  23D0          and     edx, eax  
0046805F   .  8996 60050000 mov     dword ptr \[esi+560\], edx  
00468065   .  2186 E4A70000 and     dword ptr \[esi+A7E4\], eax  
0046806B   .  8A46 5D       mov     al, byte ptr \[esi+5D\]  
0046806E   .  84C0          test    al, al  
00468070   .  75 0D         jnz     short 0046807F  
00468072   .  8B0D 343D6000 mov     ecx, dword ptr \[603D34\]  
00468078   .  C681 10010000>mov     byte ptr \[ecx+110\], 0  
0046807F   >  FF15 EC735500 call    dword ptr \[<&USER32.ReleaseCaptu>; \[ReleaseCapture  
00468085   >  33C0          xor     eax, eax  
00468087   .  5E            pop     esi  
00468088   .  C2 0800       retn    8  
断点好了不?我们就直接 Shift+F9 跑起来,经过 3 次断点后,我们发现了..  
004741F3  |. /75 67         jnz     short 0047425C  
004741F5  |. |F686 C8040000>test    byte ptr \[esi+4C8\], 1  ;这里是我们下断点的位置,直接 test 判断,明白了吧?呵呵  
004741FC  |. |75 5E         jnz     short 0047425C  
004741FE  |. |F686 D0040000>test    byte ptr \[esi+4D0\], 1  ;这里是我们下断点的位置,直接 test 判断,明白了吧?呵呵  
00474205  |. |74 55         je      short 0047425C  
接着上下翻这段代码,发现..  
00474268  |.  F686 C8040000>test    byte ptr \[esi+4C8\], 2  
0047426F  |.  75 79         jnz     short 004742EA  
00474271  |.  F686 D0040000>test    byte ptr \[esi+4D0\], 2  
00474278  |.  74 70         je      short 004742EA  
记得刚刚的左键点击压入值吗?没错,就是 1,那么我们去断右键弹起,得到的的压入值是 2,嘻嘻 也就是说左右键是这里进行判断的.接着我们就要搞清楚 ID 是从那里来了.OK,直接进入左键的点击,  
004741F5  |.  F686 C8040000>test    byte ptr \[esi+4C8\], 1  
004741FC  |.  75 5E         jnz     short 0047425C  
004741FE  |.  F686 D0040000>test    byte ptr \[esi+4D0\], 1  
00474205  |.  74 55         je      short 0047425C  
00474207  |.  8B0D 58495900 mov     ecx, dword ptr \[594958\]          ;  xy2.00ACE980  
0047420D  |.  894C24 10     mov     dword ptr \[esp+10\], ecx  
00474211  |.  8B55 14       mov     edx, dword ptr \[ebp+14\]  
单步让程序继续跑.当然要删除刚刚的全部断点了.呵呵,F7 跟进去.观察寄存器和堆栈,记得上,面我们得到的 ID 吗?开始配对,看..EDI 的值=我们的 ID.猜测开始,EDI,就寄存器上说,一个程序在一个函数内是不会轻易改变的,我们往这个函数的上面看,  
004740D9  |.  33FF          xor     edi, edi    ;清空.  
004740DB  |.  E8 80CB0200   call    004A0C60  
004740E0  |.  8B4D 08       mov     ecx, dword ptr \[ebp+8\]  
004740E3  |.  83C4 04       add     esp, 4  
004740E6  |.  85C9          test    ecx, ecx  
004740E8  |.  894424 14     mov     dword ptr \[esp+14\], eax  
004740EC  |.  74 03         je      short 004740F1  
004740EE  |.  8B79 10       mov     edi, dword ptr \[ecx+10\]  ;写入..  
004740F1  |>  8B86 E0040000 mov     eax, dword ptr \[esi+4E0\]  
004740F7  |.  3BF8          cmp     edi, eax  
发现 EDI 的写入只有这两句.没关系,我们暂时不鸟它,直接改掉 EDI 看看还能不能使用正常的 EDI 点击,先记录下 EDI 的值,一会要额外测试点其它地方把 EDI 改回去看情况.改成 0 看看.Shift+f9 跑,发现.点击变无效了,  
接(FFFFFFC3 我的 EDI 值)着我们用鼠标随便点游戏框内的一个地方,断.这次 edi 为 FFFFFFC6,随便点游戏窗口内的位置的.接着我们把 C3 改进去看看效果.Shift+F9.  
哦耶\`\`搞掂,得到了按钮 ID 和按钮处理时的 ID 修改位置,只要 HOOK 这两个地方,就能实现只要发送 WM\_LBUTTONDOWN 和 UP 就可以不用鼠标位置来操作游戏了.  
按钮是这里,接着举一反三,记得上面的 ID 处理吧?图像和其它的 ID 都能拿到,顺带一提,游戏内的 NPC ID 也是这样获取和 ID 处理的,这个是什么游戏?自己看看哪个回合制游戏能种植的,就是它了.利用种植自动跑钱的.  
我说出来的理由,是为了让大家更好理解上面说的,自己手动做一次就清楚点.呵呵.  
  
今天有点晚了,因为吃饭时有约出去了,本来打算今天把调试篇全部写完,不过算了,后续留明天吧,明天是 SEH 处理方式的内存搜索定位(有人说内存挂不稳,因为游戏容易更新,我会用一种比较变态的方式去打补丁的,所以一般更新影响不了)和加内存补丁,我会把未说完的调试一起加进去. 

都不够给力,我来个给力的,手把手一起写现在某热门游戏的跑钱外挂(三)  
  
2010 年 11 月 1 日...  
  
补充昨天的调试部分和今天的内存搜索与补丁.高潮部分到了.想写加速的消化掉这篇你就成功了.  
我即写即发的,难免会有错字或者其它错,大家要指出来哈.谢谢..  
昨天调试,到了按钮部分.多次测试按钮和 T 按钮的分别,我们发现游戏功能性按钮是按钮那里的调用,而会改变的服务器选择和游戏内物品是 T 按钮.  
人物对话框弹出的选择性语句,例如传送员的,里面也是 T 按钮,据此分析得出的结论不用说啦.哈哈.NPC 对话和物品 ID 获取完毕.接着来的是关键点的,人物移动.与模型(NPC 和游戏角色)的 ID 获取  
记得我们之前的那个无敌脚本解析点么? 分析那里,得出这个游戏的指令是先串成文字脚本形式,再由那里把脚本解析掉做相应操作的.我们在上篇文章那里那里下条件记录断,建个角色,把记录保存到文件(记录会好多,保存下来好分析).  
角色建好那我们开始 OD 加载,进入游戏.我们是要分析游戏其行为,所以条件记录断要一开始就让它生效来记录.哦,忘记了,这游戏新手村好多人,找个没人的房子进去闪一边那样会更容易分析.  
随便让角色跑几步,我们开始分析记录文件,数据太多,我拿主要部分出来说.一句一句看吧,嘛,既然是先做成脚本串再解析的,那就试试搜索自己的角色名字看看有没发现,在 log 记录那里.  
0046D2B5  COND: 04B0C610 "类名=t",B0,"",B4,"钮 鼠标=3 ",BD,"疟?,B6,"韵?c0FoWa1.fkEuxd2 ",BD,"疟?,B2,"问?0 x=102 y=195 图?,AC,"='' 颜?,AB,"=white 文",D7,"?'testdbyou' 尺",B4,"?11 ",B4,"窒",B8,"=400 alpha=0  ",B6,"云?4"  
这句.哦,角色选择时的 T 按钮.类名=T 按钮嘛..用 ACSII 来看..向下找自己角色名..OH NO 找到了..模型加载  
0046D2B5  COND: 03F62AC8 "称?,BD,"='' 颜?,AB,"2='80C0FF' 名",D7,"?'testdbyou' 颜?,AB,"='00FF00' 速",B6,"?3 站?,A2,"=shape/char/0063/stand.tcp 站?,A2,"2=shape/char/0063/stand2.tcp 行",D7,"?shape/char/0063/walk.tcp ",B2,"",BD,"距=72 ",B2,"",BD,"速=24 跑",B2,  
为何上面是模型加载?根据经验得出的,这个说不清楚,反正第 1 眼看到就觉得是重要的,还是猜的,哈哈..用上面语句,我们来在无敌记录断那里做个程序暂停的条件断点,为了得到这个模型的 ID.看  
设置条件断点,条件为 dword ptr \[edx\] == 0bdcec6b3  
0bdcec6b3='称谓'的 ACSII 逆序  
04F64680  B3 C6 CE BD 3D 27 C1 F7 BD F0 CB EA D4 C2 D2 BB  称谓='流金岁月一  
04F64690  D4 B1 27 20 D1 D5 C9 AB 32 3D 27 38 30 43 30 46  员' 颜色 2='80C0F  
04F646A0  46 27 20 C3 FB D7 D6 3D 27 74 65 73 74 64 62 79  F' 名字='testdby  
04F646B0  6F 75 27 20 D1 D5 C9 AB 3D 27 30 30 46 46 30 30  ou' 颜色='00FF00  
04F646C0  27 20 CB D9 B6 C8 3D 33 20 D5 BE C1 A2 3D 73 68  ' 速度=3 站立=sh  
04F646D0  61 70 65 2F 63 68 61 72 2F 30 30 36 33 2F 73 74  ape/char/0063/st  
04F646E0  61 6E 64 2E 74 63 70 20 D5 BE C1 A2 32 3D 73 68  and.tcp 站立 2=sh  
04F646F0  61 70 65 2F 63 68 61 72 2F 30 30 36 33 2F 73 74  ape/char/0063/st  
04F64700  61 6E 64 32 2E 74 63 70 20 D0 D0 D7 DF 3D 73 68  and2.tcp 行走=sh  
04F64710  61 70 65 2F 63 68 61 72 2F 30 30 36 33 2F 77 61  ape/char/0063/wa  
04F64720  6C 6B 2E 74 63 70 20 B2 BD BE E0 3D 37 32 20 B2  lk.tcp 步距=72   
04F64730  BD CB D9 3D 32 34 20 C5 DC B2 BD 3D 73 68 61 70  剿?24 跑步=shap  
04F64740  65 2F 63 68 61 72 2F 30 30 36 33 2F 72 75 6E 2E  e/char/0063/run.  
04F64750  74 63 70 20 C5 DC B2 BD B2 BD BE E0 3D 31 30 38  tcp 跑步步距=108  
04F64760  20 C5 DC B2 BD CB D9 B6 C8 3D 34 35 20 C5 E4 C9   跑步速度=45 配  
04F64770  AB B7 BD B0 B8 3D 27 73 68 61 70 65 2F 63 68 61  桨?'shape/cha  
04F64780  72 2F 30 30 36 33 2F 30 30 2E 70 70 27 00        r/0063/00.pp'.  
哦,看见上面语句了吗?同学们,关键的东西来了,要搞破坏的赶快.跑步速度=45,要是修改掉这句话的这句参数的话,嘻嘻..一个外面跑步的加速挂就出现了..不过我们要的不是这种破坏式的,我们继续我们的.  
看到这里,我们拿的是 ID,那么我们取消掉刚刚的田间断,向下滚代码,找到 RET 就直接 F9 断,函数间隔用 int3 的,所以只要看到几个 INT3 就知道这个函数结束了.  
1 个返回点.继续,其实也可以用 F8 去单步,不过那么效率不够,嘻嘻..  
0046D4DE  |.  8B4C24 1C     mov     ecx, dword ptr \[esp+1C\]  
0046D4E2  |.  64:890D 00000>mov     dword ptr fs:\[0\], ecx  
0046D4E9  |.  83C4 28       add     esp, 28  
0046D4EC  |.  C2 0400       retn    4  
0046D4EF  |>  2BF2          sub     esi, edx  
2 个..  
0046D543  |.  5E            pop     esi  
0046D544  |.  5D            pop     ebp  
0046D545  |.  5B            pop     ebx  
0046D546  |.  64:890D 00000>mov     dword ptr fs:\[0\], ecx  
0046D54D  |.  83C4 28       add     esp, 28  
0046D550  \\.  C2 0400       retn    4  
0046D553      CC            int3  
0046D554      CC            int3  
看见了 int3 了没?这个函数结束了,也就是返回点是这两个,我们 Shift+f9 跑起来.断到返回,F8 继续跟出去.跟到哪里?记得上面的按钮和 T 按钮吗?  
我们跟到函数返回那个位置出现浮点指令,那么那里的就是模型 ID 了,可以从那里获取到,嘻嘻..经过几次的 ret,我们跟到了这里.  
00480940   .  50            push    eax  
00480941   .  56            push    esi  
00480942   .  57            push    edi  
00480943   .  E8 183FFEFF   call    00464860  
00480948   .  894424 1C     mov     dword ptr \[esp+1C\], eax  
0048094C   .  DB4424 1C     fild    dword ptr \[esp+1C\]  
00480950   .  A1 3CA86200   mov     eax, dword ptr \[62A83C\]  
00480955   .  83C4 0C       add     esp, 0C  
00480958   .  DD5C24 10     fstp    qword ptr \[esp+10\]  
0048095C   .  DD4424 10     fld     qword ptr \[esp+10\]  
00480960   .  DD1C24        fstp    qword ptr \[esp\]  
00480963   .  50            push    eax  
00480964   .  E8 B7080200   call    004A1220  
这里什么意思呢? 对比上面的按钮处理方式,这里就是保存人物 ID 的位置,mov     dword ptr \[esp+1C\], eax,也就是 eax 就是我们的角色 ID.  
为了验证一下?好,之前我们调试过哪个左键点击 ID 那里有 ID 值的,我们来下个断,点击一下自己的角色,哦,没错了吧?点击角色的 ID 就是上面返回的 ID.到这里,上面的点,多次测试,是每个模型加载必进的定位点,也就是 NPC 的 ID 或者其它玩家的 ID 也在那里返回.  
如何判断?名字='testdbyou' 他都告诉你了,汗.这还判断不了?人物 ID 我们得到了,接着是人物的移动问题,我们继续分析之前的 log 记录,发现  
0046D2B5  COND: 0579F73C "终点 X=489 终点 Y=1841 跑=1"  
在测试一次,这次用走路的,结果是'跑=0'.....  
这个..角色移动的目标位置?还是猜测.试试吧,条件断在'终点',跟到外面看看它是干嘛的.  
条件为 dword ptr \[edx\] == 0e3b5d5d6  
进去游戏走一步..断了...我们还是上面那招,让程序跑到返回那里  
这次不像刚刚那样有先前的指令点,我们每返回一次,就在它的那个 CALL 那里下一个断点.大概返回 3-4 个吧,就够了,直接让程序跑起来.如果程序不断断在我们下断的 CALL 那里,我们就把断点去掉,排除掉那个走路的关键点,接着,排除得出只有一个 CALL 是要点击走路时才进入的,OK,开始分析.  
看堆栈.也就是看这个 CALL 的压入参数和寄存器.发现了,  
004813B1   .  55            push    ebp  
004813B2   .  8D4C24 14     lea     ecx, dword ptr \[esp+14\]  
004813B6   .  51            push    ecx  
004813B7   .  51            push    ecx  
004813B8   .  8BC4          mov     eax, esp  
004813BA   .  896424 20     mov     dword ptr \[esp+20\], esp  
004813BE   .  51            push    ecx  
004813BF   .  8938          mov     dword ptr \[eax\], edi  
004813C1   .  8BC4          mov     eax, esp  
004813C3   .  8930          mov     dword ptr \[eax\], esi  
004813C5   .  8B0D 403A6000 mov     ecx, dword ptr \[603A40\]  
004813CB   .  C74424 30 000>mov     dword ptr \[esp+30\], 0  
004813D3   .  896424 24     mov     dword ptr \[esp+24\], esp  
004813D7   .  E8 9429FFFF   call    00473D70  
  
堆栈  
0012F8EC   5131B483  
0012F8F0   B8FA9C1F  
0012F8F4   0012F90C  
0012F8F8   02201A90  
  
压入了 4 个参数,看,第 1 个参数就是我们的角色 ID,接着跟进看看其它参数,发现第 3 个参数.里面 dword 的指向值就是.  
05941BEC  D6 D5 B5 E3 58 3D 31 30 33 31 20 D6 D5 B5 E3 59  终点 X=1031 终点 Y  
05941BFC  3D 32 36 36 38 20 C5 DC 3D 31                    =2668 跑=1.  
好了,到这里已经知道角色的跑位值,接下来,我们试试修改终点值,看看它能不能跑到预想的位置,,我改成 0 看看,哦,可以,不断向地图边缘跑,不过由于游戏的关系,在场景卡死了,哈哈.没问题,知道这里是控制角色移动的就行,经过多次断,发现其它角色的移动也是通过这里的  
我们只要判断第 1 个参数是不是我们需要的 ID,接着更改路线就行了.  
跑的部分完成.这个游戏要做自动的话要从人物提示那里获取 x,y 值,但那些 x,y 和这个终点的坐标不同,差距值是乘 20,Y 值的对应是逆向的,发挥你们的智慧吧,要解决是很简单的.  
跑,对话,两部分已经完成,差不多要写个测试程序去看看成果了,接下来,我们来搜索定位内存和给程序打补丁.  
送大家一段我自己的内存定位搜索代码,用 SEH 错误处理搜索的,来,上代码,不止用在这程序,用在其它用途也很大的.  
\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_  
  
SStart    dd  401000h  
  
\_Handler  proc  \_lpExceptionRecord,\_lpSEH,\_lpContext,\_lpDispatcherContext  
  
    pushad  
    mov  edi,\_lpContext  
    mov  eax,pERRORGO  
    mov  \[edi.CONTEXT\].regEip,eax  
    mov  VSize,999h  
    popad  
    mov  eax,ExceptionContinueExecution  
    ret  
  
\_Handler  endp  
  
\_SMem    proc  \_pV1,\_Size  ;ebx 返回搜索开始位置,edi 返回搜索结束位置+1   by:nowhy  
      
    assume  fs:nothing  
    push  offset \_Handler  
    push  fs:\[0\]  
    mov  fs:\[0\],esp  
    lea  eax,\_ERRORTONEXT  
    mov  pERRORGO,eax  
    mov  ebx,SStart  
    \_ERRORTONEXT:  
    add  ebx,VSize  
    mov  VSize,0  
    \_NEXTGO1:  
    mov  ecx,\_Size  
    mov  esi,\_pV1  
    inc  ebx  
    mov  edi,ebx  
    \_NEXTCMP:  
    mov  al,byte ptr \[edi\]  
    cmp  al,byte ptr \[esi\]  
    jnz  \_NEXTGO1  
    inc  esi  
    inc  edi  
    dec  ecx  
    test  ecx,ecx  
    jnz  \_NEXTCMP  
    pop  fs:\[0\]  
    pop  eax  
  ret  
  
\_SMem endp  
\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_  
  
调用方式,\_pV1 指向要搜索的内存判断地址,\_Size 为该段的长度,由于没写搜索失败后的参数返回.所以小心使用,哈哈.上面有说明返回值的返回方式  
上面代码慢慢消化吧,不算复杂. 接着关键点<如何在程序中准确定位到内存>  
我就拿调试的'按钮'那里开始说明.开 OD,看代码.介绍我的变态定位方式.  
00475654  |.  E8 37BA0200   call    004A1090  
00475659  |.  8B4C24 4C     mov     ecx, dword ptr \[esp+4C\]  
0047565D  |.  83C4 34       add     esp, 34  
00475660  |.  53            push    ebx  
00475661  |.  6A 07         push    7  
00475663  |.  51            push    ecx  
00475664  |.  894424 30     mov     dword ptr \[esp+30\], eax  
00475668  |.  E8 23BA0200   call    004A1090  
0047566D  |.  8B5424 28     mov     edx, dword ptr \[esp+28\]  
00475671  |.  8B4C24 30     mov     ecx, dword ptr \[esp+30\]  
00475675  |.  83C4 08       add     esp, 8  
00475678  |.  50            push    eax  
00475679  |.  8B4424 28     mov     eax, dword ptr \[esp+28\]  
0047567D  |.  52            push    edx  
0047567E  |.  57            push    edi  
0047567F  |.  56            push    esi  
00475680  |.  50            push    eax  
00475681  |.  51            push    ecx  
00475682  |.  8D5424 2C     lea     edx, dword ptr \[esp+2C\]  
00475686  |.  68 68B15500   push    0055B168  
0047568B  |.  52            push    edx  
0047568C  |.  E8 2F900400   call    004BE6C0  
00475691  |.  83C4 24       add     esp, 24  
00475694  |.  8D4C24 28     lea     ecx, dword ptr \[esp+28\]  
00475698  |.  C64424 6C 01  mov     byte ptr \[esp+6C\], 1  
0047569D  |.  E8 6E840400   call    004BDB10  
  
就上面那段说明,0047568C 我们要定位到这里,怎么定位好呢? 我给的建议是,避开全局变量,因为重新编译程序那会改动的几率太大了,接着避开局部变量,也就是 lea     edx, dword ptr \[esp+2C\]这种,那里函数定义的局部变量一改变就变化了,  
那么,我们定位的主要就只要.  
0047565D  |.  83C4 34       add     esp, 34  
00475660  |.  53            push    ebx  
00475661  |.  6A 07         push    7  
00475663  |.  51            push    ecx  
这里的几个字节,对于程序来说,是不容易改变的.   
83 C4 34 53 6A 07 51  
我们用这个来 CALL 上面的搜索函数.  
incoke  \_SMem,lp,7  
返回值 ebx 是搜索开始的点,edi 是搜索结束也就是搜索内存尾部的位置.直接 edi+28h 那样的话,就能定位到 0047568C 如果实在找不到那些指令点,看见了我上面定义的那个 SStart,按照出现次数压入覆盖调用就能定位到了  
接着要做什么?呵呵,对它写入呢...把它的 CALL HOOK 到我们的地址.我随便说明一下,  
call    004BE6C0  
比如要 HOOK 这个地址.我们的替换函数地址为 edi,这个地址位置为 ebx.  
mov  eax,dword ptr \[ebx+1\]  
lea  eax,dword ptr \[eax+ebx+5\]  
这样就能从地址得到它的目标要 CALL 的地址,eax=4BE6C0,将 EAX 到 HOOK 函数返回地址那里位置,预留 5 字节,先 0e9h(jmp)之后计算目标位置.填充进后面的 4 字节,那么程序 HOOK 处理完成后就 JMP 跳到游戏原来 CALL 的位置.  
接着把 CALL 的地址替换为我们的处理地址位置.  
sub  edi,ebx  
sub  edi,5  
mov  dword ptr \[ebx+1\],edi  
接着程序跑到我们 HOOK 的地址,该干嘛干嘛去,记得寄存器不要改变,堆栈也是.  
pushad  
处理....  
popad  
db 0e9h  
dd 0  ;这句是返回计算压入的位置,就是上面 eax 计算得出的  
搞掂...那么大家努力,明天给大家发针对这个游戏更加细化的处理方式.  
  
喜欢汇编的理由?? 够自由啊,,我爱怎么改就怎么改,喜欢怎么搞寄存器就怎么搞.比那些高级语言的自由度大多了,总体来说也比高级语言更简单,直接,有人要反驳我也没办法.我只是想简单明白的知道程序到底在干什么,要怎么做,有可能出现哪些问题.  
这些高级语言是做不到的.不过要考虑的也比高级语言的多很多,所以,个人喜欢啦.哈哈. 其实我也会用 C/C++来写程序. 

2010 年 11 月 2 日...  
  
鉴于有同学直接爆出了是哪个游戏..我没办法继续分析下去了..这次附件附上我的调试日志,大家下回来自己分析好了,  
由于游戏的多次更新,里面除了定位的数据外,地址基本都是错误的,自己按照上面的定位字节和说明去用 OD 搜索地址位查看分析地址,  
  
调试日志我删除了游戏保护的调试部分,这个我只能说不好意思.具体分析在日志内都有说明,我就此结束分析说明.谢谢大家支持.  
迟点在安全编程区发个我自己写的程序保护驱动让大家玩玩,呵呵.就是第 1 篇所说的那个进程启动限制的程序.  
  
真看不懂可以 PM 下我,我可以做简单的说明,个人化的东西,哈\`\`调试日志有点乱,因为我调试时游戏有个强壳保护,将就下看吧..

上传的附件
| ![](img/30b3588c36e23afc1abe1f92c2cce6e9.jpg) || 大话调试.txt  
 |