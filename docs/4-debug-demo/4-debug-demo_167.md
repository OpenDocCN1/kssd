
# 将 CrackMe 改造成注册机

> 原文：[`www.pediy.com/kssd/pediy11/116844.html`](https://www.pediy.com/kssd/pediy11/116844.html)

受[【原创】借尸还魂--把程序自己改成注册机](http://bbs.pediy.com/showthread.php?t=100440&highlight=%E6%94%B9%E9%80%A0)的启发，现在将一个 `CrackMe` 改造成注册机。

程序来自：`光盘\chap02\2.1 OllyDbg 调试器\2.1.4 基本操作\bin\ASCII 版\TraceMe.exe`

`TraceMe.rar`

## 第一部分：爆破

这部分书上有，比较简单。

`od` 载入，`Ctrl+G` 跟随 `GetDlgItemTextA`，来到 `77D6B05E`，然后 `F2` 下断点，运行。

输入用户名和注册码后，单击 `check` 就断下来了，`F2` 取消断点，然后 `Alt+F9` 来到 `004011B6` 处。

一路 `F8` 来到 `004011F5` 处，这里就是关键跳转。

按空格键后输入 `nop`，再 `F9`，就显示"恭喜你"了。

## 第二部分：注册机

这也不难，在关键跳转上方找到 `00401340`，按回车键跟进去，可以看到计算序列号的代码：

```
00401340  /$  55             push    ebp
00401341  |.  8B6C24 0C      mov     ebp, dword ptr [esp+C]
……
0040139B      40             inc     eax
0040139C      5D             pop     ebp
0040139D  \.  C3             retn
```

仔细分析一下还是能写出注册机的：

(`dev-cpp 4.9.9.2` 下测试成功)

```
#include <cstdio>
#include <cstdlib>
using namespace std;
const int b[8]={0x0C,0x0A,0x13,0x09,0x0C,0x0B,0x0A,0x08};
int main(){
    unsigned char a[100];
    printf("用户名：");
    scanf("%s",&a);
    int len=0;
    while(a[len])len++;
    if(len<5){
        printf("用户名不能小于 5 位!\n");
        system("pause");
        return 0;
    }
    unsigned s=0;
    for(int i=3,j=0;i<len;i++,j++){
        if(j>7)j=0;
        s=s+(a[i]*b[j]);
        printf("%.8X\n",s);
    }
    printf("\n%u\n",s);
    system("pause");
    return 0;
}
```

## 第三部分：改造

这才是重点，这部分是我写这个帖子的原因。

基本的思路就是在计算出序列号后，用 `MessageBox` 函数将其输出。

先在程序的后面，找出一段空间来存放自己的代码

![](img/0d99c9adba4e6f2432ad2f66259c4661.jpg)

先看一下计算序列号的函数的结尾：

![](img/d3a7731732eff9242f854b97b2404eed.jpg)

在 `0040139B` 处，按空格键进行汇编，输入`"jmp 00403D00"`

![](img/a490996454da1b60c1cb9193f930dd8f.jpg)

然后来到 `00403D00` 处，开始汇编。

![](img/ae2faba75798e21b61227c6e32797f94.jpg)

现在运行。

![](img/80a2e307e6d27df82f80ecf603a41c8a.jpg)

可以看到注册码了。然后把所有修改保存到 `"TraceMe 修改.exe"` 里。

![](img/65df19ad3ca57c1862f838db824e6440.jpg)

运行一下 `"TraceMe 修改.exe"` 试试。

![](img/513ce758abe7df0ec612094a9ac60efa.jpg)

验证一下。

![](img/1b7322ec99dc49d58f76fc61f3687356.jpg)

---

事实上，【第三部分：改造】中有两个小问题：

1. `00403D00` 处不一定是没用的，程序本身有可能会对这段空间做一些操作。
2. `MessageBox` 函数的 `Title` 参数为 `0`，有可能造成不可预料的后果。
