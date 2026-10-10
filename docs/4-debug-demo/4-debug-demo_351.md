

# Windows Commander 4.51

> 原文：[`www.pediy.com/kssd/pediy03/forum307.htm`](https://www.pediy.com/kssd/pediy03/forum307.htm)

 注册你的 Windows Commander 4.51   
  
  最近忙得要命，好久没来看学论坛灌水了，今天要淹他一下^-^。感谢 JOHNSON 也就是 c-pen 的帮助，有空到大陆来玩呀。再有谢谢各位 cracker 的支持。   
  
  现在进入正题，如何注册你的 wincmd451? 你可以去 download 一个注册机，叫 wc451\_kg.exe，到处都是。我用 wincmd，只用它的比较功能。作为一个 cracker,用别人 crack 过的东西，尤其是注册机，非常的不爽，还是自己来吧。先进行准备工作：wincmd 是 aspack 压过的，解压缩吧，咦? unas1091 竟然不管用，看来还是偷不得懒:(。拿 trw2000 ver1.22 脱壳，用 trw 载入 wincmd32.exe,下 PNEWSEC 再下 MakePE 即可，wincmd 目录下会生成一个叫 NEWPE.EXE 的文件，这就是脱壳后的 wincmd32.exe，把 NEWPE.EXE 改名为 wincmd32.exe。好了，看看 wincmd 脱壳后还能怎么折腾:)。仔细分析了一下，它的注册机制同 wincmd4.0 一样，根本没有变化，hehe....，这下好办了。   
  准备工作完成，开始正式 crack，一共需要改 8 处。wincmd 是 keyfile 保护的那种软件，没有 keyfile 一样改。先用 trw2000 载入 wincmd32.exe，点 load,停下来后下 bpx readfile，按 f5 执行，被拦到，下 bd \*,按 f12 直到 wincmd 的领空。当走到如下 1.所示的 4ce105 时停下来，记下这个数，按 f5，接着退出 trw2000，换乘 sooftice。我为什么换成 softice，原因很简单，因为我不会用 trw:(。按 ctrl+d 进入 softice 下 bpx hmemcpy,按 f5 返回，执行 wincmd32.exe,被拦到，按 8 此 f12，进入 wincmd 的领空，下 bpx 4ce105,按 f5 又被拦到，按 f10 来到如 1.所示的程序:   
  
1\.   
017f:004ce105 e8fe53f3ff    call 00403508   
017f:004ce10a 83b80c00000000 cmp dword ptr \[eax+0c\],00   
017f:004ce111 7521 ->9090    jnz 004ce134 ->nop   
017f:004ce113 8d85e2feffff  lea eax,\[ebp\-011e\]   
  
wincmd 是靠比较内存判断是否注册的，所用的寄存器是 ebx,需要保持为 1,走到 4ce111,跳下去的话,会清掉 ebx,所以不能让他跳。按 f10 继续，接着他会比较你的 keyfile 是否为 128 字节，若是的话，提示这是旧版，让你更新。不管他，按 f10 走到如 2.所示的程序：   
  
  
2\.   
017f:004ce32a b966000000 mov ecx,0066   
017f:004ce32f babca42000 mov edx,0020a4bc   
017f:004ce334 e84ffbffff call 004cde88   
017f:004ce339 33c9      xor ecx,ecx   
017f:004ce33b b201      mov dl,01   
  
4cde88 这个 call 有问题，跟进去如 A.所示程序，把它改为 B.所示的程序。其中 b5e7c9f1c4a7b9ed20 是我注册的名字，你可以改成别的名字，其余的部分按 B.所示的改。   
  
004cde88->004cdf00   
  
  
A.                        |B.   
017f:004cde88 55    push ebp            |017f:004cde88 0517000000 add eax,00000017   
017f:004cde89 8bec  mov ebp,esp        |017f:004cde8d c60000      mov byte ptr \[eax\],00   
017f:004cde8b 83c4f8 add esp,-08        |017f:004cde90 0503000000 add eax,00000003   
017f:004cde8e 53    push ebx            |017f:004cde95 c600b5    mov byte ptr \[eax\],b5   
017f:004cde8f 56    push esi            |017f:004cde98 40        inc eax   
017f:004cde90 57    push edi            |017f:004cde99 c600e7    mov byte ptr \[eax\],e7   
017f:004cde91 8bf9  mov edi,ecx        |017f:004cde9c 40        inc eax   
017f:004cde93 8945fc mov \[ebp\-04\],eax        |017f:004cde9d c600c9    mov byte ptr \[eax\],c9   
.                        |017f:004cdea0 40           inc eax   
.                        |017f:004cdea1 c600f1    mov byte ptr \[eax\],f1   
.                        |017f:004cdea4 40        inc eax   
.                        |017f:004cdea5 c600c4    mov byte ptr \[eax\],c4   
.                            |017f:004cdea8 40        inc eax   
.                        |017f:004cdea9 c600a7    mov byte ptr \[eax\],a7   
.                        |017f:004cdeac 40        inc eax   
.                        |017f:004cdead c600b9    mov byte ptr \[eax\],b9   
.                        |017f:004cdeb0 40        inc eax   
.                        |017f:004cdeb1 c600ed    mov byte ptr \[eax\],ed   
.                        |017f:004cdeb4 40        inc eax   
017f:004cdef6 ff4df8 dec dword ptr \[ebp\-08\] |017f:004cdeb5 c60020    mov byte ptr \[eax\],20   
017f:004cdef9 75a8  jnz 004cdea3          |017f:004cdeb8 40        inc eax   
017f:004cdefb 5f    pop edi            |.   
017f:004cdefc 5e    pop esi            |.   
017f:004cdefe 59    pop ecx            |.   
017f:004cdeff 59    pop ecx            |017f:004cdefd c60000      mov byte ptr \[eax\],00   
017f:004cdf00 5d    pop ebp            |017f:004cdf00 90          nop   
017f:004cdf01 c3    ret            |017f:004cdf01 c3          ret   
  
接着按 F10 来到如 3.所示的程序，强制他跳过去，不跳的话会清掉 ebx 的。   
  
3\.   
017f:004ce38f e87846f3ff          call 00402a0c   
017f:004ce394 0f84ae000000 ->e9af000000 jz 004ce448 ->jmp 004ce448   
  
继续跟到如 4.所示的程序，按列出的方法改。   
  
4\.   
017f:004ce5c6 8d8558f8ffff    lea eax,\[ebp+fffff858\]   
017f:004ce5cc e8a786f3ff ->90  call 00406c78 ->nop   
                      ->33c9              ->xor ecx,ecx   
                       ->8808              ->mov bytes ptr \[eax\],cl   
017f:004ce5d1 ba30e94c00      mov edx,004ce930   
017f:004ce5d6 8d8558f8ffff    lea eax,\[ebp+fffff858\]   
  
当跟到 5.所示的地方，这是最后一个跳转了，强制它跳。   
  
5\.   
017f:004ce7aa e85d42f3ff  call 00402a0c   
017f:004ce7af 7402 ->eb02 jz 004ce7b3 ->jmp 004ce7b3   
017f:004ce7b1 33db        xor ebx,ebx   
  
接下来，按几次 f10 到如 6.所示的程序，按我列出的方法改。   
  
6\.   
017f:00532d13 a1b0e65300        mov eax,\[0053e6b0\]   
017f:00532d18 e8f340eeff ->eb00 call 00416e10 ->jmp 00532d1a   
             ->eb00              ->jmp 00532d1c   
             ->90              ->nop   
017f:00532d1d a1b0e65300        mov eax,\[0053e6b0\]   
017f:00532d22 e86dadeeff        call 0041da94   
  
再按几下 f10 就进入 wincmd 了，好了，用我的名字注册的。别忙还没完呢，wincmd 还没把你折腾够呢，你晃动几下鼠标，wincmd 就会冒出一个框框，告诉你感染 virus 一类的东西。这时按 ctrl+d 进入 softice,下 bpx lockmytask,按 f5 返回，点确定，被拦到，下 bd \*,按几次 f12 进入 wincmd 的地盘，往回倒到如 7.所示的地方，将 jg 004df5b0 改为 jmp 004df5b0，这是试出来的。   
  
7\.   
017f:004df56d e83e76f2ff  call 00406bb0   
017f:004df572 8b4302      mov eax,dword ptr \[ebx+04\]   
017f:004df575 83f808      cmp eax,08   
017f:004df578 7f36 ->9090  jg 004df5b0 ->jmp 004df5b0   
017f:004df57a 0f8444060000 je 004dfbc4   
  
不忙还没完呢，当你点 help,在点 about 时，会弹出一个注册信息框，你再点确定时，wincmd 会自动退出。所以你在点确定之前进入 softice,下 bpx lockmytask,按几次 f12 到如 8.所示的地方，把 je 004e8578 改为 jmp 004e8578 才能继续使用它。   
  
8\.   
017f:004e8562 a148405300  mov eax,dword ptr\[00534048\]   
017f:004e8567 2db9230000  sub eax,00000023b9   
017f:004e856c 740a ->eb0a je 004e8578 ->jmp 004e8578   
017f:004e8573 e86cbff1ff  call 004044e4   
  
完活:)   
  
最后就是用 winhex 以来的东西把 wincmd 改为注册版了，机器码我都列出来了。   
  
说明：wincmd4.0--4.51 之间的所有版本都可以这么改，只是对应的地址不一样。   
  
cracked by 电神魔鬼   
my oicq is 1789655   
thanks for your supports   
  
我贴出的改免注册版的 Winhex Ver9.52 有一点问题，竟然没人告诉我，真是太~!@#$%^&(.......   
我更正一下   
winhex ver9.52   
  
免注册   
  
\*\[45759d\]   
  
017f:00450041 jnz 00450064 ->nop   
017f:00450062 jz 00450068  ->jmp 00450068   
017f:0043e26a jz 43e277    ->jmp 43e277   
017f:0043e31b jl 0043e372  ->nop   
017f:0043e36b jz 0043e37d  ->jmp 0043e37d   
  
7521B85C524500   
9090B85C524500   
  
3B027404   
3B02eb04   
  
740bb8b0754500   
eb0bb8b0754500   
  
7C55A13C764500   
9090A13C764500   
  
741083F954   
eb1083F954   
  
改的方法不唯一，主要是让\[45759d\]永远置一就行。   4dd466d0e1e0cc123afbe504da2462ca