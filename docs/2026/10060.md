
# [旧帖][原创]3种简单的键盘记录简单介绍0.00雪花

菜鸟学习中。。。![](https://bbs.kanxue.com/view/img/face/16.gif)

争取早日专正，呵呵。

贴上刚开学时写的一份报告，一次小作业。没啥水平。和我一样菜得一塌糊涂的，可以看看。。。

本人信息安全科班生![](https://bbs.kanxue.com/view/img/face/16.gif)，水平之低，令人发指。对诸位自学成才的十分佩服。以后还请多多指教。先行谢过

## Ring3层键盘记录的实现

本文主要论述、对比了用户层主要的三种实现键盘记录的方法：利用钩子函数、轮询键盘消息和直接从输入设备获取数据。

在木马程序中，键盘记录是不可缺少的一部分，因为它是窃取别人电脑上数据的关键部分，记录这些程序的账号、密码主要就是靠键盘记录实现的。然后再将获得的账号、密码发到某个指定的邮箱里或某个FTP或网站服务器上。 在用户层实现键盘记录远比在系统层实现简单，是比较常用的方法。

### 1、利用钩子函数

最简单的方法是使用`SetWindowsHook`函数。钩子(Hook)，是Windows消息处理机制中的监视点，应用程序可以在这里安装一个监视函数以监视指定进程（本进程或其它进程都可以）发生的事件。当监视的事件消息到达后，钩子函数可以在目标窗口处理函数之前处理它。

#### 1.1 Hook的分类

总的来说, Hook可以分为Local Hook (本地钩子) ,和Remote Hook (远程钩子) 。Local Hook对本进程中发生的事件进行监视,对系统的影响比较小;而Remote Hook则可以对其他进程中发生的事件进行监视,针对整个系统的事件进行拦截。RemoteHook又可以分为Thread Hook (线程钩子)和GlobalHook (全局钩子) 。线程钩子能够监视系统内其他进程中指定线程的事件(此时该Hook函数可以放置在DLL中,也可以放置在应用程序的模块段) ;而全局钩子能够监视系统内所有进程空间地址中所有线程的事件(此时该Hook 函数必须放置在DLL中) 。

#### 1.2 Hook的实现方法

##### 1.2.1安装和卸载Hook的方法

利用Ap i函数`HHOOK SetWindowsHookEx ( int idHook, HOOKPROC lpfn, H INSTANCE hMod,DWORD dwThread Id)` ,进行钩子的安装。

其中:第一个参数为指定钩子的类型（共15种）;第二个参数为标识Hook函数的入口地址;第三个参数为钩子函数所在模块的句柄,如果是Local Hook,此值为0;第四个参数为希望挂钩线程的线程ID,为0时则拦截整个系统的消息。

`SetWindowsHookEx`总是将我们的Hook函数放置在挂接函数链的顶端,使得应用程序接收到相应消息时,我们的Hook函数能第一个被调用。Hook函数必须按照回调函数的格式声明:`LRESULT W INAP I HookProc ( int nCode,WPARAM wParam,LPARAM lParam)`。其中`nCode`指定Hook类型, `wParam`, `iParam`的取值随`nCode` 不同而不同,它代表了某种类型的Hook的某个特定动作。如果Hook函数需要将消息传递给下一个过滤函数, 则在该Hook 函数返回前还需要调用一次`CallNextHookEx()`函数。

利用Ap i 函数`BOOL UnHookWindowsHookEx(HHOOK hHook)`卸载钩子。

##### 1.2.2 利用键盘钩子监控系统键盘输入实现过程

1)使用MFC AppW izard (DLL)建立扩展动态链接库`KeyboardHook. dll`

2)在`KeyboardHook. h`中添加导出函数`InstallHook`。

原型为`__declspec ( dllexport) void W INAP I  InstallHook ( )` ,以后在应用程序中调用此函数就能安装一个全局键盘钩子。

3)在`Keyboard. cpp`中添加全局变量`Hook`和全局函数`HookProc ( )` 、`SaveKeyboardLog ( )` 分别实现处理与保存。

4)最后调用`UninstallHook ( )`卸载钩子

5）编写应用程序调用`KeyboardHook.dll`。先调用动态链接库的`InstallHook`函数安装好钩子,此时即可对系统下的键盘消息实施拦截处理。使用完毕后,通过动态链接库中的`UninstallHook`函数卸载键盘钩子。

这种键盘记录十分简单。隐蔽性较差。功能也不强，不能记录虚拟键盘的输入。

### 2.GetAsyncKeyState

利用`GetAsyncKeyState`实现键盘记录也十分简单，这个函数根据虚拟键表判断按键类型。返回值为一个16位的二进制数，如果被按下则最高位为1，即返回-32767，但是如果需要对键盘进行全局性的记录，则需要与另 一个API函数`GetKeyState`配合使用才能实现。原因在于`GetAsyncKeyState`函数只在按键的瞬间执行一次，如果按下的键是开关键 （如：caps lock），那么过了那一瞬间`GetAsyncKeyState`函数则不起任何作用。而这个时候就需要用`GetKeyState`来判断该开关键是否按下。因为每个键的虚拟码是唯一，在这样一种情况下，不管在大写情况还是小写情况下，按下的键记录下来的信息都是一样的。这个时候就需要一次判断，需要判断有两个键是否按下，一个是caps lock是否按下，一个是shift是否按下，因为shift不属于开关键，那它自然就不需要`GetKeyState`函数，但是caps lock是一个开关键它在这种情况下就需要用`GetKeyState`函数。所以在这种特例下我们就需要用`GetKeyState`函数了。另外insert 这个键也是一种特例它也需要用`GetKeyState`函数来判断。

`GetKeyState(int nKey)`用法

经过验证，检查非锁定键（除去num lock,caps lock,scroll lock外）按下状态返回-127或-128，而且键值一定是键盘上刻的值（大写字母或数字，%￥#之类的和小写字母则不能识别），非按下状态则为0或 1。检查num lock,caps lock,scroll lock三个锁定键，锁定状态（键盘指示灯亮）返回1，否则返回0。

例如，不考虑其他因素，仅判断“A”键是否按下，大写键是否锁定可以这样实现：

``` 
if (GetAsyncKeyState('A') == -32767)

{

         if (GetKeyState(VK\_CAPITAL)==1)  {log << "A";}

         else   {log << "a";}

}
```

在轮询过程中，往往需要使用死循环`while（1）`，此时为防止CPU利用过高，可以加入`windows.h`下的`sleep`函数来使程序暂停若干毫秒。

与HOOK函数相比使用这两个函数进行键盘记录虽然在程序上显的比较冗长，但是它却更加的简单，而且也不容易出现错误。而HOOK函数虽然使用起来，在代码上看起来可能比较简单，但是它却很容易暴露在杀毒软件的查杀之下。

### 3. GetRawInput

微软原来的鼠标键盘输入模型：

鼠标和键盘产生输入数据，系统中断去处理这些与设备信息相关的数据，让这些数据变得与设备无关。一个应用程序通过发送到他窗口的消息获取与设备无关的消息，例如`WM\_CHAR`, `WM\_MOUSEMOVE`

原始输入模型:

直接从设备获取数据并且可以根据他们的需要来获取。前提是一个应用程序想获取原始数据就必须注册他想要获取原始输入的那些设备，然后应用程序会收到`WM\_INPUT`消息。

主要流程：

1)向系统注册一个或者多个原始输入设备

为了注册这个设备，一个应用程序首先必须创建一个指明他所希望接受设备类别的（top level collection―TLC）`RAWINPUTDEVICE`结构。TLC被定义成为`UsagePage`（设备类）和`Usage`（设备类内的具体设备）。例如为了从键盘获取原始输入，设置`UsagePage = 1 and Usage = 6`，应用程序调用`RegisterRawInputDevice`去注册这个设备。

``` 
BOOL RegisitKeyBord(HWND hwnd)

{

   if(NULL == hwnd)

      return false;

   PRegisterRawInputDevices RegisterRawInputDevices = (PRegisterRawInputDevices)GetApiAdd("User32.dll", "RegisterRawInputDevices");

   if(NULL == RegisterRawInputDevices)

      return false;

   RAWINPUTDEVICE rid;

   rid.usUsagePage = 0x01;

   rid.usUsage = 0x06;

   rid.dwFlags = RIDEV\_INPUTSINK;

   rid.hwndTarget = hwnd;

   return RegisterRawInputDevices(&rid, 1, sizeof(RAWINPUTDEVICE));

}
```

应用程序可以注册系统当前没有的设备。当设备可用之后，Windows管理器会自动将原始输入数据发送到应用程序。应用程序可以调用`GetRawInputDeviceList`来获取系统中原始输入设备的列表。用`GetRawInputDeviceList`获取的`hDevice`，应用程序调用`GetRawInputDeviceInfo`获取设备信息。

2)在你注册的原始输入设备数据发生变化时，系统发送一个消息及新数据到你的进程

3)调用`GetRawInputData`来获取这些数据

部分代码：

``` 
case WM\_INPUT:

     if(NULL == GetRawInputData)

     {

        DefWindowProc(hWnd, message, wParam, lParam);

        return 0;

     }

     GetRawInputData((HRAWINPUT)lParam, RID\_INPUT, NULL, &dwSize, sizeof(RAWINPUTHEADER));

     lpb = new BYTE[dwSize];

     if(lpb == NULL)

     {

        DefWindowProc(hWnd, message, wParam, lParam);

        return 0;

     }

     if(GetRawInputData((HRAWINPUT)lParam, RID\_INPUT, lpb, &dwSize, sizeof(RAWINPUTHEADER)) != dwSize)

        MessageBox(NULL, "GetRawInputData doesn't return correct size !", "Raw Input Test", 0);

     raw = (RAWINPUT\*)lpb;

     if (raw->header.dwType == RIM\_TYPEKEYBOARD)

     {

      wsprintf(vk,"[%s]\r\n%s",&ti,GetKeyName(raw->data.keyboard.VKey))

     }

     delete[] lpb;
```

这类键盘监控程序比前面两种获取信息的能力要强，可以获取软键盘的输入。

### 4.汉字输入记录

1).先获取当前正在输入的窗口的输入法句柄

``` 
hIMC = ImmGetContext(hWnd);
```

2).将`ImmGetCompositionString`的获取长度设为0来获取字符串

``` 
dwSize=ImmGetCompositionString(hIMC,GCS\_RESULTSTR, NULL, 0);
dwSize += sizeof(WCHAR); // 缓冲区大小要加上字符串的NULL结束符大小,
memset(lpstr, 0, 20);
```

3). 再调用一次.`ImmGetCompositionString`获取字符串

``` 
ImmGetCompositionString(hIMC, GCS\_RESULTSTR, lpstr, dwSize);
```

现在`lpstr`里面即是输入的汉字了。

### 5.记录当前窗口

可以通过`GetForegroundWindow`函数来记录当前窗口。

``` 
if ( prev == NULL)

             {

                 prev = GetForegroundWindow();

                 GetWindowText(prev,ti,256);

                 Writetitle();//写窗口名

                 Writefile();//写按键

             }

     else if ( prev == GetForegroundWindow() )

             { writefile()； }

     else   {

                prev = GetForegroundWindow();

                GetWindowText(prev,ti,256);

                 Writetitle();

                 Writefile();

             }
```

### 6.防范

由上面的介绍可以看出，除了对记录程序进行检测处理之外，软键盘可以在一定程度上提高安全性，但十分有限，比如对`GetRawInput` 类型的键盘记录就无能为力。

实际上，更为简单有效的方法是人为打乱输入顺序，例如，输入密码1234567，可以先输入12567，再用鼠标切换焦点位置输入34。此时键盘监控程序只会记录错误的结果1256734

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [3种记录的参考代码.rar](javascript:void(0);)

  （104.87kb，299次下载）
