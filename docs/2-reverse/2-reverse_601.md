

# 打造自己的网络电视

> 原文：[`www.pediy.com/kssd/pediy06/pediy6897.htm`](https://www.pediy.com/kssd/pediy06/pediy6897.htm)

相信这篇文章的出现将造成网络电视之类的东西漫天飞舞  
1．事情起因  
同学想看什么网络电视，要注册，把我这个义工叫了过去，几下搞定了，一看注册费，嘿嘿，要不我也写一个得了，说不定我也可以搞点钱花花，住宿费还欠着呢，不交钱不让考试，穷人家的孩子啊，5555~。可是，我也不知道怎么写啊，我还没有学过 windows 编程啊，还记得上期杂志我的文章《crack 新手进化篇》吗？我的补丁程序是黑忽忽的界面啊，哎，没有办法啊，不会 windows 编程，只好分析别人的程序了，就拿网络电视小精灵下手，peid 检查，delphi 编写，郁闷，很难找到是哪个函数自动打开 realplay 或 wmplayer 播放影音资料的，要是 vc++编写的就好了啊，只要作者不刻意加密，嘿嘿，到网上找了几个，我晕，不是 delphi 就是 vb 写的。没有一个 vc++的，当然了，我没有找所有的网络电视程序（PS：要是你有信心和耐心，不妨找找）要 vb 就不说了，太混乱了。只好对网络电视小精灵动真功夫了。  
  
2．Cracker 思维：  
用 dede 反编译，过程，Tnit1 的 TForm1 看到很多事件，仔细看看，看出不少东西啊，看到了一个 cctv1Click 事件，双击来到对应代码：  
  
004A1A80   53                     push    ebx  
004A1A81   8BD8                   mov     ebx, eax  
004A1A83   6A01                   push    $01  
004A1A85   6A00                   push    $00  
004A1A87   6A00                   push    $00  
  
\* Possible String Reference to: 'http://www.cctv.com/sports/'  
|  
004A1A89   68A01A4A00             push    $004A1AA0  
004A1A8E   6A00                   push    $00  
004A1A90   8BC3                   mov     eax, ebx  
  
\* Reference to: QForms.TCustomForm.GetClientHandle(TCustomForm):QWorkspaceH;  
|  
004A1A92   E88501FAFF             call    00441C1C  
004A1A97   50                     push    eax  
  
\* Reference to: shell32.ShellExecuteA()  
|  
004A1A98   E89B94F8FF             call    0042AF38  
004A1A9D   5B                     pop     ebx  
004A1A9E   C3                     ret  
  
  
  
嘿嘿，原来用的 ShellExecuteA，打开 ollydbg，载入网络电视小精灵，下断 004A1A80，也就是 cctv1Click 事件开始的第一条语句。运行，点击中央一台，开始播放了，不对啊，竟然没有断下，仔细看看网址，原来用的 ShellExecuteA 打开网页 http://www.cctv.com/sports/ 的，cctv1Click 不是调用 wmplayer.exe 打开中央一台的事件，那这个到底是什么东东呢？打开网络电视小精灵看看，在网络导航的体育栏目看到了原来是点击 cctv 体育的事件，果然如此，昏到。  
  
继续，再找一个熟悉的，BBC1Click,从名字上看，应该是播放 bbc 电台或者电视台（不知道有没有 bbc 电视台，呵呵）的事件，双击来到对应的地方，有发现，这次的代码和上面的完全不同：  
0048E298   BAB0E24800             mov     edx, $0048E2B0  
  
\* Reference to control TForm1.WebBrowser1 : TWebBrowser  
|  
0048E29D   8B805C040000           mov     eax, \[eax+$045C\]  
  
\* Reference to : TWebBrowser.\_PROC\_0046395C()  
|  
0048E2A3   E8B456FDFF             call    0046395C  
0048E2A8   C3                     ret  
  
分析发现网络电视小精灵似乎使用 TWebBrowser 构件，来播放网上的影音资料。赶紧从网上找了一下  
TWebBrowser 的资料，原来是 delphi 的一个控件，不死心，再找其他的事件，不是网页的基本都是用 TWebBrowser 构件，看来没有戏唱了。  
  
不管了，12:00 睡觉在说，明天早上还要上课呢？  
  
刚躺到床上，突然想起前面的 ShellExecuteA 不是可以打开网页吗?那还有没有其它作用呢？用 ida 反汇编网络电视小精灵，找到 ShellExecuteA(不要问我怎么找的啊，呵呵，在 names 选项)，ok，ida 就是好啊，给出了注释：  
jmp     ds:\_\_imp\_ShellExecuteA ; Opens or prints a specified file  
看到了吗？简单翻译应该是：打开或者打印一个具体文件。（应该没有翻译错吧，英语太差，4 级才考了 42 分）  
看来有希望，网上看电影不也是打开一个具体文件吗？还记得你用 realone 网上看电影吧，突然掉线的时候，是不是弹出一个东东，说什么......\*.rm 不能播放了,这就是一个具体文件啊,呵呵。  
赶紧用百度查了查 ShellExecuteA 的资料，哇，太多了，大部分都是 vb 中怎么用，没有一个我要的，灵机一动，换一种思维方式，当我们电击一个电视台或者广播台的时候，不是用 wmplayer.exe 打开就是用 realplay.exe 打开的（什么？不知道上面 2 个是什么东西，我晕，就是 xp 系统自带的 Windows Media Player 和 realone），所以，要是可以用 ShellExecuteA 并且用 realplay.exe 作为参数可以打开一个网上的影音文件，那么用 ShellExecuteA  realplay.exe 搜索应该是有结果的。  
好了，搜索 ShellExecuteA  realplay.exe，哇，没有发现，，不会吧。Ok，再来搜索 ShellExecuteA  wmplayer.exe，也是一个结果也没有。  
只好出 2 招了，拿出 cracker 必备的 api 手册，查 ShellExecuteA，我晕，居然是用 vb 描述的。  
算了。睡觉了。   
  
第 2 天上课的时候没有心思，继续想这个问题，没有什么结果，回到同学寝室（因为我没有电脑啊，只好用别人的了）查 MSDN，有发现了：  
HINSTANCE ShellExecute(        
    HWND hwnd,  
    LPCTSTR lpOperation,  
    LPCTSTR lpFile,  
    LPCTSTR lpParameters,  
    LPCTSTR lpDirectory,  
    INT nShowCmd  
);  
lpOperation 支持的用法有：  
edit   
Launches an editor and opens the document for editing. If lpFile is not a document file, the function will fail.   
explore   
Explores the folder specified by lpFile.   
find   
Initiates a search starting from the specified directory.   
open   
Opens the file specified by the lpFile parameter. The file can be an executable file, a document file, or a folder.   
print   
Prints the document file specified by lpFile. If lpFile is not a document file, the function will fail.   
英语菜，就不翻译了，更具体请查阅 msdn，到这里，你有什么想法了吗？lpOperation 已经指出了一些操作，好象对编写网络电视有用的只有上面的"open"啊。  
还有以下几个例子，这个函数非常有用，  
如下：  
To explore a folder，To open a folder，To launch the Shell's Find utility for a directory(没有给出具体例子,请查阅 msdn)  
如果你看过：《自己试试:为 OICQ 添加显示 IP 和端口功能》，《用 pediy 实现为程序添加对话框和网址的功能》，你就知道这个函数的功能实在太强大了，如过你看完了本文，你会发现 ShellExecuteA 的功能超级强（个人看法啊，没有什么 windows 编程经验）。  
后来从网上找到了《深入浅出 ShellExecute》，这两个值得借鉴（因为这个才有了我自己的测试和发现）：  
Q: 如何打开一个网页？  
ShellExecute(this->m\_hWnd,"open","notepad.exe", "c:\\\\MyLog.log","",SW\_SHOW );  
As you can see, I haven't passed the full path of the programs.   
ShellExecute(this->m\_hWnd, "open", "http://www.google.com", "", "", SW\_SHOW );  
你觉得哪个更有用呢?什么?你回答第一个.为什么?实验一下,假如我替换 notepad.exe 为 wmplayer.exe,替换  
"c:\\\\MyLog.log"为"d:\\\\t.mid", 当然这里的"d:\\\\t.mid"是真实存在的 mid 音乐,这样行吗?先简单测试一下,开始菜单,运行,输入 wmplayer.exe d:\\t.mid,是不是弹出  
弹出  
Windows Media Player 开始播放 t.mid 了.那么......  
赶快用新 vc 新建一个  
Win32 Console Application 工程，输入如下代码  
#include <windows.h>  
  
int main(int argc, char\* argv\[\])  
{  
  ShellExecute(0, "open", "wmplayer.exe", "d:\\\\t.mid", "", SW\_SHOW);  
  return 0;  
}  
执行，呵呵，是不是一个黑忽忽的界面之后,弹出  
Windows Media Player 开始播放 t.mid 了.成功  
注意：windows98 不是 wmplayer.exe，98 下面是 mplayer2.exe，windows2000 不清楚,未具体测试.请大家对照自己的机器做实验，PS:我同学的机器是 xp  
简单思维，第 2 个例子打开一个网页（废话），用什么打开的呢？用 ie（我同学的机器），确切的说是用默认的浏览器，因为这里没有指定有什么打开，也就是说打开网页有一个关联程序，也就是默认的浏览器。（一个砖头飞过来，啊!, 借用周星驰的《整鬼专家》里话（稍做修改）说：我拼了命的教你们，你们还拍板砖。）  
是啊，大家都知道这个，可是，发散一下思维，如故把这里的 http://www.google.com 换成起它的呢？这里我换成“d:\\\\t.mid”呢？赶快用新 vc 新建一个  
Win32 Console Application 工程，输入如下代码  
#include <windows.h>  
  
int main(int argc, char\* argv\[\])  
{  
  ShellExecute(0, "open", "d:\\\\t.mid", "", "", SW\_SHOW);  
//注意与上面的不同之处,去掉了"wmplayer.exe",同时要打开文件的具体位置成了  
//第 3 个参数  
  return 0;  
}  
执行，呵呵，是不是一个黑忽忽的界面之后,弹出  
Windows Media Player 开始播放 t.mid 了。因为我测试的机器默认 mid 关联 Windows Media Player，这就是为什么我在上面强调“打开网页关联默认的浏览器“,如果你的机器默认关联其它的播放器(如豪杰系列),则应该弹出豪杰的超级音频解霸播放 t.mid  
已经出现曙光了，现在到 baidu 找一首歌曲，我找的是：陈慧琳 - 记事本，查看属性，的到地址： http://mp3.baidu.com/u?u=http://www.jsshmzx.com/zhuwei/geci/flash/ZzI$.mp3 ，ok，继续修改：  
ShellExecute(0, "open", "http://mp3.baidu.com/u?u=http://www.jsshmzx.com/zhuwei/geci/flash/  
      /ZzI$.mp3", "", "", SW\_SHOW );  
注意：这个地址是 baidu 搜索的，到你测试的时候可能已经过时了。  
运行，如何，呵呵，Windows Media Player 开始播放：陈慧琳 的 记事本了吧。  
那么如果网上的文件不被 Windows Media Player 支持呢？比如说：rm 格式呢，那就用 realone 啊，这就是为什么网络电视小精灵的帮助要提到的必被软件。但是前提是 realone 关联了 rm 格式，测试时请注意，具体测试我就不写了,原理是一样的。  
但是请不要就此满足，你可能觉得我要讲的已经过了，但是我先想你可能没有想过下面的东西：  
  ShellExecute(0, "open", "realplay”, "d:\\\\t.mid", 0, SW\_MINIMIZE);  
这里强行指定播放器为 realplay，现在你可能觉得真的够了，如果我告诉你没有，呵呵，是不是有中想扁我的冲动，没有关系，还是那句话：“我拼了命的教你们，你们还拍板砖。”仔细的发觉一下，还会有让你吃惊的东西，一会我会讲到，我希望这篇文章能够让你觉得实践和思维的重要性，我想做到授人以渔，而不是授人以鱼，I am tring to free you mind。希望我可以做得到。  
希望你还没有忘记文章的标题，我们要打造自己的网络电视，关键技术已经解决了，不是吗？  
现在用任何可视化编程工具打造你自己我网络电视吧，这里我用 vc++，因为我正在学 c++啊，用 MFC 画个界面，不要问我怎么画的，我也不会，我让我同学帮我划的，基本就是照着向导做的。然后添加代码。等等，这里有个问题，我怎么知道那么多电视台的网址呢？嘿嘿，这就是下面要说的问题了。  
  
简单 hacker 思维,当我们网上看电视的时候，Windows Media Player 或者 realone 会打开网络电视内置的网址，这个网址是要作为 Windows Media Player 或者 realone 的请求发给服务器的。所以这里有 2 个方法：  
既然是网络电视内置的网址，那么我们可以 w32dasm 反汇编之后去找网址，这里仍以网络电视小精灵为例，反 w32dasm 汇编,会发现大量的网址，但是仔细看，会发现几乎没有一个网址是用来看电视和听广播的，都是网络电视小精灵里面网络导航提供的网址。似乎此路不同，是吗？用 ollydbg 载入网络电视小精灵，查看字符参考，没有吃惊吧，基本可以找到所有的看电视和听广播的网址。为什么 w32dasm 不可以，而 ollydbg 可以呢？（注：我用 ollydbg 仅仅是载入，还没有开始调试，这个时候应该也相当于静态反汇编，经测试 ida 静态反汇编也不能获取看电视和听广播的网址）  
其实我也不知道，还请大家指点啊。  
当然还有可能就是软件对网址进行了加密处理，但是这个加密必定是可以还原的，要不 Windows Media Player 或者 realone 怎么识别呢？高手还原算法就可以了，这个不是我能力范围的事情。  
  
2．既然这个网址是要作为 Windows Media Player 或者 realone 的请求发给服务器，所以我们可以用 sniffer 获得真实地址，测试时我用的 commview，个人觉得这是我等菜鸟最好用的 sniffer，这里我简单说一下，设置过滤条件，ip 为自己机器的 ip  
，只捕获流出的数据包，请设置协议，因为播放影音文件一般的协议有这些：mms://, rtsp://等  
如果捕获不到地址，请放宽条件。  
Ok，现在解决了所有的问题，剩下的就是不停的加入网址，写代码了。  
qq:254128123  email:ngaut@126.com

 4dd466d0e1e0cc123afbe504da2462ca