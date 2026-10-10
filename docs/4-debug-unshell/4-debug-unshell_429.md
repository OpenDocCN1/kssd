

# Advanced Direct Remailer 2.17 

> 原文：[`www.pediy.com/kssd/pediy05/pediy50168.htm`](https://www.pediy.com/kssd/pediy05/pediy50168.htm)

Asprotect 1.2x 加壳的 Advanced Direct Remailer 2.17 脱壳   
\--------------------------------------------------------   
  
官方主页: http://www.mailutilities.com   
软件下载: http://mirror.mailutilities.com/adr.zip   
软件说明: 建立本地的 SMTP 服务器，支持多线程和 Socks5 Proxy，这样您的 email 客户端软件可以不通过 ISP 的 SMTP 服务器而直接向目的地址的服务器发信，特别适合群发邮件的用户，速度比通过 ISP 的服务器要快 50 倍以上！   
  
使用工具: loader、TRW2000、superbpm、Import REConstructor v1.4.2+   
  
  
这个软件同样是在汉化新世纪论坛上有朋友要的，我的水平太烂，搞了好半天才搞定了脱壳:)   
  
脱壳过程:   
  
一  找程序的入口点   
    用 fs0 大哥的 loader 载入软件主程序，几秒钟后 loader 会告诉你，程序的 OEP 是 0043835E。   
  
  
二  TRW 初步脱壳   
    下面要在入口处脱壳，先打开 SuperBPM，选中 erase 前面的复选框，然后用 TRW 载入 adr.exe，下 g 43835E，TRW 停在程序的入口处，接着下 pedump ，将内存中的程序脱出来。   
  
  
三  修复导入表   
    打开原加壳程序，在 Import REConstructor v1.4.2+ 的 Attach to an Active Process 窗口中选取 adr.exe 进程，然后在下方的 OEP 处填入正确的入口 0001000，点 IAI AutoSearch，再点 Get Imports ，然后点 Show Invalid，在 Imported Functions Found 窗口里的无效地址上点鼠标右键，选 Trace Leve11(disasm)，这样 Import REConstructor 能够自动修复的项目就搞定了。   
  
    再点 show invaids，发现还剩下几个没有修复，再次在那几个没有修复的地址上点鼠标右键，选中 Plugin Tracer(Asprotect 1.2X Emul)，这样这个插件能修复的项目也全部搞定了。再点 show invaids，所有的导入表项目都显示有效了，运气真好:)   
  
    现在点击 Fix Dump，选择 TRW 脱壳出来的文件 dump.exe，会在目录下生成修正了导入表的程序 dump\_.exe   
  
四  去软件校验   
    运行 dump\_.exe，弹出一个提示，说“crypt api not found....”云云，不爽，还有校验啊:(   
  
    TRW 载入 dump\_.exe，下 pmodule，来到程序领空，然后下断点 bpx messageboxa，F5 运行程序，马上被 TRW 断了下来。（如果这里不先下 pmodule，直接下断，F5，程序会被断在 BFFXXXX 这个地址上，不在程序的领空内）   
  
0167:00434C70 6A04            PUSH    BYTE +04   
0167:00434C72 6AFF            PUSH    BYTE -01   
0167:00434C74 FF151C224400    CALL    \`KERNEL32!GetProcAddress\`   
0167:00434C7A 85C0            TEST    EAX,EAX  // EAX 是否为 1   
0167:00434C7C A3DC7B4600      MOV      \[00467BDC\],EAX   
0167:00434C81 7516            JNZ      00434C99  // 不是则死，所以  JNZ => JMP   
0167:00434C83 6A10            PUSH    BYTE +10   
0167:00434C85 68AC634400      PUSH    DWORD 004463AC   
0167:00434C8A 687C9A4400      PUSH    DWORD 00449A7C   
0167:00434C8F 50              PUSH    EAX   
0167:00434C90 FF1530234400    CALL    \`USER32!MessageBoxA\`  // 弹出 crypt api not found.... 的提示   
0167:00434C96 33C0            XOR      EAX,EAX   
0167:00434C98 C3              RET      // 上面的校验过程结束   
0167:00434C99 B801000000      MOV      EAX,01  // 跳到此处，即可跳过校验   
0167:00434C9E C3              RET      
  
  
    所以将 0167:00434C81 的 JNZ 00434C99 改为 JMP 00434C99 即可。用 LodePE 的 FLC 功能，得 offset 34C81，用十六进制编辑工具改之 (75 -> EB)。   
  
    因为是朋友汉化需要，特别用 eXescope 编辑了一下资源，无问题。   
  
  
五  破解   
    汗...这个我跟了半天，一点头绪都没有，倒是有跟到一个跳转，改了之后随意输入注册码都会提示注册成功，不过根本没有用-\_-# 这个还是留待高手解决吧:P   
  
  
六  遗留的问题   
    在 98 下修复导入表，破掉校验的文件在 XP 下无法运行，XP 下的到了 98 下也不能运行，比较了一下，好像两个系统下修复的导入表项目有很多不同，与我上次在论坛问过的一个 aspr 1.2x 加壳的软件差不多，希望有高手能帮忙看看。   
  
  破解的问题，电神老大能不能指点一下？   
  
  
                                                                                                                                                                    炎之川 于 2002.6.20   
                                                            mail:skipli@sina.com   
                                                            http://skipli.yeah.net  4dd466d0e1e0cc123afbe504da2462ca