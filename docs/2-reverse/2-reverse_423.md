

# IDAPython 里如何获取寄存器内容

> 原文：[`www.pediy.com/kssd/pediy12/137075.html`](https://www.pediy.com/kssd/pediy12/137075.html)

如题，写了一个简单的 IDAPython 插件把内存中的一个特定的数据结构写入文件 dump 出来。但是我不知道在 IDAPython 里如何获取寄存器的内容，现在只能每次修改 py 文件写入绝对地址。  
哪位能指点下如何获取寄存器的内容？Thx 

现在运行调试器，hook 会捕捉所有的调试事件，这样就能非常精确的控制 IDA 调试器。下面的函数在调试的时候非常有用:  
AddBpt( long Address )  
  在指定的地点设置软件断点。  
        
GetBptQty()  
  返回当前设置的断点数量。  
  
GetRegValue( string Register )  
  通过寄存器名获得寄存器值。  
    
SetRegValue( long Value, string Register )  
  设定寄存器的值。  
  
有一本书上有《Python 灰帽子》,网上有下载