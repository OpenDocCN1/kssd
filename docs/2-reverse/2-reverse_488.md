

# 关于子函数参数与堆栈的学习笔记

> 原文：[`www.pediy.com/kssd/pediy10/75884.html`](https://www.pediy.com/kssd/pediy10/75884.html)

【文章标题】: 关于子函数参数与堆栈的学习笔记  
【文章作者】: eASYCODe  
【作者主页】: http://blog.sina.com.cn/77muyulong  
【作者声明】: 菜菜学习中的一点点收获，错误请诸大侠指正…  
\--------------------------------------------------------------------------------  
【详细过程】  
  玩着玩着 RA3 突然电脑重启 好扫兴 于是决定用玩的精力来记录最近学到的一点东西 故撰写此文…  
    
  我一边做一边写 所以有的时候也许看起来逻辑很混乱 包涵哈 仅是笔记罢了~  
    
  上个星期在拜读了 N 篇有关于 PEDIY 的教程之后终于决定自己下手了 选的是某个小游戏的特典包 打算给它添加几个快捷键   
  注入自己写的 DLL 但是发现在调用时有些问题 程序会出错退出 故而认真学习了有关于子函数参数与堆栈的内容 自  
  以为受益匪浅  
  首先要说明的是 这个笔记中所说的程序代码分别是 C ASM 和分汇编 对比进行 应该更有说服力  
    
  先来看这样一个程序  
    
  C 代码  
    

代码:

```
  int Fanc(int a,int b,int c)
  {
  int d,e,f;
  d=a;
  e=a+b;
  f=a+b+c;
  return f;
  }
  
  
  main()
  {
  int x=1,y=2,z=3,m;
  m=Fanc(x,y,z);
  
  }

```

  
很简单的一个子程序调用的例子 我们来把它反汇编后看看 我用的 VC++6.0 只摘录 C 代码所对应的反汇编代码  
代码:

```
  ;C 反汇编代码
  
  ;main()
  ;{
  00401088                 |.  C745 FC 01000000 mov dword ptr ss:\[ebp-4\],1
  0040108F                 |.  C745 F8 02000000 mov dword ptr ss:\[ebp-8\],2
  00401096                 |.  C745 F4 03000000 mov dword ptr ss:\[ebp-C\],3
  0040109D                 |.  8B45 F4          mov eax,dword ptr ss:\[ebp-C\]
  004010A0                 |.  50               push eax
  004010A1                 |.  8B4D F8          mov ecx,dword ptr ss:\[ebp-8\]
  004010A4                 |.  51               push ecx
  004010A5                 |.  8B55 FC          mov edx,dword ptr ss:\[ebp-4\]
  004010A8                 |.  52               push edx
  004010A9                 |.  E8 57FFFFFF      call proj.00401005                ;Fanc()
  ;{
 
  00401005                     $ /E9 16000000        jmp proj.00401020
 
  
  
  00401020                 /> \\55               push ebp
  00401021                 |.  8BEC             mov ebp,esp
  00401023                 |.  83EC 4C          sub esp,4C
  00401026                 |.  53               push ebx
  00401027                 |.  56               push esi
  00401028                 |.  57               push edi
  00401029                 |.  8D7D B4          lea edi,dword ptr ss:\[ebp-4C\]
  0040102C                 |.  B9 13000000      mov ecx,13
  00401031                 |.  B8 CCCCCCCC      mov eax,CCCCCCCC
  00401036                 |.  F3:AB            rep stos dword ptr es:\[edi\]
  00401038                 |.  8B45 08          mov eax,dword ptr ss:\[ebp+8\]
  0040103B                 |.  8945 FC          mov dword ptr ss:\[ebp-4\],eax
  0040103E                 |.  8B4D 08          mov ecx,dword ptr ss:\[ebp+8\]
  00401041                 |.  034D 0C          add ecx,dword ptr ss:\[ebp+C\]
  00401044                 |.  894D F8          mov dword ptr ss:\[ebp-8\],ecx
  00401047                 |.  8B55 08          mov edx,dword ptr ss:\[ebp+8\]
  0040104A                 |.  0355 0C          add edx,dword ptr ss:\[ebp+C\]
  0040104D                 |.  0355 10          add edx,dword ptr ss:\[ebp+10\]
  00401050                 |.  8955 F4          mov dword ptr ss:\[ebp-C\],edx
  00401053                 |.  8B45 F4          mov eax,dword ptr ss:\[ebp-C\]
  00401056                 |.  5F               pop edi
  00401057                 |.  5E               pop esi
  00401058                 |.  5B               pop ebx
  00401059                 |.  8BE5             mov esp,ebp
  0040105B                 |.  5D               pop ebp
  0040105C                 \\.  C3               retn
  
  
  ;}
  004010AE                 |.  83C4 0C          add esp,0C
  004010B1                 |.  8945 F0          mov dword ptr ss:\[ebp-10\],eax
 
  

```

  
  这只是一个节选 因为 VC++编译出来的程序中有很多无用代码 所以仅仅是把我们有用的 所对应的代码找出来而已 看起来多  
  少有点乱 我们把它写成 ASM 程序试一下~  
  因为刚刚开始 WIN32asm 之路 所以写的代码会很难看 包涵包涵~  
  
  
ASM 代码  
  
代码:

```
.386
.model flat,stdcall
option casemap:none
include Windows.inc
include kernel32.inc 
includelib kernel32.lib
.data
x  dw  1
y  dw  2
z  dw  3
.data?
m  dd  ?
.code 
start:
    push 0
    call GetCommandLine
    push eax
    push 0
    invoke GetModuleHandle, NULL
    push eax
    call WinMain
    invoke ExitProcess, eax
Fanc proc ma:DWORD,mb:DWORD,mc:DWORD 
  
  LOCAL d
  LOCAL e
  LOCAL f
  mov ecx,ma
  mov d,ecx
  add ecx,mb
  mov e,ecx
  add ecx,mc
  mov f,ecx
  mov eax,f
  ret
 
Fanc endp
WinMain PROC hInst:DWORD, hInstPrev:DWORD, szCmdLine:DWORD, nCmdShow:DWORD
push z
push y
push x
call Fanc
mov m,eax
ret
WinMain ENDP
end start

```

  
  在 RadASM\_2.2.1.1 下编译连接通过  之后我们再把它反汇编来看一下  
  
代码:

```
  ;asm 反汇编代码
  
  0040103F                           |.  66:FF35 04304000 push word ptr ds:\[403004\]                    ; /Arg3 = 00000003
  00401046                           |.  66:FF35 02304000 push word ptr ds:\[403002\]                    ; |Arg2 = 00000002
  0040104D                           |.  66:FF35 00304000 push word ptr ds:\[403000\]                    ; |Arg1 = 00000001
  00401054                           |.  E8 C4FFFFFF      call proj.0040101D                           ; \\proj.0040101D
  ;{
  
  
  0040101D                           /$  55               push ebp
  0040101E                           |.  8BEC             mov ebp,esp
  00401020                           |.  83C4 F4          add esp,-0C
  00401023                           |.  8B4D 08          mov ecx,dword ptr ss:\[ebp+8\]
  00401026                           |.  894D FC          mov dword ptr ss:\[ebp-4\],ecx
  00401029                           |.  034D 0C          add ecx,dword ptr ss:\[ebp+C\]
  0040102C                           |.  894D F8          mov dword ptr ss:\[ebp-8\],ecx
  0040102F                           |.  034D 10          add ecx,dword ptr ss:\[ebp+10\]
  00401032                           |.  894D F4          mov dword ptr ss:\[ebp-C\],ecx
  00401035                           |.  8B45 F4          mov eax,dword ptr ss:\[ebp-C\]
  00401038                           |.  C9               leave
  00401039                           \\.  C2 0C00          retn 0C
  
  
  ;}
  
  00401059                           |.  A3 08304000      mov dword ptr ds:\[403008\],eax
  

```

  
  这下清楚多了吧 基本上连动态跟踪都不用了 asm 的确是很清楚的语言哦~  
  
  ;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  ;.:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::.  
  
  好了 估计到这里旁观者已经不知道我在做什么了 其实我就是在举例子 真正的收获在后面 因为发现子函数 call 大多是通过  
  堆栈来传递参数的(好像我还没见过其他方法的)所以我决定好好学学这里 弄明白计算机调用子函数的原理  
  
  在查阅了网上的很多帖子后 才弄清楚 原来子函数是通过寄存器 ebp 来取参数和局部变量的 ebp+X 是参数 第一个参数是 ebp+8  
   为什么不是+4 因为 ebp+4 为函数的返回地址 在 call 的时候压入 第二个是 ebp+C 等等 而 ebp-X 则为局部变量 ebp-4 为第一个   
  ebp-8 为第二个 以此类推~~我说高手们怎么一看函数就能猜出参数信息什么的 原来是酱紫哒  
  之搞清楚这一点还不能算结束 还要明白子函数调用的原理 esp 寄存器永远指向栈顶 所以调参的重任才交给了 ebp 还有就是  
  eip 指向执行指令的内存地址  
  
  于是 总结子函数调用过程为请对照前面 ASm 反汇编  

引用:   首先通过父函数的 push 指令将子函数所用到的参数压进堆栈  
    
  之后将 call 指令的下一行内存地址即子函数的返回地址压栈  
    
  将子函数的入口内存地址传 eip 从而改变程序运行流程 到子函数  
    
  进入子函数后 push ebp 为保存 ebp 寄存器  
    
  mov ebp,esp 为把传参数的大任交给 ebp 负责  
    
  sub esp,0XX(或是 add esp,-0XX)为在堆栈里给局部变量分配空间  
    
  。  
  。  
  。  
    
  mov esp,ebp 要回"全力"esp 指回原位同时释放局部变量空间  
    
  pop ebp 这个时候也是结束 ebp 保护的时候 把刚才压进去的 ebp 弹出来~  
    
  pop eip ebp 弹出来了 再弹就是返回地址了 自然弹到 eip  
    
  好了 程序回到了附程序 call 的下一行 这样就结束了？？不 还没有 参数的空间还没有释放掉  
    
  add esp,0xXX 释放掉参数所占用的空间(有的编译器是在子函数中完成释放的)  
   

  
  OK 这样一个子函数就调用结束了 其实就是些最基础的指令在操纵者程序  
  
  注意在 asm 反汇编中的 leave 指令 实际上是 mov esp.ebp  pop ebp 的意思 相对的是 entry 指令  
  
  把子函数的调用过程在写几遍 加强记忆(当然主要还是对应几种不同的指令)  
  
代码:

```
  
  
  push a                                          \\
  push b                                          |
  push c                                          |==invoke
  push eip+5                  \\                   |
                              |==call 函数入口    |
  mov eip,eip+5+call 的操作数  /                   /
  {
  push ebp          \\
  mov ebp,esp       |==entry 0xXX,0
  sub esp,0xXX      /
  
  .
  .
  .
  
  mov esp,ebp       \\
                    |==leave
  pop ebp           /
  pop eip           |==ret                \\
  }                                       |==ret 0xXX
  add esp,0xXX                            /
  
  

```

  
  ;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  ;.:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::.  
  下面记录每一步后堆栈的情况  
  
代码:

```
  
  push a
  ----
  |a
  ----
  ;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
  push b 
  ----
  |b
  ----
  |a
  ----
  ;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
  push c 
  ----
  |c
  ----
  |b
  ----
  |a
  ----
  ;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
  call 函数入口
  ----
  |返回地址
  ----
  |c
  ----
  |b
  ----
  |a
  ----
  ;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
  push ebp
  ----
  |ebp
  ----
  |返回地址
  ----
  |c
  ----
  |b
  ----
  |a
  ----
  ;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
  mov ebp,esp
  不变
  ;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
  sub esp,0xC
  ----
  |
  ----
  |
  ----
  |
  ----
  |ebp
  ----
  |返回地址
  ----
  |c
  ----
  |b
  ----
  |a
  ----
  ;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
  leave
  ----
  |返回地址
  ----
  |c
  ----
  |b
  ----
  |a
  ----
  ;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
  ret
  ----
  |c
  ----
  |b
  ----
  |a
  ----
  ;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
  add esp,0xXX 
  完全释放
  ;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

```

  
  写累了 基本上这些就是对子函数调用和堆栈的学习笔记了 写得比较乱 希望对和我一样的新手有所帮助 至少可以不用像我  
  一样在去翻阅大量资料 google 大量文章了 呵呵 8)  
  
\--------------------------------------------------------------------------------  
  
  
                                                       2008 年 11 月 02 日 3:40:48