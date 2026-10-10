

# 基于 openGL 引擎的游戏不能显示汉字分析实例

> 原文：[`www.pediy.com/kssd/pediy11/119559.html`](https://www.pediy.com/kssd/pediy11/119559.html)

前几天我汉化一个了基于 openGL 引擎游戏游戏，过程中出现游戏不显示汉字，所以无法本地化。  
  
首先，OD 加载主程序，CTRL+N 找和显示有关的函数（这个只能从字面上理解某个函数是不是和显示有关），既然这个游戏使用的是 OpenGL 编写的，那么我们就主要看带有 gl 前缀和导入函数。  
  
  
游戏导入的 OPENGL32 库中只有这么多的函数了。  
如果你看过 Nehe 写的 OpenGl 的话，你会在书里的第十七章中发现有关使用 OpenGL 显示英文的例子，书中使用了 glCallLists 来贴图一个数组里的图象来达到显示目的。  
  
那么我先在 glCallLists 下断，F9 运行游戏。  
  
   
  
可以看到栈里有字符“DELETE”字符和其长度 6，我们再去看下 glCallLists 的定义。  
void glCallLists(  
  GLsizei n,  
  GLenum type,  
  const GLvoid \*lists  
);  
那么显示函数就是这个 glCallLists 函数了，函数中第二个参数 1400 为 GL\_BYTE，意思就是让 glCallLists 函数按 8bit 读取 0022EC20 中存放的数据，数据范围只是 0~255。对于游戏所用到的汉字来说这个范围太小了，所以我们要修改这个参数，将其设置为 GL\_SHORT 或是 GL\_UNSIGNED\_SHORT，让 glCallLists 函数按 16bit 读取数据，其范围为 0~65535 足够汉字使用了。  
当第二个参数改为 GL\_SHORT 后，我们需要修改第三个参数 lists，如果不修改它，那么显示数字或是必要的英文字符的话 glCallLists 函数总不能正确的读取数据，因此会显示错误。我们可以简单的将原来的 ASCII 码转换成 UNICODE 编码。UNICODE 编码是将一个字符存放到 2 个字节里的，这样的话，英文和数字就可以像原来一样正确显示了。  
由于 glCallLists 显示的是一个数组里的贴图，lists 这个参数只是一个一个的提供要显示的图像所在的位置，ASCII 编码中从空格（0x20）开始以及后边的字符都可以显示，所以，在 NeHe 和教程中，使用 ASCII 编码字符的数值减去 0x20 来对应所要显示的贴图。那么我们转换为 UNICODE 码后，我们需要制作自己的字符数组来将字符排序再通过 glCallLists 显示出来。  
接下来我们把游戏中要用到的汉字全都找出来，把这些汉字组成一个不重复的表（使用 UNICODE 来编这个表），我们通过字符对比来重新组建一个数据串提供给 glCallLists 让其在游戏内部查找，编码：  
  
WORD GetMyChar(WORD wChar)  
{  
CString str=L”字库字符”;  
For (int i=0; i<str.GetLength(); i++)  
{  
                If (wChar == str.GetAt(i)  
                {  
                        Return (I + 128 + 32);  
}  
}  
Return 0;  
}  
  
//修改 glCallLists  
WORD wNewChar\[1024\];  
For (int i=0; i<wcslen(lists); i++)  
{  
        If (0 != GetMyChar(lists\[i\*2\]))  
{  
        wNewChar = GetMyChar(\*(PWORD)(lists+i\*2));  
}  
Else  
{  
wNewChar = lists\[i\*2\];  
}  
}  
这样我们得到新的经过编码的字符串了！  
  
经测试，中文可以正常显示，到此调试完毕。  
  
PS：欢迎对游戏汉化有兴趣的朋友加我 QQ：109990997，一起交流游戏汉化技巧。