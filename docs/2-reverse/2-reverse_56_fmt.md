
# 菜菜对 od，F2 断点的小小理解

> 原文：[`www.pediy.com/kssd/pediy12/129808.html`](https://www.pediy.com/kssd/pediy12/129808.html)

**主题**：`Od` 中 `F2` 断点深入解析
**语言**：`C`，内联 `asm`
**平台**：`win32`
**工具**：`vc6.0` `od` 计算器

哈哈，自我申明，没有什么高深的技术，只是想逼自己做什么都需要动手，高手请过。。（纸上谈来终觉浅,绝知此事要躬行）。。

首先我们知道，在 `od` 中下 `F2`（或 `vc` 中 `F9`）断点通常是在运行前编译器，将断点的位置写入 `int 3`(机器码即 `0xcc`),.当被调试的进程执行 `int3` 指令时导致一个异常时，调试器会捕捉这个异常从而停在断点处，然后将断点处的指令恢复原来的指令。就是这样么？我们接下来具体操作操作，由于实际在 `od` 中下断处，我们观察内存根本看不见有 `cc` 出现（对像我这样的新手来说，其实通过其他方法是可以看到的），让我用小 code 来证明一下：两个问题

1. 是否 `0xcc` 被插入。。
2. 插入后是否一直恢复了原指令？

答案：不是，它还有一个执行完后，继续写回的操作。

### 证 1：code

```c
#include "windows.h"

void Sample()
{
  int sum=0;
  sum+=2;
  MessageBox(NULL,"SampleSelf!","Sample",MB_OK);
}

int APIENTRY WinMain(HINSTANCE hInstance,
                     HINSTANCE hPrevInstance,
                     LPSTR     lpCmdLine,
                     int       nCmdShow)
{
  byte *PointA;
  PointA=(byte *)Sample;    //Sample 处下断
  if (*PointA==0xcc)       //检测
    MessageBoxA(0,"CC exist!","Debugger",0);
  Sample();
  if (*PointA==0xcc)
    MessageBoxA(0,"CC exist!","Debugger",0);
}
```

代码很简单，呵呵。。。我们来看看载入 `od` 的结果

![](img/e17521eec24bb621e54b616339d935ee.jpg)

![](img/a2bd76a738fc9873691fc09d82fc4c65.jpg)

1 证明成立确实是 `int 3` 插入，这个来论坛没人不知道呵呵。。。

第二条证明，可能有很多像我这样的新手会认为 `F2` 断点的原理早已掌握，其实不然。。我可能是个特殊。。

### 证 2 代码：

```c
#include<windows.h>
#include<stdio.h>

DWORD  Calculation(DWORD StartAddr,DWORD dwCodeLength)
{
  _asm
  {
      xor eax,eax
      xor ebx,ebx
      mov esi,StartAddr
      mov ecx,dwCodeLength
Label1:
      mov bl,byte ptr [esi]
      add eax,ebx
      inc esi
      loop Label1
  }
}

DWORD CheckSelf(DWORD StartAddr,DWORD dwCodeLength)
{
  _asm
  {
      xor eax,eax
      xor ebx,ebx
      mov esi,StartAddr
      mov ecx,dwCodeLength
Label1:
      mov bl,byte ptr [esi]
      add eax,ebx
      inc esi
      loop Label1
      sub eax, 3763//3763 是由 dwSumbyte 提前提供
  }
}

int APIENTRY WinMain(HINSTANCE hInstance,
                     HINSTANCE hPrevInstance,
                     LPSTR     lpCmdLine,
                     int       nCmdShow)
{
  DWORD dwStartAddr=0;
  DWORD dwEndAddr=0;
  DWORD dwCodeLength=0;
  DWORD dwChecksum=0;
  DWORD dwSumbyte=0;
  DWORD dwOdd=0;
  char  temp[200]={0};
  _asm
  {
Start:
    xor eax,eax
    xor ebx,ebx
    xor ecx,ecx
    xor edx,edx
    add ebx,ecx
    add edx,ebx
    add ebx,eax
    add eax,ebx
    mov dwStartAddr,offset Start
    mov dwEndAddr,offset End
End:
  }
dwCodeLength=dwEndAddr-dwStartAddr;
dwSumbyte=Calculation(dwStartAddr,dwCodeLength);
dwOdd  = CheckSelf(dwStartAddr,dwCodeLength);
wsprintf(temp,"the Deviation is %d",dwOdd);
MessageBox(NULL,temp,"OK",MB_OK);               //显示差值，od 是否作用过。。。
}
```

代码也很简单，了解代码校验的估计都会。。所以也不多说。。

我测试的断点是 `Start` 和 `End` 标签直接。。。

好我们来看结果:

#### 1. 不下断点：

![](img/9cb86cabb97d160908d543781de6c15a.jpg)

结果正常！

#### 2. 下断点：

![](img/09d08f62ebb4f47034523eea83e51c2d.jpg)

------1, 下在此处：

![](img/3b04b419750656e860fa0af1bdead46f.jpg)

为什么是这个值呢？

答：应该是 `0xcc`，代替了图中的 `0x33`，算算就知道 `0xcc - 0x33 = 153` 正确

------2. 那你来猜猜第二次应该是多少呢？

`0xcc + 0xcc - 0x33 - 0x03` 应该等于 `354` 。。

![](img/c0aa3ae028f378750d590d4e7bc2d395.jpg)

哈哈都对了。。。

确实是机器码 `cc` 起了作用。。

但 `==`，看看我的代码 2，发现了什么异常吗？不是说过它执行完了就回把源代码还原吗，

我的 `Start` 和 `End` 之间的代码可在计算函数之前执行的哦！！为什么它还会存在 `int 3`（`0xcc`），

我想，它在之后又还是将 `int 3` 写入进程 `text` 一次，只要不结束，它就存在，是这样吗，实际确实是这样滴！（想想我们在断 `F2` 时，如果你不清除，断点是始终存在的，恩就是这个道理！）

我来实际操作看看，还是执行 `code1`，`Sample` 处下 `F2` 断点：

操作结果是：依次显示 3 副图

![](img/967d6b8ef47739bc641efed4adbfe137.jpg)
