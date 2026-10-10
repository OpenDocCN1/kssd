

# UPX 壳的 PATCH------谈 EBOOK 文章类软件的破解

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-297.htm`](https://www.pediy.com/kssd/pediy07/pediy7-297.htm)

UPX 壳的 PATCH  
                         ---------谈 EBOOK 文章类软件的破解  
  
一、简介    
  
【郑重声明】：本文只为技术而破解，纯属技术交流, 请勿用于非法用途。    
【软件名称】：国产的东东，就不说了  
【下载页面】：http://www1.skycn.com/soft/20059.html  
【软件简介】：EBOOK 小说系列  
【调试环境】：WinXP SP2、ODbyDYK、regedit、HIEW7.10、PEID0.93  
【软件限制】：共享软件，10 次使用限制，无功能限制  
  
在单位闲的无聊，翻看小说，居然发现了秘密，书籍是经过加密处理的！HOHO，左右也无聊，拉出来看先，哈：）  
  
二、破解过程  
  
1、查找 OEP，脱壳  
用 PEID0.93 查看主程序，壳为 UPX 壳，UPX 0.89.6 - 1.02 / 1.05 - 1.24 (Delphi) stub -> Markus & Laszlo \[Overlay\]  
  
用 ODbyDYK 载入后如下：  
  
004C9A40     60         pushad  
004C9A41     BE 0050480>mov esi,00485000  
004C9A46     8DBE 00C0F>lea edi,dword ptr ds:\[esi+FFF7C000\]  
004C9A4C     C787 9CC00>mov dword ptr ds:\[edi+9C09C\],B6128192  
004C9A56     57         push edi  
004C9A57     83CD FF    or ebp,FFFFFFFF  
004C9A5A     EB 0E      jmp short 004C9A6A  
.........  
.........  
004C9BA4     FF96 14B30>   call dword ptr ds:\[esi+CB314\]  
004C9BAA     61            popad  
004C9BAB   ^ E9 AC30FDFF   jmp 0049CC5C                 ；跳转到 OEP  
004C9BB0     C8 9B4C00     enter 4C9B,0  
004C9BB4     D89B 4C009CD0 fcomp dword ptr ds:\[ebx+D09C004C\]  
004C9BBA     49            dec ecx  
004C9BBB     0000          add byte ptr ds:\[eax\],al  
004C9BBD     0000          add byte ptr ds:\[eax\],al  
004C9BBF     0000          add byte ptr ds:\[eax\],al  
004C9BC1     0000          add byte ptr ds:\[eax\],al  
004C9BC3     0000          add byte ptr ds:\[eax\],al  
004C9BC5     0000          add byte ptr ds:\[eax\],al  
004C9BC7     0000          add byte ptr ds:\[eax\],al  
004C9BC9     0000          add byte ptr ds:\[eax\],al  
  
标准的 UPX 壳，PUAHAD 和 POPAD 对应，所以上面的 4C9BAB 处就是跳转到 OEP 处执行，很简单的以为 LOADPE 脱壳，RecImport160 修复就可以运行。脱壳后才发现程序挂接的数据段已经不见了，23M 大小的程序变成了 800 多 K，而且程序不能正常运行了。  
（用 PEID 的 UPX UPACKER 插件也很容易脱壳，但脱壳后效果跟上面一样）  
  
2、研究注册方法  
  
程序的注册，是利用机器号的方法，而且看来 EBOOK 为了防止写出注册机，在算法上也是下了很大功夫，并不是很容易就跟踪出来的。查找漏洞吧，软件既然给了 50 次的试用机会，那就一定有机可乘了:)！  
研究程序目录并没有什么 INI 之类的软件，估计使用信息是在注册表里。动态反编译软件，发现一个注册表串\\Software\\eBook Edit Pro，HOHO，REGEIDT 打开注册表一看，在 LOGIN 里有 2 个键值 SD 和 SO，最可疑的就是 SO 键值，后来研究表明，SD 中写着失效标志，SO 中是使用的次数。中断程序中所有标记为 SO 的代码，GO，截获后得到 EBOOK 读写 SO 的代码段如下：  
  
00498C6C     8BCE          mov ecx,esi                           ；此处 ECX 为使用次数  
00498C6E     BA D48C4900   mov edx,00498CD4                 ; ASCII "SO"  
00498C73     8B45 FC       mov eax,dword ptr ss:\[ebp-4\]  
00498C76     E8 01D7F9FF   call 0043637C                    ;我们将 PATCH 这里，改为 jmp 004CC4F0，去 4CC4F0 修改我们的 ECX 中的数据  
00498C7B     33C0          xor eax,eax  
  
  
中断上面 498C6C 处就能发现，ECX 中的数字，就是注册表中已经使用次数+1，所以在执行 call 0043637C 之前，修改程序，使 ECX 中的次数少于 50 次，不就可以了！这就需要修改代码里的 ECX 数值。  
但问题又出来了，程序脱壳后就无法正常执行，又不知道如何来修复壳中数据段，如何才能修改这里的 ECX 呢？所以只好引入动态 PATCH 了。  
  
3、UPX 壳的 PATCH  
  
UPX 壳的 OEP 简单明了，从上面 1 中也能看出，执行到 OEP 前的所有代码都是很明了的，所以不需要我们学大虾们 PATCH 猛壳那样 DUMP 一次内存了。当程序运行到 CS：004C9BAA 的时候，所有的代码已经完成解压缩，全部恢复到内存中去了，我们选择在这个时候 PATCH 我们的代码，就是修改 CS：00498C76  
本来想给程序加一个 SECTION，然后加入我们的 PATCH 代码的，后来发现 CS：004CC4A0 的下面恰好有很大的一段代码段是空闲的，刚好能放下我们的 PATCH 代码，HOHO，又便宜我们了，还犹豫什么，GOGOGO！  
  
用 HIEW7.10 改写的代码如下：  
  
004C9BAA     61            popad                
004C9BAB   - E9 FE280000   jmp 古龙.004CC4AE     ；不去执行 OEP，而是执行我们的 PATCH 代码  
004C9BB0     C8 9B4C00     enter 4C9B,0  
004C9BB4     D89B 4C009CD0 fcomp dword ptr ds:\[ebx+D09C004C\]  
004C9BBA     49            dec ecx  
004C9BBB     0000          add byte ptr ds:\[eax\],al  
004C9BBD     0000          add byte ptr ds:\[eax\],al  
004C9BBF     0000          add byte ptr ds:\[eax\],al  
  
  
PATCH 代码：  
004CC4AE     60              pushad                             ；PATCH 开始，保护现场  
004CC4AF     BE C8C44C00     mov esi,古龙.004CC4C8              ；改变后的指令代码  
004CC4B4     BF 768C4900     mov edi,古龙.00498C76              ；PATCH 的目标代码  
004CC4B9     B9 05000000     mov ecx,5                          ; 5 字节指令  
004CC4BE     F3:A4           rep movs byte ptr es:\[edi\],byte ptr ds:\[esi\]  
004CC4C0     61              popad                              ；PATCH 结束，恢复现场  
004CC4C1   - E9 9607FDFF     jmp 古龙.0049CC5C                  ；去执行 OEP，大功告成  
004CC4C6     0000            add byte ptr ds:\[eax\],al             
004CC4C8     E975380300      ；修补后，CS：498C76 处就变成了   jmp 004CC4F0  
  
  
004CC4EA     0000            add byte ptr ds:\[eax\],al  
004CC4EC     0000            add byte ptr ds:\[eax\],al  
004CC4EE     0000            add byte ptr ds:\[eax\],al  
004CC4F0     B9 01000000     mov ecx,1             ；在让 ECX 变成 1  
004CC4F5     E8 829EF6FF     call 古龙.0043637C    ；执行被改为跳转的代码部分  
004CC4FA   - E9 7CC7FCFF     jmp 古龙.00498C7B     ；返回程序原地点继续执行  
004CC4FF     90              nop  
  
  
怎么样？把修改保存，然后运行一下看看吧，HOHO，不错吧？  
  
三、总结  
     EBOOK 的保护方式，就是注册表键\\Software\\eBook Edit Pro、Login 下面的 SD 和 SO，SD 中记录了失效标志，而 SO 中是使用次数。只要删除这个根键，EBOOK 打开书籍时候又认为第一次打开了。    
     看来内存 PATCH 也是很不错的方法，而 EBOOK 本身，个人认为，虽然在算法和防 DUMP 上用了很多功夫（也可能是我这种菜鸟私家感觉:)），但小小的注册表，却是致命伤。  
       
  
     谢谢你看到这里，就当是 PATCH 壳的一个小过程了，也许能给一些人 CRACKER EBOOK 类的开一个思路。如转载请保持信息的完整性，谢谢。  
  
                                                       oep   2005.6.30 

 4dd466d0e1e0cc123afbe504da2462ca