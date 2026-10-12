
# 超级光盘管理员 1

> 原文：[`www.pediy.com/kssd/pediy03/forum809.htm`](https://www.pediy.com/kssd/pediy03/forum809.htm)

## 超级光盘管理员 1.注册码的获取方法

下载地址：`URL http://go.163.com/~cdmanager/cdm.zip`

本软件可以使用通用的万能中断 `bpx hmemcpy` 来中断并得到注册码。

使用中要注意首先判断按 `F12` 的次数。下面开始。

1. 首先载入 `vcdrom.exe`，然后选择注册。随便输入注册码和其他信息了，我输入的是用户名：`CrackerABC[BCG]`邮件地址随便，注册码：`31415926`。
2. 然后启动 `TRW2000`（防止有些软件检测 `TRW2000`）点确定，然后用热键 `Ctrl+N` 呼叫出 `trw2000`，下断点 `bpx hmemcpy`。
3. `F5` 返回程序空间后点注册，被 `TRW2000` 拦截到。
4. 在 `TRW2000` 窗口中下 `BD *`终止断点。开始按 `F12`，当按下第 15 次时，出现错误提示。
5. `BC *`清除断点然后 `X` 返回程序空间，重复 1，2，3，4 步，在 4 中，按 `F12` 的次数是 `15-1=14` 次（这就是一般程序按 `F12` 的次数的规则，出错前的为最后一次）这里呢就当然是 14 次了。
6. 下面就开始按 `F10` 了，呵呵，有时候是几次，有时候是几十或者几百次啊，当然有时候要跟进一些 `CALL` 关键子程序中去的。这里呢只按 `F10` 几次就可以了。
7. 按 `F10` 一直使光标到下面的地址：

   `0167:0049990F POP EAX ;------> 下 D EDX 得到注册码：9910`
8. 呵呵，就这么简单。整理注册，一切都变的清净了。俺喜欢！！！

## 太新了!!CCG 冷酷到底超级光盘管理员 1.5 注册机(CCG 发布)

用户名 `CCG` 注册码 `310`

### 超级光盘管理员 1.5 注册

```c
#include <stdio.h>

/*超级光盘管理员 1.5 注册机

下载 http://go.163.com/~cdmanager/cdm.zip

*/

int main(void)

{

const long a=0xa8;

int counter=0;

unsigned long zcm=0,b=0;

unsigned char yourname[63];

clrscr();

printf("CD manager1.5\n");

printf("==================\n");

printf("Made by [CCG]-2001\n");

printf("China Cracking Group\n");

printf("please enter your name:");

gets(yourname);

for(counter=0;counter<=strlen(yourname);counter++)

{

  b=b+yourname[counter]*(counter+1);

    }

zcm=b^a;

printf("your register number is:%ld\n",zcm);

printf("bye");

return(0);

}
```

4dd466d0e1e0cc123afbe504da2462ca
