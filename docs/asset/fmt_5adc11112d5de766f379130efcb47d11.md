
# Direct3D 透视教程，教你做出属于自己的透视

> 原文：[`www.pediy.com/kssd/pediy12/126933.html`](https://www.pediy.com/kssd/pediy12/126933.html)

大家好，我搞编程有一段时间了吧，终于做出了一些自己的东西。最近一直在弄 D3D 游戏的透视，被他弄得焦头烂额，很多时间都花在了解决代码错误的问题上。不过功夫不负有心人我取得了一些成果，在编写代码的过程当中参阅了大量的 DirectX 编程方面的资料。同时也在国外 N 多论坛做了一个名副其实的伸手党，了解和掌握了 D3D 程序实现透视的一些基础知识。在这里我把我自己在研究过程中的心得体会和一些经验大体整理了一下，只是希望能够找到对 D3D 感兴趣的朋友能够共同研究，话不多说了，下面开始咱们的教程。

在看这篇教程之前，你最好有一些基本的 C++ 的知识，此外还需要一些基本的 DirectX 编程的基础。我的教程呢分为两个部分，第一部分呢主要讲一些透视原理。第二部分主要讲透视 dll 的制作过程。![](img/2aa97d64a7e0b211d2f8f207eb382aa7.jpg "酷")

## 透视原理

首先来看第一部分，主要是讲解一些透视的原理，

在讲透视之前首先来说一下什么是 Z 轴缓冲

Z 轴缓冲就是 DirectX 中的成像元素之一，他是一个深度缓冲，用来决定渲染对象的遮蔽关系，在计算机图形学中，有时候通过硬件完成，有时候通过软件完成。在观察被渲染对象的时候就出现这样一个问题，如果一个对象在另一个对象的后面，并且距离观察者的距离较远那么我们是看不到他的。Z 轴缓冲也就是深度缓冲。

上面说的可能有点抽象我就直白的说一下吧：Z 轴缓冲就是要告诉摄像机，如果他在一个对象的后面，就不用显示它了，如果他在这个对象前面，就得显示他。现在根据上面所说的，你可能要说把 Z 轴缓冲禁用来实现透视就是了。但事实情况并不是这样。你会注意到，你会看到所有的东西甚至还有场景之外的东西也就是说你必须找到某一个对象，再选这个对象的时候来禁用 Z 轴缓冲。这就是：`Strides`（数据流中每个顶点所占内存的大小），`NumVertices`（渲染的顶点索引的跨度），和 `PrimitiveCounts`（渲染的图元个数）。

上面这三个就决定了透视到墙（物体）后你想看到的东西，在大部分情况下就是人物模型（也就是想弄人物透视）

如果直接禁用 Z 轴缓冲的话，那就是全图透视了，玩过 AVA 的都知道全图透视在冰冻工厂是可以使用的，不过在其他地图，或者可以说在其他游戏里，全图透视效果是很差的，所以要实现人物透视就必须找到游戏中的人物模型表示。例如，CF 的人物模型表示为 `44` 和 `40`，AVA（战地之王的人物模型标识为 `32`），所以我们说的人物透视是必须找到人物模型的表示然后再禁用 Z 轴缓冲

下面我们来禁用 Z 轴缓冲了。

首先把人物模型标识来定义一下：

```c
#define PlayerBody ( Stride == 44 || Stride == 40 )
// 注意这个 " || " 是“或”的意思。意思就是说"PlayerBody  等价于 44 或者是 40。
//这里的 Stride 即为游戏中的人物模型标识
```

好了，上面的代码是对人物模型的标识进行了定义，下面就要开始 D3D9 中的 `DrawIndexedPrimitive` 函数来修改游戏中人物模型的渲染状态

下面的代码就是在 hook the `DrawIndexedPrimitive` 这函数之中

```c
LPDIRECT3DVERTEXBUFFER8 Stream_Data;

  UINT Stride = 0;

  if (m_pD3DDev->GetStreamSource(0, &Stream_Data, &Stride) == D3D_OK)

  Stream_Data->Release();

if(Chams)//这里的 chams 是透视开关      if(Player)

      {
          {
//一旦游戏中 Stride 的值等于所设定的值，就开始执行以下代码：

DrawIndexedPrimitive(Device, Type,  MinIndex,  NumVertices, StartIndex, PrimitiveCount);

//在墙后面就是绿色    Device->SetRenderState( D3DRS_ZENABLE,false );

    Device->SetTexture( 0, texGreen );

//禁用 Z 轴缓冲    DrawIndexedPrimitive(Device, Type, MinIndex, NumVertices, StartIndex,  PrimitiveCount);

//游戏模型在墙前面的话（就是可以看见）    Device->SetRenderState( D3DRS_ZENABLE, true );

//打开 Z 轴缓冲，并填充为蓝色    Device->SetTexture( 0, texBlue);

    DrawIndexedPrimitive(Device, Type,  MinIndex,  NumVertices, StartIndex, PrimitiveCount);

          }
      }
```

我们继续：![](img/f3773294bf3a914c9a8c0c242528a003.jpg "使眼色")

如果你想知道透视上色的代码是如何运行的，它是通过消息队列进行操作的。

下面就是他的工作流程：

如果人物要被显示:

给人物上色为红色.

启用 Z 轴缓冲，在物体前面的时候为红色。

禁用 Z 轴缓冲

在物体后面的时候就为黄色

Disable stencil shaders.（这个我暂时还没搞懂）

然后这样不断重复。说到重复，有人就会问了，什么是重复。我简单地说一下吧。游戏在渲染人物模型的时候，是一帧一帧的进行渲染的，放在显存中的中的每幅图像从后备缓冲区到前台缓冲区并呈现到屏幕上。打个形象的比喻，好比放幻灯片。速度慢了，可以看成是一张一张的，当速度很快时，就可以看成连续的画面。所以上面的渲染过程至少每秒要执行 24 次，要不然画面就会很卡。![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")

![](img/093abe813ca13c0ee756f772e96da9d0.jpg)

![](img/8166758166cee761aaaaa84617f5208c.jpg)

这是我在 AVA 中实现的透视效果图，当然了，这里我是以 AVA 为例子来讲解的，其他的 FPS 游戏都大同小异，AVA 的透视半透和全透只是一个函数的差别，就是遮盖剔除函数。加上遮盖剔除函数，半透就可以变成全透

这就是透视的整个流程，其实上面只是写了最核心的人物透视，如果你还想添加其他功能，比如说准星，透视上色、字幕显示，功能菜单等等都可往里面添加函数，我会在教程的最后把 D3D 的透视模板发出来，大家想添加什么功能的话只要往里面加入代码就可以了

上面只是实现人物透视的最核心的部分，也是实现人物透视的实质内容。整个实现透视的流程为：

找到所需函数的地址 > 修改函数地址（拦截 API） 写入你想修改的代码（比如画准星，禁用 Z 轴缓冲，上色等操作）> 返回正常值

找到你想要 HOOK 的函数地址，由于 DirectX 是 COM 组件，普通的函数 HOOK 是找不到这些函数的，我们必须用能 HOOK COM 对象的函数，这就是 `Detour`，利用 `Detour` 我们就可以 HOOK COM 对象里的函数了。

```c
pEndScene = (EndScene_)DetourFunction((PBYTE)VTable[ES],(PBYTE)pEndScene);
```

上面就是一个 `detour` 的使用例子，其实很多功能都是在这个函数里面实现的，例如画准星，菜单显示等等。那么第一个参数是怎么来的呢，这里我们就需要找到 `EndScene` 这个函数的地址，怎么找呢，有两种方法：

1.  修改 PE 导入地址表的函数地址，使游戏在调用函数的时候转到我们的函数上来
2.  直接找到 `EndScene` 函数的基址和偏移，然后把这个地址的函数改成我们自己的函数

举个例子我们可以在 `EndScene` 里面去设置开关，画准星以及渲染方框

```c
if(bTip)    //bTip 初始化为 true ，注入成功进入游戏后直接显示  {
DrawFont( 600,50, D3DCOLOR_ARGB(255,0,255,255), "小键盘-> 0 文字提示, 1 准星开关，人物透视"); 

DrawFont( 600,70, D3DCOLOR_ARGB(255,0,255,255), " Coded By 顶级小白 QQ：812191628");

  }

  if(crosshair)//画准星  

          {
    D3DVIEWPORT9 viewP;                    //先定义一个接口指针
            pDevice->GetViewport( &viewP );       //获得屏幕宽度和高度
      DWORD ScreenCenterX = viewP.Width / 2;//屏幕中心 X 位置
      DWORD ScreenCenterY = viewP.Height / 2; //屏幕中心 Y 位置
      D3DRECT rec16 = {ScreenCenterX-15, ScreenCenterY, ScreenCenterX+ 15, ScreenCenterY+1};//这个 D3DRECT 是一个结构体。就是指定了画的这条直线的范围，水平的那条线
      D3DRECT rec17 = {ScreenCenterX, ScreenCenterY-15, ScreenCenterX+ 1,ScreenCenterY+15};  //这个是垂直的那条线的上下两个点
      pDevice->Clear( 1, &rec16, D3DCLEAR_TARGET, D3DCOLOR_XRGB( 0, 0, 255 ), 0,  0 );//clear 方法可以指定使用某种颜色把后备缓冲区粉刷一遍（也可以是深度缓冲区，模板缓冲区）
      pDevice->Clear( 1, &rec17,D3DCLEAR_TARGET, D3DCOLOR_XRGB( 0, 0, 255 ),0,  0 );//要清空的矩形区域的数目(第一个参数)//要清空的举行的数组，和第一个参数决定了要清除的矩形区域的组合（第二个参数）

  }

   //下面开始渲染方框

    DrawBox (100,100,100,100,D3DCOLOR_ARGB(255, 255, 0, 255), D3DCOLOR_XRGB( 255, 0, 0 ),pDevice); //红色准星

  return pEndScene(pDevice);//这里就是返回正常值
```

```c
void FillRGB( int x, int y, int w, int h, D3DCOLOR color, IDirect3DDevice9* pDevice ) 

{ 

    if( w < 0 )w = 1; 

    if( h < 0 )h = 1; 

    if( x < 0 )x = 1; 

    if( y < 0 )y = 1; 

    D3DRECT rec = { x, y, x + w, y + h }; 

    pDevice->Clear( 1, &rec, D3DCLEAR_TARGET, color, 0, 0 ); 

} 

void DrawBorder( int x, int y, int w, int h, int px, D3DCOLOR BorderColor, IDirect3DDevice9* pDevice ) 

{ 

    FillRGB( x, (y + h - px), w, px,    D3DCOLOR_XRGB( 255, 0, 0 ), pDevice ); //填充颜色

    FillRGB( x, y, px, h,                D3DCOLOR_XRGB( 255, 0, 0 ), pDevice ); 

    FillRGB( x, y, w, px,                D3DCOLOR_XRGB( 255, 0, 0 ), pDevice ); 

    FillRGB( (x + w - px), y, px, h,    D3DCOLOR_XRGB( 255, 0, 0 ), pDevice ); //红色方框

} 

void DrawBox( int x, int y, int w, int h, D3DCOLOR BoxColor, D3DCOLOR BorderColor, IDirect3DDevice9* pDevice ) //渲染一个方框

{ 

    DrawBorder( x, y, w, h, 1,    BorderColor, pDevice );// 画方框边界

}  
```

通过上面所说的就可以找到函数地址并进行 HOOK..（这其中还有很多小细节需要注意，但由于篇幅的原因，我只大概的介绍这些）

好了，第一部分暂且讲到这里，如果你有不明白的地方建议你多去看看 DirectX 编程方面的资料。

## 透视 dll 的制作

下面开始第二部分来说一说透视 dll 的制作：

Dll 的制作我是翻译的国外论坛的几篇帖子，先说明一下需要的工具：

Microsoft Visual C++ 6.0

VS 2008 也可以（本人用的就是 VS2008）

然后还需要 DX 的 SDK

DirectX 9.0 SDK Update - (Summer 2009)

D3D Starter Kit 3B

这份代码我会打包放到附件里

在你下载并安装了 C++6.0 以及 D3D SDK 之后

你就可以使用 C++ 用 SDK lib 文件了

好，打开 C++ 6（2008，2003 无所谓）

打开“工具”

![](img/7755bf375bb7fd5e590f316d6d95c019.jpg)

>选项>目录（Directories）

在路径那里选择 SDK 的文件夹

添加 Microsoft Directx 9.0 SDK include 文件夹

如下图所示：（我机子上 VC6.0 和 9.0 都安装了，第一个图这是我自己截的图）

![](img/889f1dcb4ae313ce64d18e270e25afc5.jpg)

![](img/780014f7d0f308c111c3540c82ae62ad.jpg)

做完这些之后, 开始添加“lib”文件了

在“目录”这里

![](img/f2e9fd0255c7fd3a37a82d9f54e9921d.jpg)

然后选择 Microsoft Directx 9.0 SDK lib 文件夹（我机子上 VC6.0 和 9.0 都安装了，原文截图已经丢失，这是我自己截的图）

![](img/4aa661a1209afcb676ff2bc418a676f3.jpg)

点击 OK，这样工具就设置 OK 了

现在我们可以启动 starter kit 这样我们就可以用它了

打开 Starter kit 文件夹, 来到 D3D9 文件夹

如下图所示（我机子上 VC6.0 和 9.0 都安装了，原文截图已经丢失，这是我自己截的图）

![](img/80e0229a78455438ed57292f01389539.jpg)

现在打开 `old_workspace` 文件夹, 复制两个文件到 D3D9 文件夹中（不知道是哪两个文件。。。）

然后打开 VC6.0，点击“文件”----“打开工作空间”----选择 `TatniumD3D.dsw`

做完这些之后你就可以去编译 dll 了。

大家可能注意到了，这两张截图我都渲染了一个方框，但苦于找不到人物的坐标，所以在这里我也想找几位对 D3D 透视感兴趣的朋友（熟悉 C++和 DirectX 编程）一块研究方框透视还有 CF 的透视，毕竟一个人的力量有限。我 QQ：812191628

透视模板源代码.zip

透视原始代码.zip
