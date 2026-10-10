

# 写了个小的 WinDbg 脚本，可以显示 SSDT

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-815.htm`](https://www.pediy.com/kssd/pediy08/pediy8-815.htm)

$$ ntcall Script v0.1  
$$ by 小喂 2006.10.29  
  
aS ufLinkS "<u><col fg=\\\\\\"emphfg\\\\\\"><link name=\\\\\\"%x\\\\\\" cmd=\\\\\\"uf 0x%x\\\\\\">";  
aS ufLinkE "</link></col></u>";  
  
r $t1 = nt!KeServiceDescriptorTable;  
r $t2 = poi(@$t1 + 8);  
r $t1 = poi(@$t1);  
  
.printf "\\nOrd   Address   fnAddr   Symbols\\n";  
.printf "--------------------------------\\n\\n";  
  
.for (r $t0 = 0; @$t0 != @$t2; r $t0 = @$t0 + 1)  
{  
    r $t3 = poi(@$t1);  
    .printf /D "\[%3d\] %X: ${ufLinkS}%X${ufLinkE} (%y)\\n", @$t0, @$t1, @$t3, @$t3, @$t3, @$t3;  
    r $t1 = @$t1 + 4;  
}  
  
.printf "\\n- end -\\n";  
  
ad ufLinkS;  
ad ufLinkE;  
  
   
点击链接会完整反汇编具体某个函数。![](img/77a5f7cfd3ec840c98be3aecc80df838.jpg) 

 4dd466d0e1e0cc123afbe504da2462ca