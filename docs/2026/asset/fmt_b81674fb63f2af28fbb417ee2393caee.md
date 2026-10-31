
## 题目概述

分为 5 个 stage，需要依次解决。

## 初始代码

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>

int main(int argc, char* argv[], char* envp[]) {
    printf("Welcome to pwnable.kr\n");
    printf("Let's see if you know how to give input to program\n");
    printf("Just give me correct inputs then you will get the flag :)\n");

    // argv
    if (argc != 100)
        return 0;
    if (strcmp(argv['A'], "\x00"))
        return 0;
    if (strcmp(argv['B'], "\x20\x0a\x0d"))
        return 0;
    printf("Stage 1 clear!\n");

    // stdio
    char buf[4];
    read(0, buf, 4);
    if (memcmp(buf, "\x00\x0a\x00\xff", 4))
        return 0;
    read(2, buf, 4);
    if (memcmp(buf, "\x00\x0a\x02\xff", 4))
        return 0;
    printf("Stage 2 clear!\n");

    // env
    if (strcmp("\xca\xfe\xba\xbe", getenv("\xde\xad\xbe\xef")))
        return 0;
    printf("Stage 3 clear!\n");

    // file
    FILE* fp = fopen("\x0a", "r");
    if (!fp)
        return 0;
    if (fread(buf, 4, 1, fp) != 1)
        return 0;
    if (memcmp(buf, "\x00\x00\x00\x00", 4))
        return 0;
    fclose(fp);
    printf("Stage 4 clear!\n");

    // network
    int sd, cd;
    struct sockaddr_in saddr, caddr;

    sd = socket(AF_INET, SOCK_STREAM, 0);
    if (sd == -1) {
        printf("socket error, tell admin\n");
        return 0;
    }

    saddr.sin_family = AF_INET;
    saddr.sin_addr.s_addr = INADDR_ANY;
    saddr.sin_port = htons(atoi(argv['C']));

    if (bind(sd, (struct sockaddr*)&saddr, sizeof(saddr)) < 0) {
        printf("bind error, use another port\n");
        return 1;
    }

    listen(sd, 1);

    int c = sizeof(struct sockaddr_in);
    cd = accept(sd, (struct sockaddr*)&caddr, (socklen_t*)&c);

    if (cd < 0) {
        printf("accept error, tell admin\n");
        return 0;
    }

    if (recv(cd, buf, 4, 0) != 4)
        return 0;
    if (memcmp(buf, "\xde\xad\xbe\xef", 4))
        return 0;

    printf("Stage 5 clear!\n");

    // here's your flag
    system("/bin/cat flag");
    return 0;
}
```

## 1.1 `argv`

![题目截图](https://bbs.kanxue.com/upload/attach/202304/882195_BRFF3B5RNP47AMF.png)

命令行执行程序调用的是 `execve` 系统调用，其函数原型如下所示。第一个参数是运行程序文件名，第二个参数是 `argv[]`，第三个参数是 `envp[]`。同时，检测的是 `main` 函数的 `argc` 和 `argv`。

```c
int execve(
    const char* filename,
    char* const argv[],
    char* const envp[]
);

int main(int argc, char* argv[]) {
    /* … */
}
```

`main` 函数的第一个参数 `argc` 表示的是命令行参数个数。`argv` 则是命令行参数，`argv[0] = filename`。

```c
// argv
if (argc != 100)
    return 0;
if (strcmp(argv['A'], "\x00"))
    return 0;
if (strcmp(argv['B'], "\x20\x0a\x0d"))
    return 0;
printf("Stage 1 clear!\n");
```

1. `argc == 100`
2. `argv['A'] == '\x00'`
3. `argv['B'] == '\x20\x0a\x0d'`

翻译一下：

```c
char* argv[101] = {"/home/input2/input", [1 ... 99] = "A", NULL};

argv['A'] = "\x00";
argv['B'] = "\x20\x0a\x0d";

execve("/home/input2/input", argv, NULL);
```

## 1.2 `stdio`

```c
// stdio
char buf[4];

read(0, buf, 4);
if (memcmp(buf, "\x00\x0a\x00\xff", 4))
    return 0;

read(2, buf, 4);
if (memcmp(buf, "\x00\x0a\x02\xff", 4))
    return 0;

printf("Stage 2 clear!\n");
```

```c
ssize_t read(int fd, void* buf, size_t count);
```

> `read()` attempts to read up to count bytes from file descriptor fd into the buffer starting at buf.

```c
int memcmp(const void* s1, const void* s2, size_t n);
```

> the `memcmp()` function compares the first n bytes (each interpreted as unsigned char) of the memory areas s1 and s2.

即：

1. 从标准输入流读取字符串写入内存，和 `\x00\x0a\x00\xff` 比较。
2. 从标准错误流读取字符串写入内存，和 `\x00\x0a\x02\xff` 比较。

`fd = 0` 很好解决，`stdin`，直接输入即可；`fd = 2` 的输入不是很好解决，但是 `pwntools` 提供了对应的接口。如果没有 `pwntools`，应该怎么实现？

1. `dup` 函数可以实现 IO 重定向。
2. 使用 `open` 函数可以获取到一个 `fd`，将字符串写入文件。

因此有：

```c
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <stdlib.h>
#include <fcntl.h>

int main() {
    int fd1 = open("./stdin", O_RDONLY);
    int fd2 = open("./stderr", O_RDONLY);

    dup2(fd1, 0);  // 将文件描述符 fd1 重定向到标准输入
    dup2(fd2, 2);  // 将文件描述符 fd2 重定向到标准错误

    close(fd1);
    close(fd2);

    char* argv[101] = {"/home/input2/input", [1 ... 99] = "A", NULL};

    argv['A'] = "\x00";
    argv['B'] = "\x20\x0a\x0d";

    execve("/home/input2/input", argv, NULL);
    return 0;
}
```

## 1.3 `env`

```c
// env
if (strcmp("\xca\xfe\xba\xbe", getenv("\xde\xad\xbe\xef")))
    return 0;
printf("Stage 3 clear!\n");
```

上述 `execve` 函数已经讲过，因此直接添加即可：

```c
char* env[2] = {"\xde\xad\xbe\xef=\xca\xfe\xba\xbe", NULL};

execve("/home/input/input", argv, env);
```

## 1.4 `file`

```c
// file
FILE* fp = fopen("\x0a", "r");
if (!fp)
    return 0;

if (fread(buf, 4, 1, fp) != 1)
    return 0;

if (memcmp(buf, "\x00\x00\x00\x00", 4))
    return 0;

fclose(fp);
printf("Stage 4 clear!\n");
```

基本文件流操作：

```c
FILE* fp = fopen("\x0a", "w");
fwrite("\x00\x00\x00\x00", 4, 1, fp);
fclose(fp);
```

需要注意的是，能写入的目录只有 `/tmp`，如何才能读取到想要的文件是需要考虑一下的。

## 1.5 `network`

```c
// network
int sd, cd;
struct sockaddr_in saddr, caddr;

sd = socket(AF_INET, SOCK_STREAM, 0);
if (sd == -1) {
    printf("socket error, tell admin\n");
    return 0;
}

saddr.sin_family = AF_INET;
saddr.sin_addr.s_addr = INADDR_ANY;
saddr.sin_port = htons(atoi(argv['C']));

if (bind(sd, (struct sockaddr*)&saddr, sizeof(saddr)) < 0) {
    printf("bind error, use another port\n");
    return 1;
}

listen(sd, 1);

int c = sizeof(struct sockaddr_in);
cd = accept(sd, (struct sockaddr*)&caddr, (socklen_t*)&c);

if (cd < 0) {
    printf("accept error, tell admin\n");
    return 0;
}

if (recv(cd, buf, 4, 0) != 4)
    return 0;
if (memcmp(buf, "\xde\xad\xbe\xef", 4))
    return 0;

printf("Stage 5 clear!\n");
```

1. `saddr.sin_port = htons(atoi(argv['C']));` 监听端口通过环境变量获取。
2. 通过 `socket` 接收到的内容是 `\xde\xad\xbe\xef`。
