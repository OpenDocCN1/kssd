

# kai's super goo 1.5

> 原文：[`www.pediy.com/kssd/pediy03/forum427.htm`](https://www.pediy.com/kssd/pediy03/forum427.htm)

解决 kai's super goo 1.5 读光盘的问题   
  
软件名称：kai's super goo 1.5 （35M ——〉天啦，我就是 2M 的专线网络蚂蚁下载也用了 7 分 24 秒）   
下载地址：http://www.iszero.com/php3-bin/redirect.php?ID=0&GET=/tuxing/zhizuo/Kaigoo.zip   
软件说明：   
调试工具：SoftICE4.05、W32dsm89   
    目的：解决 kai's super goo 1.5 读光盘的问题   
  
    作者：chn-boy   
调试日期：2000-12-05   
  
    其实这个软件调试工作很简单，没有必要写的这么详细，只是为了写作规范化，所以我还是把它当作那么一回事情来   
写。首先在 SoftICE 下下断点 bpx GetWindowTextA，F5 回到 Windows，在那个输入名字对话框里随便输入一些字符，运行。   
SoftICE 会自动弹出（我一直都说：so beautiful），下命令 bd \*，按 F12 一直到 superGoo 领空，到了领空，第一件事情就   
是在进入领空处的地方下一个断点——bpx xxxxxxxx，以被后用。然后就是 F10，一路跟踪，直到弹出出错对话框为止，   
如下面的程序片断：   
  
:00401EC4 85C0                    test eax, eax <-- 2）第二个步骤尝试是改这个。。   
:00401EC6 0F8532010000            jne 00401FFE   
  
:00401ECC 8D8C2454020000          lea ecx, dword ptr \[esp+00000254\]   
:00401ED3 E818650800              call 004883F0   
:00401ED8 85C0                    test eax, eax  <-- 根据下面的那个出错对话框，首先怀疑这个地方有问题   
                                                <-- 1）第一个步骤尝试是改这个。。   
:00401EDA 0F8493000000            je 00401F73   
  
:00401EE0 FFD3                    call ebx   
:00401EE2 6A06                    push 00000006   
  
\* Possible StringData Ref from Data Obj ->"custom errors\_"   
                                  |   
:00401EE4 686C104900              push 0049106C   
:00401EE9 8D8C245C010000          lea ecx, dword ptr \[esp+0000015C\]   
:00401EF0 68FF000000              push 000000FF   
:00401EF5 51                      push ecx   
:00401EF6 E8A5620600              call 004681A0   
:00401EFB 83C410                  add esp, 00000010   
:00401EFE 6A00                    push 00000000   
:00401F00 8D942458010000          lea edx, dword ptr \[esp+00000158\]   
:00401F07 52                      push edx   
:00401F08 8BCE                    mov ecx, esi   
:00401F0A FFD5                    call ebp          <--  程序在这个 Call 中弹出出错对话框，由于上面部分的程序   
                                                    <--  来看，先改尝试改变前面 1）和 2）跳转，如果不行的话，   
                                                    <--  就要跟到这个 Call 中再慢慢研究了。。这是 3）第三个步骤   
                                                      
:00401F0C 6A05                    push 00000005   
  
\* Possible StringData Ref from Data Obj ->"custom errors\_"   
                                  |   
:00401F0E 686C104900              push 0049106C   
:00401F13 8D44245C                lea eax, dword ptr \[esp+5C\]   
:00401F17 68FF000000              push 000000FF   
:00401F1C 50                      push eax   
:00401F1D E87E620600              call 004681A0   
  
  
  \*\*\*\*\*\*\*\*\*  中间的这个部分省略  \*\*\*\*\*\*\*\*\*\*   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00401EDA(C)   
|   
:00401F73 8B7C2444                mov edi, dword ptr \[esp+44\]  <--程序的正确入口   
:00401F77 83C9FF                  or ecx, FFFFFFFF   
:00401F7A 33C0                    xor eax, eax   
:00401F7C F2                      repnz   
:00401F7D AE                      scasb   
  
    根据上下文，bd \*以后，首先在 1）处下个断点，F5 回到 Windows，重新来一次，程序会中断在 1）处，改掉 1）后面的那个跳   
转，发现成功了。那么这个地方就是关键了，记下这个地方的 Code 码：85 c0 0f 84 93 00 00 00 ff d3 6a 06。用 Hex Workshop   
进行修改。   
  
    用 Hex WorkShop（当然啦，你也可以用 HView 什么之类的，我是因为看雪斑竹在破解工具栏中那个 Hex WorkShop 的介绍才一直   
使用它的。。呵呵，因为我一直都在向看雪斑竹学习、一直在向他看齐 :b ）打开 SuperGoo.exe，查找前面说的那个字符串，然   
后有两种修改方法：   
  
    1）改后面的那个 je 00401F73 为 jmp 00401F73   
    2）改 test eax, eax 为 xor eax, eax  <-- 也就是把 85 改成 33 就可以了   
  
我用的是第 2 中方法，嘿嘿（为什么？哎，因为改动少吗，只需改一个字节就可以了。。。哈哈哈哈）   
  
  
  
【最后整理】   
        
      改：85 c0 0f 84 93 00 00 00 ff d3 6a 06   
          ^^   
          33   
  
    你便可以用任何字符进去了，而且运行第一次后，下次运行不再需要输入东东了，可以直接进去了。。  \*\_^  4dd466d0e1e0cc123afbe504da2462ca