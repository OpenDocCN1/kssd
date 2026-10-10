

# 从分页机制思考内存断点

> 原文：[`www.pediy.com/kssd/pediy12/129860.html`](https://www.pediy.com/kssd/pediy12/129860.html)

最近用 OD 对内存设断点时候想到内存断点有 2 种  内存访问和内存写入 ，但为什么没有内存读取断点呢？  
下面是自己的一点想法，有错误的话麻烦各位大虾不吝指教。。![](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬")  
从 http://bbs.pediy.com/showthread.php?threadid=10829 我了解到内存断点的原理即  
如果是内存写断点，OD 就修改相应的页面属性为 RE（可读，可执行）  
如果是内存访问断点，OD 就修改相应的页面属性为 NO ACCESS（不可访问）  
  
修改页面属性肯定一般用到 VirtualProtect 这个函数有个参数是修改后的页面属性：  
关于读写和执行的有  
  
PAGE\_EXECUTE\_READWRITE  
PAGE\_EXECUTE  
PAGE\_READONLY  
PAGE\_READWRITE  
PAGE\_NOACCESS  
PAGE\_EXECUTE\_READ  
  
对于这个函数先看到这里 ，后面会用到，下面看分页机制。  
  
分页机制是保护模式的重要成员之一，引用书上的话解释就是：  
在保护模式下，控制寄存器 CR0 中的最高位 PG 位控制分页管理机制是否生效。如果 PG=1，分页机制生效，把线性地址转换为物理地址。如果 PG=0，分页机制无效，线性地址就直接作为物理地址。必须注意，只有在保护方式下分页机制才可能生效。只有在保证使 PE 位为 1 的前提下，才能够使 PG 位为 1，否则将引起通用保护故障。   
关于怎样计算的就不介绍了，可以自己网上查。  
  
分页机制个人认为分为 2 种一个是开启 PAE 的分页机制和普通的分页机制  
普通的分页机制页表和页目录项是 32 位即 4 字节而 PAE 下的是 8 字节。  
简单的讲 PAE 下的分页机制可以将虚拟地址映射到更大的物理内存上。  
对于普通的分页机制，与页的保护有关的内容是页表或页目录项的低 12 位，图如下：  
![](img/b3300e50df38d419967dbcf9b485ad0d.jpg)  
其中 U/S 位是代表相应的页可以访问的特权级，为 0 只有 RING0、1、2 可以访问 为 1 任意特权级都可以访问。  
P 位是代表所对应的页有没有映射到物理内存。  
R/W 是代表所对应的页面是否可以读/写/执行，这个标志位只对于 CPU 处于 ring3 级别才有效。为 1 的时候代表 可读可写可执行  为 0 的时候代表可读可执行  
上述条件一旦 CPU 在检查部通过的时候就会触发 14（0eh）号异常，然后执行操作系统布置好的代码。  
那么现在就清楚了：  
内存写断点是 OD 用 PAGE\_EXECUTE\_READ 调用 VirtualProtect,最后内核中的函数会把相应的页表项 r/w 位设置为 0  
  
内存访问断点是 OD 用 PAGE\_NOACCESS 调用 VirtualProtect,最后内核中的函数会把相应的页表项 P 位设置为 0  
  
至于为什么没有内存读取断点，这就显而易见了,因为页表项不支持![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")，如果 R/W 位为 0 代表可写可执行就可以了，页的执行权始终是有的  
  
另外对于开启 PAE 下的分页机制与普通的分页机制除了在计算上的差别，还有一点很重要的差别就是利用页表项的第 63 位即最高位作为代码执行权的标准。这个位置 1，表示此页不可执行。没有置位，  
表示此页可以执行。即页的执行权要依据条件了。  
XP SP2 中的 DEP 技术就是利用这一点来完成的，这在 MSDN 中有所体现，对于 PAGE\_READONLY 和  
PAGE\_READWRITE 在 MSDN 中都有这么一句话“If Data Execution Prevention is enabled, attempting to execute code in the committed region results in an access violation.“  
  
但是 PAGE\_EXECUTE 却没有这一句，原文是这样说的”An attempt to read from or write to the committed region results in an access violation. “我认为还是需要借助于 PAE 来实现的，于是我测验了一下，环境是 XP sp3 没有开启 PAE  
代码如下：  

代码:

```
int \_tmain(int argc, \_TCHAR\* argv\[\])
{
    DWORD curAddress;
  \_asm
  {
    call l;
l:
    pop eax;
    mov  curAddress ,eax;
  }

  curAddress = curAddress &(0xFFFFF000);
  printf("%8X\\n", curAddress);
  DWORD old;
  if(!VirtualProtect((LPVOID)curAddress, 0x1000, PAGE\_EXECUTE, &old))
  {
    printf("VirtualProtect Error %d\\n", GetLastError());
    return 0;
  }
    printf("旧的属性是%8X\\n", old);
  //试着去读取
  \_asm
  {
    //读取
    mov eax, curAddress;
    mov eax , dword ptr\[eax\];
    //写入
    mov eax, curAddress;
    mov dword ptr\[eax\] , eax;
  }
  return 0;
}

```

将本段代码所在的页设置为只可以执行，然后用代码对页的开始进行读取和写入的测试  
结果是只有写入的时候才报异常，读取没有异常。截图如下：  
![](img/78b01f99d4708b27c0c66b6cf0693068.jpg)  
可见并没有像微软说的这么神乎，我推断是在没有开 PAE 的机器上只是将 R/W 位置 0，这时只是可读可执行的，但是写入的话 CPU 会触发异常，然后执行操作系统的代码，弹出框框。  
关于 DEP 实现的详细内容可以参考：http://forum.byr.edu.cn/article/Security/4293  
另外怎样启动和关闭 PAE 网上也有介绍。