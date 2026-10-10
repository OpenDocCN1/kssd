

# Safewow 分析笔记

> 原文：[`www.pediy.com/kssd/pediy12/128568.html`](https://www.pediy.com/kssd/pediy12/128568.html)

最近又玩私服了，好吧我承认我蛋已经碎了……  
  这服登陆器好大，达到了 9.27M，我第一个反应就是内建了 wow 主程序，因为我压缩过的，大概 ASpack 之后是 2.56M 的样子，你看其中有充分的体积来做各种各样的事情。  
  果然，他释放了一个叫做 safewow.exe 的东西，看图标看说明就是改过的 wow 主程序。  
  什么，你问他为什么要改，你猜……  
  看了下，明显的地方告诉你这个壳的名字。你问我为什么会知道？  
                 public start  
.Safe:01A336C4 start           proc near  
.Safe:01A336C4                 call    sub\_1A336E6  
.Safe:01A336C4 start           endp ; sp-analysis failed  
.Safe:01A336C4  
.Safe:01A336C4 ; ---------------------------------------------------------------------------  
.Safe:01A336C9 aSafengineProte db 'Safengine Protector v1.8.6.0',0  ;这样够明显的了吧……  
    
  大家都知道的 wow 自身就是有一些保护措施的，运行之后要附加进去很难，具体参考 http://bbs.pediy.com/showthread.php?t=45935  
  而且外面还有一层壳，脱起来很乏味，好吧，我承认我不会脱。但是我毫无压力。  
  首先我用普通权限启动这个 wow，然后用管理员权限启动 LoadPE。为什么要用管理员权限？你没看见它说自己是反外挂登陆器么，我权限比它高了才好让他看不见呀！  
  选择 safewow.exe，dump 整个 exe，就可以载入 IDA 啦。看看，和谐不？  
  然后直觉告诉我，修改的部分肯定不会很远，也不会很多，因为本身 wow 里面就没有多少的 cave 字节，同时你觉得这个时代做这种外挂的人有几个喜欢没事去大量 patch 一个庞大的 exe？  
  然后 G 0401000，显然的出现了如下的代码：  
.text:00401000                 call    sub\_6E4780  
.text:00401005                 jmp     loc\_99EBB5  
  这边么，我想都不想的就知道第二个跳转就是跳到了 patch 地点，为什么？显然我很经常用 OD 载入 wow 的嘛……  
.text:0099EBB5 loc\_99EBB5:                             ; CODE XREF: .text:00401005j  
.text:0099EBB5                 pusha  
.text:0099EBB6                 push    6C6Ch  
.text:0099EBBB                 push    642E736Dh         
.text:0099EBC0                 push    esp  
.text:0099EBC1                 call    dword\_99F264      
.text:0099EBC7                 test    eax, eax  
.text:0099EBC9                 jz      near ptr 207A5h  
.text:0099EBCF                 add     esp, 8  
.text:0099EBD2                 push    0  
.text:0099EBD4                 push    74696E49h         
.text:0099EBD9                 push    esp  
.text:0099EBDA                 push    eax  
.text:0099EBDB                 call    dword\_10AF9EC     
.text:0099EBE1                 test    eax, eax  
.text:0099EBE3                 jz      near ptr 207A5h  
.text:0099EBE9                 add     esp, 8  
.text:0099EBEC                 call    eax  
.text:0099EBEE                 popa  
.text:0099EBEF                 jmp     loc\_40B362  
  
  很难看懂不是么？来发挥想象力：  
  
.text:0099EBB5 loc\_99EBB5:                             ; CODE XREF: .text:00401005j  
.text:0099EBB5                 pusha  
.text:0099EBB6                 push    'll'  
.text:0099EBBB                 push    'd.sm'          ; ms.dll  
.text:0099EBC0                 push    esp  
.text:0099EBC1                 call    dword\_99F264    ; loadlibrary  
.text:0099EBC7                 test    eax, eax  
.text:0099EBC9                 jz      near ptr 207A5h  
.text:0099EBCF                 add     esp, 8  
.text:0099EBD2                 push    0  
.text:0099EBD4                 push    'tinI'          ; init  
.text:0099EBD9                 push    esp  
.text:0099EBDA                 push    eax  
.text:0099EBDB                 call    dword\_10AF9EC   ; getprocaddress  
.text:0099EBE1                 test    eax, eax  
.text:0099EBE3                 jz      near ptr 207A5h  
.text:0099EBE9                 add     esp, 8  
.text:0099EBEC                 call    eax  
.text:0099EBEE                 popa  
.text:0099EBEF                 jmp     loc\_40B362  
  
  你猜我这中间有没有用 debugger？  
  好了，这一刻如果强力的你把外面的壳脱了，修复了，是不是感觉愤怒了？  
  恩恩，继续是 Safengine Protector，那我也继续。  
  看着代码，再回顾区段排列，我突然有意思明悟，这丫的是 E 语言的静态编译。  
  什么你说你不知道 OEP 在哪边？这个结合自己找，现在我只想关注 Init 函数。不过看了半天，绕晕了，还是祭出字符串搜索。然后我定位到了 sub\_4A612A2，看起来很像主处理过程。  
  一路看着函数向下，我发现了 wow 目录之中另外一个文件的名字出现了，A41E0CE3D4019.tmp，看起来 40K，不是很大，所以果断用记事本打开，发现开头熟悉的 MZ 字样，然后丢入 LoadPE，发现这个是一个 DLL，原名称为 CRCDLL.dll。  
  但是还不怎么容易理解虽然可以猜测到很多。对易语言不懂，所以还是需要挂到 OD 里面去。虽然脱壳还是很麻烦，但是我依然可以很巧妙的获得一份干净的 ms.dll。然后分析可以看出，其实它的反外挂还是老思路，建个线程扫扫窗口，扫扫网络用的 api 是不是被 hook 了。防 WPE?改改窗口名，改改注入的 dll 名，改改 patch 策略，还不是一样用……话说，居然金山游侠也进黑名单了……  
  有点虎头蛇尾的感觉。但是无所谓了。  
           说真的 SafeEngine Protector 的脱壳对我来说还是很难的，毕竟我只是一个小菜，但是保护软件的安全并不只是全靠一个猛壳就能完美的事情不是么？  
  
  值得一提的是，此刻我想起了一个我很早就在思索的问题，海风月影的 Email 是 SafeEngine.com 的，我在想他做的 StrongOD 是不是能过 SafeEngine Protector 的保护，仔细回味，这真的是一个很严肃的问题……