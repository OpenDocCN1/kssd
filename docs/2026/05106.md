
# [原创]XueTr的一个BSOD

```c
#include <Windows.h>

#include <stdio.h>

void main( )

{

	for ( int i = 0x1000; i < 0x80000000; i = i + 0x1000 )

	{

		VirtualAllocEx( (HANDLE)-1, (PVOID)i, 0x1000, MEM_RESERVE, PAGE_EXECUTE_READWRITE );

	}

	printf("完了");

	getchar();

	return;

}
```

编译以上代码为`XXX.exe`，在`Windows XP SP3`下双击运行，显示“完了”后，打开`XueTr`，右键`XXX.exe`，依次点击“查看...”->“查看进程内存”，即可蓝屏。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
