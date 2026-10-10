

# 关于壳中 APIHOOK 的一点点解析

> 原文：[`www.pediy.com/kssd/pediy11/120750.html`](https://www.pediy.com/kssd/pediy11/120750.html)

在看雪混了也有半年之余了,从一个完全不懂汇编只有一点 C 语言基础的人,慢慢地走来.如果没有这么多热心的人,和丰富的资源的吧,我可能早就放弃.随时到现在为止也依旧是一直大菜鸟,但还是一直想写篇文章来感谢一下看雪,顺带能帮助一下像我一样正在慢慢爬行的菜鸟们.可怜自己囊中羞涩啊,实在是没什么能看得出来给各位看官的:).  
          今天朋友让我脱一个壳,壳到现在还没有什么思路.不过看见了一下可对 API 的一些处理貌似是 HOOK 了 API 函数.一直对 HOOKAPI 充满着神秘感,看过几遍文章但是一直没有自己实践过什么.根据壳的思路+上自己的可怜的汇编知识+N 小时的调试.终于自己写了一个大概模拟这个思路的一个小汇编程序.主要的功能就是让我们下 API 断点也找不到到底是在哪里调用的.当然程序代码很简单,只是一个思路而已.这个壳的处理要麻烦很多,貌似还有用一个 key 值来解密。下面贴上代码：

引用:   .586  
  .model flat, stdcall  
  option casemap:none  
   
include       Windows.inc  
include       user32.inc  
include       kernel32.inc  
includelib   user32.lib  
includelib   kernel32.lib  
  
  
\_MessageBoxPtr      typedef  proto :dword,:dword,:dword,:dword  
\_MyMessageBox      typedef ptr MessageBoxPtr         
            .data  
szTitle       db          'APIHOOK 测试',0  
MyMessageBox  \_MyMessageBox    ?  
szText        db          '你找得到调用我的地方吗？',0  
szText1        db           '不在 HOOK 内了!',0  
  
            .code  
\_MyMessageCustom     proc hWnd:dword,buff1:dword,buff2:dword,Code:dword  
                  
                mov MyMessageBox,offset MessageBox  
                push eax        
                mov eax,dword ptr \[esp+1ch\]  
                mov dword ptr \[esp+8\],eax  
                pop  eax                  
                add esp,4  
                jmp MyMessageBox  
\_MyMessageCustom     endp  
        
start:  
                push offset \_Safeaddress  
                invoke \_MyMessageCustom,NULL,offset szText,offset szTitle,MB\_OK  
                db  6Ah ,0Ch ,68h,60h,0FFh ,0B9h,77h,0E8h ,0D0h,83h,0FAh ,0FFh ,33h ,0DBh ,53h ,6Ah ,01h ,8Dh ,45h ,0E7h ,50h ,6Ah ,11h ,6Ah ,0FEh ,5Eh ,56h ,0E8h ,0C4h ,6Ch ,0FAh ,0FFh  
                db  3Bh ,0C3h ,7Ch, 1Ch, 38h, 5Dh ,0E7h ,75h ,17h ,89h ,5Dh ,0FCh ,0CCh ,89h ,75h ,0FCh ,0EBh ,0Eh ,33h ,0C0h  
  
  
                            
  
\_Safeaddress:  
                invoke MessageBox,NULL,offset szText1,offset szTitle,MB\_OK  
                              
end start  
              
              
  
             

代码很简单,可能这个不算是 APIHOOK 吧?我也不太清楚,我也只看了几遍文章而已,程序之中加了一些不可能执行的代码,感觉壳里面也是这样忽悠我的,不过忽悠的技术更高明.![](img/110d920e4e703968a2a60c986d363163.jpg "尴尬")  
调试这个程序的时候可以直接下 MessageBox 断点看看,程序很烂,主要是记录一下自己的学习之路所用,大侠们见笑了.发帖前没怎么看有没有和我雷同的帖子,如有雷同纯属偶然.![](img/110d920e4e703968a2a60c986d363163.jpg "尴尬")  
还有这段代码是根据我脱的那个壳中想出来的,不知道其他壳是不是也是一样的.不过我感觉思路大概是这样的吧.!希望能有大大看到了以后,也发言两句,帮助一下我们这些菜鸟.![](img/234a76e67df951b6551d305c177a8b78.jpg "扮鬼脸")

上传的附件
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || test.rar  
 |