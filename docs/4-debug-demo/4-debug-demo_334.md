

# Button Studio 1.41

> 原文：[`www.pediy.com/kssd/pediy03/forum595.htm`](https://www.pediy.com/kssd/pediy03/forum595.htm)

暴力破解第三回 Button Studio 1.41   
目标软件:Button Studio 1.41   
保护方式:KeyFile   
破解方法:暴力破解(怎么每次都是爆破,你有暴力倾向呀!@#@&^&$#)   
破 解 人:TAE!   
软件介绍:一个制造各种漂亮按钮的工具,特点是小巧,易用,做出的按钮很漂亮.   
下载地址:www.interkodex.com   
      
    首先声明本人心理健康,乐观向上,绝对没有暴力倾向,只是由于学艺不精,只有爆破了:)   
    这个软件没有让你输入注册码的地方,我想可能是 KeyFile 保护的.所以就运行 Filemon   
再运行 Button Studio 发现它读取 buttonstudio.rg 这个文件.猜想这个文件应该是 KeyFile.   
    建立 buttonstudio.rg 文件.运行软件,奇怪,没反应,不能运行!!我猜对了,肯定是程序运行   
时,检查 KeyFile,但我建立的文件肯定不是真正的 KeyFile(费话!)所以程序发现了,就不让运行   
了.   
    运行 TRW 1.23(BTW:为什么有时候 CTRL+N 不能呼叫?)设置断点 bpx CreateFileA,运行   
程序,被中断,这时下 D EAX 查看,按 F5 运行程序,又被中断一定记住要查看 EAX 的值,就这样按   
了大约 6 次 F5,这时程序已经开始读取 buttonstudio.rg 文件了,下 Pmodule,回到 Button Studio   
的程序段.   
\* Reference To: kernel32.CreateFileA, Ord:0000h   
                                  |   
:0040636B E8B0AEFFFF              Call 00401220   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00406395(U)   
|   
:00406370 83F8FF                  cmp eax, FFFFFFFF    //回到这里.   
:00406373 7429                    je 0040639E   
:00406375 8903                    mov dword ptr \[ebx\], eax   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004063AF(U)   
|   
:00406377 5F                      pop edi   
:00406378 5E                      pop esi   
:00406379 5B                      pop ebx   
:0040637A C3                      ret   
      
稍后便会运行到这里:   
\* Possible StringData Ref from Code Obj ->"buttonstudio.rg"   
                                  |   
:004B3196 684C344B00              push 004B344C   
:004B319B 8D852CFDFFFF            lea eax, dword ptr \[ebp+FFFFFD2C\]   
:004B31A1 BA03000000              mov edx, 00000003   
:004B31A6 E8590FF5FF              call 00404104   
:004B31AB 8B952CFDFFFF            mov edx, dword ptr \[ebp+FFFFFD2C\]   
:004B31B1 8D8554FDFFFF            lea eax, dword ptr \[ebp+FFFFFD54\]   
:004B31B7 E8F22CF5FF              call 00405EAE   
:004B31BC BA01000000              mov edx, 00000001   
:004B31C1 8D8554FDFFFF            lea eax, dword ptr \[ebp+FFFFFD54\]   
:004B31C7 E80532F5FF              call 004063D1   
:004B31CC 6A00                    push 00000000   
:004B31CE 8D55F0                  lea edx, dword ptr \[ebp\-10\]   
:004B31D1 B901000000              mov ecx, 00000001   
:004B31D6 8D8554FDFFFF            lea eax, dword ptr \[ebp+FFFFFD54\]   
:004B31DC E86F2DF5FF              call 00405F50   
:004B31E1 B8FF000000              mov eax, 000000FF   
:004B31E6 2B45F0                  sub eax, dword ptr \[ebp\-10\]   
:004B31E9 8945EC                  mov dword ptr \[ebp\-14\], eax   
:004B31EC 8B75EC                  mov esi, dword ptr \[ebp\-14\]   
:004B31EF 85F6                    test esi, esi   
:004B31F1 7E49                    jle 004B323C   
:004B31F3 C745FC01000000          mov \[ebp\-04\], 00000001   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004B323A(C)   
|   
:004B31FA 6A00                    push 00000000                      //   
:004B31FC 8D55F0                  lea edx, dword ptr \[ebp\-10\]        .   
:004B31FF B901000000              mov ecx, 00000001                  .   
:004B3204 8D8554FDFFFF            lea eax, dword ptr \[ebp+FFFFFD54\]  .   
:004B320A E8412DF5FF              call 00405F50                      .   
:004B320F B8FF000000              mov eax, 000000FF                  .   
:004B3214 2B45F0                  sub eax, dword ptr \[ebp\-10\]        .这里是个循环,好像是读取文件中   
:004B3217 8945F0                  mov dword ptr \[ebp\-10\], eax        .的 Ascii,并且运算编码,懒得看了.   
:004B321A 8D8520FDFFFF            lea eax, dword ptr \[ebp+FFFFFD20\]  .   
:004B3220 8B55F0                  mov edx, dword ptr \[ebp\-10\]        .   
:004B3223 E8440DF5FF              call 00403F6C                      .   
:004B3228 8B9520FDFFFF            mov edx, dword ptr \[ebp+FFFFFD20\]  .   
:004B322E 8D45F8                  lea eax, dword ptr \[ebp\-08\]        .   
:004B3231 E8160EF5FF              call 0040404C                      .   
:004B3236 FF45FC                  inc \[ebp\-04\]                        .   
:004B3239 4E                      dec esi                            .   
:004B323A 75BE                    jne 004B31FA                      //   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004B31F1(C)   
|   
:004B323C 8B75EC                  mov esi, dword ptr \[ebp\-14\]      //将光标定位在这里,按 F7,继续向下运行   
:004B323F 85F6                    test esi, esi   
:004B3241 7E40                    jle 004B3283   
:004B3243 C745FC01000000          mov \[ebp\-04\], 00000001   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004B3281(C)   
|   
:004B324A 6A00                    push 00000000   
:004B324C 8D55F0                  lea edx, dword ptr \[ebp\-10\]   
:004B324F B901000000              mov ecx, 00000001   
:004B3254 8D8554FDFFFF            lea eax, dword ptr \[ebp+FFFFFD54\]   
:004B325A E8F12CF5FF              call 00405F50   
:004B325F 8B45F8                  mov eax, dword ptr \[ebp\-08\]   
:004B3262 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:004B3265 8A4410FF                mov al, byte ptr \[eax+edx\-01\]   
:004B3269 34FF                    xor al, FF   
:004B326B 25FF000000              and eax, 000000FF   
:004B3270 0345FC                  add eax, dword ptr \[ebp\-04\]   
:004B3273 3B45F0                  cmp eax, dword ptr \[ebp\-10\]    //到这里停一停,比较!可惜经过编码   
:004B3276 7405                    je 004B327D                    //在这里一定要跳,下面还有一处   
:004B3278 E88B09F5FF              call 00403C08                  //运行到这里程序便退出了    
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004B3276(C)   
|   
:004B327D FF45FC                  inc \[ebp\-04\]   
:004B3280 4E                      dec esi   
:004B3281 75C7                    jne 004B324A                    //又上去了.   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004B3241(C)   
|   
:004B3283 6A00                    push 00000000   
:004B3285 8D55F0                  lea edx, dword ptr \[ebp\-10\]   
:004B3288 B901000000              mov ecx, 00000001   
:004B328D 8D8554FDFFFF            lea eax, dword ptr \[ebp+FFFFFD54\]   
:004B3293 E8B82CF5FF              call 00405F50   
:004B3298 8B45F8                  mov eax, dword ptr \[ebp\-08\]   
:004B329B E8A40DF5FF              call 00404044   
:004B32A0 3B45F0                  cmp eax, dword ptr \[ebp\-10\]      //又是比较   
:004B32A3 7405                    je 004B32AA                      //一定要跳!   
:004B32A5 E85E09F5FF              call 00403C08                    //进去就完了!   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004B32A3(C)   
|   
:004B32AA 8D8554FDFFFF            lea eax, dword ptr \[ebp+FFFFFD54\]   
:004B32B0 E8632DF5FF              call 00406018   
:004B32B5 B8849C4D00              mov eax, 004D9C84   
:004B32BA 8B55F8                  mov edx, dword ptr \[ebp\-08\]   
:004B32BD E8560BF5FF              call 00403E18   
:004B32C2 33C0                    xor eax, eax   
:004B32C4 A3809C4D00              mov dword ptr \[004D9C80\], eax   
  
    将上面的两个跳转改为 Jmp,试运行,没有 Nag 了,看看 About,那个讨厌的 Not registered 变成了   
Registered to:(乱码),因为建立 KeyFile 中的 Ascii 不对,所以这里显示的是乱码.无论如何破解都   
成功了!   
    这里真是个很好的地方,能和各位学到不少东西,可惜马上就要开学,我今年中专三年级,要毕业了,   
以后就没那么多时间搞 Crack 了,好苦啊!唉!还有这个月的电话费……,反正免不了挨妈妈一顿骂.  4dd466d0e1e0cc123afbe504da2462ca