
```c
#include<windows.h>
#include<iostream>
#include<string.h>
using namespace std;

int main()
{
  HANDLE hfile=CreateFile("c:\\\\fst\\_1216\\_a1\\_cmn.mes",GENERIC\\_READ,FILE\\_SHARE\\_READ,NULL,OPEN\\_EXISTING,FILE\\_ATTRIBUTE\\_ARCHIVE,NULL);
  if(hfile==INVALID\\_HANDLE\\_VALUE)
  {
    cout<<"Can not open file!"<<endl;
    return 0;
  }
  DWORD size=GetFileSize(hfile,NULL);
  DWORD len;
  BYTE \\*buff=new BYTE \\[size\\];
  if(!ReadFile(hfile,(LPVOID)buff,size,&len,NULL))
  {
    cout<<"Can not read file!"<<endl;
    CloseHandle(hfile);
    return 0;
  }
  CloseHandle(hfile);
  int j=0,nu=0;
  hfile=CreateFile("d:\\\\a1\\_cmn.txt",GENERIC\\_READ,FILE\\_SHARE\\_READ,NULL,OPEN\\_ALWAYS,FILE\\_ATTRIBUTE\\_ARCHIVE,NULL);
  if(hfile==INVALID\\_HANDLE\\_VALUE)
  {
    cout<<"Can not create file!"<<endl;
    return 0;
  }
  DWORD textsize=GetFileSize(hfile,NULL);
  BYTE \\*temp=new BYTE \\[textsize\\];

  if(!ReadFile(hfile,(LPVOID)temp,textsize,&len,NULL))
  {
    cout<<"Can not read file!"<<endl;
    CloseHandle(hfile);
    return 0;
  }
  CloseHandle(hfile);
  hfile=CreateFile("d:\\\\fst\\_1216\\_a1\\_cmn.mes",GENERIC\\_WRITE|GENERIC\\_READ,FILE\\_SHARE\\_READ,NULL,OPEN\\_ALWAYS,FILE\\_ATTRIBUTE\\_ARCHIVE,NULL);
  if(hfile==INVALID\\_HANDLE\\_VALUE)
  {
    cout<<"Can not create new script file!"<<endl;
    return 0;
  }  
  for(int i=0;i<size;i++)
  {
    if(buff\\[i\\]>=0x4a&&buff\\[i\\]<0x4e)
    {
      while(buff\\[i++\\]){};
      while(temp\\[j\\]!=0x0d)
      {
        temp\\[j\\]-=0x20;
        WriteFile(hfile,(LPVOID)&temp\\[j++\\],1,&len,NULL);
      }
      WriteFile(hfile,(LPVOID)&nu,1,&len,NULL);
      WriteFile(hfile,(LPVOID)&buff\\[i\\],1,&len,NULL);
      j++;
    }
    else
    {
      WriteFile(hfile,(LPVOID)&buff\\[i\\],1,&len,NULL);
    }
  }
  delete \\[\\] temp;
  delete \\[\\] buff;
  SetFilePointer(hfile,0,NULL,FILE\\_BEGIN);
  size=GetFileSize(hfile,NULL);
  buff=new BYTE \\[size\\];
  if(!ReadFile(hfile,(LPVOID)buff,size,&len,NULL))
  {
    cout<<"Can not read file!"<<endl;
    CloseHandle(hfile);
    return 0;
  }
  DWORD \\*offsetarray=new DWORD \\[\\*(DWORD\\*)&buff\\[0\\]\\];
  DWORD sizeofheader=(\\*(DWORD\\*)&buff\\[0\\])\\*4+4;
  for(j=0,i=0;i<size;i++)
  {
    if(buff\\[i\\]>=0x4a&&buff\\[i\\]<0x4e)
    {
      offsetarray\\[j\\]=i-sizeofheader;
      while(buff\\[i++\\]);
      j++;
    }
  }
  SetFilePointer(hfile,4,NULL,FILE\\_BEGIN);
  WriteFile(hfile,(LPVOID)offsetarray,sizeofheader-4,&len,NULL);
  delete \\[\\] offsetarray;
  CloseHandle(hfile);
  return 0;
}
```

简单说下算法，前面的解包器是将脚本中字节的值大于等于 `0x4a` 并小于 `0x4e` 的值后的字符串提取出来然后逐字节加 `0x20`，而封包器就是打开原来的脚本文件后再将翻译好的文字减 `0x20` 替换到原来的加密的字符串的位置。由于这个游戏是用 `0` 来判断字符串结束位置的，所以像偏移地址啊字符串长度这些也不用管了。

## 四、程序修改

当我满心欢喜将新的脚本文件替换掉原来的文件之后，打开游戏，却出现了下面的画面：

![](img/89684c6fd78ea9fa0a220540da2de9c9.jpg)

干！

仔细想一下，由于原来的文本是用日文编码的，现在换成了 `GBK` 编码，不乱码就有鬼了，于是我们就要修正程序。

首先，用 `OD` 载入程序，右键选“查找当前模块中的所有名称”，看到有 `CreateFontA` 这个函数，查一下 `MSDN`，发现这个函数的第九个参数 `fdwCharSet` 正是决定字符编码的参数，在程序里面看一下，有几个连续的 `CreateFontA` 的调用：

> 00408D7A  |>  \68 500C5000   push    00500C50                 ; /FaceName = ""82,"l",82,"r",83,"S",83,"V",83,"b",83,"N"
> 00408D7F  |.  6A 04          push    4                     ; |PitchAndFamily = DEFAULT\_PITCH|4|FF\_DONTCARE
> 00408D81  |.  6A 00          push    0                     ; |Quality = DEFAULT\_QUALITY
> 00408D83  |.  6A 20          push    20                    ; |ClipPrecision = CLIP\_DEFAULT\_PRECIS|CLIP\_TT\_ALWAYS
> 00408D85  |.  6A 04          push    4                     ; |OutputPrecision = OUT\_TT\_PRECIS
> 00408D87  |.  68 86000000   push    80                    ; |CharSet = 128.
> 00408D8C  |.  8B35 2C304300 mov     esi, dword ptr [<&GDI32.CreateFo>; |GDI32.CreateFontA
> 00408D92  |.  6A 00          push    0                     ; |StrikeOut = FALSE
> 00408D94  |.  6A 00          push    0                     ; |Underline = FALSE
> 00408D96  |.  6A 00          push    0                     ; |Italic = FALSE
> 00408D98  |.  6A 00          push    0                     ; |Weight = FW\_DONTCARE
> 00408D9A  |.  6A 00          push    0                     ; |Orientation = 0
> 00408D9C  |.  6A 00          push    0                     ; |Escapement = 0
> 00408D9E  |.  6A 10          push    10                    ; |Width = 10 (16.)
> 00408DA0  |.  6A 20          push    20                    ; |Height = 20 (32.)
> 00408DA2  |.  FFD6          call    esi                  ; \CreateFontA

而 `0x80` 则是 `SHIFTJIS_CHARSET` 编码的值，在 `windows.h` 头文件中我们可以查到 `GB2312_CHARSET` 的值为 `0x86`。于是我们可以将那个 `0x80` 改成是 `0x86`，用 `OD` 保存，然后运行……

还是没有显示出中文来！

现在该怎么办？这时应该看看程序利用哪些函数来进行输出的。用右键选“查看”-》“当前模块的所有参考”可以看到有个叫 `GetGlyphOutlineA` 的函数。在 `MSDN` 中有明确说明这个函数可以将字符转换成点阵来输出的。往上看，可以看到一段奇怪的代码：

> 00406765  |.  8A06          mov     al, byte ptr [esi]
> 00406767  |.  3C 80         cmp     al, 80
> 00406769  |.  76 04         jbe     short 0040676F
> 0040676B  |.  3C A0         cmp     al, 0A0
> 0040676D  |.  72 08         jb      short 00406777
> 0040676F  |>  3C E0         cmp     al, 0E0
> 00406771  |.  72 5D         jb      short 004067D0
> 00406773  |.  3C FC         cmp     al, 0FC
> 00406775  |.  77 59         ja      short 004067D0

这段代码的作用是为了检查字符编码的边界而防止产生异常而导致不可预料的效果的。而由于 `GBK` 编码比游戏原来的编码的范围要大（`CBK` 编码的范围是 `0x80xx` 到 `0xfexx`），所以将比较处的 `0A0` 跟 `0FC` 替换成 `0FE`，保存。运行，效果如下：

![](img/b46a8e0df82c8e3ce217203a8f2510ef.jpg)

（关于编码边界检查的更详细内容可以参考这个链接 [`bbs.sumisora.com/read.php?tid=211852`](http://bbs.sumisora.com/read.php?tid=211852)）

就这样，这个游戏的汉化算是大功告成- -

`PS.` 为什么只能算是大功告成，因为我写的封包器似乎有点问题，程序的修改方面也出现一些问题（到后面几句翻译的对白会出现错别字- -|||）最重要的一点是，整个游戏的文本有这么多，我就翻了开头的几句，这怎么能算是汉化完成啊

`PS` 的 `PS.` 经某大牛指点，`mes` 文件第一个双字是脚本里面文本的个数，接下来的一个数组是文本的偏移位置。虽然据我观察程序并不对这段进行读取……现在将封包程序改一下。另外在调用 `GetGlyphOutlineA` 前有两次边界检查，如无意外，第一次应该也要改的 orz
