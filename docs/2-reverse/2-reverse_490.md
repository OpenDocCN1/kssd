

# 逆向 C++之一 基本数据类型

> 原文：[`www.pediy.com/kssd/pediy10/78163.html`](https://www.pediy.com/kssd/pediy10/78163.html)

【文章标题】: 逆向 C++之一 基本数据类型  
【文章作者】: kanghtta  
【作者邮箱】: kanghtta@hotmail.com  
【作者主页】: http://kanghtta.cublog.cn  
\--------------------------------------------------------------------------------  
【详细过程】  
     
      
    
  #include<iostream>  
  using namespace std;  
    
  int main()  
  {  
    int a=2,  
      b=4;  
    float  c=2.0f;  
    bool  v=1;  
    char  x=2;  
    return  0;  
  }  
    
   我们知道，上面定于的几个基本数据类型的局部变量，一共占用 8+4+1+1 = 14 字节；  编译链接后生成 exe 文件，用 OD 载入之，  
   由于 main 函数实际要压入三个参数，故在 GetCommandlineA 函数下不远处可找到 main 函数的调用；  
    
  004083FB  |.  E8 E05F0000   call    0040E3E0  
  00408400  |.  FF15 60F14300 call    dword ptr \[<&KERNEL32.GetCommand>; \[GetCommandLineA  
  00408406  |.  A3 A4E64300   mov     dword ptr \[43E6A4\], eax  
  0040840B  |.  E8 B05D0000   call    0040E1C0  
  00408410  |.  A3 8CCC4300   mov     dword ptr \[43CC8C\], eax  
  00408415  |.  E8 96580000   call    0040DCB0  
  0040841A  |.  E8 41570000   call    0040DB60  
  0040841F  |.  E8 7C430000   call    0040C7A0  
  00408424  |.  8B0D D8CC4300 mov     ecx, dword ptr \[43CCD8\]  
  0040842A  |.  890D DCCC4300 mov     dword ptr \[43CCDC\], ecx  
  00408430  |.  8B15 D8CC4300 mov     edx, dword ptr \[43CCD8\]  
  00408436  |.  52            push    edx  
  00408437  |.  A1 D0CC4300   mov     eax, dword ptr \[43CCD0\]  
  0040843C  |.  50            push    eax  
  0040843D  |.  8B0D CCCC4300 mov     ecx, dword ptr \[43CCCC\]  
  00408443  |.  51            push    ecx  
  00408444  |.  E8 C68BFFFF   call    0040100F  ；main 函数入口地址  
  00408449  |.  83C4 0C       add     esp, 0C  
    
    
    
  我们知道 call 指令执行的操作是： push eip 即将子程序的返回地址（也就是 call 指令的下一条指令的地址）存入堆栈中；  
  F7 跟进，在堆栈窗口中的 esp 此时指向 00408449，也就是 call 0040100F 的后条指令的地址  
  ESP ==>  > 00408449  返回到 class.<模块入口点>+0E9 来自 class.0040100F  
    
    
    
    
    
  00401030  /> \\55            push    ebp                    ；main 的反汇编代码  
  00401031  |.  8BEC          mov     ebp, esp  
  00401033  |.  83EC 54       sub     esp, 54  
  00401036  |.  53            push    ebx  
  00401037  |.  56            push    esi  
  00401038  |.  57            push    edi  
  00401039  |.  8D7D AC       lea     edi, dword ptr \[ebp-54\]  
  0040103C  |.  B9 15000000   mov     ecx, 15  
  00401041  |.  B8 CCCCCCCC   mov     eax, CCCCCCCC  
  00401046  |.  F3:AB         rep     stos dword ptr es:\[edi\]  
  00401048  |.  C745 FC 02000>mov     dword ptr \[ebp-4\], 2        ； a=2  
  0040104F  |.  C745 F8 04000>mov     dword ptr \[ebp-8\], 4        ；b=4;  
  00401056  |.  C745 F4 00000>mov     dword ptr \[ebp-C\], 40000000 ；c=2.0f;  
  0040105D  |.  C645 F0 01    mov     byte ptr \[ebp-10\], 1        ；v=1;  
  00401061  |.  C645 EC 02    mov     byte ptr \[ebp-14\], 2        ；x=2;  
  00401065  |.  33C0          xor     eax, eax                   ；return  0;  
  00401067  |.  5F            pop     edi  
  00401068  |.  5E            pop     esi  
  00401069  |.  5B            pop     ebx  
  0040106A  |.  8BE5          mov     esp, ebp  
  0040106C  |.  5D            pop     ebp  
  0040106D  \\.  C3            retn  
    
  我们知道，c 函数调用的参数入栈顺序是从右到左，而在汇编中学过，ebp 的默认段也是 ss 段，也就是说，ebp 寄存器是配合  
  堆栈段使用的，而他往往被用来作为存取局部变量的指针基址，同时也起着保存 原始 esp 指针的作用，以便在函数执行完后能  
  让 ret 指令从堆栈中读到正确的返回地址； 我们看到 ，在子程序开始时有一 push ebp  mov ebp，esp 指令，  
  而在结束时有 mov esp，ebp  pop ebp 指令；它们成对出现，在 80386 中也可以用 leave 指令来代替后两条指令；  
  指令，他们成对出现，  
    
  这里有一点我不大明白，局部变量一共只需要 14 字节就能保存，加上 ebx，esi，edi 等寄存器也才 24 字节，而 C++编译器却初始化  
  了 54 字节，并用 CCCCCCCC 初始化堆栈空间，而当 mov esp ebp 执行后，堆栈被清空，可以说有点浪费空间了，但不知道多初始化  
  这一部分空间有什么用途；  
    
  F8 跟到初始化完后，堆栈中数据如下；  
    
  EBP-54   > CCCCCCCC  
  EBP-50   > CCCCCCCC  
  EBP-4C   > CCCCCCCC  
  EBP-48   > CCCCCCCC  
  EBP-44   > CCCCCCCC  
  EBP-40   > CCCCCCCC  
  EBP-3C   > CCCCCCCC  
  EBP-38   > CCCCCCCC  
  EBP-34   > CCCCCCCC  
  EBP-30   > CCCCCCCC  
  EBP-2C   > CCCCCCCC  
  EBP-28   > CCCCCCCC  
  EBP-24   > CCCCCCCC  
  EBP-20   > CCCCCCCC  
  EBP-1C   > CCCCCCCC  
  EBP-18   > CCCCCCCC  
  EBP-14   > CCCCCC02   ；char x  
  EBP-10   > CCCCCC01   ；bool v  
  EBP-C    > 40000000   ；float c  
  EBP-8    > 00000004   ；int b  
  EBP-4    > 00000002   ；int a  
    
    
    
    
\--------------------------------------------------------------------------------  
【经验总结】  
    我们知道，了解子程序调用的栈变化情况，是利用栈溢出的基础； 故，通过一个简单的 C 程序来加深对 c 语言的理解；也  
  为进一步学习 shellcode 打下基础；   
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
  
                                                       2008 年 12 月 06 日 11:36:52