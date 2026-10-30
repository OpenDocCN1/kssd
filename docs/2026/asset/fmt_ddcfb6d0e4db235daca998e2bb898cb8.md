
```shell
>>> b\*m0
matrix([[ 6, 7, 0, 1, 2, 3, 4, 5, 8, 9, 10, 11, 34, 35, 36, 15, 14,
17, 18, 19, 20, 21, 12, 13, 22, 23, 16, 27, 28, 29, 30, 31, 32, 33,
24, 25, 26, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47]])
>>> b\*m0\*m0
matrix([[ 4, 5, 6, 7, 0, 1, 2, 3, 8, 9, 10, 11, 24, 25, 26, 15, 36,
17, 18, 19, 20, 21, 34, 35, 12, 13, 14, 27, 28, 29, 30, 31, 32, 33,
22, 23, 16, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47]])
>>> b\*m0\*m0\*m0
matrix([[ 2, 3, 4, 5, 6, 7, 0, 1, 8, 9, 10, 11, 22, 23, 16, 15, 26,
17, 18, 19, 20, 21, 24, 25, 34, 35, 36, 27, 28, 29, 30, 31, 32, 33,
12, 13, 14, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47]])
>>> b\*m0\*m0\*m0\*m0
matrix([[ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16,
17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33,
34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47]])
```

对其它组变换矩阵进行了测试，同样如此。

也就是说

```python
m[i]\*m[i]\*m[i]\*m[i] = I
```

那就好办了。

运行记录第一次的变换操作

```
00408516 COND: index = 00000004
0040851C COND: count = 00000003
00408516 COND: index = 00000004
0040851C COND: count = 00000002
00408516 COND: index = 00000003
0040851C COND: count = 00000002
00408516 COND: index = 00000005
0040851C COND: count = 00000003
00408516 COND: index = 00000004
0040851C COND: count = 00000003
00408516 COND: index = 00000001
0040851C COND: count = 00000002
00408516 COND: index = 00000001
0040851C COND: count = 00000003
00408516 COND: index = 00000000
0040851C COND: count = 00000001
00408516 COND: index = 00000004
0040851C COND: count = 00000001
00408516 COND: index = 00000002
0040851C COND: count = 00000002
00408516 COND: index = 00000001
0040851C COND: count = 00000002
00408516 COND: index = 00000000
0040851C COND: count = 00000002
00408516 COND: index = 00000005
0040851C COND: count = 00000002
00408516 COND: index = 00000005
0040851C COND: count = 00000003
00408516 COND: index = 00000001
0040851C COND: count = 00000002
00408516 COND: index = 00000001
0040851C COND: count = 00000002
00408516 COND: index = 00000000
0040851C COND: count = 00000002
00408516 COND: index = 00000004
0040851C COND: count = 00000003
00408516 COND: index = 00000000
0040851C COND: count = 00000002
00408516 COND: index = 00000000
0040851C COND: count = 00000002
00408516 COND: index = 00000003
0040851C COND: count = 00000003
00408516 COND: index = 00000005
0040851C COND: count = 00000003
00408516 COND: index = 00000002
0040851C COND: count = 00000003
00408516 COND: index = 00000001
0040851C COND: count = 00000002
```

整理下，列表中的前面数值表示变换矩阵序号，后面数值表示变换次数：

```python
lop1 = [[4,3],[4,2],[3,2],[5,3],[4,3],[1,2],[1,3],[0,1],[4,1],
[2,2],[1,2],[0,2],[5,2],[5,3],[1,2],[1,2],[0,2],[4,3],
[0,2],[0,2],[3,3],[5,3],[2,3],[1,2]]
```

看着这个就能直接写出逆变换操作：

```python
lop2 = [[1,2],[2,1],[5,1],[3,1],[4,1],[0,2],[0,2],[4,1],[0,2],[1,2],
[1,2],[5,1],[5,2],[0,2],[1,2],[2,2],[4,3],[0,3],[1,1],
[1,2],[4,1],[5,1],[3,2],[4,2],[4,1]]
```

计算出来的输入位数太多。细看操作列表，其中有好一些是可以合并操作的，进行合并，得新的操作列表：

```python
lop3 = [[1,2],[2,1],[5,1],[3,1],[4,1],[0,2],[5,3],[0,2],[1,2],
[2,2],[4,3],[0,3],[1,3],[4,1],[5,1],[3,2],[4,3]]
```

出来结果校验直接通过，出乎意料，唯一结果。

校验二的代码现在是实在不想看，开始似乎看到了ecc，也是大数，如果有时间再补充吧。

最后给出部分代码：

```python
import string
t18 = '0123456789ABCDEFGH'
t62 = string.digits+string.uppercase+string.lowercase
lop1 = [[4,3],[4,2],[3,2],[5,3],[4,3],[1,2],[1,3],[0,1],[4,1],
[2,2],[1,2],[0,2],[5,2],[5,3],[1,2],[1,2],[0,2],[4,3],
[0,2],[0,2],[3,3],[5,3],[2,3],[1,2]]
lop2 = [[1,2],[2,1],[5,1],[3,1],[4,1],[0,2],[5,3],[0,2],[1,2],[2,2],[4,3],[0,3],[1,3],[4,1],[5,1],[3,2],[4,3]]
#l=[[1,2],[2,1],[5,1],[3,1],[0,2],[0,2],[4,1],[0,2],[1,2],[1,2],[5,1],[5,2],[0,2],[1,2],[2,2],[4,3],[0,3],[1,1],[1,2],[4,1],[5,1],[3,2],[4,2],[4,1]]
#1,2 2,1 5,1 3,1 4,1 0,2 5,3 0,2 1,2 2,2 4,3 0,3 1,3 4,1 5,1 3,2 4,3
def t18to62(snum):
sum=0
for i in snum:
sum = sum\*18+t18.index(i)
s = ''
while sum:
s += t62[sum%62]
sum = sum/62
return s
def t62to18(snum):
sum=0
for i in snum:
sum = sum\*62+t62.index(i)
s = ''
while sum:
s += t18[sum%18]
sum = sum/18
return s
if \_\_name\_\_ == '\_\_main\_\_':
sn1 = ''
for [i,j] in lop2:
sn1 += t18[i\*3+j-1]
print t18to62(sn1[::-1])
```

# 看雪.TSRC 2017CTF秋季赛第五题WP

说来惭愧，这题只能写写我的做法，好多我也没有分析出来，看得好心烦。

## 查找重点流程

看图标就知道MFC写的窗体程序。拖入IDA，入口都不想去翻，直接查看字符串，好多。稍微翻下能看到`KanXueCrackMe2017`字样和一串类似base64的字串，追到`sub_406FC3`，像是验证算法。

再往上追，到`sub_4071FD`，这时我比较确信这是主流程，而下层那个应该就是算法，看伪代码：

```
signed int __thiscall sub_4071FD(_DWORD *this)

{

  _DWORD *v1; // esi

  signed int result; // eax



  v1 = this;

  CWnd::UpdateData(1);

  result = sub_406FC3(v1);

  switch ( result )

  {

    case 1:

      return CWnd::MessageBoxW(v1, (LPCWSTR)&unk_44BE10, &Caption, 0);// 成功

    case 0:

      return CWnd::MessageBoxW(v1, (LPCWSTR)&unk_44BE18, &Caption, 0);// 失败

    case 2:

      result = CWnd::MessageBoxW(v1, &Text, &Caption, 0);// 很遗憾，离成功还差一小步@￥@

      break;

  }

  return result;

}
```

主流程根据算法流程的返回不同弹出三个不同的弹窗。

## 算法流程

直接贴上算法流程的伪代码，菜鸡如我，只会f5。

```
signed int __thiscall sub_406FC3(_DWORD *this)

{

  _DWORD *v1; // ebx

  _WORD *v2; // eax

  int v3; // esi

  int i; // ecx

  size_t v5; // kr00_4

  signed int result; // eax

  signed int j; // [esp+24h] [ebp-D9F8h]

  int v9[48]; // [esp+28h] [ebp-D9F4h]

  char v11; // [esp+D9A8h] [ebp-74h]

  char v12; // [esp+D9D0h] [ebp-4Ch]

  __int16 v13; // [esp+D9E2h] [ebp-3Ah]

  char v14[28]; // [esp+D9E4h] [ebp-38h]

  CPPEH_RECORD ms_exc; // [esp+DA04h] [ebp-18h]



  v1 = this;

  sub_407307(v9);

  strcpy(&v12, "KanXueCrackMe2017");

  v13 = 0;

  sub_4084A3((int)v9, &v12);

  v2 = (_WORD *)v1[38];

  v3 = *((_DWORD *)v2 - 3);

  for ( i = 0; i < v3; ++v2 )

  {

    if ( i < 0 || i > v3 )

      ATL::AtlThrowImpl(0x80070057);

    v14[i++] = *v2;

  }

  v14[i] = 0;

  __asm { int     2Dh; Windows NT - debugging services: eax = type }

  ms_exc.registration.TryLevel = 0;

  v5 = strlen(v14);

  for ( j = 0; j < 1000; ++j )

  {

    sub_4031E6((int)&v11, (int)v14, v5);       //base64

    memcpy_0(v14, &v11, v5);

  }

  if ( !strcmp("Vm0wd2QyUXlVWGw==", &v11) )

  {

    result = 1;

    ms_exc.registration.TryLevel = -2;

  }

  else

  {

    ms_exc.registration.TryLevel = -2;

    result = 0;

  }

  return result;

}
```

先不说具体算法，先看流程。

似乎最后`strcmp("Vm0wd2QyUXlVWGw==", &v11)`这比较成功，就行了。

但再看中间的`__asm { int 2Dh; Windows NT - debugging services: eax = type }`这段。显示的调试坑啊。为了静态方便看正确的流程，我patch下代码，至于动态，网上说加了插件的OD直接过，我是把`int 2D`改成`int3`，再设置pass `int3`过的，不然过不了，哎。

```
signed int __thiscall sub_406FC3(_DWORD *this)

{

  _DWORD *v1; // ebx

  _WORD *v2; // eax

  int v3; // esi

  int i; // ecx

  signed int v5; // eax

  signed int result; // eax

  int v7[48]; // [esp+28h] [ebp-D9F4h]

  int v8[13872]; // [esp+E8h] [ebp-D934h]

  char v9; // [esp+D9D0h] [ebp-4Ch]

  __int16 v10; // [esp+D9E2h] [ebp-3Ah]

  char v11[28]; // [esp+D9E4h] [ebp-38h]

  CPPEH_RECORD ms_exc; // [esp+DA04h] [ebp-18h]



  v1 = this;

  sub_407307(v7);

  strcpy(&v9, "KanXueCrackMe2017");

  v10 = 0;

  sub_4084A3((int)v7, &v9);

  v2 = (_WORD *)v1[38];

  v3 = *((_DWORD *)v2 - 3);

  for ( i = 0; i < v3; ++v2 )

  {

    if ( i < 0 || i > v3 )

      ATL::AtlThrowImpl(0x80070057);

    v11[i++] = *v2;

  }

  v11[i] = 0;

  ms_exc.registration.TryLevel = 1;

  sub_4084A3((int)v7, v11);

  v5 = 0;

  do

  {

    if ( v7[v5] != v8[v5] )

    {

      result = 0;

      goto LABEL_7;

    }

    ++v5;

  }

  while ( v5 < 48 );

  if ( sub_40680C(v11) == 1 && strlen(v11) == 12 )

  {

    result = 1;

LABEL_7:

    ms_exc.registration.TryLevel = -2;

    return result;

  }

  return 2;

}
```

还是先看流程。校验一共有两处：一是`v7`与`v8`的48字节比较，不正确就返回`0`；二是`v11`长度是否为`12`及`sub_40680C(v11)`是否返回`1`，若不是则返回2，是则返回1，即校验成功。

那`v7`、`v8`及`v11`又是什么呢。下面开始理算法。

## 算法分析OR猜测

通过动静结合分析，`v7`、`v8`为`sub_407307`初始化的48字节数组，其16进制表示为：

```
00  01  02  03  04  05  06  07  08  09  0A  0B  0C  0D  0E  0F  

10  11  12  13  14  15  16  17  18  19  1A  1B  1C  1D  1E  1F  

20  21  22  23  24  25  26  27  28  29  2A  2B  2C  2D  2E  2F
```

`v8`后由`sub_4084A3`函数两次（实际上内部是多次）修改。而`v11`则是输入。

这里我们得到第一个比较明确的条件：
