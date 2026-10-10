

# 为程序添加拖放功能

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-427.htm`](https://www.pediy.com/kssd/pediy09/pediy09-427.htm)

> 【文章标题】: 为程序添加拖放功能  
> 【文章作者】: Suyana  
> 【作者邮箱】: suyasha@163.com  
> 【参考信息】: pll621 写的为 W32dasm 添加拖放文件功能  
>   
> \==========添加函数================  
> 用 PE 编辑工具加入下面这 3 个函数(位于 SHELL32.DLL):  
> DragAcceptFiles、DragFinish、DragQueryFile。记下它们的 RVA。  
> 在 00401A08 写入:  
> 00401A08  jmp     \[405074\]  ;  SHELL32.DragAcceptFiles  
> 00401A0E  jmp     \[405078\]  ;  SHELL32.DragFinish  
> 00401A14  jmp     \[40507C\]  ;  SHELL32.DragQueryFileA  
> 以后 call 00401A08 就是调用 SHELL32.DragAcceptFiles 函数了  
>   
> DragAcceptFiles 是设置窗口能否接受拖放消息，即能否发送 WM\_DROPFILES 消息给这个窗口  
>   
> \=========设置窗口能否接受拖放消息=====================  
> 1.设置窗口能否发送拖放消息。  查找 CreateWindowExA"。有两个，一个是创建窗口，  
>   另一个是创建 RichEdit。来到创建窗口的那条：  
> 
> 代码:
> 
> * * *
> 
> `  004018BA  mov     [40302C], eax         ;主窗口的句柄     004018BF  push    1                   ;改成 jmp     00401A9B     004018C1  push    dword ptr [40302C]  ;被覆盖`
> 
> * * *
> 
>   在 00401A9B 写入：  
> 代码:
> 
> * * *
> 
> `  00401A9B  push    1             ; /Accept = TRUE     00401A9D  push    dword ptr [40302C]  ; |hWnd = NULL     00401AA3  call    00401A08      ; \DragAcceptFiles     00401AA8  push    1     00401AAA  push    dword ptr [40302C]     00401AB0  jmp     004018C7`
> 
> * * *
> 
>       -------------------------------------------------------  
> 2.设置 RichEdit 能否发送拖放消息(可以不改，因为改了也没用，见本文最后一段)  
>   查找另一个"CreateWindowExA"。来到：  
>   
> 代码:
> 
> * * *
> 
> `  004014D1  push    200                                ; |ExtStyle = WS_EX_CLIENTEDGE     004014D6  call    <jmp.&user32.CreateWindowExA>    ; \CreateWindowExA     004014DB  mov     [403034], eax         ;RichEdit 的句柄     ...     00401547  call    00401990                         ; \SendMessageA=>jmp 00401A20     0040154C  leave     0040154D  retn`
> 
> * * *
> 
>   在函数末把 00401547  call    00401990 改成 jmp 00401A20  
>   在 00401A20 处写入：  
>  代码:
> 
> * * *
> 
> ` 00401A20  pushad     00401A21  push    1     00401A23  push    dword ptr [403034]     00401A29  call    00401A08    ;SHELL32.DragAcceptFiles     00401A2E  popad     00401A2F  call    00401990    ;user32.SendMessageA     00401A34  jmp     0040154C`
> 
> * * *
> 
> 保存，打开程序，拖个文件到 RichEdit，图标是不是已经改变了  
>   
> \=========加入判断拖放消息的代码===================  
> 首先要找到消息处理的地方，加入判断拖放消息的代码。  
> 查找"DefWindowProcA"，双击来到：004017BF 这里就是消息处理函数了。向上看，只有  
>   004017AA  jnz     short 004017B3 跳到这里("DefWindowProcA")，把它改成 jmp 00401A3D  
> 在 00401A3D 写入：  
>   代码:
> 
> * * *
> 
> `00401A3D  jnz     short 00401A49     00401A3F  call    0040154E     00401A44  jmp     004017B1     00401A49  cmp     eax, 233     00401A4E  je      short 00401A55  ;是拖放消息跳     00401A50  jmp     004017B3   ;转到默认的处理函数        00401A55  nop               ;拖放消息，这只是临时的代码，以后会替换     00401A56  jmp     004017B3   ;成打开文件的代码，现在运行才不会出错`
> 
> * * *
> 
> 现在程序能判断拖放消息，我们就需要根据这个消息做相关的动作了，也就是打开这个拖放文件。   
> 首先我们必须得到拖放文件的文件名，这就需要 DragQueryFile 函数了   
>   
>  引用:
> 
> | 
> 
> * * *
> 
>  UINT DragQueryFile(   
>     HDROP hDrop,  // handle to structure for dropped files -> 拖放的 hdrop   
>     UINT iFile,          // index of file to query                     -> 为 0 就可以了   
>     LPTSTR lpszFile,      // buffer for returned filename          -> 存放文件名的缓冲区   
>     UINT cch         // size of buffer for filename          -> 缓冲区的尺寸   
>   );  
> 
> * * *
> 
>  |
> | --- |
> 
> 现在需要找的就是 hDrop 参数，这个也就是 msg 的子参数：  
>   00401587  mov     eax, \[ebp+C\]                   ;ebp+C 是 msg，那么这个子参数就在 ebp+10 处  
>   
> \============下一步就是怎么打开文件======================  
> 程序用 CreateFileA 打开文件，查找该函数，来到：  
>   
>  代码:
> 
> * * *
> 
> ` 004010E7  call    <jmp.&comdlg32.GetOpenFileNameA> ;显示打开文件对话框     004010EC  or      eax, eax     004010EE  je      0040117E      ;是否有选择一个文件     004010F4  push    0            ; /hTemplateFile = NULL     004010F6  push    80           ; |Attributes = NORMAL     004010FB  push    3            ; |Mode = OPEN_EXISTING     004010FD  push    0            ; |pSecurity = NULL     004010FF  push    3            ; |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE     00401101  push    C0000000  ; |Access = GENERIC_READ|GENERIC_WRITE     00401106  push    00403044  ; |FileName = ""     0040110B  call    004019BA     ; \CreateFileA     00401110  cmp     eax, -1     00401113  jnz     short 0040112B`
> 
> * * *
> 
> 看上面的代码可以发现，在得到拖放的文件名后直接跳到 004010F4，但不能执行上面的代码，否则会弹出对话框。所以我们自己写一个函数(不然会出错)，跳到 004010F4 就可以了。然后，我写了代码并成功了。但是，要是文件有修改过，程序在打开其它文件前会问你是否保存或取消，但拖放的文件却不会，而是直接打开。所以，我们不直接跳到 004010F4，而是先判断文件是否修改、是否要保存，才决定是否打开文件。看上去好像要写很多代码，实际上不用，因为原程序已经写好了。  
>   
> 在"CreateFileA"上，下断点，运行，打开一个文件，程序中断，按 F8 直到程序返回，返回到：  
>   
> 代码:
> 
> * * *
> 
> `  004015D2  call    00401180  ;判断文件是否修改，若有，给出提示。     004015D7  or      eax, eax     004015D9  je      004017CB  ;按"取消"，不打开新的文件     004015DF  call    0040109F  ;打开文件，有文件对话框`
> 
> * * *
> 
> 开始写代码。知道写在哪吗?在 00401A55，忘记的话看上面：《===加入判断拖放消息的代码===》  
>   
>  代码:
> 
> * * *
> 
> ` 00401A55  push    dword ptr [ebp+10]  ;hDrop     00401A58  pop     dword ptr [403FFC]     00401A5E  push    100                 ; /缓冲区大小，不要拖放路径大于 255 字符的文件     00401A63  push    00403044         ; |Buffer = Richedit.00403044     00401A68  push    0                  ; |FileIndex = 0     00401A6A  push    dword ptr [403FFC]  ; |hDrop = NULL     00401A70  call    00401A14           ; \DragQueryFile     00401A75  call    00401AB9           ; 我们要写的自定义函数     00401A7A  jmp     004017B3         ; 跳到消息循环`
> 
> * * *
> 
> 在 00401AB9 写入我们自定义的函数：  
>   
>  代码:
> 
> * * *
> 
> ` 00401AB9  push    ebp     00401ABA  mov     ebp, esp     00401ABC  add     esp, -58      ;以上不要的话好像会错，函数开头一般都是这样滴!     00401ABF  call    00401180     ;判断文件是否修改，若有，给出提示。     00401AC4  or      eax, eax     00401AC6  je      004017CB    ;不打开文件     00401ACC  jmp     004010F4  ;打开文件     00401AD1  push    dword ptr [403FFC]    ; /hDrop = NULL     00401AD7  call    00401A0E             ; \DragFinish     00401ADC  retn`
> 
> * * *
> 
> 保存，试下吧，拖一个文件看一下吧。若上一个文件有修改过，还可以选择"保存"、"不保存"、  
> "不打开"等操作。反正我是成功了。这里有原程序和我修改过的程序。看一下吧。![](img/768c8e05987a6949d1bb5f7378212ee0.jpg "狂笑")   
>   
> 还有一点要说明，拖放文件时要拖放到程序的标题栏，这样才会打开文件。调试了一下发现拖放到 RichEdit 时，窗口不会发送"WM\_DROPFILES"消息，自然也就无法打开文件了。![](img/26c3a0df6b585d53c82b982893c84a26.jpg "伤心")   
> 可是我明明让 RichEdit 也能拖放文件，但为什么就是没有发送消息呢?   
> 谁能告诉我呢? ![](img/b1b6155d5e6bc11d8426f301b224afc3.jpg "124.gif")   
>   
> \========附件内容===============  
> Richedit 原程序.exe : 《Windows 环境下 32 位汇编语言程序设计》(著:罗云彬)的例子程序  
> Richedit.exe     : DIY PE 的成果，添加了拖放文件的功能。by Suyana  
> pll621-01.htm     : 偶的参考资料 

 4dd466d0e1e0cc123afbe504da2462ca