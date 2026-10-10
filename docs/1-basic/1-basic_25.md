

# 关于 fs 寄存器的问题

> 原文：[`www.pediy.com/kssd/pediy10/86249.html`](https://www.pediy.com/kssd/pediy10/86249.html)

od 里看了下  
IsDebuggerPresent 的实现代码就三条指令  
mov eax,dword ptr fs:\[18h\]  
mov eax,dword ptr \[eax+30h\]  
movzx eax,byte ptr \[eax+2h\]  
也就是 BYTE PTR \[\[fs:\[18h\]+30h\]+2h\]（未声明均表示 DWORD 类型）的值就表示是否在调试环境中  
问题 1：用 masm 实现时发现，必须先 assume fs:nothing  
否侧就提示非法使用寄存器，为什么？  
问题 2：fs 在我的机子中的值是 03bh，为什么  

代码:

```
fs:\[00000018\]=\[7FFDD018\]=7FFDD000

```

希望有达人解答 

对于 Ring3 的应用程序，fs:\[0\]的地址指向的是 TEB 结构，这个结构的开头是一个 NT\_TIB 结构，NT\_TIB 结构的 0x18 偏移处是一个 Self 指针，指向这个结构自身，也就是指向 TEB 结构的开头。  
TEB 结构的 0x30 偏移是一个指向 PEB 的指针。PEB 又是一个结构，这个结构的 0x2 偏移处是一个 UChar，名叫 BeingDebugged，当进程被调试时，此值为 1，未被调试时此值为 0  
  
因此以下代码逐行执行后的结果：  
mov eax,dword ptr fs:\[18h\];eax=TEB 的指针  
mov eax,dword ptr \[eax+30h\];eax＝PEB 的指针  
movzx eax,byte ptr \[eax+2h\];eax=PEB.BeingDebugged（byte 扩展为 dword）  
  
TEB 和 PEB 结构的详细内容可以在 windbg 内核调试状态下使用 dt \_TEB、dt \_PEB 命令来察看。  
  
MASM 中默认是 fs:error，也就是默认不能使用 fs 段寄存器，因此要在 masm 中使用它时必须先 assume fs:nothing  
fs 是段寄存器，即保存段选择子，对应的地址通过 GDT 或 LDT 中的相应项目来决定其范围和使用权限等。  
  
最后回到 IsDebuggerPresent，它就是通过检查 PEB 中的 BeingDebugged 字段来确定进程是否处于被调试状态的，因此修改此字段可以直接影响此 API 的返回值。  
  
以上内容在调试或者说软件加密与解密过程中是基本的常识。 

问题 1：MASM 中默认是 fs:error，也就是默认不能使用 fs 段寄存器，因此要在 masm 中使用它时必须先 assume fs:nothing  
问题 2：大致是这样，段寄存器中的 16 位用来做索引信息，索引 64 位的段描述符。在 GDT 或 LDT 中得到段的起始地址。即你在 od 里看到 03bh 后面有一个 32 位 7ffdf000（fff）。所以 fs 的基址就是 7ffdf000，你可以去看看，7ffdf000 处是一个指针，和你输入 fs：\[0\]一样。 

说说问题二吧，这里我用实例证明一下。  
  
我这里 gdtr=8003f00003ff,某个进程空间中 ring 3 下的 fs=0038h。  
  
08h 用二进制表示为 00000000 00111000，按照 intel 文档的说明，最低的 2 位表示将要访问的段的权限级为 0，第 3 位这里是 0，表示将要访问的段的段描述符从全局描述符表（即 GDT）里面取得，剩下的高 13 位是所为索引值用的，这里是 7。每个段描述符占 8 个字节，8003f000 处存放有一个 NULL 描述符。  
lkd>dd 8003f000+8\*7  
8003f038 f0000fff 7f40f3fd 0400ffff 0000f200  
  
每个描述符 8 个字节，按照 intel 文档的说明，8003f038 处的双字的高 16 位是段起始地址的低 16 位，8003f03c 处双字的高 8 位是段起始地址的高 8 位，8003f03c 处双字的低 8 位是段起始地址的中间的 8 位，综合起来就是 7ffdf000。  
  
验证一下。  
lkd> dd 7ffdf000+18 l1  
7ffdf018  7ffdf000