
# Delphi 反汇编初探

> 原文：[`www.pediy.com/kssd/pediy12/142592.html`](https://www.pediy.com/kssd/pediy12/142592.html)

俺也是菜鸟一名，可能更理解菜鸟的心态吧，偶有小感，写下来与新手分享，相信对新人还是有用的。

简要描述下过程：近来手上有个 `delphi` 程序要 `crack`，没用过 `delphi`。不管怎么样，`OD` 它再说，呵呵。有壳，先脱之，脱了，发现所需要的 `dll` 也有壳，晕倒，从没脱过 `DLL` 的壳呀，看了《加密与解密 2》中的相关章节再去网上下载有关视频，学习了，再脱之，又发现脱了壳的 `DLL` 导入表不见了，又晕，再学习修复之。一套下来我己经有些疲惫。再打开 `delphi` 主程序开始研究，发现完全不是自己想像的，脱壳后就一马平川了，里面花指令一个接一个，再加上对 `delphi` 的不熟悉，里面的很多东西都看不出思路来。好的，再来，学下 `delphi`，把 `delphi` 反汇编的思路理清楚再 `crack` 它，磨刀不误砍柴工。

下面走入正题：下载 `delphi7`，安装后，写了一个小程序，一个对话框，两个按钮，按其中一个按钮会使另外一个按钮灰掉。`delphi` 中使一个按钮灰掉只需要一条语句：

（源码）

```delphi
procedure TForm1.Button1Click(Sender: TObject);
begin
     Button2.Enabled:=false;
end;
```

（^>^以我对 `windows` 程序的理解，要实现使一个按钮变灰，需要调用 `enableWindow()` 这个 `API` 才可以）不管哪些个，反汇编这个程序看看再说，`OD` 打开，跳到这个按钮事件的函数地址(^-^想知道这个地址可以有多种方法，1.用 `DEDE` 程序可以查看，2.`pexplorer` 反汇编后也可以查到，3.`ultraedit` 查找字符串 `Button1Click` 也可以得到这个地址.)

```asm
0044D964  /$  33D2          xor     edx, edx                    //清空了，有用么？呵
0044D966  |?  8B80 FC020000 mov     eax, dword ptr [eax+2FC]   //看不明白，不会是传参数吧
0044D96C  |?  8B08          mov     ecx, dword ptr [eax]       //同上
0044D96E  |?  FF51 64       call    dword ptr [ecx+64]        //还需要调用函数，看来就这么赋值，delphi 还是做了些工作的，进去看看
0044D971  |?  C3            retn        //*****请注意上一行的调用方式,是使用的 ecx+64,
                //^-^这一点对我们理清 delphi 编译器的思路特征很重要，后面会做对比
```

以下是 `F7` 后的结果

```asm
0042CDA8   .  3A50 58       cmp     dl, byte ptr [eax+58]     //比较了一个 dl，哦，刚才哪个清空的 EDX，在这儿用上了，赋值语句的值在这儿。
0042CDAB   .  74 11         je      short 0042CDBE    //通过跟踪多次下断点，这个跳转只会发生一次，也就是已经变灰了，就不用再次执行
0042CDAD   .  8850 58       mov     byte ptr [eax+58], dl     //以下代码对我们已经基本没有,如果要破解的话，上面这个跳转就是关键跳转，但是确不能
0042CDB0   .  6A 00         push    0         //在这里爆，应该在 edx 清空的地方爆，不能在这里爆的原因，也是这篇文章的重点。呵
0042CDB2   .  33C9          xor     ecx, ecx
0042CDB4   .  BA 0CB00000   mov     edx, 0B00C
0042CDB9   .  E8 36120000   call    0042DFF4
0042CDBE   >  C3            retn
```

通过上面的分析，我们已经知道了这些信息：虽然只是一条赋值语句，但是 `delphi` 做了很多工作。

而且对我们破解来说，深一层次的调用，我们是不必关心的。因为它是 `delphi` 的底层调用。这可能也是引起一些爆破软件不好用的原因。

现在是说明本篇文章重点的时候了，就是 `delphi` 的反汇编，哪些是 `delphi` 内部的底层调用，哪些是用户代码调用。

分清这一点可以让我们在反汇编跟踪的时候，不把精力浪费在读底层(这个底层不一定是系统层，可能是 `delphi` 层)代码里。

以上的是 `delphi` 底层调用的经典用例。现在在拿一个用户代码调用的反汇编示例，相信就很清楚了。

以下是增加用户函数调用的 `delphi` 源码

```delphi
function CalculateInterest(Principal, InterestRate: Double):Double;
begin
  CalculateInterest := Principal * InterestRate;
end;

procedure TForm1.Button1Click(Sender: TObject);
begin
     Button2.Enabled:=false;
       CalculateInterest(3,4);
       button2click(Sender);
end;

procedure TForm1.Button2Click(Sender: TObject);
begin
      MessageBox(0,'cccc','',0);
end;
```

从源码可以看出来了，一共三个函数，`CalculateInterest`（自己写的），`Button1Click`，`Button2Click`。

会从 `Button1Click` 调用自己写的函数 `CalculateInterest`，还有生成的 `Button2Click`（也是用户代码）

我们来看看反汇编后的样子。

老样子，跳到 `Button1Click` 的函数事件地址。

```asm
0044D986   .  33D2          xor     edx, edx
0044D988   .  8B83 FC020000 mov     eax, dword ptr [ebx+2FC]
0044D98E   .  8B08          mov     ecx, dword ptr [eax]
0044D990   .  FF51 64       call    dword ptr [ecx+64]            ;这四行都很熟悉了，注意这里是通过 ECX 进行 call 的
0044D993   .  68 00000840   push    40080000
0044D998   .  6A 00         push    0
0044D99A   .  68 00001040   push    40100000
0044D99F   .  6A 00         push    0
0044D9A1   .  E8 BEFFFFFF   call    0044D964  ;这里是我们调用的自己写的函数 CalculateInterest,注意是直接调用的，明明我们要传的是 3，4，怎么成这个样子了，跟进去会发现使用了 ST0 寄存器
0044D9A6   .  DDD8          fstp    st
0044D9A8   .  8BD6          mov     edx, esi
0044D9AA   .  8BC3          mov     eax, ebx
0044D9AC   .  E8 03000000   call    0044D9B4  ;这里是调用的 button2click(Sender),注意这里也是直接调用的。
0044D9B1   .  5E            pop     esi
0044D9B2   .  5B            pop     ebx
0044D9B3   .  C3            retn
```

结论：通过以上的对比，我们可以得出一个结论，`delphi` 的反汇编，如果调用的是用户代码，一般使用直接调用。如果调用的是底层(`delphi` 库)代码，会通过 `ECX` 来做为调用基础。这应该是为了实现面向对象而设计的。引申猜想，如果用户用 `delphi` 设计一个类，在按钮事件的处理里，再调用这个类的对象执行动作，网上有一个说法是 `delphi` 的 `this` 指针是通过左边数第一个参数传的，没有验证这一点。（`VC` 的实现是把 `this` 指针放到 `ECX` 中）

此文仅是初探，相信对新手会有帮助。我会上传两个 `delphi` 编写的程序.

^-^！

上传的附件

| 附件 | 文件名 |
| :--- | :--- |
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | `Project2.rar` |
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | `Project1.rar` |
