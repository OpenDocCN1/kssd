

# 飞鸽传书简易消息尾巴制作记录(已经添加编译好的 DLL 和相关工具以及源码)

> 原文：[`www.pediy.com/kssd/pediy12/127614.html`](https://www.pediy.com/kssd/pediy12/127614.html)

制作目的:学习娱乐,不以耍流氓为目的  
  
飞鸽版本:飞鸽传书 V2.09  
  
使用工具:OD VS2005  
  
连接地址:http://bbs.pediy.com/  
  
功能概述:替换飞鸽传书发送的所有消息的前几个字符,达到娱乐的效果。  
  
整体思路:要向达到上述的娱乐效果首先要在内存中定位到存放发送消息的地址,然后修改之。思路很简单,实现也不复杂。  
  
实现步骤:  
  
1.调试,确定待发送消息的内存地址,传统方法下断 sendto 方法,然后根据栈回溯 慢慢追到显示消息明文的地方,很通用,很麻烦,好在程序不大,如果是个网游就得找一阵子了.  
  
这里换一个方法查找消息明文:  
  
![](img/7e26c1d66dcc0402f1ae81915c2d9a0b.jpg)  
图-1  
  
飞鸽在编写消息的对话框中有封装和上锁两个选项如(图-1 所示)是用选择框形式提示的,那么做一个合理的猜测:处理消息前会对这两个对话框状态做检测,按照猜想在飞鸽传书的导入表中找到了检测对话框状态的 API 函数 USER32.IsDlgButtonChecked,对他下断,单步走几下就发现了在内存中定位存放消息明文地址(如图-2,图-3 所示)  
  
  
![](img/9f701e4c7614d7d2ea2651b4edf99180.jpg)  
图-2  
  
![](img/a555c9670131991ee502efb881b31789.jpg)  
  
图-3  
  
其中 Eax 存放的是消息的地址,而 eax 的值是 esi 计算而来,现在明文找到了娱乐任务完成了一半,为了能在接下的编码任务中让程序自动找到明文,有两个办法第一个用外挂的思路,继续跟踪 esi 的值直到找到一个重启程序都不会变化的基质,(类似于全局变量)另外一个办法是在这个位置直接取得 esi 的值经过一步简单计算获取存放消息明文的地址,这里采用后者也就是 Hook 的方式,在图-2 中 0x4133D3 的位置跳转到自实现的方法中,这是可以轻松获取各个寄存器的值找到明文消息的地址。  
  
   
  
2.编码  
 1 // InIPmsg.cpp : 定义 DLL 应用程序的入口点。  
 2 //  
 3   
 4 #include "stdafx.h"  
 5 LPSTR lpiwant = "whxright";  
 6 VOID ChangeMemory()  
 7 {  
 8   
 9     \_\_asm{  
10             push ebx  
11             push lpiwant  
12             mov edi,esi           //获取 esi 的值  
13             add edi,0xd4        //获取明文消息地址  
14             mov esi,lpiwant        //获取要替换的内容  
15             movsb                    //替换  
16             movsb  
17             movsb  
18             mov eax,0x00404b03  //由于修改导致原来的调用被覆盖现在替它调用下保证原流程正常  
19             call eax  
20             mov eax,0x004133da    //返回地址  
21             pop esi                          //修复堆栈平衡  
22             pop edi  
23             jmp eax                         //该做的做完了跳回去保证原流程正常  
24     }  
25   
26 }  
27   
28 BOOL APIENTRY DllMain( HMODULE hModule,  
29                        DWORD  ul\_reason\_for\_call,  
30                        LPVOID lpReserved  
31                      )  
32 {  
33     if (ul\_reason\_for\_call == DLL\_PROCESS\_ATTACH) {  
34         MessageBox(NULL,L"XX",L"XXX",MB\_OK);  
35         \_\_asm{  
36                 mov eax,ChangeMemory                //自定义方法的地址  
37                 add eax,3                           //跳过编译器生成的堆栈平衡代码  
38                 sub eax,0x4133d9                    //计算跳转地址  
39                 mov ebx,0x4133d3    //修改原来的代码,让它跳到自定义函数 ChangeMemory  
40                 mov word ptr \[ebx\], 0xe990  
41                 mov    dword ptr \[ebx+2\],eax  
42                 mov byte ptr\[ebx+6\],0x90  
43         }  
44         return TRUE;  
45     }  
46   
47 }  
  
  
   
  
注释很详细  
  
大致记录下,被修改后的程序关键点(如图-4 所示:)  
  
![](img/cbd7bb0e225946917ef2f45455809424.jpg)  
  
图-4  
  
图-2 中的一个方法调用已经被 DllMain 修改成了上面的样子,原来的代码没有了程序肯定报错所以在我们的自定义方法中要完成图-2 中对 0x00404b03 方法的调用、对消息的修改、修复堆栈、还有跳回原流程这 4 个功能才能达到正常娱乐的目的,自定义函数 InIPmsg.01991003 在 OD 中的样子(如图-5 所示:)对应代码中的 ChangeMemory 方法,  
  
![](img/2c4ebf40bd129f9a86aa52b5e69fcd6e.jpg)  
  
图-5  
  
嗦了不说了。。。  
  
注意事项:  
  
(1)源码编译完成后是个 DLL 文件,需要输入到飞鸽进程才可以生效。  
  
(2)编码仓促没有实现修改飞鸽.text 段写入属性,如要实验请使用 stud\_PE 等工具手动修改飞鸽.text 的 MEM\_WRITE 属性  
  
(3)由于有硬编码的存在,导致非本地实验可能不成功,请自行调试  
  
(4)编译时请将内联汇编的注释去掉,否则可能出错  
  
(5)代码写的很烂,Bug 多多主要记录思路请轻砸…..  
  
效果图:  
  
发送消息的前 3 个字符被替换为"whx"  
  
![](img/707d246f55fc62fe94bb0808956c787a.jpg) ![](img/09f10e9c825980e7a7980899ff5d5918.jpg)

上传的附件
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || Syringe.rar  
 |
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || InIPmsg.rar  
 |