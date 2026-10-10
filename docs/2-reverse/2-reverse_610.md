

# 逆向软件原理及功能重现之初步

> 原文：[`www.pediy.com/kssd/pediy10/74890.html`](https://www.pediy.com/kssd/pediy10/74890.html)

【文章标题】: 逆向软件原理及功能重现之初步  
【文章作者】: 书呆彭  
【作者邮箱】: pengzhicheng1986@gmail.com  
【软件名称】: 魔兽争霸显血工具  
【下载地址】: http://hi.baidu.com/pylzj/blog/item/ca25a6af4f3736c97dd92a67.html  
【加壳方式】: UPX 0.89.6 - 1.02 / 1.05 - 1.24 -> Markus & Laszlo  
【编写语言】: VB 5/6  
【使用工具】: PEiD OD VC60 MSDN60  
【软件介绍】: 一个魔兽争霸的小辅助工具，非作弊性质。  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
  论坛上有人发了这个工具，请求帮助，正好无事，拿来看了一下。  
    
  该工具目前已经有新版本，作者吕志杰，他的空间是 http://hi.baidu.com/pylzj  
    
  对人家的工具分析了一次，就帮着宣传下，否则有点过意不去。  
    
  该工具其实很简单但实用。在玩魔兽争霸时，我们经常为了查看单位的血量而按 ALT 键，但 ALT 和 WIN 键是挨着的，结果很容易碰到 WIN 键而弹出游戏，非常影响。  
    
  该工具就是可以让单位的血条一直显示而不必按住 ALT，同时也提供了屏蔽 WIN 键的功能。  
  按 HOME 可以显示或隐藏自己单位的血量。按 END 是对方敌人单位的。  
    
  新版本还加入了强行退出游戏的功能，用来解决浩方对战平台玩时游戏无响应的问题（引用作者原话）。  
    
  我分析的这个版本作者留下的发布日期为 2008-03-06，请参见 http://bbs1.pediy.com/showthread.php?t=74866  
    
    
    
  好，分析开始。  
    
  首先,将它发送到 PEiD,告诉我 UPX 0.89.6 - 1.02 / 1.05 - 1.24 -> Markus & Laszlo  
  呵呵，因为本来就是个无私发布的工具，也不需要什么保护。  
  脱壳机伺候之。  
    
  脱壳之后，程序由 13.0K“长大”到了 40.3K。  
  再次用 PEiD 查看，Microsoft Visual Basic 5.0 / 6.0  
  用 P Code or Nativo 查看，PEiD 无响应，可能我的插件版本问题。  
    
  用 VB Decompiler Pro 处理，显示为 Native Code.  
    
  从左边的树形资源中可以看到，有两个类，frmMain 和 cHotKey.  
    
  cHotKey 可能是作者自己编写的或使用的第三方的类。  
    
  对 VB 不太熟悉，先试着用 OD 跟一下。  
    
    
  载入程序时有一个异常，忽略之（调试 VB 时会有大量的异常，习惯就好），来到入口点。  
    
  由于以前写程序，知道注册热键的 API,所以直接 bp RegisterHotKey， F9 运行，断下，堆栈窗口为：  
    
  0012F9D8   00405080  /CALL 到 RegisterHotKey 来自 显血工具.0040507B  
  0012F9DC   000203F2  |hWnd = 000203F2 ('关闭(&C)',class='ThunderRT6CommandButton',parent=000203FE)  
  0012F9E0   FFFFC038  |HotKeyID = FFFFC038  
  0012F9E4   00000000  |Modifiers = 0  
  0012F9E8   00000024  \\Key = VK\_HOME  
    
  可以看到下如所料，参数为 VK\_HOME  
    
  再一次运行，中断，堆栈为  
    
  0012F9D8   00405080  /CALL 到 RegisterHotKey 来自 显血工具.0040507B  
  0012F9DC   000203F2  |hWnd = 000203F2 ('关闭(&C)',class='ThunderRT6CommandButton',parent=000203FE)  
  0012F9E0   FFFFC037  |HotKeyID = FFFFC037  
  0012F9E4   00000000  |Modifiers = 0  
  0012F9E8   00000023  \\Key = VK\_END  
    
  好。这次是 VK\_END  
    
  再一次 F9 程序便运行起来，说明只注册了这两个热键。  
    
  我原本的思路是直接到与热键关联的窗口的窗口过程去分析，结果发现注册热键时用的窗口参数是 “关闭” 按钮。  
  并且转到该窗口过程下消息断点，按 HOME 键后并不中断。  
    
  换个思路。  
    
  回到 VB Decompiler，查看代码，发现 RegisterHotKey 是在 cHotKey 类的方法 AddHotKeys 中调用的。  
    
  也就是说 HotKey 的回调处理被 cHotKey 类封装起来了。由于对 VB 的类不懂，决定从其它地方下手。  
    
    
  我们知道 VB 中调用 VB 运行时的函数不需要声明，而调用其它函数需要进行声明。  
    
  我们就从他调用的 API 下手。  
    
  从 VB Decompiler 给出的结果看，它只引入了少量的外部 API,分别是  
  GlobalDeleteAtom  
  GlobalAddAtom  
  WaitMessage  
  PeekMessage  
  UnregisterHotKey  
  RegisterHotKey  
  CopyMemory  
  CallNextHookEx  
  UnhookWindowsHookEx  
  SetWindowsHookEx  
  keybd\_event  
  ShellExecute  
    
  跟 HotKey 有关的函数有 GlobalAddAtom,GlobalDeleteAtom,RegisterHotKey,UnregisterHotKey,没什么异样的  
  但从 PeekMessage 我猜到我之所以在 VB 的运行时中的窗口过程中断不下来，可能是因为 cHotKey 类处理 WM\_HOTKEY 消息的方法是 PeekMeesage 后直接处理，而没有 DispatchMessage.  
    
  从 RegisterHotKey 这条路不好走，不过我看到了感兴趣的东西，SetWindowsHookEx 和 keybd\_event  
    
  好吧，看看软件用这些 API 做了些什么。bp SetWindowsHookExA,运行程序，不中断  
    
  那我们做点什么。点选 “禁用 WIN 键”， 啪，OD 断到了。好，看一下参数：  
    
  0012F20C   00406A5A  /CALL 到 SetWindowsHookExA 来自 显血工具.00406A55  
  0012F210   0000000D  |HookType = 13.  
  0012F214   00406930  |Hookproc = 显血工具.00406930  
  0012F218   00400000  |hModule = 00400000 (显血工具)  
  0012F21C   00000000  \\ThreadID = 0  
    
  参照 MSDN，查看 WIN32 SDK 的头文件， HookType = 13 == WH\_KEYBOARD\_LL  
  MSDN 对这个参数的解释为：Installs a hook procedure that monitors low-level keyboard input events. For more information, see the LowLevelKeyboardProc hook procedure.  
    
  这是个低级键盘消息的系统钩子，我们来到参数指出的地址：00406930，看下这个钩子干了什么。  
  根据 MSDN,这个函数的原型为：  
  LRESULT CALLBACK LowLevelKeyboardProc(  
    int nCode,     // hook code  
    WPARAM wParam, // message identifier  
    LPARAM lParam  // pointer to structure with message data  
  );  
  我们需要由此分析堆栈。  
    
  注意：这里下断的话，每次按 F9 运行程序，结果每次都会断到这里。所以要么用鼠标点“运行”，要么干脆不要下断点。  
  我们分析代码（我去掉了对我们不太重要的机器码那一列，以使代码更清楚）：  
    
  00406930   >PUSH    EBX  
  00406931   >MOV     EBX, DWORD PTR SS:\[ESP+10\]       ; EBX 存放一个指针指向一个 KBDLLHOOKSTRUCT 的结构体  
  00406935   >PUSH    EBP  
  00406936   >MOV     EBP, DWORD PTR DS:\[<&MSVBVM60.\_\_>; MSVBVM60.\_\_vbaSetSystemError  
  0040693C   >PUSH    ESI  
  0040693D   >MOV     ESI, DWORD PTR SS:\[ESP+14\]  
  00406941   >PUSH    EDI  
  00406942   >MOV     EDI, DWORD PTR SS:\[ESP+14\]  
  00406946   >TEST    EDI, EDI  
  00406948   >JNZ     SHORT 显血工具.00406992           ; 这几条指令是简单地参数检查而已。  
  0040694A   >CMP     ESI, 100  
  00406950   >JE      SHORT 显血工具.0040696A  
  00406952   >CMP     ESI, 104  
  00406958   >JE      SHORT 显血工具.0040696A  
  0040695A   >CMP     ESI, 101  
  00406960   >JE      SHORT 显血工具.0040696A  
  00406962   >CMP     ESI, 105  
  00406968   >JNZ     SHORT 显血工具.00406992  
  0040696A   >PUSH    10                               ; nBytesToCopy = 0x10 ; == sizeof(KBDLLHOOKSTRUCT)  
  0040696C   >PUSH    EBX                              ; pSrc = EBX  
  0040696D   >PUSH    显血工具.00408024                    ; pDst = 408024  
  00406972   >CALL    显血工具.00402DD4                    ; 这个函数是 CopyMemory 函数的一外包装  
  00406977   >CALL    EBP                              ; \_\_vbaSetSystemError,这种 VB 运行时在 VB 程序中非常多，无视之  
  00406979   >MOV     EAX, DWORD PTR DS:\[408024\]       ; EAX = KBDLLHOOKSTRUCT.vkCode,刚复制过来的 KBDLLHOOKSTRUCT 结构体  
  0040697E   >CMP     EAX, 5B                          ; 0x5b == VK\_LWIN  
  00406981   >JL      SHORT 显血工具.00406992  
  00406983   >CMP     EAX, 5C                          ; 0x5c == VK\_RWIN  
  00406986   >JG      SHORT 显血工具.00406992  
  00406988   >POP     EDI  
  00406989   >POP     ESI  
  0040698A   >POP     EBP  
  0040698B   >OR      EAX, FFFFFFFF  
  0040698E   >POP     EBX  
  0040698F   >RETN    0C  
  00406992   >PUSH    EBX  
  00406993   >PUSH    ESI  
  00406994   >PUSH    EDI  
  00406995   >PUSH    0  
  00406997   >CALL    显血工具.00402D7C                    ; CallNextHookEx 的包装  
  0040699C   >MOV     ESI, EAX  
  0040699E   >CALL    EBP  
  004069A0   >MOV     EAX, ESI  
  004069A2   >POP     EDI  
  004069A3   >POP     ESI  
  004069A4   >POP     EBP  
  004069A5   >POP     EBX  
  004069A6   >RETN    0C  
    
  那么这个函数的功能已经非常明显了，就是屏蔽 WIN 键。它的伪代码是  
    
  if ( (LPKBDLLHOOKSTRUCT) (arg3) -> vkCode >= VK\_LWIN &&  
         (LPKBDLLHOOKSTRUCT) (arg3) -> vkCode <= VK\_RWIN)  
  {  
          return;  
  }  
    
  else  
    
  {  
          return CallNextHookEx(arg1,arg2,arg3);  
  }  
    
  这段代码非常常见，也不复杂。  
    
    
    
  好，那么现在只剩下 keybd\_event 函数了。  
    
  那么我们显示或隐藏血条的功能百分之九十以上是通过它实现的。  
    
  再看看 MSDN，The keybd\_event function synthesizes a keystroke  
    
  就是说这个函数是模拟一次键盘敲击。  
    
    
  究竟是什么，看一下就知道了。 bp keybd\_event,F9 运行起来。  
    
  我们试着按一下 HOME 键， 啪，果然被 OD 断下。  
    
  堆栈是这样的  
    
  0012EF10   00403F82  /CALL 到 keybd\_event 来自 显血工具.00403F7D  
  0012EF14   000000DB  |Key = DB  
  0012EF18   00000000  |ScanCode = 0  
  0012EF1C   00000000  |Flags = 0  
  0012EF20   00000000  \\ExtraInfo = 0  
    
  按 F9 再运行，再按 HOME,又被断下，栈区参数有一个发生变化，如下  
    
  0012EF10   00403F82  /CALL 到 keybd\_event 来自 显血工具.00403F7D  
  0012EF14   000000DB  |Key = DB  
  0012EF18   00000000  |ScanCode = 0  
  0012EF1C   00000002  |Flags = KEYEVENTF\_KEYUP  
  0012EF20   00000000  \\ExtraInfo = 0  
    
  其中参数 Flags 在两次调用时不相同。反复按 HOME,这个参数就在这两个值之间变化。  
  另外，按 END 键时参数 Key = 0xDD,其它都相同。  
    
  MSDN:KEYEVENTF\_KEYUP If specified, the key is being released. If not specified, the key is being depressed.   
    
  那么就非常清晰了，当你按下这个键，就显示血量，当你松开这个键，就不显示。  
  那么这个键是键盘上的哪个键呢？？？  
  查找 MSDN,0xDB,0xDD,没有对应的 VK\_XXXX 的常量定义，而是  
   DBE4          OEM specific   
    
  哦，这两个键不对应于键盘，是真正的“虚拟”键码了，呵呵。  
    
  那么就是说，如果键盘上有某个键，它的键码是 0xDB,那么这个键在玩魔兽时就是用来显示自己单位血条的，而 0xDD 是敌人的。  
  OEM Specific,那么暴雪就利用了这两个键码。  
    
  好吧，原来是这么简单的东西，那就自己写个程序验证一下。屏蔽 Win 键的因为有很多人都说过，也比较公开，就不写了，我只是简单地写了个显示和隐藏血条的功能。  
  在 VC60 下编译，运行，进入魔兽争霸，按一下 HOME,自己单位的血条显示出来了。按下 END,敌人也成功。  
    
  以下是写的代码，比较丑陋，大家不要见笑。  
    
    
  // 血条显示.cpp : Defines the entry point for the application.  
  //  
  #include <windows.h>  
    
    
  HINSTANCE  hInst;              // current instance  
  TCHAR    szTitle\[\]  = "Demo";        // The title bar text  
  TCHAR    szWindowClass\[\]  = "DemoClass";        // The title bar text  
  TCHAR    szHello\[\]  = "按 HOME 键显示或隐藏自己的血条\\n 按 END 键显示或隐藏敌人的血条\\n 对魔兽争霸有效";  
    
  // Foward declarations of functions included in this code module:  
  ATOM      MyRegisterClass(HINSTANCE hInstance);  
  LRESULT CALLBACK  WndProc(HWND, UINT, WPARAM, LPARAM);  
  LRESULT CALLBACK  About(HWND, UINT, WPARAM, LPARAM);  
    
  int APIENTRY WinMain(HINSTANCE hInstance,  
                       HINSTANCE hPrevInstance,  
                       LPSTR     lpCmdLine,  
                       int       nCmdShow)  
  {  
     // TODO: Place code here.  
    hInst = hInstance; // Store instance handle in our global variable  
    MSG msg;  
    
    ATOM homeAtom  = ::GlobalAddAtom( "HOME" );  
    ATOM delAtom  = ::GlobalAddAtom( "DEL"  );  
    
    WNDCLASSEX wcex;  
    
    wcex.cbSize = sizeof(WNDCLASSEX);   
    
    wcex.style    = CS\_HREDRAW | CS\_VREDRAW;  
    wcex.lpfnWndProc  = (WNDPROC)WndProc;  
    wcex.cbClsExtra    = 0;  
    wcex.cbWndExtra    = 0;  
    wcex.hInstance    = hInstance;  
    wcex.hIcon    = 0;  
    wcex.hCursor    = 0;  
    wcex.hbrBackground  = (HBRUSH)(COLOR\_WINDOW+1);  
    wcex.lpszMenuName  = 0;  
    wcex.lpszClassName  = szWindowClass;  
    wcex.hIconSm    = 0;  
    
    ::RegisterClassEx(&wcex);  
    HWND hWnd  = ::CreateWindow(szWindowClass, szTitle, WS\_OVERLAPPEDWINDOW,  
          CW\_USEDEFAULT, 0, CW\_USEDEFAULT, 0, NULL, NULL, hInstance, NULL);  
    
    
    ::RegisterHotKey( hWnd, homeAtom, 0, VK\_HOME );  
    ::RegisterHotKey( hWnd, delAtom,  0, VK\_END  );  
    
    ShowWindow(hWnd, nCmdShow);  
    UpdateWindow(hWnd);  
    
    
    // Main message loop:  
    while (GetMessage(&msg, NULL, 0, 0))   
    {  
      if ( msg.message == WM\_DESTROY )  
      {  
        ::UnregisterHotKey( hWnd, homeAtom );  
        ::UnregisterHotKey( hWnd, delAtom  );  
      }  
    
      TranslateMessage(&msg);  
      DispatchMessage(&msg);  
    }  
    
    ::GlobalDeleteAtom( homeAtom );  
    ::GlobalDeleteAtom( delAtom  );  
    
    return msg.wParam;  
  }  
    
    
    
  //  
  //  FUNCTION: WndProc(HWND, unsigned, WORD, LONG)  
  //  
  //  PURPOSE:  Processes messages for the main window.  
  //  
  //  WM\_HOTKEY   - process the hotkey  
  //  WM\_DESTROY  - post a quit message and return  
  //  
  //  
  LRESULT CALLBACK WndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)  
  {  
    int wmId, wmEvent;  
    PAINTSTRUCT ps;  
    HDC hdc;  
    
    
    switch (message)   
    {  
      case WM\_PAINT:  
        hdc = BeginPaint(hWnd, &ps);  
        // TODO: Add any drawing code here...  
        RECT rt;  
        GetClientRect(hWnd, &rt);  
        DrawText(hdc, szHello, strlen(szHello), &rt, DT\_CENTER);  
        EndPaint(hWnd, &ps);  
        break;  
      case WM\_DESTROY:  
        PostQuitMessage(0);  
        break;  
      case WM\_HOTKEY:  
        {  
          UINT  vk  = (UINT) HIWORD(lParam);  
          if ( vk == VK\_HOME )            // home 键按下，显示或隐藏自己部队的血条  
          {  
            static bool keydown  = false;      //初始不显示  
            if ( keydown )  
            {  
              keydown = false;  
              ::keybd\_event( 0xdb, 0, KEYEVENTF\_KEYUP, 0 );  // 隐藏  
            }  
            else  
            {  
              keydown = true;  
              ::keybd\_event( 0xdb, 0, 0, 0 );      // 显示  
            }  
    
            break;  
          }  
          if ( vk == VK\_END )            // END 键，敌人血条  
          {  
            static bool keydown  = false;  
    
            if ( keydown )  
            {  
              keydown = false;  
              ::keybd\_event( 0xdd, 0, KEYEVENTF\_KEYUP, 0 );  
            }  
            else  
            {  
              keydown = true;  
              ::keybd\_event( 0xdd, 0, 0, 0 );  
            }  
    
            break;  
          }  
        }  
    
      default:  
        return DefWindowProc(hWnd, message, wParam, lParam);  
     }  
     return 0;  
  }  
    
    
    
    
     
    
    
\--------------------------------------------------------------------------------  
【经验总结】  
  以上过程是我整理后写的，与我实际的分析过程有些许差别。  
    
  我实际分析时，脱壳后直接用 OD 跟踪，我凭以往的经验，分别在以下 API 下过断点，  
  OpenProcess  
  PostThreadMessage  
  SetKeyboardState  
  RegisterHotKey  
    
  因为我之前以为该工具是直接操作魔兽争霸的进程，甚至试了 WriteProcessMemory  
    
  结果可想而知，只有 RegisterHotKey 函数被断下来了，然而却没有太大用。  
    
  后来我才想到 VB 的专用分析工具，才最终把它弄明白了。  
  
  本文水平不算太高，权当抛砖引玉之用。如有错误或不恰当的地方，欢迎大家提出，不甚感谢。  
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2008 年 10 月 18 日 20:47:19