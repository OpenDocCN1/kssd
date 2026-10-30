
# [分享]pwnable.kr input day6

## 题目

![](upload/attach/202101/760268_4HBCXMG4FBMRJM7.png)

## 解题过程

### 1. 查看文件列表

![](upload/attach/202101/760268_BCRR4R933YZB7MT.png)

`flag` 只对创建者 input2_pwn 和 root 可读，而我们的登录用户是 input2，无读权限。input2_pwn 与 input2 用户对 `input` 文件有读和执行权限，且权限里含有 s，因此 input2 用户在执行该文件时，会被赋予 root 权限。

此题调试过程较复杂，先将 `input` 文件复制到本地，再生成一个本地的 `flag` 便于测试。

`scp -P 2222 input2@pwnable.kr:/home/input2 ./`

`echo "This is a test!" > flag`

### 2. 查看 `input.c` 文件

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>
int main(int argc, char* argv[], char* envp[]){
printf("Welcome to pwnable.kr\n");
printf("Let's see if you know how to give input to program\n");
printf("Just give me correct inputs then you will get the flag :)\n");
// argv
if(argc != 100) return 0;
if(strcmp(argv['A'],"\x00")) return 0;
if(strcmp(argv['B'],"\x20\x0a\x0d")) return 0;
printf("Stage 1 clear!\n");
// stdio
char buf[4];
read(0, buf, 4);
if(memcmp(buf, "\x00\x0a\x00\xff", 4)) return 0;
read(2, buf, 4);
if(memcmp(buf, "\x00\x0a\x02\xff", 4)) return 0;
printf("Stage 2 clear!\n");
// env
if(strcmp("\xca\xfe\xba\xbe", getenv("\xde\xad\xbe\xef"))) return 0;
printf("Stage 3 clear!\n");
// file
FILE* fp = fopen("\x0a", "r");
if(!fp) return 0;
if( fread(buf, 4, 1, fp)!=1 ) return 0;
if( memcmp(buf, "\x00\x00\x00\x00", 4) ) return 0;
fclose(fp);
printf("Stage 4 clear!\n");
// network
int sd, cd;
struct sockaddr_in saddr, caddr;
sd = socket(AF_INET, SOCK_STREAM, 0);
if(sd == -1){
printf("socket error, tell admin\n");
return 0;
}
saddr.sin_family = AF_INET;
saddr.sin_addr.s_addr = INADDR_ANY;
saddr.sin_port = htons( atoi(argv['C']) );
if(bind(sd, (struct sockaddr*)&saddr, sizeof(saddr)) < 0){
printf("bind error, use another port\n");
return 1;
}
listen(sd, 1);
int c = sizeof(struct sockaddr_in);
cd = accept(sd, (struct sockaddr *)&caddr, (socklen_t*)&c);
if(cd < 0){
printf("accept error, tell admin\n");
return 0;
}
if( recv(cd, buf, 4, 0) != 4 ) return 0;
if(memcmp(buf, "\xde\xad\xbe\xef", 4)) return 0;
printf("Stage 5 clear!\n");
// here's your flag
system("/bin/cat flag");
return 0;
}
```

我们的目标是执行 `system("/bin/cat flag")` 需满足 5 个条件。

#### 条件 1 `argv`

##### 基础知识

> `int main(int argc, char *argv[], char *envp[])`

`argc` 代表程序的参数个数，程序名也作为参数之一，所以参数个数最少为 1。

`argv[]` 以 null 结尾的字符串数组。第一个字符串（`argv[0]`）是程序名，后面的每个字符串都是从命令行传递给程序的参数。最后一个字符串（`argv[argc]`）为 null。

`envp[]` 指向系统的环境变量数组的指针，"名称=值"的形式，以 `NULL` 结束。

`int execve(const char *filename, char *const argv[], char *const envp[]);`

##### c 源码

```
// argv
if(argc != 100) return 0;
if(strcmp(argv['A'],"\x00")) return 0;
if(strcmp(argv['B'],"\x20\x0a\x0d")) return 0;
printf("Stage 1 clear!\n");
```

##### 要求

程序在执行时输入 100 个参数，且第 65 个参数应该是 `"\x00"`，第 66 个参数应该是 `"\x20\x0a\x0d"`。（65、66 分别是 `A`、`B` 的 ASCII 码）

而且 `"\x00"` 代表 `NULL`，`"\x20"` 代表空格，`"\0a"` 代表换行，`"\x0d"` 代表回车，这些参数无法在命令行中直接输入，此处采用 `execve()` 来传参。

##### exp

```
#include <stdio.h>
#include <unistd.h>
int main() {
char *argv[100];
argv[0] = "/home/whoami/pwn/input/input";
for (int i = 1; i < 100; i++)
argv[i] = "A";
argv[65] = "\x00";
argv[66] = "\x20\x0a\x0d";
argv[100] = NULL;
char* envp[] = {0,NULL};
execve("/home/whoami/pwn/input/input",argv,envp);
}
```

##### 执行结果

![](upload/attach/202101/760268_QX7WNPREDST5QST.png)

#### 条件 2 `stdio`

##### 基础知识

> `ssize_t read(int fd, void *buf, size_t count);`

`fd` 是文件描述符，在 `fd` 一题中遇到过。`fd == 0` 是标准输入 `stdin`，`fd == 1` 是标准输出 `stdout`，`fd == 2` 是标准错误输出 `stderr`

`int pipe(int fd[2]);`

管道两端可分别用描述字 `fd[0]` 以及 `fd[1]` 来描述，`fd[0]` 端只能用于读，称其为管道读端；`fd[1]` 端则只能用于写，称其为管道写端。

`int dup2(int odlfd, int newfd);`

`dup2()` 用来复制参数 `oldfd` 所指的文件描述词，并将它拷贝至参数 `newfd` 后一块返回。

##### c 源码

```
// stdio
char buf[4];
read(0, buf, 4);
if(memcmp(buf, "\x00\x0a\x00\xff", 4)) return 0;
read(2, buf, 4);
if(memcmp(buf, "\x00\x0a\x02\xff", 4)) return 0;
printf("Stage 2 clear!\n");
```

##### 要求

第一个 `read()` 中的 `fd` 为 0，表示标准输入，`"\x00\x0a\x00\xff"` 无法通过命令行输入，第二个 `read()` 中的 `fd` 为 2，表示标准错误输出，`"\x00\x0a\x02\xff"` 也没有办法从命令行输入。此处考虑借助管道实现 I/O 重定向，最终实现将这些特殊字符输入到对应 `buf` 中。

##### exp

```
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
int main() {
/* stage 1 */
char *argv[100];
argv[0] = "/home/whoami/pwn/input/input";
for (int i = 1; i < 100; i++)
argv[i] = "A";
argv[65] = "\x00";
argv[66] = "\x20\x0a\x0d";
argv[100] = NULL;
char* envp[] = {0,NULL};
/* stage 2 */
int pipe_stdin[2], pipe_stderr[2];
pid_t pid_child;
if(pipe(pipe_stdin) < 0 || pipe(pipe_stderr) < 0) {
perror("Cannot create pipe!");
exit(-1);
}
if((pid_child = fork()) < 0) {
perror("Cannot create child process!");
exit(-1);
}
if(pid_child == 0) {
/* 子进程先等待父进程重定向管道读，然后关闭不使用的管道读
** 继而向两个管道写对应的字符串，父进程此时已经把管道读重定向到 stdin 和 stderr
** 最终由 input 程序接收
*/
sleep(1);
close(pipe_stdin[0]);
close(pipe_stderr[0]);
write(pipe_stdin[1], "\x00\x0a\x00\xff", 4);
write(pipe_stderr[1], "\x00\x0a\x02\xff", 4);
}
else {
//父进程无需管道写操作，首先 close 掉，然后把两个管道读重定向到 stdin 和 stderr 即 0 和 2
close(pipe_stdin[1]);
close(pipe_stderr[1]);
dup2(pipe_stdin[0], 0);
dup2(pipe_stderr[0], 2);
execve("/home/whoami/pwn/input/input", argv, envp);
}
}
```

##### 执行结果

![](upload/attach/202101/760268_TCSUTWWYACRC9PG.png)

#### 条件 3 `env`

##### 基础知识

> `char *getenv(const char *name);`

该函数返回一个以 null 结尾的字符串，该字符串为被请求环境变量的值。如果该环境变量不存在，则返回 `NULL`。

##### c 源码

```
// env
if(strcmp("\xca\xfe\xba\xbe", getenv("\xde\xad\xbe\xef")))
return 0;
printf("Stage 3 clear!\n");
```

##### 要求

使环境变量名 `"\xde\xad\xbe\xef"` 对应的值为 `"\xca\xfe\xba\xbe"` 即可。

利用 `execve()` 传参。

##### exp

将 `char *envp[] = {0, NULL};` 修改为 `char *envp[2] = {"\xde\xad\xbe\xef=\xca\xfe\xba\xbe", NULL};` 即可。

##### 执行结果

![](upload/attach/202101/760268_KXVUBVJBGFY2YGN.png)

#### 条件 4 `file`

##### 基础知识

> `size_t fread(void *buffer, size_t size, size_t count, FILE *stream);`

`buffer` 为接收数据的地址，`size` 为一个单元的大小，`count` 为单元个数，`stream` 为文件流。

`fread()` 函数每次从 `stream` 中最多读取 `count` 个单元，每个单元大小为 `size` 个字节，将读取的数据放到 `buffer`；文件流的位置指针后移 `size * count` 字节。

返回值为实际读取的单元个数。如果小于 `count`，则可能文件结束或读取出错；可以用 `ferror()` 检测是否读取出错，用 `feof()` 函数检测是否到达文件结尾。

`size_t fwrite(void *buffer, size_t size, size_t count, FILE *stream);`

`buffer` 为数据源地址，`size` 为每个单元的字节数，`count` 为单元个数，`stream` 为文件流指针。

`fwrite()` 函数每次向 `stream` 中写入 `count` 个单元，每个单元大小为 `size` 个字节；文件流的位置指针后移 `size * count` 字节。

返回值为成功写入的单元个数。如果小于 `count`，则说明发生了错误，文件流错误标志位将被设置，随后可以通过 `ferror()` 函数判断。

##### c 源码

```
// file
FILE* fp = fopen("\x0a", "r");
if(!fp) return 0;
if( fread(buf, 4, 1, fp)!=1 ) return 0;
if( memcmp(buf, "\x00\x00\x00\x00", 4) ) return 0;
fclose(fp);
printf("Stage 4 clear!\n");
```

##### 要求

读文件 `"\x0a"`，前四个字节是 `"\x00\x00\x00\x00"` 即可。

##### exp

```
/* stage 4 */
FILE *fp = NULL;
char buf[] = "\x00\x00\x00\x00";
fp = fopen("\x0a","wb");
if(!fp) {
perror("Cannot open file.");
exit(-1);
}
fwrite(buf,4,1,fp);
fclose(fp);
fp = NULL;
```

##### 执行结果

![](upload/attach/202101/760268_24EFMARRDAXGF2F.png)

#### 条件 5 `network`

##### 基础知识
