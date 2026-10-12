
# 第一章:1.8、乘法的识别与优化原理

> 原文：[`www.pediy.com/kssd/pediy11/118062.html`](https://www.pediy.com/kssd/pediy11/118062.html)

到现在还依稀记得大学里讲位运算的情形，记得在讲位移的的时候老师就讲过“左右移位就相当于在做乘除计算，左移几位就相当于是 2 的几次方...”

### 1.8.1、乘法优化之位移

那么如果让我们来做乘法的优化，我们会怎么做呢？很显然位移是必须要被利用的，但是除此之外微软的编译器还利用了 `lea` 指令，但是乘法的优化是非常多变的，本小节的目的是让各位读者再看见某一块指令时知道“哦！这是乘法...”就可以了。

我们先看看简单的位移优化，经过笔者的总结，当乘数为 2 的次方，且大于 8 时编译器才会使用此优化，让我们看看优化前与优化后的效果：

源码：

```
int nNum = 16;
printf("%p",nNum*argc);
```

Debug：

```
mov [ebp+nNum], 10h
mov eax, [ebp+nNum]
imul eax, [ebp+argc] ; 乘法运算
mov esi, esp
push eax
push offset Format ; "%p"
call ds:__imp__printf
add esp, 8
```

Release:

```
mov eax, [esp+argc]
shl eax, 4 ; 左移 4 位，变形的乘法运算(2⁴=16)
push eax
push offset Format ; "%p"
call ds:__imp__printf
add esp, 8
```

按照我们平时对编译器的理解，如果我我们乘的是 17 的话，那么编译器肯定会将其分解为一个左位移再加一个加法，事实真的是如此吗？我们直接看看乘以 17 后的 Release 版的汇编代码：

```
mov eax, [esp+argc]
mov ecx, eax
shl ecx, 4 ; 变形乘法
add ecx, eax ; 果然如此！
push ecx
push offset Format ; "%p"
call ds:__imp__printf
add esp, 8
```

嗯，如果比 2 的次方多 1 会采用加法，那么比 2 的次方少 1 是否会左位移后在加一个减法操作呢？我们共同看看这个 Release 版汇编代码：

```
shl ecx, 4
sub ecx, eax
```

看来我们想的没错，除此之外，我们就要思考一下 `lea` 指令在乘法中的优化及应用了。

### 1.8.2、乘法优化之 lea

众所周知，`lea` 是 Intel 工程师比较得意的一条指令，它的作用是传递操作数地址，并具有指令周期短、与逻辑指令流水线无关等特点，下面我们就一起欣赏一下微软的编译器是怎样使用它来优化乘法的。

为了节约版面，这里我直接给出一个较全面的例子，可以帮助各位读者快速了解乘法 `lea` 的优化方案，先看源代码：

```
int _tmain(int argc, _TCHAR* argv[])
{
  int a = 1, b, c, d, e, f, g;
  b = argc+a*4+6;
  c = argc+a*3+6;
  d = argc*2;
  e = argc*3;
  f = argc*4;
  g = argc*11;
  printf("%d %d %d %d %d %d",b,c,d,e,f,g);
  return 0;
}
```

我们先看 DeBug 版：

```
.text:00412FF0     push ebp
.text:00412FF1     mov ebp, esp
.text:00412FF3     sub esp, 114h
......
.text:0041300E     mov [ebp+a], 1 ; 给局部变量 a 赋值
.text:00413015     mov eax, [ebp+a]
.text:00413018     mov ecx, [ebp+argc]
.text:0041301B     lea edx, [ecx+eax*4+6] ; edx = argc+a*4+6
.text:0041301F     mov [ebp+b], edx
.text:00413022     mov eax, [ebp+a]
.text:00413025     imul eax, 3 ; 先做了 a*3
.text:00413028     mov ecx, [ebp+argc]
.text:0041302B     lea edx, [ecx+eax+6] ; edx = argc+eax+6 (eax=a*3)
.text:0041302F     mov [ebp+c], edx
.text:00413032     mov eax, [ebp+argc]
.text:00413035     shl eax, 1 ; eax = eax*2 (用到了位移优化)
.text:00413037     mov [ebp+d], eax
.text:0041303A     mov eax, [ebp+argc]
.text:0041303D     imul eax, 3 ; eax = eax*3 (直接使用了乘法指令)
.text:00413040     mov [ebp+e], eax
.text:00413043     mov eax, [ebp+argc]
.text:00413046     shl eax, 2 ; eax = eax*4 (用到了位移优化)
.text:00413049     mov [ebp+f], eax
.text:0041304C     mov eax, [ebp+argc]
.text:0041304F     imul eax, 0Bh ; eax = eax*11 (直接使用了乘法指令)
.text:00413052     mov [ebp+g], eax
.text:00413055     mov esi, esp
.text:00413057     mov eax, [ebp+g]
.text:0041305A     push eax
.text:0041305B     mov ecx, [ebp+f]
.text:0041305E     push ecx
.text:0041305F     mov edx, [ebp+e]
.text:00413062     push edx
.text:00413063     mov eax, [ebp+d]
.text:00413066     push eax
.text:00413067     mov ecx, [ebp+c]
.text:0041306A     push ecx
.text:0041306B     mov edx, [ebp+b]
.text:0041306E     push edx
.text:0041306F     push offset Format ; "%d %"
.text:00413074     call ds:__imp__printf
.text:0041307A     add esp, 1Ch
......
.text:00413084     xor eax, eax
.text:00413086     pop edi
.text:00413087     pop esi
.text:00413088     pop ebx
.text:00413089     add esp, 114h
......
.text:00413096     mov esp, ebp
.text:00413098     pop ebp
.text:00413099     retn
```

通过上面的例子我们可以看到即便是 DeBug 版，编译器仍然应用了一些优化方案，那么 Release 版编译器究竟会将上面的代码变成什么样子的呢，请过目：

```
.text:00401000     mov eax, [esp+argc]
.text:00401004     mov ecx, eax
.text:00401006     imul ecx, 0Bh ; ecx = ecx*11 (直接使用了乘法指令)【标注 1】
.text:00401009     push ecx
.text:0040100A     lea edx, ds:0[eax*4] ; edx = argc*4
.text:00401011     push edx
.text:00401012     lea ecx, [eax+eax*2] ; ecx = argc+argc*2 (这是一个 lea 优化，原代码为 e=argc*3)
.text:00401015     push ecx
.text:00401016     lea edx, [eax+eax] ; edx = argc+argc (这是一个 lea 优化，原代码为 d=argc*2)
.text:00401019     push edx
.text:0040101A     lea ecx, [eax+9] ; ecx = argc+9 (这是一个 lea 优化，原代码为 c=argc+a*3+6，且 a*3+6 被直接预先计算成了 9)
.text:0040101D     push ecx
.text:0040101E     add eax, 0Ah ; argc = argc + 10 (这是一个很特别的优化，源代码为 b=argc+a*4+6)【标注 2】
.text:00401021     push eax
.text:00401022     push offset Format ; "%d %d %d %d %d %d"
.text:00401027     call ds:__imp__printf
.text:0040102D     add esp, 1Ch
.text:00401030     xor eax, eax
.text:00401032     retn
```

我想看到这里之后部分读者肯定已经被编译器的强大所折服了吧？

我们在本小节前面提到过“`lea` 具有指令周期短、与逻辑指令流水线无关等特点”，但是为什么编译器在优化时会交替使用 `ecx` 与 `edx` 两个寄存器呢？难道一个寄存器不能解决问题吗？答案是肯定的，原理很简单，我们的 `push` 指令可是与逻辑指令流水线有关的。

另外在“标注 1”的地方我们发现编译器在这里直接使用了乘法指令而并没有将其优化成例如“`lea ecx, [eax+eax*5]`”，这是因为 `lea` 后面的比例因子必须为 2 的倍数，因此我们想象的这条指令是无法被执行的。但是也有个别版本的编译器会将其拆分为两条 `lea` 指令，但这种情况并不常见，因此无须深究。

而“标注 2”所示的这个优化恐怕会难倒一大批人，为什么同样的乘法带加法运算，一个用的是 `lea`，而另一个却是用的 `add` 呢？其实这个问题可简可繁，往简单了说，上面那条指令之所以没有直接用 `add` 是因为会改变寄存器 `eax` 的值，对后面的计算造成不便。而往难了说，那我们就要讨论为什么 `add` 要比 `lea` 的优先级高，我个人认为这还是出于指令周期上的考虑，虽然 `lea` 在逻辑处理器中只占用 1 个指令周期，并且可以再 mmx 协处理器中成对执行，但是其相对负荷产生还是要远高于像 `add` 这种“原生态”指令的。

**后记：**

最近不但时间紧迫、状态不佳，而且还有非常多的事情要忙，因此这个教程一而再、再而三的向后拖延了三个多星期，在这里笔者真诚的向一直关注此教程的读者表示歉意，虽然在接下来的日子里笔者仍然像往常一样繁忙，但是会尽力加快这个系列教程的更新速度。

**[【返回到目录】：http://bbs.pediy.com/showthread.php?t=113689](http://bbs.pediy.com/showthread.php?t=113689)**
