

# Windows XP 系统自带扑克牌资源动态链接库 cards.dll 逆向分析笔记

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-229.htm`](https://www.pediy.com/kssd/pediy09/pediy09-229.htm)

> 真是晕啊，这篇文章写到一半的时候竟然有人说 cards.dll 的源代码可以在 MSDN Library 中找到，我看了一下本机的 MSDN，反正我是没找到，有谁知道在哪找的麻烦指点一下，哎，害的我后面写那么一大堆废话来凑篇幅  
>   
> Windows XP 系统自带扑克牌资源动态链接库 cards.dll 逆向分析笔记  
>   
>   
> 使用工具：IDA Pro, Resource Hacker  
>   
>   
> 0. 前言  
>   
> 　　cards.dll 是 Windows 系统目录下的一个动态链接库，主要提供扑克牌图像及相关操作等资源，以供  
> Windows 附带的扑克游戏程序（如纸牌、红心大战等）使用。  
>   
> 　　我们希望知道 cards.dll 具体提供了哪些东西，可供自由编程所用。  
>   
>   
> 1. 反编译  
>   
>   
> 　　一般而言，将原始二进制文件还原成高级语言源文件的逆向工程有两个步骤：一是反编译，根据目  
> 标文件反汇编的内容，识别指令、存储单元等基本要素并理解这些要素之间的相互关系，从而写出相应  
> 的高级语言语句；二是自文档化，根据程序的上下文，理解程序中各个变量所代表的含义，给它们重新  
> 赋予意义明了的名称，必要时添加一定的注释，以使得源程序具有一定的文档性，容易看懂和理解。  
>   
> 　　由于这里是人工逆向，这两个步骤分得并不清楚，实际上也难以分清楚。毕竟，原始的反汇编文本  
> 中充斥着大量形如 dword\_70142004 以及 var\_8 之类缺乏意义的符号，不同变量的名字看上去都差不多，  
> 如果不及时给它们起个容易分辨的名称，就很容易弄错。这样看来，上述的第二步就部分地提前了。同  
> 时，为了简洁起见，在下面的正文中也不打算贴那种大片的反汇编文本（事实上也不必要，若要看这些  
> 内容，自己用 IDA 打开这个 DLL 文件就能看到了），只提供还原出来的源代码，源代码中的变量都已赋予  
> 合适的名称。这就是说，反编译的过程在正文中也省略了。  
>   
> 　　但毕竟反编译是本文的一个重要组成部分，不能将它真正省略掉。由于要说清楚这个过程势必又要  
> 附上大量的反汇编文本，因此，关于反编译的一般原则（如识别高级语言的控制结构等内容）我会以附  
> 录形式另起一章，而具体的反汇编过程则把它放到附件里了，附件里的 origdasm.asm 是最初的没有经过  
> 任何分析的反汇编文本，而 annodasm.asm 是分析过后的文本，我的分析主要以注释形式放在其中，大家  
> 可以对照着看。  
>   
> 　　IDA 加载后切换到 Exports 选项卡，可以看到 7 个导出函数：  
>   
> 　　  WEP  
>       cdtAnimate  
>       cdtDraw  
>       cdtDrawExt  
>       cdtInit  
>       cdtTerm  
>       DllEntryPoint  
>   
> 其中 DllEntryPoint 就是通常所谓的 DllMain。其代码如下：  
>   
>   
> 
> 代码:
> 
> * * *
> 
> `  HINSTANCE hDllModule;    //全局变量     BOOL DllMain (HINSTANCE hInstDll, DWORD dwReason, LPVOID lpReserved)     {         hDllModule = hInstDll;             return TRUE;     }　`
> 
> * * *
> 
>   
> WEP 和 cdtAnimate 这两个函数都极其简单：  
>   
> 代码:
> 
> * * *
> 
> `　BOOL __stdcall WEP (int Arg1)     {         return TRUE;     }        BOOL __stdcall cdtAnimate (int Arg1, int Arg2, int Arg3, int Arg4, int Arg5)     {         return TRUE;     }`
> 
> * * *
> 
> 返回值（eax 的出口值）只有 0 和 1 两种可能的函数，我们认为它是 BOOL 类型的。如此简单的函数为什么  
> 还要导出呢？实际上，大体可以推测这些函数在前一个版本的操作系统中曾经是有用的，但是后来由于  
> 种种原因，这些函数被废弃了，函数体的代码已基本被删掉，成为我们现在看到的空壳子。  
>   
> 　　所有导出函数中最复杂的莫过于 cdtDrawExt，这是一个绘画扑克牌的函数。我们还是先从比较简单  
> 的 cdtInit 和 cdtTerm 两个函数入手，避免一开始就啃硬骨头。  
>   
> 　　cdtInit 函数中包含一个 GetObject 函数的调用，这是逆向 cdtInit 的突破口。弄清了这个函数调用  
> 的作用后，其他就简单了。  
>   
>   
> 代码:
> 
> * * *
> 
> `//全局变量     int nNormalWidth;         //正常大小下卡片的宽      int nNormalHeight;        //正常大小下卡片的长     int nInitCount = 0;       //“初始化计数”，每调用 cdtInit 一次增加 1，                               //每调用 cdtTerm 一次减少 1     HBITMAP hPlainBack = NULL;    //平实型背面的位图句柄     HBITMAP hCrossBack = NULL;    //斜十字图案型背面的位图句柄     HBITMAP hCircleBack = NULL;   //圆圈图案型背面的位图句柄   //函数定义体       BOOL __stdcall cdtInit (LPINT lpNormalWidth, LPINT lpNormalHeight)     {         BITMAP loc_stBM;            if (nInitCount++ == 0)         {              hPlainBack = LoadBitmap (hDllModule, 53);              hCrossBack = LoadBitmap (hDllModule, 67);              hCircleBack = LoadBitmap (hDllModule, 68);                            if (hPlainBack == NULL || hCrossBack == NULL || hCircleBack == NULL)              {                   MyDeleteHbm (hPlainBack);                   MyDeleteHbm (hCrossBack);                   MyDeleteHbm (hCircleBack);                   return FALSE;              }                 GetObject (hPlainBack, sizeof(BITMAP), &loc_stBM);                 nNormalWidth = *lpNormalWidth = loc_stBM.bmWidth;              nNormalHeight = *lpNormalHeight = loc_stBM.bmHeight;         }else{              *lpNormalWidth = nNormalWidth;              *lpNormalHeight = nNormalHeight;         }         return TRUE;     }`
> 
> * * *
> 
>   
> 由此可见，cdtInit 首次被调用时会装入 3 种样式的卡片背面位图，并且获得和保存这些位图的尺寸。如  
> 果装入位图失败，则返回值是 FALSE，应用程序就不能使用这些风格的背面。而往后该函数若再次被调  
> 用时（这时 nInitCount > 0），则不做任何事情，只是简单地把首次调用时保存下来的卡片尺寸返回。  
> 由于 cdtInit 的两个参数是指针类型，而且函数体内对其所指向的对象进行写操作，所以实参不能是空  
> 指针。正常的用法是，在应用程序中定义两个全局变量并将它们的地址作为实参传递给 cdtInit 函数，  
> 如此就可以在应用程序模块和库模块中各自保存一份卡片正常尺寸的数据信息。  
>   
> 　　cdtInit 中还调用了一个叫 MyDeleteHbm 的函数，其代码如下：  
>   
>   
> 代码:
> 
> * * *
> 
> `  void __stdcall MyDeleteHbm (HBITMAP hBM)     {         if (hBM)            DeleteObject (hBM);         return;     }`
> 
> * * *
> 
>   
> 实际上这个函数可以认为就是 DeleteObject，只不过显得更加保险点而已。  
>   
> 　　cdtTerm 的代码也很简单：  
>   
>   
> 代码:
> 
> * * *
> 
> `//全局变量     HBITMAP hCards[52];   //一副扑克牌的位图句柄     HBITMAP hCustomBack;  //自选的卡片背面的位图句柄   //函数定义体     void cdtTerm (void)     {         int loc_i;            if (--nInitCount <= 0)         {             for (loc_i = 0; loc_i < 52; loc_i++)                 MyDeleteHbm (hCards[loc_i]);                MyDeleteHbm (hPlainBack);             MyDeleteHbm (hCustomBack);             MyDeleteHbm (hCrossBack);             MyDeleteHbm (hCircleBack);         }         return;     }`
> 
> * * *
> 
>   
> 显然，这个函数的主要功能是用来做些善后工作，但当“初始化计数”尚大于 1 时则什么也不做。在 IDA  
> 中查看变量 nInitCount 的引用参考可以发现，只有上述的 cdtInit 和 cdtTerm 两个函数引用到它，前者将  
> 其加一而后者将其减一。因此程序中 cdtInit 和 cdtTerm 必须成对使用，才能保证最后一次调用 cdtTerm  
> 时善后工作得以完成，就如同熟知的 LoadLibrary/FreeLibrary 那样。  
>   
> 　　接下来是主菜 cdtDrawExt 登场了。在正式给出其源代码之前罗嗦几句。跟踪这个函数之难，不  
> 惟在于其规模的庞大，同时亦在于其反汇编代码的若干不规范之处，例如它中途将栈上原本存放形参的  
> 存储空间挪去做了局部变量的用途，而且这个局部变量跟被替代的形参还不是同一类型。我开始就是忽  
> 略了这一点，结果错认了参数的类型，不得不从头开始。由于代码优化等原因，寄存器被频繁使用，当  
> 一个寄存器所寄存的不同对象多起来的时候，要理出头绪就不那么容易了。虽然从原则上说，非出口的  
> 形参在子程序中不应该被改写，而所有的寄存器到了最后都可以被消去（见附录），但这些工作毕竟很  
> 难一步到位。于是，除了子程序原有的局部变量外，我们还不得不创建一些临时变量，以代表这些形参  
> 和寄存器参与源代码的构建，到最后阶段再设法消去它们。在本函数建议用一个双字变量来代表寄存器  
> eax，用两个双字变量代表 Arg3（第三个参数，即原始反汇编文本中的 arg\_8）。  
>   
> 　　逆向这个函数的突破口是其中一连串的 PatBlt 函数调用，根据它的形参类型可以确定 cdtDrawExt 大  
> 部分形参的类型和意义，剩下的可以连猜带蒙弄出来。只要知道参数的意义是什么，其他就容易了。  
>   
> 　　下面是这个函数的代码：  
>   
>   
> 代码:
> 
> * * *
> 
> `  BOOL __stdcall cdtDrawExt (HDC hDC,                //将要绘画的目标设备环境句柄                      int nXDest,             //nXDest 和 nYDest 两个参数表示一个点的 x 和 y 坐标                      int nYDest,             //以该点为左上角绘画卡片                      int nActualWidth,                      int nActualHeight,      //以上两个参数表示所绘卡片的实际宽和长                      int nIndexOfCard,       //指定所绘卡片的序号                      DWORD dwDrawingOption,  //绘画方式选项                      COLORREF crBackColor    //背景色                     )     {          COLORREF loc_crCornerData[12];     //卡片四个角的颜色数据。用于圆角方式绘画卡片          COLORREF loc_crBkColor = crBackColor;   //拷贝一份 crBackColor 的值          static HGDIOBJ hObjectToDraw;                   POINT loc_stP1;          int loc_dwA;         //代表寄存器 eax          DWORD loc_dwROP = 0;               //ROP 码          DWORD loc_dwDO = dwDrawingOption; //拷贝一份 dwDrawingOption 的值          BOOL loc_bSharpCorner = FALSE;    //绘画卡片时选择圆角或尖角          HGDIOBJ loc_hObj1;          HDC loc_hCompDC;                  //这两个变量都代表反汇编文本中的 arg_8（Arg3）             if (loc_dwDO & 0x80000000)          {              loc_dwDO -= 0x80000000;              loc_bSharpCorner = TRUE;          }             if (loc_dwDO <= 7)          {              switch (loc_dwDO)              {                  case 0:                     hObjectToDraw = _LoadCard (nIndexOfCard);                     loc_dwROP = SRCCOPY;                     loc_crBkColor = 0xFFFFFF;   //白色背景                     break;                  case 1:                     if (FLoadBack (nIndexOfCard) == FALSE) return FALSE;                     hObjectToDraw = hCustomBack;                     loc_dwROP = SRCCOPY;                     break;                  case 2:                     hObjectToDraw = _LoadCard (nIndexOfCard);                     loc_dwROP = NOTSRCCOPY;     //反色绘制卡片                     break;                  case 3:                  case 4:                     if ((loc_hObj1 = CreateSolidBrush (loc_crBkColor)) == NULL)                         return FALSE;                     GetDCOrgEx (hDC, &loc_stP1);                     SetBrushOrgEx (hDC, loc_stP1.x, loc_stP1.y, NULL);                     if ((loc_hObj1 = SelectObject (hDC, loc_hObj1)) != NULL)                     {                                               PatBlt (hDC, nXDest, nYDest, nActualWidth,                                                      nActualHeight, PATCOPY);                          //在卡片区域涂上 crBackColor 指定的颜色                         if ((loc_dwA = (int)SelectObject (hDC, loc_hObj1)) != NULL)                             DeleteObject ((HGDIOBJ)loc_dwA);                          //恢复 DC 中原来的画刷对象，并删除临时对象                     }                     if (loc_dwDO == 4) return TRUE;                     //如果 dwDrawingOption 等于 4，则只着色，不进一步绘画背面                     //掉入下一分支，继续                  case 5:                     hObjectToDraw = hPlainBack;                     loc_dwROP = SRCAND;                     break;                  case 6:                     hObjectToDraw = hCrossBack;                     loc_dwROP = SRCCOPY;                     break;                  case 7:                     hObjectToDraw = hCircleBack;                     loc_dwROP = SRCCOPY;              }          }             if (hObjectToDraw == NULL || (loc_hCompDC = CreateCompatibleDC (hDC)) == NULL)              return FALSE;          if ((hObjectToDraw = SelectObject (loc_hCompDC, hObjectToDraw)) == NULL)          {              DeleteDC (loc_hCompDC);              return TRUE;   //这里明显意味着绘画失败，为什么会返回 TRUE 呢？          }          loc_crBkColor = SetBkColor (hDC, loc_crBkColor);  //设置 hDC 背景色          if (loc_bSharpCorner == FALSE)              SaveCorners (hDC, loc_crCornerData, nXDest, nYDest,                                                nActualWidth, nActualHeight);          //画卡片          if (nActualWidth == nNormalWidth && nActualHeight == nNormalHeight)          {              BitBlt (hDC, nXDest, nYDest, nNormalWidth, nNormalHeight,                                                         loc_hCompDC, 0, 0, loc_dwROP);          }else{              StretchBlt (hDC, nXDest, nYDest, nActualWidth, nActualHeight,                               loc_hCompDC, 0, 0, nNormalWidth, nNormalHeight, loc_dwROP);          }          SelectObject (loc_hCompDC, hObjectToDraw);  //恢复 loc_hCompDC 中原对象             if (loc_dwDO == 0)          {              loc_dwA = (nIndexOfCard / 4) % 13 + (nIndexOfCard % 4) * 13 + 1;              if (loc_dwA >= 14 && loc_dwA <= 23 || loc_dwA >= 27 && loc_dwA <= 36)              {              //如果要绘画的卡片是红心 A 至 10 或方块 A 至 10，则给它们画上黑色边框。              //注意在资源中看到的这几张牌原本是带红色边框的                   PatBlt (hDC, nXDest + 2, nYDest, nActualWidth - 4, 1, BLACKNESS);                   PatBlt (hDC, nXDest + nActualWidth - 1, nYDest + 2,                                                    1, nActualHeight - 4, BLACKNESS);                   PatBlt (hDC, nXDest + 2, nYDest + nActualHeight - 1,                                                     nActualWidth - 4, 1, BLACKNESS);                   PatBlt (hDC, nXDest, nYDest + 2, 1, nActualHeight - 4, BLACKNESS);                   SetPixel (hDC, nXDest + 1, nYDest + 1, 0x0);                   SetPixel (hDC, nXDest + nActualWidth - 2, nYDest + 1, 0x0);                   SetPixel (hDC, nXDest + nActualWidth - 2,                                              nYDest + nActualHeight - 2, 0x0);                   SetPixel (hDC, nXDest + 1, nYDest + nActualHeight - 2, 0x0);              }          }          if (loc_bSharpCorner == FALSE)              RestoreCorners (hDC, loc_crCornerData, nXDest, nYDest,                                                nActualWidth, nActualHeight);          SetBkColor (hDC, loc_crBkColor);  //恢复 hDC 原来的背景色          DeleteDC (loc_hCompDC);          return TRUE;     }`
> 
> * * *
> 
> 其中调用了 4 个子程序，SaveCorners、RestoreCorners、FLoadBack 和\_LoadCard，它们的代码如下：  
>   
>   
> 代码:
> 
> * * *
> 
> `  void __stdcall SaveCorners (HDC hDC,                        LPCOLORREF lpcrCornerData,                       int nXLeft,                       int nYLeft,                       int nWidth,                       int nHeight)        {          if (nWidth == nNormalWidth && nHeight == nNormalHeight)          //对于正常尺寸的卡片，存储其四个角的颜色数据          {              lpcrCornerData[0] = GetPixel (hDC, nXLeft, nYLeft);              lpcrCornerData[1] = GetPixel (hDC, nXLeft + 1, nYLeft);              lpcrCornerData[2] = GetPixel (hDC, nXLeft, nYLeft + 1);    //左上角              lpcrCornerData[3] = GetPixel (hDC, nXLeft + nWidth - 1, nYLeft);              lpcrCornerData[4] = GetPixel (hDC, nXLeft + nWidth - 2, nYLeft);              lpcrCornerData[5] = GetPixel (hDC, nXLeft + nWidth - 1, nYLeft + 1);   //右上角              lpcrCornerData[6] = GetPixel (hDC, nXLeft + nWidth - 1, nYLeft + nHeight - 1);              lpcrCornerData[7] = GetPixel (hDC, nXLeft + nWidth - 1, nYLeft + nHeight - 2);              lpcrCornerData[8] = GetPixel (hDC, nXLeft + nWidth - 2,                                                              nYLeft + nHeight - 1);  //右下角              lpcrCornerData[9] = GetPixel (hDC, nXLeft, nYLeft + nHeight - 1);              lpcrCornerData[10] = GetPixel (hDC, nXLeft + 1, nYLeft + nHeight - 1);              lpcrCornerData[11] = GetPixel (hDC, nXLeft, nYLeft + nHeight - 2);   //左下角          }          return;     }        void __stdcall RestoreCorners (HDC hDC,                           LPCOLORREF lpcrCornerData,                          int nXLeft,                          int nYLeft,                          int nWidth,                          int nHeight)        {              if (nWidth == nNormalWidth && nHeight == nNormalHeight)          //将由 SaveCorners 所存储的颜色数据恢复到卡片四角          {              SetPixel (hDC, nXLeft, nYLeft, lpcrCornerData[0]);              SetPixel (hDC, nXLeft + 1, nYLeft, lpcrCornerData[1]);              SetPixel (hDC, nXLeft, nYLeft + 1, lpcrCornerData[2]);              SetPixel (hDC, nXLeft + nWidth - 1, nYLeft, lpcrCornerData[3]);              SetPixel (hDC, nXLeft + nWidth - 2, nYLeft, lpcrCornerData[4]);              SetPixel (hDC, nXLeft + nWidth - 1, nYLeft + 1, lpcrCornerData[5]);              SetPixel (hDC, nXLeft + nWidth - 1, nYLeft + nHeight - 1, lpcrCornerData[6]);              SetPixel (hDC, nXLeft + nWidth - 1, nYLeft + nHeight - 2, lpcrCornerData[7]);              SetPixel (hDC, nXLeft + nWidth - 2, nYLeft + nHeight - 1, lpcrCornerData[8]);              SetPixel (hDC, nXLeft, nYLeft + nHeight - 1, lpcrCornerData[9]);              SetPixel (hDC, nXLeft + 1, nYLeft + nHeight - 1, lpcrCornerData[10]);              SetPixel (hDC, nXLeft, nYLeft + nHeight - 2, lpcrCornerData[11]);         }         return;     }        BOOL __stdcall FLoadBack (int nIndexOfCard)     {         static int nIndexOfLastCard;  //上次调用本函数时所要求的卡片序号                  if (nIndexOfCard != nIndexOfLastCard)         {              MyDeleteHbm (hCustomBack);              hCustomBack = LoadBitmap (hDllModule, LOWORD(nIndexOfCard));              nIndexOfLastCard = (hCustomBack == NULL? 0: nIndexOfCard);         }         return (nIndexOfLastCard != 0);     }        HBITMAP __fastcall _LoadCard (int nIndexOfCard)  //参数通过 eax 传递     {         static int nNumOfCardsLoaded;    //已装入的卡片位图数目         static int nSearchIndex;         //这个变量仅作为某种索引使用，无特别含义         int loc_nTransId;          //为方便而设置的中间变量，等于所需的位图资源 ID            if (hCards[nIndexOfCard] == NULL)         {              if (nNumOfCardsLoaded >= 5)     //已装入的卡片位图太多              {                          while (hCards[nSearchIndex] == NULL)  //查找下一个不为空的卡片位图                    {                          nSearchIndex = (nSearchIndex == 51? 0: nSearchIndex + 1);                  }                  DeleteObject (hCards[nSearchIndex]);                  --nNumOfCardsLoaded;       //并删除之                  hCards[nSearchIndex] = NULL;              }              //将卡片序号转换成位图资源 ID              loc_nTransId = (nIndexOfCard / 4) % 13 + (nIndexOfCard % 4) * 13 + 1;              while ((hCards[nIndexOfCard] = LoadBitmap (hDllModule,                                                LOWORD(loc_nTransId))) == NULL)              {                  if (nNumOfCardsLoaded == 0) return NULL;                  //如果删完了所有已加载的位图，还是无法装入所要求的位图，则返回 NULL                  while (hCards[nSearchIndex] == NULL)                    {                          nSearchIndex = (nSearchIndex == 51? 0: nSearchIndex + 1);                  }                  DeleteObject (hCards[nSearchIndex]);                  hCards[nSearchIndex] = NULL;                  --nNumOfCardsLoaded;                             }              ++nNumOfCardsLoaded;         }         return hCards[nIndexOfCard];     }`
> 
> * * *
> 
> 　　这样，cdtDrawExt 函数就分析完了。而另一个导出函数 cdtDraw，只不过是到 cdtDrawExt 的一个重  
> 定向而已：  
>   
>   
> 代码:
> 
> * * *
> 
> `  BOOL __stdcall cdtDraw (HDC hDC,                              int nXDest,                              int nYDest,                              int nIndexOfCard,                              DWORD dwDrawingOption,                             COLORREF crBackColor                            )     {         return cdtDrawExt (hDC, nXDest, nYDest, nNormalWidth, nNormalHeight,                             nIndexOfCard, dwDrawingOption, crBackColor);     }`
> 
> * * *
> 
>   
>   
>   
> 2. 关于使用 cards.dll 的一些讨论  
>   
> 　　从上面的分析可以看出：  
>   
> 　　(1) 由于画卡片的关键步骤要引用变量 nNormalWidth 和 nNormalHeight，在正式使用 cdtDrawExt 函  
> 数之前应该调用 cdtInit 以设置它们的正确值，否则它们的值是零，画卡片的函数就无法正常工作。与  
> 此相对的，程序结束时应该调用 cdtTerm 来清除资源，如果由于建立多个线程之类的原因而多次调用了  
> cdtInit，那么也必须调用同样多次的 cdtTerm，否则资源不会被真正释放。  
>   
> 　　(2) cdtDrawExt 的主要功能可以用一句话概括，即在 hDC 指定的设备环境中，以坐标为(nXDest,   
> nYDest)的点为左上角，nActualWidth 和 nActualHeight 为卡片的宽和长，绘画由 nIndexOfCard 指定的卡  
> 片。具体的绘画方式可以有多种，由 dwDrawingOption 来决定。以下我们把 dwDrawingOption 的低 31 位等  
> 于几的函数调用，简称为第几号功能。  
>   
> 　　(3) 绘画卡片正面常用的是第 0 和第 2 号功能，由这两个功能分支所对应的 ROP 码可以看出，0 号功能  
> 是原样绘制卡片（除将红心 A 至 10、方块 A 至 10 绘画为黑色边框外），而 2 号功能是反色绘制卡片将  
> 资源中位图各像素取反后复制过去。然而这两种 ROP 模式毕竟都只与源像素有关，没有用到 crBackColor  
> 提供的背景颜色，不免过于单调。如果能把这里的 ROP 码数据改掉，还能画出更多种不同颜色的卡片。  
> 这一点可以通过应用程序自己的代码来完成，兹不赘述。  
>   
> 　　(4) 在这两个功能所调用的\_LoadCard 子过程中，首先将卡片序号 nIndexOfCard 经过下式转换：  
> 代码:
> 
> * * *
> 
> `         nTransId = (nIndexOfCard / 4) % 13 + (nIndexOfCard % 4) * 13 + 1;`
> 
> * * *
> 
> 才将 nTransId 解释为位图资源 ID。这使我们观察到如下特点：当 nIndexOfCard 从 0 到 51 依次递变时，正  
> 好遍历一副扑克（不含大小 Joker）的 52 张牌，nIndexOfCard 的低 2 位代表该牌的花色，从 0 至 3 依次表示  
> 草花、红心、方块、黑桃；高 30 位的值加上 1 就是这张牌的点数。例如：问序号是 27 的牌是哪一张？因  
> 为 27 % 4 == 3，27 / 4 + 1 = 7，所以这张牌是黑桃 7。注意到这一点，对编写程序是有帮助的。  
>   
> 　　(5) 第 1 号功能从表面上看是用来绘画卡片背面（调用“FLoadBack”函数），但实际也能用来绘画  
> 卡片的正面。由 FLoadBack 函数的代码可见，它并不象\_LoadCard 那样先将 nIndexOfCard 转换后解释成位  
> 图 ID，而是直接将其解释成位图 ID。因此要用 1 号功能遍历一副牌，所提供的 nIndexOfCard 范围就不是  
> 从 0 到 51，而是从 1 到 52 了；并且牌的对应关系也有所不同。  
>   
> 　　(6) 4 号功能并不绘画卡片，而是用 crBackColor 颜色填充一个卡片大小的矩形。5 号功能以 SRCAND  
> 方式绘画一个平实背景。3 号功能是上述二者的组合。6 号和 7 号功能分别画一个叉或一个圆圈。  
>   
> 　　(7) dwDrawingOption 的最高位可以被置位或复位，以决定是否调用 SaveCorners/RestoreCorners  
> 来保存卡片四角的颜色数据。这样一来，当 dwDrawingOption 最高位被复位时，画出来的卡片将是圆角  
> 的。但从 SaveCorners/RestoreCorners 的内部逻辑来看，如果卡片的实际尺寸不等于正常尺寸，则无论  
> dwDrawingOption 最高位是否置位，所绘制的卡片都是尖角的。  
>   
> 　　(8) 关于寄存器保护的问题。这个问题在汇编编程中很关键。几乎所有的 Windows API 函数都会保  
> 持 ebx，esi，edi 和 ebp 寄存器，我们自然期望 cards.dll 的导出函数也如此。察看反汇编文本就能发  
> 现，事实正是如此。  
>   
>   
> 3. （附录）逆向活动的一些心得体会  
>   
> 　　要把编译器生成的汇编代码翻译回高级语言，首先要对目的高级语言的各种运算和操作相当熟悉。  
> 作为 C 语言而言，常用的运算有加减乘除、位运算、指针操作和取地址等等。位运算和指针是 C 语言与汇  
> 编的共通之处，汇编代码中出现这类操作，几乎可以毫不费力地找到它们在 C 语言中的等价物。但也有  
> 一些例外的情况，如移位操作具有部分乘除法的功能，常与加法操作配合来完成部分简单的乘除法，特  
> 别是在移位指令比乘除指令快的 80x86 平台上，优化编译器常常生成移位指令来完成特定的乘除法，这  
> 一来，移位指令究竟是翻译成移位操作还是乘除操作，就要根据操作数的含义而定，不能一概处理。与  
> 此相仿，汇编代码中出现 lea 指令时，也不能认为一定是在取地址，有可能只是在做简单的加法操作。  
>   
> 　　高级语言有着一整套的流程控制结构，如条件、分支、循环等等，这一切在汇编语言中只能用转移  
> 指令和条件转移指令来实现，因此凡是出现这些指令的地方，就要特别留意分支的条件是什么，各个分  
> 支（或循环体）到哪里结束。譬如：  
>   
> 代码:
> 
> * * *
> 
> `      mov eax, A         cmp eax, B         jne @F                     ;do something       @@:`
> 
> * * *
> 
> 条件转移指令是 jne 或“不等则跳”，那么如果执行到 do something 的地方，那一定意味着 A 与 B 应该相  
> 等了，因此翻译回 C 语言的形式就是：  
>   
> 代码:
> 
> * * *
> 
> `      if (A == B){             //do something         }`
> 
> * * *
> 
> 简而言之，当你碰到一个条件转移指令的时候，随手写下一个 if，在 if 的括号中写下与此条件转移指令  
> 所示条件相反的条件（很简单：既然是特定条件造成转移，相反的条件自然意味着没有转移而顺序执行  
> 下去！），这应当是一个好习惯。当然，有些情况下实际代码不一定是 if 分支而是 while 循环，但只要  
> 识别出循环，把 if 改 while 也只不过是举手之劳。  
>   
> 　　带有 else 从句的 if 分支则可能实现如下：  
>   
> 代码:
> 
> * * *
> 
> `　　  mov eax, A         cmp eax, B         jne @F                     ;do something         jmp _next   ;注意这里多了一条 jmp 指令       @@:                     ;do else-thing    _next:`
> 
> * * *
> 
> 至于 switch...case 分支结构，当各 case 子句的控制表达式在数值上连续时，可能用跳转表的方式实  
> 现。否则一般以 if 分支的某种扩展形式实现。  
>   
> 　　循环结构的一个明显特征是出现从后往前跳的转移指令，但这不是本质特征。只有跟踪某条流程到  
> 一定时候发现重复执行了某块代码，才能大体判断出这是一个循环结构。  
>   
> 　　甚至有些地方看上去没有分支，而实际上有分支。例如：  
>   
> 代码:
> 
> * * *
> 
> `      mov eax, A         mov edx, C         neg eax         sbb eax, eax         and eax, edx         mov B, eax`
> 
> * * *
> 
> 这本质上就是：  
>   
> 代码:
> 
> * * *
> 
> `      B = (A == 0? 0: C);`
> 
> * * *
> 
> 这些实际上是前人在进行汇编编程时创造出来的技巧，现在被应用在了编译器中。对于这种技巧，除非  
> 自己也多进行汇编编程而熟悉它们，否则只有硬性分析了。  
>   
> 　　C 语言是一种表达式语言，它允许在分支或循环的判断条件中使用非常复杂的表达式。出现这种情  
> 况时就更应当加以细致分析。例如：  
>   
> 代码:
> 
> * * *
> 
> `      mov eax, B         test eax, eax         mov A, eax         jnz @F              ;do something       @@:`
> 
> * * *
> 
> 取出 B 的值判断是否为 0，但在判断结果出来之前却先将其赋给 A，只可能是：  
>   
> 代码:
> 
> * * *
> 
> `　　if ((A = B) == 0){           //do something       }`
> 
> * * *
> 
> 多个逻辑表达式用逻辑运算符组成的复合表达式也可以很复杂，在很短的代码中出现大量的条件转移指  
> 令。这时，记住 C 语言中求值逻辑表达式时所遵循的“短路求值规则”，对分析是有帮助的。  
>   
> 　　多数编译器在将函数参数入栈时都生成 push 指令，但也有些编译器例外，如 MinGW GCC 编译器等。  
> 带有某些特殊声明（非 C 语言标准所规定）的函数，可能不是用栈而是用其他方式来传递参数，例如  
> \_\_fastcall 声明采用寄存器传递参数。在这些情况下，call 指令之前就不会有任何的 push 指令。要辨别  
> \_\_fastcall 声明，主要看函数的入口附近有没有引用一个未经赋值的寄存器的指令，如果有，大体就是  
> 这种声明了。在函数返回时，C 调用约定不会在 ret 指令后带任何操作数，凡是带了的都是 stdcall 或其  
> 他调用约定。  
>   
> 　　要特别重视 ebx，esi，edi 寄存器当然这是就 Windows 系统上运行的程序说的因为基本上所  
> 有的 Windows API 都会保持这 3 个寄存器，换言之，只要应用程序自己的代码不去修改它们，它们的值是  
> 不会改变的。现代的优化编译器往往把一些频繁被引用的对象放在寄存器中，而由于这三个寄存器内容  
> 的恒定性，往往就把这些对象放在这三个寄存器中。因此从一个局部上来看，时常会出现没有给这三个  
> 寄存器赋值就使用它们的值的情况，但实际上不是没有赋值，而是在很远以前（如子程序开头）曾经赋  
> 过值，只是它们的值一直没有被改动而已。这就要求每当这三个寄存器被赋值时，要记住所赋予的值是  
> 什么，因为它们可能在往后的代码中随时被引用。  
>   
> 　　在汇编语言中，定义在数据段中的变量就是全局变量，定义在栈帧上的变量就是自动变量。但在 C  
> 语言中则不尽然，栈帧上的变量固然是自动变量，数据段中的变量却未必是全局变量，它还可能是某个  
> 子程序中的静态变量。在反编译中如果发现数据段中某个变量只在单个子程序中被引用，那么本着尽量  
> 缩小变量作用域的原则，最好把该变量声明为所属子程序的静态变量。  
>   
> 　　关于寄存器的可消去性这是一条粗略的原则，即汇编代码中出现的几乎所有的寄存器到了还原  
> 成高级语言时都可以被消去。这条原则理论上是基于高级语言与机器体系的无关性，高级语言中不可能  
> 有寄存器这样的概念。只有变量的概念，而变量是一种存储单元的抽象。假如我们能够在机器级别同时  
> 操作两个变量，也就是直接对两个存储器操作数进行操作，而且这比操作寄存器还来得快，显然我们就  
> 没有理由去使用寄存器了。然而在 80x86 上这些都并非事实，所以编译器才会生成使用寄存器的代码。  
> 汇编代码中所出现的寄存器，其作用无非有二：一是存储某种源对象以便取用，二是存储某些运算的中  
> 间结果。但在第一种情形这个寄存器可以被源对象本身所取代，第二种情形下则可以被产生这个中间结  
> 果的表达式所取代。因此对于寄存器应当尽量消去，不宜图省事而随意创造一些变量来作为寄存器的替  
> 代物。一个简单的例子有如：  
>   
> 代码:
> 
> * * *
> 
> `　　　　mov eax, A           mov ebx, B           add eax, ebx           mov A, eax`
> 
> * * *
> 
> 你总不至于极端到把它写成下面这样的语句吧：  
>   
> 代码:
> 
> * * *
> 
> `        regEax = A;           regEbx = B;           regEax += regEbx;           A = regEax;`
> 
> * * *
> 
> 实际上，eax 是保存中间结果的，而 ebx 就是源对象 B。因此：  
>   
> 代码:
> 
> * * *
> 
> `        A += B;`
> 
> * * *
> 
> 就完全搞定。除非源代码中的变量是临时声明的（如用于循环计数），编译器并不给它分配存储单元而  
> 直接放在寄存器中。这个变量是一个实体，但没有对应的存储单元，一生在寄存器中度过。在这种情况  
> 下，才能把寄存器看成实体而还原出相应变量，除此之外，寄存器不能看成实体。  
>   
> 　　但理论与现实总是存在差距，如果完全消去寄存器会造成很冗长很复杂的表达式，则可以引入一个  
> 或多个中间变量来降低表达式的复杂程度。另外一种情况是象下面这样：  
>   
> 代码:
> 
> * * *
> 
> `        push A           call func_1  ;假设返回值在 eax 中           test eax, eax           jz   @F           push eax           call func_2        @@:`
> 
> * * *
> 
> func\_1 (A)的返回值先经过判断，不等于零才作为参数调用 func\_2。按照前述原则，应当将 func\_1 的返  
> 回值 eax 用表达式 func\_1 (A)本身来替代。但 func\_1 只能调用一次，而这个表达式却必须在两个地方出  
> 现（一次是判断，一次是作为 func\_2 的参数）。如果不创建一个临时对象来代表 eax，我是想不出这个  
> 问题该怎么解决。  
>   
> 　　上传的附件：origdasm.asm 原始反汇编文本  
> 　　　　　　　　annodasm.asm 经分析和注释的反汇编文本 

 4dd466d0e1e0cc123afbe504da2462ca