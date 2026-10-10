

# 还原精灵 6.1 主程序脱壳

> 原文：[`www.pediy.com/kssd/pediy12/142886.html`](https://www.pediy.com/kssd/pediy12/142886.html)

在调试还原精灵 6.1 还原驱动时发现总是无法在 Windbg 里面进行单步调试，很奇怪，看那一下驱动，跟驱动没有关系，通过跟踪发现在其应用程序启动后，无法单步调试，HDDGMon.EXE,这个其主核心应用程序加那 Anti-Debug 调试功能，首先使用 PEID 查看其是否加壳，如下图所示：  
 ![](img/b154871944c8f6cd449d488c9f596fb0.jpg)  
图 1  
这是一个老牌的壳，这里采取两次内存断点方法，所谓两次内存断点法的原理是：外壳会依次对.text、.rdata、.data、.rsrc 的区块进行解压、密处理，因此先对.rsrc 块下一个 F2 内存访问断点（这个断点在访问后会自动删除），在此之前，我 shilft+F9 发现改程序会经常跳入到异常处理机制 ZwRaiseException，并且看到应为如下代码：div edi、div dx 这些发生异常的指令，这说明这个壳通过大量的异常指令来干扰我们进行单步调试，因此在 ollydbg 中忽略所有异常，shiflt+F9 运行，会跳到如下位置:  
004FA0A9            F3:A5            rep movs dword ptr es:\[edi\],dword ptr ds:\[esi\]  
004FA0AB            8BCB            mov ecx,ebx  
004FA0AD            F3:A4            rep movs byte ptr es:\[edi\],byte ptr ds:\[esi\]  
004FA0AF            8BF2             mov esi,edx  
004FA0B1            8B7C24 28         mov edi,dword ptr ss:\[esp+28\]  
esi 指向资源块，edi 指向外壳的区块，估计处理完 rsrc 后就会执行代码段，因此对.text 下 F2 内存断点,shilft+F9 运行，跳到如下位置:  
00448FA4            55                    push ebp  
00448FA5            8BEC                 mov ebp,esp  
00448FA7            6A FF                 push -1  
00448FA9            68 606C4500           push HDDGMon.00456C60  
00448FAE            68 0E914400           push HDDGMon.0044910E                 
; jmp to msvcrt.\_except\_handler3  
00448FB3            64:A1 00000000         mov eax,dword ptr fs:\[0\]  
00448FB9            50                    push eax  
00448FBA            64:8925 00000000       mov dword ptr fs:\[0\],esp  
00448FC1            83EC 68               sub esp,68  
00448FC4            53                    push ebx  
00448FC5            56                    push esi  
00448FC6            57                    push edi  
从上面得知 0x00448FA4 就是其 OEP。下面是对输入表进行修复，首先我们看一下其 IAT，部分的输入表被隐藏在外壳里，如下图 2 所示：  
![](img/a0a0a112aa7570528a0447ff09b1300d.jpg)   
图 2  
下面是通过 0xAA0000 最终定位到如图 3 的地址，也就是真正的 API 地址，这里我采取编写脚本来替换它，这个可以参考(http://bbs.pediy.com/showthread.php?p=903627)  
 ![](img/fd14df194db83f175123a74e7cef187c.jpg)  
图 3  
  
这里有两种情况,一是 inc eax,一是 nop,所以分情况写 ，下面是脚本的代码(本人对脚本还是不大熟悉，写的比较挫)如下：  
var iatstart  
var iatend  
var temp  
var lkpd  
  
mov iatstart,00450b28  
mov iatend,00450cf4  
  
labl:  
    mov temp,\[iatstart\]  
    log temp  
      
    cmp temp,0  
    je next  
      
    find temp,#FF30C3#    //找到这个 FF30C3 就是 PUSH DWORD PTR DS:\[EAX\]   RETN  
    mov temp,$RESULT     //找到上面指令的首要地址  
  
  
    mov lkpd,temp  
    sub temp,5  
    mov temp,\[temp\]  
  
    sub lkpd,1  
    mov lkpd,\[lkpd\]  
      
    cmp lkpd,C330FF40  
    je ok  
  
labk:  
    mov temp,\[temp\]  
    log temp  
    mov \[iatstart\],temp  
      
     
next:  
   add iatstart,4  
   cmp iatstart,iatend  
   jbe labl  
   jmp end  
  
error:  
    msg "错误"  
  
ok:  
    add temp,1  
    jmp labk  
  
end:  
    ret  
再次用 peid 打开,发现是 vc++6.0 写的,如下图 4：  
![](img/8896c7d213887c174270da4d3bb1e9ea.jpg)   
图 4  
可直接运行，但是发现其脱壳后的大小比用工具脱的要大一些，后面才发现有工具可以直接脱，反正就当自己学习一下，下面覆盖原来的文件，进行内核调试，去掉壳之后，可以进行单步调试,看样子，所有的 anti-debug 都在壳里，脱掉之后，再无干扰，还原精灵还是一款不错的软件，其修改 MBR,主引导记录，先启动自己代码，进行一系列操作之后，在从 12 扇区取出保存的原有 MBR；还有其竟有多点还原的功能，虽然只有 5 个，但是其是 2005 年的产品，可见当时其编程功力的深厚，值得我近一步的研究学习，随文附上未脱壳及脱壳后的主程序，安装文件自己去下一个，有点大，传不上来!  
还原精灵 6.1 主程序.rar![](img/afde35a5f3b3a4675abb3221cf07c6d1.jpg "124.gif")