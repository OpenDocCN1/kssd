
使用`main2.cpp`和`injector.cpp`编译出 exe 文件，先执行`main2`取消被钩住的进程，再使用`injector`注入`dllmain.cpp`编译出的动态链接库，达到效果。

同样做的时候发现了不稳定的因素，有时候可以使用，有时候不能使用，因此采取第二种方式来解决。

同样要先把所有钩子卸掉，最好的办法还是重启那些已经被钩住的程序。

这部分程序与之前`main1-2.cpp`中的区别仅仅是少了 `system("taskkill /f /im WorkingService.exe");` 而已。

然后把上面的 dll 注入到两个进程中去，这一段代码没有变化。

通过火绒剑的事件分析可以发现，文件创建的频率明显下降很多（因为每次强制休息了 500MS）：

![](https://bbs.kanxue.com/upload/attach/202402/919002_AXV7AZ67BY4RM6M.jpg)

并且没有影响进程功能，CPU 占用率也大大降低了。

## 实现相同功能

首先就要分析它写入了哪四个文件了，从火绒剑的分析来看，应该是有四个线程在不同的写，因为很容易注意到，四个文件几乎是同时打开的。

对进程进行内存 dump，拖入 IDA 分析，发现一串 base64 的表，于是在 CE 中找到主要编码的函数。

![](https://bbs.kanxue.com/upload/attach/202402/919002_WTPKRBJG9TKD2UF.jpg)

通过参数分析找到了内存的位置，通过仔细观察，发现和初赛的 flag：`catchmeifyoucan` 十分相似。

经过分析，发现它们存在互补关系。

![](https://bbs.kanxue.com/upload/attach/202402/919002_BMJUMUVEVCWU24F.jpg)

通过某次虚拟机炸了之后，意外得到了两份落地的文件。

![](https://bbs.kanxue.com/upload/attach/202402/919002_3R7UV9VEUG9H95J.jpg)

经过 base64 解码发现，其中一个是 `catchmeifyoucan` + `id`，一个是不可见的字符，而且刚好等于上面不可见字符的补集。

![](https://bbs.kanxue.com/upload/attach/202402/919002_N85DHC4Z52UQ2E6.jpg)

![](https://bbs.kanxue.com/upload/attach/202402/919002_74RF68B57CQHVXC.jpg)

本来并没有什么头绪，但是发现了一个细节，在火绒剑当中：

![](https://bbs.kanxue.com/upload/attach/202402/919002_D5K6NEYG9RQDWYY.jpg)

可以发现`contestxxxx.txt`的 `xxxx` 就是线程的 `id`。

大概分析了一下，应该是有两串 `content`，根据线程 `id`，选择到底是 ASCII 字符还是非 ASCII 字符，然后 base64 后 + `TID`。

经过多次测试之后发现，只会出现两种：两种互补，就像这样子：

![](https://bbs.kanxue.com/upload/attach/202402/919002_AKJ55QEYF6TMKKS.jpg)

根据逻辑分析，是计算 `TID` 的总和，然后根据位数，选择这一位到底是选择 ASCII 还是非 ASCII 字符。

根据之前进程的行为写出了以下的程序，这里需要 `CREATE_ALWAYS` 和 `FILE_FLAG_DELETE_ON_CLOSE` 两个标志位，以便于达到和之前进程一样的行为。

```cpp
#include <stdio.h>
#include <windows.h>

char flag1[] = "catchmeifyoucan";
char flag2[] = "\x15\x17\x02\x15\x1e\x1b\x13\x1f\x10\x0f\x19\x03\x15\x17\x18";
DWORD written;
const char* BASE64_CHARS = "ABCDEFGHIJKLMNOPwxyz0123456789+/ghijklmnopqrstuvQRSTUVWXYZabcdef";
DWORD all = 0;

void base64_encode(const char* input, char* output) {
    int i, j;
    unsigned char buf[3];
    int num_bits = 0, bits;
    size_t len = strlen(input);

    for (i = 0, j = 0; i < len; i++) {
        buf[num_bits++] = input[i];
        if (num_bits == 3) {
            bits = (buf[0] << 16) | (buf[1] << 8) | buf[2];
            output[j++] = BASE64_CHARS[(bits >> 18) & 0x3F];
            output[j++] = BASE64_CHARS[(bits >> 12) & 0x3F];
            output[j++] = BASE64_CHARS[(bits >> 6) & 0x3F];
            output[j++] = BASE64_CHARS[bits & 0x3F];
            num_bits = 0;
        }
    }

    if (num_bits > 0) {
        bits = (buf[0] << 16) | ((num_bits == 2) ? (buf[1] << 8) : 0);
        output[j++] = BASE64_CHARS[(bits >> 18) & 0x3F];
        output[j++] = BASE64_CHARS[(bits >> 12) & 0x3F];
        output[j++] = (num_bits == 1) ? '=' : BASE64_CHARS[(bits >> 6) & 0x3F];
        output[j++] = '=';
    }

    output[j] = '\0';
}

void process(LPVOID f) {
    DWORD TID = GetCurrentThreadId();
    HANDLE hFile = NULL;
    char filename[20];
    char content[50];
    char c[50];
    char tmp[50];
    sprintf(filename, "contest%d.txt", TID);
    while (!all);
    DWORD x = all;
    printf("%d\n", *(int*)f);
    for (int i = 0; i < 15; i++) {
        if ((*(int*)f % 2)^(x&1)) {
            c[i] = flag1[i];
        } else {
            c[i] = flag2[i];
        }
        x >>= 1;
    }
    c[15] = 0;
    base64_encode(c, tmp);
    sprintf(content, "%s%d", tmp, TID);
    //printf(content);
    do {
        hFile = CreateFileA(filename, GENERIC_WRITE, 0, NULL, CREATE_ALWAYS, FILE_FLAG_DELETE_ON_CLOSE, NULL);
        if (hFile != (HANDLE)-1) WriteFile(hFile, content, strlen(content), &written, 0);
        Sleep(1000);
    } while (1);

    CloseHandle(hFile);
}

int main()
{
    HANDLE hThread[4];
    DWORD ans=0;
    int k[] = {0,1,2,3};
    for (int i = 0; i < 4; i++) {
        DWORD dwThreadId;
        hThread[i] = CreateThread(
            NULL, // lpThreadAttributes 参数，一般传递 NULL
            0, // dwStackSize 参数，线程堆栈大小，一般为 0
            (LPTHREAD_START_ROUTINE)process, // lpStartAddress 参数，线程函数
            &k[i], // lpParameter 参数，线程函数的参数，这里不需要传递参数
            0, // dwCreationFlags 参数，线程的创建标志，一般为 0
            &dwThreadId // lpThreadId 参数，线程 ID 的输出参数
        );
        ans += dwThreadId;
    }
    all = ans&0xffff;
    for (int i = 0; i < 4; i++) {
        WaitForSingleObject(hThread[i], -1);
    }
    return 0;
}
```

在附件中，编译`main3.cpp`，运行可以直接达到效果，并且拥有很低的 CPU 占用率。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
