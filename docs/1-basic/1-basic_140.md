

# 硬件对编程的影响

> 原文：[`www.pediy.com/kssd/pediy06/pediy6929.htm`](https://www.pediy.com/kssd/pediy06/pediy6929.htm)

在给 VProtector 除 BUG 的过程中发现了硬件的一个特点，想写出来给大家知道，免得以后有朋友因为这方面的问题阻碍了软件开发的过程  
  
inter 的 CPU 在从 P4 开始，cpu 指令的调用有个细节和以往的 cpu 不一样，  
我们在调用函数的时候 一般的流程是：  
  
push 下个指令的地址  
jmp  目标函数  
  
而 P4 的 cpu 的过程不一样，调用过程如下：  
  
jmp  目标函数  
push 下个指令的地址  
  
我们在设计利用堆栈执行指令的时候必须注意这个细节，比如 call esp 必须  
要针对当前 cpu 的特点定位 esp

> 对付这样的解决办法也找到  
> 我发现 call \[esp\] 不会存在这样的问题

> 老的 CPU 上面执行 call esp 的过程:  
>   
> 假如 esp = 401000  
> 那么 call esp 将会调用 400ffc 处的代码  因为 cpu 首先将返回地址压栈入 400ffc 处 再进入 esp 的地址  
>   
> P4 的运行情况恰恰相反 调用的始终使 esp 处的代码
> 
>  4dd466d0e1e0cc123afbe504da2462ca