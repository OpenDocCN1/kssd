
`myCart`这块内存就是在`main`函数被初始化的内存，这里用来保存我们的商品数据结构。`myCart[0]`为链表头，`myCart[2]`开始才是我们的第一个商品。

```
unsigned int delete()
{
  signed int count; // [esp+10h] [ebp-38h]
  Apple *apple_target; // [esp+14h] [ebp-34h]
  int index_int; // [esp+18h] [ebp-30h]
  Apple *bk_apple; // [esp+1Ch] [ebp-2Ch]
  Apple *fd_apple; // [esp+20h] [ebp-28h]
  char index; // [esp+26h] [ebp-22h]
  unsigned int canary; // [esp+3Ch] [ebp-Ch]

  canary = __readgsdword(0x14u);
  count = 1;
  apple_target = myCart.device;
  printf("Item Number> ");
  fflush(stdout);
  my_read(&index, 0x15u);
  index_int = atoi(&index);
  while ( apple_target )
  {
    if ( count == index_int )
    {
      bk_apple = apple_target->bk;
      fd_apple = apple_target->fd;
      if ( fd_apple )
        fd_apple->bk = bk_apple;
      if ( bk_apple )
        bk_apple->fd = fd_apple;
      printf("Remove %d:%s from your shopping cart.\n", count, apple_target->name);
      return __readgsdword(0x14u) ^ canary;
    }
    ++count;
    apple_target = apple_target->bk;
  }
  return __readgsdword(0x14u) ^ canary;
}
```

`delete`函数用来将双向链表从链中拆除，操作类似于`unlink`，即修改前后链表的`fd`和`bk`，但是这个并没有任何防护手段。

```
int cart()
{
  signed int index; // eax MAPDST
  int total_price; // [esp+1Ch] [ebp-2Ch]
  Apple *apple; // [esp+20h] [ebp-28h]
  char choice; // [esp+26h] [ebp-22h]
  unsigned int canary; // [esp+3Ch] [ebp-Ch]

  canary = __readgsdword(0x14u);
  index = 1;
  total_price = 0;
  printf("Let me check your cart. ok? (y/n) > ");
  fflush(stdout);
  my_read(&choice, 0x15u);
  if ( choice == 'y' )
  {
    puts("==== Cart ====");
    for ( apple = myCart.device; apple; apple = apple->bk )
    {
      printf("%d: %s - $%d\n", ++index, apple->name, apple->price);
      total_price += apple->price;
    }
  }
  return total_price;
}
```

`cart`函数打印所有商品信息，可以考虑信息泄露。

```
unsigned int checkout()
{
  int total_price; // [esp+10h] [ebp-28h]
  Apple apple; // [esp+18h] [ebp-20h]
  unsigned int canary; // [esp+2Ch] [ebp-Ch]

  canary = __readgsdword(0x14u);
  total_price = cart();
  if ( total_price == 7174 )
  {
    puts("*: iPhone 8 - $1");
    asprintf(&apple.name, "%s", "iPhone 8");
    apple.price = 1;
    insert(&apple);
    total_price = 7175;
  }
  printf("Total: $%d\n", total_price);
  puts("Want to checkout? Maybe next time!");
  return __readgsdword(0x14u) ^ canary;
}
```

`checkout`函数乍一看好像没什么卵用，并没有后门函数什么的，但是这有一个小彩蛋，如果总价格是7174的话，会附赠你一部一美元的手机。不妨回顾一下`create`函数，正常`add`函数添加的商品结构体都是`malloc`出来的，也就是在堆上，而这个一美元的手机却是保存在栈上的，这个函数结束之后栈地址依然留在我们的双向链表之中，这有点类似于堆题目里面的Use After Free。

## 利用方法

1. 首先，我们要触发菜单，才能创造出这个堆里面的栈地址，但这个四元一次方程运算量还是有那么一点的，这里我们借助`python`来计算一下：

```
[0] % python
Python 2.7.12 (default, Apr 15 2020, 17:07:12)
[GCC 5.4.0 20160609] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> from z3 import *
>>> v1,v2,v3,v4,v5=Ints('v1 v2 v3 v4 v5')
>>> x=Solver()
>>> x.add(199*v1+299*v2+499*v3+399*v4+199*v5==7174)
>>> x.check()
sat
>>> x.model()
[v2 = -2, v4 = -10, v3 = 14, v1 = -6, v5 = 30]
>>> x.add(v1>=0,v2>=0,v3>=0,v4>=0,v5>=0)
>>> x.check()
sat
>>> x.model()
[v2 = 0, v4 = 10, v3 = 0, v1 = 16, v5 = 0]
```

也就是说，购买1号商品16个，4号商品10个正好就是7174块钱。

2. 到这可能就卡壳了，虽然我们把一个局部变量的地址放到了堆里面，但是我们并没有办法来控制它的内容，哪怕能修改，那么栈地址在函数返回之后不久没用了么，还怎么操作？这里我们先看一看`handler`函数的汇编实现：

```
.text:08048C31                 jmp     eax             ; switch jump
.text:08048C33 ; ---------------------------------------------------------------------------
.text:08048C33
.text:08048C33 loc_8048C33:                            ; CODE XREF: handler+5Ej
.text:08048C33                                         ; DATA XREF: .rodata:08049088o
.text:08048C33                 call    list            ; jumptable 08048C31 case 1
.text:08048C38                 jmp     short loc_8048C63
.text:08048C3A ; ---------------------------------------------------------------------------
.text:08048C3A
.text:08048C3A loc_8048C3A:                            ; CODE XREF: handler+5Ej
.text:08048C3A                                         ; DATA XREF: .rodata:08049088o
.text:08048C3A                 call    add             ; jumptable 08048C31 case 2
.text:08048C3F                 jmp     short loc_8048C63
.text:08048C41 ; ---------------------------------------------------------------------------
.text:08048C41
.text:08048C41 loc_8048C41:                            ; CODE XREF: handler+5Ej
.text:08048C41                                         ; DATA XREF: .rodata:08049088o
.text:08048C41                 call    delete          ; jumptable 08048C31 case 3
.text:08048C46                 jmp     short loc_8048C63
.text:08048C48 ; ---------------------------------------------------------------------------
.text:08048C48
.text:08048C48 loc_8048C48:                            ; CODE XREF: handler+5Ej
.text:08048C48                                         ; DATA XREF: .rodata:08049088o
.text:08048C48                 call    cart            ; jumptable 08048C31 case 4
.text:08048C4D                 jmp     short loc_8048C63
.text:08048C4F ; ---------------------------------------------------------------------------
.text:08048C4F
.text:08048C4F loc_8048C4F:                            ; CODE XREF: handler+5Ej
.text:08048C4F                                         ; DATA XREF: .rodata:08049088o
.text:08048C4F                 call    checkout        ; jumptable 08048C31 case 5
.text:08048C54                 jmp     short loc_8048C63
.text:08048C56 ; ---------------------------------------------------------------------------
```

只截了关键部分，提醒一下，我们将栈地址放到堆上的时候，局部变量保存在`ebp-22`的地址。通过汇编代码可以看到，每个函数在结束调用之后，并没有对栈进行操作，所以这几个函数用的栈都是同一个地址的内存空间，也就是说，如果`checkout`函数之外的函数对`ebp-22`进行了操作，也就是对堆上的第27个结构进行了操作。回顾一下`add`、`delete`、`cart`三个函数，我们就可以发现我们是可以对`ebp-22`这块地址进行操作的，只不过我们需要用`\x00`来隔断函数需要的选项和我们构造的结构体。比如`cart`函数，我们可以先输入一个`'y\x00'`来进行下一步，接着在字符串后面跟上我们构造的结构体，这样`cart`就会打印任意地址的数据。

根据这个特点，我们泄露`libc`基地址，堆地址和栈地址。

3. 光泄露了信息还不行，我们还需要进一步的操作才能`getshell`。程序还有一个漏洞，就是`delete`函数。这个函数的操作非常类似`unlink`，而且还没有任何防护，所以我们可以通过这个函数进行任意地址写，`delete`函数关键部分如下：

```
bk_apple = apple_target->bk;
fd_apple = apple_target->fd;
if ( fd_apple )
    fd_apple->bk = bk_apple;
if ( bk_apple )
    bk_apple->fd = fd_apple;
```

简化一下就成了

```
fd_apple->bk = bk 相当于 fd[3]=bk
bk_apple->fd = fd 相当于 bk[2]=fd
```

这样，我们只要把`bk`修改为目标地址-8，就可以将任意的`fd`写入目标地址了。这里我们可以劫持`ebp`，从而控制`handler`函数的栈。只要把栈迁移到`GOT`底部-22的位置，我们就可以用`handler`中的`my_read`函数改写`GOT`表中的值，从而`getshell`。

## exp
