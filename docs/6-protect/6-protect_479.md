

# 揭开 winlicense 对 regmon sst 监视的“神秘面纱”！【原创】

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-457.htm`](https://www.pediy.com/kssd/pediy09/pediy09-457.htm)

> regmon 激活时，修改了一些 sst 服务函数地址，winliencse 针对  
> 此进行判断 regmon 是否激活，逐一排除发现马脚在 NtFlushKey,  
> 恢复其原有的服务函数地址就可跳过 winliencse 的监视！至于修改方法  
> 应该很简单了。顺手牵羊，同时启动修改版的 regmon 和 filemon,对 winlisence  
> 的 trial 及其 key 中的 trial 进行反监视，狐狸尾巴全部揭穿，很失望，太失望了！  
>   
> 简单说一下恢复 NtFlushKey sst 服务函数地址方法：  
> 1。运行 regmon,激活监视  
> 2。 r0cmd /dumpsst\_nt:sst.bin  
> 3.启动 icesword,在 ssdt 查看 NtFlushKey 原服务函数地址和修改服务函数地址  
> 4。启动 winhex，修改 sst.bin  
> 5. r0cmd /loadsst\_nt:sst.bin  
> 6.启动 winlicense,不再提示。 

 4dd466d0e1e0cc123afbe504da2462ca