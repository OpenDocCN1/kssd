
根据这些赋值，我们可以推测出`Content`结构体：

```
00000000 Content         struc ; (sizeof=0x18, mappedto_9)
00000000 magic           dd ?
00000004 pad             dd ?
00000008 unknown_ptr     dq ?
00000010 only_user_ptr   dq ?
00000018 Content         ends
```

其中，`unknown_ptr`的大小与初始化与`qword_41A8`相似，可以推断出它们类型应该一致。

然后将`Physic`结构体的`content_ptr`类型修改为`Content *`。

还原后的结构体如下所示：

![image-20250731110825968](https://bbs.kanxue.com/upload/attach/202508/912098_EC3YYSDSNNN99TF.png)

最终，程序返回`manage_physic[0]`的地址给全局变量`a1`指针。

### 结构体还原-`PhysicWithPermissionConfirm`

此时，再次进入`user_operation()`函数：

![image-20250731111315122](https://bbs.kanxue.com/upload/attach/202508/912098_38DJEZEMFTUE66N.png)

可以发现，代码可读性有所提高。结合代码，我们可以推断出`qword_41A8`类型的的结构体：

```
00000000 PhysicWithPermissionConfirm struc ; (sizeof=0x18, mappedto_10)
00000000 content_ptr     dq ?                    ; offset
00000008 perm            dd ?
0000000C pad1            dd ?
00000010 permission_confirm dd ?
00000014 pad2            dd ?
00000018 PhysicWithPermissionConfirm ends
```

它在`Physic`结构体的基础上，多了一个`permission_confirm`字段用于根据`op`选择程序分支。

此时，`user_operation()`函数非常清晰：

![image-20250731111949149](https://bbs.kanxue.com/upload/attach/202508/912098_HVYK8KBVV7U9UEX.png)

我们之前提到过，`Content`结构体中的`unknown_ptr`与该变量类型一致，将其修改为`PhysicWithPermissionConfirm *`类型并改名。

此时，我们再次进入`manager_operation()`函数，发现代码可读性提高：

![image-20250731112507335](https://bbs.kanxue.com/upload/attach/202508/912098_GGTV8M7UCPA3QEF.png)

### 逆向分析还原后的完整程序

`main`函数：

![image-20250731112916687](https://bbs.kanxue.com/upload/attach/202508/912098_DCVVQSCRXYSYJ8Y.png)

`init`函数：

![image-20250731112817731](https://bbs.kanxue.com/upload/attach/202508/912098_H5GVCP7BP9SXKDV.png)

`user_operation`函数：

![image-20250731112834426](https://bbs.kanxue.com/upload/attach/202508/912098_V5A39AWBHKMDKUA.png)

`manager_operation`函数：

![image-20250731112943369](https://bbs.kanxue.com/upload/attach/202508/912098_9JV232Z3J8K2X3U.png)

至此，我们已经完成逆向分析的所有工作，后续只需要理清程序逻辑即可进行进一步的漏洞利用。

## 利用思路

### 整体利用思路

逆向分析完成后，我们发现`manager_operation()`函数中有一段代码非常奇怪：

```
case 6:
    write(1, **(a1->only_user_ptr + 8), 0xFFuLL);
    break;
case 3:
    read(0, **(a1->only_user_ptr + 8), 0xFFuLL);
    break;
```

它没有写入到某个变量中，而是写入到计算后的地址指向的内存空间中。

而`a1->only_user_ptr`在`init()`函数中被初始化为`onlyuser`变量的地址：

```
content_ptr->only_user_ptr = &onlyuser;
```

我们可以在内存空间中查看`onlyuser`变量：

![image-20250731113231711](https://bbs.kanxue.com/upload/attach/202508/912098_WBJWTTJ4NNTAQHA.png)

而`(a1->only_user_ptr + 8)`刚好是`physicEntry`变量的地址（在`user_operation()`函数中被赋值的全局变量）：

![image-20250731113405363](https://bbs.kanxue.com/upload/attach/202508/912098_ATE3ASBT8X3CRES.png)

内存结构如图所示：

![image-20250731161726138](https://bbs.kanxue.com/upload/attach/202508/912098_APDVQ4RUGPQVKWZ.png)

如果我们可以控制`a1->only_user_ptr`就可以实现**任意地址读写**。

我们在`init()`函数中可以发现一个很有意思的事情，`manage_physic[]`数组的有效存储空间实际为下标1-19。

而全局变量`a1`始终指向`manage_physic[0].content_ptr`，它在初始化时被赋值：

```
content_ptr = manage_physic[0].content_ptr;
manage_physic[0].content_ptr->magic = 1;
content_ptr->only_user_ptr = &onlyuser;
content_ptr->physicWithPermissionConfirm_ptr = malloc(0x18uLL);
content_ptr->physicWithPermissionConfirm_ptr->permission_confirm = 0;
content_ptr->physicWithPermissionConfirm_ptr->content_ptr = 0LL;
content_ptr->physicWithPermissionConfirm_ptr->perm = 0;
```

如果我们能通过程序修改`manage_physic[0]->content_ptr`的内容，就能进一步控制`a1->only_user_ptr`。

当`physicEntry = manage_physic[0]`时，内存布局如下图所示：

![image-20250731162331978](https://bbs.kanxue.com/upload/attach/202508/912098_JB3H9R3W3Z5YRGM.png)

此时，我们修改`**(a1->only_user_ptr + 8)`就是在修改最右侧的结构体，进而实现篡改`only_user_ptr`指针。

### 细节-权限绕过

但我们还没有考虑`check()`函数的权限校验，它可能不允许我们操作下标为0的元素，我们再次进入`check()`函数：

![image-20250731115059408](https://bbs.kanxue.com/upload/attach/202508/912098_YZ3DEPAPSAEDXNQ.png)

如果我们输入`USER_write`，程序要求该元素`perm`的最低为1。如果我们输入`MANAGER_write`，程序要求该元素的次低位为1。

而在`init()`函数中，`manage_physic[0]->perm`被赋值为`16`，无法满足这两个write操作的条件。

这时候，我们只能寄希望于`MANAGER_visit`进行读写操作，它只会去检查`physicEntry->content_ptr`是否非空（在`user_operation()`函数中被赋值）。

此外，想要使用`manager_operation()`函数的`MANAGER_visit`，还需要满足`checkvisit()`函数的条件：

![image-20250731165943088](https://bbs.kanxue.com/upload/attach/202508/912098_HF95E4S2MDYD385.png)

即`permission_confirm`的第3bit只能为`0`，所以`USER_read`不满足条件。

所以，我们现在的思路是尝试在`user_operation()`函数中将`manage_physic[0]`的地址赋值给全局变量`physicEntry`。

在`user_operation()`函数中：

![image-20250731115551107](https://bbs.kanxue.com/upload/attach/202508/912098_VPET5ZS3TCURNQ4.png)

只有通过`check()`函数的检查，我们才能为`physicEntry`变量赋值。

而`check()`函数中，我们使用`Manager_read`或`USER_read`（对应的`permission_confirm`第3bit不为0）和`MANAGER_write`或`USER_write`（`perm`检查不通过）都无法通过检查。

不过，在前面的逆向分析中，我们发现`check()`函数检查并不严格，我们可以在`user_operation()`函数中使用`manager`的`op`。

我们使用`MANAGER_visit`以通过权限检查：

![image-20250731143541779](https://bbs.kanxue.com/upload/attach/202508/912098_JYQCW2XRUK9K2BE.png)

然后，程序会设置`physicEntry->content_ptr = manage_physic[0].content_ptr`，但不会执行后门的任何操作。

### 细节-指针改写

然后，我们开始覆盖`manage_physic[0]->content_ptr->only_user_ptr`指针实现任意地址读写。

由于程序读写`**(a1->only_user_ptr + 8)`，我们需要两个跳板指向以便让`**(a1->only_user_ptr + 8)`指向`target`。

覆盖前：

![image-20250731164025903](https://bbs.kanxue.com/upload/attach/202508/912098_WKMMEZ3N4K7CMJZ.png)

覆盖后：

![image-20250731164326543](https://bbs.kanxue.com/upload/attach/202508/912098_4TWV7AJ4GEEBJJ3.png)

此后，我们只需要修改`chunk_B`中的内容就可以实现任意地址读写操作。

### 细节-`getShell`

通过指针改写，我们已经实现任意地址读写。题目开启`RELRO`无法修改`GOT`，并且所给的`glibc`为2.35版本，没有`hook`函数。

利用思路同常规的题目，我们先泄露`libc`，然后泄露`environ`变量中的栈地址，最后改写返回地址为`ROP`即可。

### 具体利用思路

首先，通过`user_operation()`函数，设置`physicEntry->content_ptr = manager_physic[0].content_ptr`。

脚本如下所示：

```
# 1. physicEntry->content_ptr = manager_physic[0].content_ptr
p.sendlineafter(b'(1:manager 1000:user)\n', b'1000')
p.sendafter(b'logs(USER_write)\n', b'USER_read')
p.sendlineafter(b'can visit\n', b'0')
```

此时，`*(a1->only_user_ptr + 8) = manager_physic[0].content_ptr`。

接着，我们使用`manager_operation()`的`MANAGER_visit`读取出`manager_physic[0]->content_ptr`以泄露`heap`和`elf_base`。

此时，它会依次输出`magic`（4字节）、`pad`（4字节）、`physicWithPermissionConfirm_ptr`（8字节）和`only_user_ptr`（8字节）。

脚本如下所示：

```
# 2. leak heap and elf_base by manager_physic[0]->content_ptr
p.sendlineafter(b'(1:manager 1000:user)\n', b'1')
p.sendafter(b'user(MANAGER_visit)\n', b'MANAGER_visit')
p.sendlineafter(b'can visit\n', b'11')
p.sendlineafter(b'to user_logs\n', b'1')

p.recv(8)

leak_addr = u64(p.recv(8))
heap_base = leak_addr - 0x22d0
success("leak_addr = " + hex(leak_addr))

elf_base = u64(p.recv(8)) - 0x41A0
success("elf_base = " + hex(elf_base))
```

然后，我们开始覆盖`manage_physic[0]->content_ptr->only_user_ptr`指针实现任意地址读写。

由于程序读写`**(a1->only_user_ptr + 8)`，我们需要两个跳板指向以便让`**(a1->only_user_ptr + 8)`指向`target`。

覆盖前：

![image-20250731164025903](https://bbs.kanxue.com/upload/attach/202508/912098_WKMMEZ3N4K7CMJZ.png)

覆盖后：

![image-20250731164326543](https://bbs.kanxue.com/upload/attach/202508/912098_4TWV7AJ4GEEBJJ3.png)

通过动态调试，先通过拿到`manage_physic[11]->content_ptr`作为`chunk_A`，`manage_physic[12]->content_ptr`作为`chunk_B`。

接着依次改写，脚本如下所示：

