
# 分享一下 OD 反汇编引擎。

> 原文：[`www.pediy.com/kssd/pediy10/94573.html`](https://www.pediy.com/kssd/pediy10/94573.html)

这次，分享的引擎我已经封装为类，调用起来会更方便点，而且此次修复了几处问题。

## 修复内容

1: 修复了像如下几处错误：

```c
void CMgDisasmBase::DecodeIM(int constsize,int sxt,int type)
{
  int i;
  signed long data;
  ulong l;
  char name[TEXTLEN]={0},comment[TEXTLEN]={0};
  //增加了初始化，由于作者是 C 程序员，而且还有变量不初始化的习惯，所以像这样的溢出比较多点
  //发现的已经修复。
}
```

2: 修改了一处循环，使其可以正确识别像如下的指令：

```
//第一次修复是逐一添加判别，经仔细分析代码，发现是作者疏忽判断，几次修改后，
//这次的是我比较认同的方法，即贴近原代码，又能解决 BUG，基本上代码看起来变化
//了那么一点点，但是这一点点让我花了好几个小时，几次修改发现都不如愿，总觉得
//增加的代码太多^^,![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")
```

`0046AB8D  |.  64:A1 00000000        mov     eax, dword ptr fs:[0]`

3: 修复了 `REP` 指令的识别，按 `OD` 来确定。

```
//将 REP 和 MOV 分开，不太确定是否妥当.
```

`00417E64  |.  F3:AB                  rep     stos dword ptr es:[edi]  //这里为???`
`00417E66  |.  66:AB                  stos    word ptr es:[edi]`

4: 估计在符点指令上还有识别问题，没时间测试了，给大家放出来，喜欢的，估计能用到的就拿去用好了，放出来是为了方便需要的人.

如一本版不否，请斑竹移一下贴~~！谢谢~~!

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || [`MgAsmCom.rar`](http://bbs.pediy.com/showthread.php?t=94573)  |

## 跟风分享

跟风，我也分享一个 `OD` 反汇编引擎。

跟楼主的没太大区别，`vs2008` 的工程，`c`。那几个 `BUG` 也改掉了很多指令判断错误的 `BUG` 原因在于原作者把很多只需要 1 字节长度的变量定义为 `int`，导致用 `vs2008` 编译后，在指令判断上出现了错误（比如对 `arg` 变量赋值 `0x82`，由于 `arg` 变量定义成了 `int`，导致赋值后变成了 `0xffffff82`……把定义改为 `byte` 就好了。）

也许有人想要这个版本的吧![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || [`OD_DASM.rar`](http://bbs.pediy.com/showthread.php?t=94573)  |

呃，忘了说用法了。

### 反汇编函数

```c
ulong Disasm(char *src,ulong srcsize,ulong srcip, t_disasm *disasm,int disasmmode)
```

返回值是指令长度

`src` 指向需要反汇编的机器码指针

`srcsize` 机器码的总长度

`srcip` 机器码在被调试进程的内存中的内存地址

`disasm` 指向 `t_disasm` 结构的指针。该结构体用来存放反汇编数据。

`disasmmode` 反汇编模式。

### 汇编功能

```c
int Assemble(char *cmd, ulong ip, t_asmmodel *model, int attempt, int constsize, char *errtext)
```

返回值是指令长度

`cmd` 指向汇编指令字符串的指针

`ip` 该句汇编指令在远程进程内存中的地址

`model` 指向 `t_asmmodel` 结构体的指针，用于存放汇编数据

`attempt` 这个……我想不起来了=。= 一般是 `NULL`

`constsize` 立即数的长度（1、2、4 字节）该参数会影响到汇编结果的长度。如果对汇编结果长度有要求，则需要设定对应的值。否则 `NULL` 即可。

`errtext` 汇编时的出错信息。

作者用的是 `borland C`

与 `VC` 有差异是正常的
