

# PE 文件中引入表中的函数的方法 

> 原文：[`www.pediy.com/kssd/pediy05/pediy50338.htm`](https://www.pediy.com/kssd/pediy05/pediy50338.htm)

当我们补丁一个 PE 文件时，常常要用到调用这个 PE 文件引入表中的函数，那么怎样来调用呢？要想很透彻的了解其原理，需要对引入表很熟悉，并且要对 PE 文件的加载机制有所了解。   
    对于引入表的介绍和 PE 加载机制，看雪老大的著作上已经介绍的很清楚，只要花点时间，很容易弄清楚。按照下面步骤，即使对不太熟悉引入表的朋友，也能直接使用其中的函数。   
  
准备的工具：LoadPe、Softice 或 TRW2000   
  
步骤：   
1、用 LoadPe 打开 PE 文件，查看其目录中的导入表，选中要使用的 DLL 并纪   
  录下它的 FirstThunk 的值(这是 IAT 表的起始 RVA)，假设为：E1B7。   
2、找到要在这个 DLL 中要使用的函数，假设为：LoadLibraryA。   
3、数一数这个函数在这个 DLL 函数列表中是第几个，假设为：第 3 个。   
4、那么在补丁中调用这个函数就是：   
  call dword ptr\[e1b7+ImageBase\]\[(3-1)\*4\]   
5、ImageBase 的获得：   
  当我们补丁一个程序时，补丁的起始 RVA 是已知的(可以根据 VA 用 LoadPe   
  的 FLC 计算出来)，请看下面程序：   
  假设补丁开始地址(VA)为：0040d000，通过计算知道对应的 RVA 为 n   
  0040d000:pushad   
  0040d001:call  0040d006   
  0040d006:pop  ebp   
  xxxxxxxx:sub  ebp-n-6  ；此时，ebp 就是 ImageBase，减 6 是   
                          ；pushad 和 call 0040d006 的指令长度   
  xxxxxxxx:call dword ptr\[ebp+e1b7\]\[(3-1)\*4\]  ；调用   
                                              ；LoadLibraryA   
            .   
            .       
            .   
  xxxxxxxx:popad   
  
  仅此抛砖引玉，欢迎高手赐教其他方法。   
  
                              Spring.W   
                              2002/11/3 

 4dd466d0e1e0cc123afbe504da2462ca