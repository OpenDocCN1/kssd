

# 看雪学苑

> 原文：[`www.pediy.com/kssd/pediy02/forum169.htm`](https://www.pediy.com/kssd/pediy02/forum169.htm)

 小弟急需用 arpr1.0，原来的 beta1 已经有了 loader，但这个正式版目前还没有人破解。http://www.elcomsoft.com/ARPR/arpr.zip  

 脱壳最重要的一点是找到真正的 Entry Point.   
ARPR 1.0 build 45 的 EP 是 cs:401000   
找 EP 的方法现在还没有一个现成的，主要是靠经验。   
我见过很多 asprotect 的版本，有的还防 trw,   
找 ep 的方法真的是有嘴说不清，反正我都是 f8 跟踪过来的，   
有些地方有经验了就 f10.   
1、trw2k1.22 启动后一定要输入 faults off, 然后运行 arpr   
2、bpx cs:401000 (cs 为 017f, 视个人的机器）   
3、运行到 2 时，如果有注册的 trw2k, 就 makepe.   
  否则，将 cs:eip 改为 eb fe, 运行后再转到 procdump 将其 dump,   
  好像是上面的 dump , 下面的不行。   
  然后将 dump 后的程序中的 eb fe 改回原值，修正 ep.   
  
此时的 dump 程序还不能运行，需手工修改 import 表。   4dd466d0e1e0cc123afbe504da2462ca