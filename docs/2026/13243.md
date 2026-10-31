
# [原创]为任意程序添加SplashScreen

- 【文章标题】:为任意程序添加SplashScreen
- 【文章作者】: Kangaroo
- 【软件名称】: 任意
- 【下载地址】: 看附件
- 【使用工具】: VC、LoarPE
- 【操作平台】: Winsp2
- 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教,该文章本人已在黑客防线2006年第8期发表。

相信玩过逆向工程的人，都会给任意的`exe`文件加入`MessageBox`吧(也就是在程序启动前先弹出个`MessageBox`对话框，当用户点击确定的时候，进入程序)，我先总结下上述过程的实现方法，然后进入本文的主题:

一、在`.data`段找到cave地方，输入`MessageBox`显示的字符串

二、在`.text`段找到cave地方（主要用来写patch代码）

```asm
.004012B4: 6A00         push 000 （类型）
.004012B6: 682C314000   push 00040312C（标题字符串地址）
.004012BB: 6848314000   push 000403148（内容字符串地址）
.004012C0: 6A00         push 000（句柄）
.004012C2: FF1518204000 call MessageBoxA（调用函数）
.004012C8: E933FDFFFF   jmp .000401000（跳到原oep处）
```

三、用`LordPE`将程序的`OEP`修改为`004012B4`

至此就成功的插入了个`MessageBox`

好了，费话说了那么多，现在我们的主要目标不是为`exe`文件插入`MessageBox`，而是为`exe`文件插入`splashscreen`(程序启动界面，例如Photoshop、Flash等大型软件都有漂亮的启动界面)，相信大家目前已经想到用上面的方法同样实现吧，别急，想想可行性:

一、需要调用`CreateWindow`来创建窗口，并为该窗口指定一个消息处理函数，然后在`WM_CREATE`处调用`SetTimer`创建时间器，接着在`WM_PAINT`分支加载图片，在`WM_TIMER`分支处关闭窗口，当窗口关闭时需要回到原程序入口点，继续执行，使原程序能够正常工作（SDK编程基础）

二、如果上述需要用的函数在输入表中没有的话，那就需要我们自己去导入这些函数了

上述问题就意味着你需要patch大量的代码，这样即使你有足够的精力去做的话，那也必将浪费掉大量的时间如何快速插入一个splashscreen这将是我们最大的挑战。

**分析：**

分析下PE装载器的工作原理，相信会有事半功倍的效果，PE装载首先找到输入表中程序需要用到的函数，然后并将该函数所在的DLL加载到进程空间中（在加载DLL到进程空间时，PE装载器将会调用DLL中的`DllMain`函数，执行完该函数后，继续加载其它动态库，最后进入程序入口点执行程序....）

哈哈，这不是我们正想要的地方吗？如果我们在`DllMain`中调用`DialogBoxParam`函数，在资源中创建对话框（`DialogBoxParam`函数直到对话关闭时才返回，这意为着当我们的对话没有关闭前，`DllMain`函数不可能执行完毕，根据上面PE装载器工作过程的分析在对话框关闭前程序不可能执行，此时PE装载器在等`DllMain`函数退出嘛，然后在继续加载其它模块，最后主程序才开始执行。）这样不正好产生了SplashScreen效果!!!

**实现：**

创建个dll工程，加入如下代码：

```cpp
#include "stdafx.h"

int DummyFunc() //空函数
{
    return 0;//这个空函数一定需要，我们把它放入到dll的输出表中
    //因为等下我们需要把该函数加入到我们需要加添加splashscreen的程序的输入表中，如果该dll一个输出函数都没有，那我们将无法为原程序添加输入函数，也间接意味着，PE装载器不会加载我们写的dll到程序的进程空间中去。
}

BOOL DlgProc(HWND hwnd, UINT msg, WPARAM wparam, LPARAM lparam)
{
    DWORD TimerID;
    switch(msg)
    {
        case WM_CLOSE:
        {
            KillTimer(hwnd, TimerID);
            EndDialog(hwnd, 0);
        }
        break;
        case WM_INITDIALOG:
        {
            int ScreenWidth,ScreenHeight,StartPosX,StartPosY,WndWidth,WndHeight;
            RECT WndRect;
            ScreenWidth=::GetSystemMetrics(SM_CXSCREEN);
            ScreenHeight=::GetSystemMetrics(SM_CYSCREEN);
            ::GetWindowRect(hwnd,&WndRect);
            WndWidth=WndRect.right-WndRect.left;
            WndHeight=WndRect.bottom-WndRect.top;
            TimerID = SetTimer(hwnd, 466, 3000, NULL);
//呵呵，这里插入点汇编，主要是跟大家介绍下，右移是除以二，左移是乘以二（移位指令比除法指令快的，希望大家能接受这一思想），这顺便介绍了如何在C++中插入汇编。下面这段算法主要是将窗口显示在屏幕的中央。
            __asm{
                mov    eax,ScreenWidth
                sub    eax,WndWidth
                shr    eax,1   //右移1位等价于除以2
                mov    StartPosX,eax
                mov    eax,ScreenHeight
                sub    eax,WndHeight
                shr    eax,1
                mov    StartPosY,eax
            }
//上面代码的等价C++代码如下：
// StartPosX=(ScreenWidth-WndWidth)>>1;
// StartPosY=(ScreenHeight-WndHeight)>>1;
            SetWindowPos(hwnd,HWND_TOPMOST,StartPosX,StartPosY,WndWidth,WndHeight,SWP_HIDEWINDOW);
        }
        break;
        case WM_TIMER:
        {
            SendMessage(hwnd, WM_CLOSE,0,0);
        }
        break;
        default:break;    }
    return FALSE;
}

BOOL APIENTRY DllMain(HANDLE hModule, DWORD fdwReason,LPVOID lpReserved)
{
    switch(fdwReason)
    {
        case DLL_PROCESS_ATTACH:
        {
            DialogBoxParam((HINSTANCE)hModule, MAKEINTRESOURCE(102), NULL, (DLGPROC)DlgProc, NULL);
        }
        break;
        case DLL_PROCESS_DETACH:
        {
        }
        break;
    }
    return TRUE;
}
```

一定需要把建立个def文件，将`DummyFunc`函数导出

Def文件内容如下:

```def
LIBRARY
EXPORTS
   DummyFunc
```

OK! 上面已经给出了主要代码了，至于资源方面的设计相信大家都会，这里就不在多说。(具体见源代码)接下来，我们需要讲的是，如何使用我们这个`dll`，刚才不是说了吗？需要将我们刚才输出的`DummyFunc`添加到我们需要修改的程序的输入表中，手动添加当然可以（这要求对PE文件的结构有足够的了解），不过现在教大家用工具快速添加，打开`LordPE`，加载我们需要修改的`exe`文件，如图一:

![图一](http://bbs.pediy.com/upload/2006/4/image/1.jpg_605.jpg)

选择`Directories`按钮，弹出对话框，如图二:

![图二](http://bbs.pediy.com/upload/2006/4/image/2.jpg_681.jpg)

选择`ImportTable`右边的…按钮，此时再次弹出个对话，我们就可以为该程序加入函数了。如图三:

![图三](http://bbs.pediy.com/upload/2006/4/image/3.jpg_740.jpg)

点击`add import`，在弹出的对话框中，如下设置(如图四)

![图四](http://bbs.pediy.com/upload/2006/4/image/4.jpg_783.jpg)

选择`+`按钮，此时成功加入，OK至此，我们已经成功的为一个程序加入了SplashScreen了如果想为其它程序加入，此时只要带上我们写的动态库，和`LordPE`就可以快速搞定，哈哈，爽吧!

**总结：**

程序中给出了两种方案，第一种方案只适用于写patch代码量比较小的程序，而第二种方案你就能彻底的为程序添加丰富的功能了，包括木马的后门等，这就需要大家多动动脑，在此我就不多说了。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
