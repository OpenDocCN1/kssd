
# [原创]linux C rand(),srand()函数算法

做到一道Linux逆向题跟踪`rand()`，用网上说的两种线性同余算法均不正确，于是查看源码找到了现版本linux C库的`rand()`和`srand()`实现。

## `rand()`

`rand()`函数源码如下

```C
int
__random_r (buf, result)
struct random_data *buf;
int32_t *result;
{
int32_t *state;
if (buf == NULL || result == NULL)
goto fail;
state = buf->state;
if (buf->rand_type == TYPE_0)
{
int32_t val = state[0];
val = ((state[0] * 1103515245) + 12345) & 0x7fffffff;
state[0] = val;
*result = val;
}
else
{
int32_t *fptr = buf->fptr;
int32_t *rptr = buf->rptr;
int32_t *end_ptr = buf->end_ptr;
int32_t val;
val = *fptr += *rptr;
/* Chucking least random bit. */
*result = (val >> 1) & 0x7fffffff;
++fptr;
if (fptr >= end_ptr)
{
fptr = state;
++rptr;
}
else
{
++rptr;
if (rptr >= end_ptr)
rptr = state;
}
buf->fptr = fptr;
buf->rptr = rptr;
}
return 0;
fail:
__set_errno (EINVAL);
return -1;
}
```

可以看到，随机算法取决于`buf->rand_type`，同时`buf->state`也起到了至关重要的作用，进一步查看`buf`的类型`struct random_data`

```C
static struct random_data unsafe_state =
{
/* FPTR and RPTR are two pointers into the state info, a front and a rear
pointer. These two pointers are always rand_sep places aparts, as they
cycle through the state information. (Yes, this does mean we could get
away with just one pointer, but the code for random is more efficient
this way). The pointers are left positioned as they would be from the call:
initstate(1, randtbl, 128);
(The position of the rear pointer, rptr, is really 0 (as explained above
in the initialization of randtbl) because the state table pointer is set
to point to randtbl[1] (as explained below).) */
fptr : &randtbl[SEP_3 + 1],
rptr : &randtbl[1],
/* The following things are the pointer to the state information table,
the type of the current generator, the degree of the current polynomial
being used, and the separation between the two pointers.
Note that for efficiency of random, we remember the first location of
the state information, not the zeroth. Hence it is valid to access
state[-1], which is used to store the type of the R.N.G.
Also, we remember the last location, since this is more efficient than
indexing every time to find the address of the last element to see if
the front and rear pointers have wrapped. */
state : &randtbl[1],
rand_type : TYPE_3,
rand_deg : DEG_3,
rand_sep : SEP_3,
end_ptr : &randtbl[sizeof (randtbl) / sizeof (randtbl[0])]
};
```

`rand_type`默认为`TYPE_3`，故不是直接采用线性同余产生随机数；接着看下`state`，它指向了数组`randtbl`第一项索引，`randtbl`在此源文件中也有定义，是一个长度为31的数组（不含第一项）

```C
static int32_t randtbl[DEG_3 + 1] =
{
TYPE_3,
-1726662223, 379960547, 1735697613, 1040273694, 1313901226,
1627687941, -179304937, -2073333483, 1780058412, -1989503057,
-615974602, 344556628, 939512070, -1249116260, 1507946756,
-812545463, 154635395, 1388815473, -1926676823, 525320961,
-1009028674, 968117788, -123449607, 1284210865, 435012392,
-2017506339, -911064859, -370259173, 1132637927, 1398500161,
-205601318,
};
```

再根据宏定义即可推出`struct random_data`中各项成员的含义（也可以直接看注释）

`fptr`：相当于循环队列的`front`指针，初始位置为4

`rptr`：相当于循环队列的`rear`指针，初始位置为1

`endptr`：指向数组末尾，以指示队列的循环

接着看回算法，简单分析可知，大概是队头自加队尾值，将此值保存为结果，然后队头队尾统一后移一项，再将结果作为生成的随机数返回即可。

## `srand()`

此外，随机数的播种也是有必要了解的，查看源代码

```C
int
__srandom_r (seed, buf)
unsigned int seed;
struct random_data *buf;
{
int type;
int32_t *state;
long int i;
long int word;
int32_t *dst;
int kc;
if (buf == NULL)
goto fail;
type = buf->rand_type;
if ((unsigned int) type >= MAX_TYPES)
goto fail;
state = buf->state;
/* We must make sure the seed is not 0. Take arbitrarily 1 in this case. */
if (seed == 0)
seed = 1;
state[0] = seed;
if (type == TYPE_0)
goto done;
dst = state;
word = seed;
kc = buf->rand_deg;
for (i = 1; i < kc; ++i)
{
/* This does:
state[i] = (16807 * state[i - 1]) % 2147483647;
but avoids overflowing 31 bits. */
long int hi = word / 127773;
long int lo = word % 127773;
word = 16807 * lo - 2836 * hi;
if (word < 0)
word += 2147483647;
*++dst = word;
}
buf->fptr = &state[buf->rand_sep];
buf->rptr = &state[0];
kc *= 10;
while (--kc >= 0)
{
int32_t discard;
(void) __random_r (buf, &discard);
}
done:
return 0;
fail:
return -1;
}
```

即是根据传入的种子，经过一些算法扩展到整个`state`中，具体算法在注释中已经解释出来了，再调用`rand()`30次进行一定程度上的混淆。

大致就是如此，为了方便逆向使用脚本，我用`python`复刻了一遍代码，各位需要的自取

```python
state = [
-1726662223, 379960547, 1735697613, 1040273694, 1313901226, 1627687941,
-179304937, -2073333483, 1780058412, -1989503057, -615974602, 344556628,
939512070, -1249116260, 1507946756, -812545463, 154635395, 1388815473,
-1926676823, 525320961, -1009028674, 968117788, -123449607, 1284210865,
435012392, -2017506339, -911064859, -370259173, 1132637927, 1398500161,
-205601318
]
class unsafestate:
def __init__(self) -> None:
self.fptr = 3
self.rptr = 0
self.end_ptr = 31
buf = unsafestate()
def rand():
fptr = buf.fptr
rptr = buf.rptr
end_ptr = buf.end_ptr
state[fptr] += state[rptr]
val = state[fptr]
result = (val >> 1) & 0x7fffffff
fptr += 1
if fptr >= end_ptr:
fptr = 0
rptr += 1
else:
rptr += 1
if rptr >= end_ptr:
rptr = 0
buf.fptr = fptr
buf.rptr = rptr
return result & 0xffffffff
def srand(seed):
state[0] = seed
dst = 0
word = seed
kc = 31
for i in range(30):
hi = word // 127773
lo = word % 127773
word = 16807 * lo - 2836 * hi
if word < 0:
word += 214748367
dst += 1
state[dst] = word
buf.fptr = 3
buf.rptr = 0
kc *= 10
kc -= 1
while kc >= 0:
rand()
kc -= 1
```

相关源代码分享在附件里
