

# [原创]4月13日 Chrome爆出的v8漏洞车祸现场分析

## 0x1: 关于v8引擎

`1.1`：v8的指针为真实地址+1，这样最后一位为1，据说是为了加快寻址的速度，根据最后一位直接就可以判断该值其代表的是指针还是自然数。

这点在逆向的角度就是优化后生成的指令，

- 看到到有 `add rX,1` 这样的指令片段一般就是代表着开始对指针指向对象的初始化。
- 在内存中看到的数值为真实数值\*2。

`1.2`：在比较新版本的v8对一个地址只是存储其低4位，要进行寻址的时候再加上其高八位。

`1.3`：v8在多次执行一个函数的时候，会触发JIT（优化）过程，直接生成函数对应的机器码，这也是这篇分析文章的理论基础。

## 0x2：JIT产生的优化指令分析

分析环境为Windows 10 x64

v8版本8.9.25

分析工具为x64debug

<https://bbs.pediy.com/thread-267049.htm>

紧跟着前面前面

poc为：

```
const _arr = new Uint32Array([2**31]);

function foo() {
    var x = 1;
    x = (_arr[0] ^ 0) + 1;

    x = Math.abs(x);
    x -= 2147483647;
    x = Math.max(x, 0);
    x -= 1;//
    if(x==-1) x = 0;
    var corr = new Array(x);
    arr.shift();
    var arr = [1.1, 1.2, 1.3];

    return [arr, cor];
}

console.log("ready !!");
for(i=0;i<0x3000;i++)
{
    foo();
}

%SystemBreak();
var x = foo();
var corr=x[0];
var arr=x[1];
%DebugPrint(corr);
%DebugPrint(arr);
console.log("Analyze Over!")
```

### 2.1：v8优化后生成指令对有符号数的错误处理分析

#### 2.1.1

x64debug打开`d8.exe`程序加载poc之后，会在运行到在`%SystemBreak()`时会断下；在断点返回以后单步运行一段时间可以看到：

```
00000039000C404C | 49:BA 60EB0C43F77F0000   | mov r10,**<d8.Builtins_CompileLazyDeoptimizedCode>**    | 7FF7430CEB60:"I嫕Hi
```

![图2.1.1.1](https://bbs.kanxue.com/upload/attach/202104/808412_HSXKFBAACHRESW6.jpg)

分析一下图2.1.1.1的指令逻辑，这里是对是否已经优化做判断，如果已经优化就执行优化后生成的指令，否则则是通过`r10`跳转到`d8.Builtins_CompileLazyDeoptimizedCode`，先执行优化过程。

运行到这里显然已经完成了优化，继续下去执行的是优化后生成的指令。

![图2.1.1.2](https://bbs.kanxue.com/upload/attach/202104/808412_XQ7WHJMF234JXPW.jpg)

![图2.1.1.3](https://bbs.kanxue.com/upload/attach/202104/4QQXM3YVDJ4Q6W5.jpg)

#### 2.1.2

调试器走到图2.1.1.3所在的位置，这里的指令为：

```
mov ecx,edi
add rcx,r8
mov ecx,dword ptr ds:[rcx]
```

这里进行的是v8的数组的取值操作，取完值以后对该值进行+1操作，对应的是poc中的`x=(_arr[0] ^ 0)+1`的js代码。

![图2.1.2.1](https://bbs.kanxue.com/upload/attach/202104/808412_WWA53GNEJ76U2FE.jpg)

![图2.1.2.2](https://bbs.kanxue.com/upload/attach/202104/808412_5B8VYAM4KKZBWGG.jpg)

走到图2.1.2.1的`add rcx,1`这句指令后，结果为`0x80000001`，并将其放在`rcx`指令中，接下来用一些特殊指令进行处理，如下图所示：

![图2.1.2.3](https://bbs.kanxue.com/upload/attach/202104/808412_2HDZ785AWXXENG7.jpg)

#### 2.1.3

这些指令对应的是poc中的`x = Math.abs(x)` js语句，应该是做了些优化；

但是这些指令执行完以后，`rcx`还是为`0x80000001`，而后面的指令中使用的是`ecx`作为参数进行运算。

也就是说v8优化后产生的这些指令，并未正确处理`x = Math.abs(x);`的操作。这也是后面一连串错误操作的起始原因。

![图2.1.3.1](https://bbs.kanxue.com/upload/attach/202104/808412_HKHP9KEKXBPSPPX.jpg)

#### 2.1.4

流程运行到这里直接用`rdi`寄存器的的`ecx`来进行计算，由上图可以看到，导致运算为`0x80000001-0x7fffffff`，结果等于2。

接下来是指令

```
cmp ecx,0
jl 39000C40CA
```

这里是执行源poc里面的`x = Math.max(x, 0)`的js语句，最后结构返回`x=2`。这段指令将结果放入到`rdi`寄存器中，此时`edi`表示参数`x`。

![图2.1.4.1](https://bbs.kanxue.com/upload/attach/202104/808412_H5AC68PFWTQY4D3.jpg)

![图2.1.4.2](https://bbs.kanxue.com/upload/attach/202104/808412_CYXZT64ZMB8SYAW.jpg)

![图2.1.4.3](https://bbs.kanxue.com/upload/attach/202104/808412_K4EXTESGUGSHQ3B.jpg)

#### 2.1.5

上图所示，`edi`进行-1运算(也就是poc js文件里面的参数`x`)，此时`edi=1`，结果自然是不等于`0xFFFFFFFF`，最后越过poc中的`x==-1`的判断。对应poc中的js代码`if(x==-1) x = 0;`

然后在之后的流程里`var corr = new Array(x);`这语句执行的时候，就会生成一个大小为1的数组

### 2.2：漏洞数组初始化分析

![图2.2.1.1](https://bbs.kanxue.com/upload/attach/202104/808412_564QPBQ4TB9Y7AF.jpg)

图2.1.4.1上`EIP`指向的指令为

```
cmp rdi,0
je 39000C419D
```

也就是对`x`是否为0进行了判断。

根据v8指针的特点，以及调试分析可以判断，这里执行的流程是进行数组对象的初始化。

#### 2.2.1：初始化`r8-1`指向的数组（这里将其定义为`array1`）

```
add r8,1
lea r9d,qword ptr ds:[rdi+rdi]//rdi=x
mov r12,qword ptr ds:[r13+D0]
mov dword ptr ds:[r8-1],r12d  //初始化array1数组的map。
mov dword ptr ds:[r8+3],r9d   //初始化array1数组的大小，内存值为2*x。
xor r9,r9
lea r14,qword ptr ds:[r9+1]
mov r15,qword ptr ds:[r13+98]
mov dword ptr ds:[r8+r9*4+7],r15d  /*这里进行了初始化，也就是该数组第一个元素为[r8+7]，在[array1+8] */
mov r9,r14
cmp r9,rdi
jb 99000C4180
```

![图2.2.1.2](https://bbs.kanxue.com/upload/attach/202104/808412_V4G9H2S27AV2EMM.jpg)

#### 2.2.2：图2.2.1.2`EIP`前面的一段指令是初始化`r9-1`指向的数组（这里将其定义为`array2`）

```
add r9,1
mov r14d,824394D
mov dword ptr ds:[r9-1],r14d  //初始化array2数组的map
mov r14,qword ptr ds:[r13+158]
mov dword ptr ds:[r9+3],r14d  //初始化array2数组的prototype
lea r15d, qword ptr ds:[rdi+rdi]
mov dword ptr ds:[r9+7],r8d   //初始化array2数组的大小，内存值为2*x。
mov [r9+7],r8d  //初始化array2数组的elements，让其指向array1
```

![图2.2.1.1、R8、R9寄存器此时指向的内存](https://bbs.kanxue.com/upload/attach/202104/808412_HS5W44G27HS7CAM.jpg)

![图2.2.2.2、array1指向的内存](https://bbs.kanxue.com/upload/attach/202104/808412_E7C2RNN7PFVQZ99.jpg)

![图2.2.2.3、array2指向的内存](https://bbs.kanxue.com/upload/attach/202104/808412_QAGJ28VPQ8SHFN6.jpg)

#### 2.2.3

此时`array1`和`array2`两者内存关系如下图所示：

![图2.2.3.4、array1和array2的内存关系图](https://bbs.kanxue.com/upload/attach/202104/808412_74NTR6HR6EJCGEN.jpg)

这内存关系在后面的漏洞利用比较重要。如果改变了`sizeofarray1`的值，就对后面的内存进行读写，我们在poc中的js对`corr`进行索引，其实是索引`element1`开始的内存，也就是实际上控制这里的`array1`数组。`corr[0]`索引到的值为`element1`，`corr[1]`索引到的值为`array2`（map），`corr[2]`索引为到的值为`sizeofarray2`，由此继续下去。

### 2.3：数值越界的直接原因

#### 2.3.1

接下来执行的是对应的是poc中的`corr.shift();`这句的流程：

![图2.3.1.1](https://bbs.kanxue.com/upload/attach/202104/808412_64GE8V2V8BRKE9A.jpg)

图2.3.1.1这里的指令为：`mov dword ptr ds:[r9+B],FFFFFFFE;`

接下来的指令是直接将`0xFFFFFFFE`这个硬编码放入了`array2`存放数组大小的内存位置中，v8引擎对`array2`索引的时候，会把`0xFFFFFFFE`当成`array2`数组的大小，从而将`array2`视为一个`0xFFFFFFFE/2`大小的数组。(不过经过调试这个内存的数值并不是v8用来索引poc中js数组`corr`的数值，只要该值不为0，就会去索引实际控制内存的数组`array1`)

#### 2.3.2：紧接着会执行指令

```
mov rdi,qword ptr ds:[r13+98]
mov dword ptr ds:[r8+3],edi
```

这两句指令是对原来`array1`的数组大小的内存位置，也就是`[array1+4]`的值进行填充，填充的值为`[r13+98]`指向的值，这里是数字`0x08042429`，v8引擎会在后面把这个填进来的数字解释为`array1`的大小。(这是造成这漏洞数组`corr`可以越界读写的直接原因)

![图 2.3.2.1](https://bbs.kanxue.com/upload/attach/202104/808412_MTBF6CMAYFBJE6D.jpg)

对这一段指令总结：

- 在v8执行生成和执行`poc`优化后的`corr.shift()` js指令时，并没有对`array2`指向`array1`的关系进行改变。
- 在对`array2`数组处理时，原来存放数组大小的位置改为了数字`0xFFFFFFFE`，但这个数值v8会认为其是合法的数组数值，因此我们poc中的js语句中还是用`corr`数组还是可以继续索引到`array1`数组，而`array1`会因为优化的原因，把代表着`array1`长度的内存值修改为`0x08042429`(这个数值根据调试环境而定，但是一定是个比2大很多的值)。而v8解释器会把这个值解释为`array1`数组的大小也就是poc的js中`corr`数组的大小，从而造成了`corr`数组可以越界读写。

## 0x3：浮点数组生成验证越界读写

这一段是开始其实算是漏洞利用的部分，和我要分析的东西没太大关联，在这里我是用来验证越界读取的位置，不过从逆向分析的角度来看这一段加进来还是很有必要的，毕竟浮点数的初始化的特征非常明显，可以帮助你很快定位到优化后生成的指令：

![图3.1.1.1](https://bbs.kanxue.com/upload/attach/202104/808412_274RCTD9KZDTE6U.jpg)

### 3.1.1

图3.1.1.1的流程是在`array2`数组的内存之后生成一个浮点数组，在这个浮点数组生成后，整个内存结构就变成如下图所示：

![图3.1.1.2](https://bbs.kanxue.com/upload/attach/202104/808412_P2T3MGBENGWDGYJ.jpg)

### 3.1.2

最后生成内存数据排列如图3.1.2.1图所示：

![图3.1.2.1](https://bbs.kanxue.com/upload/attach/202104/808412_VAAA64GM4WBR8J9.jpg)

### 3.1.3

图3.1.2.1的位置关系可以看到，`corr`发生了越界读写的话，`corr[8]`如果合法会读到到了浮点数`0x3FF199999999999A`的前四个字节`0x3FF19999`，v8把这值当成是地址解析，所以会在前面加上`external_pointer`，在这调试环境下是`0x003900000000`，最后读取出来就`corr[8]`会变成了`0x00393ff19999`，如下图所示：

![图3.1.3.1](https://bbs.kanxue.com/upload/attach/202104/808412_RSSRVZ45DNQBXM2.jpg)

### 3.1.4

这里可以看到，此时原来的`array1`大小区域变为`0x08042429`，v8把他他当成数组的大小就会解释为`0x08042429/2==0x04021214`，也就是十进制数67244564。(这个值和该数组第一个元素的尾部四位是一样的，也就是在内存中为同样的值而表示`array2`的大小的内存值为`0xFFFFFFFE`，在这里打印出`corr.length`的时候解释为-1，但是在实际判断中却被先当成有符号数-2，然后除2为-1，也就是`0xFFFFFFFF`(十进制数为4294967295)，然后作为合法的数组大小索引，这里用Ubuntu上的v8运行可以直接看到：

![图3.1.4.1](https://bbs.kanxue.com/upload/attach/202104/808412_JK4985CP8ES4P3X.jpg)

## 0x4：写在最后

