
```
00481BA1    D9E4                 FTST                            //与 0 比较  
00481BA3    DFE0                 FSTSW AX                        //存入  
00481BA5    F6C4 01             TEST AH,1  
00481BA8    74 02               JE SHORT tmjqk64g.00481BAC  
00481BAA    D9E0                 FCHS  
00481BAC    DC1D 05764100       FCOMP QWORD PTR DS:[417605]  
00481BB2    DFE0                 FSTSW AX  
00481BB4    F6C4 41             TEST AH,41                        //AH 和 41 比较  
00481BB7    B8 00000000         MOV EAX,0                         //EAX=0  
00481BBC    0F95C0              SETNE AL                          //条件为假时置 0 为真(ZF=0)置 1  
```

走到这里我们看到这部分的比较方法和上面的一样，也是注册码右边 `6` 位除以 `5` 再减机器码后 `5` 位必须等于零 那么算法就出来了 `^\_^`  

```
00481BBF    8945 D0             MOV DWORD PTR SS:[EBP-30],EAX     //[EBP-30]=EAX  
00481BC2    8B4D FC             MOV ECX,DWORD PTR SS:[EBP-4]  
00481BC5    81F9 04000080       CMP ECX,80000004  
00481BCB    74 0C               JE SHORT tmjqk64g.00481BD9  
00481BCD    81F9 05000080       CMP ECX,80000005  
00481BD3    0F85 10000000       JNZ tmjqk64g.00481BE9  
00481BD9    8B5D F4             MOV EBX,DWORD PTR SS:[EBP-C]  
00481BDC    85DB                 TEST EBX,EBX  
00481BDE    74 09               JE SHORT tmjqk64g.00481BE9  
00481BE0    53                   PUSH EBX  
00481BE1    E8 C6070000         CALL tmjqk64g.004823AC  
00481BE6    83C4 04             ADD ESP,4  
00481BE9    837D D0 00          CMP DWORD PTR SS:[EBP-30],0  
00481BED    0F84 71010000       JE tmjqk64g.00481D64              //跳则 over  
```

`......省略部分代码`  

```
00481C6B    BB 24060000         MOV EBX,624  
00481C70    E8 43070000         CALL tmjqk64g.004823B8  
00481C75    83C4 1C             ADD ESP,1C  
00481C78    68 04000080         PUSH 80000004  
00481C7D    6A 00               PUSH 0  
00481C7F    68 1D764100         PUSH tmjqk64g.0041761D  
00481C84    68 01030080         PUSH 8000301  
00481C89    6A 00               PUSH 0  
00481C8B    68 00000000         PUSH 0  
00481C90    68 04000080         PUSH 80000004  
00481C95    6A 00               PUSH 0  
00481C97    68 28764100         PUSH tmjqk64g.00417628  
00481C9C    68 03000000         PUSH 3  
00481CA1    BB 00030000         MOV EBX,300  
00481CA6    E8 0D070000         CALL tmjqk64g.004823B8            //这里弹出成功提示  
```

`......省略部分代码`  

```
00481DDD    68 04000080         PUSH 80000004                     //跳到这里         
00481DE2    6A 00               PUSH 0  
00481DE4    68 BE764100         PUSH tmjqk64g.004176BE  
00481DE9    68 01030080         PUSH 8000301  
00481DEE    6A 00               PUSH 0  
00481DF0    68 00000000         PUSH 0  
00481DF5    68 04000080         PUSH 80000004  
00481DFA    6A 00               PUSH 0  
00481DFC    68 C9764100         PUSH tmjqk64g.004176C9  
00481E01    68 03000000         PUSH 3  
00481E06    BB 00030000         MOV EBX,300  
00481E0B    E8 A8050000         CALL tmjqk64g.004823B8            //这里弹出错误消息匡  
```

### 算法总结：  

注册码左边 `6` 位除以 `5` 再减机器码前 `5` 位必须等于零且  

注册码右边 `6` 位除以 `5` 再减机器码后 `5` 位必须等于零  

```
str=2244616410  

Code=Left(str,5)*5+Right(str,5)*5  

所以  

22446*5=112230  

16410*5=82050  

机器码=2244616410  

注册码=11223082050  
```

注册成功后写入安装目录下的 `reg.edb` `set.edb` 两个文件  

恢复这两个文件又可以再玩一次  

菜鸟写破文，感谢大家看完  

`==================================================================================`  

`【工程声明】本过程只供内部学习之用！如要转载请保持过程完整！`  

`==================================================================================` 

`4dd466d0e1e0cc123afbe504da2462ca`
