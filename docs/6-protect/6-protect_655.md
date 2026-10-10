

# HProtect 虚拟指令初步的分析与分析方法

> 原文：[`www.pediy.com/kssd/pediy12/130775.html`](https://www.pediy.com/kssd/pediy12/130775.html)

![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")最近被几个朋友拉去玩征途 2。打了一段时间人就开始想偷懒了。这游戏最多只有三开。于是乎想自己动手丰衣足食。  
今天搭建了一下调试环境开搞。  
icesword 看了一下表面看来搞了  
NtOpenProcess  
NtOpenThread  
NtProtectVirtualMemory  
NtQueryAttributesFile  
NtTerminateProcess  
NtWriteVirtualMemory  
以上函数。二话不说。上来直接在 ntdll!KiFaseSystemCall 处往下找到 call ebx 这句下断点 "if eax==7a" 我这里 7a 是 ntopenprocess  
单步进入虚拟机  
8187ebc0 push 819042e8  
8187ebc5 jmp 818a8ab6--|  
                    <--|  
818a8ab6 pushad ;保存现场  
818a8ab7 pushfd ;保存现场  
818a8ab8 mov esi,\[esp+24\] ;等于 eip = mem 初始化虚拟机的 eip  
818a8abc mov ebp,esp  
818a8abe sub esp,00000180 ;申请一块局部空间  
818a8ac4 mov edi,esp    ;edi 初始化寄存器,edi 是一块内存可以看作寄存器基地址  
818a8ac6 cld  
818a8ac7 mov ebx,8186a000 ;opcode 对应处理过程表基地址  
  
\_\_\_818a8acc: lodsb 装入第一条指令到 al 同时 esi +1 也就是开始执行指令,这里读取的是指令 opcode 通过这个 code  
计算出一个处理过程并且跳转过去  
818a8acd movzx eax,al 只保留最低位,保证指令是一个字节。也可以理解为整个指令 opcode 编码在 FF 范围内  
818a8ad0 mov eax,\[eax\*4+ebx+003e8da\] ;计算当前指令处理过程  
818a8ad7 lea eax,\[eax+ebx\]  ;得到最终跳转地址  
818a8ada jmp eax    ;转到处理代码  
以下是 6b 指令的处理过程，6b 指令总长度 2 字节第一条指令是 6b 1e  
6b 操作码 1E 寄存器索引号  
818a8810 xor eax,eax  
818a8812 lodsb  ;再读取一个字节 1e  
818a8813 push dword ptr \[ebp\]    ;模拟 pop  
818a8816 add ebp,04      ;模拟 pop  
818a8819 pop dword ptr \[eax\*4+edi\]  ;模拟 pop 放入 1e 寄存单元 总的翻译为 pop \[eax\*4+edi\] = pop \[1e\*4+edi\]  
本条指令模拟完毕 jmp 回原处继续处理其他指令   
818a881c jmp \_\_\_818a8acc:  
  
以下是解读的部分指令。  
6b 10 = pop \[10\]  ; add ebp,4  
6a 15 1a = add DWORD PTR \[1a\], DWROD PTR \[15\]  
24 19 02 = mov \[02\],\[19\]  
56 02 = push \[02\],sub ebp,4  
3b = 检查堆栈指针是否越界  
2f 05 07 = movb \[07\],05  
2f 01 05 = movb \[05\],01  
1f 07 = not \[07\]  
52 05 07 = add byte ptr \[05\],byte ptr \[07\]  
1f 04 = not \[4\]    
2f ff 1f = movb \[1f\],ff  
04 1f 07 = and \[07\],\[1f\]  
24 07 0d = mov \[0d\],\[07\]  
27 1a = not \[1a\]  
6a 0d 1a = add dword ptr \[1a\],\[0d\]  
27 1a ...  
24 1a 16 = mov \[16\],\[1a\]  
24 16 19 ...  
24 1a 1f ...  
1e d0 fd ff ff 0e = movd \[0e\],fffffdd0  
1e ff ff ff ff 1d = movd \[1d\],ffffffff  
  
纰漏肯定有感兴趣的朋友大家一起讨论吧。有问题的地方请指出我们共同进步。  
至于其保护过程还要花时间来看。 

体力活。眼睛都看花了。点到为止。他这个混淆引擎可大可小。  
补上一个完整点的指令对照吧。目前 HProtect.sys 的指令还没重新混淆。  
00 1d 05 = XOR BYTE PTR \[05\],BYTE PTR \[1d\]  
04 1f 07 = and \[07\],\[1f\]  
06 0d = JMP ESI+\[0d\]  
0f 1f 05 = SHR \[05\],\[1f\]  
10 03 1d = mov \[\[1d\]\],\[03\]  
13 1d 1b = MOV BYTE PTR \[1b\],BYTE PTR \[1d\]  
19 14 0d = IMUL \[0d\],\[14\]  
1c 02 1d = RORB \[02\],\[1d\]  
1e d0 fd ff ff 0e = movd \[0e\],fffffdd0  
1f 07 = not \[07\]  
22 12 1d = SHL \[1d\],\[12\]  
24 19 02 = mov \[02\],\[19\]  
27 1a = not \[1a\]  
29 18 04= ROL \[18\],04  
2e 1d 1e = OR \[1e\],\[1d\]  
2f 05 07 = movb \[07\],05  
32 05 05 = MOV \[05\],\[\[05\]\]  
33 15 08 0c = MOVW \[0c\],0815  
39 1e 04 = PUSH \[1e\] ,POPFD  
3b = 检查堆栈指针是否越界  
41 18 03 = ROL \[03\],\[18\]  
4d 03 = MOV \[03\],EBX  
52 05 07 = add byte ptr \[05\],byte ptr \[07\]  
56 02 = push \[02\],sub ebp,4  
59 1e = POPFD \[1e\]  
62 50 = mov esp,ebp 虚拟机结束 = push esi+1,push xxx  
        popad  
        popfd  
        ret xxx ret 做为 call 返回到 esi + 1  
  特征码:8B E5 61 9D C2   
  
  
6a 15 1a = add DWORD PTR \[1a\], DWROD PTR \[15\]  
6b 10 = pop \[10\]  ; add ebp,4  
6c 11 09 = XOR DWORD PTR \[09\],DWORD PTR \[11\]  
74 01 0d = movb \[01\],\[0d\]