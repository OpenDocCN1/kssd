
比如练码为`87654+3210+12345+678`，第 3 组就是`12345`。

第 3 组累和：得到值=`255`=`FF`(H)

### 4.7 浮点计算值与第 1 组注册码的运算：

```
0048E61C     8B45 F0          mov eax,dword ptr ss:[ebp-10]       ; 第 1 组注册码

0048E61F     E8 F8B7F7FF      call dumped_.00409E1C

0048E624     DB7D 84         fstp tbyte ptr ss:[ebp-7C]

0048E627     9B              wait

0048E628     8B45 E4         mov eax,dword ptr ss:[ebp-1C]

0048E62B     E8 ECB7F7FF     call dumped_.00409E1C

0048E630     DB6D 84         fld tbyte ptr ss:[ebp-7C]

0048E633     DEE1            fsubrp st(1),st                       ; 浮点值与第 1 组注册码差值

0048E635     D81D 8CE84800   fcomp dword ptr ds:[48E88C]           ; 差值与 4000 比较

0048E63B     DFE0            fstsw ax

0048E63D     9E              sahf

0048E63E     0F87 53010000   ja dumped_.0048E797

0048E644     8B45 E4         mov eax,dword ptr ss:[ebp-1C]

0048E647     E8 D0B7F7FF     call dumped_.00409E1C

0048E64C     DBBD 78FFFFFF   fstp tbyte ptr ss:[ebp-88]

0048E652     9B              wait

0048E653     8B45 F0         mov eax,dword ptr ss:[ebp-10]

0048E656     E8 C1B7F7FF     call dumped_.00409E1C

0048E65B     DBAD 78FFFFFF   fld tbyte ptr ss:[ebp-88]

0048E661     DEE1            fsubrp st(1),st                       ; 浮点值与第 1 组注册码差值

0048E663     D81D 8CE84800   fcomp dword ptr ds:[48E88C]           ; 差值与 4000 比较

0048E669     DFE0            fstsw ax

0048E66B     9E              sahf

0048E66C     0F87 25010000   ja dumped_.0048E797
```

这两个`ja`不能跳，跳就完。

试了好几个方法，让`fsubrp st(1),st`的值=`0`，都不会跳。

反复调试发现：`fsubrp st(1),st`的值只要不超过`48E88C`的值=`4000`就能通过。

### 4.8 通过上面 2 个`ja`后，比较：

```
0048E672     8B45 E8         mov eax,dword ptr ss:[ebp-18]         ; 第 4 组注册码

0048E675     8B55 E0         mov edx,dword ptr ss:[ebp-20]         ; 第 3 组注册码累和 ASCII 码

0048E678     E8 4F5EF7FF     call dumped_.004044CC                 ; 比较

0048E67D     75 0B          jnz short dumped_.0048E68A             ; 核心跳转

0048E67F     C605 647F4900 0>mov byte ptr ds:[497F64],1

0048E686     C645 F7 01      mov byte ptr ss:[ebp-9],1

0048E68A     33C0            xor eax,eax
```

只要修改`0048E67D`处的跳转，这样就能爆破成功。

### 5. 算法小结：

注册码需要 3 个`＋`号格开得到 4 组注册码。

用户名与第 2 组注册码运算得到浮点值，浮点值要约等于第 1 组注册码。

第 3 组注册码累和＝第 4 组注册码。

好，让第 2 组注册码小点，设为`0`，第 3 组注册码如果为`12345`的话，第 4 组就要=`255`，这样只需要反推第 1 组注册码，假设练码为：`8+0+12345+255`

得到浮点值=`st(1)=8.5259734170344796160e+15`

即`8525973417034479.616`，

第一组注册码从`8525973417030480－8525973417038479`任意数都可以注册成功。

用户名：`cyto`

注册码：`8525973417034479+0+12345+255`

写入注册表：

```
HKLM\Software\SWIFTDOG\NetScream\Name   SUCCESS   "cyto"

HKLM\Software\SWIFTDOG\NetScream\Serial   SUCCESS   "8525973417034479+0+12345+255"
```

`4dd466d0e1e0cc123afbe504da2462ca`
