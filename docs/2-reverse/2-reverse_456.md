

# 改变控制台属性

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-790.htm`](https://www.pediy.com/kssd/pediy08/pediy8-790.htm)

经常用 hiew 修改文件,但是默认的窗口太小很不方便,而且因为 codepage 是中文的看着很不舒服.手动改起来太麻烦,开始想用一个 bat 文件来,查了半天也没查到 bat 里面如何设置 console 窗口的大小.知道的请指教  
写一个小程序来完成这个任务  
编译以后放到 send to 文件夹,直接发送到它就好了,不用改变系统默认的控制台属性  
1  #include<windows.h>  
 2  #include<tchar.h>  
 3  #define MAXPATH 256  
 4  #define MIN(x,y) (x<y?x:y)  
 5  
 6  //在这里定义参数  
 7  #define COLUMN 128           //控制台窗口列数  
 8  #define ROW 60               //控制台窗口行数  
 9  #define OUTPUT\_CODEPAGE 437  //控制台代码页  
10  #define HIEWPATH "d:\\\\meilin\\\\mary\\\\hiew\\\\hiew32.exe " //路径,注意多加一个空格!!!  
11  
12  void \_tmain( int argc, \_TCHAR \* argv\[\] )  
13  {  
14          HANDLE hStdOut = INVALID\_HANDLE\_VALUE;  
15          \_TCHAR aHiewPath\[MAXPATH\] = HIEWPATH;  
16          COORD dwScreenSize;  
17          SMALL\_RECT ddScreenPosition;  
18          PTSTR pszCmdLine = NULL;  
19  
20          hStdOut = GetStdHandle( STD\_OUTPUT\_HANDLE );  
21  
22          if ( INVALID\_HANDLE\_VALUE == hStdOut )  
23          {  
24                  perror( "Can't open ScreenBuffer" );  
25                  return;  
26          }  
27  
28          if ( FALSE == SetConsoleOutputCP( OUTPUT\_CODEPAGE ) )  
29          {  
30                  perror( "Can't change pagecode" );  
31                  return;  
32          }  
33  
34          dwScreenSize.X = COLUMN;  
35          dwScreenSize.Y = ROW;  
36  
37          if ( FALSE == SetConsoleScreenBufferSize( hStdOut, dwScreenSize ) )  
38          {  
39                  perror( "Can't set Buffer Size " );  
40                  return;  
41          }  
42  
43          ddScreenPosition.Top = 0;  
44          ddScreenPosition.Left = 0;  
45          ddScreenPosition.Bottom = ROW - 1;  
46          ddScreenPosition.Right = COLUMN -1;  
47  
48          if ( FALSE == SetConsoleWindowInfo(hStdOut, TRUE, &ddScreenPosition) )  
49          {  
50                  perror( "Can't set console position" );  
51                  return;  
52          }  
53  
54          pszCmdLine = GetCommandLine();  
55          if ((NULL != pszCmdLine && \*pszCmdLine ))  
56          {  
57                  while (pszCmdLine && \*pszCmdLine != ' ')  
58                  {  
59                          pszCmdLine++;  
60                  }  
61                  pszCmdLine++;  
62          }  
63          \_tcsncat(aHiewPath, pszCmdLine, MAXPATH - \_tcslen(aHiewPath));  
64          system(aHiewPath);  
65          return;  
66  
67  } 

 4dd466d0e1e0cc123afbe504da2462ca