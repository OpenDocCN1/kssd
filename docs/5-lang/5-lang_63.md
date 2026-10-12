
# inline patch MaxtoCode(纯属恶搞)大牛勿进。

> 原文：`www.pediy.com/kssd/pediy08/pediy8-816.htm`

`rick`大牛，已经写出了`MaxtoCode`得`unpacker`.太牛了。不知道是不是把所有的`mathod`都执行一遍，然后`Dump`->修复。  

`unpacker`还不公开，所以像我这种小菜想破解`MaxtoCode`还得`inline patch`。  

`patch`主程序不是很现实，因为原始的`mathod`已经被`Kill`了，运行的时候动态的填充。那怎么办呢？  

很简单，随便在能运行到的`method`上内存写入断点，断下来后在`system.data.dll`里面，这个`dll`就是`MaxtoCode`的`DLL`,负责动态的解密`IL.patch`。这个是不错的选择。  

注意看断下来的地方的指令：

```
mov dl,byte ptr [ecx+eax]  
mov byte ptr [eax],dl
```

这里`eax`是被填充的地址，`dl`就是`IL opcode`.  

上面的指令刚好是`5 byte`.so........  

改为`jmp xxxxxxxx`(具体地址根据实际情况选择，选择`reloc`段不错)。  

然后就可以`Do what U want to do`了  

我做了这些：  

```
mov dl,byte ptr [ecx+eax]  
cmp eax,xxxxxxxx  
jnz end  
mov dl,xx  
cmp eax xxxxxxxx  
jnz end  
mov dl,xx  
end:  
mov byte ptr [eax],dl  
jmp xxxxxxxx
```

应该能看懂吧？  

猜测：其实完全可以利用这个`DLL`做个`unpacker`。  

`TO Jason.NET:.NET`保护只能走`VFP`那条路，自己接管`EE`, `JITCompiler`.但是那样`.NET`就失去了意义。所以目前没啥好办法保护`.NET`程序  

`rick`的`unpacker`似乎只支持`2.0`.我也没见过，听说而已。这个方法`1.0 2.0`通吃。  

可能利用`MaxtoCode`的`DLL`写`unpacker`可能是比较通吃也比较简便的办法 

`4dd466d0e1e0cc123afbe504da2462ca`
