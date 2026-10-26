
- 游戏中释放技能是按 `鼠标右键`，切换技能是 `键盘按键A~G`；所以是不是可以更改它的窗口回调函数，在触发这两类消息时，把当前内力值设置为最大内力值（为了省事，选择鼠标事件吧）

- 用VS内的工具Spy++，查找剑侠情缘的窗口标题和类名

![](https://bbs.kanxue.com/upload/attach/202103/832491_PKKBF45QV9G6PW6.jpg)

- VS -> 选择 MFC 动态链接库 -> 在DLL类型中选择 ‘ 静态链接到 MFC 的常规DLL ’

  - 主要逻辑：自己写的回调函数里响应鼠标右键（释放技能）消息

|  |  |

| --- | --- |

```
    // 唯一的 CSwordCheatApp 对象
    CSwordCheatApp theApp;
    // -------------------------------------------------------------------
    HWND g_hWnd;
    // 接收剑侠情缘窗口句柄
    WNDPROC g_OldProc;
    // 旧的窗口回调函数
    PDWORD g_MagicNow = (PDWORD)0x004F1A40;
    // 当前内力值
    PDWORD g_MagicMax = (PDWORD)0x004F1A44;
    // 最大内力值
    // 自己的回调函数
    LRESULT CALLBACK MyWindowProc(
      _In_ HWND hWnd,
      _In_ UINT Msg,
      _In_ WPARAM wParam,
      _In_ LPARAM lParam)
    {
      // 1. 判断消息类型
      if (Msg == WM_RBUTTONDOWN)
      {
        OutputDebugString(L"释放技能");
        // 通过一个全局变量找到内力值相关的地址
        // 0042E007 | > 8 > mov eax, dword ptr ds : [ecx]           eax = 1
        // 0042E009 | . 8 > mov edx, eax                 edx = 1
        // 0042E00B | .C > shl edx, 0x5                 edx = 32
        // 0042E00E | . 0 > add edx, eax                             edx = 33
        // 0042E010 | . 8 > lea edx, dword ptr ds : [eax + edx * 4]  edx = 1+33*4(133)
        // 0042E013 | . 8 > lea edx, dword ptr ds : [eax + edx * 4]  edx = 1+133*4(533)
        // 0042E016 | . 8 > lea eax, dword ptr ds : [eax + edx * 2]  eax = 1+533*2(1067)
        // 0042E019 | .C > shl eax, 0x4                 eax = 1067*16(17072)
        // 0042E01C | . 8 > mov edx, dword ptr ds : [eax + 0x4ED794] ;  edx 是最大内力值 004F1A44
        // 0042E022 | . 8 > mov edi, dword ptr ds : [eax + 0x4ED790] ;  edi 当前内力值 004F1A40
        *g_MagicNow = *g_MagicMax;
      }
      // 执行之前的窗口回调函数
      return CallWindowProc(g_OldProc, hWnd, Msg, wParam, lParam);
    }
    // CSwordCheatApp 初始化         <--- 这里当作主函数
    BOOL CSwordCheatApp::InitInstance()
    {
      CWinApp::InitInstance();

      // 1. 通过查找窗口，获取窗口句柄
      g_hWnd = ::FindWindow(L"Sword Class", L"Sword Window");
      if (NULL == g_hWnd)
      {
        OutputDebugString(L"没有找到剑侠情缘窗口！");
        return FALSE;
      }

      // 2. 设置窗口回调函数（SetWindowLong设置新的窗口回调，返回值是老的窗口回调函数）
      g_OldProc = (WNDPROC)SetWindowLong(g_hWnd, GWL_WNDPROC, (LONG)MyWindowProc);
      if (NULL == g_OldProc)
      {
        OutputDebugString(L"设置窗口回调函数失败！");
        return FALSE;
      }

      return TRUE;
    }
```

- 效果gif

  ![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202103/832491_E338J25C27X7DZ8.jpg)

总结：童年不再有，，，这个15年前的游戏逆向起来还是比较简单的，比较适合我这样的新手分析；dll编写思路大家可以尽情发挥，做技术没有标准答案；兴趣和坚持才能一直走下去，共勉！

ps:在看雪还是第一次发帖，欢迎大家交流指正

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
