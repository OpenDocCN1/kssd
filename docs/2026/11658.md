
# [原创]突发奇想！让windows不再windows

希望不要随便利用：汇编代码（附件是演示的程序，最多两分钟后会安全返回）

```
.386

.model flat,stdcall

option casemap:none

include  windows.inc

include  kernel32.inc

includelib  kernel32.lib

include  user32.inc

includelib  user32.lib

.data?

hwnd dd ?

hwdesk dd ?

mesg  MSG <>

.const

szClassName        db        'Progman',0

szCaptionMain        db        'Program Manager',0

.code

start:

   invoke FindWindow,addr szClassName,addr szCaptionMain

   mov hwdesk,eax

   invoke SetTimer,NULL,1,500,0

   .while 1

   invoke GetMessage,addr mesg,NULL,0,0

    .if mesg.message==WM_TIMER

    invoke GetForegroundWindow

     .if eax!=hwdesk

       mov hwnd,eax

       invoke IsWindowVisible, hwnd

         .if eax

      invoke ShowWindow,hwnd,SW_HIDE

         .endif

     .endif

    .endif

    .endw

    invoke KillTimer,NULL,1

    invoke ExitProcess,NULL

end start
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#其他内容](https://bbs.kanxue.com/forum-4-1-10.htm)

上传的附件：

- [nowindow.rar](javascript:void(0);)

（13.85kb，523次下载）
