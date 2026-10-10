

# EasyCHM

> 原文：[`www.pediy.com/kssd/pediy06/pediy6852.htm`](https://www.pediy.com/kssd/pediy06/pediy6852.htm)

呵呵，不知不觉 EasyCHM 已经升级到 1.9.7.320（今天才知道，就谈论这个版本），增加了一些我还不会用的功能：  
  
v1.97 Build 320:    ( 11-22-04 )  
          
  + Bug fixed : Sometimes the project file will be load incorrectly, we fixed this bug.   
                                            
  
这里我可不是说要破解这个程序，如果你用过这个程序（选项布局比较惹人喜爱），你就会发现每当你用它编译一个 chm 文件时，它都会（不管是否存在程序目录）先生成一个必备的文件 hhc.exe，而当它完成编译时，却留下它置之不理（虽然仅 50k 大小，但我们反安装时就显得不干净），还有系统 temp 文件夹编译留下的文件--涉及破解限制不在论述范围。  
  
当然少不了，先脱壳，upx 壳，简蛋（无非是找 pushad 和 popad jmp xxxx 后 Dump），修正入口用 ImportREC 修复获取的 imt，脱壳就完事了。  
程序一运行就报“病毒”，呵呵，它需要你要“安抚”它！--> 去掉 CRC  
  
CRC 脱轨 简便方案：  
在主程序代码段搜索以下命令（你会发现仅 2 条）  
  
cmp dword \[ebp\-0A\],0  
  
修改它们紧接着的第 1 个跳转类型指令为 Nop （可以节省指令运行的开支，当然其中一条不这样改，还可以改得更省的就不说了，有兴趣自己 check 吧）  
  
好了，现在程序能够工作了，但仍是未注册版本！  
  
我现在要给程序添加个小功能：编译完成后点关闭则删除 hhc.exe 文件  
  
使用一次，就可以知道我们最后总要点击 “关闭”按钮  
  
好了，用资源工具（ResScope）看一下个“关闭”按钮的名称  
  
找到“TGHCUSTOMPROCESSFORM1”，再是找到“BtnClose”  
  
好了，我们要找出它的对应事件代码地址  
  
OD 载入主程序，光标定位到代码段的起始地址，并转存到内存窗口，进行二进制搜索“BtnClose”，留意它其后是否紧接有“Click”，OK，发现在 50CF21，上一点是字符标识长度，再向上一点就是事件代码地址了，就是它：50D16C  
  
come on here:  
  
0050D16C   mov byte ptr ds:\[eax+2F8\],1    ; // \[生成 Chm\] -> here is 关闭按钮事件  
0050D173   mov byte ptr ds:\[eax+318\],1  
0050D17A   call EASYCHM.0047A940  
0050D17F   retn  
  
我从 0050D16C 开始进行补丁：  
  
0050D16C   call EASYCHM.00602070          ; // \[生成 Chm\] -> here is 关闭按钮事件 src: mov byte ptr ds:\[eax+2F8\],1  
0050D171   nop  
0050D172   nop  
0050D173   mov byte ptr ds:\[eax+318\],1  
  
601070 是我处理过的地址，你可以才个别的好用的地址  
  
我们需要哪些 API 呢？很简单就 kernel32.GetModuleFileNameA 和 kernel32.DeleteFileA  
找出它们在程序的地址就好办了，刚好程序有，补丁如下：  
  
00602070   push ebp                                  ; // 保存栈  
00602071   mov ebp,esp                               ; // 保存 esp  
00602073   pushad                                    ; // 保存寄存器  
00602074   add esp,-120                              ; // 分配堆栈空间  
0060207A   push 105                                  ; // 定义缓冲区大小  
0060207F   lea eax,dword ptr ss:\[esp+4\]              ; // 缓冲区起始地址  
00602083   push eax  
00602084   push 0  
00602086   call 00401378                             ; kernel32.GetModuleFileNameA  
0060208B   mov ebx,esp                               ; //程序文件名路径  
0060208D   add ebx,eax  
0060208F   cmp byte ptr ds:\[ebx\],5C  
00602092   je short EASYCHM.00602097  
00602094   dec ebx  
00602095   jmp short EASYCHM.0060208F  
00602097   inc ebx                                   ; // 下面进行链接  
00602098   mov dword ptr ds:\[ebx\],2E636868           ; // "hhc."  
0060209E   add ebx,4  
006020A1   mov dword ptr ds:\[ebx\],657865             ; // "exe"  
006020A7   push esp  
006020A8   call 00407294                             ; kernel32.DeleteFileA  
006020AD   add esp,120                               ; // 下面是恢复上面的保存动作  
006020B3   popad  
006020B4   mov esp,ebp  
006020B6   pop ebp  
006020B7   mov byte ptr ds:\[eax+2F8\],1               ; // 执行补丁失去的指令  
006020BE   retn                                      ; // 返回原处  
  
OK，现在保存所有，运行编译并关闭，验证通过收工，后来又想到应该找到按钮切换的事件才够准，不过又没时间玩了；最近看见 PEdiy 论坛很多猛人忙于发贴，真是佩服，谢谢你观看无聊之极的叙述...![](img/fba31feea7bebf005cc8ab366071a14f.jpg)   
  
written by askformore  
  11.22.2004 非  4dd466d0e1e0cc123afbe504da2462ca