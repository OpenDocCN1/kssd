
# IceExt 用的小程序

> 原文：[`www.pediy.com/kssd/pediy06/pediy6540.htm`](https://www.pediy.com/kssd/pediy06/pediy6540.htm)

刚刚开始用 IceExt，觉得很好用！

IceExt 有个 `!dumpscreen` 功能，使得不用跑到 softice 外面运行 `loader` 来保存屏幕信息，反汇编代码等等，很方便！有点象 `TRW2000` 下的反汇编输出到文件的命令。

但是 IceExt 保存的是 `RAW` 格式，虽然有个 `siwrender` 工具将其变成 `BMP` 格式，但是不能拷贝粘贴字符串，有时候还是不太方便。

我看了一下 IceExt 保存的 `RAW` 格式文件，还好格式比较简单，前一个字节保存字符信息，后一个字节保存颜色信息。

于是做了这个小程序，把颜色信息都扔掉了，只留下了字符，在 `dos` 窗口下运行，显示保存的屏幕信息，也可以重定向到文件。

使用方法：

`raw2t 源文件 [宽度 高度]`

宽度和高度可以在 `softice` 窗口中用 `width` 和 `hight` 命令来查看。默认情况下分别是 `80` 和 `55`（我的 `softice` 配置）。源程序一并给出，你也可以自己修改。

程序没有做太多的纠错功能，毕竟我们是学习 `crack` 的人，不是傻瓜用户。

源程序 `raw2t.c`

```c
01  #include <stdio.h>
02  #include <stdlib.h>
03  #include <memory.h>
04
05
06  int main(int argc, char* argv[])
07  {
08     int width = 80, hight = 55;
09     int i, j;
10     char *strLine;
11     FILE *fs;
12
13     if( argc < 2 ) {
14       printf("Convert IceExt raw files to text. Version 0.1  Made by kunlong.");
15       printf("\r\nUsege: %s raw_filename [width hight]", argv[0]);
16       printf("\r\nor   : %s raw_filename [width hight] > destination_filename", argv[0]);
17       printf("\r\nDefault width:80\r\nDefault hight:55\r\n");
18
19       return 0;
20     }
21
22     if( argc == 4 ) {
23       width = atoi( argv[2] );
24       hight = atoi( argv[3] );
25     }
26
27     strLine = malloc( width+1 );
28     memset(strLine, 0, width+1);
29
30     fs = fopen( argv[1], "r" );
31     if( fs == NULL ) {
32       perror( "open failed on input file" );
33
34       return 1;
35     }
36
37     fseek(fs, 0, SEEK_SET);
38
39     for( i = 0; i < hight; i++ ) {
40       for( j = 0; j < width; j++ ) {
41         strLine[j] = fgetc(fs);
42         fseek(fs, 1, SEEK_CUR);
43       }
44       puts(strLine);
45     }
46
47     free(strLine);
48     fclose( fs );
49
50     return 0;
51 }
52
```

顺便试了一下老罗的工具，不错呀！嘿嘿。

`4dd466d0e1e0cc123afbe504da2462ca`
