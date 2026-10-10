

# EasyCHM

> 原文：[`www.pediy.com/kssd/pediy06/pediy6569.htm`](https://www.pediy.com/kssd/pediy06/pediy6569.htm)

题目：我的还原经历  
  
由于曾经用自己未完全破解（当然在一个偶然才发现）的 EasyCHM 编译了一本定名为 abc.chm，并将相关的源文件痛快地删除了，后来发现后，就顺便把 EasyCHM 破解完全！可惜该 abc.chm 文件内的所有文件都被附加 Unregistered 的东西（“加料料理”，我不太接受），它特征为：每个文件尾部都加上 F1h 长度的固定 16 进制串。因为释放文件数量比较大，如果人工用 16 进制工具进行还原，天啊，所以需要批量处理释放文件，于是到 bbs.pediy.com 求助，谁知不久后论坛就给人攻击了，无奈只好亲自就从反编译功能方面着手调试跟踪。反编译当然是释放文件啦，那么就是要写文件，跟踪发现程序（功能已破解完全，不然反编译还会“加料料理”）反编译中会对每一个被释放文件以每次 1000h 的长度划分到缓冲区（待写），不足 1000h 就认为是该文件的最后一次写操作。  
  
下面跟踪到写文件必经地方（具体地址已忽略）：  
  

> ```
> 代码:
> 00333314   jmp dword ptr ds:\[<&kernel32.WriteFile>\]; // 可从这修改到指定代码实行的地方，如：999900
> 0033331C   push ebp
> 0033331D   mov ebp,esp
> 0033331F   push ecx
> 00333320   push ebx
> 00333321   push esi
> 00333322   push edi
> 00333323   mov esi,ecx
> 00333325   mov edi,edx
> 00333327   mov ebx,eax
> 00333329   mov eax,dword ptr ss:\[ebp+10\]
> 0033332C   movzx edx,word ptr ds:\[ebx+4\]
> 00333330   and edx,eax
> 00333332   cmp eax,edx
> 00333334   jnz short EasyCHM.0033338E
> 00333336   push 0                              ; // lpOverlapped 倘若在指定 FILE\_FLAG\_OVERLAPPED 的前提下打开文件，这个参数就必须引用一个特殊的结构。那个结构定义了一次异步写操作。否则，该参数应置为空
> 00333338   lea eax,dword ptr ss:\[ebp\-4\]
> 0033333B   push eax                            ; // lpNumberOfBytesWritten 实际写入文件的字节数量
> 0033333C   mov eax,dword ptr ds:\[ebx+8\]
> 0033333F   imul esi
> 00333341   push eax                            ; // nNumberOfBytesToWrite 要写入数据的字节数量。
> 00333342   push edi                            ; // lpBuffer 要写入的一个数据缓冲区
> 00333343   mov eax,dword ptr ds:\[ebx\]
> 00333345   push eax                            ; // hFile 文件的句柄
> 00333346   call dword ptr ss:\[ebp+C\]           ; // 实际是 Call 00333314 ; kernel32.WriteFile
> 00333349   test eax,eax             ; // 写文件成功则返回 1
> 0033334B   jnz short EasyCHM.0033335E
> 0033334D   call <jmp.&kernel32.GetLastError>
> 
> ```
> 
>   

  
  
WriteFile 函数堆栈参考：  
  

> ```
> 代码:
> 0081E760    00333349  /CALL 到 WriteFile 来自 EASYCHM.00333346
> 0081E764    000000C8  |hFile = 000000C8 (region)
> 0081E768    0081E7E8  |Buffer = 0081E7E8
> 0081E76C    00001000  |nBytesToWrite = 1000 (4096.)
> 0081E770    0081E784  |pBytesWritten = 0081E784
> 0081E774    00000000  \\pOverlapped = NULL
> 0081E778    01281D0C  ASCII "<br><hr><br> This file is decompiled from a .CHM file <br>by an UNREGIST>......"
> 
> ```
> 
>   

  
  
由于发此贴时还原文件走运，所以未发觉代码有问题（可能出现的情况未作考虑和 cmps 指令理解不佳），现在修正如下：  
  
实行代码 Begin:  
  

> ```
> 代码:
> 00999900   pushfd                          ; // 压入标志  --\\
> 00999901   pushad                          ; // 压入寄存器--/ 共 9 DWORDS
> 00999902   mov esi,dword ptr ss:\[esp+3C\]   ; // 放入取得 Unregistered 字符串解码地址
> 00999906   mov edi,dword ptr ss:\[esp+2C\]   ; // 取得待写 buffer 首地址
> 0099990A   mov edx,dword ptr ss:\[esp+34\]   ; // 取写文件大小（buffer）的实际地址
> 0099990E   mov ecx,dword ptr ds:\[edx\]      ; // 取写文件大小的实际地址
> 00999910   cmp ecx,0F1                     ; // 假设不幸，Unregistered 字符串被分别分割到两个缓冲区（最后一个和倒数第二个）。
> 00999916   jb short EASYCHM.00999950       ; // 就要让它暂停，将当前的文件名在 eax 中给出（需手工查找修复，但为数应该不会多的，可能就那么一二三个。）
> 00999918   add edi,ecx                     ; // 调整到待写缓冲区的底部
> 0099991A   sub edi,0F1                     ; // 缓冲区底部指针 上移 到 指定位置
> 00999920   mov ecx,0F1                     ; // 从指定的位置比较
> 00999925   xor eax,eax                     ; // 清 eax
> 00999927   xor ebx,ebx                     ; // 清 ebx
> 00999929   cmps byte ds:\[esi\],byte es:\[edi\]; // 此指令要理解其含义，一开始搞错了，小心！
> 0099992A   sete al                         ; // 相等让 al 为 True
> 0099992D   add bl,al                       ; // 累计测试结果
> 0099992F   inc esi                         ; // 调整到下一字节
> 00999930   inc edi                         ; // 调整到下一字节
> 00999931   dec ecx                         ; // 
> 00999932   jnz short EASYCHM.00999929      ; // 循环直到 ecx 为 0
> 00999934   cmp bl,0F1                      ; // 测试相等的字节数是否为 F1
> 00999937   jnz short EASYCHM.00999948     ; // 不相等则正常处理
> 00999939   sub dword ptr ss:\[edx\],0F1      ; // 重写 buffer Size
> 00999940   sub dword ptr ss:\[esp+30\],0F1   ; // \*\*\*\*\*\*重新写要输出 Size 关键\*\*\*\*\*\*
> 00999948   popad                           ; // 还原标志
> 00999949   popfd                           ; // 还原寄存器
> 0099994A   jmp dword ds:\[kernel32.WriteFile; // 恢复执行原指令
> 00999950   mov eax,dword ptr ss:\[esp+8D\]   ; // 当前被写的文件名地址填到 eax ，执行后 have a look eax
> 00999954   jmp short EASYCHM.00999948      ; // 在此指令上下断点，运行中不能批量修复的文件则会在此中断，需手工到释放目录查找 eax 指向的文件修复。
> 
> ```
> 
>   

  
  
实行代码 End:  
  
代码已经写好了，现在就来执行一下，谁知只成功释放了一个还原文件，就进入了异常陷井，进入了一个无法继续的异常。重新试了好几次都这样，于是从地址 00333349 下断，一直跟踪发现这条指令有问题：  
  

> ```
> 代码:
> 00333379   cmp esi,dword ptr ss:\[ebp\-4\]    ; // 陷井：如果你更改了的实输出 Size ，则判断属于异常错误！esi 为原输出 Size，\[ebp-4\]为你实际输出 Size
> 0033337D   je short EasyCHM.0033339D //不跳。。。哈哈
> 
> ```
> 
>   

  
  
修改后重来，按确定 ---- 就这样，我的 abc.chm 成功地还原了！:D 4dd466d0e1e0cc123afbe504da2462ca