
# [原创]Win PE系列之重定位表解析与模块注入技术

## 一.重定位表的作用

考虑下面这段代码

```
	char szHello[] = { "Hello 1900" };

00111038  mov         eax,dword ptr ds:[001881C0h]  

0011103D  mov         dword ptr [ebp-10h],eax  

00111040  mov         ecx,dword ptr ds:[001881C4h]  

00111046  mov         dword ptr [ebp-0Ch],ecx  

00111049  mov         dx,word ptr ds:[001881C8h]  

00111050  mov         word ptr [ebp-8],dx  

00111054  mov         al,byte ptr ds:[001881CAh]  

00111059  mov         byte ptr [ebp-6],al
```

为了给数组`szHello`赋值，程序从0x01881C0地址处开始依次取出相应的数据赋给数组`szHello`，而这个地址中，保存的就是要赋值给数组的字符串

![](https://bbs.kanxue.com/upload/attach/202110/835440_BFYWX8HMNWF4WZ7.jpg)

但问题是，由于程序装载到内存中的基地址，也就是`ImageBase`是不一样的(dll文件和开了ASLR的exe文件)，这就导致了这个字符串的地址会发生改变。重新编译上面的程序，会得到如下的结果

```
	char szHello[] = { "Hello 1900" };

003A1038  mov         eax,dword ptr ds:[004181C0h]  

003A103D  mov         dword ptr [ebp-10h],eax  

003A1040  mov         ecx,dword ptr ds:[004181C4h]  

003A1046  mov         dword ptr [ebp-0Ch],ecx  

003A1049  mov         dx,word ptr ds:[004181C8h]  

003A1050  mov         word ptr [ebp-8],dx  

003A1054  mov         al,byte ptr ds:[004181CAh]  

003A1059  mov         byte ptr [ebp-6],al
```

![](https://bbs.kanxue.com/upload/attach/202110/FKE4HJVPY8H4ZQJ.jpg)

可以看到，此时的字符串的地址变成了0x004181C0。所以为了让程序运行的时候找到正确的字符串的地址，这就需要一块内存来保存所有引用了这个字符串地址的地址。当程序加载的时候，对这个地址进行重定位来找到正确的字符串的地址，这块内存就是重定位表。

## 二.重定位表的解析

重定位表位于数据目录的第6项，在文档中的定义如下

```
#define IMAGE_DIRECTORY_ENTRY_BASERELOC       5   // Base Relocation Table
```

所指的地址指向了`IMAGE_BASE_RELOCATION`结构体，结构体的定义如下

```
typedef struct _IMAGE_BASE_RELOCATION {

    DWORD   VirtualAddress;

    DWORD   SizeOfBlock;

//  WORD    TypeOffset[1];

} IMAGE_BASE_RELOCATION;

typedef IMAGE_BASE_RELOCATION UNALIGNED * PIMAGE_BASE_RELOCATION;
```

这些需要重定位的地址都是按块保存的，每一个块都是一个`IMAGE_BASE_RELOCATION`后面跟上若干两字节的偏移地址。当找个一个块，它的`VirtualAdreess`和`SizeOfBlock`都是0的时候，说明已经没有重定位块要解析了。

而偏移地址有多少，就需要根据`IMAGE_BASE_RELOCATION`中的`SizeOfBlock`来决定，这个字段说明的是整个重定位块的大小，减去`VirtualAddress`和`SizeOfBlock`的占的8字节，在除以2就可以得到偏移地址的个数。

每个偏移地址的高4位如果是3的话，那么它的低12位就代表了偏移地址，这个偏移地址加上这个块的`IMAGE_BASE_RELOCATION`中的`VirtualAddress`就得到了需要进行重定位的地址。

所以可以用下面的代码来解析重定位表

```
void PEParse::PrintRelocationTable()

{

	PIMAGE_BASE_RELOCATION pRelocationTable = this->pRelocationTable;

	DWORD dwSizeOfBlock = 0, i = 0, dwVirtualAddress = 0, dwNum = 0;

	PWORD pAddr = NULL;



	if (pRelocationTable == NULL)

	{

		printf("The file does not have relocation table.\n");

		goto exit;

	}



	printf("==================The relocation table information===================\n");

	while (pRelocationTable->VirtualAddress != 0 || pRelocationTable->SizeOfBlock != 0)

	{

		dwVirtualAddress = pRelocationTable->VirtualAddress;

		dwSizeOfBlock = pRelocationTable->SizeOfBlock;

		dwNum = (dwSizeOfBlock - 2 * sizeof(DWORD)) / 2;

		pAddr = (PWORD)((DWORD)pRelocationTable + 8);

		for (i = 0; i < dwNum; i++)

		{

			if (pAddr[i] & 0x3000)

			{

				printf("The address of need to be relocation: 0x%X\n", this->RVAToFOA(dwVirtualAddress + (pAddr[i] & 0x0FFF)));

			}

		}

		pRelocationTable = (PIMAGE_BASE_RELOCATION)((DWORD)pRelocationTable + dwSizeOfBlock);

	}

	printf("==================The relocation table information===================\n");

exit:;

}
```

部分输出结果如下

![](https://bbs.kanxue.com/upload/attach/202110/835440_MS9A5QF7DS3N73R.jpg)

## 三.模块注入

实现模块注入的相应过程如下

1. 将要注入的模块在内存中的PE状态读入到内存中

2. 将被注入的程序启动，并在它的进程空间中申请可以容纳注入的模块大小的空间

3. 根据申请到的地址，去重定位读入到内存中的注入的模块的重定位表

4. 将重定位以后的内存中的内容写入到申请到的地址

5. 通过创建远程线程的方式启动要运行的模块

6. 运行的模块中有修复`IAT`表的操作

其中的第三步，修复重定位的操作是，获取要重定位的数据的地址。计算第二步申请到的地址与注入的模块的基地址相减得到的结果与重定位数据进行相加。

而第6步中的修复`IAT`表，就是遍历`INT`和`IAT`的同时，使用`LoadLibrary`和`GetProcAddress`来获得正确的函数地址，在填入`IAT`表中。

具体代码如下
