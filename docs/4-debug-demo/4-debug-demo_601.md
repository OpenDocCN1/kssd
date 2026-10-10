

# FoxitReaderPro PDF

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-305.htm`](https://www.pediy.com/kssd/pediy08/pediy8-305.htm)

FoxitReaderPro 官方最新版本 1.3,它是一款小巧绿色不用安装即使系统重装,只要从新启动一下就 ok 的 PDF 浏览器.  
我一直在用它.可是每当看到它的窗口顶部出现个广告条就很不舒服,今天就与它来个亲密接触吧,舒服一下...  
原程序用 Microsoft Visual C++ MFC 编写的，这里我一边分析汇编,一边分析 MFC 类库.水平有限错误之处,不吝指！  
\-----qiweixue  
  
下边要跟踪几个 MFC 核心类库:CObject-->CCmdTarget-->CWnd-->CFrameWnd,CWinApp 等...  
  
第一：找窗口类,消息回调的路线地址请根据 CWinApp 类.当然也可以从汇编代码层开始我就是.这里我不仔细说看分析就可以！  
第二：根据 MFC 注册的消息回调处理函数,筛选出自己想要的消息处理句柄来,分析如下：  
  
先来个 MFC 类层次调用继承关系,如下就是 MFC Message Dispatch and Message Routing：  
AfxWndProc-->AfxCallWndProc-->CWnd::WindowProc--->CWnd::OnWndMsg--->CFrameWnd::OnCommand-->CWnd::OnCommand-->  
CFrameWnd::OnCmdMsg-->CView::OnCmdMsg->CDocument::OnCmdMsg-->CCmdTarget::DispatchCmdMsg-->终点您亲密接触广告条吧.  
  
第一站：  
LRESULT CALLBACK AfxWndProc(HWND hWnd, UINT nMsg, WPARAM wParam, LPARAM lParam)  
在 AfxWndProc 中有个特殊的消息处理句柄,它 ID 号为 360,在 MFC 宏定义为 WM\_QUERYAFXWNDPROC,然后它就可以调用 AfxCallWndProc.  
  
00555BE5  /.  55            push    ebp  
00555BE6  |.  8BEC          mov     ebp, esp  
00555BE8  |.  817D 0C 60030>cmp     dword ptr \[ebp+C\], 360 --->\[ebp+C\]是存放消息的寄存器，消息 ID：360 对应的宏为：WM\_QUERYAFXWNDPROC       
00555BEF  |.  56            push    esi  
00555BF0  |.  75 05         jnz     short 00555BF7--->不是就跳  
00555BF2  |.  6A 01         push    1  --->参数成功置 1,压栈  
00555BF4  |.  58            pop     eax--->eax 存放的处理 WM\_QUERYAFXWNDPROC 的返回值.eax=1  
00555BF5  |.  EB 34         jmp     short 00555C2B-->返回 AfxWndProc  
00555BF7  |>  8B75 08       mov     esi, \[ebp+8\] --->\[ebp+C\]是存放的 hWnd 变量  
00555BFA  |.  56            push    esi  
00555BFB  |.  E8 5FFFFFFF   call    00555B5F---->函数：CWnd::FromHandlePermanent(hWnd)作用把一个 MFC 窗口对象影射一个 MFC 类 C++对象.  
00555C00  |.  85C0          test    eax, eax  
00555C02  |.  74 17         je      short 00555C1B ---如果 eax 返回的 C++对象指针为空,就转向默认处理方式  
00555C04  |.  3970 1C       cmp     \[eax+1C\], esi------------>VC 的中一个检测变量安全的宏类似:ASSERT(m\_hWnd == hWnd)  
00555C07  |.  75 12         jnz     short 00555C1B----以上意思是如果没有找到一个与 hWnd 对应的 C++对象(永久对象)就转向默认处理方式  
00555C09  |.  FF75 14       push    dword ptr \[ebp+14\] --->  lpararm  
00555C0C  |.  FF75 10       push    dword ptr \[ebp+10\] ---> wpaparm  
00555C0F  |.  FF75 0C       push    dword ptr \[ebp+C\]--->nMsg  
00555C12  |.  56            push    esi--->hWnd //00170144  
00555C13  |.  50            push    eax------->pWnd //00C97BA8   
00555C14  |.  E8 54FDFFFF   call    0055596D     ------>换角色了：AfxCallWndProc(pWnd, hWnd, nMsg, wParam, lParam);  
00555C19  |.  EB 10         jmp     short 00555C2B  
00555C1B  |>  FF75 14       push    dword ptr \[ebp+14\]   --- >lParam            
00555C1E  |.  FF75 10       push    dword ptr \[ebp+10\] ---> wParam               
00555C21  |.  FF75 0C       push    dword ptr \[ebp+C\] --->nMsg                
00555C24  |.  56            push    esi --->hWnd                               
00555C25  |.  FF15 CC165900 call    \[<&USER32.DefWindowProcA>\]  ---->没有找到一个与 hWnd 对应的 C++对象(永久对象)就调用我了.  
00555C2B  |>  5E            pop     esi  
00555C2C  |.  5D            pop     ebp  
00555C2D  \\.  C2 1000       retn    10  
第二站：  
AfxCallWndProc(pWnd, hWnd, nMsg, wParam, lParam);  
在这个函数中要为每一个线程构造\_AFX\_THREAD\_STATE 结构体,用到 MFC 全局变量\_afxThreadState,它的作用使得一个线程被安全局部化,  
另外它还处理 WM\_DESTROY，WM\_INITDIALOG 这两个消息,另外在调用 WindowProc(nMsg, wParam, lParam);  
0055596D   $  B8 90625800   mov     eax, 00586290--->新 SEH 指针  
00555972   .  E8 E528EFFF   call    0044825C--->构造新 SEH 栈  
00555977   .  83EC 34       sub     esp, 34  
0055597A   .  53            push    ebx //00000  
0055597B   .  56            push    esi---保存窗口句柄 hWnd  
0055597C   .  57            push    edi ///0000  
0055597D   .  B9 64C46000   mov     ecx, 0060C464 //全局变量 0060C464 为 pThreadState 结构指针...  
00555982   .  8965 F0       mov     \[ebp-10\], esp  
00555985   .  68 89185700   push    00571889 //全局变量 00571889 为\_afxThreadState  
0055598A   .  E8 D6EA0100   call    00574465--->调用\_afxThreadState.GetData();  
0055598F   .  8BD8          mov     ebx, eax  
00555991   .  6A 07         push    7  
00555993   .  59            pop     ecx  
00555994   .  8D7D C0       lea     edi, \[ebp-40\]  
00555997   .  8D43 34       lea     eax, \[ebx+34\]  
0055599A   .  8365 FC 00    and     dword ptr \[ebp-4\], 0  
0055599E   .  8BF0          mov     esi, eax  
005559A0   .  895D EC       mov     \[ebp-14\], ebx  
005559A3   .  F3:A5         rep     movs dword ptr es:\[edi\], dword p>  
005559A5   .  8B4D 0C       mov     ecx, \[ebp+C\]  
005559A8   .  8B75 10       mov     esi, \[ebp+10\]-->\[ebp+10\]  
005559AB   .  8B7D 08       mov     edi, \[ebp+8\]   
005559AE   .  8908          mov     \[eax\], ecx  
005559B0   .  8B45 14       mov     eax, \[ebp+14\]  
005559B3   .  83FE 02       cmp     esi, 2--------------------->处理 WM\_DESTROY 消息句柄  
005559B6   .  8943 3C       mov     \[ebx+3C\], eax  
005559B9   .  8B45 18       mov     eax, \[ebp+18\]  
005559BC   .  8973 38       mov     \[ebx+38\], esi  
005559BF   .  8943 40       mov     \[ebx+40\], eax  
005559C2   .  75 0E         jnz     short 005559D2  
005559C4   .  8B4F 34       mov     ecx, \[edi+34\]  
005559C7   .  85C9          test    ecx, ecx  
005559C9   .  74 07         je      short 005559D2  
005559CB   .  8B01          mov     eax, \[ecx\]  
005559CD   .  6A 00         push    0  
005559CF   .  FF50 5C       call    \[eax+5C\]------>pWnd->m\_pCtrlCont->OnUIActivate(NULL);  
005559D2   >  8365 0C 00    and     dword ptr \[ebp+C\], 0  
005559D6   .  81FE 10010000 cmp     esi, 110--------------->处理 WM\_INITDIALOG 消息句柄  
005559DC   .  75 0E         jnz     short 005559EC  
005559DE   .  8D45 0C       lea     eax, \[ebp+C\]  
005559E1   .  50            push    eax  
005559E2   .  8D45 DC       lea     eax, \[ebp-24\]  
005559E5   .  50            push    eax  
005559E6   .  57            push    edi  
005559E7   .  E8 0EFEFFFF   call    005557FA----->调用\_AfxPreInitDialog(pWnd, rectOld, dwStyle);  
005559EC   >  FF75 18       push    dword ptr \[ebp+18\]--->lParam  
005559EF   .  8B07          mov     eax, \[edi\]  
005559F1   .  8BCF          mov     ecx, edi--->edi 把 CWnd 的对象指针给 ecx  
005559F3   .  FF75 14       push    dword ptr \[ebp+14\]--->wParam  
005559F6   .  56            push    esi--->nMsg  
005559F7   .  FF90 98000000 call    \[eax+98\] ------------>在这里调用 CWnd::WindowProc(nMsg, wParam, lParam);                 
005559FD   .  81FE 10010000 cmp     esi, 110-------------->处理 WM\_INITDIALOG 消息句柄  
00555A03   .  8945 08       mov     \[ebp+8\], eax  
00555A06   .  75 43         jnz     short 00555A4B  
00555A08   .  FF75 0C       push    dword ptr \[ebp+C\]                  
00555A0B   .  8D45 DC       lea     eax, \[ebp-24\]                      
00555A0E   .  50            push    eax                               
00555A0F   .  57            push    edi                                
00555A10   .  E8 08FEFFFF   call    0055581D  ---->调用\_AfxPostInitDialog(pWnd, rectOld, dwStyle);                   
00555A15   .  EB 34         jmp     short 00555A4B  
  
第三站：  
  
LRESULT CWnd::WindowProc(UINT message, WPARAM wParam, LPARAM lParam)  
在这个函数中先调用 OnWndMsg(message, wParam, lParam, &lResult)函数来过滤处理消息句柄.如果程序中没有此消息然后返回  
调用 DefWindowProc(UINT message, WPARAM wParam, LPARAM lParam)函数来默认处理.这两个函数都是虚拟函数.  
      
  
00556A8D  /$  55            push    ebp  
00556A8E  |.  8BEC          mov     ebp, esp  
00556A90  |.  51            push    ecx  
00556A91  |.  56            push    esi  
00556A92  |.  8BF1          mov     esi, ecx---->ecx 把 CWnd 对象指针给 esi  
00556A94  |.  8D4D FC       lea     ecx, \[ebp-4\]   
00556A97  |.  8365 FC 00    and     dword ptr \[ebp-4\], 0--->并且把它内容清零,用来返回函数结果用  
00556A9B  |.  8B06          mov     eax, \[esi\]-->取虚拟表指针  
00556A9D  |.  51            push    ecx---->&lResult  
00556A9E  |.  FF75 10       push    dword ptr \[ebp+10\]----->消息字参数 lParam  
00556AA1  |.  8BCE          mov     ecx, esi--->对象指针又在传递之中.  
00556AA3  |.  FF75 0C       push    dword ptr \[ebp+C\]----->消息字参数 wParam,高字节是通知 code,低字节为 ID 表识  
00556AA6  |.  FF75 08       push    dword ptr \[ebp+8\]--->消息参数 message  
00556AA9  |.  FF90 9C000000 call    \[eax+9C\]---------->开始调用 CWnd::OnWndMsg(message, wParam, lParam, &lResult)虚函数  
00556AAF  |.  85C0          test    eax, eax  
00556AB1  |.  75 16         jnz     short 00556AC9  
00556AB3  |.  FF75 10       push    dword ptr \[ebp+10\] //lParam  
00556AB6  |.  8B06          mov     eax, \[esi\]  
00556AB8  |.  8BCE          mov     ecx, esi--->esi 把 CWnd 对象指针给 ecx  
00556ABA  |.  FF75 0C       push    dword ptr \[ebp+C\]//wParam  
00556ABD  |.  FF75 08       push    dword ptr \[ebp+8\] //message  
00556AC0  |.  FF90 A0000000 call    \[eax+A0\]-------->默认处理虚函数：CWnd::DefWindowProc(message, wParam, lParam);  
00556AC6  |.  8945 FC       mov     \[ebp-4\], eax  
00556AC9  |>  8B45 FC       mov     eax, \[ebp-4\]  
00556ACC  |.  5E            pop     esi  
00556ACD  |.  C9            leave  
00556ACE  \\.  C2 0C00       retn    0C  
  
第四站：  
  
进入 CWnd::OnWndMsg 消息处理虚函数  
  
BOOL CWnd::OnWndMsg(UINT message, WPARAM wParam, LPARAM lParam, LRESULT\* pResult)  
它主要处理 WM\_COMMAN,WM\_NOTIFY, WM\_ACTIVATE,WM\_SETCURSOR 这四个特殊消息句柄,另外在查找匹配处理消息影射表中的事件句柄.  
要经过：OnCommand(wParam, lParam)、OnNotify(wParam, lParam, &lResult)等过程...  
  
00556AD1  /$  B8 08635800   mov     eax, 00586308        -------------->SEH 新指针  
00556AD6  |.  E8 8117EFFF   call    0044825C                          ;构造新 SEH 框架函数  
00556ADB  |.  83EC 54       sub     esp, 54 ---------->分配 54 字节栈  
00556ADE  |.  8365 F0 00    and     dword ptr \[ebp-10\], 0  ----->初始化指针变量\*pResult 的为 0  
00556AE2  |.  53            push    ebx  
00556AE3  |.  8B5D 08       mov     ebx, \[ebp+8\]    ----------- > \[ebp+8\] 为消息 Msg 结构体的 Message 域里边存放的是消息 ID 号  
00556AE6  |.  56            push    esi  
00556AE7  |.  57            push    edi  
00556AE8  |.  81FB 11010000 cmp     ebx, 111  ---->注意这里:111 为 WM\_COMMAND 消息 ID 号,开始进行消息判断  
00556AEE  |.  8BF9          mov     edi, ecx  ------>ecx 是所谓的 this 指针...  
00556AF0  |.  75 18         jnz     short 00556B0A--->如果不是 WM\_COMMAND 就跳过去.  
00556AF2  |.  FF75 10       push    dword ptr \[ebp+10\]                ;  Case 111 (WM\_COMMAND) of switch 00556AE8  
00556AF5  |.  8B07          mov     eax, \[edi\]  ---->  \[edi\] 指向一个  MFC MainFrame 对象,它把对象指针传给 eax,给下边的虚函数用  
00556AF7  |.  FF75 0C       push    dword ptr \[ebp+C\]---------->注意这里：菜单 ID 号入栈了...这里是：8024  
00556AFA      FF50 78       call    \[eax+78\]--------->这是 CFrameWnd::OnCommand（wparam,lparam）虚函数  
00556AFD  |.  85C0          test    eax, eax -->eax 存放函数返回值,把结果放到\*pResult 指针变量中.  
00556AFF  |.  0F84 55010000 je      00556C5A-->当 eax 为 0,转  
00556B05  |.  E9 1D040000   jmp     00556F27-->当 eax 为 1,转  
00556B0A  |>  83FB 4E       cmp     ebx, 4E----------------------------------------->这里 4E 为 WM\_NOTIFY 消息 ID 号,  
00556B0D  |.  75 28         jnz     short 00556B37  
00556B0F  |.  8B45 10       mov     eax, \[ebp+10\]                      
00556B12  |.  8338 00       cmp     dword ptr \[eax\], 0  
00556B15  |.  0F84 3F010000 je      00556C5A--->--->如果不是 WM\_NOTIFY 就跳到 WM\_ACTIVATE 消息处理  
00556B1B  |.  8B17          mov     edx, \[edi\]  
00556B1D  |.  8D4D F0       lea     ecx, \[ebp-10\]---->&lResult 指针地址保存到 ecx 中压栈传参用.  
00556B20  |.  51            push    ecx  
00556B21  |.  50            push    eax  
00556B22  |.  FF75 0C       push    dword ptr \[ebp+C\]  
00556B25  |.  8BCF          mov     ecx, edi  
00556B27  |.  FF52 7C       call    \[edx+7C\]---->这是 CWnd::OnNotify(wParam,lparam,&lResult)虚函数  
00556B2A  |> /85C0          test    eax, eax  
00556B2C  |. |0F85 39040000 jnz     00556F6B  
00556B32  |. |E9 23010000   jmp     00556C5A  
  
第五站：  
因为我这里要处理的是 WM\_COMMAND 的消息嘛，所以进入：  
BOOL CFrameWnd::OnCommand(WPARAM wParam, LPARAM lParam)  
对了还有一个参数就是菜单的 ID 号：80A2,存放在 wParam 低字节中.  
恩,是这样得当我用鼠标触发那个广告菜单命令的时候,广告就出现在菜单工具栏上,所以菜单 ID 不能少！  
还有个关键问题：  
FoxitReaderPro 对象好象重载了 OnCommand(WPARAM wParam, LPARAM lParam)方法,那以后的消息路线转变了：  
应该是调用 CFoxitReaderFrameWnd::OnCommand 然后在调用 FrameWnd::OnCommand 和 CWnd::OnCommand 方法.在普通的 MFC 类库中 FrameWnd::OnCommand 里边没有很多实例方法,它是直接调用 CWnd::OnCommand  
  
  
0056DD6D   .  53            push    ebx-->：CWnd::OnWndMsg 函数中的 message 参数变量,这里是 WM\_COMMAND 消息 ID  
0056DD6E   .  56            push    esi-->保存对象指针 00C97BA8  
0056DD6F   .  57            push    edi-->保存对象指针 00C97BA8，  
0056DD70   .  8BF9          mov     edi, ecx ---->ecx 与 edi 指向同一对象指针  
0056DD72   .  6A 00         push    0  
0056DD74   .  E8 4B040000   call    0056E1C4 ---> 调用 CWnd::GetTopLevelFrame(),这个函数发了 WM\_MDIGETACTIVE 消息得到当前 MDI Child Window  
0056DD79   .  8B5C24 10     mov     ebx, \[esp+10\]  ---> 菜单 ID 号出现 80A2  
0056DD7D   .  BE 11010000   mov     esi, 111  
0056DD82   .  85C0          test    eax, eax ->函数的返回值,由于广告条在父窗口中并没有在子类或者其他 CView 类出现,所以我把所以子窗口全部 shutdown 了,免的又在阴沟 over 了！  
0056DD84   .  74 13         je      short 0056DD99------>在这里跳！  
0056DD86   .  FF7424 14     push    dword ptr \[esp+14\]  
0056DD8A   .  53            push    ebx  
0056DD8B   .  56            push    esi  
0056DD8C   .  FF70 1C       push    dword ptr \[eax+1C\]  
0056DD8F   .  50            push    eax  
0056DD90   .  E8 D87BFEFF   call    0055596D--->注意这个虚函数中有很多对象实例方法里边判断广告的显示方法也在其中了,通过分析它才知道 FoxitReaderPro 对象应该重载了 OnCommand(WPARAM wParam, LPARAM lParam)方法。  
0056DD95   .  85C0          test    eax, eax  
0056DD97   .  75 30         jnz     short 0056DDC9  
0056DD99   >  FF7424 14     push    dword ptr \[esp+14\]---//这个变量为 0  
0056DD9D   .  8BCF          mov     ecx, edi--->赋值 CFrameWnd 对象指针.  
0056DD9F   .  53            push    ebx        ----->触发菜单 ID 号：8042                        
0056DDA0   .  E8 4BE1FFFF   call    0056BEF0    ---------------------------来到这里是通往广告条的显示之门.  
0056DDA5   .  85C0          test    eax, eax  
0056DDA7   .    
  
进入 FoxitReaderPro 判断广告条的实例方法：  call    0056BEF0 (wParam,lParam)  
  
  
0056BEF0  /$  53            push    ebx  
0056BEF1  |.  56            push    esi  
0056BEF2  |.  57            push    edi  
0056BEF3  |.  8BF1          mov     esi, ecx--->对象指针.  
0056BEF5  |.  0FB77C24 10   movzx   edi, word ptr \[esp+10\] ----》 触发广告条菜单 ID 号： 8024  
0056BEFA  |.  E8 D7B3FEFF   call    005572D6                                  ;  c++对象指针与窗口句柄影射  
0056BEFF  |.  33C9          xor     ecx, ecx  
0056BF01  |.  3948 50       cmp     \[eax+50\], ecx  
0056BF04  |.  74 4C         je      short 0056BF52--->走到了,开始跳了  
0056BF06  |.  394C24 14     cmp     \[esp+14\], ecx  
0056BF0A  |.  75 46         jnz     short 0056BF52  
0056BF0C  |.  81FF 46E10000 cmp     edi, 0E146   --->其他彩单 ID 0E146  
0056BF12  |.  74 3E         je      short 0056BF52  
0056BF14  |.  BB 47E10000   mov     ebx, 0E147  --->其 菜单 ID E147  
0056BF19  |.  3BFB          cmp     edi, ebx  
0056BF1B  |.  74 35         je      short 0056BF52  
0056BF1D  |.  81FF 45E10000 cmp     edi, 0E145  
0056BF23  |.  74 2D         je      short 0056BF52  
  
...中间省略了不必要的代码看关键部分  
从上边跳过来的.  
0056BF52  |> \\FF7424 14     push    dword ptr \[esp+14\]  |Arg1---> 这个参数是我触发显示广告菜单的 ID 号：8042                     
0056BF56  |.  8BCE          mov     ecx, esi                                   
0056BF58  |.  FF7424 14     push    dword ptr \[esp+14\]  |Arg2--->大家注意拉！看堆栈！                      
0056BF5C  |.  E8 0FB1FEFF   call    00557070 ============>这是关键广告函数,真是没有想到 FoxitReaderPro 给这个 ID 号另起炉灶了！  
0056BF61  |>  5F            pop     edi  
0056BF62  |.  5E            pop     esi  
0056BF63  |.  5B            pop     ebx  
0056BF64  \\.  C2 0800       retn    8  
  
看堆栈：  
0012FB40   000080A2  |Arg1 = 000080A2 =====触发菜单 ID 号：8042，这个参数是 wParam  
0012FB44   00000000  |Arg2 = 00000000  
  
  
  
00557070  /$  55            push    ebp  
00557071  |.  8BEC          mov     ebp, esp  
00557073  |.  83EC 2C       sub     esp, 2C  
00557076  |.  8B45 08       mov     eax, \[ebp+8\] ---》wParam 参数  
00557079  |.  53            push    ebx  
0055707A  |.  56            push    esi  
0055707B  |.  57            push    edi  
0055707C  |.  0FB7F8        movzx   edi, ax  
0055707F  |.  33DB          xor     ebx, ebx  
00557081  |.  8BF1          mov     esi, ecx  
00557083  |.  C1E8 10       shr     eax, 10  ----把 wParam 右移动 16 位,这时候 wParam 变为 0 了.  
00557086  |.  395D 0C       cmp     \[ebp+C\], ebx---->\[ebp+c\]  
00557089  |.  8945 08       mov     \[ebp+8\], eax  
0055708C  |.  75 3A         jnz     short 005570C8---》到这里相当跳.  
0055708E  |.  3BFB          cmp     edi, ebx  
00557090  |.  74 66         je      short 005570F8  
00557092  |.  8D4D D4       lea     ecx, \[ebp-2C\]  
00557095  |.  E8 A9FFFFFF   call    00557043  
0055709A  |.  8B06          mov     eax, \[esi\]  
0055709C  |.  8D4D D4       lea     ecx, \[ebp-2C\]                             ;  lpararm c++对象指针  
0055709F  |.  53            push    ebx  
005570A0  |.  51            push    ecx  
005570A1  |.  6A FF         push    -1  
005570A3  |.  57            push    edi                                       ;  彩单 ID  
005570A4  |.  8BCE          mov     ecx, esi  
005570A6  |.  897D D8       mov     \[ebp-28\], edi                             ;  80A2 菜单 ID 号...  
005570A9  |.  FF50 0C       call    \[eax+C\]                                   ;  a.0056ddd6   比较之处  
005570AC  |.  395D FC       cmp     \[ebp-4\], ebx  
005570AF  |.  74 3E         je      short 005570EF  
005570B1  |.  895D 08       mov     \[ebp+8\], ebx  
005570B4  |>  8B06          mov     eax, \[esi\]  
005570B6  |.  53            push    ebx  
005570B7  |.  53            push    ebx  
005570B8  |.  8BCE          mov     ecx, esi  
005570BA  |.  FF75 08       push    dword ptr \[ebp+8\]  
005570BD  |.  57            push    edi  
005570BE  |.  FF50 0C       call    \[eax+C\]       ------------------------》关键函数：  
005570C1  |>  5F            pop     edi  
005570C2  |.  5E            pop     esi  
005570C3  |.  5B            pop     ebx  
005570C4  |.  C9            leave  
005570C5  |.  C2 0800       retn    8  
  
  
第六站直接亲密  
.....  
我写这累了,相比你看这也累了,我直接给出结局来！  
  
原来的时候我是下 USER32.dll.ShowWindow 断点可是广告条根本就不吃这一套,断不下来的！还有当点击广告条的时候会进入它官方网站,这时候又下的 SHELL32.ShellExecute 断点等也没有断下来.只能断个 MessageBox,感觉交互信息太少,还有它的菜单 ID 迟迟不比较等等！直到最后跟到这里才恍然大悟...  
原来 FoxitReaderPro 是用的 InsertMenuA 插入广告条  
用 DeleteMenu 删除广告条  
用 DrawMenuBar 来刷新广告条的颜色,你会看到红,绿等色的交替！  
用 SetRectEmpty 来清理广告条的矩形对象  
如下：  
  
0040C9C0  /$  56            push    esi  
0040C9C1  |.  57            push    edi  
0040C9C2  |.  8BF9          mov     edi, ecx  
0040C9C4  |.  8B47 1C       mov     eax, \[edi+1C\]  
0040C9C7  |.  50            push    eax                                       ; /hWnd  
0040C9C8  |.  FF15 44175900 call    \[<&USER32.GetMenu>\]                       ; \\GetMenu  
0040C9CE  |.  50            push    eax  
0040C9CF  |.  E8 E5D21400   call    00559CB9  
0040C9D4  |.  8BF0          mov     esi, eax  
0040C9D6  |.  85F6          test    esi, esi  
0040C9D8      0F85 90000000 jnz     0040CA6E  
0040C9DE  |.  8B4424 0C     mov     eax, \[esp+C\]  
0040C9E2  |.  83F8 05       cmp     eax, 5  
0040C9E5  |.  75 40         jnz     short 0040CA27  
0040C9E7  |.  68 00410000   push    4100  
0040C9EC  |.  56            push    esi  
0040C9ED  |.  8BCF          mov     ecx, edi  
0040C9EF  |.  E8 3CFBFFFF   call    0040C530  
0040C9F4  |.  83F8 FF       cmp     eax, -1                                   ;  打开 advise....  
0040C9F7      75 61         jnz     short 0040CA5A  
0040C9F9  |.  8B4E 04       mov     ecx, \[esi+4\]  
0040C9FC  |.  68 0C875D00   push    005D870C                                  ; /pNewItem = "PDF 编?,AD,"?,F7,""  
0040CA01  |.  68 00410000   push    4100                                      ; |NewItemID = 4100 (16640.)  
0040CA06  |.  68 00450000   push    4500                                      ; |Flags = MF\_BYPOSITION|MF\_ENABLED|MF\_STRING|MF\_OWNERDRAW|MF\_HELP  
0040CA0B  |.  50            push    eax                                       ; |ItemID  
0040CA0C  |.  51            push    ecx                                       ; |hMenu  
0040CA0D  |.  FF15 28175900 call    \[<&USER32.InsertMenuA>\]                   ; \\InsertMenuA  
0040CA13  |.  8B4F 1C       mov     ecx, \[edi+1C\]  
0040CA16  |.  51            push    ecx                                       ; /hWnd  
0040CA17  |.  FF15 30175900 call    \[<&USER32.DrawMenuBar>\]                   ; \\DrawMenuBar  
0040CA1D      5F            pop     edi  
0040CA1E      B8 01000000   mov     eax, 1  
0040CA23  |.  5E            pop     esi  
0040CA24  |.  C2 0400       retn    4  
0040CA27  |>  85C0          test    eax, eax  
0040CA29  |.  75 2F         jnz     short 0040CA5A  
0040CA2B  |.  68 00410000   push    4100  
0040CA30  |.  56            push    esi  
0040CA31  |.  8BCF          mov     ecx, edi  
0040CA33  |.  E8 F8FAFFFF   call    0040C530  
0040CA38  |.  83F8 FF       cmp     eax, -1  
0040CA3B  |.  74 1D         je      short 0040CA5A  
0040CA3D  |.  8B56 04       mov     edx, \[esi+4\]  
0040CA40  |.  68 00040000   push    400                                       ; /Flags = MF\_BYPOSITION|MF\_ENABLED|MF\_STRING  
0040CA45  |.  50            push    eax                                       ; |ItemId  
0040CA46  |.  52            push    edx                                       ; |hMenu  
0040CA47  |.  FF15 34175900 call    \[<&USER32.DeleteMenu>\]                    ; \\DeleteMenu  
0040CA4D  |.  8D87 28020000 lea     eax, \[edi+228\]  
0040CA53  |.  50            push    eax                                       ; /pRect  
0040CA54  |.  FF15 B4175900 call    \[<&USER32.SetRectEmpty>\]                  ; \\SetRectEmpty  
0040CA5A  |>  8B4F 1C       mov     ecx, \[edi+1C\]  
0040CA5D  |.  51            push    ecx                                       ; /hWnd  
0040CA5E  |.  FF15 30175900 call    \[<&USER32.DrawMenuBar>\]                   ; \\DrawMenuBar  
0040CA64  |.  5F            pop     edi  
0040CA65  |.  B8 01000000   mov     eax, 1  
0040CA6A  |.  5E            pop     esi  
0040CA6B  |.  C2 0400       retn    4  
0040CA6E  |>  5F            pop     edi  
0040CA6F  |.  33C0          xor     eax, eax  
0040CA71  |.  5E            pop     esi  
0040CA72  \\.  C2 0400       retn    4  
  
任务完成了,该找出来的都出来,到这里你稍微一动鼠标,广告就 over 掉了......  
末了：  
FoxitReaderPro 的做的广告条正是以菜单栏为父窗口,因此为什么当跟踪的时候始终没有看到影射出一个 CWnd 类或者是 CFarme 类的 C++对象,还有那些 ShowWindow 等窗体的断点 API 根本不管用的！  
  
另外 FoxitReaderPro 还有一个有意思的地方就是当你点击广告条的时候,它会启动浏览器然后加载它的主页地址.我一开始用的是 SHELL32.ShellExecute 断点,不管用！它用的&KERNEL32.CreateProcess,它把官方 URL 作为参数启动 IE,当触发事件的时候就执行了 CreateProcess.....  
  
好累了,直到写完了我才吃了点东西！  
  
Copyright © 2000 - 2006 PEdiy.com All Rights Reserved.By KanXue Studio 

 4dd466d0e1e0cc123afbe504da2462ca