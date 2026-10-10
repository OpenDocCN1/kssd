

# 对一个 IAT 加密壳的分析

> 原文：[`www.pediy.com/kssd/pediy10/93019.html`](https://www.pediy.com/kssd/pediy10/93019.html)

这几天研究脱壳，上论坛看到一老兄求助地址（http://bbs.pediy.com/showthread.php?t=92330），由于自己也是新手，所以下载下来研究半天也无进展，幸好得到看雪老大的帮助，让我柳暗花明又一村，呵呵，经过老大的详细指点，终于将这个壳去掉了，下面将我的心得写出来，也算是留个记号吧，好久没来看雪发文章了！  
  
    OD 载入后，不分析，如图，一开始就是一个异常安装，并故意产生一个向\[00000000\]写入的异常如图:  
   ![](img/966d1f364854bd33405541f4838781f0.jpg)  
  
    SHIFT+F7 进入异常处理，在 ZWContinue 前看堆栈中有一个地址是 00401016,如图：  
  
  ![](img/6685d4140d80e0d4959d35d9ce2665f2.jpg)  
  
    到 00401016 处，是一个无条件跳转指令：  
  
  ![](img/25f797e83a5f96512461207f09841e22.jpg)  
  
    那么我们就到 00720FAF 处直接下断，然后 F9 到此处，如图：  
  
   ![](img/646f58acf1134861ffe1eee66d2a04e2.jpg)  
  
    取消此处的断点，然后继续向下跑，来到此处：  
  
   ![](img/3bc7df9477fe2993498b1bb1c86d8a26.jpg)    
  
    此时，应该进入到此 CALL 内部，因为直接 F8 的话会产生不能继续的异常(进程终止)，估计此 CALL 中有作怪的地方，F7 进入：  
    
   ![](img/8f090b7b12f1f4545ec2d100547d8b0f.jpg)  
  
   继续向下，来到这里，果然发现了捣乱的地方：  
  
   ![](img/222ef241f844502e5284268e93c8388b.jpg)  
  
    此处的两个 je 任意一个让其实现即可避过壳的检测。如果用 SoftICE，会有通过 CreateFile 检测调试器的代码，本文忽略。过了拦路虎，就能顺利通过此 CALL 了，如图:  
  
    ![](img/3f8cb104ceef06b9f9258db39f5ca88d.jpg)  
  
    继续 F8 向下，发现一个大的跨段跳跃，通过它走向 OEP 处的（如图）：  
  
   ![](img/f9665ad7d9906ad1ba8302ab4eda40fa.jpg)  
  
    F8 来到了程序的 OEP，此时 DUMP，然后用 ImportREC 修复，发现有很多无效的的函数指针，如图：  
  
   ![](img/e485c30a6412d86d6fa4c2248d6e2f0d.jpg)  
  
    这时可以用它强行一个一个地对函数的 IAT 进行修复，但是数量十分大，上百个函数，得找到一个更明智的办法才行啊！为什么加壳的程序能正常运行，而我们脱出来的就不行了，还有这么多无效的指针？应该是壳对部分的 IAT 进行了加密处理，只要找到了加密部分，绕过去不就能正常脱壳了么？  
  
    然后就来实现吧，CTRL+F2 重新载入程序,找到 IAT 的起始地址 1D4090 加上基址就是 5D4090,CTRL+F2 重新载入程序，d 5d4090, 在此处下硬件写入断点  
  
   ![](img/d61e1b7ef8c406b008ef954ce3f054bc.jpg)  
  
    重复前面的一些步骤，F9 几次以后来到这里，发现向 IAT 的首地址写入了数据：  
   
   ![](img/7439f4f60c60257291feb6ee94b25a0c.jpg)  
  
    而后面是一个回跳，继续向 IAT 的后续地址写入函数指针，所以可以判断，这就是 IAT 的加密代码部分（这个壳加密的是 kernel32.dll 中的部分函数 IAT）：  
  
003914A7    C783 CC1A0010 0>mov     dword ptr \[ebx+10001ACC\], 0  
003914B1    8B02            mov     eax, dword ptr \[edx\]  
003914B3    85C0            test    eax, eax  
003914B5    74 67           je      short 0039151E  
003914B7    52              push    edx  
003914B8    8983 CC1A0010   mov     dword ptr \[ebx+10001ACC\], eax  
003914BE    A9 00000080     test    eax, 80000000  
003914C3    74 09           je      short 003914CE  
003914C5    25 FFFFFF7F     and     eax, 7FFFFFFF  
003914CA    6A 00           push    0  
003914CC    EB 0E           jmp     short 003914DC  
003914CE    8B4D 08         mov     ecx, dword ptr \[ebp+8\]  
003914D1    0341 08         add     eax, dword ptr \[ecx+8\]  
003914D4    33C9            xor     ecx, ecx  
003914D6    66:8B08         mov     cx, word ptr \[eax\]  
003914D9    51              push    ecx  
003914DA    40              inc     eax  
003914DB    40              inc     eax  
003914DC    50              push    eax  
003914DD    FF75 FC         push    dword ptr \[ebp-4\]  
003914E0    FF93 0A210010   call    dword ptr \[ebx+1000210A\]  
003914E6    5A              pop     edx  
003914E7    50              push    eax  
003914E8    8B02            mov     eax, dword ptr \[edx\]  
003914EA    A9 00000080     test    eax, 80000000  
003914EF    75 18           jnz     short 00391509  
003914F1    8B4D 08         mov     ecx, dword ptr \[ebp+8\]  
003914F4    0341 08         add     eax, dword ptr \[ecx+8\]  
003914F7    C600 00         mov     byte ptr \[eax\], 0  
003914FA    40              inc     eax  
003914FB    C600 00         mov     byte ptr \[eax\], 0  
003914FE    40              inc     eax  
003914FF    8A08            mov     cl, byte ptr \[eax\]  
00391501    C600 00         mov     byte ptr \[eax\], 0  
00391504    40              inc     eax  
00391505    84C9            test    cl, cl  
00391507  ^ 75 F6           jnz     short 003914FF  
00391509    58              pop     eax  
0039150A    85C0            test    eax, eax  
0039150C  ^ 0F84 3FFFFFFF   je      00391451  
00391512    8906            mov     dword ptr \[esi\], eax  
00391514    8902            mov     dword ptr \[edx\], eax  
00391516    83C2 04         add     edx, 4  
00391519    83C6 04         add     esi, 4  
0039151C  ^ EB 89           jmp     short 003914A7  
      
    我们要让 IAT 部分得到正确的地址，就必须让 EAX 的值是对应的函数地址值！由于我们到达加密 IAT 代码部分时 IAT 的首部已经被写入了加密的 IAT 值，也就是说加密函数至少已经运行了一次！所以此时必须 CTRL+F2 再次重新载入程序，来到加密代码首部，然后继续向下，修改代码如下，保持 EDX 的值是压栈时的值，得到正确的函数地址指针：  
  
   ![](img/bffd4ff9588d6bd13a9585c3df17534f.jpg)  
  
    这样，这个加密 IAT 的部分就相当于被我们绕过去了。直接 F4 到加密结束部分，发现 IAT 部分已经被写入了正确的函数地址信息：  
  
   ![](img/6a2b75aaf5d1ab4417a31b6adff2fe7c.jpg)  
  
    最后，再次将我们修改的部分还原，就可以直接到达原来的 OEP 处了！（CTRL+G，输入原来我们找到的 OEP 地址，F2 下断，F9 到达后 dump 即可！）再用 ImportREC 修复时，发现所有的 IAT 都有效了，如图：  
  
   ![](img/2c5468b63cdf7774e2e22dccaaaceadf.jpg)  
  
    修复后程序能正常运行，PEID 查壳，VC++编写：  
  
   ![](img/7ac1c091ec5b20dc5f628c6e2f397dbf.jpg)