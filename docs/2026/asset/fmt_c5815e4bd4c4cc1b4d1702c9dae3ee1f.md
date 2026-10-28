
```cpp
struct vm_operations_struct {
void(*open)(struct vm_area_struct * area);
void(*close)(struct vm_area_struct * area);
int(*mremap)(struct vm_area_struct * area);
int(*fault)(struct vm_area_struct *vma, struct vm_fault *vmf);
int(*pmd_fault)(struct vm_area_struct *, unsigned long address, pmd_t *, unsigned int flags);
void(*map_pages)(struct fault_env *fe, pgoff_t start_pgoff, pgoff_t end_pgoff);
int(*page_mkwrite)(struct vm_area_struct *vma, struct vm_fault *vmf);
int(*pfn_mkwrite)(struct vm_area_struct *vma, struct vm_fault *vmf);
int(*access)(struct vm_area_struct *vma, unsigned long addr, void *buf, int len, int write);
const char *(*name)(struct vm_area_struct *vma);
#ifdef CONFIG_NUMA
int(*set_policy)(struct vm_area_struct *vma, struct mempolicy *new);
struct mempolicy *(*get_policy)(struct vm_area_struct *vma, unsigned long addr);
#endif
struct page *(*find_special_page)(struct vm_area_struct *vma, unsigned long addr);
};
```

如上文描述，这些函数指针可以用于实现特定的handler。关于此的详细描述在《Linux Device Drivers（Linux设备驱动）》一书中可以找到。

在实现内存分配器时，一个通俗可见的主流的行为是开发者实现了一个`fault` handler。例如，看看这一段：

```cpp
static struct file_operations fops = {
.open = dev_open,
.mmap = simple_vma_ops_mmap,
.release = dev_release,
};
static struct vm_operations_struct simple_remap_vm_ops = {
.open = simple_vma_open,
.close = simple_vma_close,
.fault = simple_vma_fault,
};
static int simple_vma_ops_mmap(struct file *filp, struct vm_area_struct *vma)
{
printk(KERN_INFO "MWR: Device simple_vma_ops_mmap\n");
vma->vm_private_data = filp->private_data;
vma->vm_ops = &simple_remap_vm_ops;
simple_vma_open(vma);
printk(KERN_INFO "MWR: Device mmap OK\n");
return 0;
}
void simple_vma_open(struct vm_area_struct *vma)
{
printk(KERN_NOTICE "MWR: Simple VMA open, virt %lx, phys %lx\n", vma->vm_start, vma->vm_pgoff << PAGE_SHIFT);
}
void simple_vma_close(struct vm_area_struct *vma)
{
printk(KERN_NOTICE "MWR: Simple VMA close.\n");
}
int simple_vma_fault(struct vm_area_struct *vma, struct vm_fault *vmf)
{
struct page *page = NULL;
unsigned long offset;
printk(KERN_NOTICE "MWR: simple_vma_fault\n");
offset = (((unsigned long)vmf->virtual_address - vma->vm_start) + (vma->vm_pgoff << PAGE_SHIFT));
if (offset > PAGE_SIZE << 4)
goto nopage_out;
page = virt_to_page(vma->vm_private_data + offset);
vmf->page = page;
get_page(page);
nopage_out:
return 0;
}
```

上述代码中我们可以看到`simple_vma_ops_mmap`函数用于控制`mmap`调用。它什么都没做，除了指派了`simple_remap_vm_ops`结构体作为虚拟内存操作的handler。

让我们看看下列代码在该driver上运行的效果：

```cpp
int fd = open("/dev/MWR_DEVICE", O_RDWR);
unsigned long size = 0x1000;
unsigned long * addr = (unsigned long *)mmap((void*)0x42424000, size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0x1000);
```

dmesg的结果：

```sh
[268819.067085] MWR: Device has been opened 2 time(s)
[268819.067121] MWR: Device simple_vma_ops_mmap
[268819.067123] MWR: Simple VMA open, virt 42424000, phys 1000
[268819.067125] MWR: Device mmap OK
```

进程地址空间的映射：

```sh
42424000-42425000 rw-s 00001000 00:06 140215 /dev/MWR_DEVICE
```

如我们所见，`simple_vma_ops_mmap`函数被调用了，内存映射也创建了。例子中`simple_vma_fault`函数没有被调用。问题在于，我们有了个地址范围为0x42424000-0x42425000的地址空间却不清楚它指向何处。我们没有为它关联物理内存，因此当进程试图访问这段地址的任一部分时，`simple_vma_fault`都会执行。

所以让我们看看这段用户空间代码：

```cpp
int fd = open("/dev/MWR_DEVICE", O_RDWR);
unsigned long size = 0x2000;
unsigned long * addr = (unsigned long *)mmap((void*)0x42424000, size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0x1000);
printf("addr[0]: %x\n", addr[0]);
```

代码唯一的改变在于使用了`printf`函数来访问映射内存。因为内存区域无效所以我们的`simple_vma_fault`例程被调用了。`dmesg`的输出可以看到：

```sh
[285305.468520] MWR: Device has been opened 3 time(s)
[285305.468537] MWR: Device simple_vma_ops_mmap
[285305.468538] MWR: Simple VMA open, virt 42424000, phys 1000
[285305.468539] MWR: Device mmap OK
[285305.468546] MWR: simple_vma_fault
```

在`simple_vma_fault`函数中，我们可以观察到`offset`变量使用了指向一个没有被映射的地址的`vmf->virtual_address`进行了计算。我们这里就是`addr[0]`的地址。下一个`page`结构体由`virt_to_page`宏得到，该宏将新获取的`page`赋值给`vmf->page`变量。这一赋值意味着当fault handler返回时，`addr[0]`会指向由`simple_vma_fault`计算出来的某个物理地址。该内存可以被用户进程所访问而无需其他任何代码。如果程序试图访问`addr[513]`（假定`sizeof(unsigned long)`为8），fault handler会被再次调用，这是由于`addr[0]`和`addr[513]`在两个不同的内存页上，而此前仅有一个内存页被映射过。

这就是源码：

```cpp
int fd = open("/dev/MWR_DEVICE", O_RDWR);
unsigned long size = 0x2000;
unsigned long * addr = (unsigned long *)mmap((void*)0x42424000, size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0x1000);
printf("addr[0]: %x\n", addr[0]);
printf("addr[513]: %x\n", addr[513]);
```

生成内核log：

```sh
[286873.855849] MWR: Device has been opened 4 time(s)
[286873.855976] MWR: Device simple_vma_ops_mmap
[286873.855979] MWR: Simple VMA open, virt 42424000, phys 1000
[286873.855980] MWR: Device mmap OK
[286873.856046] MWR: simple_vma_fault
[286873.856110] MWR: simple_vma_fault
```

## 3. 经典`mmap` Handler议题

### 3.1 用户输入有效性的不足

让我们看看前面的`mmap` handler例子：

```cpp
static int simple_mmap(struct file *filp, struct vm_area_struct *vma)
{
printk(KERN_INFO "MWR: Device mmap\n");
if ( remap_pfn_range( vma, vma->vm_start, virt_to_pfn(filp->private_data), vma->vm_end - vma->vm_start, vma->vm_page_prot ) )
{
printk(KERN_INFO "MWR: Device mmap failed\n");
return -EAGAIN;
}
printk(KERN_INFO "MWR: Device mmap OK\n");
return 0;
}
```

前面展示的代码展示了一个通用的实现`mmap` handler的途径，相似的代码可以在《Linux设备驱动》一书中找到。示例代码主要的议论点在于`vma->vm_end`和`vma->vm_start`的值从未检查有效性。取而代之的，它们被直接传递给`remap_pfn_range`作为尺寸参数。这意味着一个恶意进程可以用一个不受限的尺寸来调用`mmap`。在我们这里，允许一个用户空间进程去`mmap`所有的在`filp->private_data` buffer之后的物理内存地址空间。这包括所有的内核内存。这意味着恶意进程能够从用户空间读写整个内核内存。

另一个流行的用法如下：

```cpp
static int simple_mmap(struct file *filp, struct vm_area_struct *vma)
{
printk(KERN_INFO "MWR: Device mmap\n");
if ( remap_pfn_range( vma, vma->vm_start, vma->vm_pgoff, vma->vm_end - vma->vm_start, vma->vm_page_prot ) )
{
printk(KERN_INFO "MWR: Device mmap failed\n");
return -EAGAIN;
}
printk(KERN_INFO "MWR: Device mmap OK\n");
return 0;
}
```

上面的代码中我们可以看到用户控制的 offset `vma->vm_pgoff`被直接传递给了`remap_pfn_range`函数作为物理地址。这会使得恶意进程有能力传递一个任意物理地址给`mmap`，也就在用户空间拥有了整个内核内存的访问权限。在一些对示例进行微小改动的情景中经常可以看到，要么`offset`有了掩码，要么使用了另外一个值来计算。

### 3.2 整数溢出

经常可以看到开发者试图使用复杂的计算、按位掩码、位移、尺寸和偏移和等方法去验证映射的尺寸和偏移（`size` and `offset`）。不幸的是，这常常导致了创建的复杂性以及不寻常的计算和验证过程晦涩难懂。在对`size`和`offset`值进行少量fuzzing后，找到可以绕过有效性检查的值并非不可能。

让我们看看这段代码：

```cpp
static int integer_overflow_mmap(struct file *filp, struct vm_area_struct *vma)
{
unsigned int vma_size = vma->vm_end - vma->vm_start;
unsigned int offset = vma->vm_pgoff << PAGE_SHIFT;
printk(KERN_INFO "MWR: Device integer_overflow_mmap( vma_size: %x, offset: %x)\n", vma_size, offset);
if (vma_size + offset > 0x10000)
{
printk(KERN_INFO "MWR: mmap failed, requested too large a chunk of memory\n");
return -EAGAIN;
}
if (remap_pfn_range(vma, vma->vm_start, virt_to_pfn(filp->private_data), vma_size, vma->vm_page_prot))
{
printk(KERN_INFO "MWR: Device integer_overflow_mmap failed\n");
return -EAGAIN;
}
printk(KERN_INFO "MWR: Device integer_overflow_mmap OK\n");
return 0;
}
```

上面的代码展示了一个典型的整数溢出漏洞，它发生在一个进程调用`mmap2`系统调用时使用了`size`为0xfffa000以及`offset`为0xf0006的情况。0xfffa000和0xf0006000的和等于0x100000000。由于最大的unsigned int值为0xffffffff，最高符号位会被清掉，最终的和会变成0x0。这种情况下，`mmap`系统调用会成功绕过检查，进程会访问到预期buffer外的内存。如上文提到的，有两个独立的系统调用`mmap`和`mmap2`。`mmap2`使得应用程序可以使用一个32位的`off_t`类型来映射大文件（最大为2^44字节），这是通过支持使用一个大数`offset`参数实现的。有趣的是`mmap2`系统调用通常在64位内核系统调用表中不可用。然而，如果操作系统同时支持32位和64位进程，他就通常在32位进程中可用。这是因为32位和64位进程各使用独立的系统调用表。

### 3.3 有符号整型类型

另一个老生常谈的议题就是`size`变量的有符号类型。让我们看看这段代码：
