

# Hasp Envelope

> 原文：[`www.pediy.com/kssd/pediy04/kanxue136.htm`](https://www.pediy.com/kssd/pediy04/kanxue136.htm)

1) 用冲击波找到 OEP   
2) bpx FreeEnvironmentStringsa   
3) 中断后，bd\*，G OEP   
4) 再次中断后就可以 dump 了   
5) 用 PE editor 修正 dump 后的 Entry point 为 OEP   
  
在第 4)步，如果你使用的是 SoftIce 但不会用 IceDump 的话，先把 OEP 的第一条指令改为 jmp OEP，然后 F5 退出 SoftIce，用 ProcDump 把进程 Dump full。第 5)步之后，再把 OEP 指令改回去就行了。  4dd466d0e1e0cc123afbe504da2462ca