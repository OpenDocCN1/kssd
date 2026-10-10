

# CAJViewer7.0 中 OCR 功能的提取

> 原文：[`www.pediy.com/kssd/pediy10/64924.html`](https://www.pediy.com/kssd/pediy10/64924.html)

【文章标题】: CAJViewer7.0 中 OCR 功能的提取  
【文章作者】: yfliu  
【作者邮箱】: 80600414@qq.com  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
  CAJViewer7.0 中的文字识别功能相当强大，识别准确率很高，但该功能只局限在 CAJViewer7.0 能打开的有限的几种格式(nh,kdh,ceb,pdf)中，并不能直接用于识别图片。本文描述了一种方法，将 OCR 功能从 CAJViewer7.0 中剥离出来，独成一体。  
  工具：ida,syser debug,filemon, CAJViewer 7.0.1.sfx  
  
  一.为什么开始动手做  
  一直认为 caj 的 ocr 功能很强大，以前一直没做这件事是因为我以为他是同方自己开发的系统，和 caj 融为一体，提取难度较大。  
  今天偶然发现在 caj 的目录下有个 ocr 目录，里面有一个 exe 和若干 dll 文件，便猜想 ocr 功能只是 caj 的一个插件，既然是插件，难度就小多了。  
  
  二.首先想到的：ocr 以 dll 的方式向 caj 提供服务。  
  用 ida 检查导入表，有 LoadLibraryA，查看对该函数的引用，从加载的文件名和代码来看，并不是在加载 ocr 相关的代码，全是加载 caj 的文件。排除了这种可能  
  
  三.ocr 以单独的进程完成识别，然后将识别结果以某种方式传回 caj。  
  观察到：每当我使用 ocr 功能时，在任务管理器中总会多出一个 THOCRecog 的进程，然后很快地又消失了。  
  
  四.  
  既然是通过单独的进程来处理，完成后进程消失，那么程序得调用 CreateProcess 了。  
  打开 ida 发现了很多对 CreateProcessA 的调用  
  对于在 ida 中的每一个对 CreateProcessA 的调用：1.如果根据 lpCommandLine 或 lpApplicationName 不能判断出在创建 ocr 相关的进程。则 2.记下它的虚拟地址，在 syser 中下断点。如：  
  .text:004295FC                 add     esp, 28h  
  .text:004295FF                 lea     eax, \[ebp+3DCh+hObject\]  
  .text:00429602                 push    eax             ; lpProcessInformation  
  .text:00429603                 lea     eax, \[ebp+3DCh+StartupInfo\]  
  .text:00429606                 push    eax             ; lpStartupInfo  
  .text:00429607                 push    esi             ; lpCurrentDirectory  
  .text:00429608                 push    esi             ; lpEnvironment  
  .text:00429609                 push    8000000h        ; dwCreationFlags  
  .text:0042960E                 push    esi             ; bInheritHandles  
  .text:0042960F                 push    esi             ; lpThreadAttributes  
  .text:00429610                 push    esi             ; lpProcessAttributes  
  .text:00429611                 lea     eax, \[ebp+3DCh+CommandLine\]  
  .text:00429614                 push    eax             ; lpCommandLine  
  .text:00429615                 push    esi             ; lpApplicationName  
  .text:00429616                 call    ds:CreateProcessA ;   
  根据上下文是不能判断是否在创建 ocr 相关进程的。  
  在 syser 中切换到 caj 的空间，对 429616 下断点，然后查看 lpCommandLine,即 eax  
     
  在最上面就是内存窗口，在右上面显示就是 lpCommandLine 的值，可以看出这个 CreateProcess 函数在创建 ocr 相关进程。  
  还可以知道，caj 根据用户选择的区域，临时生成了一张 bmp 图像，然后传给 ocr 处理。  
  .text:00429615                 push    esi             ; lpApplicationName  
  .text:00429616                 call    ds:CreateProcessA ;   
  .text:0042961C                 test    eax, eax  
  .text:0042961E                 jnz     short loc\_42962F  
  .text:0042961E  
  .text:00429620                 push    offset ValueName  
  .text:00429625                 mov     ecx, edi  
  .text:00429627                 call    ds:ATL::CStringT<char,StrTraitMFC\_DLL<char,ATL::ChTraitsCRT<char>>>::CStringT<char,StrTraitMFC\_DLL<char,ATL::ChTraitsCRT<char>>>(char const \*)  
  .text:0042962D                 jmp     short loc\_42965E  
  .text:0042962D  
  .text:0042962F ; ---------------------------------------------------------------------------  
  .text:0042962F  
  .text:0042962F loc\_42962F:                             ; CODE XREF: sub\_429581+9D j  
  .text:0042962F                 push    0FFFFFFFFh      ; dwMilliseconds  
  .text:00429631                 push    \[ebp+3DCh+hObject\] ; hHandle  
  .text:00429634                 call    ds:WaitForSingleObject  
  .text:0042963A                 lea     eax, \[ebp+3DCh+ExitCode\]  
  .text:0042963D                 push    eax             ; lpExitCode  
  .text:0042963E                 push    \[ebp+3DCh+hObject\] ; hProcess  
  .text:00429641                 call    ds:GetExitCodeProcess  
  .text:00429647                 push    \[ebp+3DCh+hObject\] ; hObject  
  .text:0042964A                 mov     esi, ds:CloseHandle  
  .text:00429650                 call    esi ; CloseHandle  
  .text:00429652                 push    dword ptr \[ebp-38h\] ; hObject  
  .text:00429655                 call    esi ; CloseHandle  
  .text:00429657                 push    edi  
  .text:00429658                 call    sub\_4294E6  
  继续观察 CreateProcess 后的操作：程序将新创建的句柄传递给 WaitForSingleObject，这个函数等待新创建的进程结束；然后程序调用一个过程 sub\_4294E6。  
  这个过程主要是对系统的剪切板操作：  
  .text:004294E6 sub\_4294E6      proc near               ; CODE XREF: sub\_429581+D7 p  
  .text:004294E6                 mov     eax, offset loc\_450FAF  
  .text:004294EB                 call    \_\_EH\_prolog  
  .text:004294EB  
  .text:004294F0                 push    ecx  
  .text:004294F1                 and     dword ptr \[ebp-10h\], 0  
  .text:004294F5                 push    esi  
  .text:004294F6                 lea     ecx, \[ebp-10h\]  
  .text:004294F9                 call    ds:ATL::CStringT<char,StrTraitMFC\_DLL<char,ATL::ChTraitsCRT<char>>>::CStringT<char,StrTraitMFC\_DLL<char,ATL::ChTraitsCRT<char>>>(void)  
  .text:004294FF                 and     dword ptr \[ebp-4\], 0  
  .text:00429503                 push    offset szFormat ; "THOCR\_FMT"  
  .text:00429508                 call    ds:RegisterClipboardFormatA  
  .text:0042950E                 push    0               ; hWndNewOwner  
  .text:00429510                 mov     esi, eax  
  .text:00429512                 call    ds:OpenClipboard  
  .text:00429518                 test    eax, eax  
  .text:0042951A                 jnz     short loc\_42952C  
  .text:0042951A  
  .text:0042951C                 mov     ecx, \[ebp+8\]  
  .text:0042951F                 push    offset ValueName  
  .text:00429524                 call    ds:ATL::CStringT<char,StrTraitMFC\_DLL<char,ATL::ChTraitsCRT<char>>>::CStringT<char,StrTraitMFC\_DLL<char,ATL::ChTraitsCRT<char>>>(char const \*)  
  .text:0042952A                 jmp     short loc\_429568  
  .text:0042952A  
  .text:0042952C ; ---------------------------------------------------------------------------  
  .text:0042952C  
  .text:0042952C loc\_42952C:                             ; CODE XREF: sub\_4294E6+34 j  
  .text:0042952C                 push    esi             ; uFormat  
  .text:0042952D                 call    ds:GetClipboardData  
  .text:00429533                 mov     esi, eax  
  .text:00429535                 test    esi, esi  
  .text:00429537                 jz      short loc\_429555  
  .text:00429537  
  .text:00429539                 push    esi             ; hMem  
  .text:0042953A                 call    ds:GlobalLock  
  .text:00429540                 test    eax, eax  
  .text:00429542                 jz      short loc\_429555  
  .text:00429542  
  .text:00429544                 push    eax  
  .text:00429545                 lea     ecx, \[ebp-10h\]  
  .text:00429548                 call    ds:ATL::CStringT<char,StrTraitMFC\_DLL<char,ATL::ChTraitsCRT<char>>>::operator=(char const \*)  
  .text:0042954E                 push    esi             ; hMem  
  .text:0042954F                 call    ds:GlobalUnlock  
  .text:0042954F  
  .text:00429555  
  .text:00429555 loc\_429555:                             ; CODE XREF: sub\_4294E6+51 j  
  .text:00429555                                         ; sub\_4294E6+5C j  
  .text:00429555                 call    ds:CloseClipboard  
  小结一下 caj 的处理过程：  
  1.caj 通过 CreateProcessA 创建一个进程，THOCRecog  
  2.caj 调用 WaitForSingleObject,等待 THOCRecog 处理完成，THOCRecog 将处理结果放在系统剪切板中  
  3.caj 调用 GetClipboardData 从系统剪切板获得数据  
  4.caj 通过一些 mfc 的字符串函数解析第 3 步取得的数据  
  
  五.明白了 caj 的处理过程，怎么利用？  
  1.给 THOCRecog 传递一个图片的路径  
  2.待 THOCRecog 处理完后从系统剪切板取回数据。  
  还有两个问题：  
  1. THOCRecog 会判断它的父进程是否为 caj 吗？如果不是 caj 调用它，它还能工作吗？  
  2.系统剪切板中数据的格式的解析方式是什么？  
  
  六 .解决上面两个问题------ THOCRecog 的运行流程  
  在命令行方式下直接启动 THOCRecog，后跟一个图片文件路径，通过 filemon 发现 THOCRecog 同样会在图片路径创建一些文件，但是很快又删除了,怀疑这就是 THOCRecog 处理的结果。  
  如果我跳开 THOCRecog 中的删除文件的函数，就可以查看这些文件的内容。  
  找到 THOCRecog 中对 DeleteFileA 引用的部分。  
  .text:00401313 loc\_401313:                             ; CODE XREF: sub\_4010B0+1FD j  
  .text:00401313                                         ; sub\_4010B0+207 j  
  .text:00401313                 mov     edx, \[ebp+lpFileName\]  
  .text:00401316                 mov     esi, ds:DeleteFileA  
  .text:0040131C                 push    edx             ; lpFileName  
  .text:0040131D                 call    esi ; DeleteFileA  
  将 00401313 处的代码改为 retn。  
  观察图片目录，多了 4 个文件，其中一个和图片同名的 txt 中存放了识别的结果。  
  上面的处理达到了效果，但是显得很暴力。  
  分析该 DeleteFileA 所在的函数，可以推测 THOCRecog 大概的过程：  
  1.根据传入 bmp 文件路径，THOCRecog 分析该文件，将分析结果存在图片目录，与图片文件同名的 txt 文件中。  
  2. THOCRecog 读取该 txt 文件并把读取的内存规格为与 caj 约定的格式。  
  3. THOCRecog 将已经规格化的内容存在系统剪切板中。  
  4. THOCRecog 删除 txt 等图片目录下的 4 个文件。结束进程。  
  
  七.怎么利用  
  我对 mfc 不熟悉，没法逆向解析剪切板数据的相关过程，所以采取下面方法  
  .text:00401207                 test    eax, eax  
  .text:00401209                 jz      short loc\_401214  
  .text:00401209  
  .text:0040120B                 mov     byte ptr \[ebp+var\_4\], 9  
  .text:0040120F                 jmp     loc\_40135C  
  .text:0040120F  
  .text:00401214 ; ---------------------------------------------------------------------------  
  .text:00401214  
  .text:00401214 loc\_401214:                             ; CODE XREF: sub\_4010B0+159 j  
  .text:00401214                 call    sub\_401710  
  将.text:00401209                 jz      short loc\_401214 改为 nop 可以直接跳过上面的 4 个过程  
  然后用自己的程序读取数据，删除文件…  
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2008 年 05 月 16 日 11:50:48