
```cpp
// 07.0地址执行代码.cpp : 定义控制台应用程序的入口点。

//





#include "stdafx.h"

#include <Windows.h>



int _tmain(int argc, _TCHAR* argv[])

{

	int*x =  NULL;

	PVOID mem = VirtualAlloc(0,0x100,MEM_COMMIT,PAGE_EXECUTE_READWRITE);  //申请内存



	printf("%x\r\n	",mem);  

	char bufcode[]=

	{

		//硬编码

		0x6A,0,

		0x6A,0,

		0x6A,0,

		0x6A,0,

		0xb8,0,0,0,0,

		0xff,0xd0,

		0xc2,4,0

		

	};



	char user32[]={'u','s','e','r','3','2','.','d','l','l',0};

	char MessageBoxAA[]={"MessageBoxA"};



	HMODULE hmodule = LoadLibraryA(user32);

	ULONG messageBox = (ULONG)GetProcAddress(hmodule,MessageBoxAA);

	*(PULONG)&bufcode[9] = messageBox;

	memcpy(mem,bufcode,sizeof(bufcode));



	system("pause");

	

	HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE,3576);  //另一个进程的pid



	HANDLE hThread = CreateRemoteThread(hProcess,NULL,NULL,0,NULL,NULL,NULL);

	

	CloseHandle(hThread);

	system("pause");



	printf("%x\r\n",*x);

	

	system("pause");

	



	return 0;

}
```

### 3.挂物理页

把A进程的PTE挂到test进程的0地址

![](https://bbs.kanxue.com/upload/attach/202208/939298_XTAGGWBX5XE4ZWS.jpg)

### 1.11.页属性

![](https://bbs.kanxue.com/upload/attach/202208/939298_ENC3RUQSB8B35GJ.jpg)

**P**

```
0：表示不在内存中
1：表示在物理内存中
```

**R/W**

```
0：只读
1：可读可写
```

**U/S**

```
0：该页为超级用户权限
1：该页为普通用户权限
```

**A**

```
是否曾经被访问过。
```

**D**

```
是否曾经被写入过
```

**PS**

```
0：页尺寸为4KB，页目录项指向一个页表
1：大页，页尺寸为32位寻址的4MB，页目录项指向一个页
```

**G**

```
全局页。当一个页被表明为全局，并且CR4中的启用全局页（PGE）标志被置位时，一旦CR3寄存器被
载入或者发生任务切换，TLB中的页表或者指向页的目录项并不失效。这个标志可以防止使TLB中频繁
使用的页失效
```

### 1.12.页基址

1.分析`MiIsAddressValid`函数

![](https://bbs.kanxue.com/upload/attach/202208/939298_XUBJYH8HJ3TGCTH.jpg)

2.通过上面分析得出结论

- `MiIsAddressValid`函数只有一个参数，就是线性地址。
- 判断一个线性地址是否有效，分大页和不是大页两种情况

```
线性地址是否有效

#大页

PDE.p=1, PS=1 ,则有效

#不是大页

PDE.p=1，PS=0， PTE.p=1， PTE.PAT=0 则有效
```

- PDE的页基址位`C0300000`

```
#PDE计算

线性地址右移20位，然后and FFC

PDE：C0300000+上面的结果
```

- PTE的页基址位`C0000000`

```
#PTE计算

线性地址右移10位，然后and 3FFFFC

PDE：C0000000+上面的结果
```

3.实验证明页基址，拿gdtr当线性地址80b93000

```
线性地址：80b93000

cr3= 7760e000

==================================

!dd 7760e000+202*4    第一个10

===================================

0018a063      把后面三位去掉补000， 结果：0018a000

!dd 0018a000 +393*4

===================================

00b93163     把后面三位去掉补000， 结果：00b93000

!dd 00b93000
```

![](https://bbs.kanxue.com/upload/attach/202208/939298_JKH84BVXJ86GK4R.jpg)

用页基址的方式查找

```
线性地址：80b93000

#PDE

80b93000 >> 20   #右移20得到80b

80b & FFC    	 #得到808, PDE：C0300000+808

#PTE

80b93*4 & 3FFFFC   #得到202E4C，PTE：C0000000+202E4C
```

![](https://bbs.kanxue.com/upload/attach/202208/939298_CSVH984WMK38EB5.jpg)

### 1.13.29912分页

#### 1.13.1.拆分线性地址

1.用ce搜索记事本内本，得到线性地址002244E8

![](https://bbs.kanxue.com/upload/attach/202208/939298_JAFQ94SJBXZNP3Y.jpg)

2.拆分线性地址

```
线性地址：00224 4E8

0000 0000 0010 0010 1000 

00	      ->0		
00 0000 001   ->1
0 0010 1000   ->24
4e8                ->4e8

CR3：7edf85c0

PDPTE
!dq 7edf85c0+0*8

PDE
!dq 00000000`653df000+1*8

PTE
!dq 00000000`23b12000+24*8

物理页
!dq 80000000`22c70000+4e8
```

![](https://bbs.kanxue.com/upload/attach/202208/939298_2EDU8J7WCFZBWY5.jpg)

#### 1.13.2.页基址

分析`MiIsAddressValid`函数

```
PDE页基址：0xC0600000
PTE页基址：0XC0000000
```

![](https://bbs.kanxue.com/upload/attach/202208/939298_HPVCWDD97VVXCSX.jpg)

![](https://bbs.kanxue.com/upload/attach/202208/939298_FBDM9W2JJYQPFKC.jpg)

#### 1.13.3.页属性

```
#XD位

XD=1：不可执行
XD=0：可执行
```

![](https://bbs.kanxue.com/upload/attach/202208/939298_VHGM69PWPHUUDCJ.jpg)

### 1.14.缓存

1.WC,WB,WT

```
WC写组合，从L1拿地址，采用hash加密，一次64个字节

WB(write-back)回写，L1缓存，一次4个字节

WT直写
```

2.PAT,PCD,PWT

`PAT(Page Attribute Table)`

![](https://bbs.kanxue.com/upload/attach/202208/939298_KQDMYARWABW7KPS.jpg)

### 1.15.TLB

验证TLB的存在

```
1.申请2个地址，分别赋值
2.把第一个地址挂在0地址上，然后访问，取出结果保存变量
3.把第二个地址挂在0地址上，然后访问，取出结果保存变量
4.观察变量1，变量2的结果有什么不同
```

实验

```cpp
// 08.TLB.cpp : 定义控制台应用程序的入口点。

#include "stdafx.h"
#include <Windows.h>

PUCHAR addr1 = NULL;
PUCHAR addr2 = NULL;
ULONG temp1=0,temp2 = 0;

void __declspec(naked) test1()
{
	__asm
	{
		pushad;
		pushfd;
		push 0x30;
		pop fs;

		mov eax,[addr1];//线性地址1
		shr eax,0x9;    //下面三行汇编代码是取PTE，29912分页
		and eax,0x7ffff8;
		add eax,0xc0000000;
		mov ecx,[eax];   //取PTE低4字节
		mov edx,[eax+4]; //取PTE高4字节
		or ecx,0x100;    //把PTE的G位改成1
		mov dword ptr ds:[0xc0000000],ecx;
		mov dword ptr ds:[0xc0000004],edx;
		mov eax,dword ptr ds:[0];
		mov [temp1],eax;

		//mov eax,cr3;
		//mov cr3,eax;   //刷新cr3

		mov eax,[addr2];//线性地址2
		shr eax,0x9;
		and eax,0x7ffff8;
		add eax,0xc0000000;
		mov ecx,[eax];   //取低4字节
		mov edx,[eax+4]; //取高4字节

		mov dword ptr ds:[0xc0000000],ecx;
		mov dword ptr ds:[0xc0000004],edx;

		invlpg dword ptr ds:[0]  //清除0地址页的TLB项目

		mov eax,dword ptr ds:[0];
		mov [temp2],eax;

		push 0x3b;
		pop fs;
		popfd;
		popad;

		retf;
	}
}

int _tmain(int argc, _TCHAR* argv[])
{
	addr1 = (PUCHAR)VirtualAlloc(NULL,0x1000,MEM_COMMIT,PAGE_EXECUTE_READWRITE);
	addr2 = (PUCHAR)VirtualAlloc(NULL,0x1000,MEM_COMMIT,PAGE_EXECUTE_READWRITE);
	*(PULONG)addr1 = 0x1000;
	*(PULONG)addr2 = 0x2000;

	temp1 = 0;
	temp2 = 0;

	printf("addr1 = %x,addr2 = %x,test1=%x\r\n",addr1,addr2,test1);
	system("pause");

	char bufcode[]={0,0,0,0,0x48,0};   //调佣门

	__asm
	{
		call fword ptr bufcode;
	}
	system("pause");
	printf("temp1 = %x,temp2 = %x\r\n",temp1,temp2);
	system("pause");

	return 0;
}
```

![](https://bbs.kanxue.com/upload/attach/202208/939298_EJ3C94WQQWD4674.jpg)

### 1.16.控制寄存器

![](https://bbs.kanxue.com/upload/attach/202208/939298_YUBF9R237X7KMXT.jpg)

`CR2`

```
保存出现异常的线性地址
```

`CR0`

```
PE=1：开启保护模式
PG=1：开启分页模式
CD=1：关闭所有缓存
AM：对齐位
WP：写保护。wp=0:关闭写保护
```

`CR4`

```
PSE：是否允许大页，如果PSE=0,即使PDE.ps=1大页也不生效
PAE：PAE=0是101012分页，PAE=1是29912分页
PGE：页表的G位是否有效，如果PGE=0，即使G=1也无效
SMAP：super mode execute protected #等于1（必须CR0的AM=1）,内核（U/S=0）不能执行3环（U/S=1）的代码
SMEO：super mode access protected  #等于1，内核（U/S=0）不能访问3环（U/S=1）的代码
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2022-8-11 16:36

被zhang_derek编辑

，原因：

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)
