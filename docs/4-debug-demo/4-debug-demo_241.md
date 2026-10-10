

# EXEmp3 v2.0 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6980.htm`](https://www.pediy.com/kssd/pediy06/pediy6980.htm)

去除 EXEmp3 软件 10 秒钟烦人的 NAG   
  
  
使用软件，OD                修改后软件在 XP 下测试运行正常   
  
软件名称：EXEmp3 v2.0   
软件大小：431KB   
软件语言：英文   
软件类别：国外软件/共享版/MP3 制作   
运行环境：Win9x/Me/NT/2000/XP   
作者主页：http://www.exemp3.com/    
  
    EXEmp3 是一款很有趣的软件，它可以将 MP3 文件转换为 .exe 可执行文件格式，并且生成的   
 .exe 文件能够脱离任何播放器自动进行播放。EXEmp3 还允许你创建附加歌词或其他文字信息的   
 .exe 文件，生成的可执行程序界面也非常精巧美观，你可以在其中方便地调整音量、导入播放列表、   
设置循环播放、浏览歌词、切换为迷你界面等。难能可贵的是，EXEmp3 转换出来的可执行文件只比   
转换前的 .mp3 源文件增大约 48 KB 。   
  
本人属于破解水平低劣，基本属于看雪论坛中的最菜最菜的鸟，所以文章中错误或者疏漏欢迎大家批评指正。   
今天半夜在 QQ 上的朋友，看雪论坛网友发过来了此软件的英文版，让我试试，其实该软件，没注册就是开始运行的时候新出现 10 秒种的等待时间的有一个窗口。显示 0 秒后，变成 OK 字样，点击后，进入真正的软件使用界面。作者在提示上说正式版无此提示窗口。   
  
因为时间短，没太仔细分析，就选择个一下处理方案，1 修改时间，让软件不等待 10 秒，2 是让软件代替点击使用者点击 OK（就是说要机器自动快速的通过此画面就是目的，当然有更好的解决方法，这个软件花１小时就可以看明白个大概．细节时间的长点，可是我没那么多时间，懒惰啊）   
  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004011FA(C)   
|   
:004012AE 8B742408           mov esi, dword ptr \[esp+08\]   
:004012B2 6A00                    push 00000000   
  
\* Possible Reference to Dialog: DialogID\_0065, CONTROL\_ID:03E8, "www.exemp3.com"   
                                  |   
:004012B4 68E8030000       push 000003E8 //        
\---------------等于 10 进制的 1000，其实就是转换后的 1 秒，修该为 push01  ----6101 之后补 nop    
:004012B9 6A01                    push 00000001   
:004012BB 56                         push esi   
  
\* Reference To: USER32.SetTimer, Ord:0000h   
                                  |   
:004012BC FF1564B14000            Call dword ptr \[0040B164\]   
:004012C2 6A03                    push 00000003   
:004012C4 6A00                    push 00000000   
:004012C6 6A00                    push 00000000   
:004012C8 6A00                    push 00000000   
:004012CA 6A00                    push 00000000   
:004012CC 6AFF                    push FFFFFFFF   
:004012CE 56                          push esi   
  
其实到这基本 10 秒中以及除掉了。什么没看见判断时间的地方，不要紧我马上找给你   
  
  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040120A(C)   
|   
:00401212 A130D04000              mov eax, dword ptr \[0040D030\]   
:00401217 85C0                           test eax, eax     /////////////////////    
  
这个是测试时间的   
:00401219 7F3D                           jg 00401258         -------------nop nop   
这个跳转 nop 掉在也不会需要你等待了，时间问题解决，这个软件简单，也没输入注册号的地方所以也没找算法的必要，   
不过只没睡觉的人（还清醒）就知道这个跳转条到 0041258 是去减 1 去了，有兴趣的去看看很简单。算了我还是也复制出来吧   
  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00401219(C)   
|   
:00401258 48                      dec eax                               ------------------------------过 1 秒指令运行到此减 1 秒，开始的时候 EAX=A   
就是 10 秒，之后 9，8，7。。。。。。   
:00401259 6A0A                           push 0000000A                     
:0040125B 6868024100              push 00410268                       
:00401260 50                                 push eax   
:00401261 A330D04000              mov dword ptr \[0040D030\], eax    ——————\[0040D030\]作者定义的全局变量保存时间的   
:00401266 E8EB990000              call 0040AC56    
:0040126B 83C40C                  add esp, 0000000C   
:0040126E 50                      push eax   
:0040126F 8B44240C                mov eax, dword ptr \[esp+0C\]   
:00401273 6A01                    push 00000001   
:00401275 50                      push eax   
  
时间解决了，开始解决哪个倒霉 OK 按扭   
  
:0040121B 8B742408                 mov esi, dword ptr \[esp+08\]   
:0040121F 57                               push edi   
  
\* Reference To: USER32.GetDlgItem, Ord:0000h   
                                  |   
:00401220 8B3DB0B14000            mov edi, dword ptr \[0040B1B0\]   
  
\* Possible StringData Ref from Data Obj ->"OK"   
                                  |   
:00401226 685CD04000              push 0040D05C   
:0040122B 6A01                            push 00000001   
:0040122D 56                                 push esi   
:0040122E FFD7                           call edi   
:00401230 50                                 push eax   
  
\* Reference To: USER32.SetWindowTextA, Ord:0000h   
                                  |   
:00401231 FF1544B14000            Call dword ptr \[0040B144\]   
:00401237 6A01                              push 00000001   
:00401239 6A01                             push 00000001   
:0040123B 56                                  push esi   
:0040123C FFD7                          call edi   
:0040123E 50                                push eax   
  
\* Reference To: USER32.EnableWindow, Ord:0000h   
                                  |   
:0040123F FF15B8B14000            Call dword ptr \[0040B1B8\]   
:00401245 6A01                              push 00000001   
:00401247 56                                   push esi   
  
\* Reference To: USER32.KillTimer, Ord:0000h   
                                  |   
:00401248 FF15B4B14000            Call dword ptr \[0040B1B4\]   
:0040124E 5F                                   pop edi   
:0040124F B801000000                 mov eax, 00000001    /<-开始在这修改  修改成 JMP 00401298 就可以了|   
:00401254 5E                                   pop esi           
:00401255 C21000                          ret 0010                                                                            
                                                                                                                                      
                                                                                                                                      
                                                                                                                                      
:0040128C 8B442410                mov eax, dword ptr \[esp+10\]                              
:00401290 25FFFF0000              and eax, 0000FFFF   
:00401295 48                      dec eax   
:00401296 753D                    jne 004012D5   
:00401298 8B4C2408                mov ecx, dword ptr \[esp+08\]    <-  就跳到这个地方   
:0040129C 6A01                    push 00000001   
:0040129E 51                      push ecx   
  
好了可以收工了，估计不会浪费修改的人 1 个小时，这个外国人很善良。因为时间仓促我没花太长时间，用别的方法也能解决问题，反正 NAG 已经不用你点鼠标和等待了，   
  
感谢看雪论坛众多网友平时发的帖子让我收益，作为回报写个破（我写的很破的）文，看我写的不好的尽管扔西红柿（我喜欢吃西红柿），砖头别扔，太危险。   
  
 QQ 19226491  
 4dd466d0e1e0cc123afbe504da2462ca