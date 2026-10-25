
# 酷狗 2011 去 flash 广告补丁编写

> 原文：[`www.pediy.com/kssd/pediy12/132381.html`](https://www.pediy.com/kssd/pediy12/132381.html)

酷狗音乐播放器，挺好的一个播放器~~

可是美中不足的是它的 flash 广告

2011 版的 `KuGoo` 的 flash 广告就如下图一样在右上边的一条，闪来闪去~~ 不爽

于是就有了想把它去掉的心思~

![](img/29e9fde7aafa9f2169fcf0245a9690ff.jpg)

开始的时候试过好多方法~~ 包括看看它 flash 的联网怎么获取的~~ 还有什么

`CreateWindow` 的时候让它返回失败，不让它创建出来这个窗口，可是都不行或找不到，或程序崩溃~

后来找来工具 `spy++` 查看这个窗口

`Spy++` 果然给力~~ 不过话说查找这个 flash 窗口还得弄好久~~ 不容易聚焦到它上面

不过还是可以成功的，找到的，就如下图

![](img/2deb3a13e74a09788dd815a9bf8cff48.jpg)

看到了 flash 是个窗口类为 `"Internet Explorer_Server"` 的窗口，它的上层窗口 有 `TfmAdBrowser` , `TfmSuite`, `TfrmMain`

其中 `TfrmMain` 是最上层的主窗口类名，从名称中就可以看出，`TfmAdBrowser`---很显然是"广告(Ad)浏览(Browser)"类

于是 `FindWinowEx` 找到 `Internet Explorer_Server` 这个 flash，调用 `ShowWindow` 隐藏之！，隐藏是成功了，

可是会出现个空洞，

更加不美观~~~

再看`"TfmAdBrowser"`是个好东西~~ 专管广告~ 哈，隐藏它试试--- 果然成功了！ 广告就这样，没了！

不过广告没了就意味着~~右上方的那块地方成了没图的空地~~~ 虽然没广告 也不算完美 ~~

要不，我们自己在这里绘制个窗口吧~ 用我们喜欢的图片做背景 ~~ 哈哈

![](img/cd86dcb7f7dc9568ddb2b48acaf39516.jpg)

嗯~~ 那怎么在酷狗界面上创建窗口呢~~ ，一定得有个消息循环，在酷狗的进程空间内管理这个窗口~~~

好吧 来个 dll，自己写个 dll，里面写好创建窗口和消息循环的代码~~ ，然后 patch `kugoo.exe` 让它加载我们的 dll

就是这样了~~

这个 dll 就叫 `KuGoo_Ads.dll` 吧

关键代码如下:

**更新：**以前 dll 的代码用的是 `FindWindowEx`,今天改为 `EnumChildWindows`，更通用些，不依赖窗口排列次序~~ 2011/4/18

代码:

```c
int WINAPI DllMain (HANDLE hInstance, DWORD dwReason, LPVOID  lpReserved)
{
  switch (dwReason)
  {
    case DLL_PROCESS_ATTACH:
      hDllInst = hInstance ;
      // 创建工作线程
      CreateThread(NULL, 0, ThreadFunc, NULL, 0, &dwThreadId) ;   
      return TRUE ;
    case DLL_PROCESS_DETACH:

      return TRUE ;
  }
  return TRUE ;
}


// Thread Function
DWORD WINAPI ThreadFunc(LPVOID lParam)
{

    DWORD  dwTimeBegin, dwTimeCurrent, len ;
    HBITMAP hBitmap ;
    char  szExeName[MAX_PATH] , *p ;
    RECT  rcParentWnd ;

    GetModuleFileName(NULL, szExeName, sizeof(szExeName)) ;
    len = strlen(szExeName) ;

    p = &szExeName[len-1] ;
    while(*p != '\\' && p != szExeName) p-- ;
    // 判断是否是 KuGoo.exe 加载的
    if(lstrcmpi(p+1, szKeyName) != 0)
    {
      FreeLibrary(hDllInst) ;
      return 0 ;
    }
    dwTimeBegin = GetTickCount() ;
    FindAdWindow() ;
    while(!hAdParent || !hAdWnd)
    {
        dwTimeCurrent = GetTickCount() ;
        if(dwTimeCurrent - dwTimeBegin > 60*1000)
          return 0 ;    // 超时就返回
        Sleep(500) ;    // 延时
        FindAdWindow() ;
    }

     
    hBitmap = LoadBitmap(hDllInst, (LPSTR)IDB_BITMAP) ;
    GetClientRect(hAdWnd, &rcParentWnd) ;
      CreateNewWindow(hAdParent, hBitmap, 0, 0, rcParentWnd.right, rcParentWnd.bottom) ;
    return 1 ;
}


// EnumChildProc 可以遍历所有子孙窗口
BOOL CALLBACK EnumChildProc(HWND hWnd,  LPARAM lParam)
{
  char szClassName[MAX_PATH] = {0} ;

  GetClassName(hWnd, szClassName, MAX_PATH) ;
  if(!lstrcmpi(szClassName, "TfmAdBrowser"))
    {
      hAdWnd = hWnd ;
      hAdParent = GetParent(hAdWnd) ;
      ShowWindow(hAdWnd, SW_HIDE) ;
    }
  return TRUE ;
}


// EnumChildWindows
BOOL FindAdWindow()
{
  HWND hWndMain ;
   
  hWndMain = FindWindow("TfrmMain", NULL) ;   
  if(!hWndMain) return 0 ;
  EnumChildWindows(hWndMain, EnumChildProc, 0) ;
  return 1 ;
}


// 创建自定义窗口
BOOL CreateNewWindow(HWND hParent,HBITMAP hBitmap, int x, int y, int cx, int cy)
{
  MSG      msg ;
  HWND     hWnd ;
  WNDCLASS  wcWndClass ;

  memset(&wcWndClass, 0, sizeof(WNDCLASS)) ; 
  wcWndClass.style = 8 ;
  wcWndClass.lpfnWndProc = WndProc ;
  wcWndClass.hInstance = hDllInst ;
  wcWndClass.hbrBackground = (HBRUSH) CreatePatternBrush(hBitmap) ;
  wcWndClass.lpszClassName = szClassName ;
  RegisterClass(&wcWndClass) ;
  hWnd = CreateWindowEx (WS_EX_CONTROLPARENT, szClassName, NULL, WS_CHILD | WS_CLIPSIBLINGS | WS_CLIPCHILDREN, 
                       x, y, cx, cy, hParent, NULL, hDllInst, NULL) ;
  ShowWindow(hWnd, SW_NORMAL) ;
  UpdateWindow(hWnd) ;
   
  while(GetMessage(&msg, NULL, 0, 0))
  {
    TranslateMessage(&msg) ;
    DispatchMessage(&msg) ;
  }
  return 0 ;
}


// 新建窗口的窗口过程
LRESULT CALLBACK WndProc(HWND hWnd,  UINT uMsg, WPARAM wParam, LPARAM lParam)
{

  switch(uMsg)
  {

    case WM_CLOSE:
      DestroyWindow(hWnd);
      PostQuitMessage(0);
      return 0;
    default:
      return DefWindowProc(hWnd, uMsg, wParam, lParam);
  }
}
```

当然这些做完后还要写个外壳就是补丁主程序，来实现修改 `KuGoo.exe` 让它加载我们的这个 dll,

`PEID` 查看的值它恰好有导入函数 `LoadLibraryA` 哈哈~~ 很好 

`OD` 载入，`LoadLibraryA` 下断，断下后就知道了 `IAT` 表中 `LoadLibraryA` 存储在哪里了~~

直接记录~~

我这里得到的是这样的：

```
004773E5   .  68 10754700   push KuGoo.00477510                   ; /FileName = "imm32.dll"
004773EA   .  E8 ED12F9FF   call <jmp.&kernel32.LoadLibraryA>     ; \LoadLibraryA
004773EF   .  A3 CCAC7900   mov dword ptr ds:[79ACCC],eax        ;  IMM32.771F0000
```

此处 `call` 是下面这个：

```
call 004086DC      // LoadLibraryA 的地址的存储单元的地址
```

那么我们在 `KuGoo.exe` 中应该这样做了：

1)修改入口点代码为 `jmp XXXXX` (XXXXX 是我们的代码，这部分代码我把它安排在了 `DOS` 块中)

2)XXXXX 处代码：

```
  mov eax,0x004086DC   // LoadLibraryA 的地址为[004086DC]
  push YYYYY           // YYYYY 是我们的 dll 名字字符串的地址
  call eax         // Load
  --------
  --------           // 执行刚才入口点由于被修改为 jmp 而毁坏的指令
  jmp  ZZZZZ          // 调回继续执行原程序
```

以上这些是我们要 `WriteFile` 到 `KuGoo.exe` 的.

把外壳写好我们的这个补丁程序也就大功告成了 ~~

弄好后就是这样了：(我用的自定义新窗口背景图片也是酷狗的，所以不要以为它没换实质已经换了）![](img/081868d20f20fe7a9b4b88ab0d0d33c6.jpg "狂笑")

![](img/7512934bed8c4f1d863a5f87073cd55f.jpg)

说明：

    附件中含有 Dll 源码，和已经编译好的 dll 还有一个已经做好的补丁，补丁 EXE 可单独使用了 ~~因为它资源里包含了这个 dll

    修改 `KuGoo.exe` 时，部分杀软会报威胁~~

    补丁程序会帮您备份一份未补丁时的文件，在 `KuGoo.exe` 的同目录，有需要可换回

    您也可以自己备份再来补丁

    自创建的新窗口可自定义背景的但是要注意尺寸。我这里用的是 kugoo 的一个截图来的背景。

擦**~~~ 360 残忍的木马我的附件~~ 唉 -- 为清白 外壳 的代码 C 文件我也串上来吧 有时间的自己编译去吧 ~~ 唉。。。**

 要使用附件的 EXE，添加一下信任文件吧。。。。![](img/8665dab461df9ff5e20e52adf9adf6de.jpg "伤心")

\>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

**   更新： **

由于 dll 中获得`"TfmAdBrowser"`类时利用 `FindWindowEx`，过于依赖窗口排列书序，书写不便，不利于通用，于是将核心改良了一下，利用 `EnumChildWindows` 来遍历子窗口，找到首个 `"TfmAdBrowser"`类！ 这样更通用些~~~

附件第三个就是 更新的 dll 文件的 C 工程文件夹~~~

另：支持正式版 `6.2.23` , 其它版本可能不成功， 因为 补丁外壳 Patch 时 用了硬编码 （包括 `KuGoo.exe` 的入口点） 因此可能就失败了~~~ 有时间的话 可以自己 

    依据源代码，稍稍调试一下 改一下~~ 就应该可以成功的 ~![](img/081868d20f20fe7a9b4b88ab0d0d33c6.jpg "狂笑")

**读者注意啦！**

酷狗更新喽~~ `6.2.24` 都出来啦~~ 对最新版本补丁主程序会失效，原因就是 patch 时使用了硬编码，dll 文件是有效地~~~ ，你怎么样让 `kugoo.exe` 加载这个 dll，就看你自己怎么 DIY 啦~~ 

++++++++++++++++++++++++++++++++++++++++++++++++++++

**哈哈~~ **有更新了一下~~~ 解决部分硬编码问题~~~ 适用范围更广喽。。。基本不受版本限制了

2011/5/17

**Sorry,本附件暂不支持 酷狗 7**

Author: Vsbat

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `KuGoo_deAdsShell.rar`  
|

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || 更新的 `KuGoo_Ads_dll.rar`  
|

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || 最新更新_`KuGoo` 去广告主程序.rar  
|
