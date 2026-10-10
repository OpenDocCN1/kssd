

# 29A 杂志#8:编译迷你 Win32 可执行文件(使用 C 代码)

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-745.htm`](https://www.pediy.com/kssd/pediy08/pediy8-745.htm)

不知道哪位前辈帮忙开通一下上传附件功能～谢谢  
  
\[声明\] By Anskya  
初学翻译,由于英语基础和写作风格的差异,可能导致译文与原文的一些差异  
如有不明请阅读原文,谢谢大家支持....  
为了更适合大家阅读和理解文章内容这里将使用一些比较容易理解的语句  
请大家不要责怪小弟翻译水平不好.(原文直接翻译会令人非常费解)  
  
由于本人能力有限,翻译过程中难免会出现错误,请大家指正,  
写希望大家多多交流,谢谢各位支持和看完此文,转载请保留版权  
\[By Anskya\] Email:Anskya@Gmail.com QQ:115447  
  
//-------------------------------------------------  
编译迷你 Win32 可执行文件(使用 C 代码)  
作者:DoxtorL.,西元 2004 年  
  
如今人们使用各类加壳软件程序压缩应用程序使自己的程序更短小,  
使得大家都忽略(原文:是否因为懒惰?)了如何自己编译一个小巧的二进制文件,  
希望下面所讲述的可以给你一些思路..  
(个人理解是这个意思～不明的地方看原文～这里只是依照个人思维翻译的)  
  
摘要:  
简介.  
第 1 讲.不使用代码,定位代码入口点和 C 语言连接时间库(就是\*.lib)  
第 2 讲.融合 PE 节  
第 3 讲.使用 Dos stub(DOS 插桩柱)来存储初始化数据(可以解释成常量!)  
第 X 讲.再进一步......等等....  
  
源代码:test.c,test2.c  
  
介绍:  
编译迷你的应用程序是建立在一个好的连接器基础上的.  
这里我使用的是 Pelles-c 的连接库  
(译者注:Pelles-c,类似 LCC 的一种 C 语言编译器,其实就是 LCC 源代码修改的)  
Pelles-c 是一款免费很不错的编译器(只支持 C 语言)Polink(指的是连接器)  
是模拟科隆 M$ VC 的连接器的.  
  
我使用一个"Hello World"程序(test.c)来示范编译一个迷你的应用程序.ā  
(原文意思是～我应该用次来优化一个二进制文件...不理解他是什么意思)  
  
\*\*\*\*\* 第 0 讲: 默认编译  
使用命令:  
cc.exe test.c  
polink.exe c:\\progra~1\\pellesc\\lib\\crt.lib c:\\progra~1\\pellesc\\ ->  
lib\\win\\kernel32.lib test.obj  
  
result: test.exe ,size=27648 bytes  
这个体积有点大了,但是程序只使用了 Kernel32.dll 中的函数  
\*\*\*\*\* 第 1 讲:  
不使用代码,重定位代码入口点和 C 语言连接时间库(就是\*.lib),  
连接器使用一个常规的入口点,代码功能:读取和解析附加参数.  
  
我们的例子不需要上面的特征(命令),为了优化我们的代码我们决定去除这些东西  
  
.  
连接器有个参数:  
  
/ENTRY: <symbol>          
  
(/ENTRY:后面跟程序入口函数--一般我们的程序是 main 开始的)  
  
symbol 是你的程序 main 函数的入口点(译者注:当然你也可以设置成别的函数例如:  
  
/ENTRY:MyMain)  
如果我们需要使用一个 C 连接库可以使用:  
  
/NODEFAULTLIB  
  
编译命令:  
  
cc.exe test.c  
polink.exe /NODEFAULTLIB /ENTRY:main   
  
c:\\progra~1\\pellesc\\lib\\win\\msvcrt.lib test.obj  
  
result: test.exe, size=2560 bytes  
  
\*\*\*\*\* 第 2 讲:融合 PE 节  
(译者注:许多加壳软件加壳后程序的节明显比原来小了.这里所说的就是这个意思  
  
)  
我们的 test.exe 程序编译后拥有 2 个 PE 节: .text ,.rdata ,.data  
我们可以融合三个节成为一个节  
  
连接器设置:  /MERGE:<source section>=<destination section>  
  
(原文:this option can be used several times!,感觉无意义就没有翻译)  
  
编译命令:  
  
cc.exe test.c  
polink.exe /MERGE:.rdata=.text /MERGE:.data=.text /NODEFAULTLIB   
  
/ENTRY:main ->  
c:\\progra~1\\pellesc\\lib\\win\\msvcrt.lib test.obj  
  
result: test.exe, size=1536 bytes  
  
\*\*\*\*\* 第 3 讲:使用 Dos stub(DOS 插桩柱)来存储初始化数据  
DOS Stub 头部是 PE 程序为了兼容 msdos 系统所作的,如果你的程序的程序  
在 msdos 环境下运行,会显示一个"This program cannot be run in DOS mode"  
的警告信息然后退出.(译者注:就和 printf 打印这条信息一个结果～不过他使用的  
  
是 int21 中断)  
他可以作一些你需要让程序在 dos 下作的事情(那是你自己的事情这里就不细说了)  
dos stub 尺寸是 512 的倍数,最小的可能性也是 512 字节,如果我们去除警告信息 dos   
  
stub  
只有 160 字节,剩下的空间可以用来存储我们的初始化数据(译者注:许多 DIY PE 的  
  
牛人都作过小于  
512 字节的 dos stub..似乎原作者认为 dos stub 必须是 512 字节不然就会发生什么  
  
事情..:P)  
  
我们必须改变少量的 C 代码,  
  
我们需要将数据分成两个部分,一个部分保存在 dos   
  
stub 中,剩下的部分保存在.text 段中.  
  
Polink 连接器可以设置我们需要使用的 dos stub.这里我们用一个免费的汇编语言  
  
编译器 FASM 来作  
(译者注:B 人很喜欢的一个编译器,开源,有大量的宏指令可以使用,由于是低级编  
  
译器可以按照自己的设想编译 PE 结构)  
  
编译器设置参数: /STUB:<my dos stub> ,这里设置我们需要的 dos stub 的二进  
  
制文件(这么说话好别扭~:D)  
  
编译命令:  
  
fasm stub.asm stub.bin  
cc.exe testb.c  
polink.exe /STUB:stub.bin /MERGE:.rdata=.text /MERGE:.data=.text   
  
/NODEFAULTLIB /ENTRY:main ->  
c:\\progra~1\\pellesc\\lib\\win\\msvcrt.lib testb.obj  
  
\*\*\*\*\*第 X 讲:再进一步  
(译者注:dos stub 中)可以保存你需要导入的 DLL 的函数名和～"By Name"等等字符  
  
串信息  
(似乎好多壳都这样作了～比如(Win)Upack,FSG 等等壳),一般使用导入序数不是一  
  
个好主意,  
因为操作系统的差别～(译者注:比如 Win2k,WinXp,Win9x 下的函数未必就是一个序  
  
号.大家需要注意这里)  
总之..一些 DLL,如 wsock32.dll,你完全没有必要在 Windows 下使用序数....  
对导入表使用序数,你需要使用一些特别的方法导入连接库....  
一个不需要使用代码的方法,在你连接程序的创建导入表列表的方法.  
polib.exe 这个工具可以创建你连接库的导入表保存成一个 .def 的文件  
这里是一个演示的文件:  
\*\*\*\*\* example of a .def file  
LIBRARY WSOCK32  
EXPORTS  
accept @1  
bind @2  
closesocket @3  
connect @4  
htons @9  
listen @13  
recv @16  
send @19  
socket @23  
gethostbyname @52  
WSAAsyncSelect @101  
WSACleanup @116  
WSAStartup @115  
\*\*\*\*\* end of file  
.def 文件保存了我们需要导入的 wsock32.dll 的函数,不是全部只有我们需要的  
<symbols> ā@XXX   
symbols 是设置我们需要使用的函数名称-.xxx 是我们需要导入的函数序数  
polib.exe 同时会创建一个.lib 的文件  
改变 PE 字节的一种 BT 方法(译者注释:原文翻译是这样:获取文件数据的一种 BT 方法  
  
.)  
,一个 PE 文件必须是 512 的倍数(老毛病又犯了.)  
和一些小于 1024 字节的程序还是可以和 Win32 相兼容的.  
(译者注:这里说的是文件对齐～512 字节,一开始我也没看懂以后看了下面的代码  
  
才明白)  
这个方法是:调整 PE 文件的对齐字节(设置成 512 的倍数--当然也可以是使用 512)  
这样调整后,文件的第一个节的虚拟地址偏移+大小就是文件的大小(接近于 1000h  
  
的倍数.也可以是 1024)  
(译者注:剩下的部分原作者都没有写～也许是我翻译的有问题,反正就没了～呵呵  
  
不知道是什么意思)  
  
以下为源代码  
/\*\*\*\*\* test.c\*/  
#include <stdio.h>  
  
#define POEM "\\  
It might be lonelier\\r\\n\\  
Without the Loneliness --\\r\\n\\  
I'm so accustomed to my Fate --\\r\\n\\  
Perhaps the Other -- Peace --\\r\\n\\r\\n\\  
Would interrupt the Dark --\\r\\n\\  
And crowd the little Room --\\r\\n\\  
Too scant -- by Cubits -- to contain\\r\\n\\  
The Sacrament -- of Him --\\r\\n\\r\\n\\  
I am not used to Hope --\\r\\n\\  
It might intrude upon --\\r\\n\\  
Its sweet parade -- blaspheme the place --\\r\\n\\  
Ordained to Suffering --\\r\\n\\r\\n\\  
It might be easier\\r\\n\\  
To fail -- with Land in Sight --\\r\\n\\  
Than gain -- My Blue Peninsula --\\r\\n\\  
To perish -- of Delight --\\r\\n\\r\\n\\  
(Emily Dickinson, 1830-1886)\\r\\n"  
  
void main()  
{  
printf(POEM);  
}  
/\*\*\*\*\* End of source code\*/  
  
/\*\*\*\*\* test2.c\*/  
#include <stdio.h>  
  
#define PTR\_POEM1 0x400040  
  
#define POEM2 "\\  
&nābsp;little Room --\\r\\n\\  
Too scant -- by Cubits -- to contain\\r\\n\\  
The Sacrament -- of Him --\\r\\n\\  
I am not used to Hope --\\r\\n\\  
It might intrude upon --\\r\\n\\  
Its sweet parade -- blaspheme the place --\\r\\n\\  
Ordained to Suffering --\\r\\n\\r\\n\\  
It might be easier\\r\\n\\  
To fail -- with Land in Sight --\\r\\n\\  
Than gain -- My Blue Peninsula --\\r\\n\\  
To perish -- of Delight --\\r\\n\\r\\n\\  
(Emily Dickinson, 1830-1886)\\r\\n"  
  
void main()  
{  
printf("%s%s",PTR\_POEM1,POEM2);  
}  
/\*\*\*\*\* End of source code\*/  
  
作者原文没有帖 FASM 部分的代码我这里补上  
下载最新的 FASM 版本然后直接编译就好了,会生成一个 stub.bin 文件  
请自行修改成 stub.exe 然后在按照上面所说的文章进行编译就可以了  
/\*\*\*\*\* stub.asm\*/  
DOS\_Header:   
   .e\_magic     dw "MZ"      ;IMAGE\_DOS\_SIGNATURE   
   .e\_cblp      dw 0x0080   
   .e\_cp        dw 0x0001   
   .e\_crlc      dw 0x0000   
   .e\_cparhdr   dw 0x0004   
   .e\_minalloc  dw 0x0010   
   .e\_maxalloc  dw 0xFFFF   
   .e\_ss        dw 0x0000   
   .e\_sp        dw 0x0140   
   .e\_csum      dw 0x0000   
   .e\_ip&nbsāp;       dw 0x0000   
   .e\_cs        dw 0x0000   
   .e\_lfarlc    dw 0x0040   
   .e\_ovno      dw 0x0000   
   .e\_res       rw 4   
   .e\_oemid     dw 0x0000   
   .e\_oeminfo   dw 0x0000   
   .e\_res2      rw 10   
   .e\_lfanew    dd PE\_header  ;PE header Offset   
  
DOS\_Stub:   
 org $+DOS\_Stub   
  
POEM1 db "It might be lonelier",0dh,0ah  
      db "Without the Loneliness --",0dh,0ah  
      db "I'm so accustomed to my Fate --",0dh,0ah  
      db "Perhaps the Other -- Peace --",0dh,0ah,0dh,0ah  
      db "Would interrupt the Dark --",0dh,0ah  
      db "And crowd the",0  
  
rb 160-(DOS\_Stub-$)  
PE\_header:  
/\*\*\*\*\* End of source code\*/  
  
由于本人能力有限,翻译过程中难免会出现错误,请大家指正,  
写希望大家多多交流,谢谢各位支持和看完此文,转载请保留版权  
\[By Anskya\] Email:Anskya@Gmail.com QQ:115447 

 4dd466d0e1e0cc123afbe504da2462ca