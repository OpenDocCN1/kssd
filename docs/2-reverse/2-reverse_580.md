

# 对\[FCG\]KeyGen2.0 按钮特效的分析 3、4、5

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-223.htm`](https://www.pediy.com/kssd/pediy09/pediy09-223.htm)

> 　　首先，注意一个比较特别的现象。原始版本在某些 Windows 上会崩溃，而崩溃后的 Runtime Error 对话框的 OK 按钮居然也是特效。说明什么？一定是钩子。  
> 　　用 IDA 反汇编正常的 KG2，注意到和 KG1 的不同点，可以发现多了一个 WM\_SHOWWINDOW。其处理程序如下:  
>   
> CODE:00408482 loc\_408482:  
> CODE:00408482                 push    0  
> CODE:00408484                 call    sub\_406984  
> CODE:00408489                 jmp     loc\_4088E0  
>   
> 　　可以看到，这是个 stdcall/pascal 类型的过程。至于到底是哪种，无关紧要。这里的参数只有一个，所以不必担心顺序。  
>   
> CODE:00406984 sub\_406984      proc near  
> CODE:00406984  
> CODE:00406984 arg\_0           = dword ptr  8  
> CODE:00406984  
> CODE:00406984                 push    ebp  
> CODE:00406985                 mov     ebp, esp  
> //这里是典型的 pascal/stdcall 调用的开头。但是因为只有 1 个参数，无法肯定是哪个类型。  
> //值得注意的是，这是个 VC 的 obj。所以不能因为是 VC 编译的 obj 就一概使用 cdecl。  
> CODE:00406987                 push    ebx  
> CODE:00406988                 mov     ebx, \[ebp+arg\_0\]  
> CODE:0040698B                 xor     eax, eax  
> CODE:0040698D                 test    ebx, ebx  
> CODE:0040698F                 jnz     short loc\_4069CC  
> CODE:00406991                 cmp     ds:hhk, 0  
> CODE:00406998                 jnz     short loc\_4069D2  
> CODE:0040699A                 push    1               ; lParam  
> CODE:0040699C                 push    offset fn       ; lpfn  
> CODE:004069A1                 call    GetCurrentThreadId\_0  
> CODE:004069A6                 push    eax             ; dwThreadId  
> CODE:004069A7                 call    EnumThreadWindows  
> CODE:004069AC                 call    GetCurrentThreadId\_0  
> CODE:004069B1                 push    eax             ; dwThreadId  
> CODE:004069B2                 push    0               ; hmod  
> CODE:004069B4                 push    offset sub\_406D4A ; lpfn  
> CODE:004069B9                 push    4               ; idHook  
> CODE:004069BB                 call    SetWindowsHookExA  
> CODE:004069C0                 mov     ds:hhk, eax  
> CODE:004069C5                 mov     eax, ds:hhk  
> CODE:004069CA                 jmp     short loc\_4069D2  
> CODE:004069CC  
> CODE:004069CC loc\_4069CC:  
> CODE:004069CC                 push    ebx  
> CODE:004069CD                 call    sub\_406A1F  
> CODE:004069D2  
> CODE:004069D2 loc\_4069D2:  
> CODE:004069D2                 pop     ebx  
> CODE:004069D3                 pop     ebp  
> CODE:004069D4                 retn    4  
> CODE:004069D4 sub\_406984      endp  
>   
> 　　这个初始化过程（实际是个函数）的流程还是非常清楚的。但是这个参数到底是什么？从上面来看，如果参数为 0，那么就枚举当前线程的窗口。看来应该是某窗口的句柄。通过查看 sub\_406A1F，发现 00406A53 处有对 GetWindowLong 的调用，而且传过去的 ebx 作为首个参数。这样，可以判断它是个窗口句柄。这里给出该过程的 Pascal 形式  
> 
> 代码:
> 
> * * *
> 
> `Var     CurrentHook:HHOOK;      Function Init(Window:HWND):HHOOK;   Begin     Result:=0;     If Window=0 Then       Begin         If CurrentHook=0 Then           Begin             EnumThreadWindows(GetCurrentThreadId,,1);             CurrentHook:=SetWindowsHookEx(WH_CALLWNDPROC,sub_406D4A,0,GetCurrentThreadId);             Result:=CurrentHook;           End;       End     Else       Begin         sub_406A1F(Window);       End;   End;` 
> 
> * * *
> 
> 　　 CurrentHook 又是什么呢？从上面看，应该是起类似于标志的作用。初始值为 0，Hook 过以后就是  
> Hook 函数的句柄。所以修改一下定义  
> 代码:
> 
> * * *
> 
> `Const     CurrentHook:HHOOK=0;` 
> 
> * * *
> 
> ????    首先看看 sub\_406D4A，这是一个 CallWndProc 函数。MSDN 中有关定义如下：  
> 代码:
> 
> * * *
> 
> `LRESULT CALLBACK CallWndProc(       int nCode,       WPARAM wParam,       LPARAM lParam   );` 
> 
> * * *
> 
> ????    再看看函数体：  
>   
> seg000:00406D4A ; LRESULT \_\_stdcall sub\_406D4A(int,WPARAM,LPARAM)  
> seg000:00406D4A sub\_406D4A      proc near               ; DATA XREF: sub\_406984+30  
> seg000:00406D4A  
> seg000:00406D4A arg\_0           = dword ptr  8  
> seg000:00406D4A wParam          = dword ptr  0Ch  
> seg000:00406D4A arg\_8           = dword ptr  10h  
> seg000:00406D4A  
> seg000:00406D4A                 push    ebp  
> seg000:00406D4B                 mov     ebp, esp  
> seg000:00406D4D                 push    ebx  
> seg000:00406D4E                 push    esi  
> seg000:00406D4F                 mov     ebx, \[ebp+arg\_8\]  
> //ebx:=LPARAM  
> //注意这里是一个地址，指向一个 CWPSTRUCT 结构  
> seg000:00406D52                 mov     esi, \[ebp+arg\_0\]  
> //esi:=nCode  
> seg000:00406D55                 cmp     dword ptr \[ebx+8\], 1  
> //判断 CWPSTRUCT.Message  
> //由于 BN\_PAINT=WM\_CREATE=1，所以  
> //具体 1 是哪个 message，需要根据进一步的分析来判断  
> seg000:00406D59                 jnz     short loc\_406D67  
> seg000:00406D5B                 test    esi, esi  
> seg000:00406D5D                 jl      short loc\_406D67  
> seg000:00406D5F                 push    dword ptr \[ebx+0Ch\]  
> //CWPSTRUCT.hwnd  
> seg000:00406D62                 call    sub\_406A1F  
> seg000:00406D67  
> seg000:00406D67 loc\_406D67:                             ; CODE XREF: sub\_406D4A+F  
> seg000:00406D67                                         ; sub\_406D4A+13  
> seg000:00406D67                 push    ebx             ; lParam  
> seg000:00406D68                 push    \[ebp+wParam\]    ; wParam  
> seg000:00406D6B                 push    esi             ; nCode  
> seg000:00406D6C                 push    ds:hhk          ; hhk  
> seg000:00406D72                 call    CallNextHookEx  
> seg000:00406D77                 pop     esi  
> seg000:00406D78                 pop     ebx  
> seg000:00406D79                 pop     ebp  
> seg000:00406D7A                 retn    0Ch  
> seg000:00406D7A sub\_406D4A      endp  
>   
>     其中的 CWPSTRUCT 结构：  
> 代码:
> 
> * * *
> 
> `typedef struct {       LPARAM lParam;       WPARAM wParam;       UINT message;       HWND hwnd;   } CWPSTRUCT, *PCWPSTRUCT;` 
> 
> * * *
> 
> ????    这个函数也不复杂：  
> 代码:
> 
> * * *
> 
> `Procedure CallBackWndProc(nCode:Integer;wParam:Cardinal;var lParam:CWPSTRUCT);stdcall;   Begin     If (lParam.message=1) AND (nCode<0) Then       Begin         sub_406A1F(lParam.hwnd);       End;     CallNextHookEx(CurrentHook,nCode,wParam,lParam);      End;` 
> 
> * * *

> 下面开始分析 sub\_406A1F，看代码：  
>   
> seg000:00406A1F sub\_406A1F      proc near               ; CODE XREF: sub\_406984+49  
> seg000:00406A1F                                         ; fn+18...  
> seg000:00406A1F  
> seg000:00406A1F var\_4           = dword ptr -4  
> seg000:00406A1F arg\_0           = dword ptr  8  
> seg000:00406A1F  
> seg000:00406A1F                 push    ebp  
> seg000:00406A20                 mov     ebp, esp  
> seg000:00406A22                 push    ecx  
> seg000:00406A23                 push    ebx  
> seg000:00406A24                 push    esi  
> seg000:00406A25                 push    edi  
> //首先根据前面的分析，得到函数的类型。由此判断以上代码为保存寄存器  
> seg000:00406A26                 mov     esi, \[ebp+arg\_0\]  
> seg000:00406A29                 push    esi  
> seg000:00406A2A                 call    sub\_406B69  
> //sub\_406B69 需要传进来的窗口句柄，稍后分析  
> seg000:00406A2F                 test    eax, eax  
> //注意返回值的类型  
> seg000:00406A31                 jnz     loc\_406AE3  
> //非 0 就跳  
> seg000:00406A37                 push    esi  
> seg000:00406A38                 call    sub\_406B89  
> //再次调用  
> seg000:00406A3D                 mov     edi, eax  
> //保存返回值  
> seg000:00406A3F                 test    edi, edi  
> seg000:00406A41                 jb      loc\_406AE3  
> //非常奇怪，因为按照 Test 的说明，CF 是一定为 0 的。  
> //这样 jb 就一定不会跳转  
> //谁看明白了，请告诉我。反正我觉得这个是废代码，可能是作者干的事情。  
> seg000:00406A47                 cmp     edi, 1  
> seg000:00406A4A                 ja      loc\_406AE3  
> //若大于 1 就跳。这样，返回值应该是整数类型  
> seg000:00406A50                 push    0FFFFFFF0h  
> seg000:00406A52                 push    esi  
> seg000:00406A53                 call    GetWindowLongA\_0  
> seg000:00406A58                 mov     \[ebp+var\_4\], eax  
> //局部变量  
> seg000:00406A5B                 push    14h             ; dwBytes  
> //20 字节！  
> seg000:00406A5D                 push    0               ; dwFlags  
> seg000:00406A5F                 call    GetProcessHeap  
> //注意，这里 GetProcessHeap 是无参 API  
> seg000:00406A64                 push    eax             ; hHeap  
> seg000:00406A65                 call    HeapAlloc  
> //返回指针  
> seg000:00406A6A                 mov     ebx, eax  
> seg000:00406A6C                 mov     eax, ds:dword\_40A150  
> seg000:00406A71                 mov     \[ebx+10h\], eax  
> seg000:00406A74                 mov     ds:dword\_40A150, ebx  
> //将全局变量 dword\_40A150 的内容放入刚刚申请的内存块的最后 4 字节，再将刚刚申请的内存块的的地址存入 dword\_40A150。这里的操作让人想起单向链表！  
> seg000:00406A7A                 mov     \[ebx\], esi  
> //窗口句柄  
> seg000:00406A7C                 mov     \[ebx+4\], edi  
> //某个神秘的返回值！  
> seg000:00406A7F                 test    byte ptr \[ebp+var\_4+3\], 8  
> //这里是测试 var\_4 的最高位  
> seg000:00406A83                 jz      short loc\_406A8C  
> //为 0 则跳  
> seg000:00406A85                 mov     edx, 1  
> seg000:00406A8A                 jmp     short loc\_406A8E  
> seg000:00406A8C  
> seg000:00406A8C loc\_406A8C:                             ; CODE XREF: sub\_406A1F+64  
> seg000:00406A8C                 xor     edx, edx  
> seg000:00406A8E  
> seg000:00406A8E loc\_406A8E:                             ; CODE XREF: sub\_406A1F+6B  
> //edx 为 var\_4 的最高位  
> seg000:00406A8E                 mov     \[ebx+8\], edx  
> //内存块  
> seg000:00406A91                 call    GetFocus  
> seg000:00406A96                 cmp     esi, eax  
> seg000:00406A98                 jnz     short loc\_406A9E  
> seg000:00406A9A                 or      dword ptr \[ebx+8\], 8  
> //置位，注意 dword ptr！  
> seg000:00406A9E  
> seg000:00406A9E loc\_406A9E:                             ; CODE XREF: sub\_406A1F+79  
> seg000:00406A9E                 push    offset sub\_406D7D  
> seg000:00406AA3                 push    0FFFFFFFCh  
> seg000:00406AA5                 push    esi  
> seg000:00406AA6                 call    SetWindowLongA\_0  
> //为参数指定的窗口设置 Callback  
> //这里的 nIndex 是 GWL\_WNDPROC  
> seg000:00406AAB                 mov     \[ebx+0Ch\], eax  
> //该窗口属性的原始值  
> seg000:00406AAE                 sub     edi, 1  
> //参见上文  
> seg000:00406AB1                 jb      short loc\_406AB7  
> seg000:00406AB3                 jz      short loc\_406AC6  
> seg000:00406AB5                 jmp     short loc\_406AD0  
> seg000:00406AB7  
> seg000:00406AB7 loc\_406AB7:                             ; CODE XREF: sub\_406A1F+92  
> seg000:00406AB7                 mov     eax, \[ebp+var\_4\]  
> seg000:00406ABA                 and     eax, 1Fh  
> seg000:00406ABD                 dec     eax  
> seg000:00406ABE                 jnz     short loc\_406AD0  
> seg000:00406AC0                 or      dword ptr \[ebx+8\], 10h  
> seg000:00406AC4                 jmp     short loc\_406AD0  
> seg000:00406AC6  
> seg000:00406AC6 loc\_406AC6:                             ; CODE XREF: sub\_406A1F+94  
> seg000:00406AC6                 test    byte ptr \[ebp+var\_4+1\], 8  
> //var\_4 的低字的第 12 位  
> seg000:00406ACA                 jz      short loc\_406AD0  
> seg000:00406ACC                 or      dword ptr \[ebx+8\], 20h  
> seg000:00406AD0  
> seg000:00406AD0 loc\_406AD0:                             ; CODE XREF: sub\_406A1F+96  
> seg000:00406AD0                                         ; sub\_406A1F+9F ...  
> seg000:00406AD0                 push    705h            ; flags  
> seg000:00406AD5                 push    0               ; hrgnUpdate  
> seg000:00406AD7                 push    0               ; lprcUpdate  
> seg000:00406AD9                 push    esi             ; hWnd  
> seg000:00406ADA                 call    RedrawWindow  
> seg000:00406ADF                 mov     eax, ebx  
> seg000:00406AE1                 jmp     short loc\_406AE5  
> seg000:00406AE3  
> seg000:00406AE3 loc\_406AE3:                             ; CODE XREF: sub\_406A1F+12  
> seg000:00406AE3                                         ; sub\_406A1F+22 ...  
> seg000:00406AE3                 xor     eax, eax  
> seg000:00406AE5  
> seg000:00406AE5 loc\_406AE5:                             ; CODE XREF: sub\_406A1F+C2  
> //以下代码为函数出口，恢复寄存器  
> seg000:00406AE5                 pop     edi  
> seg000:00406AE6                 pop     esi  
> seg000:00406AE7                 pop     ebx  
> seg000:00406AE8                 pop     ecx  
> seg000:00406AE9                 pop     ebp  
> seg000:00406AEA                 retn    4  
> seg000:00406AEA sub\_406A1F      endp  
>   
>     显然，这又是一个 stdcall/pascal 的函数。  
>   
>     看看那个神秘的内存块里是什么东西：  
>   
> 10H    ds:dword\_40A150，疑为指针  
> 0CH    该窗口属性的原始值  
> 08H    var\_4 的最高位  
> 04H    sub\_406B89 的返回值  
> 00H    窗口句柄  
>   
>     看起来应该是作为单向链表元素的结构体。但是目前尚无法判断是否为作者自定义结构。这个函数有个很值得注意的地方：从函数体看，它返回一个指向上述结构体的指针（链表的头）。但是调用它的函数并没有使用这个返回值。就函数代码看，这个返回值是被放到一个全局变量 ds:dword\_40A150 里的。看起来，象是作者干的事情。  
>   
>     既然有链表，就一定有分配和释放。分配的代码已经有了。释放的代码有待分析。  
>   
>     整理一下代码，得到初步结果：  
>   
> Type  
>   PWindowProperty=^WindowProperty;  
>   WindowProperty=Record  
>     CurrentWindow:HWND;  
>     Preserve1:Cardinal;//sub\_406B89 的返回值  
>     Preserve2:Cardinal;//var\_4 的最高位  
>     OrignalProperty:Cardinal;  
>     NextWindowProperty:PWindowProperty;  
>   End;  
>   
> Var  
>   WinPropLinkHead:PWindowProperty;//就是 ds:dword\_40A150  
>     
>     
> Function sub\_406A1F(Window:HWND):PWindowProperty;  
> Var  
>   CurrentWinProp:PWindowProperty;  
>   CurrentWinStyle:Cardinal;  
>   EDI:Cardinal;  
> Begin  
>   Result:=0;  
>   If sub\_406B69(Window)<>0 Then Exit;  
>   EDI:=sub\_406B89(Window);  
>   If EDI>1 Then Exit;  
>   CurrentWinStyle:=GetWindowLongA(Window,GWL\_STYLE);  
>   CurrentWinProp:=HeapAlloc(GetProcessHeap,0,20);  
>   CurrentWinProp^.NextWindowProperty:=WinPropLinkHead;  
>   WinPropLinkHead:=CurrentWinProp;  
>   CurrentWinProp^.CurrentWindow:=Window;  
>   CurrentWinProp^.Preserve1:=edi;  
>   CurrentWinProp^.Preserve2:=CurrentWinStyle SHR 31;//WS\_POPUP?  
>   
>   If GetFocus=Window Then  
>     Begin  
>       CurrentWinProp^.Preserve2:=CurrentWinProp^.Preserve2 OR 8;  
>     End;  
>   
>   CurrentWinProp^.OrignalProperty:=SetWindowLongA(Window,GWL\_WNDPROC,sub\_406D7D);  
>     
>   If EDI<1 Then  
>     Begin  
>       If CurrentWinStyle AND $1F=1 Then  
>       //$1F=WS\_EX\_ACCEPTFILES OR WS\_EX\_TOPMOST OR WS\_EX\_NOPARENTNOTIFY OR WS\_EX\_DLGMODALFRAME  
>         Begin  
>           CurrentWinProp^.Preserve2:=CurrentWinProp^.Preserve2 OR $10;  
>         End;    
>     End;    
>     
>   If EDI=1 Then  
>     Begin  
>       If CurrentWinStyle AND $80000<>0 Then  
>       //WS\_SYSMENU  
>         Begin  
>           CurrentWinProp^.Preserve2:=CurrentWinProp^.Preserve2 OR $20;  
>         End;    
>     End;    
>       
>   RedrawWindow(Window,Nil,0,RDW\_FRAME OR RDW\_ERASENOW OR RDW\_UPDATENOW OR RDW\_ERASE OR RDW\_INVALIDATE);  
>   Result:=CurrentWinProp;  
> End;  
>   
>     这里说一下我整理代码的一般步骤：  
>   
>     1、首先从 API 入手，根据 API 的说明弄清楚它用到了哪些寄存器。然后就可以知道哪些寄存器是干吗的：是否是什么句柄、是否指向什么结构……  
>     2、将 API 调用中的常数整理出来。这个需要查相关资料。  
>     3、将其余的指令直接用你觉得方便的伪代码写出来。只要你能看懂而且不会弄错就可以。注意，IDA 自动生成的地址标签一定要保留，这是下面步骤的基础。比如 loc\_406AC6 我就写成：  
>   
>     loc\_406AC6:  
>     if var\_4 and $800=0 then goto loc\_406AD0  
>     else ebx\_008h=ebx\_008h or $20  
>   
>     另外一个值得注意的地方是堆栈寻址。必要的时候，可以用 debug 写段代码验证一下。不过我用的是 GRDB，比 debug 好不少。  
>   
>     4、重写控制结构，比如 If Else 等……到这一步，程序基本就出来了。  
>     5、对程序里使用的变量和数据结构做出猜测并重新命名。最后再用你要使用的语言写一遍就是了。不会花太久的。 

> 首先，废话几句。不知道有没有人还记得这个系列？在下因为个人原因，暂停了很久。不过最近在努力继续写。敬请期待……  
>   
> 另：如果您发现有错误/值得讨论的地方，请告诉我。  
>   
> \==========================================  
>     sub\_406A1F 中一共使用了 3 个 func：sub\_406B69、sub\_406B89、sub\_406D7D。  
>   
>     首先看 sub\_406B69：  
>   
> seg000:00406B69 sub\_406B69      proc near               ; CODE XREF: sub\_406A1F+B  
> seg000:00406B69                                         ; sub\_406AED+A ...  
> seg000:00406B69  
> seg000:00406B69 arg\_0           = dword ptr  8  
> //从前面的分析知道这个 arg 是窗口的句柄  
> seg000:00406B69  
> seg000:00406B69                 push    ebp  
> seg000:00406B6A                 mov     ebp, esp  
> seg000:00406B6C                 mov     edx, \[ebp+arg\_0\]  
> seg000:00406B6F                 mov     eax, ds:dword\_40A150  
> //即上文的 WinPropLinkHead  
> seg000:00406B74                 test    eax, eax  
> seg000:00406B76                 jz      short loc\_406B83  
> //为 null  
> seg000:00406B78  
> seg000:00406B78 loc\_406B78:                             ; CODE XREF: sub\_406B69+18  
> seg000:00406B78                 cmp     edx, \[eax\]  
> //非空指针，将指针指向结构的 CurrentWindow 和参数比较  
> //其实就是寻找参数对应的窗口是否有相应的控制参数块  
> seg000:00406B7A                 jz      short loc\_406B85  
> //找到，直接返回指针，即 eax  
> seg000:00406B7C                 mov     eax, \[eax+10h\]  
> //取下一块的地址  
> seg000:00406B7F                 test    eax, eax  
> //下一块的地址是否是 null  
> seg000:00406B81                 jnz     short loc\_406B78  
> //不是 null 则循环，否则表示已到链尾  
> seg000:00406B83  
> seg000:00406B83 loc\_406B83:                             ; CODE XREF: sub\_406B69+D  
> seg000:00406B83                 xor     eax, eax  
> //直接返回 null  
> seg000:00406B85  
> seg000:00406B85 loc\_406B85:                             ; CODE XREF: sub\_406B69+11  
> seg000:00406B85                 pop     ebp  
> seg000:00406B86                 retn    4  
> seg000:00406B86 sub\_406B69      endp  
>   
>     这个函数很容易写出：  
> 
> 代码:
> 
> ```
>     Function FindWindowInChain(Window:HWND):PWindowProperty;
>     Begin
>       Result:=WinPropLinkHead;
>       
>       While (Result<>Nil) Do
>         Begin
>           If Result^.CurrentWindow=Window Then
>             Begin
>               Break;
>             End;
>           Result:=Result^.NextWindowProperty;
>         End;
>     End;
> 
> ```
> 
>     这样，前面的程序也需要修改。但是最好等到最后整理的时候再改。  
>       
>     sub\_406B89：  
>   
> seg000:00406B89 sub\_406B89      proc near               ; CODE XREF: sub\_406A1F+19  
> seg000:00406B89  
> seg000:00406B89 ClassName       = byte ptr -104h  
> seg000:00406B89 hWnd            = dword ptr  8  
> seg000:00406B89  
> seg000:00406B89                 push    ebp  
> seg000:00406B8A                 mov     ebp, esp  
> seg000:00406B8C                 add     esp, 0FFFFFEFCh  
> //猜测是对齐  
> seg000:00406B92                 push    ebx  
> seg000:00406B93                 push    esi  
> //函数功能开始  
> seg000:00406B94                 mov     esi, \[ebp+hWnd\]  
> //参数  
> seg000:00406B97                 push    104h            ; nMaxCount  
> seg000:00406B9C                 lea     eax, \[ebp+ClassName\]  
> seg000:00406BA2                 push    eax             ; lpClassName  
> seg000:00406BA3                 push    esi             ; hWnd  
> seg000:00406BA4                 call    GetClassNameA  
> seg000:00406BA9                 xor     ebx, ebx  
> seg000:00406BAB  
> seg000:00406BAB loc\_406BAB:                             ; CODE XREF: sub\_406B89+41  
> seg000:00406BAB                 mov     eax, ebx  
> seg000:00406BAD                 shl     eax, 5  
> //这里实际是乘以 20，字符串数组的第 1 维大小  
> seg000:00406BB0                 add     eax, offset dword\_40A154  
> //字符串数组的地址  
> seg000:00406BB5                 push    eax  
> seg000:00406BB6                 lea     edx, \[ebp+ClassName\]  
> seg000:00406BBC                 push    edx  
> seg000:00406BBD                 call    lstrcmpi  
> seg000:00406BC2                 test    eax, eax  
> seg000:00406BC4                 jz      short loc\_406BCC  
> //该窗口的类名符合条件  
> seg000:00406BC6                 inc     ebx  
> seg000:00406BC7                 cmp     ebx, 2  
> seg000:00406BCA                 jb      short loc\_406BAB  
> //数组就 2 个元素  
> seg000:00406BCC  
> seg000:00406BCC loc\_406BCC:                             ; CODE XREF: sub\_406B89+3B  
> seg000:00406BCC                 sub     ebx, 1  
> seg000:00406BCF                 jb      short loc\_406BD5  
> //是 Button 类  
> seg000:00406BD1                 jz      short loc\_406BF2  
> //是 Edit 类  
> seg000:00406BD3                 jmp     short loc\_406BF9  
> //都不是  
> seg000:00406BD5  
> seg000:00406BD5 loc\_406BD5:                             ; CODE XREF: sub\_406B89+46  
> seg000:00406BD5                 push    0FFFFFFF0h  
> //GWL\_STYLE  
> seg000:00406BD7                 push    esi  
> seg000:00406BD8                 call    GetWindowLongA\_0  
> seg000:00406BDD                 mov     ebx, eax  
> seg000:00406BDF                 and     ebx, 1Fh  
> seg000:00406BE2                 sub     ebx, 2  
> seg000:00406BE5                 jb      short loc\_406BE9  
> seg000:00406BE7                 jmp     short loc\_406BED  
> seg000:00406BE9  
> seg000:00406BE9 loc\_406BE9:                             ; CODE XREF: sub\_406B89+5C  
> seg000:00406BE9                 xor     ebx, ebx  
> seg000:00406BEB                 jmp     short loc\_406BFC  
> seg000:00406BED  
> seg000:00406BED loc\_406BED:                             ; CODE XREF: sub\_406B89+5E  
> seg000:00406BED                 or      ebx, 0FFFFFFFFh  
> seg000:00406BF0                 jmp     short loc\_406BFC  
> seg000:00406BF2  
> seg000:00406BF2 loc\_406BF2:                             ; CODE XREF: sub\_406B89+48  
> seg000:00406BF2                 mov     ebx, 1  
> seg000:00406BF7                 jmp     short loc\_406BFC  
> seg000:00406BF9  
> seg000:00406BF9 loc\_406BF9:                             ; CODE XREF: sub\_406B89+4A  
> seg000:00406BF9                 or      ebx, 0FFFFFFFFh  
> seg000:00406BFC  
> seg000:00406BFC loc\_406BFC:                             ; CODE XREF: sub\_406B89+62  
> seg000:00406BFC                                         ; sub\_406B89+67 ...  
> seg000:00406BFC                 mov     eax, ebx  
> seg000:00406BFE                 pop     esi  
> seg000:00406BFF                 pop     ebx  
> seg000:00406C00                 mov     esp, ebp  
> seg000:00406C02                 pop     ebp  
> seg000:00406C03                 retn    4  
> seg000:00406C03 sub\_406B89      endp  
>   
>     这个函数比较麻烦。因为 dword\_40A154 数组的元素是按照 32 个字节对齐的，而我在 D5 里无法写出对应的形式。当然，可以用 Array of Char 解决这个问题，但是这样一是比较麻烦，二是我得完全按照汇编来写而不能享受数组的方便快捷。所以我在这里不打算做直接的对照翻译。等价的形式会更快。   
>   
> 代码:
> 
> ```
>     Const
>       Classes:Array\[0..1\] of PChar=('Button','Edit');
>       
>     Function sub\_406B89(Window:HWND):Integer;
>     Var
>       ClassName:PChar;
>       i:LongWord;
>     Begin
>       GetMem(ClassName,$104);
>       GetClassName(Window,ClassName,$104);
>     
>       For i:=0 To 1 Do
>         Begin
>           If lstrcmpi(ClassName,Classes\[i\])=0 Then Break;
>         End;
>         
>       Case i of
>         0:
>         Begin
>           If GetWindowLong(Window,GWL\_STYLE) AND $1F<2 Then
>             Begin
>               Result:=0;
>             End
>           Else
>             Begin
>               Result:=-1;
>             End;
>         End;
>         1:
>         Result:=1;
>       Else
>         Result:=-1;
>       End;
>     End;
> 
> ```
> 
> ＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝  
>   
>   
> 第 4、5 篇均在第一页  
>   
>   
> \====================================  
>   
> 补充一下，00406BBD 处的窗口类名比较时使用的类名实际是和程序有关的。在此例中，窗口由 RC 建立，所以类名为 Button 等…… 

> **对\[FCG\]KeyGen2.0 按钮特效的分析 4**  
>   
>     粗略看了一下 sub\_406D7D，发现其中有很多对程序里其它函数的调用。所以这部分延后分析。  
>   
>     sub\_406D7D 调用了以下函数：sub\_406B69、sub\_406974、sub\_4070D9、sub\_407274，其中 sub\_406B69 已经分析。  
>   
> seg000:00406974 sub\_406974      proc near               ; CODE XREF: sub\_406D7D+53  
> seg000:00406974                                         ; sub\_406D7D+80 ...  
> seg000:00406974  
> seg000:00406974 arg\_4           = dword ptr  0Ch  
> seg000:00406974 arg\_8           = dword ptr  10h  
> seg000:00406974  
> seg000:00406974                 push    ebp  
> seg000:00406975                 mov     ebp, esp  
> seg000:00406977                 mov     eax, \[ebp+arg\_4\]  
> seg000:0040697A                 cmp     \[ebp+arg\_8\], 0  
> seg000:0040697E                 jnz     short loc\_406982  
> seg000:00406980                 not     eax  
> seg000:00406982  
> seg000:00406982 loc\_406982:                             ; CODE XREF: sub\_406974+A  
> seg000:00406982                 pop     ebp  
> seg000:00406983                 retn  
> seg000:00406983 sub\_406974      endp  
>   
>     逻辑非常简单：如果参数 3 为 0 则返回参数 2 的反值，否则直接返回参数 2。从堆栈看，有参数 1，只是被忽略了。  
> 
> 代码:
> 
> ```
>     Function sub\_406974(param1:LongWord;param2:LongWord;param3:LongWord)
>     Begin
>       If param3=0 Then
>         Result:=param2 XOR $FFFFFFFF
>       Else
>         Result:=param2;
>     End;
> 
> ```
> 
>     sub\_4070D9 调用了 sub\_406C06、sub\_406F99、sub\_406CA5 三个函数。  
>   
> seg000:00406C06 sub\_406C06      proc near               ; CODE XREF: sub\_4070D9+25  
> seg000:00406C06  
> seg000:00406C06 Rect            = tagRECT ptr -10h  
> seg000:00406C06 arg\_0           = dword ptr  8  
> seg000:00406C06  
> seg000:00406C06                 push    ebp  
> seg000:00406C07                 mov     ebp, esp  
> seg000:00406C09                 add     esp, 0FFFFFFF0h  
> //分配临时变量  
> seg000:00406C0C                 push    ebx  
> seg000:00406C0D                 mov     ebx, \[ebp+arg\_0\]  
> //ebx=arg0  
> seg000:00406C10                 lea     eax, \[ebp+Rect\]  
> seg000:00406C13                 push    eax             ; lpRect  
> seg000:00406C14                 push    dword ptr \[ebx\] ; hWnd  
> //从这里可以看出，arg0 实际是指针，指向的结构体开始部分为窗口句柄  
> seg000:00406C16                 call    GetWindowRect  
> seg000:00406C1B                 push    dword ptr \[ebx\] ; hWnd  
> seg000:00406C1D                 call    GetWindowDC  
> seg000:00406C22                 mov     \[ebx+4\], eax  
> //将对应的 DC 保存到结构体的\[4\]  
> seg000:00406C25                 push    dword ptr \[ebx+4\] ; HDC  
> seg000:00406C28                 call    CreateCompatibleDC  
> seg000:00406C2D                 mov     \[ebx+8\], eax  
> //memdc  
> seg000:00406C30                 push    1               ; int  
> seg000:00406C32                 push    dword ptr \[ebx+8\] ; HDC  
> seg000:00406C35                 call    SetBkMode  
> seg000:00406C3A                 cmp     dword ptr \[ebx+10h\], 0  
> //如果结构体的\[10h\]是 0  
> seg000:00406C3E                 jz      short loc\_406C4D  
> //非 0 就直接选择对象  
> seg000:00406C40                 push    dword ptr \[ebx+10h\] ; HGDIOBJ  
> seg000:00406C43                 push    dword ptr \[ebx+8\] ; HDC  
> seg000:00406C46                 call    SelectObject  
> seg000:00406C4B                 jmp     short loc\_406C6F  
> seg000:00406C4D  
> seg000:00406C4D loc\_406C4D:                             ; CODE XREF: sub\_406C06+38  
> seg000:00406C4D                 mov     edx, \[ebp+Rect.bottom\]  
> seg000:00406C50                 sub     edx, \[ebp+Rect.top\]  
> seg000:00406C53                 push    edx             ; int  
> seg000:00406C54                 mov     ecx, \[ebp+Rect.right\]  
> seg000:00406C57                 sub     ecx, \[ebp+Rect.left\]  
> seg000:00406C5A                 push    ecx             ; int  
> seg000:00406C5B                 push    dword ptr \[ebx+4\] ; HDC  
> seg000:00406C5E                 call    CreateCompatibleBitmap  
> seg000:00406C63                 push    eax             ; HGDIOBJ  
> seg000:00406C64                 push    dword ptr \[ebx+8\] ; HDC  
> seg000:00406C67                 call    SelectObject  
> seg000:00406C6C                 mov     \[ebx+10h\], eax  
> //看来\[10h\]是 bitmap 的句柄  
> seg000:00406C6F  
> seg000:00406C6F loc\_406C6F:                             ; CODE XREF: sub\_406C06+45  
> seg000:00406C6F                 cmp     dword ptr \[ebx+0Ch\], 1  
> seg000:00406C73                 jnz     short loc\_406C9B  
> //\[0ch\]是什么，从下文看不出  
> seg000:00406C75                 push    0CC0020h        ; DWORD  
> //SRCCOPY  
> seg000:00406C7A                 push    0               ; int  
> seg000:00406C7C                 push    0               ; int  
> seg000:00406C7E                 push    dword ptr \[ebx+4\] ; HDC  
> seg000:00406C81                 mov     eax, \[ebp+Rect.bottom\]  
> seg000:00406C84                 sub     eax, \[ebp+Rect.top\]  
> seg000:00406C87                 push    eax             ; int  
> seg000:00406C88                 mov     edx, \[ebp+Rect.right\]  
> seg000:00406C8B                 sub     edx, \[ebp+Rect.left\]  
> seg000:00406C8E                 push    edx             ; int  
> seg000:00406C8F                 push    0               ; int  
> seg000:00406C91                 push    0               ; int  
> seg000:00406C93                 push    dword ptr \[ebx+8\] ; HDC  
> seg000:00406C96                 call    BitBlt  
> //保存原始 DC？  
> seg000:00406C9B  
> seg000:00406C9B loc\_406C9B:                             ; CODE XREF: sub\_406C06+6D  
> seg000:00406C9B                 mov     eax, \[ebx+8\]  
> //返回 MEM DC 的句柄  
> seg000:00406C9E                 pop     ebx  
> seg000:00406C9F                 mov     esp, ebp  
> seg000:00406CA1                 pop     ebp  
> seg000:00406CA2                 retn    4  
> seg000:00406CA2 sub\_406C06      endp  
>   
> 代码:
> 
> ```
> Type
>   WindowsDCInfo=Record
>     hWindow:HWND;
>     WinDC:HDC;
>     CompDC:HDC;
>     unknown:LongWord;
>     hBitmap:HBITMAP;
>   End;
>   PWindowsDCInfo=^WindowsDCInfo;
> 
> Function sub\_406C06(Var DCInfo:WindowsDCInfo):HDC;
> Var
>   Rect:TRect;
> Begin
>   GetWindowRect(DCInfo.hWindow,Rect);
>   DCInfo.WinDC:=GetWindowDC(DCInfo.hWindow);
>   DCInfo.CompDC:=CreateCompatibleDC(DCInfo.WinDC);
>   SetBkMode(DCInfo.CompDC,TRANSPARENT);
>   
>   If DCInfo.hBitmap<>0 Then
>     Begin
>       SelectObject(DCInfo.CompDC,DCInfo.hBitmap);
>     End
>   Else
>     Begin
>       DCInfo.hBitmap:=CreateCompatibleBitmap(DCInfo.WinDC,Rect.right-Rect.left,Rect.bottom-Rect.top);
>       SelectObject(DCInfo.CompDC,DCInfo.hBitmap);
>     End;
>     
>   If DCInfo.unknown=1 Then
>     Begin
>       BitBlt(DCInfo.CompDC,0,0,Rect.right-Rect.left,Rect.bottom-Rect.top,DCInfo.WinDC,0,0,SRCCOPY);
>     End;
>     
>   Result:=DCInfo.CompDC;
> 
> End;
> 
> ```
> 
>     显然，某些东西依然未知。   
>   
>     看起来，sub\_406F99 就是生成渐变的核心过程。各位打起精神来！ 首先，从 GradientFill 的两个 call 得到，var\_48 处是一个 Arrayp\[0..3\] of TRIVERTEX 的结构数组。var\_54 和 var\_8 分别是 GRADIENT\_TRIANGLE 和 GRADIENT\_RECT。但 d5 里对于 TRIVERTEX 的定义似乎有问题：COLOR16 是 8 位而不是 16 位。最好的办法是自己重新写定义。  
>   
> seg000:00406F99 ; int \_\_stdcall sub\_406F99(HDC,int,int)  
> seg000:00406F99 sub\_406F99      proc near               ; CODE XREF: sub\_4070D9+DC  
> seg000:00406F99                                         ; sub\_4070D9+F0  
> seg000:00406F99  
> seg000:00406F99 var\_54          = dword ptr -54h  
> seg000:00406F99 var\_50          = dword ptr -50h  
> seg000:00406F99 var\_4C          = dword ptr -4Ch  
> seg000:00406F99 var\_48          = dword ptr -48h  
> seg000:00406F99 var\_8           = dword ptr -8  
> seg000:00406F99 var\_4           = dword ptr -4  
> seg000:00406F99 arg\_0           = dword ptr  8  
> seg000:00406F99 arg\_4           = dword ptr  0Ch  
> seg000:00406F99 arg\_8           = dword ptr  10h  
> seg000:00406F99  
> seg000:00406F99                 push    ebp  
> seg000:00406F9A                 mov     ebp, esp  
> seg000:00406F9C                 add     esp, 0FFFFFFACh  
> //分配局部堆栈变量  
> seg000:00406F9F                 push    ebx  
> seg000:00406FA0                 push    esi  
> seg000:00406FA1                 push    edi  
> seg000:00406FA2                 mov     eax, \[ebp+arg\_8\]  
> //eax=arg\_8  
> seg000:00406FA5                 mov     edx, \[ebp+arg\_4\]  
> //edx=arg\_4  
> seg000:00406FA8                 lea     esi, \[ebp+var\_48\]  
> seg000:00406FAB                 xor     ebx, ebx  
> //ebx=0  
> seg000:00406FAD  
> seg000:00406FAD loc\_406FAD:                             ; CODE XREF: sub\_406F99+70  
> seg000:00406FAD                 xor     ecx, ecx  
> seg000:00406FAF                 mov     cl, \[eax+ebx\*4\]  
> //arg\_8 应该是指针  
> seg000:00406FB2                 shl     ecx, 8  
> //ecx=((eax+ebx\*4)^) shl 8  
> seg000:00406FB5                 mov     edi, ebx  
> seg000:00406FB7                 shl     edi, 4  
> seg000:00406FBA                 mov     \[esi+edi+8\], cx  
> //注意：TRIVERTEX 长度为 10h=4h shl 4，所以这里应该写成  
> //var\_48\[ebx\].Red=ecx and $FFFF  
> seg000:00406FBF                 movzx   ecx, word ptr \[eax+ebx\*4\]  
> seg000:00406FC3                 sar     ecx, 8  
> seg000:00406FC6                 and     ecx, 0FFh  
> seg000:00406FCC                 shl     ecx, 8  
> //ecx=(((arg\_8+ebx\*4)^ sar 8)and $ff) shl 8  
> seg000:00406FCF                 mov     edi, ebx  
> seg000:00406FD1                 shl     edi, 4  
> seg000:00406FD4                 mov     \[esi+edi+0Ah\], cx  
> //var\_48\[ebx\].Green=ecx and $FFFF  
> seg000:00406FD9                 mov     ecx, \[eax+ebx\*4\]  
> seg000:00406FDC                 shr     ecx, 10h  
> seg000:00406FDF                 and     ecx, 0FFh  
> seg000:00406FE5                 shl     ecx, 8  
> seg000:00406FE8                 mov     edi, ebx  
> seg000:00406FEA                 shl     edi, 4  
> seg000:00406FED                 mov     \[esi+edi+0Ch\], cx  
> //var\_48\[ebx\].Blue=(((arg\_8+ebx\*4)^ shr $10)and $ff) shl 8  
> seg000:00406FF2                 mov     ecx, \[eax+ebx\*4\]  
> seg000:00406FF5                 shr     ecx, 18h  
> seg000:00406FF8                 shl     ecx, 8  
> seg000:00406FFB                 mov     edi, ebx  
> seg000:00406FFD                 shl     edi, 4  
> seg000:00407000                 mov     \[esi+edi+0Eh\], cx  
> //var\_48\[ebx\].Alpha=(((arg\_8^+ebx\*4) shr $18) shl 8) and $ffff  
> seg000:00407005                 inc     ebx  
> seg000:00407006                 cmp     ebx, 4  
> seg000:00407009                 jl      short loc\_406FAD  
> //loc\_406FAD 到此为一个循环  
> //for ebx:=0 to 3 do ....  
> //综合一下，arg\_8 的结构应该是 Array\[0..3\] of DWORD，元素的结构为 TColorRef，但是最高 8 位为 Alpha 值  
> seg000:0040700B                 mov     ecx, \[edx\]  
> seg000:0040700D                 mov     \[esi\], ecx  
> //看来 arg\_4 也是指针  
> //var48\[0\].x=arg\_4^  
> seg000:0040700F                 mov     ecx, \[edx+4\]  
> seg000:00407012                 mov     \[esi+4\], ecx  
> //var48\[0\].y=arg\_4^\[4\]  
> seg000:00407015                 mov     ecx, \[edx+8\]  
> seg000:00407018                 lea     ebx, \[esi+10h\]  
> seg000:0040701B                 mov     \[ebx\], ecx  
> //var48\[1\].x=arg\_4^\[8\]  
> seg000:0040701D                 mov     ecx, \[edx+4\]  
> seg000:00407020                 lea     ebx, \[esi+10h\]  
> seg000:00407023                 mov     \[ebx+4\], ecx  
> //var48\[1\].y=arg\_4^\[4\]  
> seg000:00407026                 mov     ecx, \[edx\]  
> seg000:00407028                 lea     ebx, \[esi+20h\]  
> seg000:0040702B                 mov     \[ebx\], ecx  
> //var48\[2\].x=arg\_4^  
> seg000:0040702D                 mov     ecx, \[edx+0Ch\]  
> seg000:00407030                 lea     ebx, \[esi+20h\]  
> seg000:00407033                 mov     \[ebx+4\], ecx  
> //var48\[2\].y=arg\_4^\[$c\]  
> seg000:00407036                 mov     ecx, \[edx+8\]  
> seg000:00407039                 lea     ebx, \[esi+30h\]  
> seg000:0040703C                 mov     \[ebx\], ecx  
> //var48\[3\].x=arg\_4^\[$8\]  
> seg000:0040703E                 mov     edx, \[edx+0Ch\]  
> seg000:00407041                 lea     ecx, \[esi+30h\]  
> seg000:00407044                 mov     \[ecx+4\], edx  
> //var48\[3\].y=arg\_4^\[$c\]  
> //看来 arg\_4 指向的应该是 rect 结构  
> seg000:00407047                 mov     edx, \[eax\]  
> seg000:00407049                 cmp     edx, \[eax+8\]  
> seg000:0040704C                 jnz     short loc\_407054  
> //if arg\_8^<>arg\_8^\[8\] then goto loc\_407054  
> seg000:0040704E                 mov     ecx, \[eax+4\]  
> seg000:00407051                 cmp     ecx, \[eax+0Ch\]  
> //无实际用途？  
> seg000:00407054  
> seg000:00407054 loc\_407054:                             ; CODE XREF: sub\_406F99+B3  
> seg000:00407054                 mov     edx, \[eax\]  
> seg000:00407056                 cmp     edx, \[eax+4\]  
> seg000:00407059                 jnz     short loc\_40706A  
> //if arg\_8^<>arg\_8^\[4\] then goto loc\_40706A  
> seg000:0040705B                 mov     ecx, \[eax+8\]  
> seg000:0040705E                 cmp     ecx, \[eax+0Ch\]  
> seg000:00407061                 jnz     short loc\_40706A  
> //if arg\_8^\[8\]<>arg\_8^\[$c\] then goto loc\_40706A  
> seg000:00407063                 mov     ebx, 1  
> //ebx=GRADIENT\_FILL\_RECT\_V  
> seg000:00407068                 jmp     short loc\_40706F  
> seg000:0040706A  
> seg000:0040706A loc\_40706A:                             ; CODE XREF: sub\_406F99+C0  
> seg000:0040706A                                         ; sub\_406F99+C8  
> seg000:0040706A                 mov     ebx, 2  
> //ebx=GRADIENT\_FILL\_TRIANGLE  
> seg000:0040706F  
> seg000:0040706F loc\_40706F:                             ; CODE XREF: sub\_406F99+CF  
> seg000:0040706F                 cmp     ebx, 2  
> seg000:00407072                 jnz     short loc\_407089  
> //if ebx<>GRADIENT\_FILL\_TRIANGLE then goto loc\_407089  
> seg000:00407074                 xor     eax, eax  
> seg000:00407076                 mov     \[ebp+var\_54\], eax  
> seg000:00407079                 mov     \[ebp+var\_50\], 1  
> seg000:00407080                 mov     \[ebp+var\_4C\], 2  
> //var\_54.Vertex1=0  
> //var\_54.Vertex2=1  
> //var\_54.Vertex3=2  
> seg000:00407087                 jmp     short loc\_407095  
> seg000:00407089  
> seg000:00407089 loc\_407089:                             ; CODE XREF: sub\_406F99+D9  
> seg000:00407089                 xor     edx, edx  
> seg000:0040708B                 mov     \[ebp+var\_8\], edx  
> seg000:0040708E                 mov     \[ebp+var\_4\], 3  
> //var\_8.UpperLeft=0  
> //var\_8.LowerRight=3  
> seg000:00407095  
> seg000:00407095 loc\_407095:                             ; CODE XREF: sub\_406F99+EE  
> seg000:00407095                 push    ebx             ; ULONG  
> seg000:00407096                 push    1               ; ULONG  
> seg000:00407098                 cmp     ebx, 2  
> seg000:0040709B                 jnz     short loc\_4070A2  
> seg000:0040709D                 lea     ecx, \[ebp+var\_54\]  
> seg000:004070A0                 jmp     short loc\_4070A5  
> seg000:004070A2  
> seg000:004070A2 loc\_4070A2:                             ; CODE XREF: sub\_406F99+102  
> seg000:004070A2                 lea     ecx, \[ebp+var\_8\]  
> seg000:004070A5  
> seg000:004070A5 loc\_4070A5:                             ; CODE XREF: sub\_406F99+107  
> seg000:004070A5                 push    ecx             ; PVOID  
> seg000:004070A6                 push    4               ; ULONG  
> seg000:004070A8                 push    esi             ; PTRIVERTEX  
> seg000:004070A9                 push    \[ebp+arg\_0\]     ; HDC  
> seg000:004070AC                 call    GradientFill  
> //注意 0040709B 的判断  
> seg000:004070B1                 cmp     ebx, 2  
> seg000:004070B4                 jnz     short loc\_4070D0  
> //如果 ebx=GRADIENT\_FILL\_TRIANGLE，那么继续  
> seg000:004070B6                 mov     \[ebp+var\_54\], 3  
> //var\_54.Vertex1=3  
> seg000:004070BD                 push    2               ; ULONG  
> //GRADIENT\_FILL\_TRIANGLE  
> seg000:004070BF                 push    1               ; ULONG  
> seg000:004070C1                 lea     eax, \[ebp+var\_54\]  
> seg000:004070C4                 push    eax             ; PVOID  
> seg000:004070C5                 push    4               ; ULONG  
> seg000:004070C7                 push    esi             ; PTRIVERTEX  
> seg000:004070C8                 push    \[ebp+arg\_0\]     ; HDC  
> seg000:004070CB                 call    GradientFill  
> seg000:004070D0  
> seg000:004070D0 loc\_4070D0:                             ; CODE XREF: sub\_406F99+11B  
> seg000:004070D0                 pop     edi  
> seg000:004070D1                 pop     esi  
> seg000:004070D2                 pop     ebx  
> seg000:004070D3                 mov     esp, ebp  
> seg000:004070D5                 pop     ebp  
> seg000:004070D6                 retn    0Ch  
> seg000:004070D6 sub\_406F99      endp  
>   
>     从调用代码看，这段代码没有返回值。下面的代码做了一点整理。  
>   
> 代码:
> 
> ```
>     
> Type
>   COLOR16 = Word;
> 
>   PTriVertex = ^TTriVertex;
>   \_TRIVERTEX = Packed Record
>     x: Longint;
>     y: Longint;
>     Red: COLOR16;
>     Green: COLOR16;
>     Blue: COLOR16;
>     Alpha: COLOR16;
>   End;
>   TTriVertex = \_TRIVERTEX;
>   TRIVERTEX = \_TRIVERTEX;
>   
>   AlphaRGB=Packed Record
>     Case Byte Of
>     1:(
>     Alpha:Byte;
>     Blue:Byte;
>     Green:Byte;
>     Red:Byte);
>     2:(
>     ARGB:LongWord);
>   End;
> 
> Procedure DoGradientFill(WinDC:HDC;DCRect:PRect;AlphaRGBArray:Array\[0..3\] of AlphaRGB);
> Var
>   GT:GRADIENT\_TRIANGLE;
>   TVs:Array\[0..3\] of TRIVERTEX;
>   GR:GRADIENT\_RECT;
>   i:LongWord;
> Begin
> 
>   For i:=0 To 3 Do
>     Begin
>       TVs\[i\].Red:=AlphaRGBArray\[i\].Red;
>       TVs\[i\].Green:=AlphaRGBArray\[i\].Green;
>       TVs\[i\].Blue:=AlphaRGBArray\[i\].Blue;
>       TVs\[i\].Alpha:=AlphaRGBArray\[i\].Alpha;
>     End;
> 
>   TVs\[0\].x:=DCRect^.Left;
>   TVs\[0\].y:=DCRect^.Top;
>   TVs\[1\].x:=DCRect^.Right;
>   TVs\[1\].y:=DCRect^.Top;
>   TVs\[2\].x:=DCRect^.Left;
>   TVs\[2\].y:=DCRect^.Bottom;
>   TVs\[3\].x:=DCRect^.Right;
>   TVs\[3\].y:=DCRect^.Bottom;
>   
>   If (DCRect^.Left=DCRect^.Top) AND (DCRect^.Right=DCRect^.Bottom) Then
>     Begin
>       GR.UpperLeft:=0;
>       GR.LowerRight:=3;
>       GradientFill(WinDC,@TVs,4,@GR,1,GRADIENT\_FILL\_RECT\_V);
>     End
>   Else
>     Begin
>       GT.Vertex1:=0;
>       GT.Vertex2:=1;
>       GT.Vertex3:=2;
>       GradientFill(WinDC,@TVs,4,@GT,1,GRADIENT\_FILL\_TRIANGLE);
> 
>       GT.Vertex1:=3;
>       GradientFill(WinDC,@TVs,4,@GT,1,GRADIENT\_FILL\_TRIANGLE);
>     End;
> 
> End;
> 
> ```
> 
>     小结：保持清醒的头脑是非常重要的。我曾经算错 mov 的地址，后来从头看才发现错误所在。  
>   
>     OK，休息。 

> **对\[FCG\]KeyGen2.0 按钮特效的分析 5**  
>   
> 这个过程应该比较简单。  
>   
> seg000:00406CA5 sub\_406CA5      proc near               ; CODE XREF: sub\_4070D9+18D  
> seg000:00406CA5  
> seg000:00406CA5 Rect            = tagRECT ptr -10h  
> seg000:00406CA5 arg\_0           = dword ptr  8  
> seg000:00406CA5  
> seg000:00406CA5                 push    ebp  
> seg000:00406CA6                 mov     ebp, esp  
> seg000:00406CA8                 add     esp, 0FFFFFFF0h  
> seg000:00406CAB                 push    ebx  
> seg000:00406CAC                 mov     ebx, \[ebp+arg\_0\]  
> seg000:00406CAF                 cmp     dword ptr \[ebx+0Ch\], 1  
> //arg\_0 又是指针   
> seg000:00406CB3                 jnz     short loc\_406CE6  
> seg000:00406CB5                 lea     eax, \[ebp+Rect\]  
> seg000:00406CB8                 push    eax             ; lpRect  
> seg000:00406CB9                 push    dword ptr \[ebx\] ; hWnd  
> seg000:00406CBB                 call    GetWindowRect  
> seg000:00406CC0                 push    0CC0020h        ; DWORD  
> seg000:00406CC5                 push    0               ; int  
> seg000:00406CC7                 push    0               ; int  
> seg000:00406CC9                 push    dword ptr \[ebx+8\] ; HDC  
> seg000:00406CCC                 mov     edx, \[ebp+Rect.bottom\]  
> seg000:00406CCF                 sub     edx, \[ebp+Rect.top\]  
> seg000:00406CD2                 push    edx             ; int  
> seg000:00406CD3                 mov     ecx, \[ebp+Rect.right\]  
> seg000:00406CD6                 sub     ecx, \[ebp+Rect.left\]  
> seg000:00406CD9                 push    ecx             ; int  
> seg000:00406CDA                 push    0               ; int  
> seg000:00406CDC                 push    0               ; int  
> seg000:00406CDE                 push    dword ptr \[ebx+4\] ; HDC  
> seg000:00406CE1                 call    BitBlt  
> seg000:00406CE6  
> seg000:00406CE6 loc\_406CE6:                             ; CODE XREF: sub\_406CA5+E  
> seg000:00406CE6                 cmp     dword ptr \[ebx+10h\], 0  
> seg000:00406CEA                 jz      short loc\_406CFD  
> seg000:00406CEC                 push    dword ptr \[ebx+10h\] ; HGDIOBJ  
> seg000:00406CEF                 push    dword ptr \[ebx+8\] ; HDC  
> seg000:00406CF2                 call    SelectObject  
> seg000:00406CF7                 push    eax             ; HGDIOBJ  
> seg000:00406CF8                 call    DeleteObject  
> seg000:00406CFD  
> seg000:00406CFD loc\_406CFD:                             ; CODE XREF: sub\_406CA5+45  
> seg000:00406CFD                 push    dword ptr \[ebx+8\] ; HDC  
> seg000:00406D00                 call    DeleteDC  
> seg000:00406D05                 push    dword ptr \[ebx+4\] ; hDC  
> seg000:00406D08                 push    dword ptr \[ebx\] ; hWnd  
> seg000:00406D0A                 call    ReleaseDC  
> seg000:00406D0F                 pop     ebx  
> seg000:00406D10                 mov     esp, ebp  
> seg000:00406D12                 pop     ebp  
> seg000:00406D13                 retn    4  
> seg000:00406D13 sub\_406CA5      endp  
>   
>     实在没有什么难度，注释都不必写。唯一值得注意的是，arg\_0 是个什么玩意。对照一下 sub\_406C06 的分析，可以看出，arg\_0 就是 WindowsDCInfo 的指针。  
>   
> 
> 代码:
> 
> ```
> Procedure sub\_406CA5(DCInfo:PWindowsDCInfo);
> Var
>   Rect:TRect;
> Begin
>   If DCInfo^.unknown=1 Then
>     Begin
>       GetWindowRect(DCInfo.hWindow,Rect);
>       BitBlt(DCInfo.WinDC,0,0,Rect.right-Rect.left,Rect.bottom-Rect.top,DCInfo.CompDC,0,0,SRCCOPY);
>     End;
>     
>   If DCInfo^.hBitmap<>0 Then
>     Begin
>       DeleteObject(SelectObject(DCInfo.CompDC,DCInfo.hBitmap));
>     End;
>     
>   DeleteDC(DCInfo.CompDC);
>   ReleaseDC(DCInfo.hWindow,DCInfo.WinDC);
> End;
> 
> ```
> 
>     看来这段代码是释放 sub\_406C06 生成的 DC 资源。  
>       
>     终于开始分析 sub\_4070D9 了：  
>   
> seg000:004070D9 sub\_4070D9      proc near               ; CODE XREF: sub\_406D7D+5C  
> seg000:004070D9                                         ; sub\_406D7D+89 ...  
> seg000:004070D9  
> seg000:004070D9 String          = byte ptr -124h  
> seg000:004070D9 var\_24          = dword ptr -24h  
> seg000:004070D9 hDC             = dword ptr -1Ch  
> seg000:004070D9 var\_18          = dword ptr -18h  
> seg000:004070D9 var\_14          = dword ptr -14h  
> seg000:004070D9 var\_10          = dword ptr -10h  
> seg000:004070D9 arg\_0           = dword ptr  8  
> seg000:004070D9  
> seg000:004070D9                 push    ebp  
> seg000:004070DA                 mov     ebp, esp  
> seg000:004070DC                 add     esp, 0FFFFFEDCh  
> seg000:004070E2                 push    ebx  
> seg000:004070E3                 push    esi  
> seg000:004070E4                 push    edi  
> seg000:004070E5                 mov     ebx, \[ebp+arg\_0\]  
> seg000:004070E8                 lea     edi, \[ebp+var\_10\]  
> seg000:004070EB                 mov     eax, \[ebx\]  
> //arg\_0 仍然是指针  
> seg000:004070ED                 mov     \[ebp+var\_24\], eax  
> //联系 004070FE 处代码，这里 eax 即 arg\_0^应该是窗口句柄。  
> seg000:004070F0                 xor     edx, edx  
> seg000:004070F2                 mov     \[ebp+var\_18\], edx  
> //unknown  
> seg000:004070F5                 xor     ecx, ecx  
> seg000:004070F7                 mov     \[ebp+var\_14\], ecx  
> //hBitmap  
> seg000:004070FA                 lea     eax, \[ebp+var\_24\]  
> seg000:004070FD                 push    eax  
> seg000:004070FE                 call    sub\_406C06  
> //内部调用  
> seg000:00407103                 push    edi             ; lpRect  
> seg000:00407104                 push    dword ptr \[ebx\] ; hWnd  
> seg000:00407106                 call    GetWindowRect  
> //var\_10 就是 Rect 了  
> seg000:0040710B                 mov     edx, \[edi\]  
> seg000:0040710D                 sub     \[edi+8\], edx  
> //var\_10.right-=var\_10.left  
> seg000:00407110                 mov     ecx, \[edi+4\]  
> seg000:00407113                 sub     \[edi+0Ch\], ecx  
> //var\_10.bottom-=var\_10.top  
> seg000:00407116                 xor     eax, eax  
> seg000:00407118                 mov     \[edi+4\], eax  
> //var\_10.top=0  
> seg000:0040711B                 mov     \[edi\], eax  
> //var\_10.left=0  
> seg000:0040711D                 test    byte ptr \[ebx+8\], 1  
> seg000:00407121                 jz      short loc\_407131  
> //if arg\_0^\[8\] and 1 =0 then goto loc\_407131  
> seg000:00407123                 push    0Fh             ; nIndex  
> //COLOR\_3DFACE  
> seg000:00407125                 call    GetSysColor  
> seg000:0040712A                 sub     eax, 202020h  
> seg000:0040712F                 jmp     short loc\_407136  
> seg000:00407131  
> seg000:00407131 loc\_407131:                             ; CODE XREF: sub\_4070D9+48  
> seg000:00407131                 mov     eax, 733C00h  
> seg000:00407136  
> seg000:00407136 loc\_407136:                             ; CODE XREF: sub\_4070D9+56  
> seg000:00407136                 push    eax             ; COLORREF  
> seg000:00407137                 call    CreateSolidBrush  
> seg000:0040713C                 mov     esi, eax  
> //esi=hBrush  
> seg000:0040713E                 push    esi             ; hbr  
> seg000:0040713F                 push    edi             ; lprc  
> seg000:00407140                 push    \[ebp+hDC\]       ; hDC  
> seg000:00407143                 call    FrameRect  
> //这里的 hDC 实际是 CompDC  
> seg000:00407148                 push    esi             ; HGDIOBJ  
> seg000:00407149                 call    DeleteObject  
> seg000:0040714E                 push    0FFFFFFFFh      ; dy  
> seg000:00407150                 push    0FFFFFFFFh      ; dx  
> seg000:00407152                 push    edi             ; lprc  
> seg000:00407153                 call    InflateRect  
> seg000:00407158                 test    byte ptr \[ebx+8\], 1  
> seg000:0040715C                 jz      short loc\_40717C  
> //if arg\_0^\[8\] and 1 =0 then goto loc\_40717C  
> seg000:0040715E                 push    0EAF4F5h        ; COLORREF  
> seg000:00407163                 call    CreateSolidBrush  
> seg000:00407168                 mov     esi, eax  
> seg000:0040716A                 push    esi             ; hbr  
> seg000:0040716B                 push    edi             ; lprc  
> seg000:0040716C                 push    \[ebp+hDC\]       ; hDC  
> seg000:0040716F                 call    FillRect  
> seg000:00407174                 push    esi             ; HGDIOBJ  
> seg000:00407175                 call    DeleteObject  
> seg000:0040717A                 jmp     short loc\_4071BA  
> seg000:0040717C  
> seg000:0040717C loc\_40717C:                             ; CODE XREF: sub\_4070D9+83  
> seg000:0040717C                 test    byte ptr \[ebx+8\], 2  
> //看来 arg\_0^\[8\]应该是位组合  
> seg000:00407180                 jz      short loc\_407186  
> seg000:00407182                 xor     eax, eax  
> seg000:00407184                 jmp     short loc\_4071A8  
> seg000:00407186  
> seg000:00407186 loc\_407186:                             ; CODE XREF: sub\_4070D9+A7  
> seg000:00407186                 test    byte ptr \[ebx+8\], 4  
> seg000:0040718A                 jz      short loc\_407190  
> seg000:0040718C                 xor     eax, eax  
> seg000:0040718E                 jmp     short loc\_4071A8  
> seg000:00407190  
> seg000:00407190 loc\_407190:                             ; CODE XREF: sub\_4070D9+B1  
> seg000:00407190                 test    byte ptr \[ebx+8\], 8  
> seg000:00407194                 jnz     short loc\_40719C  
> seg000:00407196                 test    byte ptr \[ebx+8\], 10h  
> seg000:0040719A                 jz      short loc\_4071A3  
> seg000:0040719C  
> seg000:0040719C loc\_40719C:                             ; CODE XREF: sub\_4070D9+BB  
> seg000:0040719C                 mov     eax, 2  
> seg000:004071A1                 jmp     short loc\_4071A8  
> seg000:004071A3  
> seg000:004071A3 loc\_4071A3:                             ; CODE XREF: sub\_4070D9+C1  
> seg000:004071A3                 mov     eax, 2  
> //上面这一段代码有点奇怪。特别是 40719A。怀疑是原作者削掉了一些条件值。   
> seg000:004071A8  
> seg000:004071A8 loc\_4071A8:                             ; CODE XREF: sub\_4070D9+AB  
> seg000:004071A8                                         ; sub\_4070D9+B5 ...  
> seg000:004071A8                 shl     eax, 4  
> seg000:004071AB                 add     eax, offset dword\_40A19C  
> //dword\_40A19C 应该是数组首地址。注意这里的 eax 计算，暗示这是 3\*4 的 AlphaRGB  
> seg000:004071B0                 push    eax             ; int  
> seg000:004071B1                 push    edi             ; int  
> seg000:004071B2                 push    \[ebp+hDC\]       ; HDC  
> seg000:004071B5                 call    sub\_406F99  
> seg000:004071BA  
> seg000:004071BA loc\_4071BA:                             ; CODE XREF: sub\_4070D9+A1  
> seg000:004071BA                 test    byte ptr \[ebx+8\], 2  
> seg000:004071BE                 jz      short loc\_4071CE  
> seg000:004071C0                 push    offset dword\_40A1CC ; int  
> seg000:004071C5                 push    edi             ; int  
> seg000:004071C6                 push    \[ebp+hDC\]       ; HDC  
> seg000:004071C9                 call    sub\_406F99  
> seg000:004071CE  
> seg000:004071CE loc\_4071CE:                             ; CODE XREF: sub\_4070D9+E5  
> seg000:004071CE                 test    byte ptr \[ebx+8\], 8  
> seg000:004071D2                 jz      short loc\_4071E7  
> seg000:004071D4                 push    0FFFFFFFFh      ; dy  
> seg000:004071D6                 push    0FFFFFFFFh      ; dx  
> seg000:004071D8                 push    edi             ; lprc  
> seg000:004071D9                 call    InflateRect  
> seg000:004071DE                 push    edi             ; lprc  
> seg000:004071DF                 push    \[ebp+hDC\]       ; hDC  
> seg000:004071E2                 call    DrawFocusRect  
> seg000:004071E7  
> seg000:004071E7 loc\_4071E7:                             ; CODE XREF: sub\_4070D9+F9  
> seg000:004071E7                 push    100h            ; nMaxCount  
> seg000:004071EC                 lea     edx, \[ebp+String\]  
> seg000:004071F2                 push    edx             ; lpString  
> seg000:004071F3                 push    dword ptr \[ebx\] ; hWnd  
> seg000:004071F5                 call    GetWindowTextA  
> //取得窗口标题，最多 256 字节  
> seg000:004071FA                 test    eax, eax  
> seg000:004071FC                 jz      short loc\_40725B  
> seg000:004071FE                 test    byte ptr \[ebx+8\], 2  
> seg000:00407202                 jz      short loc\_40720E  
> seg000:00407204                 push    1               ; dy  
> seg000:00407206                 push    1               ; dx  
> seg000:00407208                 push    edi             ; lprc  
> seg000:00407209                 call    OffsetRect  
> seg000:0040720E  
> seg000:0040720E loc\_40720E:                             ; CODE XREF: sub\_4070D9+129  
> seg000:0040720E                 test    byte ptr \[ebx+8\], 1  
> seg000:00407212                 jz      short loc\_40721B  
> seg000:00407214                 mov     ecx, 94A2A5h  
> seg000:00407219                 jmp     short loc\_40721D  
> seg000:0040721B  
> seg000:0040721B loc\_40721B:                             ; CODE XREF: sub\_4070D9+139  
> seg000:0040721B                 xor     ecx, ecx  
> seg000:0040721D  
> seg000:0040721D loc\_40721D:                             ; CODE XREF: sub\_4070D9+140  
> seg000:0040721D                 push    ecx             ; COLORREF  
> seg000:0040721E                 push    \[ebp+hDC\]       ; HDC  
> seg000:00407221                 call    SetTextColor  
> seg000:00407226                 push    0  
> seg000:00407228                 push    0  
> seg000:0040722A                 push    31h  
> //WM\_GETFONT  
> seg000:0040722C                 push    dword ptr \[ebx\]  
> seg000:0040722E                 call    SendMessageA\_0  
> seg000:00407233                 push    eax             ; HGDIOBJ  
> seg000:00407234                 push    \[ebp+hDC\]       ; HDC  
> seg000:00407237                 call    SelectObject  
> seg000:0040723C                 mov     esi, eax  
> seg000:0040723E                 push    25h  
> //DT\_SINGLELINE+DT\_VCENTER+DT\_CENTER  
> seg000:00407240                 push    edi  
> seg000:00407241                 push    0FFFFFFFFh  
> seg000:00407243                 lea     eax, \[ebp+String\]  
> seg000:00407249                 push    eax  
> seg000:0040724A                 push    \[ebp+hDC\]  
> seg000:0040724D                 call    DrawTextA\_0  
> seg000:00407252                 push    esi             ; HGDIOBJ  
> seg000:00407253                 push    \[ebp+hDC\]       ; HDC  
> seg000:00407256                 call    SelectObject  
> seg000:0040725B  
> seg000:0040725B loc\_40725B:                             ; CODE XREF: sub\_4070D9+123  
> seg000:0040725B                 mov     \[ebp+var\_18\], 1  
> seg000:00407262                 lea     edx, \[ebp+var\_24\]  
> seg000:00407265                 push    edx  
> seg000:00407266                 call    sub\_406CA5  
> seg000:0040726B                 pop     edi  
> seg000:0040726C                 pop     esi  
> seg000:0040726D                 pop     ebx  
> seg000:0040726E                 mov     esp, ebp  
> seg000:00407270                 pop     ebp  
> seg000:00407271                 retn    4  
> seg000:00407271 sub\_4070D9      endp  
>   
>     通过查找引用，可以知道 arg\_0 是 PWindowProperty 类型的。  
>   
> 代码:
> 
> ```
> Const
>   ARGBs:Array\[0..2\] of Array\[0..3\] of AlphaRGB=
>   (
>   ((ARGB:$A5B2B5),(ARGB:$CED7D6),(ARGB:$CED7D6),(ARGB:$DEEFF7)),
>   ((ARGB:$CEF3FF),(ARGB:$63CBFF),(ARGB:$63CBFF),(ARGB:$31B2FF)),
>   ((ARGB:$FAE3BA),(ARGB:$CCA372),(ARGB:$BC9362),(ARGB:$BE9463))
>   )
>   ;
> 
>   GradientARGBs:Array\[0..3\] of AlphaRGB=
>     ((ARGB:$DEE3E7),(ARGB:$E7E7E7),(ARGB:$DEE3E7),(ARGB:$FFDEE3E7));
> 
> Procedure sub\_4070D9(wp:PWindowProperty);
> Var
>   title:Array\[0..256\] of Char;
>   r:Rect;
>   DCInfo:WindowsDCInfo;
>   Brush:HBRUSH;
>   color:COLORREF;
>   clrindex:Byte;
>   fnt:HFONT;
> 
> Begin
>   DCInfo.hWindow:=wp^.CurrentWindow;
>   DCInfo.unknown:=0;
>   DCInfo.hBitmap:=0;
>   sub\_406C06(DCInfo);
> 
>   GetWindowRect(wp^.CurrentWindow,r);
>   r.right:=r.right-r.left;
>   r.bottom:=r.bottom-r.top;
>   r.top:=0;
>   r.left:=0;
> 
>   If wp^.Preserve2 AND 1<>0 Then
>     Begin
>       color:=GetSysColor(COLOR\_3DFACE)-$202020;
>     End
>   Else
>     Begin
>       color:=$733C00;
>     End;
> 
>   Brush:=CreateSolidBrush(color);
>   FrameRect(DCInfo.CompDC,r,Brush);
>   DeleteObject(Brush);
>   
>   InflateRect(r,-1,-1);
> 
>   If wp^.Preserve2 AND 1<>0 Then
>     Begin
>       Brush:=CreateSolidBrush($EAF4F5);
>       FillRect(DCInfo.CompDC,r,Brush);
>       DeleteObject(Brush);
>     End
>   Else
>     Begin
> 
>       If (wp^.Preserve2 AND 2=0) AND (wp^.Preserve2 AND 4=0) Then
>         Begin
>           clrindex:=2;
>         End
>       Else
>         Begin
>           clrindex:=0;
>         End;
> 
>       sub\_406F99(DCInfo.CompDC,r,ARGBs\[clrindex\]);
>     End;
>     
>   If wp^.Preserve2 AND 2<>0 Then
>     Begin
>       DoGradientFill(DCInfo.CompDC,r,GradientARGBs);
>     End;
> 
>   If wp^.Preserve2 AND 8<>0 Then
>     Begin
>       InflateRect(r,-1,-1);
>       DrawFocusRect(DCInfo.CompDC,r);
>     End;
> 
>   If GetWindowText(wp^.CurrentWindow,@title,$100)<>0 Then
>     Begin
>       If wp^.Preserve2 AND 2<>0 Then
>         Begin
>           OffsetRect(r,1,1);
>         End;
> 
>       If wp^.Preserve2 AND 1<>0 Then
>         Begin
>           clrindex:=94A2A5h
>         End
>       Else
>         Begin
>           clrindex:=0;
>         End;
>         
>       SetTextColor(DCInfo.CompDC,clrindex);
>       fnt:=SelectObject(DCInfo.CompDC,SendMessage(wp^.CurrentWindow,WM\_GETFONT,0,0));
>       DrawTextA(DCInfo.CompDC,@title,-1,r);
>       SelectObject(DCInfo.CompDC,fnt);
>     End;
> 
>   DCInfo.unknown:=1;
>   sub\_406CA5(DCInfo);
> End;
> 
> ```
> 
>     我不得不说，这段函数确实够长……换句话说，长得该死……  
>     不过，wp^.Preserve2 到底是什么玩意，确实值得分析。 

 4dd466d0e1e0cc123afbe504da2462ca