
# nSpack v1.3

> 原文：[`www.pediy.com/kssd/pediy06/pediy6861.htm`](https://www.pediy.com/kssd/pediy06/pediy6861.htm)

国产北斗压缩解壳脚本，基于 OM1.3

```
//  
//  FileName    :   nSpack V1.3.oms (北斗程序压缩 V1.3 脱壳脚本)  
//  Author      :   chenjiwl  
//  Date        :   2004-11-25 20:36  
//  Comment     :   Search "popad,popfd,jmp" to get the OEP  
//                  通过搜索特征码，来得到 OEP 地址  
//  
```
```
EOB FindBreak   //当中断时转到 FindBreak
invoke Find, eip, "619DE9????????"  //搜索特征码
cmp reg00, -1     //是否找到特征码
je err    //如果没找到，则显示错误信息
mov reg01, reg00
invoke bp, reg01             //设置中断
run          //F9
halt
err:
invoke msg, "错误：可能不是使用 nSpack 加壳的程序!\\n 请检查后再运行本脚本。"
halt
FindBreak:
invoke BC, reg01            //清除中断
stepover       //F8
invoke msg, "  代码窗中有三个连续的代码是:\\n1-->popad\\n2-->popfd\\n3-->jmp ????????\\n\\n 这就是关键代码，jmp 所指向的位置就是 OEP 地址。\\n 请按 F8 跳转到这个地址，并 Dump 出来修复!  "
```

`4dd466d0e1e0cc123afbe504da2462ca`
