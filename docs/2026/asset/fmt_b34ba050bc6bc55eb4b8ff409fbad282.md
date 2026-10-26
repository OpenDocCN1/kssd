
00556ABD  |.  FF75 08       push    dword ptr [ebp+8] //message

00556AC0  |.  FF90 A0000000 call    [eax+A0]-------->默认处理虚函数：`CWnd::DefWindowProc(message, wParam, lParam)`;

00556AC6  |.  8945 FC       mov     [ebp-4], eax

00556AC9  |>  8B45 FC       mov     eax, [ebp-4]

00556ACC  |.  5E            pop     esi

00556ACD  |.  C9            leave

00556ACE  \.  C2 0C00       retn    0C

### 第四站：

进入`CWnd::OnWndMsg`消息处理虚函数

`BOOL CWnd::OnWndMsg(UINT message, WPARAM wParam, LPARAM lParam, LRESULT* pResult)`

它主要处理`WM_COMMAND`, `WM_NOTIFY`, `WM_ACTIVATE`, `WM_SETCURSOR`这四个特殊消息句柄,另外在查找匹配处理消息影射表中的事件句柄.

要经过：`OnCommand(wParam, lParam)`、`OnNotify(wParam, lParam, &lResult)`等过程...

```
00556AD1  /$  B8 08635800   mov     eax, 00586308        -------------->SEH新指针
00556AD6  |.  E8 8117EFFF   call    0044825C                          ;构造新SEH框架函数
00556ADB  |.  83EC 54       sub     esp, 54 ---------->分配54字节栈
00556ADE  |.  8365 F0 00    and     dword ptr [ebp-10], 0  ----->初始化指针变量*pResult的为0
00556AE2  |.  53            push    ebx
00556AE3  |.  8B5D 08       mov     ebx, [ebp+8]    ----------- > [ebp+8] 为消息Msg结构体的Message域里边存放的是消息ID号
00556AE6  |.  56            push    esi
00556AE7  |.  57            push    edi
00556AE8  |.  81FB 11010000 cmp     ebx, 111  ---->注意这里:111为WM_COMMAND消息ID号,开始进行消息判断
00556AEE  |.  8BF9          mov     edi, ecx  ------>ecx是所谓的this指针...
00556AF0  |.  75 18         jnz     short 00556B0A--->如果不是WM_COMMAND就跳过去.
00556AF2  |.  FF75 10       push    dword ptr [ebp+10]                ;  Case 111 (WM_COMMAND) of switch 00556AE8
00556AF5  |.  8B07          mov     eax, [edi]  ---->  [edi] 指向一个  MFC MainFrame 对象,它把对象指针传给eax,给下边的虚函数用
00556AF7  |.  FF75 0C       push    dword ptr [ebp+C]---------->注意这里：菜单ID号入栈了...这里是：8024
00556AFA      FF50 78       call    [eax+78]--------->这是`CFrameWnd::OnCommand(wparam,lparam)`虚函数
00556AFD  |.  85C0          test    eax, eax -->eax存放函数返回值,把结果放到*pResult指针变量中.
00556AFF  |.  0F84 55010000 je      00556C5A-->当eax为0,转
00556B05  |.  E9 1D040000   jmp     00556F27-->当eax为1,转
00556B0A  |>  83FB 4E       cmp     ebx, 4E----------------------------------------->这里4E为 WM_NOTIFY消息ID号,
00556B0D  |.  75 28         jnz     short 00556B37
00556B0F  |.  8B45 10       mov     eax, [ebp+10]
00556B12  |.  8338 00       cmp     dword ptr [eax], 0
00556B15  |.  0F84 3F010000 je      00556C5A--->--->如果不是WM_NOTIFY就跳到WM_ACTIVATE消息处理
00556B1B  |.  8B17          mov     edx, [edi]
00556B1D  |.  8D4D F0       lea     ecx, [ebp-10]---->&lResult指针地址保存到ecx中压栈传参用.
00556B20  |.  51            push    ecx
00556B21  |.  50            push    eax
00556B22  |.  FF75 0C       push    dword ptr [ebp+C]
00556B25  |.  8BCF          mov     ecx, edi
00556B27  |.  FF52 7C       call    [edx+7C]---->这是`CWnd::OnNotify(wParam,lparam,&lResult)`虚函数
00556B2A  |> /85C0          test    eax, eax
00556B2C  |. |0F85 39040000 jnz     00556F6B
00556B32  |. |E9 23010000   jmp     00556C5A
```

### 第五站：

因为我这里要处理的是`WM_COMMAND`的消息嘛，所以进入：

`BOOL CFrameWnd::OnCommand(WPARAM wParam, LPARAM lParam)`

对了还有一个参数就是菜单的ID号：80A2,存放在`wParam`低字节中.

恩,是这样得当我用鼠标触发那个广告菜单命令的时候,广告就出现在菜单工具栏上,所以菜单ID不能少！

还有个关键问题：

`FoxitReaderPro`对象好象重载了`OnCommand(WPARAM wParam, LPARAM lParam)`方法,那以后的消息路线转变了：

应该是调用`CFoxitReaderFrameWnd::OnCommand`然后在调用`FrameWnd::OnCommand`和`CWnd::OnCommand`方法.在普通的MFC类库中`FrameWnd::OnCommand`里边没有很多实例方法,它是直接调用`CWnd::OnCommand`

```
0056DD6D   .  53            push    ebx-->：`CWnd::OnWndMsg`函数中的message参数变量,这里是WM_COMMAND消息ID
0056DD6E   .  56            push    esi-->保存对象指针00C97BA8
0056DD6F   .  57            push    edi-->保存对象指针00C97BA8，
0056DD70   .  8BF9          mov     edi, ecx ---->ecx与edi指向同一对象指针
0056DD72   .  6A 00         push    0
0056DD74   .  E8 4B040000   call    0056E1C4 ---> 调用`CWnd::GetTopLevelFrame()`,这个函数发了WM_MDIGETACTIVE消息得到当前MDI Child Window
0056DD79   .  8B5C24 10     mov     ebx, [esp+10]  ---> 菜单ID号出现 80A2
0056DD7D   .  BE 11010000   mov     esi, 111
0056DD82   .  85C0          test    eax, eax ->函数的返回值,由于广告条在父窗口中并没有在子类或者其他CView类出现,所以我把所以子窗口全部shutdown了,免的又在阴沟over了！
0056DD84   .  74 13         je      short 0056DD99------>在这里跳！
0056DD86   .  FF7424 14     push    dword ptr [esp+14]
0056DD8A   .  53            push    ebx
0056DD8B   .  56            push    esi
0056DD8C   .  FF70 1C       push    dword ptr [eax+1C]
0056DD8F   .  50            push    eax
0056DD90   .  E8 D87BFEFF   call    0055596D--->注意这个虚函数中有很多对象实例方法里边判断广告的显示方法也在其中了,通过分析它才知道`FoxitReaderPro`对象应该重载了`OnCommand(WPARAM wParam, LPARAM lParam)`方法。
0056DD95   .  85C0          test    eax, eax
0056DD97   .  75 30         jnz     short 0056DDC9
0056DD99   >  FF7424 14     push    dword ptr [esp+14]---//这个变量为0
0056DD9D   .  8BCF          mov     ecx, edi--->赋值`CFrameWnd`对象指针.
0056DD9F   .  53            push    ebx        ----->触发菜单ID号：8042
0056DDA0   .  E8 4BE1FFFF   call    0056BEF0    ---------------------------来到这里是通往广告条的显示之门.
0056DDA5   .  85C0          test    eax, eax
0056DDA7   .
```

进入`FoxitReaderPro`判断广告条的实例方法： `call    0056BEF0 (wParam,lParam)`

```
0056BEF0  /$  53            push    ebx
0056BEF1  |.  56            push    esi
0056BEF2  |.  57            push    edi
0056BEF3  |.  8BF1          mov     esi, ecx--->对象指针.
0056BEF5  |.  0FB77C24 10   movzx   edi, word ptr [esp+10] ----》 触发广告条菜单ID号： 8024
0056BEFA  |.  E8 D7B3FEFF   call    005572D6                                  ;  c++对象指针与窗口句柄影射
0056BEFF  |.  33C9          xor     ecx, ecx
0056BF01  |.  3948 50       cmp     [eax+50], ecx
0056BF04  |.  74 4C         je      short 0056BF52--->走到了,开始跳了
0056BF06  |.  394C24 14     cmp     [esp+14], ecx
0056BF0A  |.  75 46         jnz     short 0056BF52
0056BF0C  |.  81FF 46E10000 cmp     edi, 0E146   --->其他彩单ID 0E146
0056BF12  |.  74 3E         je      short 0056BF52
0056BF14  |.  BB 47E10000   mov     ebx, 0E147  --->其 菜单ID E147
0056BF19  |.  3BFB          cmp     edi, ebx
0056BF1B  |.  74 35         je      short 0056BF52
0056BF1D  |.  81FF 45E10000 cmp     edi, 0E145
0056BF23  |.  74 2D         je      short 0056BF52
...中间省略了不必要的代码看关键部分
```

从上边跳过来的.

```
0056BF52  |> \FF7424 14     push    dword ptr [esp+14]  |Arg1---> 这个参数是我触发显示广告菜单的ID号：8042
0056BF56  |.  8BCE          mov     ecx, esi
0056BF58  |.  FF7424 14     push    dword ptr [esp+14]  |Arg2--->大家注意拉！看堆栈！
0056BF5C  |.  E8 0FB1FEFF   call    00557070 ============>这是关键广告函数,真是没有想到`FoxitReaderPro`给这个ID号另起炉灶了！
0056BF61  |>  5F            pop     edi
0056BF62  |.  5E            pop     esi
0056BF63  |.  5B            pop     ebx
0056BF64  \.  C2 0800       retn    8
```

看堆栈：

```
0012FB40   000080A2  |Arg1 = 000080A2 =====触发菜单ID号：8042，这个参数是wParam
0012FB44   00000000  |Arg2 = 00000000
```

```
00557070  /$  55            push    ebp
00557071  |.  8BEC          mov     ebp, esp
00557073  |.  83EC 2C       sub     esp, 2C
00557076  |.  8B45 08       mov     eax, [ebp+8] ---》wParam参数
00557079  |.  53            push    ebx
0055707A  |.  56            push    esi
0055707B  |.  57            push    edi
0055707C  |.  0FB7F8        movzx   edi, ax
0055707F  |.  33DB          xor     ebx, ebx
00557081  |.  8BF1          mov     esi, ecx
00557083  |.  C1E8 10       shr     eax, 10  ----把wParam右移动16位,这时候wParam变为0了.
00557086  |.  395D 0C       cmp     [ebp+C], ebx---->[ebp+c]
00557089  |.  8945 08       mov     [ebp+8], eax
0055708C  |.  75 3A         jnz     short 005570C8---》到这里相当跳.
0055708E  |.  3BFB          cmp     edi, ebx
00557090  |.  74 66         je      short 005570F8
00557092  |.  8D4D D4       lea     ecx, [ebp-2C]
00557095  |.  E8 A9FFFFFF   call    00557043
0055709A  |.  8B06          mov     eax, [esi]
0055709C  |.  8D4D D4       lea     ecx, [ebp-2C]                             ;  lpararm c++对象指针
0055709F  |.  53            push    ebx
005570A0  |.  51            push    ecx
005570A1  |.  6A FF         push    -1
005570A3  |.  57            push    edi                                       ;  彩单ID
005570A4  |.  8BCE          mov     ecx, esi
```
