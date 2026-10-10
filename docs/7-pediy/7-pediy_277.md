

# 用 MASM32 的 Pre-Built Dialog 给 PE 文件增加密码登陆框

> 原文：[`www.pediy.com/kssd/pediy10/64050.html`](https://www.pediy.com/kssd/pediy10/64050.html)

【文章标题】: 用 MASM32 的 Pre-Built Dialog 给 PE 文件增加密码登陆框  
【文章作者】: stalker  
【作者邮箱】: superstalker@163.com  
【编写语言】: 汇编语言  
【使用工具】: MASM32 v9.0  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
  在 MASM32 的 Library Reference 中可以找到 Pre-Built Dialogs，里面有一个函数为 GetTextInput，说明如下  
  GetTextInput proc hParent   :DWORD,Instance :DWORD,  
                    Icon      :DWORD,caption  :DWORD,  
                    subcaption:DWORD,lpbuffer :DWORD  
    
  Description  
  The GetTextInput dialog is used to obtain text from the user. It has the option to display an additional text field above the edit control where the user enters the text.  
    
  Parameters  
  1.  Parent  The handle of the windows that calls the dialog box.  
    
  2.  Instance  The instance handle for the application.  
    
  3.  Icon  The handle of the application's icon.  
    
  4. caption  A zero terminated string that contains the text to display on the title bar of the dialog.  
    
  5. subcaption  A zero terminated string that contains the text to display in the addition field above the edit conrol.  
    
  6. lpbuffer  The address of the buffer that will receive the text entered by the user. This buffer should be at least 128 bytes in length.  
    
  Return Value  
  The return value in EAX is always zero.  
    
  Comments  
  To test if the user has entered text, the first byte of the buffer should be tested for zero. If the value is not zero, the buffer contains the text typed by the user.  
    
  在论坛里面看过几篇给 PE 文件增加启动消息框和 Splash Screen 的文章，里面有提到用 DLL 来给 PE 文件增加功能比较简单，所以今天  
  我们也用 DLL 来实现给程序添加密码登陆框。  
    
  首先来编写 DLL，DLL 源文件的主结构如下  
  LibMain proc instance:DWORD,reason:DWORD,unused:DWORD   
    
      .if reason == DLL\_PROCESS\_ATTACH  
        push instance  
        pop hInstance  
        mov eax, TRUE  
    
      .elseif reason == DLL\_PROCESS\_DETACH  
    
      .elseif reason == DLL\_THREAD\_ATTACH  
    
      .elseif reason == DLL\_THREAD\_DETACH  
    
      .endif  
    
      ret  
    
  LibMain endp  
  其中 DLL\_PROCESS\_ATTACH 分支里的代码就是当 DLL 被加载时执行的代码，后面三个分支通过名字就应该很好理解了吧  
  我们就把验证密码的代码写在 DLL\_PROCESS\_ATTACH 分支里面  
    
  验证的流程很简单  
  首先调用 GetTextInput 获取用户输入，然后验证用户输入是否为预置密码，如果密码正确，则返回 PE 文件继续执行下面的代码，  
  如果不正确，直接调用 ExitProcess 退出进程  
    
  整个 DLL 的源代码如下：  
  ;by stalker  
  
        .486                        
        .model flat, stdcall        
        option casemap :none        
    
        include \\masm32\\include\\windows.inc  
        include \\masm32\\include\\masm32.inc  
        include \\masm32\\include\\user32.inc  
        include \\masm32\\include\\kernel32.inc  
     
    
        includelib \\masm32\\lib\\masm32.lib  
        includelib \\masm32\\lib\\user32.lib  
        includelib \\masm32\\lib\\kernel32.lib  
    
    
    
    
        .data?  
          hInstance dd ?  
          szInput db 128 dup(?)  
        .data  
          szLoginCode db "opensesame",0  
          szCaption db "登陆",0  
          szSubCaption db "请输入密码",0  
        .code  
    
  
  
    
  LibMain proc instance:DWORD,reason:DWORD,unused:DWORD   
    
      .if reason == DLL\_PROCESS\_ATTACH  
        invoke GetTextInput,0,0,0,offset szCaption,offset szSubCaption,offset szInput  
        invoke szCmp,offset szLoginCode,offset szInput;szCmp 也是 masm32 中的函数，自己实现这个函数也可以，就是比较两个字符串是否相同  
        or eax,eax  
        jz @f  
        push instance  
        pop hInstance  
        mov eax, TRUE  
        ret  
        @@:  
        invoke ExitProcess,NULL  
      .elseif reason == DLL\_PROCESS\_DETACH  
    
      .elseif reason == DLL\_THREAD\_ATTACH  
    
      .elseif reason == DLL\_THREAD\_DETACH  
    
      .endif  
    
      ret  
    
  LibMain endp  
    
  
  
    
  end LibMain  
    
  DLL 编译好了，如何让一个 exe 文件加载它呢，可以使用工具如 LoadPE 等（如果要使用工具添加的话，好象至少要在 def 文件中输出  
  一个函数)  
  我们就用手工的方法来添加吧，以系统自带的 calc.exe 为例，首先我们用 TOPO 给它增加一点空间，并改写它的入口地址  
  然后用 OD 载入，写入如下代码  
  0101E60A >  6A 6C           push    6C  
  0101E60C    68 6E2E646C     push    6C642E6E  
  0101E611    68 4C6F6769     push    69676F4C；压入的是"Login.dll"的 ASCII 码  
  0101E616    54              push    esp  
  0101E617    FF15 24100001   call    dword ptr \<&KERNEL32.LoadLibrar>; kernel32.LoadLibraryA  
  0101E61D    83C4 0C         add     esp, 0C；调整好堆栈  
    
  然后复制到可执行文件，保存，运行之，出现了登陆密码框，大功告成！  
    
  由于我还没有权限上传附件,所以请 dll 的源码和 def 文件,还有修改后的 calc.exe,请大家在下面这个地址下载  
http://rapidshare.de/files/39278383/login.rar.html  
  
谢谢版主加精，现在有权限了，把附件传上来  
[login.rar \[解压密码：pediy\]   
    
\--------------------------------------------------------------------------------  
【经验总结】  
  这只是一个很简单的例子，密码是明文保存的，大家可以继续拓展，可以先把密码加密，然后获取用户输入，进行同样的加  
  密之后再进行比较等等。  
  另外，我本来是想再写一个程序直接往 PE 文件中注入用 OD 写入的那几句代码的，无奈功底尚浅，暂时还无法做到。  
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2008 年 05 月 01 日 9:43:57