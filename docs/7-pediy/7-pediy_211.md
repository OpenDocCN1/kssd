

# 双击记事本自动打开文本

> 原文：[`www.pediy.com/kssd/pediy11/116595.html`](https://www.pediy.com/kssd/pediy11/116595.html)

前几天看了一篇文章,关于 overlay 介绍的,里面说可以实现记事本打开单一文本，所以自己手痒就想试试.  
我的第一步是让记事本程序在打开时，自动读取已经存在的文本文件.  
大家都很清楚双击文本文件程序会自己用记事本打开,这说明这个文件名被当作参数传递给了记事本.  
例如: notepad.exe C:\\QQ600.txt  
  
接下来我们要做的就是获取那个参数的位置,然后修改,实现自己的功能。  
  
ollydbg 载入程序 notepad,选择菜单 Debug>>Attachment，输入参数 C:\\QQ600.txt，提示重新载入程序,参数才会生效。\[不理他,等下面设置了端点后,再重新载入\]  
要想获取参数的位置,必须要对程序下断点,文件读取一般会在 CreateFile,ReadFile 等上下断点  
  
利用快捷键 Ctrl+N,查看程序的函数,看到了 CreateFileW,下面对他断点,在命令栏中输入 bpx CreateFileW，Ctrl+F2 重新载入程序,重新载入程序后 F9 运行，在 01004A07 处程序被断下来，查看一下寄存器与堆栈,你会发现 edi 中存放的值与参数值完全一致.  
那么现在我们要搞清楚的就是 edi 在何时被赋值的  
  
  
010049CE    6A 10                     push 10  
010049D0    FF35 30980001             push dword ptr ds:\[1009830\]  
010049D6    FF15 A4120001             call dword ptr ds:\[<&USER32.PostMessageW>; USER32.PostMessageW  
010049DC    E9 77010000               jmp NOTEPAD-.01004B58  
010049E1    66:391F                   cmp word ptr ds:\[edi\],bx  
010049E4    0F84 B1000000             je NOTEPAD-.01004A9B  
010049EA    57                        push edi  
010049EB    BF 00A90001               mov edi,NOTEPAD-.0100A900                ; UNICODE "C:\\QQ600.txt"  
010049F0    57                        push edi  
010049F1    E8 A3F5FFFF               call NOTEPAD-.01003F99  
010049F6    53                        push ebx  
010049F7    68 80000000               push 80  
010049FC    6A 03                     push 3  
010049FE    53                        push ebx  
010049FF    6A 03                     push 3  
01004A01    68 00000080               push 80000000  
01004A06    57                        push edi  
01004A07    FF15 04110001             call dword ptr ds:\<&KERNEL32.CreateFile>; kernel32.CreateFileW  
  
大家看到了吗?在 010049EB 处对 edi 赋值,edi 的值来源于 0100A900 处  
010049EB    BF 00A90001               mov edi,NOTEPAD-.0100A900                ; UNICODE "C:\\QQ600.txt"  
  
也许大家已经想到了将 0100A900 处的数据改为自己的数据,但是修改完了你会发现数据无法保存，在这里我就不介绍怎么保存数据了.  
0100A900 处修改后无法保存,那么我们就将 0100A900 的地址指到其他可以保存数据的位置.例如 0100874E  
010049EB    BF 00A90001               mov edi,NOTEPAD-.0100A900                ; UNICODE "C:\\QQ600.txt"  
修改后  
010049EB    BF 4E870001               mov edi,NOTEPAD-.0100874E         ; UNICODE "C:\\QQ600.txt"  
  
然后在 0100874E 处写入自己的参数数据,程序调试完成后，双击运行程序,你会发现程序打开后,并没有读取 C:\\QQ600.txt 的数据.  
不要急,程序能够正常运行而没有出现错误,这说明调试没有出错,可能调试的代码没有生效，  
  
Ollydbg 载入调试过的程序,Debug>>Attachment，输入"C:\\QQ600.txt"，对函数 CreateFileW 下断点,F9 运行,程序在 01004A07 处被断下,  
调试过的部分上面有几处跳转,其中有一个是跳过调试代码,我们在他前面下断，我选择 010049C8，然后重新载入程序,F9 运行，在 010049C8 处程序被断下来,现在开始 F8 单步向下分析一下跳转是否实现了  
  
分析后大家,只需让 010049E4 处的跳转不实现就可以了  
。  
  
  
\=====================================================  
  
建议大家自己分析一下程序参数是如何传递到 edi 中的   
在这里留个问题吧，希望能够刺激大家努力学习!  
  
将文本的数据添加到记事本程序的尾部,然后每次打开程序时自动读取尾部的数据

上传的附件
| ![ || 自动读取文本.rar  
 |