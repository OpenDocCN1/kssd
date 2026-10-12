
# 仿照小喂 写了个小的 WinDbg 脚本，可以显示 SSDT Shadow

> 原文：[`www.pediy.com/kssd/pediy10/89432.html`](https://www.pediy.com/kssd/pediy10/89432.html)

```
$$ Win32KCall Script v0.1  

$$ by Tesla 2009.05.20  

aS ufLinkS "<u><col fg=\\\"emphfg\\\"><link name=\\\"%x\\\" cmd=\\\"uf 0x%x\\\">";  

aS ufLinkE "</link></col></u>";  

r $t1 = nt!KeServiceDescriptorTableShadow;  

$$r $t1 = 8055a6c0;  

r $t2 = poi(@$t1 + 18);  

r $t1 = poi(@$t1+10);  

.printf "\nOrd   Address   fnAddr   Symbols\n";  

.printf "--------------------------------\n\n";  

.for (r $t0 = 0; @$t0 != @$t2; r $t0 = @$t0 + 1)  

{  

    r $t3 = poi(@$t1);  

    .printf /D "[%3d] %X: ${ufLinkS}%X${ufLinkE} (%y)\n", @$t0, @$t1, @$t3, @$t3, @$t3, @$t3;  

    r $t1 = @$t1 + 4;  

}  

.printf "\n- end -\n";  

ad ufLinkS;  

ad ufLinkE;  
```

![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")
