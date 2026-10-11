
```
        jae     GarbageExit
```

```
 
```

```
        mov     eax,EndPolyTable-PolyTable
```

```
        call    r_range
```

```
        lea     ebx,[ebp+PolyTable]
```

```
        mov     eax,[ebx+eax*4]
```

```
        add     eax,ebp
```

```
        call    eax
```

```
 
```

```
GarbageExit:
```

```
        dec     byte ptr [ebp+recursion_level]
```

```
        ret
```

因此，我们的引擎能够生成大量的含有 `calls` 的垃圾代码:)，当然，这些也能应用在 `PUSH` 和 `POP` 之间。

### 结束语

多态性定义了代码，因此，我不再多说。自己去做而不是仅仅拷贝代码。不仅仅去做典型的使用一种简单加密操作类型和基本花指令例如 `MOV` 等的引擎。使用所有你能想到的方法。例如，有许多类型的 `Calls` 可以去做：三种方式（前面描述的），除了这些还有，你可以建立堆栈框架，`PUSHAD/POPAD`,由 `PUSH` 传递参数,等等，随你了!

`4dd466d0e1e0cc123afbe504da2462ca`
