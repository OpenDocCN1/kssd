
# 在论坛2位高手mm的基础上谈谈结构化异常处理SEH (理论+CrackMe例子分析)

由于不是破文，也不知道前面该加点什么

最近刚好看到`riijj` mm的`CrackMe11`，原帖：

http://bbs.pediy.com/showthread.php?s=&threadid=38021

和绫濑遥 mm的破文，原帖：

http://bbs.pediy.com/showthread.php?s=&threadid=38125

2位mm都讲的很到位，偶原本也没什么好说的了，不过偶看到网上很多关于`SEH`的资料都是理论居多，较少有和实际分析的结合，刚好这里又有个例子，所以我在2 mm的基础上说几句，我力求把关键的东东讲的明白。水平有限，不对的地方欢迎大家指出。

为了说清楚问题，文章可能会有点长，如果你想对`SEH`有更深的理解，我想看了应该是对你有些帮助的。当然大牛可以跳过，完全不用理偶。

首先一步一步讲几个`SEH`相关的概念，会出现的数据结构和需要注意的地方，我认为这是必须要知道的：

## 1. 所谓结构化异常处理`SEH`,我们要知道它属于`OS`给我们提供的一种机制，也就是说，`windows`系统给所有程序都默认做了这个处理，和程序的开发环境开发语言（`vc`、`vb`、`c#`...）是无关的。平时我们没有捕获异常代码的时候，系统就做默认处理，比如程序崩溃时常见的那个错误提示框，就是系统的处理。

## 2. 我们知道`cpu`是以线程为单位来分配运行时间片的，所以`os`也是以线程为单位来处理异常的。不同线程一般有不同异常处理函数。当然也可以多个线程使用同一个函数来处理。

## 3. 异常发生的时候，系统到底做了什么呢：当`cpu`执行出现异常的时候，它会产生一个中断，通过中断将控制权转交到操作系统，操作系统就调用异常处理函数做处理。注意异常处理函数可能不止一个，而是以链表的形式有很多个。这里就会有2个问题：

a. 要是有很多个异常处理函数，是全部都要调用吗？

b. 操作系统是如何取得异常处理函数的地址的？

下面的4和5分别来回答这2个疑问。

## 4. 异常处理函数是有一个返回值的，系统通过这个返回值来决定是否还要调用后面的异常处理函数。比如第一个异常处理函数它的返回值表示“我成功处理了这个异常，程序可以继续执行”(这里我用汉语，方便大家理解)，那系统就不用调用后面的异常处理函数了，而是将控制权交还程序，让它继续执行。而如果第一个异常处理函数返回值为“郁闷，偶没搞定这个异常，你让后面的兄弟来处理吧”，那么操作系统就会取下一个异常处理函数，让它来处理。最后如果该线程链表上的所有异常处理函数都没能成功处理这个异常，那操作系统就会释放对应的资源，然后毫不犹豫地`kill`掉这个程序。

## 5. 我们来回答3中的问题b，这里会引出几个重要的数据结构，希望大家好好理解。我们经常说任何事情都有一个环境，线程也不例外。操作系统在启动一个线程的时候，也会保存该线程的一个运行环境。这就是我们经常说的线程的环境块`TEB`，早些时候的操作系统称为`TIB`，这个`TIB`好象很神秘，那么它到底是个什么东东呢？？其实它就是一个数据结构，里面保存了与本线程相关的一些信息而已。可能到这里你已经想到了，对，我们的异常处理函数的链表指针就保存在这里。哈哈，到此我们b问题基本搞明白了，下面讲解一下`TIB`这个结构，本来该结构早有大牛分析过了，不过由于它很重要，不弄明白它后面会比较吃力，所以我这里再说下：

```c
typedef struct _TIB
{
PEXCEPTION_REGISTRATION_RECORD pvExcept;//我们只关心这里，注意是一个结构的指针
...
后面还有很多，由于我们这里不用，我省略之
...
} TIB, *PTIB;
```

下面看看`EXCEPTION_REGISTRATION_RECORD`结构的定义：

```c
typedef struct _EXCEPTION_REGISTRATION_RECORD
{
    struct _EXCEPTION_REGISTRATION_RECORD * pNext;
    FARPROC                                 pfnHandler;
} EXCEPTION_REGISTRATION_RECORD, *PEXCEPTION_REGISTRATION_RECORD;
```

这个结构就十分简单了，一个指向自身`EXCEPTION_REGISTRATION_RECORD`结构的指针，一个`FARPROC`，也就是函数的指针。`pNext`指向下一个异常处理结构的地址，通过它就构成了上面4中所说的异常处理函数的链表(当然其实是一个`EXCEPTION_REGISTRATION_RECORD`结构指针的链表，为了便于理解，请原谅我使用“异常处理函数链表”)，而`pfnHandler`也就是当前异常处理函数的指针了。

这里特别说明`TIB`结构的指针是保存在`FS:[0]`这里的！！！！！！！！！！！！！

罗罗嗦嗦的说了这么多，也不知道你是否理解。明白了我们就接着讲，要是没理解请再看一遍。。。如果还不理解，请相信是我没讲明白，你可以休息休息再来研究。

`Go on...`

## 6. 我们在写程序的时候，为了程序的健壮性，可能会不止一处用`__try __except __finally`这样的代码来捕获并处理异常。当这样的语句太多的时候，如果我们为每个`__try`写一个异常处理函数，那异常处理函数就会非常多，这是我们不愿意看到的。到这里不得不佩服一下`MS`，它早想到了这个问题，并且给我们完美的解决了。说白了挺简单：不管有多少个`__try`，我们只写一个异常处理函数，不同的`__try`我们给这唯一的一个异常处理函数传递不同的参数，根据参数做不同处理。那么这个参数都需要包含什么信息呢？这里又要引入一个数据结构了，我们传的参数就是这个数据结构的指针，一切一切需要的信息都在这个结构中。下面是这个叫`SCOPETABLE`的结构的定义：

```c
typedef struct _SCOPETABLE
{
DWORD previousTryLevel;        //外层try链表指针
DWORD lpfnFilter;                //异常处理代码地址
DWORD lpfnHandler;                //如果异常成功处理后程序接着执行的地址
} SCOPETABLE, *PSCOPETABLE;
```

上面结构中的3个变量我都做了说明，可是似乎没有哪一个是标志特定`__try`的啊。聪明！！！还真是没有，所以还需要一个参数来唯一确定是哪个`__try`中抛出的异常。所以我们引入一个和`__try`一一对应的标识做为另一个参数，叫`TryLevel`吧，听起来还是蛮专业的，嘿嘿...

操作系统如何处理这个`TryLevel`呢？在没有`__try`的时候，初始化`TryLevel=-1`。以后第1个`__try`时`TryLevel=0`，第2个`__try`时`TryLevel=1`，第2个`__try`时`TryLevel=2` ...以次类推。我们仔细想象，这样不但区别了不同的`__try`，是不是还能有其他的作用，其他的什么作用呢？？注意到`0 1 2 3 4...`这样的东东是不是很象数组的下标，哈哈，对，操作系统正是用了它来做为`SCOPETABLE`结构的索引。

理论先说这么多，基本说完一个`SEH`的轮廓了，如果你坚持看到了这里，那么恭喜，你成功一半了。接下来我们以`riijj` mm的`CrackMe11`为例子结合绫濑遥 mm的破文来对上面提到的几点东东做实际代码级的分析。

`OD`打开`CrackMe11.exe`，程序停在入口点，这里就出现了对`SEH`的初始化，这就是系统默认的异常处理设置，看下面代码：

```
004022AA >/$  55            PUSH EBP                                        ；这是程序入口点，标准初始化操作
004022AB  |.  8BEC          MOV EBP,ESP
004022AD  |.  6A FF         PUSH -1                                                ；上面6中说的，初始化TryLevel=-1
004022AF  |.  68 40A24000   PUSH riijjcm1.0040A240                ；6中2个参数的SCOPETABLE的结构指针
004022B4  |.  68 E81F4000   PUSH riijjcm1.00401FE8      ；00401FE8地址是异常处理函数入口
004022B9  |.  64:A1 0000000>MOV EAX,DWORD PTR FS:[0]
004022BF  |.  50            PUSH EAX                                        ；原始指针
004022C0  |.  64:8925 00000>MOV DWORD PTR FS:[0],ESP        ；设置新指针
```

上面的代码也就是在栈上构造了一个5中讲到的`EXCEPTION_REGISTRATION_RECORD`结构，并使这个结构的`pNext`指向了原始的`EXCEPTION_REGISTRATION_RECORD`结构。也就是在原来的异常处理函数链的头上加上了我们构造的这个新的`EXCEPTION_REGISTRATION_RECORD`结构，从而有异常的时候我们的异常处理函数会有优先处理该异常的机会，谁让偶在链表头呢？？嘿嘿。至于偶处理后返回什么值，是否还要向链表后面的异常处理函数传递该异常，程序是否还能继续执行等等，就看偶心情了(如不明白偶在说什么，请看上面的第5点，那个引号里面的汉语说明)，哈哈。

接着我们看`WinMain`代码部分，前面是热身，这才是重点。`WinMain`的入口在`00401BB0`，别问偶是怎么看出来的，这不属于本文要讲解的范围。看代码：

```
00401BB0   $  55            PUSH EBP                        ;函数标准入口处理方式
00401BB1   .  8BEC          MOV EBP,ESP
00401BB3   .  6A FF         PUSH -1                         ;初始化trylevel=-1
00401BB5   .  68 18A24000   PUSH riijjcm1.0040A218          ;6中SCOPETABLE结构指针
00401BBA   .  68 E81F4000   PUSH riijjcm1.00401FE8          ;异常处理函数地址
00401BBF   .  64:A1 0000000>MOV EAX,DWORD PTR FS:[0]
00401BC5   .  50            PUSH EAX                        ;新的结构中指向原来结构的指针
00401BC6   .  64:8925 00000>MOV DWORD PTR FS:[0],ESP        ;设置FS[0]为上面构造的结构的指针
00401BCD   .  83EC 08       SUB ESP,8                       ;给局部变量分配空间
00401BD0   .  53            PUSH EBX
00401BD1   .  56            PUSH ESI
00401BD2   .  57            PUSH EDI
00401BD3   .  8965 E8       MOV DWORD PTR SS:[EBP-18],ESP
00401BD6   .  C745 FC 00000>MOV DWORD PTR SS:[EBP-4],0      ;设置trylevel=0，也就是 try{ 开始SEH
00401BDD   .  33C0          XOR EAX,EAX                     ;EAX=0
00401BDF   .  C600 00       MOV BYTE PTR DS:[EAX],0         ;mov [0],0这里会抛异常
00401BE2   .  C745 FC FFFFF>MOV DWORD PTR SS:[EBP-4],-1
00401BE9   .  EB 4C         JMP SHORT riijjcm1.00401C37                ;这里跳了就完蛋
00401BEB      B8 01000000   MOV EAX,1                       ;__except(1)
00401BF0   .  C3            RETN
00401BF1   .  8B65 E8       MOV ESP,DWORD PTR SS:[EBP-18]
00401BF4   .  C745 FC 01000>MOV DWORD PTR SS:[EBP-4],1
00401BFB   .  33C0          XOR EAX,EAX
00401BFD   .  33D2          XOR EDX,EDX
00401BFF   .  F7F0          DIV EAX                         ;__try{0/0}
00401C01   .  83C8 FF       OR EAX,FFFFFFFF
00401C04   .  EB 2B         JMP SHORT riijjcm1.00401C31                ;这里跳了就完蛋
00401C06   .  B8 01000000   MOV EAX,1                       ;__except(1)
00401C0B   .  C3            RETN
00401C0C   .  8B65 E8       MOV ESP,DWORD PTR SS:[EBP-18]
00401C0F   .  C745 FC 02000>MOV DWORD PTR SS:[EBP-4],2
00401C16   .  E8 55FBFFFF   CALL riijjcm1.00401770                        ;该call调试模式下不会抛异常
00401C1B      EB 0E         JMP SHORT riijjcm1.00401C2B                ;这里跳了就完蛋
00401C1D   .  B8 01000000   MOV EAX,1
00401C22   .  C3            RETN
00401C23   .  8B65 E8       MOV ESP,DWORD PTR SS:[EBP-18]
00401C26   .  E8 75FDFFFF   CALL riijjcm1.004019A0
00401C2B   >  83C8 FF       OR EAX,FFFFFFFF
00401C2E   .  8945 FC       MOV DWORD PTR SS:[EBP-4],EAX
00401C31   >  8945 FC       MOV DWORD PTR SS:[EBP-4],EAX
00401C34   .  8945 FC       MOV DWORD PTR SS:[EBP-4],EAX
00401C37   >  33C0          XOR EAX,EAX
00401C39   .  8B4D F0       MOV ECX,DWORD PTR SS:[EBP-10]
00401C3C   .  64:890D 00000>MOV DWORD PTR FS:[0],ECX
00401C43   .  5F            POP EDI
00401C44   .  5E            POP ESI
00401C45   .  5B            POP EBX
00401C46   .  8BE5          MOV ESP,EBP
00401C48   .  5D            POP EBP
00401C49   .  C2 1000       RETN 10
```

上面是`WinMain`的全部代码，有些注释，还很多都是绫濑遥 mm破文中的，下面我们来一句一句的分析：

```
00401BB0   $  55            PUSH EBP                        ;函数标准入口处理方式
00401BB1   .  8BEC          MOV EBP,ESP
00401BB3   .  6A FF         PUSH -1                         ;初始化trylevel=-1
00401BB5   .  68 18A24000   PUSH riijjcm1.0040A218          ;6中SCOPETABLE结构指针
00401BBA   .  68 E81F4000   PUSH riijjcm1.00401FE8          ;异常处理函数地址
00401BBF   .  64:A1 0000000>MOV EAX,DWORD PTR FS:[0]
00401BC5   .  50            PUSH EAX                        ;新的结构中指向原来结构的指针
00401BC6   .  64:8925 00000>MOV DWORD PTR FS:[0],ESP        ;设置FS[0]为上面构造的结构的指针
00401BCD   .  83EC 08       SUB ESP,8                       ;给局部变量分配空间
```
