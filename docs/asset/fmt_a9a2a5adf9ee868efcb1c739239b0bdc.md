
# MIX 教程

> 原文：[`www.pediy.com/kssd/pediy12/144973.html`](https://www.pediy.com/kssd/pediy12/144973.html)

## 1. 简介

`MIX` 是 `The Art of Computer Programming` 书中所用的汇编语言。

- **寄存器**：有 9 个寄存器，`A`、`X`、`J`、`I1`、`I2`、`I3`、`I4`、`I5`、`I6`。`I1`~`I6` 为变址寄存器。参见后面的图 3。
- **字结构**：`MIX` 中一个字由 5 个字节和一位符号位组成。
- **字符集**：`MIX` 中的字符包括 `A-Z` 的大写字母，`0-9` 的数字和部分特殊符号。

**指令格式**（这里是说机器指令）：

| 0 | 1 | 2 | 3 | 4 | 5 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 地址 (3 bytes) | index | mod | 操作码 | | |

这是定长指令格式，所有指令都是 6 字节，其中第 1 个字节为符号位，`0`、`1`、`2` 号字节共同组成地址。

我觉得不用太关心机器指令，不过书上说了很多。

在实例 1 中具体说明。

## 2. 实例讲解

### 实例 1

代码在第一卷 `1.3.2` 节 `140` 页，算法在 `1.2.10` 节 `93` 页。

代码：

```assembly
X        EQU    1000
         ORIG    3000

START    STJ    EXIT
INIT     ENT3   0,1
         JMP    CHANGEM
LOOP     CMPA   X,3
         JGE    *+3
CHANGEM  ENT2   0,3
         LDA    X,3
         DEC3   1
         J3P    LOOP
EXIT     JMP    *
         END    START
```

第一个例子，我尽量说详细。这是一个求最大值的例子，求数组 `x[1]...x[n]` 的最大值 `m`，储存到寄存器 `A`。

**初始化部分**：
- 寄存器 `I1` 为数组长度 `n`。
- 寄存器 `I3` 用于存 `k`。
- 寄存器 `I2` 存 `j`。

`n` 是提前给了初值的，`k` 和 `j` 是运行的时候再初始化的。

- `ORIG    3000`
  表示程序是从 `3000` 的地址开始运行的。

- `X        EQU    1000`
  表示数据是从 `1000` 号单元开始存放的，这里省略了数据的初始化。
  即提前在 `1000`、`1001`、`1002`...`1000+N` 号单元存放了一组数据，这个程序是要找出其中的最大值。

- `STJ  EXIT`
  `STJ` 就是 `store rJ`，即储存寄存器 `J` 到 `EXIT` 所在的地址。`J` 即 `jump`，是一个跳转用的寄存器。
  `EXIT` 是个标签，怎么能存到标签呢？实际是存到那个机器指令的地址字段，如图 1：

![](img/e6ec8715555ea786e3da8ac6f68a6da9.jpg)

这张图是一个模拟器的，到后面会说。

执行到 `JMP *` 的时候就会跳转到寄存器 `J` 的地址。这是子程序的开头所以要那么做。

在 `intel` 汇编里面一般是利用栈实现的：
- 在子程序开头：
  ```assembly
  push ebp
  mov ebp, esp
  ```
- 然后在子程序结尾进行相反的操作。

`MIX` 是用寄存器 `J` 实现的。

- `ENT3    0,1`
  `ENT3` 就是 `enter3`，就是存入寄存器 `3`。把什么存进去呢？
  - `1` 是寄存器 `I1`。
  - `0` 是机器码的 `mod`。
  所以 `ENT3    0,1` 是把 `I1` 赋值给 `I3`。
  这个 `0` 其实没多大用，但如果写 `ENT3    1` 意思是 `I3` 赋值为 `1`，所以必须要那个 `mod`。

我们刚才说了初始化寄存器 `I1` 为数组长度 `n`，`I3` 寄存器为变量 `k`。
那么 `ENT3    0,1` 就是 `k=n`。

- `JMP    CHANGEM`
  就是一个无条件跳转，大家应该都懂的。

- `ENT2    0,3`
  就是把寄存器 `I3` 赋值给寄存器 `I2`。`I2` 为变量 `j`。
  那么 `ENT2    0,3` 就相当与 `j=k`。

- `LDA    X,3`
  显然是 `load to A`，把什么存到寄存器 `A` 呢？
  `3` 是寄存器 `I3`，前面说过，`I1`~`I6` 是变址寄存器。
  所以是 `A=X[I3]`，把 `I3` 当作数组下标，`I3` 就是 `k`，`A` 为最大值 `m`。
  所以 `LDA    X,3` 就是 `m=X[k]`。

- `DEC3    1`
  这句就是 `I3` 寄存器的值减 1。`DEC` 不难理解，这里单独的数字就不是寄存器 `I1` 了。

- `J3P    LOOP`
  `jump if I3 positive`，如果 `I3` 为正数，跳到 `LOOP`。

- `CMPA    X,3`
  `CMPA` 是与寄存器 `A` 比较，`X,3` 为 `X[I3]`，即比较 `m` 与 `X[k]`。

- `JGE    *+3`
  `*` 是当前指令的地址，跳转到 `3` 条指令后。

### 实例 2

以第三卷的 `5.2.1` 节直接插入排序。大家应该都会 `80x86` 汇编，我就仿照 `MIX` 的代码用 `x86` 汇编写一个做对照。

先看 `C` 语言的代码复习一下插入排序的原理：

代码：

```c
#include <stdio.h>
#define N 6

int main()
{
  int a[N]={3,2,1,6,5,4};
  int i,j,temp;
  for(j=1;j<N;j++)
  {
    temp=a[j];
    i=j-1;
    while(temp<a[i]&&i>=0)
    {
      a[i+1]=a[i];
      i--;
    }
    a[i+1]=temp;
  }
  for(i=0;i<N;i++)
    printf("%d\t",a[i]);
  return 0;
}
```

汇编是这样的：

代码：

```assembly
.386
.model flat, stdcall
option casemap :none
include    windows.inc
include    kernel32.inc
includelib  kernel32.lib
include    user32.inc
includelib  user32.lib

.data
arr    db  3,2,1,6,5,4  ;数组
count  equ  $-arr     ;数组长度
szBuffer  db  100  dup(0)
szFormat  db  "排序后数组为：%d,%d,%d,%d,%d,%d",0
szTitle    db  "直接插入排序",0
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
.code
start:
  mov si,1    ;j=1
S2:
  mov al,arr[si]  ;temp=a[j]
  mov di,si    ;i=j-1
  dec di
S3:
  cmp al,arr[di]  ;cmp temp,a[i]
  jge S5
  mov ah,arr[di]  ;a[i+1]=a[i]
  mov arr[di+1],ah
  dec di    ;i--
  cmp di,0
  jge S3
S5:
  inc di    ;这两行相当于 mov arr[di+1],al,但那样有 bug
  mov arr[di],al
  inc si
  cmp si,count
  jl S2
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;输出
  xor eax,eax
  mov si,count-1
S6:
  mov al, arr[si] ;逆序压栈
  push eax
  dec si
  cmp si,0
  jge S6
  invoke  wsprintf,addr szBuffer,addr szFormat
  invoke MessageBox,NULL,addr szBuffer,addr szTitle,MB_OK
  invoke  ExitProcess,NULL
  end  start
```

我用 `masm32` 编译运行成功。

完整的 `MIX` 的代码我就不给了，在书上 `65-66` 页。基本上是一条汇编对应一条 `MIX` 指令，大家可以自己对照着看。

## 3. 模拟器

代码总是要能亲自运行一下才好。

原书作者认为 `intel` 的汇编为了保证兼容性而太过复杂，效率不高，所以自己发明了 `MIX` 汇编。

可是 `mix` 的机器是不存在的，所以只能用模拟器来运行。

模拟器可以在 `http://www.gnu.org/software/mdk/mdk.html` 下载，只能在 `linux` 下安装。

安装过程很简单，就是打开终端依次输入以下三个命令：

```bash
./configure
make
sudo make install
```

然后弄 `emacs` 的语法加亮，这一步跳过，主要是为了好看。

输入 `emacs ~/.emacs`

打开 `emacs` 的配置文件，然后输入以下内容：

```lisp
(setq load-path (cons "/usr/local/share/mdk" load-path))
(autoload 'mixal-mode "mixal-mode" t)
(add-to-list 'auto-mode-alist '("\\.mixal\\'" . mixal-mode))
(autoload 'mixvm "mixvm" "mixvm/gud interaction" t)
```

如图 2，不要忘了括号，按 `Ctrl+X`、`Ctrl+S` 保存

![](img/4d8b5949d67a90e316942f6320accac8.jpg)

下面来看一个简单的例子，所谓的 `helloworld`。

代码：

```assembly
*标签  指令(instuction)  操作符(operand)    注释
*
TERM  EQU  19           控制台(terminal)
  ORIG  2000          基地址
START   OUT  MSG(TERM)    在 TERM(即控制台)输出 MSG
  HLT
MSG   ALF  "HEllO"
  ALF  ",PEDI"
  ALF  "Y.   "
  END  START
```

保存为 `hello.mixal`

编译：`mixasm hello`

编译后为 `hello.mix`

执行：`[l@localhost mix-1.2.6]$ mixvm -r hello`

输出：

```text
Program loaded. Start address: 2000
Running ...
HELLO,PEDIY.
... done
```

调试：`mixvm hello`

或者用图形化界面调试更直观 `gmixvm`

如图 3

![](img/21e27a5bbb6dbd501171ead2d9e27bb8.jpg)

点 `File->Load` 加载 `hello.mix`

![](img/b1a76c11046ccc8550a3931b05ad88e5.jpg)

最左边一列是地址，第二列是机器指令。

对 `hello.mixal` 再解释一下：

每一行从左到右依次为 标签、指令(instuction)、操作符(operand)、注释。
用 `TAB` 对齐，比如没有标签第一列就必须留空，因为不对齐就编译不过，这跟其他语言是不一样的，为了让大家看清对齐的格式，我再放张图。

![](img/2779e6c6ac3f0464f4e7ad86615c6133.jpg)

`ORIG 2000` 是定义基地址，类似于 `windows pe` 程序里常见的 `0x00400000`，`2000` 是随便写的，也可以指定其他值。

`HLT` 就是停止。

前面说过，`MIX` 只有大写字母和部分符号，如果在程序里写小写的 `hello`，结果输出还会是大写的，如果用 `！` 等一些不支持符号就不能编译。

用 `ALF` 定义字符串。

因为 `MIX` 的机器指令是定长的，所以不能直接写 `ALF   “HELLO，PEDIY.”`。
指令除了符号位只有 5 个字节，那么只能存 5 个字符，于是有了以上古怪的写法。
