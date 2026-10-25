
# 【读书笔记】C++反汇编与逆向分析技术揭秘2

[TOC]

# 一、准备工作

## 1.1.编译环境

- VS2019
- Release/Debug x86

## 1.2.OllyDbg

常用快捷键

- F2：设置断点
- F3：加载一个可执行程序
- F4：程序执行到光标处
- F5：缩小，还原当前窗口
- F7：单步步入
- F8：单步步过
- F9：运行程序
- Ctrl+F2：重新运行程序到起始处
- Ctrl+F9：执行到函数返回处，用于跳出函数实现
- Alt+F9：执行到用户代码处，用于快速跳出系统函数
- Ctrl+G：快速定位跳转地址

## 1.3.IDA

常用快捷键

- 空格键：反汇编窗口切换文本跟图形
- a：解析成字符串的首地址
- b：十六进制与二进制转换
- c：解释位一条指令
- d：解释为数据，每按一次转换数据长度
- g：快速查找到对应地址
- h：十六进制与十进制转换
- k：将数据解释为栈变量
- m：解释为枚举成员
- n：重新命名
- t：把偏移改为结构体
- u：取消定义函数、代码、数据的定义
- x：查看交叉引用
- y：更改变量的类型
- 分号：添加注释
- shift+F9：添加结构体
- Alt+T：搜索文本
- ins：插入结构体
- Alt+Q：修改数据类型为结构体类型

# 二、基本数据类型的表现形式

## 2.1.无符号整数

以`unsigned int`为例

- 取值范围：0～4294967295（0x00000000～0xFFFFFFFF）
- 小尾方式存放：低数据位存放在内存的低端，高数据位存放在内存的高端
- 不存在正负之分，都是正数

## 2.2.有符号正数

以`int`为例

- 最高位是符号位，0表示正数，1表示负数
- 取值范围：-2147483648~2147483648
- 正数区间（0x00000000～0x7FFFFFFF），负数区间（0x80000000～0xFFFFFFFF）
- 负数在内存中都是以补码形式存放的，可以表达位：对这个数值取反+1

## 2.3.浮点数类型

SSE指令集

- 八个寄存器：`XMM0`-`XMM8`，每个寄存器占16字节（128bit）

![图片描述](upload/attach/202211/939298_BCVV8FXYVCJH2CX.png)

# 三、认识启动函数，找用户入口

## 3.1.找`main`函数入口方法

VS2019 Release版本

- 有3个参数，`main`函数是启动函数中唯一具有3个参数的函数
- 找到入口代码第一次调用`exit`函数处，离`exit`最近的且有3个参数的函数通常就是`main`函数

找`main`函数入口

![图片描述](upload/attach/202211/939298_GZ82HFZFTH74GZ6.png)

![图片描述](upload/attach/202211/939298_YFKXV4VEGXW2VUN.png)

# 四、观察各种表达式的求值过程

## 4.1.加法

release版

```c++
#include <stdio.h>
int main(int argc, char *argv[]) {
    int n1 = argc;
    int n2 = argc; //复写传播：n2等价于引用argc， n2则被删除
    n1 = n1 + 1; //下一句n1重新赋值了，所以这句被删除了
    n1 = 1 + 2; //常量折叠：n1 = 3
    n1 = n1 + n2; //常量传播和复写传播： n1 = 3 + argc
    printf("n1 = %d\n", n1);
    return 0;
}
```

ida

![图片描述](upload/attach/202211/939298_8DAQ7XJPPZ6QHCF.png)

## 4.2.减法

release

```c++
#include <stdio.h>
int main(int argc, char *argv[]) {
    int n1 = argc;
    int n2 = 0;
    scanf_s("%d", &n2);
    n1 = n1 - 100;
    n1 = n1 + 5 - n2; //n1 = n1 -95 - n2
    printf("n1 = %d \r\n", n1);
    return 0;
}
```

ida

![图片描述](upload/attach/202211/939298_HZ4QCH3GN4H3A9P.png)

## 4.3.乘法

release

```c
#include <stdio.h>
int main(int argc, char *argv[]) {
    int n1 = argc;
    int n2 = argc;
    printf("n1 * 15 = %d\n", n1 * 15); //变量乘常量 ( 常量值为非 2 的幂 )
    printf("n1 * 16 = %d\n", n1 * 16); //变量乘常量 ( 常量值为 2 的幂 )
    printf("2 * 2 = %d\n", 2 * 2); //两常量相乘
    printf("n2 * 4 + 5 = %d\n", n2 * 4 + 5); //混合运算
    printf("n1 * n2 = %d\n", n1 * n2); //两变量相乘
    return 0;
}
```

ida

![图片描述](upload/attach/202211/939298_VHK22V7XQ2Z2TGY.png)

## 4.4.除法

常用指令

- `cdq`：把`eax`的最高位填充到`edx`，如果`eax` ≥ 0，`edx` = 0，如果`eax` ＜ 0，`edx` = 0xFFFFFFFF
- `sar`：算术右移
- `shr`：逻辑右移
- `neg`：将操作数取反+1
- `div`：无符号数除法
- `idiv`：有符号除法
- `mul`：无符号数乘法
- `imul`：有符号数乘法

### 4.4.1.除数为无符号2的幂

release

```c
#include <stdio.h>
int main(unsigned argc, char *argv[]) {
    printf("a / 16 = %u", argc / 16);
    return 0;
}
```

ida

![图片描述](upload/attach/202211/939298_Q6RZ7EDEHMNPZFB.png)

### 4.4.2.除数为无符号非2的幂

release

```c++
#include <stdio.h>
int main(int argc, char *argv[]) {
    printf("argc / 3 = %u", (unsigned)argc / 3); //变量除以常量，常量为无符号非2的幂
    return 0;
}
```

ida

![图片描述](upload/attach/202211/939298_XGUEHRFVUAAA7XT.png)

![图片描述](upload/attach/202211/939298_7MEXFHQ4QMVDVKX.png)

### 4.4.3.另一种除数为无符号非2的幂

release

```c
#include <stdio.h>
int main(unsigned argc, char *argv[]) {
    printf("a / 7 = %u", argc / 7);
    return 0;
}
```

ida反汇编

![图片描述](upload/attach/202211/939298_TJSXKMD4D2J8CBF.png)

![图片描述](upload/attach/202211/939298_HWM9SV45W6N6FCM.png)

### 4.4.4.除数为有符号2的幂

release

```c
#include <stdio.h>
int main(int argc, char *argv[]) {
    printf("a / 8 = %d", argc / 8);
    return 0;
}
```

ida

![图片描述](upload/attach/202211/939298_7R34BFYQFGYZC8Z.png)

![图片描述](upload/attach/202211/939298_XYMAH7GK9BHNK76.png)

### 4.4.5.除数为有符号非2的幂

release

```c
#include <stdio.h>
int main(int argc, char *argv[]) {
    printf("a / 9 = %d", argc / 9); ////变量除以常量，常量为非2的幂
    return 0;
}
```

ida

![图片描述](upload/attach/202211/939298_V7A72BZH25SCCCU.png)

![图片描述](upload/attach/202211/939298_2S22HTTS2HS2S96.png)

### 4.4.6.第二种除数为有符号非2的幂

release版

```c++
#include <stdio.h>
int main(int argc, char *argv[]) {
    printf("argc / 7 = %d", argc / 7); //变量除以常量，常量为非2的幂
    return 0;
}
```

ida

![图片描述](upload/attach/202211/939298_PV2GAVVAWRNERCF.png)

![图片描述](upload/attach/202211/939298_9DRPW23J56XBKAQ.png)

### 4.4.7.除数为有符号负2的幂

release版

```c
#include <stdio.h>
int main(int argc, char *argv[]) {
    printf("a / -4 = %d", argc / -4);
    return 0;
}
```

ida反汇编

![图片描述](upload/attach/202211/939298_6DK85A8CBB5G5QE.png)

### 4.4.8.除数为有符号负非2的幂

release

```c
#include <stdio.h>
int main(int argc, char *argv[]) {
    printf("a / -5 = %d", argc / -5);
    return 0;
}
```

ida

![图片描述](upload/attach/202211/939298_5WPMHKS2627JA73.png)

![图片描述](upload/attach/202211/939298_V6H45VVD5DYVZEB.png)

### 4.4.9.另一种除数为有符号负非2的幂

release版

```c++
#include <stdio.h>
int main(int argc, char *argv[]) {
    printf("argc / -7 = %d", argc / -7); //变量除以常量，常量为负非2的幂
    return 0;
}
```

ida

![图片描述](upload/attach/202211/939298_6SUUFS9PYRFTTX9.png)

![图片描述](upload/attach/202211/939298_FH3AXY3FH4DEPGJ.png)

## 4.5.取模

release版

```c
#include <stdio.h>
int main(int argc, char *argv[]) {
    printf("%d", argc % 8); //变量模常量，常量为2的幂
    printf("%d", argc % 9); //变量模常量，常量为非2的幂
    return 0;
}
```

ida反汇编

![图片描述](upload/attach/202211/939298_VQ5FHQFDEFDY6DP.png)

![图片描述](upload/attach/202212/939298_FG248RMCSQTWDFP.png)

## 4.6.条件跳转指令表

![图片描述](upload/attach/202211/939298_DRQ465WDZYYW84Q.png)

## 4.7.条件表达式

**第一种，相差为**1

release

```c
#include <stdio.h>
int main(int argc, char *argv[])
{
    printf("%d\r\n",argc == 5 ? 5:6);
    return 0;
}
```

ida

![图片描述](upload/attach/202211/939298_5K6HPQ44D86UYGN.png)

**第二种,相差大于1**

release

```c++
#include <stdio.h>
int main(int argc, char *argv[])
{
    printf("%d\r\n",argc == 5 ? 4:10);
    return 0;
}
```

ida

![图片描述](upload/attach/202211/939298_52YPPJ39TCWABQK.png)

**第三种变量表达式**

release

```c++
#include <stdio.h>
int main(int argc, char *argv[]) {
    int n1, n2;
    scanf_s("%d %d", &n1, &n2);
    printf("%d\n", argc ? n1 : n2);
    return 0;
}
```

ida

![图片描述](upload/attach/202211/939298_9K2BTJTRMZAMT79.png)

**第四种表达式无优化使用分支**

release

```c++
#include <stdio.h>
int main(int argc, char *argv[]) {
    int n1, n2;
    scanf_s("%d %d", &n1, &n2);
    printf("%d\n", argc ? n1 : n2 + 3);
    return 0;
}
```

ida

![图片描述](upload/attach/202211/939298_XF4XAH9THRKFU77.png)

# 五、流程控制语句的识别

## 5.1.if

release

```c++
#include <stdio.h>
int main(int argc, char *argv[]) {
    if (argc == 0) {
        printf("argc == 0");
    }
    return 0;
}
```

ida

![图片描述](upload/attach/202211/939298_ZAQR26SNXVNTY5Q.png)

总结

![图片描述](upload/attach/202211/939298_H978MEGMCR4GTA9.png)

## 5.2.if else

release

```c++
#include <stdio.h>
int main(int argc, char *argv[]) {
    if (argc > 0) {
        printf("argc > 0");
    }
    else if (argc == 0) {
        printf("argc == 0");
    }
    else {
        printf("argc <= 0");
    }
    return 0;
}
```

ida

![图片描述](upload/attach/202211/939298_JDWAHVMPDMFYVKD.png)

## 5.3.switch

### 5.3.1.分支少于4个

release

```c++
#include <stdio.h>
int main(int argc, char *argv[]) {
    int n = 1;
    scanf_s("%d", &n);
    switch (n) {
    case 1:
        printf("n == 1");
        break;
    case 3:
        printf("n == 3");
        break;
    case 100:
        printf("n == 100");
        break;
    }
    return 0;
}
```

ida

![图片描述](upload/attach/202211/939298_B5H9B3JQARZDP33.png)

![图片描述](upload/attach/202211/939298_6SNX3VUSD2US32P.png)

### 5.3.2.分支大于4个且值连续

会对`case`语句块制作地址表，以减少比较跳转次数

release

```c++
#include <stdio.h>
int main(int argc, char *argv[]) {
    int n = 1;
    scanf_s("%d", &n);
    switch (n) {
    case 1:
        printf("n == 1");
        break;
    case 2:
        printf("n == 2");
        break;
    case 3:
        printf("n == 3");
        break;
    case 5:
        printf("n == 5");
        break;
    case 6:
        printf("n == 6");
        break;
    case 7:
        printf("n == 7");
        break;
    }
    return 0;
}
```

ida

![图片描述](upload/attach/202211/939298_CR58FZGWXVAQS2W.png)

![图片描述](upload/attach/202211/939298_7ZXYYGV2EYQH5X6.png)

![图片描述](upload/attach/202211/939298_AZNMYV3MWRT3GYW.png)

### 5.3.3.分支大于4个，值不连续，且最大`case`值和`case`值的差小于256

有两张表

- `case`语句块地址表：每一项保存一个`case`语句块的首地址，有几个`case`就有几项，`default`也在里面
- `case`语句块索引表：保存地址表的编号，索引表的大小等于最大`case`值和最小`case`值的差

release版
