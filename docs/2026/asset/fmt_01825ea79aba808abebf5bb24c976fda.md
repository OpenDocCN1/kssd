
```
00401010       sub esp,0x14                        ; 分配数组空间
00401013       push esi
00401014       push edi
00401015       mov edi,0x5                         ; 对应变量i，这里是自减循环
0040101A       mov dword ptr ss:[esp+0x8],0x1      ; 在栈上创建数组
00401022       mov dword ptr ss:[esp+0xC],0x2
0040102A       mov dword ptr ss:[esp+0x10],0x3
00401032       mov dword ptr ss:[esp+0x14],0x4
0040103A       mov dword ptr ss:[esp+0x18],edi
0040103E       lea esi,dword ptr ss:[esp+0x8]      ; 让esi指向数组，对应变量piArray
00401042       mov eax,dword ptr ds:[esi]          ; 调用printf输出
00401044       push eax
00401045       push ReverseT.00414A30
0040104A       call ReverseT.00401080
0040104F       add esp,0x8
00401052       add esi,0x4                         ; 根据指针类型大小加，而不是加1
00401055       dec edi
00401056       jnz short ReverseT.00401042
00401058       pop edi
00401059       xor eax,eax
0040105B       pop esi
0040105C       add esp,0x14
0040105F       retn
```

从上面可以知道，指针的加法是和类型有关的，所以两指针相加也是没有意义的。

### 引用

有人说，引用是在指针之后发明的另一种访问方式，引用是基于指针实现的，简化了指针操作，可以通过一段代码来证实一下：

```
#include <stdio.h>

int main()
{
    int iVar;
    scanf("%d", &iVar);
    printf("%d", iVar);
    return 0;
}
```

在VC++6.0中打开优化编译选项“Maximize speed”，编译结果如下：

```
00401010       push ecx
00401011       lea eax,dword ptr ss:[esp]          ; iVar的指针
00401015       push eax
00401016       push ReverseT.00414A30
0040101B       call ReverseT.0040F890              ; 调用scanf
00401020       mov ecx,dword ptr ss:[esp+0x8]
00401024       add esp,0x8
00401027       push ecx
00401028       push ReverseT.00414A30
0040102D       call ReverseT.00401080              ; 调用printf
00401032       xor eax,eax
00401034       add esp,0xC
00401037       retn
```

由此可以，不论是引用还是指针，都是使用`lea`指令，所以他们其实是一类东西。

## 常量

常量在程序运行前就存在，直接被编译到可执行文件中，当程序启动之后，它们便会被加载进来。这些数据通常都在常量数据区(`iData`段)中保存，该区域没有可写权限，所以对常量进行修改就会引发程序异常。

此处略去108个字。

---

**声明：** 上面的代码有的是自己写的，有的是书上的，全部经过测试，如果感觉有问题请先动手尝试。其中一些插图来自书上，小部分来自网络。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2019-8-8 00:21 被0xlane编辑，原因：部分错误
