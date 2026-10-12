
```
015F:0043DAF4 F6D8             NEG      AL                                    |   
015F:0043DAF6 88440D9C        MOV      [EBP+ECX-64],AL                       |   
015F:0043DAFA 0FBE440D88      MOVSX    EAX,BYTE [EBP+ECX-78]                 |   

015F:0043DAFF 0FBE540D9C      MOVSX    EDX,BYTE [EBP+ECX-64]                 | 计   

015F:0043DB04 0BC2            OR       EAX,EDX                               算      


015F:0043DB06 BE1A000000      MOV      ESI,1A                                注   

015F:0043DB0B 99              CDQ                                           册   

015F:0043DB0C F7FE            IDIV     ESI                                  码   

015F:0043DB0E 80C241          ADD      DL,41                                

015F:0043DB11 88540D9C        MOV      [EBP+ECX-64],DL                       |   

015F:0043DB15 41              INC      ECX                                    |   

015F:0043DB16 83F910          CMP      ECX,BYTE +10                          |   

015F:0043DB19 7CD1            JL       0043DAEC                                ←——————   

015F:0043DB1B 8D459C          LEA      EAX,[EBP-64]                         

015F:0043DB1E 8D4DEC          LEA      ECX,[EBP-14]                          <---EAX 指向真注册码   

015F:0043DB21 50              PUSH     EAX                                  

015F:0043DB22 E8C9880000      CALL     `MFC42!ord_00000219`                  <---EAX 指向真注册码   

015F:0043DB27 8B75EC          MOV      ESI,[EBP-14]                         

015F:0043DB2A 8B45E8          MOV      EAX,[EBP-18]                         

015F:0043DB2D C645FC05        MOV      BYTE [EBP-04],05                    

015F:0043DB31 8A10            MOV      DL,[EAX]                             

015F:0043DB33 8ACA            MOV      CL,DL                                

015F:0043DB35 3A16            CMP      DL,[ESI]                             

015F:0043DB37 751C            JNZ      0043DB55                            

015F:0043DB39 84C9            TEST     CL,CL                                

015F:0043DB3B 7414            JZ       0043DB51                            

015F:0043DB3D 8A5001          MOV      DL,[EAX+01]                          

015F:0043DB40 8ACA            MOV      CL,DL                                

015F:0043DB42 3A5601          CMP      DL,[ESI+01]                          

015F:0043DB45 750E            JNZ      0043DB55                            

015F:0043DB47 83C002          ADD      EAX,BYTE +02                        

015F:0043DB4A 83C602          ADD      ESI,BYTE +02                        

015F:0043DB4D 84C9            TEST     CL,CL                                

015F:0043DB4F 75E0            JNZ      0043DB31                            

015F:0043DB51 33C0            XOR      EAX,EAX                              

015F:0043DB53 EB05            JMP      SHORT 0043DB5A                      

015F:0043DB55 1BC0            SBB      EAX,EAX                              

015F:0043DB57 83D8FF          SBB      EAX,BYTE -01                        

015F:0043DB5A 85C0            TEST     EAX,EAX                              

015F:0043DB5C 743F            JZ       0043DB9D                              <---此处跳转就能成功注册   

015F:0043DB5E 6A00            PUSH     BYTE +00                             

015F:0043DB60 68B4364600      PUSH     DWORD 004636B4                      

015F:0043DB65 6894364600      PUSH     DWORD 00463694                      

015F:0043DB6A 8BCB            MOV      ECX,EBX                              

015F:0043DB6C E8398C0000      CALL     `MFC42!ord_00001080`                 <---call 出出错窗口   

015F:0043DB71 8D4DEC          LEA      ECX,[EBP-14]                         

015F:0043DB74 C645FC03        MOV      BYTE [EBP-04],03                    

015F:0043DB78 E84F880000      CALL     `MFC42!ord_00000320`                

015F:0043DB7D 8D4DE8          LEA      ECX,[EBP-18]                         

015F:0043DB80 C745FCFFFFFFFF  MOV      DWORD [EBP-04],FFFFFFFF              

015F:0043DB87 E840880000      CALL     `MFC42!ord_00000320`                

015F:0043DB8C 8B4DF4          MOV      ECX,[EBP-0C]                         

015F:0043DB8F 5F              POP      EDI                                   

015F:0043DB90 5E              POP      ESI                                   

015F:0043DB91 64890D00000000  MOV      [FS:00],ECX                          

015F:0043DB98 5B              POP      EBX                                   

015F:0043DB99 8BE5            MOV      ESP,EBP                              

015F:0043DB9B 5D              POP      EBP                                   
```

当我们来到 `0043DB22` 处，`D EAX` 就能瞧见我们的注册码（当然是真的啦）。再往下继续走，发现 `0043DB6C E8398C0000 CALL MFC42!ord_00001080` 是 call 出出错窗口的地方，我们回过头往前看，在其不远处恰巧有一跳转，能够跳过这个该死的 call，据此我们可以肯定地判断：这个跳转就是关键的一跳。

## 6、好了，现在关键的地方已经找到，那么我们应该如何处置它呢？办法当然有多个，即要么爆破，要么用 `Keymake` 做出它的注册机：

### ⑴、爆破：

从上面的分析中我们得知，在 `0043DB5C 743F JZ 0043DB9D` 处只要反跳就能成功注册，所以，我们将跳转命令 `JZ` 改成 `JNZ`，即将机器码 `743F` 改成 `753F` 就行。因此我们先备份该软件的主文件，再用 16 进制编辑工具（如 `UltraEdit`）把 `0043DB5C 743F JZ 0043DB9D` 改成 `0043DB5C 743F JNZ 0043DB9D`，也即将将机器码 `743F` 改成 `753F`，然后存盘退出，修改后的程序，注册时我们随意填写注册码（当然是假的哦）均可成功注册。

非常有趣的是，只要当你跳过 `0043DB6C E8398C0000 CALL MFC42!ord_00001080` （call 出出错窗口），不但能够让你成功注册，而且它还自动地将正确的注册码填入注册码的窗口中！

### ⑵、做内存注册机：

首先我们得感谢刘建英为我等 Cracker 打造了一个巨酷的工具软件：`Keymake`。运行 `Keymake V1.73`。

我们知道当程序运行到在 `0043DB22 E8C9880000 CALL MFC42!ord_00000219` 处，`D EAX` 就可在 `TRW2000` 的数据窗口内瞧见你所需要的真注册码，因此我们点击 `Keymake` 菜单：“其他/内存注册机”，在中断地址列表中填上如下内容：

*   中断地址：`43Db22`
*   次数：`1`
*   指令：`E8`
*   长度：`5`
*   注册码：内存方式--->`EAX`

然后点击“生成”按钮即可生成我们需要的内存注册机了。

当你注册成功后，软件会在下列地方做些改动：

#### ㈠、在注册表中增加如下键值：

```
[HKEY_USERS\.DEFAULT\Software\爬山虎软件工作室\爬山虎工资管理专家\Options]
"zcm"="HVTH-TLBP-XHBK-DGYT"
"dwmc"="duhe"
```

#### ㈡、在其数据库文件（`GZ.mdb`）中填入你的注册码相关信息。

## 问题：

由于本人太菜而看不懂上面的算法。如果我要用 `Keymake` 做个它的算法注册机，请问各位高手，我该如何编写它的 `rek` 文件呢？

先谢了！ `4dd466d0e1e0cc123afbe504da2462ca`
