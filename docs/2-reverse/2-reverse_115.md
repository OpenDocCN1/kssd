
# MEW1.1 脱壳脚本 For OM

> 原文：[`www.pediy.com/kssd/pediy06/pediy6899.htm`](https://www.pediy.com/kssd/pediy06/pediy6899.htm)

## 脚本信息

```
//  
//  FileName    :   Mew 11 SE v1.0.oms (Peid Scan :\*Neolite 2.0 -> Neoworx Inc.\*)  
//  Author      :   chenjiwl  
//  Date        :   2004-11-27 13:21  
//  Comment     :   Search "test,jnz,retn" to get the OEP  
//                  通过搜索特征码，来得到 OEP 地址  
//  
```
## 脚本内容

```
EOB FindBreak                   //当中断时转到 FindBreak
stepover                        //先得 F8 一下
invoke Find, eip, "85C075??C3"  //搜索特征码
cmp reg00,-1                    //是否找到特征码
je err                          //如果没找到，则显示错误信息
mov reg01, reg00                //保护 reg00
add reg01,4                     //修正地址，使脚本结束时能准确停在 OEP 处，
                                //目前 OM 还不能精确的多次 F7/F8 情况下，
                                //这个办法好用
invoke bp, reg01                //设置中断
run                             //F9
halt                            //脚本停止
err:
invoke msg,"错误：可能不是使用 MEW 加壳的程序!\\n 请检查后再运行本脚本。"
halt
FindBreak:
invoke BC, reg01                //清除中断
stepover                        //F8
invoke msg, "  代码窗中所指的地址就是 OEP 地址。\\n 请 Dump 出来修复!  "
```

附件:`NEM_v11.rar`

`4dd466d0e1e0cc123afbe504da2462ca`
