
# 反汇编代码分析

```
0044CD6D  |.  50  push eax                                        ; |Arg1  
0044CD6E  |.  8B>mov ecx,esi                                      ; |  
0044CD70  |.  C7>mov dword ptr ss:[esp+DC],0                      ; |  
0044CD7B  |.  E8>call PC\_Eyes.0044CC70                            ; \\再次将用户名计算出来的 5 组数据在表中取值得到 40 个数据  
0044CD80  |.  8B>mov esi,dword ptr ss:[esp+DC]  
0044CD87  |.  8D>lea eax,dword ptr ss:[esp+48]  
0044CD8B  |>   8A>/mov dl,byte ptr ds:[eax]                        ;  得到用户名处理后的 5 组注册码  
0044CD8D  |.  8A>|mov bl,byte ptr ds:[esi]                         ;  用注册码取值计算出来的  
0044CD8F  |.  8A>|mov cl,dl  
0044CD91  |.  3A>|cmp dl,bl                                         ;  最后比较了！！！！  
0044CD93  |.  75>|jnz short PC\_Eyes.0044CDB3  
0044CD95  |.  84>|test cl,cl  
0044CD97  |. /74>|je short PC\_Eyes.0044CDAF  
0044CD99  |.  |8A>|mov dl,byte ptr ds:[eax+1]  
0044CD9C  |.  |8A>|mov bl,byte ptr ds:[esi+1]  
0044CD9F  |.  |8A>|mov cl,dl  
0044CDA1  |.  |3A>|cmp dl,bl  
0044CDA3  |.  |75>|jnz short PC\_Eyes.0044CDB3  
0044CDA5  |.  |83>|add eax,2  
0044CDA8  |.  |83>|add esi,2  
0044CDAB  |.  |84>|test cl,cl  
0044CDAD  |.^|75>\\jnz short PC\_Eyes.0044CD8B  
0044CDAF  |> \\33>xor eax,eax  
0044CDB1  |.  EB>jmp short PC\_Eyes.0044CDB8  
0044CDB3  |>  1B>sbb eax,eax  
0044CDB5  |.  83>sbb eax,-1  
0044CDB8  |>  85>test eax,eax  
0044CDBA  |.  5B  pop ebx  
0044CDBB  |.  C7>mov dword ptr ss:[esp+CC],-1  
0044CDC6  |.  8D>lea ecx,dword ptr ss:[esp+8]  
0044CDCA  |.  74>je short PC\_Eyes.0044CDD5                         ;  必须跳  
0044CDCC  |.  E8>call PC\_Eyes.0044CE40  
0044CDD1  |.  33>xor eax,eax  
0044CDD3  |.  EB>jmp short PC\_Eyes.0044CDDF  
0044CDD5  |>  E8>call PC\_Eyes.0044CE40  
0044CDDA  |.  B8>mov eax,1                                          ;  真正的标志位  
0044CDDF  |>  8B>mov ecx,dword ptr ss:[esp+C4]  
0044CDE6  |.  5F  pop edi  
0044CDE7  |.  5E  pop esi  
0044CDE8  |.  64>mov dword ptr fs:[0],ecx  
0044CDEF  |.  81>add esp,0C8  
0044CDF5  \\.  C2>retn 8  
```

## 总结

首先检查用户名是否是 `crsky`，如果是它，完了，关机。

### 注册码处理

分两步：

1. 先每次取三位注册码将它们转化为 4 个内存单元
2. 每个内存单元在密码表取值得到 4 个字符

如果注册码不是 3 的倍数，如果余数是 1 最后的 2 个字符将变成“＝”如果余数是 2 最后的 1 个字符将变成“＝”

### 用户名处理

分 3 次：

**第 1 步**：初步处理用户名和上面处理注册码过程一样

**第 2 步**：将上面处理过的用户名用 `MD5` 算法处理，得到的 32 结果再分成 5 组中间用“－”连接：

- 第 1 组：从第 4 位开始连续取 5 个字符作为一组
- 第 2 组：从第 12 位开始连续取 5 个字符作为一组
- 第 3 组：从第 22 位开始连续取 5 个字符作为一组
- 第 4 组：从反过来的 `MD5` 值中从第 14 位开始连续取 5 个字符作为一组
- 第 5 组：从反过来的 `MD5` 值中从第 2 位开始连续取 5 个字符作为一组

**第 3 步**：再将得到 5 组数据再和处理注册码一样得到 40 位数据

最后 40 位数据和对注册码处理出来的数据比较。这里面处理注册码与用户名部分算法一样（密码表），这样就觉得“形同虚设”没多少意义。

算法简单，自己搞定！

`4dd466d0e1e0cc123afbe504da2462ca`
