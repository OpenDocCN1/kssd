
### 创建离屏表面

00429F0A  |.  C746 18 01000> mov      dword ptr [esi+18], 1                //这一行其实没用了，不过懒得去掉了

00429F11  |.  8B11             mov      edx, dword ptr [ecx]

00429F13  |.  57             push     edi

00429F14  |.  50             push     eax

00429F15  |.  51             push     ecx

00429F16  |.  FF52 18        call     dword ptr [edx+18]

修改前和修改后的代码 Offset 不一样，这个一会再说。

接下来的错误信息是"Initialize Error:Cannot Build Background Plane"，该创建离屏表面了，这里还是要修改，这里贴一下翻译后的代码：

原来的是这样的：

```
ddscaps.dwCaps = DDSCAPS_BACKBUFFER;
hRet = g_pDDSFront->GetAttachedSurface(&ddscaps, &g_pDDSBack);
```

要改成这样的：

```
ddsd.dwSize = sizeof( ddsd );
ddsd.dwFlags = DDSD_CAPS | DDSD_WIDTH | DDSD_HEIGHT;
ddsd.ddsCaps.dwCaps = DDSCAPS_OFFSCREENPLAIN;
ddsd.dwWidth = 640;
ddsd.dwHeight = 480;
hRet = g_pDD->CreateSurface(&ddsd, &g_pDDSBack, NULL);
```

基本上来说，跟创建主表面的 `CreateSurface` 差不多，只要修改一下结构的参数，并且调用参数传入 `g_pDDSBack` 的指针就好了。

这里解释一下创建主表面那个代码修改后为什么 offset 不一样，因为创建离屏表面这里修改后字节比较多，于是把原本用于设置 ModeX 的代码覆盖掉了，整体代码上移。创建离屏表面的代码如下，基本和创建主表面的代码是一致的，参数有变化：

```
00429F3E  |>  \8B0E           mov      ecx, dword ptr [esi]
00429F40  |.  8D46 04        lea      eax, dword ptr [esi+4]
00429F43  |.  8D7E 74        lea      edi, dword ptr [esi+74]
00429F46  |.  6A 00         push     0
00429F48  |.  C700 6C000000 mov      dword ptr [eax], 6C
00429F4E  |.  C746 08 07000> mov      dword ptr [esi+8], 7
00429F55  |.  C746 6C 40000> mov      dword ptr [esi+6C], 40
00429F5C  |.  C746 10 80020> mov      dword ptr [esi+10], 280
00429F63  |.  C746 0C E0010> mov      dword ptr [esi+C], 1E0
00429F6A  |.  8B11           mov      edx, dword ptr [ecx]
00429F6C  |.  57             push     edi
00429F6D  |.  50             push     eax
00429F6E  |.  51             push     ecx
00429F6F  |.  FF52 18        call     dword ptr [edx+18]
```

### 解决画面异常问题

好啦，现在可以启动看效果啦，发现两个问题：

1. 画面是花的；
2. 画面依然刷的整个屏幕都乱七八糟的；

问题 1 是因为游戏是 16 色的，在设置 ModeX 那里可以看到（可惜现在已经被覆盖了），而桌面是 32 色的，天之痕作者是深入程序彻底修改为 32 色，我比较懒，直接把桌面颜色模式设置为 16 色。

问题 2，是因为窗口模式的 DirectX 需要设置 Clipper，于是在创建完离屏表面之后，设置一下 Clipper，C 代码如下：

```
LPDIRECTDRAWCLIPPER pMyClipper;
hRet = g_pDD->CreateClipper(0, &pMyClipper, NULL);
hRet = pMyClipper->SetHWnd(0, g_hMainWnd);
hRet = g_pDDSFront->SetClipper(pMyClipper);
```

`g_hMainWnd` 的值可以参考 `SetCooperativeLevel` 的参数。

好在刚才覆盖了设置 ModeX 的代码，空间还是比较大的，创建完离屏表面的代码后面添加以下代码：

```
00429F97  |>  \6A 00             push     0
00429F99  |.  8D5424 04         lea      edx, dword ptr [esp+4]
00429F9D  |.  52                 push     edx
00429F9E  |.  8B0E               mov      ecx, dword ptr [esi]
00429FA0  |.  8B11               mov      edx, dword ptr [ecx]
00429FA2  |.  6A 00             push     0
00429FA4  |.  51                 push     ecx
00429FA5  |.  FF52 10            call     dword ptr [edx+10]
00429FA8  |.  8B96 10110000      mov      edx, dword ptr [esi+1110]
00429FAE  |.  8B0424             mov      eax, dword ptr [esp]
00429FB1  |.  8B08               mov      ecx, dword ptr [eax]
00429FB3  |.  52                 push     edx
00429FB4  |.  6A 00             push     0
00429FB6  |.  50                 push     eax
00429FB7  |.  FF51 20            call     dword ptr [ecx+20]
00429FBA  |.  8D7E 70            lea      edi, dword ptr [esi+70]
00429FBD  |.  8B3F               mov      edi, dword ptr [edi]
00429FBF  |.  8B1424             mov      edx, dword ptr [esp]
00429FC2  |.  8B0F               mov      ecx, dword ptr [edi]
00429FC4  |.  52                 push     edx
00429FC5  |.  57                 push     edi
00429FC6  |.  FF51 70            call     dword ptr [ecx+70]
```

### 修复内存越界问题

好了，现在两个问题都解决了，运行一下看看，动画正常，点击一下，游戏死掉了。

`OD` 调试一下发现错误是在这里：

```
0042DC27  |.  F3:AB             rep      stos dword ptr es:[edi]
```

分析一下，大概是申请了一块内存，然后操作越界了。仔细观察前后代码，发现这块空间是 `g_pDDSBack->Lock` 之后得到的，并且出错代码的操作是要对这块空间进行 `1E0*400*4=1E0000` 字节的操作，但是这个空间只有 `1E0*280*2=96000` 大小，那么如何满足他呢？

猜测，他的需求只和 `1E0`（离屏表面的高）有关，那么就把离屏表面的宽设置为 `1E0000/2/1E0=800`，于是在创建离屏表面那里，把：

```
00429F5C  |.  C746 10 80020> mov      dword ptr [esi+10], 280
```

改成：

```
00429F5C  |.  C746 10 00080> mov      dword ptr [esi+10], 800
```

再次运行，OK 了。

==========================

### 更新：动态获取离屏表面宽度

这个数值不是固定 800，而是需要执行如下代码来获取：

```
ddsd.dwSize = sizeof( ddsd );
ddsd.dwFlags = DDSD_PITCH;
g_pDDSFront->GetSurfaceDesc(&ddsd);
```

执行的时机是创建完成主表面之后，然后 `ddsd.lPitch/2` 就是离屏表面需要设置的宽度了。

具体代码就不贴了。

==========================

### 修复画面拉伸问题

画面清晰了就可以看到，画面被拉长了。似乎是分辨率的问题，这里就创建窗口和离屏表面涉及到分辨率，换一下分辨率试试看，发现和创建窗口那里没关系，于是在创建离屏表面那里试试看，有门，答案就是高度要和屏幕分辨率高度一样，于是改一下创建离屏表面的代码，如下：

```
00429F3E  |>  \A1 14F14400    mov      eax, dword ptr [<&USER32.GetSyst>
00429F43  |.  6A 01          push     1                        ; /Index = SM_CYSCREEN
00429F45  |.  FFD0          call     eax                       ; \GetSystemMetrics
00429F47  |.  8946 0C       mov      dword ptr [esi+C], eax
00429F4A  |.  8B0E          mov      ecx, dword ptr [esi]
00429F4C  |.  8D46 04       lea      eax, dword ptr [esi+4]
00429F4F  |.  8D7E 74       lea      edi, dword ptr [esi+74]
00429F52  |.  6A 00         push     0
00429F54  |.  C700 6C000000 mov      dword ptr [eax], 6C
00429F5A  |.  C746 08 07000> mov      dword ptr [esi+8], 7
00429F61  |.  C746 6C 40000> mov      dword ptr [esi+6C], 40
00429F68  |.  C746 10 00080> mov      dword ptr [esi+10], 800
00429F6F  |.  8B11          mov      edx, dword ptr [ecx]
00429F71  |.  57             push     edi
00429F72  |.  50             push     eax
00429F73  |.  51             push     ecx
00429F74  |.  FF52 18        call     dword ptr [edx+18]
```

好啦，现在一切正常啦，玩游戏啦～

### 致谢与说明

本人不会 DirectX，思路及相关知识是从《天之痕窗口化》及 Google 获取的，在此表示感谢。

另外本文是在修改完成之后对比原来的程序贴的代码，所以可能有疏漏并且没有截图，请各位见谅和指正！

附上修改过的可窗口化的游戏主程序：

![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) `lob_Win.rar` [请到论坛下载]

### 主表面参数修正

引用: *最初由 **yeyeshun**发布*

```
00429F52  |.  C700 6C000000 mov      dword ptr [eax], 6C                  //这里填充一个 DDSURFACEDESC 结构，6c 是 size
00429F58  |.  C746 08 21000> mov      dword ptr [esi+8], 21                //ddsd.dwFlags = DDSD_CAPS | DDSD_BACKBUFFERCOUNT
00429F5F  |.  C746 6C 18020> mov      dword ptr [esi+6C], 218             //ddsd.ddsCaps.dwCaps = DDSCAPS_PRIMARYSURFACE |
                                   //DDSCAPS_FLIP | DDSCAPS_COMPLEX
00429F66  |.  C746 18 01000> mov      dword ptr [esi+18], 1                //ddsd.dwBackBufferCount = 1;
00429F6D  |.  8B11          mov      edx, dword ptr [ecx]
00429F6F  |.  57             push     edi
00429F70  |.  50             push     eax
00429F71  |.  51             push     ecx
00429F72  |.  FF52 18        call     dword ptr [edx+18]
```

这里需要改一下，因为窗口模式使用这些参数 `CreateSurface` 会失败，具体原因我就不知道了，我不会 DirectX=。=

失败是因为创建 Surface 的时候指定了 `DDSCAPS_FLIP` 参数，表示使用页面翻转的形式来向主表面传送数据，这个参数只能在全屏模式下使用。
