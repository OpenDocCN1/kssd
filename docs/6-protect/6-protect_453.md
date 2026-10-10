

# ANTI－W32dasm

> 原文：[`www.pediy.com/kssd/pediy03/forum736.htm`](https://www.pediy.com/kssd/pediy03/forum736.htm)

哪位大侠能解决这个对抗 W32DASM 的“轻松试卷”???   
  
　　各位大哥，小弟已用动态跟踪破了“轻松试卷”，想写一篇破解心得，可它无法反汇编，用 W32DASM v8.93 一打开就“没有响应”，试过很多电脑都是这样，猜想它可能对抗 W32DASM，试过很多办法都不行。有谁能帮我解决它，使它能用 W32DASM 打开，好让我写出这篇心得文章。   
　　我用过 TRW2000 的 U 命令，可保存的文件根本不适合写文章，看得很不爽。   
　　各位大侠，帮帮小弟好吗？   
  
请到此下载“轻松试卷”   
  
http://easypaper.yeah.net

两个 ANTI－W32dasm 的程序的解决办法   
作者：小牧童\[CCG\]   
版权：CCG 所有，转载请保持完整。   
难度：易   
  
程序 1：http://www.my169.com/~zxhxmz/porciins.exe   
程序 2：就是上面这位大哥所想要解决的轻松试卷。http://www.shijun.com/easypaper/cn/download/eps404.zip   
  
现象：这两个程序用 W32dasm 打开后陷入没有反应之中，只有用 Ctrl+alt+del 才能使其退出。   
思路：W32dasm 陷入无应之中估计进入某处死循环。呵呵，小牧童就是会钻死牛角尖^\_^！有了思路 ANTI 功能自然就解了。   
方法：   
程序 1：   
运行 W32dasm  并打开 animal.exe 文件进行编译，W32dasm 进入死循环。   
下 Ctrl+D 进入 softice 按 2 次 F12 到下面：   
  
:0046149F E8DCDB0400              call 004AF080   
:004614A4 83C408                  add esp, 00000008   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00461494(C)   
|   
:004614A7 8A9C35E9FDFFFF          mov bl, byte ptr \[ebp+esi\-00000217\]   
:004614AE 80FB2F                  cmp bl, 2F   
:004614B1 7615                    jbe 004614C8   
:004614B3 80FB3A                  cmp bl, 3A   
:004614B6 7310                    jnb 004614C8   
:004614B8 889D0CF6FFFF            mov byte ptr \[ebp+FFFFF60C\], bl   
:004614BE C6850DF6FFFF00          mov byte ptr \[ebp+FFFFF60D\], 00   
:004614C5 83C602                  add esi, 00000002   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:004614B1(C), :004614B6(C)   
|   
:004614C8 8D850CF6FFFF            lea eax, dword ptr \[ebp+FFFFF60C\]   
:004614CE 50                      push eax   
:004614CF E8EC9D0400              call 004AB2C0   
:004614D4 59                      pop ecx   
:004614D5 8945F4                  mov dword ptr \[ebp\-0C\], eax   
:004614D8 33D2                    xor edx, edx   
:004614DA 8955F8                  mov dword ptr \[ebp\-08\], edx   
:004614DD 8B4DF8                  mov ecx, dword ptr \[ebp\-08\]   
:004614E0 8B45F4                  mov eax, dword ptr \[ebp\-0C\]   
:004614E3 3BC8                    cmp ecx, eax   
:004614E5 0F83B6FDFFFF          jae 004612a1 //这里改为 909090909090 跳出死循环。    
  
:004614E6 90                      nop   
:004614E7 90                      nop   
:004614E8 90                      nop   
:004614E9 90                      nop   
:004614EA 90                      nop   
  
  
程序 2：   
运行 W32dasm  并打开 easypaper.exe 文件进行编译，W32dasm 进入死循环。   
下 Ctrl+D 进入 softice 按 2 次 F12 到下面：   
  
:0046151B E8BCD60400            call  KERNEL32!lstrcat    //按 2 次 F12 后到这里   
:00461520 FF45F8                  inc \[ebp\-08\]   
:00461523 8B4DF8                  mov ecx, dword ptr \[ebp\-08\]   
:00461526 8B45F4                  mov eax, dword ptr \[ebp\-0C\]   
:00461529 3BC8                    cmp ecx, eax   
:0046152B 72BE                    jb 004614EB        ／/将这里改为 9090   
:0046152D E96FFDFFFF            jmp 004612A1        
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0046121D(C)   
|   
:00461532 8B957CFFFFFF            mov edx, dword ptr \[ebp+FFFFFF7C\]   
:00461538 85D2                    test edx, edx   
:0046153A 7411                    je 0046154D   
  
将 0046152b 的比较取消，顺着 0046152d 的 jmp 来到下面：   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0046152D(U)   
|   
:004612A1 C7857CFFFFFF01000000    mov dword ptr \[ebp+FFFFFF7C\], 00000001   
:004612AB 8B8D78FFFFFF            mov ecx, dword ptr \[ebp+FFFFFF78\]   
:004612B1 85C9                    test ecx, ecx   
:004612B3 7410                    je 004612C5   
:004612B5 33C0                    xor eax, eax   
:004612B7 89857CFFFFFF            mov dword ptr \[ebp+FFFFFF7C\], eax   
:004612BD 33D2                    xor edx, edx   
:004612BF 899578FFFFFF            mov dword ptr \[ebp+FFFFFF78\], edx   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004612B3(C)   
|   
:004612C5 84DB                    test bl, bl   
:004612C7 0F85D9FEFFFF      jne 004611A6    ／／这里改为 9090909090 跳出死循环。   
:004612CD 6A01                    push 00000001  4dd466d0e1e0cc123afbe504da2462ca