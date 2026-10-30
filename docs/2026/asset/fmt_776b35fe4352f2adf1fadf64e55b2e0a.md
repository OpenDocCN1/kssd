
```
eax=06eafea4 ebx=01581fb0 ecx=0012e1b8 edx=0012e8a8 esi=75c13e59 edi=00000000
eip=00465730 esp=0012e19c ebp=0012eee4 iopl=0 nv up ei pl nz na po nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000202
LabelPrint+0x65730:
00465730 6aff push 0FFFFFFFFh
0:000> dc eax
06eafea4 0061006e 0065006d 00320000 00310030 n.a.m.e...2.0.1.
06eafeb4 00000035 0000004e 00000000 72b6d559 5...N.......Y..r
06eafec4 80000000 0061006b 00650062 0050006c ....k.a.b.e.l.P.
06eafed4 00690072 0074006e 00460020 006c0069 r.i.n.t. .F.i.l.
06eafee4 00000065 72b6d55c 88000000 00000014 e...\..r........
06eafef4 00790053 00740073 006d0065 00690054 S.y.s.t.e.m.T.i.
06eaff04 0065006d 00000000 00000000 72b6d563 m.e.........c..r
06eaff14 80000000 0061007a 00650062 0050006c ....z.a.b.e.l.P.
.......
eax=0012e180 ebx=01581fb0 ecx=0012e1b8 edx=0012e8a8 esi=0012e1b8 edi=00000000
eip=00465758 esp=0012e174 ebp=0012eee4 iopl=0 nv up ei pl nz ac po nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000212
LabelPrint+0x65758:
00465758 ff1588f14800 call dword ptr [LabelPrint+0x8f188 (0048f188)] ds:0023:0048f188={OLEAUT32!VariantInit (75c13ed5)}
0:000> p
eax=00000000 ebx=01581fb0 ecx=0012e180 edx=0012e8a8 esi=0012e1b8 edi=00000000
eip=0046575e esp=0012e178 ebp=0012eee4 iopl=0 nv up ei pl zr na pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000246
LabelPrint+0x6575e:
0046575e 8b4608 mov eax,dword ptr [esi+8] ds:0023:0012e1c0=01afabd0
0:000> p
eax=01afabd0 ebx=01581fb0 ecx=0012e180 edx=0012e8a8 esi=0012e1b8 edi=00000000
eip=00465761 esp=0012e178 ebp=0012eee4 iopl=0 nv up ei pl zr na pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000246
LabelPrint+0x65761:
00465761 8b08 mov ecx,dword ptr [eax] ds:0023:01afabd0={msxml3!DOMNamedNodeMapList::`vftable' (6927cc90)}
```

对应的C代码如下

![图片描述](upload/attach/202212/949068\_KPWBA7CJ6H9KRAU.png)

上面先是调用`VariantInit`初始化`pvarg`，`pvarg`是变体类型`Variant`，`Variant` 是一种特殊的数据类型，除了定长`String`数据及用户定义类型外，可以包含任何种类的数据。`Variant` 也可以包含`Empty`、`Error`、`Nothing`及`Null`等特殊值。`VARIANT`数据结构包含两个域(如果不考虑保留的域)。`vt`域描述了第二个域的数据类型。

之后取`DOMNamedNodeMapList`对象及其虚表指针。

我们接着往下走：

```
.....
eax=01afabd0 ebx=01581fb0 ecx=6927cc90 edx=06eafea4 esi=0012e1b8 edi=00000000
eip=00465776 esp=0012e16c ebp=0012eee4 iopl=0 nv up ei pl zr na pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000246
LabelPrint+0x65776:
00465776 ff511c call dword ptr [ecx+1Ch] ds:0023:6927ccac={msxml3!DOMNamedNodeMapList::getNamedItem (6927ce11)}
0:000>
eax=00000000 ebx=01581fb0 ecx=6927ceb2 edx=00000001 esi=0012e1b8 edi=00000000
eip=00465779 esp=0012e178 ebp=0012eee4 iopl=0 nv up ei pl zr na pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000246
LabelPrint+0x65779:
00465779 85c0 test eax,eax
0:000> p
eax=01dda6c0 ebx=01661fc8 ecx=6991ceb2 edx=00000001 esi=0012e1b8 edi=00000000
eip=00465785 esp=0012e178 ebp=0012eee4 iopl=0 nv up ei pl zr na pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000246
LabelPrint+0x65785:
00465785 85c0 test eax,eax
0:000> dd eax
01dda6c0 698f6508 698f6608 699b8b60 698f6604
01dda6d0 0000000e 698f8808 698f65c8 015b8640
01dda6e0 00000000 00000000 05ba7c97 0000f7f2
01dda6f0 01dda4e0 01dd00c4 0bbb7c98 0800f7f7
01dda700 0021cfe8 0000000c 003a0043 0050005c
01dda710 006f0072 00720067 006d0061 00610044
01dda720 00610074 0043005c 00620079 00720065
01dda730 0069004c 006b006e 0045005c 006f0076
0:000> ln 698f6508
(698f6508) msxml3!DOMNode::`vftable' | (69932fbc) msxml3!DOMChildList::`vftable'
Exact matches:
msxml3!DOMNode::`vftable' = <no type information>
0:000> p
eax=00000000 ebx=01581fb0 ecx=6927ceb2 edx=00000001 esi=0012e1b8 edi=00000000
eip=0046577b esp=0012e178 ebp=0012eee4 iopl=0 nv up ei pl zr na pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000246
LabelPrint+0x6577b:
0046577b 0f8c99000000 jl LabelPrint+0x6581a (0046581a) [br=0]
```

对应的C代码如下：

![图片描述](upload/attach/202212/949068\_7XHA4Q5JFXX4STP.png)

上面调用了`DOMNamedNodeMapList::getNamedItem`将`DOMNode`对象传递给`v10`，接着跟踪：

```
.......
eax=01dda6c0 ebx=01661fc8 ecx=6991ceb2 edx=00000001 esi=0012e1b8 edi=00000000
eip=0046578d esp=0012e178 ebp=0012eee4 iopl=0 nv up ei pl nz na pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000206
LabelPrint+0x6578d:
0046578d 8b08 mov ecx,dword ptr [eax] ds:0023:01dda6c0={msxml3!DOMNode::`vftable' (698f6508)}
0:000>
eax=01dda6c0 ebx=01661fc8 ecx=698f6508 edx=00000001 esi=0012e1b8 edi=00000000
eip=0046578f esp=0012e178 ebp=0012eee4 iopl=0 nv up ei pl nz na pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000206
LabelPrint+0x6578f:
0046578f 8d542408 lea edx,[esp+8]
0:000>
eax=01dda6c0 ebx=01661fc8 ecx=698f6508 edx=0012e180 esi=0012e1b8 edi=00000000
eip=00465793 esp=0012e178 ebp=0012eee4 iopl=0 nv up ei pl nz na pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000206
LabelPrint+0x65793:
00465793 52 push edx
0:000> dd 0012e180 0012e180 是pvarg的地址
0012e180 00120000 75c14677 00491268 002062e0 这里00120000 的低两位是vt，类型为VarType
0012e190 0012eed4 0048b218 00000000 00410c26 0012e180+8是bstrVal
0012e1a0 07006984 0012e8a8 0012f360 01661fc8
0012e1b0 00000000 05c7d838 01dda490 01dda4ac
0012e1c0 01ddabd0 06f24638 0012e234 01dda458
0012e1d0 00000000 00000000 01dda3c8 06f80f74
0012e1e0 760dc744 0012e204 75db3a0c 06fe9e76
0012e1f0 00000000 0012e234 00000000 00000000
0:000> dc 00491268
00491268 0061006e 0065006d 00000000 00520054 n.a.m.e.....T.R.
00491278 00430041 0000004b 00790053 00740073 A.C.K...S.y.s.t.
00491288 006d0065 00690054 0065006d 00000000 e.m.T.i.m.e.....
00491298 00610064 00650074 00000000 00750061 d.a.t.e.....a.u.
004912a8 00680074 0072006f 00000000 00690074 t.h.o.r.....t.i.
004912b8 006c0074 00000065 004e0049 004f0046 t.l.e...I.N.F.O.
004912c8 004d0052 00540041 004f0049 0000004e R.M.A.T.I.O.N...
004912d8 006f0062 00640072 00650072 0063005f b.o.r.d.e.r.\_.c.
0:000> p
eax=01dda6c0 ebx=01661fc8 ecx=698f6508 edx=0012e180 esi=0012e1b8 edi=00000000
eip=00465794 esp=0012e174 ebp=0012eee4 iopl=0 nv up ei pl nz na pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000206
LabelPrint+0x65794:
00465794 50 push eax
0:000>
eax=01dda6c0 ebx=01661fc8 ecx=698f6508 edx=0012e180 esi=0012e1b8 edi=00000000
eip=00465795 esp=0012e170 ebp=0012eee4 iopl=0 nv up ei pl nz na pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000206
LabelPrint+0x65795:
00465795 ff5120 call dword ptr [ecx+20h] ds:0023:698f6528={msxml3!DOMNode::get\_nodeValue (6992eb26)}
0:000>
eax=00000000 ebx=01661fc8 ecx=6992ebd1 edx=00000001 esi=0012e1b8 edi=00000000
eip=00465798 esp=0012e178 ebp=0012eee4 iopl=0 nv up ei pl zr na pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000246
LabelPrint+0x65798:
00465798 66837c240808 cmp word ptr [esp+8],8 ss:0023:0012e180=0008
0:000> dd 0012e180 0012e180 是pvarg的地址
0012e180 00120008 75c14677 06fb65f4 002062e0 00120008 低两位为8代表字符串
0012e190 0012eed4 0048b218 00000000 00410c26 0012e180+8是bstrVal
0012e1a0 07006984 0012e8a8 0012f360 01661fc8
0012e1b0 00000000 05c7d838 01dda490 01dda4ac
0012e1c0 01ddabd0 06f24638 0012e234 01dda458
0012e1d0 00000000 00000000 01dda3c8 06f80f74
0012e1e0 760dc744 0012e204 75db3a0c 06fe9e76
0012e1f0 00000000 0012e234 00000000 00000000
0:000> dc 06fb65f4
06fb65f4 00420042 00420042 00420042 00420042 B.B.B.B.B.B.B.B.
06fb6604 00420042 00420042 00420042 00420042 B.B.B.B.B.B.B.B.
06fb6614 00420042 00420042 00420042 00420042 B.B.B.B.B.B.B.B.
06fb6624 00420042 00420042 00420042 00420042 B.B.B.B.B.B.B.B.
06fb6634 00420042 00420042 00420042 00420042 B.B.B.B.B.B.B.B.
06fb6644 00420042 00420042 00420042 00420042 B.B.B.B.B.B.B.B.
06fb6654 00420042 00420042 00420042 00420042 B.B.B.B.B.B.B.B.
06fb6664 00420042 00420042 00420042 00420042 B.B.B.B.B.B.B.B.
```

对应C代码如下

![图片描述](upload/attach/202212/949068\_NH78ZDPWRPMVSJ8.png)

`VarType`如下

![图片描述](upload/attach/202212/949068\_HXB37TSEBSD28HX.png)

由上可知通过调用`DOMNode::get_nodeValue`将`pvarg.vt`赋值为8，同时取`name`属性中的值放入`pvarg.bstrVal`中。

接下来就是一些数值传递：

![图片描述](upload/attach/202212/949068\_S84B7H4PQ5ZRE49.png)

最后调用拷贝函数，在此之前需要确定拷贝的目标地址：
