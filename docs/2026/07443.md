
# [原创] 不调用系统api读写物理内存的方法

话不多说，直接上代码

```
PVOID Mapper = 0;

PTE_64* MapperPte = 0;

u64 MapperPteOrigPfn = 0;



VOID Init()

{

	Mapper = MmAllocateIndependentPages(0x1000, -1);

	memset(Mapper, 0, 0x1000);

	MapperPte = MiGetPteAddress(Mapper);

	MapperPteOrigPfn = MapperPte->PageFrameNumber;

}



VOID ReadPhysicalMemoryInPage(PVOID Buffer, u64 Phys, u64 Size)

{

	_disable();



	MapperPte->PageFrameNumber = Phys >> PAGE_SHIFT;

	__invlpg(Mapper);



	__movsb((PUCHAR)Buffer, (PUCHAR)Mapper + (Phys & 0xFFF), Size);



	MapperPte->PageFrameNumber = MapperPteOrigPfn;



	_enable();

}
```

## 原理

- 先分配一个`Mapper`，用`MmAllocateIndependentPages`的原因是不会分配出大页，用`ExAllocatePool`等api是有概率分配出大页`Pde`内存的哦
- 获取`Mapper`的`Pte`并保存原始`Pfn`，用于后续还原
- 改`Pte->Pfn`使得`Mapper`映射我们的物理地址，刷`tlb`，拷贝内存，还原`Pfn`
- 关中断的原因是`invlpg`之后切到其他核心会导致在其他核心的`tlb`没被刷新从而拷贝到错误的数据，其实提升`irql`就行了

## 注意事项

- 多核情况要注意分配多个`Mapper`给每个核心使用
- 仅拷贝一个页面，跨页要自己处理
- 未修复内存缓存属性，读物理内存没什么问题，写入物理内存就要注意了，可以通过`pte.pat`实现

## 总结

- 非常简单的手动映射物理内存读写
- 其实类似的代码在各个开源的VT上多少有出现过
- UC上的人还在用`MmCopy...`

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#系统内核](https://bbs.kanxue.com/forum-41-1-131.htm)

[#驱动开发](https://bbs.kanxue.com/forum-41-1-132.htm)
