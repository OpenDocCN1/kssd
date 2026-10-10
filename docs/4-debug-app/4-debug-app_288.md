

# 超级扫雷机制作（windowsxp 下扫雷游戏）

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-329.htm`](https://www.pediy.com/kssd/pediy07/pediy7-329.htm)

超级扫雷机制作（windowsxp 下扫雷游戏）  
\[作者\]peansen  
\[工具\]破解常用工具  
\[目标\]做出一个超级扫雷机  
  
在闲暇之余，拿出扫雷游戏，正想着怎么玩好，于是就有了做一个扫雷机的想法。  
要想做出一个万能扫雷机必须知道这么多格子里面哪个有雷哪个没有。我是这样进行分析的：  
这个程序肯定是没有壳的，呵呵。  
1.察看扫雷游戏的界面，发现只有一个 Windows，也就是说这些格子都是通过鼠标的点击消息获知的  
2.应该是在 paint 里面画出网格及各网格的状态  
3.一定有一个函数来产生地雷在格子中的位置，这个函数和随机数有关  
4.每次点击一定会访问已经放好地雷位置的数据  
  
现在从随机函数入手，我们察看输入表，很容易找到两个调用它的地址 0x10036cd 和 0x10036db，我们很容易的来到这里  
0100367A   /$  A1 AC560001   mov eax,dword ptr ds:\[10056AC\]  
0100367F   |.  8B0D A8560001 mov ecx,dword ptr ds:\[10056A8\]  
01003685   |.  53            push ebx  
01003686   |.  56            push esi  
01003687   |.  57            push edi  
01003688   |.  33FF          xor edi,edi  
0100368A   |.  3B05 34530001 cmp eax,dword ptr ds:\[1005334\]  
01003690   |.  893D 64510001 mov dword ptr ds:\[1005164\],edi  
01003696   |.  75 0C         jnz short winmine.010036A4  
01003698   |.  3B0D 38530001 cmp ecx,dword ptr ds:\[1005338\]  
0100369E   |.  75 04         jnz short winmine.010036A4  
010036A0   |.  6A 04         push 4  
010036A2   |.  EB 02         jmp short winmine.010036A6  
010036A4   |>  6A 06         push 6  
010036A6   |>  5B            pop ebx  
010036A7   |.  A3 34530001   mov dword ptr ds:\[1005334\],eax 通过对照扫雷的格子很容易知道这里存放 x 方向上的格子数  
010036AC   |.  890D 38530001 mov dword ptr ds:\[1005338\],ecx 这个存放的是 y  
010036B2   |.  E8 1EF8FFFF   call winmine.01002ED5 这个函数里对数据作了一些初始化，下面会提到  
010036B7   |.  A1 A4560001   mov eax,dword ptr ds:\[10056A4\]  
010036BC   |.  893D 60510001 mov dword ptr ds:\[1005160\],edi  
010036C2   |.  A3 30530001   mov dword ptr ds:\[1005330\],eax  
010036C7   |>  FF35 34530001 push dword ptr ds:\[1005334\]  
010036CD   |.  E8 6E020000   call winmine.01003940（就是这个产生随机数）这个产生随机数会有一个参数，就是产生这个数以下的随机数  
010036D2   |.  FF35 38530001 push dword ptr ds:\[1005338\]  
010036D8   |.  8BF0          mov esi,eax  
010036DA   |.  46            inc esi 产生了随机数以后都加 1  
010036DB   |.  E8 60020000   call winmine.01003940（就是这个产生随机数）  
010036E0   |.  40            inc eax 这边也加 1，注意这边都加了一，所以在后面写扫雷机时要注意这个  
010036E1   |.  8BC8          mov ecx,eax  
010036E3   |.  C1E1 05       shl ecx,5//说明数据的第一维是 32  
010036E6   |.  F68431 405300>test byte ptr ds:\[ecx+esi+1005340\],80//看看是否已经有雷了  
010036EE   |.^ 75 D7         jnz short winmine.010036C7//如果已经由雷了，那么这次随机产生的雷不算，跳回去再来一次  
010036F0   |.  C1E0 05       shl eax,5  
010036F3   |.  8D8430 405300>lea eax,dword ptr ds:\[eax+esi+1005340\]  
010036FA   |.  8008 80       or byte ptr ds:\[eax\],80//最高位置一表示该处有雷了  
010036FD   |.  FF0D 30530001 dec dword ptr ds:\[1005330\]//要产生这么多雷  
01003703   |.^ 75 C2         jnz short winmine.010036C7  
01003705   |.  8B0D 38530001 mov ecx,dword ptr ds:\[1005338\]  
0100370B   |.  0FAF0D 345300>imul ecx,dword ptr ds:\[1005334\]  
01003712   |.  A1 A4560001   mov eax,dword ptr ds:\[10056A4\]  
01003717   |.  2BC8          sub ecx,eax  
01003719   |.  57            push edi  
0100371A   |.  893D 9C570001 mov dword ptr ds:\[100579C\],edi  
01003720   |.  A3 30530001   mov dword ptr ds:\[1005330\],eax  
01003725   |.  A3 94510001   mov dword ptr ds:\[1005194\],eax  
0100372A   |.  893D A4570001 mov dword ptr ds:\[10057A4\],edi  
01003730   |.  890D A0570001 mov dword ptr ds:\[10057A0\],ecx  
01003736   |.  C705 00500001>mov dword ptr ds:\[1005000\],1  
01003740   |.  E8 25FDFFFF   call winmine.0100346A  
01003745   |.  53            push ebx                                ; /Arg1  
01003746   |.  E8 05E2FFFF   call winmine.01001950                   ; \\winmine.01001950  
0100374B   |.  5F            pop edi  
0100374C   |.  5E            pop esi  
0100374D   |.  5B            pop ebx  
0100374E   \\.  C3            retn  
  
初始化的一个函数，就是往各数据块里添加数据，这个我们暂时不要管太多，只要知道现在我们关心的  
01002ED5   /$  B8 60030000   mov eax,360  
01002EDA   |>  48            /dec eax  
01002EDB   |.  C680 40530001>|mov byte ptr ds:\[eax+1005340\],0F  
01002EE2   |.^ 75 F6         \\jnz short winmine.01002EDA 把从 0x1005340 开始的 0x360 个数据清为 0x0f  
01002EE4   |.  8B0D 34530001 mov ecx,dword ptr ds:\[1005334\]  
01002EEA   |.  8B15 38530001 mov edx,dword ptr ds:\[1005338\]  
01002EF0   |.  8D41 02       lea eax,dword ptr ds:\[ecx+2\]  
01002EF3   |.  85C0          test eax,eax  
01002EF5   |.  56            push esi  
01002EF6   |.  74 19         je short winmine.01002F11  
01002EF8   |.  8BF2          mov esi,edx  
01002EFA   |.  C1E6 05       shl esi,5  
01002EFD   |.  8DB6 60530001 lea esi,dword ptr ds:\[esi+1005360\]  
01002F03   |>  48            /dec eax  
01002F04   |.  C680 40530001>|mov byte ptr ds:\[eax+1005340\],10  
01002F0B   |.  C60406 10     |mov byte ptr ds:\[esi+eax\],10  
01002F0F   |.^ 75 F2         \\jnz short winmine.01002F03  
01002F11   |>  8D72 02       lea esi,dword ptr ds:\[edx+2\]  
01002F14   |.  85F6          test esi,esi  
01002F16   |.  74 21         je short winmine.01002F39  
01002F18   |.  8BC6          mov eax,esi  
01002F1A   |.  C1E0 05       shl eax,5  
01002F1D   |.  8D90 40530001 lea edx,dword ptr ds:\[eax+1005340\]  
01002F23   |.  8D8408 415300>lea eax,dword ptr ds:\[eax+ecx+1005341\]  
01002F2A   |>  83EA 20       /sub edx,20  
01002F2D   |.  83E8 20       |sub eax,20  
01002F30   |.  4E            |dec esi  
01002F31   |.  C602 10       |mov byte ptr ds:\[edx\],10  
01002F34   |.  C600 10       |mov byte ptr ds:\[eax\],10  
01002F37   |.^ 75 F1         \\jnz short winmine.01002F2A  
01002F39   |>  5E            pop esi  
01002F3A   \\.  C3            retn  
  
现在已经知道了雷的存放地址，长宽，地雷数目等的地址都已经知晓。每次 paint 时候都是根据这个（0x1005340）地址的状态来决定怎么绘制出图来。我们不关系其他，知道了这里每个字节的最高位为 1 时有雷就好了。  
为了能够制作扫雷机我们还要知道这些格子的大小，当然截图看方格大小是可行的，我们也可以在 0x1005340 地址随便设断，运行到这里  
010026DB   |> /68 2000CC00   |/push 0CC0020                          ; /ROP = SRCCOPY  
010026E0   |. |33C0          ||xor eax,eax                           ; |  
010026E2   |. |8A0433        ||mov al,byte ptr ds:\[ebx+esi\]          ; |  
010026E5   |. |6A 00         ||push 0                                ; |YSrc = 0  
010026E7   |. |6A 00         ||push 0                                ; |XSrc = 0  
010026E9   |. |83E0 1F       ||and eax,1F                            ; |  
010026EC   |. |FF3485 205A00>||push dword ptr ds:\[eax\*4+1005A20\]     ; |hSrcDC  
010026F3   |. |6A 10         ||push 10                               ; |Height = 10 (16.)  
010026F5   |. |6A 10         ||push 10                               ; |Width = 10 (16.)  
010026F7   |. |FF75 F8       ||push dword ptr ss:\[ebp-8\]             ; |YDest，中断的第一次这个值是 55  
010026FA   |. |FF75 FC       ||push dword ptr ss:\[ebp-4\]             ; |XDest，中断的第一次这个值是 12  
010026FD   |. |FF75 08       ||push dword ptr ss:\[ebp+8\]             ; |hDestDC  
01002700   |. |FF15 5C100001 ||call dword ptr ds:\[<&GDI32.BitBlt>\]   ; \\BitBlt  
  
想都不要想，方格的大小为 16 个像素，最走上角的位置是（12，55）  
好，现在所有的基本信息都有了，可以按照这些信息写扫雷机了  
核心代码如下：（注意前面的产生随机数以后加 1 的情形）  
  char flag\[32 \* 40\];  
  DWORD dwProcessId;  
  HWND hWnd = ::FindWindow("扫雷","扫雷");  
  if(hWnd == NULL)  
  {  
    MessageBox("没有找到扫雷程序","错误", MB\_ICONERROR);  
    return ;  
  }  
  GetWindowThreadProcessId(hWnd,&dwProcessId);  
  HANDLE hProcess = ::OpenProcess(PROCESS\_ALL\_ACCESS,FALSE,dwProcessId);  
  if(hProcess == NULL)  
  {  
    MessageBox("打开进程失败","错误", MB\_ICONERROR);  
    return ;  
  }  
  DWORD height, width, num;  
  ReadProcessMemory(hProcess, (LPVOID)0x1005334, (LPVOID)&width, 4, NULL);  
  ReadProcessMemory(hProcess, (LPVOID)0x1005338, (LPVOID)&height, 4, NULL);  
  ReadProcessMemory(hProcess, (LPVOID)0x1005330, (LPVOID)&num, 4, NULL);  
  ReadProcessMemory(hProcess, (LPVOID)0x1005360, (LPVOID)flag, 32 \* 40, NULL);//这里的址本来是 0x1005340，由于加一的原因就加了 0x20，因为宽度为 32  
  ::CloseHandle(hProcess);  
  for (unsigned int i = 0; i < height; i ++)  
  {  
    for (unsigned int j = 0; j < width; j ++)  
    {  
      if((flag\[i \* 32 + j + 1\] & 0x80) == 0)//这里也要注意加意  
      {  
        ::SendMessage(hWnd, WM\_LBUTTONDOWN, 0, 12 + 8 + j \* 16 + ((55 + 8 + i \* 16) << 16));//定位到每个格子的中间，发送鼠标消息 12---走上角 x 坐标 + 8---格子的一半 + j \* 16----格子的长度和这个格子的位置，纵坐标一样的方式考虑  
        ::SendMessage(hWnd, WM\_LBUTTONUP, 0, 12 + 8 + j \* 16 + ((55 + 8 + i \* 16) << 16));  
      }  
    }  
  }  
  CString str;  
  str.Format("宽: %d,高: %d\\n 地雷数目: %d", width, height, num);  
  MessageBox(str.GetBuffer(), "完成");  
我从我写的程序里拷下来的的这段，意义已经很明确了。呵呵  
至此扫雷机已经做完了，你可以在 1 秒内完成扫雷，其实你根本不需要一秒。我想这个扫雷记录是不会被刷新了。  
要下载程序爽爽可以这里（我的 blog）  
http://peansen.blog.edu.cn/user1/8184/archives/2005/251960.shtml 

 4dd466d0e1e0cc123afbe504da2462ca