
# 网络吸血鬼 3.3

> 原文：[`www.pediy.com/kssd/pediy03/forum711.htm`](https://www.pediy.com/kssd/pediy03/forum711.htm)

暴力破解《网络吸血鬼 3.3》

作者：`mjing`(菜鸟级)

E-mail: `mjing@wx88.net`

时间：`2001.3.15`

工具：`soft-ice 4.01`, `icedump 6.015`, `ultraedit 8.0`, `FileMon`

下载地址：(某 D 版光盘)

保护方式：这个软件采用注册码保护，其将输入的注册码保存到其目录下的一个名为 `VAMPIRE.key` 的文件中，到下次启动时再进行判断。

分析：基于上面的原因，断点不太好下，尝试用 `bpx createfilea` 断点，但是这个家伙启动时使用 `createfilea` 函数次数过多，严重干扰了跟踪，我花了很长时间逐一查看，居然没有发现有用线索？？？(它倒底是用什么函数打开文件？？，哪位指点一下)而且用 `DASM` 反汇编也不好使，找不到 “`VAMPIRE.key`” 等字符串，简直把我逼到了山穷水尽的地步。还好，借助 `FileMon` 的神威，终于发现它的一个致使弱点，每次它都会删除 `VAMPIRE.key` 文件，即使该文件不存在！

载入 `Vampire.exe`，下断点，`bpx deletefilea`，按 `F5` 运行，过一会儿，被拦下后，按一下 `F11`，回到程序的领空中

```
015F:0048BF58  POP       EDX                                      
015F:0048BF59  POP       ECX                                      
015F:0048BF5A  POP       ECX                                      
015F:0048BF5B  MOV       FS:[EAX],EDX                            
015F:0048BF5E  JMP       0048BF9C                               
015F:0048BF60  JMP       00403234                               
015F:0048BF65  MOV       EAX,004ADA60                           
015F:0048BF6A  CALL      00403A84                               
015F:0048BF6F  LEA       EDX,[EBP-08]                            
015F:0048BF72  XOR       EAX,EAX                                  
015F:0048BF74  CALL      004028CC                               
015F:0048BF79  MOV       EAX,[EBP-08]                            
015F:0048BF7C  LEA       ECX,[EBP-04]                            
015F:0048BF7F  MOV       EDX,0048BFF8                             
015F:0048BF84  CALL      00407CE4                               
015F:0048BF89  MOV       EAX,[EBP-04]                            
015F:0048BF8C  CALL      00403EC4                               
015F:0048BF91  PUSH      EAX                                      
015F:0048BF92  CALL      KERNEL32!DeleteFileA ;删除了 VAMPIRE.key 文件
015F:0048BF97  CALL      0040354C                               
015F:0048BF9C  XOR       EAX,EAX                                  
015F:0048BF9E  POP       EDX                                      
015F:0048BF9F  POP       ECX                                      
015F:0048BFA0  POP       ECX                                      
015F:0048BFA1  MOV       FS:[EAX],EDX                             
015F:0048BFA4  PUSH      0048BFBE                                
015F:0048BFA9  LEA       EAX,[EBP-0C]                            
015F:0048BFAC  MOV       EDX,00000003                             
015F:0048BFB1  CALL      00403AA8                               
015F:0048BFB6  RET                                              
015F:0048BFB7  JMP       00403430                               
015F:0048BFBC  JMP       0048BFA9                               
015F:0048BFBE  POP       EDI                                      
015F:0048BFBF  POP       ESI                                      
015F:0048BFC0  POP       EBX                                      
015F:0048BFC1  MOV       ESP,EBP                                  
015F:0048BFC3  POP       EBP                                      
015F:0048BFC4  RET                                              
015F:0048BFC5  ADD       [EAX],AL                                
015F:0048BFC7  ADD       BH,BH                                  
015F:0048BFC9  INVALID                                          
```

小心地按 `F10`，返回几个 `CALL` 之后，到达如下地盘：

```
015F:0049AF27  MOV       EAX,[004AC96C]                           
015F:0049AF2C  CMP       DWORD PTR [EAX],07                        
015F:0049AF2F  JNZ       0049AF49                               
015F:0049AF31  MOV       DL,01                                     
015F:0049AF33  MOV       EAX,[EBP-04]                            
015F:0049AF36  CALL      0049AD30                               
015F:0049AF3B  MOV       EAX,[004AC9A8]                           
015F:0049AF40  MOV       EAX,[EAX]                                  
015F:0049AF42  CALL      004316EC                               
015F:0049AF47  JMP       0049AF5D                               
015F:0049AF49  MOV       EAX,[EBP-04]                            
015F:0049AF4C  MOV       BYTE PTR [EAX+00000630],01               
015F:0049AF53  XOR       EDX,EDX                                  
015F:0049AF55  MOV       EAX,[EBP-04]                            
015F:0049AF58  CALL      0049AD30                               
015F:0049AF5D  CALL      0048BEDC ;停在这里
015F:0049AF62  MOV       EAX,[004AC9A0]      
015F:0049AF67  CMP       DWORD PTR [EAX],00 --- 
015F:0049AF6A  JNZ       0049AF8D               |  这个
015F:0049AF6C  MOV       EAX,[EBP-04]           |  结构
015F:0049AF6F  MOV       EAX,[EAX+0000050C]    |  大大
015F:0049AF75  CALL      0047210C               |  可疑
015F:0049AF7A  MOV       EAX,[EBP-04]           |
015F:0049AF7D  MOV       EAX,[EAX+0000050C]     |
015F:0049AF83  CALL      00471C24               |
015F:0049AF88  JMP       0049B00D          ---
015F:0049AF8D  XOR       EDX,EDX               
015F:0049AF8F  MOV       EAX,[EBP-04]          
015F:0049AF92  MOV       EAX,[EAX+0000035C]    
015F:0049AF98  CALL      00421AB8              
015F:0049AF9D  XOR       EDX,EDX               
015F:0049AF9F  MOV       EAX,[EBP-04]                      
015F:0049AFA2  MOV       EAX,[EAX+00000360]                
015F:0049AFA8  CALL      00421AB8                        
015F:0049AFAD  XOR       EDX,EDX                         
015F:0049AFAF  MOV       EAX,[EBP-04]                      
015F:0049AFB2  MOV       EAX,[EAX+00000364]                
015F:0049AFB8  CALL      00421AB8                        
015F:0049AFBD  XOR       EDX,EDX                         
015F:0049AFBF  MOV       EAX,[EBP-04]                      
015F:0049AFC2  MOV       EAX,[EAX+000003B8]                
```

在 `015F:0049AF6A` 一行试着下命令:

```
r fl z
```

`F5` 运行，运气不错，瞎猫碰上死耗子，广告窗居然没有了，抄下机器码，用 `UEdit` 一改，就破解了:>

切入 `015F:0049AF5D  CALL      0048BEDC` 再跟踪一下，想找出注册码，但是奇迹再也没有出现，只看见许多可疑的代码，但实在看不懂其算法，哪位高手指点一下？？

最后，跟踪到如下一段

```
015F:0048BF14  CALL      00461634                             
015F:0048BF19  MOV       EAX,[EBP-04]                          
015F:0048BF1C  PUSH      EAX                                    
015F:0048BF1D  LEA       EAX,[EBP-08]                          
015F:0048BF20  PUSH      EAX                                    
015F:0048BF21  LEA       EAX,[EBP-0C]                          
015F:0048BF24  MOV       ECX,0048BFE4                           
015F:0048BF29  MOV       EDX,[004ADA60]                         
015F:0048BF2F  CALL      00403D4C                             
015F:0048BF34  MOV       EAX,[EBP-0C]                          
015F:0048BF37  MOV       ECX,00000009                           
015F:0048BF3C  MOV       EDX,00000001                           
015F:0048BF41  CALL      00403F04                             
015F:0048BF46  MOV       EDX,[EBP-08]                          
015F:0048BF49  POP       EAX                                    
015F:0048BF4A  CALL      00403E10                             
015F:0048BF4F  JZ        0048BF56  ;改为 JMP  就没有广告窗了
                     因此上面的 Call 有重大 
                     嫌疑，很可能是注册比较 
                     但看不懂:(
015F:0048BF51  CALL      0040ACF4                             
015F:0048BF56  XOR       EAX,EAX                                  
015F:0048BF58  POP       EDX
```



```
015F:0048BF59  POP       ECX
015F:0048BF5A  POP       ECX
015F:0048BF5B  MOV       FS:[EAX],EDX
015F:0048BF5E  JMP       0048BF9C
015F:0048BF60  JMP       00403234
015F:0048BF65  MOV       EAX,004ADA60
015F:0048BF6A  CALL      00403A84
015F:0048BF6F  LEA       EDX,[EBP-08]
015F:0048BF72  XOR       EAX,EAX
015F:0048BF74  CALL      004028CC
015F:0048BF79  MOV       EAX,[EBP-08]
015F:0048BF7C  LEA       ECX,[EBP-04]
015F:0048BF7F  MOV       EDX,0048BFF8
015F:0048BF84  CALL      00407CE4
015F:0048BF89  MOV       EAX,[EBP-04]
015F:0048BF8C  CALL      00403EC4
015F:0048BF91  PUSH      EAX
015F:0048BF92  CALL      KERNEL32!DeleteFileA
015F:0048BF97  CALL      0040354C
015F:0048BF9C  XOR       EAX,EAX
015F:0048BF9E  POP       EDX
```

OK，又算是完成了一篇“破”文，确实很破（谁让我是只菜鸟呢），

不要问我原理，我也不知道。

破解软件运气也很哦，当然，这上建立在一定的经验和功力上的。

原本不打算再写破文了，但是破了的东东不写出来，总好像没完成

一件什么事似的，顺便说一句，上面的方法也可以搞掉新出来的

`Net Vampire Pro 4.0b`。我好像只会暴力破解，不过本人非常热爱

和平:)好了，以后有机会再见吧！ `4dd466d0e1e0cc123afbe504da2462ca`
