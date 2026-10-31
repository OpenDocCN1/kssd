
# [原创]老鸟飞过。。。来坛里几个月了发一下新手学习笔记（未完。。）

目的：学习

发帖目的：共同学习和记录学习（每星期希望都能坚持发。。）

内容：对反跟踪技术原理的学习。。

虽然看雪那本书上讲了很多，但本菜还是认为 自己动手 坐一下比较好，虽然很简单。。

## 第一节：对peb标志位的检测

![](https://bbs.kanxue.com/upload/attach/201012/356233_nlkfhn9ujfowd2d.jpg)

相应扩展习题：（在双进程保护中进行解除） [`Exam_CrackMe4.rar`](https://bbs.kanxue.com/attach-download-52110.htm)

相应示例代码：

```c
#include <stdio.h>
#include <Windows.h>

BOOL WINAPI IsDebuggerPresent(void);

int WINAPI WinMain(          HINSTANCE hInstance,
    HINSTANCE hPrevInstance,
    LPSTR lpCmdLine,
    int nCmdShow
)
{
        HWND        hDebug;
        if(IsDebuggerPresent())
        {
                hDebug=GetForegroundWindow();
                SendMessage(hDebug,WM_CLOSE,NULL,NULL);                //终止od
                return        FALSE;
        }
        MessageBox(NULL,"NOT FOUND OD!","SEARCH",MB_OK);
                return        TRUE;
}
```

很简单 自己动手，但也可以学很多东西。。

## 2. 调试时间限制

加密解密上提到过：但没有真正让我感受到 所以自己就动手写了：

![](https://bbs.kanxue.com/upload/attach/201012/356233_g94mfoisjck0szu.jpg)

示例源码：

```c
#include <stdio.h>
#include <windows.h>

int WINAPI WinMain(          HINSTANCE hInstance,
    HINSTANCE hPrevInstance,
    LPSTR lpCmdLine,
    int nCmdShow
)
{
        DWORD        dTime1;
        DWORD        dTime2;
        dTime1=GetTickCount();
        GetCurrentProcessId();//加入你自己的认为可能会断下的地方放入 获取tickcount函数之间
        GetCurrentProcessId();
        GetCurrentProcessId();
        GetCurrentProcessId();
        GetCurrentProcessId();
        dTime2=GetTickCount();
        if(dTime2-dTime1>100)        // 如果时延过大判定 其为处于调试状态
        {
                MessageBox(NULL,"FOUND!","SEARCH",MB_OK);
        }
        else
        {
                MessageBox(NULL,"Not FOUND!","SEARCH",MB_OK);
        }
        return TRUE;
}
```

未完待续。。。。

希望在雪坛里里面，加速进步。。。。![](https://bbs.kanxue.com/view/img/face/27.gif)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [`chap1.jpg`](javascript:void(0);) （144.83kb，153次下载）
- [`2.jpg`](javascript:void(0);) （111.06kb，149次下载）
- [`Exam_CrackMe4.rar`](javascript:void(0);) （3.98kb，13次下载）
