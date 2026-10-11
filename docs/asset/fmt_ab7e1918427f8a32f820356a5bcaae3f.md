
# 自己动手，开发 VS 对战平台挤房工具[已更新,后台挤房见 22 楼]

> 原文：[`www.pediy.com/kssd/pediy11/104655.html`](https://www.pediy.com/kssd/pediy11/104655.html)

相信经常上 VS 对战平台玩游戏的朋友都曾有过下面这样的经历

![](img/ebb7bfc298a3dc92bbdab7c4aafd47df.jpg)

过程非常痛苦！！！怎么办？难道花钱充 VIP？做个 RMB 玩家固然感觉非常好，但 stalker 还是个学生，没有收入，把父母的血汗钱花到游戏上，实在不忍心。

于是想到用按键精灵写一个脚本，虽然能代替手工操作，但生成小精灵广告频弹，非常不爽。stalker 一直很讨厌广告，并不是因为广告烦人，而是因为自己没有钱购买广告中的商品![](img/45dd044a2743704cbb333715eae4a5f6.jpg "狂笑")

恰好最近正在学习 C++程序设计，于是打算自己动手开发一个程序，取代那令人纠结的小精灵。

## 首先我设计了一个类，如下

代码:

```cpp
//SimulatingActions.h

class SimulatingActions
{
  public:
      void MouseDown(bool bLeftRight);
      void MouseUp(bool bLeftRight);
      void MouseClick(bool bLeftRight);
      void MouseDbClick(bool bLeftRight);
      void KeyDown(char vKey);
      void KeyUp(char vKey);
      void KeyHit(char vKey);
};
```

从类的成员函数名称，很容易看出它们的作用

前四个成员函数，分别模拟鼠标按键的按下，弹起，单击和双击动作，参数 `bLeftRight` 用于指示模拟左键还是右键动作（设计时没有考虑滚轮^\_^）。

后三个成员函数，分别模拟键盘键的按下，弹起和击键动作（stalker 对击键动作的定义是：键被按下并弹起），参数 `vKey` 用于指定具体的键。

设计完成之后，就剩下具体的实现，众所周知，模拟鼠标和键盘动作可以用 Windows API `mouse_event`，`keybd_event` 和 `SendInput`。由于前面两个函数比较古老，stalker 选择了二者功能兼备的 `SendInput`。

具体代码如下，由于本人初学 C++，编码可能有很多不好的地方，还希望大家能多指教！

代码:

```cpp
//SimulatingActions.cpp

#define _WIN32_WINNT 0x0500

#include<windows.h>

#include"SimulatingActions.h"

void SimulatingActions::MouseDown(bool bLeftRight)
{
  INPUT myinput;
  MOUSEINPUT mymi;
  POINT mypoint;
  GetCursorPos(&mypoint);
  mymi.dx = mypoint.x;
  mymi.dy = mypoint.y;
  mymi.mouseData = 0;
  mymi.dwFlags = MOUSEEVENTF_LEFTDOWN;
  mymi.time = 0;
  mymi.dwExtraInfo = GetMessageExtraInfo();
  if(bLeftRight)
      mymi.dwFlags = MOUSEEVENTF_RIGHTDOWN;
  myinput.type = INPUT_MOUSE;
  myinput.mi = mymi;
  SendInput(1, &myinput, sizeof(INPUT));
}

void SimulatingActions::MouseUp(bool bLeftRight)
{
  INPUT myinput;
  MOUSEINPUT mymi;
  POINT mypoint;
  GetCursorPos(&mypoint);
  mymi.dx = mypoint.x;
  mymi.dy = mypoint.y;
  mymi.mouseData = 0;
  mymi.dwFlags = MOUSEEVENTF_LEFTUP;
  mymi.time = 0;
  mymi.dwExtraInfo = GetMessageExtraInfo();
  if(bLeftRight)
      mymi.dwFlags = MOUSEEVENTF_RIGHTUP;
  myinput.type = INPUT_MOUSE;
  myinput.mi = mymi;
  SendInput(1, &myinput, sizeof(INPUT));
}

void SimulatingActions::MouseClick(bool bLeftRight)
{
  MouseDown(bLeftRight);
  MouseUp(bLeftRight);
}

void SimulatingActions::MouseDbClick(bool bLeftRight)
{
  MouseClick(bLeftRight);
  MouseClick(bLeftRight);
}

void SimulatingActions::KeyDown(char vKey)
{
  INPUT myinput;
  KEYBDINPUT myki;
  myki.wVk = vKey;
  myki.wScan = MapVirtualKey(vKey, 0);
  myki.dwFlags = KEYEVENTF_EXTENDEDKEY;
  myki.time = 0;
  myki.dwExtraInfo = GetMessageExtraInfo();
  myinput.type = INPUT_KEYBOARD;
  myinput.ki = myki;
  SendInput(1, &myinput, sizeof(INPUT));
}

void SimulatingActions::KeyUp(char vKey)
{
  INPUT myinput;
  KEYBDINPUT myki;
  myki.wVk = vKey;
  myki.wScan = MapVirtualKey(vKey, 0);
  myki.dwFlags = KEYEVENTF_KEYUP;
  myki.time = 0;
  myki.dwExtraInfo = GetMessageExtraInfo();
  myinput.type = INPUT_KEYBOARD;
  myinput.ki = myki;
  SendInput(1, &myinput, sizeof(INPUT));
}

void SimulatingActions::KeyHit(char vKey)
{
  KeyDown(vKey);
  KeyUp(vKey);
}
```

在 VS 对战平台中，进入一个房间的操作非常简单，就是双击想要进入的房间，而挤房的操作就是不停的双击，如果弹出人满对话框，关闭之，继续双击，如此循环反复，直到进入该房间为止。

关闭弹出来的人满对话框，有三种方法：

1. 按 `ESC` 键
2. 按 `ALT+F4` 组合键
3. 移动鼠标点对话框右上角的 `X`

其中方法 3 最难，我选择了最简单的方法 1

## 最后的主程序如下

代码:

```cpp
#include<windows.h>
#include"SimulatingActions.h"
DWORD WINAPI ThreadProc(LPVOID lpParameter);

int WINAPI WinMain(HINSTANCE a, HINSTANCE b, LPSTR c, int d)
{
    bool bSwitch = true;
    SHORT s1, s2, s3;
    HANDLE hThread, hMutex;
    //判断程序是否已经运行，如果已经运行则退出
    CreateMutex(NULL, false, "specialmutexofstalker");
    if(ERROR_ALREADY_EXISTS == GetLastError())
    {
        MessageBox(NULL, "Program already run!", "Information", MB_ICONERROR);
        ExitProcess(0);
    }
    while(true)
    {
        s1 = GetKeyState(VK_CONTROL);
        s1 >>= (sizeof(SHORT)*8 - 1);
        s2 = GetKeyState(VK_SHIFT);
        s2 >>= (sizeof(SHORT)*8 - 1);
        s3 = GetKeyState('Q');
        s3 >>= (sizeof(SHORT)*8 - 1);
        //Ctrl+Shift+Q 被按下，则退出程序
        if(s1 && s2 && s3)
            ExitProcess(0);

        s1 = GetKeyState(VK_CONTROL);
        s2 = GetKeyState('P');
        s1 >>= (sizeof(SHORT)*8 - 1);
        s2 >>= (sizeof(SHORT)*8 - 1);
        //Ctrl+P 被按下，停止动作，并设置 bSwitch 为 true，
        //否则程序将不能再创建模拟动作线程
        if(s1 && s2 && (!bSwitch))
        {
            TerminateThread(hThread, 0);
            bSwitch = true;
        }

        s1 = GetKeyState(VK_CONTROL);
        s2 = GetKeyState('S');
        s1 >>= (sizeof(SHORT)*8 - 1);
        s2 >>= (sizeof(SHORT)*8 - 1);
        //Ctrl+S 被按下，开始动作，并设置 bSwitch 为 false，
        //防止程序创建多个线程
        if(s1 && s2 && bSwitch)
        {
            hThread = CreateThread(NULL, 0, &ThreadProc, NULL, 0, NULL);
            if(NULL != hThread)
                bSwitch = false;
        }
        Sleep(100);
    }
    return 0;
}

DWORD WINAPI ThreadProc(LPVOID lpParameter)
{
    SimulatingActions mySA;
    while(true)
    {
        mySA.MouseDbClick(0);
        Sleep(500);
        mySA.KeyHit(VK_ESCAPE);
    }
}
```

在程序中我没有使用热键，而是使用 `GetKeyState` 函数判断键的状态来控制程序的行为，这样做有一个缺点，就是不太灵敏，但也有优点，就是不用担心热键已经被其他程序注册![](img/234a76e67df951b6551d305c177a8b78.jpg "扮鬼脸")

附件中有整个工程以及编译好的可执行文件

本程序使用 `Dev-C++` 开发

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || source.rar |

## 后台挤房实现

想要实现后台挤房，就不能再用 `SendInput` 来模拟鼠标动作。我们运行 VS 竞技游戏平台，spy lite 表窗口的信息，如下图所示

![](img/06825191b54960cda0dd2e44b20cda32.jpg)

是一个 `ListView` 窗口，直接对该窗口发送 `WM_LBUTTONDBLCLK` 消息即可取代以前的方法实现后台挤房,大概过程如下

![](img/7c9d6af48a946822401ffea0ae99189b.jpg)

使用 `Spy Lite` 和 MS 的 `Spy++` 可以得到如下的窗口层次结构:

VS 竞技游戏平台主窗口-->窗口 1(`CtlID0x49A`)-->窗口 2(`CtlID0x483`)-->窗口 3(`CtlID0x47B`)-->目标窗口(`CtlID0x1F6`)

VS 主窗体句柄可以通过枚举窗体并判断标题是否包含"VS 竞技游戏平台"字符串获得

如何判断已经成功进入房间这个问题比较棘手，我们首先来对比一下进入房间前后 vs 界面上的差别

进入之前

![](img/0b372b7c8b7d279655a4becb581af13e.jpg)

进入之后

![](img/340d65190e3d2a957d085325c1a6b40e.jpg)

在上图红色圈住的地方有很多 `window`，开始我想可以通过判断其中某个 `window` 的 `Text` 是否变化来确定是否成功进入房间,结果用 `spylite` 一看,那些 `window` 的 `Text` 无论进入前后都是空串`""`

最后发现通过判断聊天窗口(同样使用前面提到的工具来获得它与 vs 主窗体的关系)的 `text` 是否为空可以判断是否已经成功进入房间

发一张程序截图

![](img/11b80b2457f398c6cb87e253d9eeee0c.jpg)

成品

letmeinexe.rar

工程

LetMeIn.rar

stalker 的 blog:

[`hi.baidu.com/89419883`](http://hi.baidu.com/89419883)

希望能够认识更多的朋友![](img/234a76e67df951b6551d305c177a8b78.jpg "扮鬼脸")
