

# Regsnap

> 原文：[`www.pediy.com/kssd/pediy03/forum583.htm`](https://www.pediy.com/kssd/pediy03/forum583.htm)

今日回到北京，又拿起了 regsnap 进行研究，记得有大侠说其注册算法极为复杂，对我这种菜鸟来说就干脆采取暴力法好了。   
  
    经 Dasm 反汇编，发现其对本身文件进行校验，有改动会拒绝执行，由于 winme 上使用 softice 太麻烦，用 trw2000 来代替，所以不知道这个东东是不是对 softice 有检测。   
  
    查看文件头的时候，发现 pe 标志前有一段乱码，不知是校验文件完整性还是用来探测 softice 的。不知哪位大侠给解释一下。   
  
    改了很长时间，由于我的水平很菜，那个起始 splashscreen 非常讨厌，一直无法搞掉。几乎准备放弃了，最后努力一次吧。使用 pe-editor 看看文件，没有找到.idata，只找到.data，猜想大概差不多吧，算了算 importtable 的地址，决定用 trw2000 来看看。load 进去以后就一直 d 那个输入表的地址，看看何时它从??变成有内容了。果然，在一个 call 后，内容出现了，干脆 pedump 一下（什么逻辑嘛，这时候就 pedump 了）......   
  
    奇怪的事情终于发生了，首先并没有指望那个 dump 出来的程序可以运行，而它就能运行！而且那个 splashscreen 没了！好像也不再做 CRC 校验了！难道前面说的 pe 标志前的乱码就是干这些事情用的？这是怎么搞的嘛，乱七八糟的，高手给讲讲呀。接下来用 reshacker 找到第二个要求购买的 dialog，回到调用它的地方，   
:00406D3F FF1518784100            Call dword ptr \[00417818\]   
:00406D45 6A00                    push 00000000   
:00406D47 8D8D28FFFFFF            lea ecx, dword ptr \[ebp+FFFFFF28\]   
:00406D4D E8CEBD0000              call 00412B20  <------------就是这个 dialog 调用   
:00406D52 B30E                    mov bl, 0E   
:00406D54 8D8D28FFFFFF            lea ecx, dword ptr \[ebp+FFFFFF28\]   
:00406D5A 885DFC                  mov byte ptr \[ebp\-04\], bl   
  
\* Reference To: MFC42.Ordinal:09D2, Ord:09D2h   
                                  |   
:00406D5D E856D20000              Call 00413FB8    
:00406D62 83F801                  cmp eax, 00000001   
:00406D65 7449                    je 00406DB0    <------------这里成 EB   
:00406D67 8D4D88                  lea ecx, dword ptr \[ebp\-78\]   
:00406D6A C645FC0F                mov \[ebp\-04\], 0F   
  
    这样当过期的时候按下 quit 还是可以进入主程序。接下来就是去掉这个 dialog 的问题了，用 hedit 打开主程序，寻找 t r y，中间的空格其实是 0x00，找到后前面的 FF FF 80 改为 FF FF 90，好了，虽然提示没有注册，但是没有使用时间限制了。   
  
    最后声明一点，由于我过早将时间调过 30 天试用期，所以在 30 天内是否可以正常运行我不清楚，不过应该可以吧。  4dd466d0e1e0cc123afbe504da2462ca