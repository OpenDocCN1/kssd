
# [原创]为无源码的DLL增加接口功能

![](upload/attach/202301/940598_BCTED4VJ7PGXPWD.png)

![](upload/attach/202301/940598_TFHED88U8SAH377.png)

---

**完整源代码:**

```cpp
#include <iostream>
#include <thread>
#include <Windows.h>
#pragma warning(disable:4996)
using namespace std;
void __declspec(dllexport) CalculateFunc(int count)
{
int index = 0;
for (int i = 0; i < count; i++)
{
printf("i=%d, %d...\n", i, ++index);
Sleep(1000);
}
printf("Threader Over...\n");
}
int g_indexRef = 0;
void myThread(int count)
{
__asm
{
push eax
mov eax, esp
sub eax, 0x10
mov g_indexRef, eax
push count
call CalculateFunc
add esp,4
pop eax
}
}
void CalculateFuncAsync()
{
std::thread thread(myThread, 100);
thread.join();//等待线程结束
}
int main()
{
std::thread thread(CalculateFuncAsync);
Sleep(3000);
if (g_indexRef)
{
*((int*)g_indexRef) = 100;
thread.join();
}
printf("Main Over...\n");
return 0;
}
```

![](https://bbs.kanxue.com/upload/attach/202301/940598_BCTED4VJ7PGXPWD.png)

![](https://bbs.kanxue.com/upload/attach/202301/940598_TFHED88U8SAH377.png)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2023-1-9 22:26 被_THINCT编辑，原因：

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)

[#开源分享](https://bbs.kanxue.com/forum-41-1-134.htm)

[#开发技巧](https://bbs.kanxue.com/forum-41-1-135.htm)

[#其他](https://bbs.kanxue.com/forum-41-1-138.htm)
