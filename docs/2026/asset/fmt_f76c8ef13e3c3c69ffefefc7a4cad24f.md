
|   |   |
| --- | --- |
| 1 2 | `if (__builtin_expect (nextchunk->size <= 2 * SIZE_SZ, 0) || __builtin_expect (nextsize >= av->system_mem, 0))` |

作者使用的shell code很短，不会将`chunk2->nextchunk->size`覆盖为异常值，即使shell code很长，也可以通过`jmp`跳过`nextchunk->size`字段，另外`av->system_mem`，在攻击者可以构造的范围，所以，可以保证这个判断不成立。

- 判断8

    |   |   |
    | --- | --- |
    | 1 | `if (!prev_inuse(p))` |

    `chunk2->size = 0x40d`，`PREV_INUSE`标志位为1，显然不会通过这个判断，将`chunk2`与其前一个`chunk`合并。

- 判断9

    |   |   |
    | --- | --- |
    | 1 | `if (nextchunk != av->top)` |

    和控制判断4的道理一样，由于`av->top`的值，是受攻击者控制的，所以相应也很容易控制这个判断，使其成立。

- 判断10

    |   |   |
    | --- | --- |
    | 1 | `if (!nextinuse)` |

    `chunk2->nextchunk`，也就是漏洞程序中最后一次执行`malloc()`分配的`chunk`，它这时显然还没有释放，所以这个判断不成立，从而最终欺骗`glibc`执行到`else`分支中的代码（如果漏洞程序中没有最后一次`malloc()`，应该也能攻击成功，因为那样的话，`chunk2`后面就是`top chunk`，而`top chunk`一定是`inuse`状态的，这是通过它顶部的`fencepost`标记的）。

## glibc改造

  随着各种攻击技术的出现，glibc其实一直都在改造，以上看到的这些判断，很多就是为了缓解攻击，但glibc的改造，是受限于两个因素的：

- 不能影响正常逻辑

  不能添加一个判断后，正常的逻辑也不对了。

- 不能影响性能

  比如业务层调用`free()`，glibc会将释放`chunk`放在相应的缓存链表中，而判断是否`double free`，只会拿当前释放`chunk`和链表头中的第一个`chunk`，进行地址对比，而不会遍历整个链表对比。

  所以，glibc只能缓存攻击，根本避免被攻击，在业务层的源头就要开始防范。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#漏洞利用](https://bbs.kanxue.com/forum-150-1-154.htm)

[#Linux](https://bbs.kanxue.com/forum-150-1-161.htm)
