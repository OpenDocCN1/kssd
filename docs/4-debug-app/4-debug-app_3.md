

# 简单自己修改游戏窗口化！

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-302.htm`](https://www.pediy.com/kssd/pediy09/pediy09-302.htm)

> 不知道这里发这贴合适否，发出来给大家玩一下。我就不打什么标题什么了。喜欢的能用的着的就看一下，以后碰到了自己就修改了，别老用别人的。自己的东西，用着塌实！！！  
>   
>   
> 主要修改文件是一个游戏程序的 DLL，这个游戏的方式有点让人想起来，那个《剑侠情缘》也是这样的风格，还有《大话西游》也是这样的风格，什么风格呢？很简单的，就是主程序在 DLL 中，而 EXE 只是负责加载而已！  
>   
> 觉得发这样的贴有点没水平了，不过，通常这样的最实用吧:)!  
>   
>   
> 简要说明一下：  
> 游戏主要加速有两类：一是 OpenGL,二是 DirectX  
> 而现在不知道怎么滴国产游戏都是后着，我们熟悉一下后者：  
>   
> /-----------------------------------------------------------------------------------------  
> 我们看到 3D 类游戏都要用到的一部分初始化，而熟悉这类编程的都说很简单的东西哦！  
>   
> HRESULT CPNDX::InitD3D(HWND hWd)  
> {  
>   hWnd=hWd;  
>   // 创建 D3D 对象  
>   if( NULL == ( g\_pD3D = Direct3DCreate8( D3D\_SDK\_VERSION ) ) )  
>     return E\_FAIL;  
>   
>   // 获取当前的显示模式  
>   D3DDISPLAYMODE d3ddm;  
>   if( FAILED( g\_pD3D->GetAdapterDisplayMode( D3DADAPTER\_DEFAULT, &d3ddm ) ) )  
>     return E\_FAIL;  
>   
>   D3DPRESENT\_PARAMETERS d3dpp;   
>   ZeroMemory( &d3dpp, sizeof(d3dpp) );  
>   
>   d3dpp.Windowed = TRUE;            // 窗口模式  
>   d3dpp.SwapEffect = D3DSWAPEFFECT\_DISCARD;  // 设置交换模式  
>       d3dpp.BackBufferWidth = d3ddm.Width;  
>       d3dpp.BackBufferHeight = d3ddm.Height;  
>   d3dpp.BackBufferFormat = d3ddm.Format;    // 设置背景缓冲区格式为当前左面格式  
>   d3dpp.EnableAutoDepthStencil = TRUE;    // 打开深度缓冲 zbuffer  
>   d3dpp.AutoDepthStencilFormat = D3DFMT\_D16;  // 深度缓冲格式  
>   
>   // 创建 D3D 设备  
>   // 第一个参数：使用默认的显卡适配器  
>   // 第二个参数：请求使用硬件抽象层（HAL）  
>   // 第三个参数：窗口句柄  
>   // 第四个参数：使用软件处理顶点  
>   // 第五个参数：创建的参数  
>   // 第六个参数：创建的 D3D 设备指针  
>   if( FAILED( g\_pD3D->CreateDevice( D3DADAPTER\_DEFAULT,   
>                     D3DDEVTYPE\_HAL,   
>                     hWnd,  
>                     D3DCREATE\_SOFTWARE\_VERTEXPROCESSING,  
>                     &d3dpp,   
>                     &g\_pd3dDevice ) ) )  
>   {  
>     return E\_FAIL;  
>   }  
>   
>   // 关闭 culling，让我们能看到 3 角型的正反面  
>   g\_pd3dDevice->SetRenderState( D3DRS\_CULLMODE, D3DCULL\_NONE );  
>   
>   // 打开 ZBUFFER  
>   g\_pd3dDevice->SetRenderState( D3DRS\_ZENABLE, TRUE );  
>   
>   // Turn off D3D lighting  
>     g\_pd3dDevice->SetRenderState( D3DRS\_LIGHTING, FALSE );  
>   
>   return S\_OK;  
> }  
> /---------------------------------------------------------------------------------------------------  
>   
> 我们注意的地方是：GetAdapterDisplayMode 这个函数全屏幕一般不需要，一般都是枚举  
> 而窗口化是必须要的，如果不要这个函数也可以，那就必须自己初始化 D3DDISPLAYMODE 结构才可以。  
> 比如：  
> //前面用其他函数给它初始化了宽和高，那么后面还要初始化屏幕格式和刷新频率。才能窗口化！  
>   d3ddm.Format     =D3DFMT\_R5G6B5;  
>   d3ddm.RefreshRate=0x3C;  
>   
>   
> 把前面的 D3DPRESENT\_PARAMETERS d3dpp 这个结构赋值修改如下：  
>     d3dpp.Windowed = FALSE;  
>     d3dpp.BackBufferCount = 1;  
>     d3dpp.BackBufferFormat = d3ddm.Format;  
>     d3dpp.BackBufferWidth = d3ddm.Width;  
>     d3dpp.BackBufferHeight = d3ddm.Height;  
>     d3dpp.hDeviceWindow = hWnd;  
>     d3dpp.SwapEffect = D3DSWAPEFFECT\_COPY\_VSYNC;   //这个也是必须的，不然屏幕会闪烁。  
>     d3dpp.FullScreen\_RefreshRateInHz = D3DPRESENT\_RATE\_DEFAULT;  
>     d3dpp.FullScreen\_PresentationInterval = D3DPRESENT\_INTERVAL\_ONE;  
>   
> 简要说完了，大家还是不太清楚就查查资料吧！没说太难，就通俗点说一下！  
> /----------------------------------------------------------------------------------------------------  
>   
> 下面的就简单了：  
> 我们来看一下，把一个程序反汇编会是什么样子：  
> //很容易就发现这个了：  
> 00406FD0  |.  68 DC000000   push    0DC  
> 00406FD5  |.  E8 987F0300   call    <jmp.&d3d8.Direct3DCreate8>  
>   
> \==============================  
> g\_pD3D = Direct3DCreate8( D3D\_SDK\_VERSION ); //也就是这一句了  
> /-----------------------------------------------------------------  
>   
> //再往下就看到了这两句：  
> 0040709D  |> \\C745 F8 17000>mov     dword ptr \[ebp-8\], 17  
> 004070A4  |.  C745 F4 3C000>mov     dword ptr \[ebp-C\], 3C  
>   
> \==================================  
>   d3ddm.Format     =D3DFMT\_R5G6B5;  
>   d3ddm.RefreshRate=0x3C;  
>   
> /------------------------------------------------------------------  
> //再往下看就到了一个结构，关建的是这个结构：  
> 004070BB  |.  C745 D4 01000>mov     dword ptr \[ebp-2C\], 1  
> 004070C2  |.  C745 C4 01000>mov     dword ptr \[ebp-3C\], 1  
> 004070C9  |.  8B55 F8       mov     edx, dword ptr \[ebp-8\]  
> 004070CC  |.  8955 C0       mov     dword ptr \[ebp-40\], edx  
> 004070CF  |.  8B45 EC       mov     eax, dword ptr \[ebp-14\]  
> 004070D2  |.  8945 B8       mov     dword ptr \[ebp-48\], eax  
> 004070D5  |.  8B4D F0       mov     ecx, dword ptr \[ebp-10\]  
> 004070D8  |.  894D BC       mov     dword ptr \[ebp-44\], ecx  
> 004070DB  |.  8B55 08       mov     edx, dword ptr \[ebp+8\]  
> 004070DE  |.  8955 D0       mov     dword ptr \[ebp-30\], edx  
> 004070E1  |.  C745 CC 04000>mov     dword ptr \[ebp-34\], 4  
> 004070E8  |.  C745 E4 00000>mov     dword ptr \[ebp-1C\], 0  
> 004070EF  |.  C745 E8 00000>mov     dword ptr \[ebp-18\], 0  
>   
> /------------------------------------------------------------------------------------------------  
> //再往下我们认识一个函数：  
> 004070F8  |.  6A 47         push    47  
> 004070FA  |.  6A 01         push    1  
> 004070FC  |.  6A 02         push    2  
> 004070FE  |.  8B45 F8       mov     eax, dword ptr \[ebp-8\]  
> 00407101  |.  50            push    eax  
> 00407102  |.  6A 01         push    1  
> 00407104  |.  6A 00         push    0  
> 00407106  |.  8B4D FC       mov     ecx, dword ptr \[ebp-4\]  
> 00407109  |.  8B51 04       mov     edx, dword ptr \[ecx+4\]  
> 0040710C  |.  8B45 FC       mov     eax, dword ptr \[ebp-4\]  
> 0040710F  |.  8B48 04       mov     ecx, dword ptr \[eax+4\]  
> 00407112  |.  8B01          mov     eax, dword ptr \[ecx\]  
> 00407114  |.  52            push    edx  
> 00407115  |.  FF50 28       call    dword ptr \[eax+28\]  
>   
> \=============================================================  
> m\_pD3D->CheckDeviceFormat(D3DADAPTER\_DEFAULT, D3DDEVTYPE\_HAL, d3ddm.Format,   
>     D3DUSAGE\_DEPTHSTENCIL, D3DRTYPE\_SURFACE, D3DFMT\_D32)  
>   
> /---------------------------------------------------------------------------------------------------  
> //还有一个关键函数：  
> 004071F3  |.  50            push    eax  
> 004071F4  |.  8D4D B8       lea     ecx, dword ptr \[ebp-48\]  
> 004071F7  |.  51            push    ecx  
> 004071F8  |.  6A 20         push    20  
> 004071FA  |.  8B55 08       mov     edx, dword ptr \[ebp+8\]  
> 004071FD  |.  52            push    edx  
> 004071FE  |.  6A 01         push    1  
> 00407200  |.  6A 00         push    0  
> 00407202  |.  8B45 FC       mov     eax, dword ptr \[ebp-4\]  
> 00407205  |.  8B48 04       mov     ecx, dword ptr \[eax+4\]  
> 00407208  |.  8B55 FC       mov     edx, dword ptr \[ebp-4\]  
> 0040720B  |.  8B42 04       mov     eax, dword ptr \[edx+4\]  
> 0040720E  |.  8B10          mov     edx, dword ptr \[eax\]  
> 00407210  |.  51            push    ecx  
> 00407211  |.  FF52 3C       call    dword ptr \[edx+3C\]  
>   
> \===============================================================  
>     //Create a Direct3D device.  
>     if(FAILED(m\_pD3D->CreateDevice(D3DADAPTER\_DEFAULT, D3DDEVTYPE\_HAL, hWnd,   
>                                    D3DCREATE\_SOFTWARE\_VERTEXPROCESSING, &d3dpp, &m\_pD3DDevice)))  
>   
> /----------------------------------------------------------------------------------------------------  
>   
> 窗口化的结束了---------------  
>   
>   
> /-------------------------------------------------------------------------------------------------------  
>   
> 看看全屏幕有什么不同呢，前面一样我就不贴了，主要看结构，根据代码不同就有所不同：  
>   
> 004070AD  |.  C745 D4 00000>mov     dword ptr \[ebp-2C\], 0    //必须  
> 004070B4  |.  C745 C4 01000>mov     dword ptr \[ebp-3C\], 1  
> 004070BB  |.  8B55 F8       mov     edx, dword ptr \[ebp-8\]  
> 004070BE  |.  8955 C0       mov     dword ptr \[ebp-40\], edx  //EDX=16  
> 004070C1  |.  8B45 EC       mov     eax, dword ptr \[ebp-14\]  
> 004070C4  |.  8945 B8       mov     dword ptr \[ebp-48\], eax  //EAX=窗口宽，  
> 004070C7  |.  8B4D F0       mov     ecx, dword ptr \[ebp-10\]  
> 004070CA  |.  894D BC       mov     dword ptr \[ebp-44\], ecx  //ECX=窗口高  
> 004070CD  |.  8B55 08       mov     edx, dword ptr \[ebp+8\]  
> 004070D0  |.  8955 D0       mov     dword ptr \[ebp-30\], edx  //EDX=HWND 句柄  
> 004070D3  |.  C745 CC 04000>mov     dword ptr \[ebp-34\], 4  
> 004070DA  |.  C745 E4 00000>mov     dword ptr \[ebp-1C\], 0    //必须  
> 004070E1  |.  C745 E8 01000>mov     dword ptr \[ebp-18\], 1    //必须  
>   
> /---------------------------------------------------------------------------------------------------------  
>   
> 现在大家都明白了吧！就是改个结构的意思了，简单的很。。。和我一样的菜鸟都来改吧！前面是我自己写的代码，现在正式开工：  
>   
> 100198B0  /$  83EC 14       sub     esp, 14  
> 100198B3  |.  56            push    esi  
> 100198B4  |.  8BF1          mov     esi, ecx  
> 100198B6  |.  6A 1F         push    1F  
> 100198B8  |.  E8 63930100   call    <jmp.&d3d9.Direct3DCreate9>      ;创建 D3D 了，我们很容易就找到了，如果看了我  
> 100198BD  |.  85C0          test    eax, eax                         ;关于 StarForce 的朋友都知道，这个是怎么找的吧！  
> 100198BF  |.  8986 14070000 mov     dword ptr \[esi+714\], eax  
> 100198C5  |.  75 15         jnz     short 100198DC  
> 100198C7  |.  6A 02         push    2                                ; /Arg2 = 00000002  
> 100198C9  |.  68 01000082   push    82000001                         ; |Arg1 = 82000001  
> 100198CE  |.  8BCE          mov     ecx, esi                         ; |  
> 100198D0  |.  E8 3B140000   call    1001AD10                         ; \\gbengine.1001AD10  
> 100198D5  |.  33C0          xor     eax, eax  
> 100198D7  |.  5E            pop     esi  
> 100198D8  |.  83C4 14       add     esp, 14  
> 100198DB  |.  C3            retn                                     ;这里是返回了肯定不对。而且它返回的是指针，不可能是 NULL  
>   
>   
> //仔细发现会出现在这里：  
> 10019A68  |> \\8BCE          mov     ecx, esi  
> 10019A6A  |.  E8 B10C0000   call    1001A720                         ;这里就 CALL 进去了  
> 10019A6F  |.  85C0          test    eax, eax  
> 10019A71  |.  7D 11         jge     short 10019A84  
> 10019A73  |.  6A 02         push    2                                ; /Arg2 = 00000002  
> 10019A75  |.  50            push    eax                              ; |Arg1  
> 10019A76  |.  8BCE          mov     ecx, esi                         ; |  
> 10019A78  |.  E8 93120000   call    1001AD10                         ; \\gbengine.1001AD10  
> 10019A7D  |.  33C0          xor     eax, eax  
> 10019A7F  |.  5E            pop     esi  
> 10019A80  |.  83C4 14       add     esp, 14  
> 10019A83  |.  C3            retn  
>   
> //跟进 call    1001A720 ，往下翻一看就知道这个地方是 D3D 的 API 调用的  
> //仔细跟到这里窗口果然全屏幕了，这么一大段怎么改啊！  
> 1001A8A0   .  57            push    edi  
> 1001A8A1   .  55            push    ebp  
> 1001A8A2   .  8BAE B8000000 mov     ebp, dword ptr \[esi+B8\]  
> 1001A8A8   .  8B10          mov     edx, dword ptr \[eax\]  
> 1001A8AA   .  53            push    ebx  
> 1001A8AB   .  55            push    ebp  
> 1001A8AC   .  8B6C24 20     mov     ebp, dword ptr \[esp+20\]  
> 1001A8B0   .  8B6D 04       mov     ebp, dword ptr \[ebp+4\]  
> 1001A8B3   .  55            push    ebp  
> 1001A8B4   .  51            push    ecx  
> 1001A8B5   .  50            push    eax  
> 1001A8B6   .  FF52 40       call    dword ptr \[edx+40\]  
>   
> /----------------------------------------------------------------------------------  
> 别头疼来仔细看一下，往上：  
> 我们会发现 mov     eax, dword ptr \[esi+714\]这个很关键，  
> mov     edx, dword ptr \[eax\]  
> 下面就是函数调用了：  
> call    dword ptr \[edx+40\] //我们只要它就可以了  
>   
> 然后在前面随便一个代码地方 NOP 掉。。。然后添加自己的代码：  
> 1001A880   .  84C0          test    al, al  
> 1001A882   .  E9 79D00B00   jmp     100D7900 //跳到我们修补的代码地方  
> 1001A887      90            nop  
> 1001A888      90            nop  
> 1001A889      90            nop  
> 我选择了上面的地方，注意地方一定不是与主程序有关的，要不然会影响程序正常的：  
>   
> /-----------------------------------------------------------------------------------  
>   
> //我们把代码修补如下：  
> 100D7900   > \\8B86 14070000 mov     eax, dword ptr \[esi+714\]  //把 D3D 指针调出来用  
> 100D7906   .  8B38          mov     edi, dword ptr \[eax\]      //放入 EDI  
> 100D7908   .  68 E0780D10   push    100D78E0                  //准备了一个地方存放 D3DDISPLAYMODE 结构  
> 100D790D   .  6A 00         push    0                         //目前默认吧  
> 100D790F   .  50            push    eax                         
> 100D7910   .  FF57 20       call    dword ptr \[edi+20\]        //我们很容易就找到了\[edi+20\]=GetAdapterDisplayMode 函数  
> 100D7917   .  85C0          test    eax, eax                  //检测返回值  
> 100D7919   .^ 0F85 6D2FF4FF jnz     1001A88C                  //显示器模式初始化失败，就从这里处理  
>   
> //上面已经分析了，结构的基本信息我不解释了，这个是我自己添加的结构内容，来实现窗口化！  
> 100D791F   .  B8 80780D10   mov     eax, 100D7880            //这个是结构存放的地址  
> 100D7924   .  8D96 DC060000 lea     edx, dword ptr \[esi+6DC\]  
> 100D792A   .  8B0A          mov     ecx, dword ptr \[edx\]  
> 100D792C   .  8908          mov     dword ptr \[eax\], ecx  
> 100D792E   .  8B4A 04       mov     ecx, dword ptr \[edx+4\]  
> 100D7931   .  8948 04       mov     dword ptr \[eax+4\], ecx  
> 100D7934   .  C740 08 17000>mov     dword ptr \[eax+8\], 17  
> 100D793B   .  C740 0C 00000>mov     dword ptr \[eax+C\], 0  
> 100D7942   .  C740 10 00000>mov     dword ptr \[eax+10\], 0  
> 100D7949   .  C740 14 00000>mov     dword ptr \[eax+14\], 0  
> 100D7950   .  8B4A 18       mov     ecx, dword ptr \[edx+18\]  
> 100D7953   .  8948 18       mov     dword ptr \[eax+18\], ecx  
> 100D7956   .  8B4A 1C       mov     ecx, dword ptr \[edx+1C\]  
> 100D7959   .  8948 1C       mov     dword ptr \[eax+1C\], ecx  
> 100D795C   .  C740 20 01000>mov     dword ptr \[eax+20\], 1  
> 100D7963   .  C740 24 01000>mov     dword ptr \[eax+24\], 1  
> 100D796A   .  C740 28 50000>mov     dword ptr \[eax+28\], 50  
> 100D7971   .  C740 2C 01000>mov     dword ptr \[eax+2C\], 1  
> 100D7978   .  C740 30 00000>mov     dword ptr \[eax+30\], 0  
> 100D797F   .  C740 34 00000>mov     dword ptr \[eax+34\], 0  
>   
> //然后自己再写这个 CreateDevice 函数调用，我们就实现了 D3D 的窗口初始化，因此我们的游戏也就窗口化了。  
> 100D798C   .  52            push    edx  
> 100D798D   .  68 80780D10   push    100D7880  
> 100D7992   .  6A 20         push    20  
> 100D7994   .  8B96 B8000000 mov     edx, dword ptr \[esi+B8\]  
> 100D799A   .  52            push    edx  
> 100D799B   .  6A 01         push    1  
> 100D799D   .  6A 00         push    0  
> 100D799F   .  8B86 14070000 mov     eax, dword ptr \[esi+714\]  
> 100D79A5   .  50            push    eax  
> 100D79A6   .  8B38          mov     edi, dword ptr \[eax\]  
> 100D79A8   .  FF57 40       call    dword ptr \[edi+40\]  
> 100D79AB   .^\\E9 092FF4FF   jmp     1001A8B9   
>   
> 直接跳到下面：  
> 1001A8B9   > /85C0          test    eax, eax  
> 1001A8BB   . |0F8C 0C030000 jl      1001ABCD  
> 1001A8C1   . |8BCE          mov     ecx, esi  
>   
> //上面正好是原来的 CreateDevice 函数调用，我们自己写了一段，把原来的代替了，这个方法有点像 HOOK API:)  
>   
>   
> /--------------------------------------------------------------------------------------  
>   
> 实在很简单的吧。。。哈哈！D3D 确实有的时候很容易就窗口化了。  
>   
> 大家试试吧！我发出来的方法努力是做到简单方便，而又能适合和我一样菜鸟的人嘻嘻！  
>   
>   
>                                                                   -By EasyStudy For PhantomNet  
>   

 4dd466d0e1e0cc123afbe504da2462ca