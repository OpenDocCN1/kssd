
# 任务门进1环

## 0x1：准备工作

**需要了解**：段任务段、任务门、权限。

打开`windbg`，输入下面指令：

- 构造一系列需要的段描述符：
  - 任务门：`eq 8003f500 0000e500`00C30000`
  - 任务段：`eq 8003f0c0 0000a912`FDCC0068`
  - `cs`：`eq 8003f090 00cfbb00`0000ffff`
  - `ds`：`eq 8003f0b0 00cfb300`0000fff`

## 0x2：代码

```c
#include <STDIO.H>

#include <WINDOWS.H>

DWORD dwOK;

DWORD dwESP;

DWORD dwCS;

void __declspec(naked) func()

{

	dwOK = 1;

	__asm

	{

		//int 3

	    mov eax,esp

            mov dwESP,eax

            mov ax,cs

            mov word ptr [dwCS],ax

            iretd

	}

}



int main(int argc, char* argv[])

{

	char bu[0x10];  //12ff70

	//char esp0[0x10];  //0012FF60

	int iCr3;

	

	printf("input CR3:\n");

	scanf("%x", &iCr3);

	

	//0x0012FDCC

	DWORD iTss[0x68] = {

		        0x00000000,		//link

			0x00000000,		//esp0		//(DWORD)bu

			0x00000000,		//ss0

			(DWORD)bu+4,		//esp1

			0x000000B1,		//ss1

			0x00000000,		//esp2

			0x00000000,		//ss2

			(DWORD)iCr3,	//cr3

			(DWORD)func,	//eip

			0x00000000,		//eflags

			0x00000000,		//eax

			0x00000000,		//ecx

			0x00000000,		//edx

			0x00000000,		//ebx

			(DWORD)bu,		//esp

			0x00000000,		//ebp

			0x00000000,		//esi

			0x00000000,		//edi

			0x00000023,		//es

			0x00000091,		//cs		0x0000001B //0huan 0x00000008

			0x000000B1,		//ss		0x00000023 //0huan 0x00000010

			0x00000023,		//ds

			0x000000b9,		//fs

			0x00000000,		//gs

			0x00000000,		//dit

			0x20ac0000};

		

	char buff[6];

		

	*(DWORD*)&buff[0] = 0x12345678;

	*(WORD*)&buff[4] = 0xC0;

		

	__asm

	{

	    int 0x20

	}

		

	printf("ok=%d \t ESP=%x \t CS=%x \n", dwOK, dwESP, dwCS);

		

	//getchar();

	return 0;

}
```

## 0x3：运行与验证

- 运行你的程序（停留在等待输入`CR3`那里），然后记住自己程序的名字，比如`1.exe`。
- 打开`windbg`，输入`!process 0 0`，找到对应的`CR3`（红线部分）。
- 在`windbg`的`image`那栏找到你自己程序的名字。
- 看控制台输出内容是不是自己构造的1环段选择子，是就说明成功了。

![图片](https://bbs.kanxue.com/upload/attach/202004/768344_AMKGRRE2U4X7NSK.jpg)

## 0x4：Tss

![图片](https://bbs.kanxue.com/upload/attach/202004/768344_VH9N69MBGAQXWR2.jpg)

### 总结

1. 上图中使用任务门进入1环时候，`esp`和`ss`不需要赋值，程序直接取得就是`esp`和`ss`（经过测验得出结论，不知道原因）。
2. `fs`：3环1环都可以正常运行。
3. `cs`：代码段描述符（`dpl = 1`）。
4. `ss`：数据段描述符就可以（`dpl = 1`）。
5. 其他值都是我做测试用的，无视掉就可以。
6. 总结下：修改`cs`、`ss`、`esp`就可以成功进入1环。
7. 其他门可以直接修改`esp 0,1,2`和`ss 0,1,2`（不得原因 = = 有兴趣的大家可以测试下）。
8. 蓝蓝（你懂的）更健康，哈哈。

![图片](https://bbs.kanxue.com/upload/attach/202004/768344_2DXKQ3WSQ48TSE7.jpg)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
