
Go语言的map使用Hash表作为底层实现，可以在 `$GOROOT/src/pkg/runtime/hashmap.goc` 找到它的实现，其中 `<a href="elink@4dbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1M7X3q4$3k6h3&6W2M7%4y4Q4x3X3g2E0k6g2)9J5c8X3N6G2L8r3q4F1k6#2)9J5c8Y4c8J5k6h3g2Q4x3V1k6J5N6h3&6@1K9h3#2W2i4K6u0W2K9r3#2S2M7l9`.`.">runtime.hmap</a>`是最核心的结构体，我们先来了解一下该结构体的内部字段：

```go
type hmap struct {
count int //map中键值对数量
flags uint8 //map当前是否处于写入状态登
B uint8 //2的B次幂表示当前map中桶的数量
noverflow uint16 //map中溢出桶的数量，当溢出桶太多时，map会进行等量扩容
hash0 uint32 //生成hash的随机数种子
buckets unsafe.Pointer //当前map对应的桶的指针
oldbuckets unsafe.Pointer //map扩容时指向旧桶的指针，当所有旧桶中的数据转移到新桶时，清空
nevacuate uintptr //扩容时，用于标记当前旧桶中小于nevacute的数据都已经转移到了新桶
extra \*mapextra //存储map的溢出桶
}
type mapextra struct {
overflow \*[]\*bmap
oldoverflow \*[]\*bmap
nextOverflow \*bmap
}
```

这个hash结构使用的是一个可扩展哈希的算法，由hash值mod当前hash表大小决定某一个值属于哪个桶，而hash表大小是2的指数，即上面结构体中的2^B。每次扩容，会增大到上次大小的两倍。结构体中有一个`buckets`和一个`oldbuckets`是用来实现增量扩容的。正常情况下直接使用`buckets`，而`oldbuckets`为空。如果当前哈希表正在扩容中，则`oldbuckets`不为空，并且`buckets`大小是`oldbuckets`大小的两倍。

具体的Bucket结构如下所示：

```go
type bmap struct {
tophash [8]uint8 //存储Hash值的高8位
data []byte //key value数据：key/key/key.../value/value/value...
overflow \*bmap //溢出bucket的地址
}
```

- `BUCKETSIZE`是用宏定义的8，每个bucket中存放最多8个key/value对, 如果多于8个，那么会申请一个新的bucket，并将它与之前的bucket链起来。

- 按key的类型采用相应的hash算法得到key的hash值。将hash值的低位当作`Hmap`结构体中`buckets`数组的index，找到key所在的bucket。将hash的高8位存储在了bucket的`tophash`中。**注意，这里高8位不是用来当作key/value在bucket内部的offset的，而是作为一个主键，在查找时对tophash数组的每一项进行顺序匹配的**。先比较hash值高位与bucket的`tophash[i]`是否相等，如果相等则再比较bucket的第i个的key与所给的key是否相等。如果相等，则返回其对应的value，反之，在overflow buckets中按照上述方法继续寻找。

- `data`区存放的是key—value数据，其中keys放在一起，values放在一起，如此存储是为了节省字节对齐带来的空间浪费。例如`map[int64]int8`。

- `overflow`指针指向的是下一个bucket，据此将所有冲突的键连接起来

### 逆向中map的常见函数

字典操作常见的会转换为如下函数

一般`fastrand`和`makemap`连用返回一个map，它为一个指针

读字典时使用`mapaccess1`和`mapaccess2`，写字典时会使用`mapassign`函数，它返回一个地址，将value写入该地址，另外还比较常见的是对字典进行遍历，会使用`mapiterinit`和`mapiternext`配合。

```go
func fastrand() uint32
func makemap(mapType \*byte, hint int, mapbuf \*any) (hmap map[any]any)
func mapaccess1(mapType \*byte, hmap map[any]any, key \*any) (val \*any)
func mapaccess2(mapType \*byte, hmap map[any]any, key \*any) (val \*any, pres bool)
func mapassign(mapType \*byte, hmap map[any]any, key \*any) (val \*any)
func mapiterinit(mapType \*byte, hmap map[any]any, hiter \*any)
func mapiternext(hiter \*any)
```

`mapaccess`和`mapassign`的第一个参数代表字典的类型，因此能很容易知道字典操作参数和返回值的类型。

### 赋值和访问

赋值

```go
func mapassign(t \*maptype, h \*hmap, key unsafe.Pointer) unsafe.Pointer{}
```

3个参数，第三个参数是key的指针，rsp + 16， 返回值是key对应的数据指针.

访问

```go
func mapaccess1(t \*maptype, h \*hmap, key unsafe.Pointer) unsafe.Pointer {}
```

和赋值同理，区别是这个不会为不存在的 key 创建 key pair.

```go
package main
import "fmt"
func main() {
countryCapitalMap := map[string]string{"France": "Paris", "Italy": "Rome", "Japan": "Tokyo", "India": "New delhi"}
fmt.Println("France首都是", countryCapitalMap ["France"])
}
```

![图片描述](upload/attach/202204/921830\_UP2N3PZ52W5QE2V.png)

比如上方在执行完

![图片描述](upload/attach/202204/921830\_T6QAW3ZKSX5E7C7.png)

之后，map buckets是存放了一个键值对的，可以过去查看v5(`h \*hmap`) ，0x000000C00014FE38来查看

![图片描述](upload/attach/202204/921830\_BRJSM4484E8YSA2.png)

可以看见`buckets` unsafe.Pointer为`byte_C00014FE68`

![图片描述](upload/attach/202204/921830\_29SZWRQMDKJ9Q26.png)

前面8字节的是`tophash`数组，目前只存放了一个键值对，就只有一个存放了，hash(”France”)的高8bit，往下就是存放键值对

```go
package main
import "fmt"
func main() {
var countryCapitalMap map[string]string /\*创建集合, 默认map是nil\*/
//如果不初始化 map，那么就会创建一个 nil map, nil map 不能用来存放键值对
countryCapitalMap = make(map[string]string)
countryCapitalMap [ "France" ] = "巴黎"
countryCapitalMap [ "Italy" ] = "罗马"
countryCapitalMap [ "Japan" ] = "东京"
countryCapitalMap [ "India " ] = "新德里"
//或者：countryCapitalMap := map[string]string{"France": "Paris", "Italy": "Rome", "Japan": "Tokyo", "India": "New delhi"}
/\*使用键输出地图值 \*/
for country := range countryCapitalMap {
fmt.Println(country, "首都是", countryCapitalMap [country])
}
}
```

```go
void \_\_cdecl main\_main()
{
int v0; // [rsp+10h] [rbp-230h]
\_\_int64 v1; // [rsp+20h] [rbp-220h]
\_\_int64 v2; // [rsp+20h] [rbp-220h]
\_\_int64 v3; // [rsp+20h] [rbp-220h]
\_\_int64 v4; // [rsp+20h] [rbp-220h]
\_QWORD \*v5; // [rsp+30h] [rbp-210h]
\_\_int64 \*v6; // [rsp+30h] [rbp-210h]
\_\_int64 v7; // [rsp+38h] [rbp-208h]
\_\_int64 v8; // [rsp+40h] [rbp-200h]
\_\_int64 v9; // [rsp+48h] [rbp-1F8h]
\_\_int64 v10; // [rsp+50h] [rbp-1F0h]
\_\_int64 v11; // [rsp+58h] [rbp-1E8h]
\_\_int64 v12; // [rsp+60h] [rbp-1E0h]
\_QWORD v13[5]; // [rsp+68h] [rbp-1D8h] BYREF
\_\_int64 v14; // [rsp+90h] [rbp-1B0h] BYREF
\_\_int128 v15; // [rsp+98h] [rbp-1A8h] BYREF
\_\_int128 v16; // [rsp+A8h] [rbp-198h]
\_\_int128 v17; // [rsp+B8h] [rbp-188h]
\_\_int64 v18[12]; // [rsp+C8h] [rbp-178h] BYREF
char v19; // [rsp+128h] [rbp-118h] BYREF
while ( (unsigned \_\_int64)&v14 <= \*(\_QWORD \*)(\*(\_QWORD \*)NtCurrentTeb()->NtTib.ArbitraryUserPointer + 16LL) )
runtime\_morestack\_noctxt();
v15 = 0LL;
v16 = 0LL;
v17 = 0LL;
((void (\*)(void))loc\_46651C)();
\*(\_QWORD \*)&v16 = &v19;
runtime\_fastrand();
HIDWORD(v15) = v0;
runtime\_mapassign\_faststr((\_\_int64)&unk\_4B7040, (\_\_int64)&v15, (\_\_int64)"France", 6LL);
v5[1] = 6LL;
if ( runtime\_writeBarrier )
runtime\_gcWriteBarrier();
else
\*v5 = "巴黎";
runtime\_mapassign\_faststr((\_\_int64)&unk\_4B7040, (\_\_int64)&v15, (\_\_int64)"Italy", 5LL);
v5[1] = 6LL;
if ( runtime\_writeBarrier )
runtime\_gcWriteBarrier();
else
\*v5 = "罗马";
runtime\_mapassign\_faststr((\_\_int64)&unk\_4B7040, (\_\_int64)&v15, (\_\_int64)"Japan", 5LL);
v5[1] = 6LL;
if ( runtime\_writeBarrier )
runtime\_gcWriteBarrier();
else
\*v5 = "东京";
v8 = runtime\_mapassign\_faststr((\_\_int64)&unk\_4B7040, (\_\_int64)&v15, (\_\_int64)"India ", 6LL);
v5[1] = 9LL;
if ( runtime\_writeBarrier )
runtime\_gcWriteBarrier();
else
\*v5 = "新德里";
((void (\*)(void))loc\_466551)();
runtime\_mapiterinit((\_\_int64)&unk\_4B7040, &v15, (\_\_int64)v18);
while ( v18[0] )
{
v11 = \*(\_QWORD \*)v18[0];
v10 = \*(\_QWORD \*)(v18[0] + 8);
runtime\_convTstring(\*(\_QWORD \*)v18[0], v10, v1);
v12 = v2;
v9 = runtime\_mapaccess1\_faststr((\_\_int64)&unk\_4B7040, (\_\_int64)&v15, v11, v10, (\_\_int64)v5);
v7 = runtime\_convTstring(\*v6, v6[1], v3);
v13[0] = &unk\_4B3260;
v13[1] = v12;
v13[2] = &unk\_4B3260;
v13[3] = &off\_4EF518;
v13[4] = &unk\_4B3260;
v14 = v4;
fmt\_Fprintln((\_\_int64)&go\_itab\_\_os\_File\_io\_Writer, os\_Stdout, (\_\_int64)v13, 3LL, 3LL, v7, v8, v9);
runtime\_mapiternext((\_\_int64)v18);
}
}
```

### 查找过程

1. 根据key计算出hash值。

2. 如果存在old table, 首先在old table中查找，如果找到的bucket已经evacuated，转到步骤3。 反之，返回其对应的value。

3. 在new table中查找对应的value。

这里一个细节需要注意一下。不认真看可能会以为低位用于定位bucket在数组的index，那么高位就是用于key/valule在bucket内部的offset。事实上高8位不是用作offset的，而是用于加快key的比较的。

```go
do { //对每个桶b
//依次比较桶内的每一项存放的tophash与所求的hash值高位是否相等
for(i = 0, k = b->data, v = k + h->keysize \* BUCKETSIZE; i < BUCKETSIZE; i++, k += h->keysize, v += h->valuesize) {
if(b->tophash[i] == top) {
k2 = IK(h, k);
t->key->alg->equal(&eq, t->key->size, key, k2);
if(eq) { //相等的情况下再去做key比较...
\*keyp = k2;
return IV(h, v);
}
}
}
b = b->overflow; //b设置为它的下一下溢出链
} while(b != nil);
```

### 插入过程

1. 根据key算出hash值，进而得出对应的bucket。

2. 如果bucket在old table中，将其重新散列到new table中。

3. 在bucket中，查找空闲的位置，如果已经存在需要插入的key，更新其对应的value。

4. 根据table中元素的个数，判断是否grow table。

5. 如果对应的bucket已经full，重新申请新的bucket作为overbucket。

6. 将key/value pair插入到bucket中。

这里也有几个细节需要注意一下。

在扩容过程中，oldbucket是被冻结的，查找时会在oldbucket中查找，但不会在oldbucket中插入数据。如果在oldbucket是找到了相应的key，做法是将它迁移到新bucket后加入evalucated标记。并且还会额外的迁移另一个pair。

然后就是只要在某个bucket中找到第一个空位，就会将key/value插入到这个位置。也就是位置位于bucket前面的会覆盖后面的(类似于存储系统设计中做删除时的常用的技巧之一，直接用新数据追加方式写，新版本数据覆盖老版本数据)。找到了相同的key或者找到第一个空位就可以结束遍历了。不过这也意味着做删除时必须完全的遍历bucket所有溢出链，将所有的相同key数据都删除。所以目前map的设计是为插入而优化的，删除效率会比插入低一些。

### 删除过程

删除元素实际上也是先查找元素，如果元素存在则把元素从相应的bucket中删除，如果不存在则什么也不做。

**notice**
