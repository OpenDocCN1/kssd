
## 汇编代码片段

```
CODE:0040EA2D mov     ebx, eax

CODE:0040EA2F mov     esi, [ebx+TList_obj.Property2]

CODE:0040EA32 cmp     esi, [ebx+TList_obj.Property3]

CODE:0040EA35 jnz     short loc_40EA3D

CODE:0040EA37 mov     eax, ebx

CODE:0040EA39 mov     edx, [eax+TList_obj.pVTBL]

CODE:0040EA3B call    dword ptr [edx] ;TList::Grow

CODE:0040EA3D

CODE:0040EA3D loc_40EA3D:                             ; CODE XREF: Classes::TList::Add(void *)+Dj

CODE:0040EA3D mov     eax, [ebx+TList_obj.Property1]

CODE:0040EA40 mov     [eax+esi*4], edi

CODE:0040EA43 inc     [ebx+TList_obj.Property2]

CODE:0040EA46 mov     eax, esi

CODE:0040EA48 pop     edi

CODE:0040EA49 pop     esi

CODE:0040EA4A pop     ebx

CODE:0040EA4B retn

CODE:0040EA4B __fastcall Classes::TList::Add(void *) endp
```

考虑到`VBTL`的结构，很容易想到:

```
CODE:0040EA3B call    dword ptr [edx]
```

就是`TList::Grow`,

因为

```
CODE:0040D5D8 pVTBL dd offset TList::Grow
```

现在我们可以对类的成员做一点深入的分析了。比方说，看到下面的代码:

```
CODE:0040EA3D mov     eax, [ebx+TList_obj.Property1]

CODE:0040EA40 mov     [eax+esi*4], edi

CODE:0040EA43 inc     [ebx+TList_obj.Property2]
```

就可以知道`Property2`是`TList`中元素的计数器。因为增加一个元素时，它也被加一。`Property1`是指向元素数组的指针。`Property 2`可以看作是数组的索引。而`Property 3`则是一个`list`里最多允许的元素数目。此外，只有当`Property2`等于`Property3`时，`TList::Grow`被调用。通过逻辑推理，我们知道了这些。现在，一切都清楚起来了。顺便看看帮助文档，给这些成员命名吧：

```
CODE:0040EA28 __fastcall Classes::TList::Add(void *) proc near

CODE:0040EA28                                         ; CODE XREF: @RegisterClass+9Bp

CODE:0040EA28                                         ; @RegisterIntegerConsts+20p ...

CODE:0040EA28                 push    ebx

CODE:0040EA29                 push    esi

CODE:0040EA2A                 push    edi

CODE:0040EA2B                 mov     edi, edx

CODE:0040EA2D                 mov     ebx, eax

CODE:0040EA2F                 mov     esi, [ebx+TList_obj.Count]

CODE:0040EA32                 cmp     esi, [ebx+TList_obj.Capacity]

CODE:0040EA35                 jnz     short loc_40EA3D

CODE:0040EA37                 mov     eax, ebx

CODE:0040EA39                 mov     edx, [eax+TList_obj.pVTBL]

CODE:0040EA3B                 call    dword ptr [edx]

CODE:0040EA3D

CODE:0040EA3D loc_40EA3D:                             ; CODE XREF: Classes::TList::Add(void *)+Dj

CODE:0040EA3D                 mov     eax, [ebx+TList_obj.Items]

CODE:0040EA40                 mov     [eax+esi*4], edi

CODE:0040EA43                 inc     [ebx+TList_obj.Count]

CODE:0040EA46                 mov     eax, esi

CODE:0040EA48                 pop     edi

CODE:0040EA49                 pop     esi

CODE:0040EA4A                 pop     ebx

CODE:0040EA4B                 retn

CODE:0040EA4B __fastcall Classes::TList::Add(void *) endp
```

对象的结构已经分析好了，下面是对象成员。

## 查找对象方法

对象的方法可以是以下几种:公开/私有(保护)，虚方法/非虚方法以及静态方法.

由于编译后的静态方法和普通的过程没有什么区别，所以静态方法是无法被识别的。这些函数和某个特定的类之间的关系也是无法确定的。但是，应该指出的是，如果某个静态方法在类的方法里被调用，那么，它是可见的。否则寻找静态方法的企图只是在浪费时间。

虚方法很容易找到――它们都位于`VTBL`里。但是我们应该如何查找一般的方法呢？想想`OOP`：当对象方法被调用时，指向对象本身的指针被隐含地传递给该方法。实际上，这就意味着每个方法的第一个参数就是指向对象的指针。也就是说，如果该方法被声明为`fastcall`类型，指向对象的指针是放在`EAX`里的。而对于`cdecl`或`stdcall`类型的方法，首个参数是放在堆栈里的。让我们来看看指向对象的指针被放在什么地方……好！在`dword_4A45F8`里。通过查找对`4A45F8`的交叉引用，我们可以找到很多非虚拟方法。我们还可以在`4A45F8`下一个断点，追踪对对象实例指针的复制以找出余下的方法。

在本例中，由于使用了全局变量，一切都很容易。但是如果使用的是局部变量或者代码无法被执行（比如说，一个驱动程序。或者该代码不允许被执行），又应该怎么做呢？这就需要一个特别的办法。

### 步骤

1) 首先要找到所有调用构造函数的地方。

对每个调用重复以下步骤

2) 跟去看看指向当前对象实例的指针被写到哪里了。

3) 把所有调用了构造函数的函数作为对象方法。

4) 如果没有这样的函数调用，就看构造函数下面的一个调用。否则就查看所有对已经找到的方法的交叉引用。这样就可以找到不在构造函数附近的调用。由于我们已经知道方法的首个参数是指向对象本身的指针，于是就可以查找对象指针的交叉引用。用这样的方法，我们可以一层一层地分析代码，直到出现僵局或者找到对象方法。

5) 分析下一个已经找到的方法。

例如，我们已经找到了`Classes::TList::Add`，而且也找到了对`Classes::TList::Add`的一个引用：

```
CODE:0040F020 TThreadList::Add proc near              ; CODE XREF: TCanvas::`...'+9Ep

CODE:0040F020                                         ; Graphics::_16725+C4p

CODE:0040F020

CODE:0040F020 var_4           = dword ptr -4

CODE:0040F020

CODE:0040F020                 push    ebp

CODE:0040F021                 mov     ebp, esp

CODE:0040F023                 push    ecx

CODE:0040F024                 push    ebx

CODE:0040F025                 mov     ebx, edx

CODE:0040F027                 mov     [ebp+var_4], eax

CODE:0040F02A                 mov     eax, [ebp+var_4]

CODE:0040F02D                 call    TThreadList::LockList

CODE:0040F032                 xor     eax, eax

CODE:0040F034                 push    ebp

CODE:0040F035                 push    offset loc_40F073

CODE:0040F03A                 push    dword ptr fs:[eax]

CODE:0040F03D                 mov     fs:[eax], esp

CODE:0040F040                 mov     eax, [ebp+var_4]

CODE:0040F043                 mov     eax, [eax+4]

CODE:0040F046                 mov     edx, ebx

CODE:0040F048                 call    TList::IndexOf

CODE:0040F04D                 inc     eax

CODE:0040F04E                 jnz     short loc_40F05D

CODE:0040F050                 mov     eax, [ebp+var_4]

CODE:0040F053                 mov     eax, [eax+4]

CODE:0040F056                 mov     edx, ebx

CODE:0040F058                 call    Classes::TList::Add(void *)
```

就是说，我们找到了`TList::IndexOf`方法。

进一步分析发现，我们处在`TthreadList`对象的方法中，`TList`是它的成员之一。这里没有什么可以看的东西。假定一下，没有其他对`Classes::TList::Add`的引用。进到`TList::IndexOf`方法并且查看对它的引用。下面是其中的一个：

```
CODE:0040EE38 TList::Remove   proc near               ; CODE XREF: TThreadList::Remove+28p

CODE:0040EE38                                         ; TCollection::RemoveItem+Bp ...

CODE:0040EE38                 push    ebx

CODE:0040EE39                 push    esi

CODE:0040EE3A                 mov     ebx, eax

CODE:0040EE3C                 mov     eax, ebx

CODE:0040EE3E                 call    TList::IndexOf

CODE:0040EE43                 mov     esi, eax

CODE:0040EE45                 cmp     esi, 0FFFFFFFFh

CODE:0040EE48                 jz      short loc_40EE53

CODE:0040EE4A                 mov     edx, esi

CODE:0040EE4C                 mov     eax, ebx

CODE:0040EE4E                 call    TList::Delete

CODE:0040EE53

CODE:0040EE53 loc_40EE53:                             ; CODE XREF: TList::Remove+10j

CODE:0040EE53                 mov     eax, esi

CODE:0040EE55                 pop     esi

CODE:0040EE56                 pop     ebx

CODE:0040EE57                 retn

CODE:0040EE57 TList::Remove   endp
```

这样，`TList::Delete`和`TList::Remove`就有了。

下面就是所有对象指针的交叉引用和相关变量。

这里是查找变量的例子：

```
CODE:0041319D mov     eax, [ebp+var_4]

CODE:004131A0 mov     edx, ds:pTList

CODE:004131A6 mov     [eax+30h], edx  ;对象指针

CODE:004131A9 jmp     short loc_4131BD
```

下面可以看到：

```
CODE:00413236 mov     eax, [eax+30h]

CODE:00413239 mov     edx, [ebp+var_10]

CODE:0041323C call    TList::Get
```

## 如何分辨公开方法和私有方法

只有当所有的方法全部找到以后才可以做这件事。私有方法只有在其它方法里才有调用。就是说，必须查看交叉引用了。查找方法以前，建议先把它们编号。也即把你找到的方法依次命名为`Object1::Method1`，`Object1::Method2`……所有的方法全部出来以后，就可以开始分析它们的参数（主要是个数和类型）了。

## 确定方法参数的个数

关于`cdecl`和`stdcall`几乎没有什么可说的。只要把`IDA`找到的参数个数减去`1`就可以了（还记得吗？第一个参数是对象指针，其它的才是真正的参数）。

`fastcall`要复杂点儿。首先我们要记住参数的次序：`EAX`，`EDX`，`ECX`，堆栈。首先要看看`IDA`找到了几个通过堆栈传递的参数。如果至少有一个，那么参数的个数要加`3`（`3`个寄存器参数加上堆栈参数）。由于第一个参数是对象指针`This`，这个数目还要减去`1`才是真正的参数个数。如果没有堆栈参数的话，就要看看函数的开头了。由于`Delphi`试图不去搅乱寄存器的值，结果每个`fastcall`函数的开头都要保存`EAX`，`EDX`和`ECX`：

```
mov esi, edx ; 第一个参数

mov ebx, eax ; This指针

mov edi, ecx ; 第二个参数
```

根据被复制的寄存器个数就可以判断出参数的个数。比如：

```
CODE:0040EBE0 TList::Get      proc near               ; CODE XREF: @GetClass+1Dp

CODE:0040EBE0                                         ; @UnRegisterModuleClasses+24p ...

CODE:0040EBE0

CODE:0040EBE0 var_4           = dword ptr -4

CODE:0040EBE0

CODE:0040EBE0                 push    ebp

CODE:0040EBE1                 mov     ebp, esp

CODE:0040EBE3                 push    0

CODE:0040EBE5                 push    ebx

CODE:0040EBE6                 push    esi

CODE:0040EBE7                 mov     esi, edx

CODE:0040EBE9                 mov     ebx, eax

CODE:0040EBEB                 xor     eax, eax
```

一共`2`个参数，其中一个是`This`指针。那么`TList::Get`有`1`个参数。

```
CODE:004198CC                 push    ebp

CODE:004198CD                 mov     ebp, esp

CODE:004198CF                 add     esp, 0FFFFFF8Ch

CODE:004198D2                 push    ebx
```
