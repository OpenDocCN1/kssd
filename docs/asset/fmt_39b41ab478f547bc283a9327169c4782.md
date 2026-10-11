
# 两个变量交换，不使用第三个变量如何实现？

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-608.htm`](https://www.pediy.com/kssd/pediy09/pediy09-608.htm)

> 如题，变量 `A`，`B` 进行互换，但不许使用第三个中间变量，如何实现？

> 假设 `a` `b` 不等

```
a = a^b
b = b^a
a = a^b
```

> `delphi`

> `asm`
```
mov eax, A
mov edx, B
xchg eax, edx
mov A, Eax
mov B, Edx
end;
```

```
push a
push b
pop a
pop b
```

> 引用:

> `* * *`

> *最初由 `nig`发布* (帖子 312346) `delphi`

> `asm`
```
mov eax, A
mov edx, B
xchg eax, edx
mov A, Eax
mov B, Edx
end;
```

> `* * *`

> `|`

> |

> | --- |

> 既然这样了,为什么不如此操作?

> 代码:

> `* * *`

> ```
> asm    mov eax,A    mov edx,B    mov A,edx    mov B,eax   end;
> ```

> `* * *`

> 如果是 `A`,`B` 是独立数的话,也可以:

```
A = A + B
B = A - B
A = A - B
```

 4dd466d0e1e0cc123afbe504da2462ca
