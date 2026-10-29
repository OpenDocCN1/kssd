
## `filled-new-array/range` 指令

`filled-new-array/range {vCCCC .. vNNNN}, type@BBBB`：`filled-new-array/range`指令与`filled-new-array`指令非常类似，只是它的参数寄存器是以范围形式给出的，`vCCCC`和`vNNNN`分别表示参数寄存器的起始位置和结束位置，`type@BBBB`表示数组元素的类型。`vC`是`filled-new-array/range`指令的第一个参数寄存器，表示数组大小。`N=A+C-1`表示参数寄存器的结束位置，其中`A`表示数组大小，`C`表示第一个参数寄存器的编号。因为`filled-new-array/range`指令的参数寄存器是连续的一段，所以`N`的值等于`A+C-1`，即参数寄存器的结束位置。例如，如果`filled-new-array/range {v2 .. v5}, type@BBBB`指令被执行，那么数组大小为`v2`中存储的整数值，参数寄存器的编号为`v2`、`v3`、`v4`、`v5`，因此`N=A+C-1=4+v2-1=3+v2`。

## `fill-array-data` 指令

`fill-array-data vAA, 偏移量`：用指定的字面量数组数据填充到目标数组中，字面量数组数据的位址是当前指令位址加偏移量的和。该指令填充的数组类型必须为基本类型的数组和字符串数组，其中基本类型包括`boolean`、`byte`、`short`、`char`、`int`、`float`和`double`。填充数组时，字面量数组数据的每个元素都必须与目标数组的元素类型相同。另外，该指令只能用于填充已经创建的目标数组，不能用于创建新的数组对象。

举个例子，假设有如下代码：

```
...
fill-array-data v0, :array_0 // 将字面量数组数据填充到 v0 寄存器所代表的数组中，:array_0为字面量数组数据的偏移地址(偏移量)
...
:array_0
.array-data 2 // 表示每个数组元素占用两个字节
0x14ebs // 数组元素值为 0x14eb（short 类型）
0x15f0s // 数组元素值为 0x15f0（short 类型）
0x15c7s // 数组元素值为 0x15c7（short 类型）
0x15c7s // 数组元素值为 0x15c7（short 类型）
0x15c7s // 数组元素值为 0x15c7（short 类型）
0x15c7s // 数组元素值为 0x15c7（short 类型）
0x15c7s // 数组元素值为 0x15c7（short 类型）
0x15c7s // 数组元素值为 0x15c7（short 类型）
...
```

## 异常指令

### `throw` 指令

`throw vAA`：抛出异常对象，异常对象的引用在`vAA`寄存器。

## 跳转指令

### 无条件跳转指令

`goto 目标`：通过短偏移量无条件跳转到目标。

### 分支跳转指令

#### `packed-switch` 指令

`packed-switch vAA, 索引表偏移量`：实现一个`switch`语句，`case`常量是连续的，这个指令使用索引表，`vAA`是在表中找到具体`case`的指令偏移量的索引，如果无法在表中找到`vAA`对应的索引将继续执行下一个指令（即`default case`），执行完`case`中的代码后，如果`Java`源代码设置了`break`强制跳出`switch`，就会无条件跳转到所有`case`的出口处，如果没有强制跳出`switch`，就会执行完`default case`中的代码后再跳转到所有`case`的出口处继续执行代码。

举个例子，假设有如下代码：

Java代码：

```Java
switch (this.mType) {
case 1:
return Math.signum(0.5d - (getP(d) % 1.0d));
case 2:
abs = Math.abs((((getP(d) * 4.0d) + 1.0d) % 4.0d) - 2.0d);
break;
case 3:
return (((getP(d) * 2.0d) + 1.0d) % 2.0d) - 1.0d;
case 4:
abs = ((getP(d) * 2.0d) + 1.0d) % 2.0d;
break;
case 5:
return Math.cos(this.PI2 * getP(d));
case 6:
double abs2 = 1.0d - Math.abs(((getP(d) * 4.0d) % 4.0d) - 2.0d);
abs = abs2 * abs2;
break;
default:
return Math.sin(this.PI2 * getP(d));
}
```

smali代码：

```
packed-switch v0, :pswitch_data_5e # 带着v0寄存器中的值跳转到索引表偏移量:pswitch_data_5e位址寻找对应的case指令偏移量的索引
.line 120
# default分支语句
iget-wide v0, p0, Landroidx/constraintlayout/motion/utils/Oscillator;->PI2:D
invoke-virtual {p0, p1, p2}, Landroidx/constraintlayout/motion/utils/Oscillator;->getP(D)D
move-result-wide p1
mul-double/2addr v0, p1
invoke-static {v0, v1}, Ljava/lang/Math;->sin(D)D
move-result-wide p1
return-wide p1
.line 132
:pswitch_17 # case 6
invoke-virtual {p0, p1, p2}, Landroidx/constraintlayout/motion/utils/Oscillator;->getP(D)D
move-result-wide p1
mul-double/2addr p1, v1
rem-double/2addr p1, v1
sub-double/2addr p1, v3
invoke-static {p1, p2}, Ljava/lang/Math;->abs(D)D
move-result-wide p1
sub-double p1, v5, p1
mul-double/2addr p1, p1
:goto_25 # 所有case的出口，也可以说是在case中执行break后跳转到此处
sub-double/2addr v5, p1
return-wide v5
.line 130
:pswitch_27 # case 5
iget-wide v0, p0, Landroidx/constraintlayout/motion/utils/Oscillator;->PI2:D
invoke-virtual {p0, p1, p2}, Landroidx/constraintlayout/motion/utils/Oscillator;->getP(D)D
move-result-wide p1
mul-double/2addr v0, p1
invoke-static {v0, v1}, Ljava/lang/Math;->cos(D)D
move-result-wide p1
return-wide p1
.line 128
:pswitch_33 # case 4
invoke-virtual {p0, p1, p2}, Landroidx/constraintlayout/motion/utils/Oscillator;->getP(D)D
move-result-wide p1
mul-double/2addr p1, v3
add-double/2addr p1, v5
rem-double/2addr p1, v3
goto :goto_25 # 跳转到偏移量为goto_25的目标位址
.line 126
:pswitch_3b # case 3
invoke-virtual {p0, p1, p2}, Landroidx/constraintlayout/motion/utils/Oscillator;->getP(D)D
move-result-wide p1
mul-double/2addr p1, v3
add-double/2addr p1, v5
rem-double/2addr p1, v3
sub-double/2addr p1, v5
return-wide p1
.line 124
:pswitch_44 # case2
invoke-virtual {p0, p1, p2}, Landroidx/constraintlayout/motion/utils/Oscillator;->getP(D)D
move-result-wide p1
mul-double/2addr p1, v1
add-double/2addr p1, v5
rem-double/2addr p1, v1
sub-double/2addr p1, v3
invoke-static {p1, p2}, Ljava/lang/Math;->abs(D)D
move-result-wide p1
goto :goto_25 # 跳转到偏移量为goto_25的目标位址
:pswitch_51 # case 1
const-wide/high16 v0, 0x3fe0000000000000L # 0.5
.line 122
invoke-virtual {p0, p1, p2}, Landroidx/constraintlayout/motion/utils/Oscillator;->getP(D)D
move-result-wide p1
rem-double/2addr p1, v5
sub-double/2addr v0, p1
invoke-static {v0, v1}, Ljava/lang/Math;->signum(D)D
move-result-wide p1
return-wide p1
:pswitch_data_5e
.packed-switch 0x1 # 索引表，case常量从1开始，依次递增
:pswitch_51 # case 1
:pswitch_44 # case 2
:pswitch_3b # case 3
:pswitch_33 # case 4
:pswitch_27 # case 5
:pswitch_17 # case 6
.end packed-switch
```

#### `sparse-switch` 指令

`sparse-switch vAA, 查询表偏移量`：实现一个`switch`语句，`case`常量是非连续的，这个指令使用查询表，用于表示`case`常量和每个`case`常量的偏移量。如果`vAA`寄存器中的值无法在表中匹配将继续执行下一个指令（即`default case`）。执行完`case`中的代码后，如果`Java`源代码设置了`break`强制跳出`switch`，就会无条件跳转到所有`case`的出口处，如果没有强制跳出`switch`，就会执行完`default case`中的代码后再跳转到所有`case`的出口处继续执行代码。

举个例子，假设有如下代码：

Java代码：

```java
switch (transit) {
case 4097:
return 8194;
case 4099:
return 4099;
case 8194:
return 4097;
default:
return 0;
}
```

smali代码：

```
sparse-switch p0, :sswitch_data_e # sparse-switch指令，传入寄存器p0的值作为查找表的索引，:sswitch_data_e是查询表的偏移量
.line 2025 # 源代码的行号
:goto_4 # 这里是所有case的出口，如果case常量没匹配上查询表中的值，那么v0寄存器中的值就为0，这里就返回零，也就是default下的代码return 0;
return v0 # 返回v0寄存器中的值
.line 2016 # 源代码的行号
:sswitch_5 # 标签，对应表中的第一个case常量0x1001
const/16 v0, 0x2002 # 将0x2002赋值给v0寄存器
.line 2017 # 源代码的行号
goto :goto_4 # 无条件跳转到:goto_4标签处执行
.line 2019 # 源代码的行号
:sswitch_8 # 标签，对应表中的第三个case常量0x2002
const/16 v0, 0x1001 # 将0x1001赋值给v0寄存器
.line 2020 # 源代码的行号
goto :goto_4 # 无条件跳转到:goto_4标签处执行
.line 2022 # 源代码的行号
:sswitch_b # 标签，对应表中的第二个case常量0x1003
const/16 v0, 0x1003 # 将0x1003赋值给v0寄存器
goto :goto_4 # 无条件跳转到:goto_4标签处执行
.line 2014 # 源代码的行号
:sswitch_data_e # 标签，作为查询表的偏移量
.sparse-switch # 表示接下来是一个查询表
0x1001 -> :sswitch_5 # case常量0x1001跳转到标签:sswitch_5处执行
0x1003 -> :sswitch_b # case常量0x1003跳转到标签:sswitch_b处执行
0x2002 -> :sswitch_8 # case常量0x2002跳转到标签:sswitch_8处执行
.end sparse-switch # 标志着查询表的结束
```

### 条件跳转指令

`if-eq vA, vB, 目标`：如果`vA == vB`，跳转到目标。

`if-ne vA, vB, 目标`：如果`vA != vB`，跳转到目标。

`if-lt vA, vB, 目标`：如果`vA < vB`，跳转到目标。

`if-ge vA, vB, 目标`：如果`vA >= vB`，跳转到目标。

`if-gt vA, vB, 目标`：如果`vA > vB`，跳转到目标。

`if-le vA, vB, 目标`：如果`vA <= vB`，跳转到目标。

`if-eqz vA, 目标`：如果`vA == 0`，跳转到目标。

`if-nez vA, 目标`：如果`vA != 0`，跳转到目标。

`if-ltz vA, 目标`：如果`vA < 0`，跳转到目标。

`if-gez vA, 目标`：如果`vA >= 0`，跳转到目标。

`if-gtz vA, 目标`：如果`vA > 0`，跳转到目标。

`if-lez vA, 目标`：如果`vA <= 0`，跳转到目标。

## 比较指令

`cmpl-float vAA,vBB,vCC`：比较`vBB`和`vCC`的`float`值并在`vAA`存入`int`型返回值。非数值默认为小于，如果参数为非数值将返回`-1`。如果`vBB`寄存器大于`vCC`寄存器，则结果为`-1`，相等则结果为`0`，小于则结果为`1`。

`cmpg-float vAA,vBB,vCC`：比较`vBB`和`vCC`的`float`值并在`vAA`存入`int`型返回值。非数值默认为大于，如果参数为非数值将返回`1`。如果`vBB`寄存器大于`vCC`寄存器，则结果为`1`，相等则结果为`0`，小于则结果为`-1`。

`cmpl-double vAA,vBB,vCC`：比较`vBB`和`vCC`的`double`值并在`vAA`存入`int`型返回值。非数值默认为小于，如果参数为非数值将返回`-1`。如果`vBB,vBB+1`寄存器的值大于`vCC,vCC+1`寄存器的值，则结果为`-1`，相等则结果为`0`，小于则结果为`1`。

`cmpg-double vAA,vBB,vCC`：比较`vBB`和`vCC`的`double`值并在`vAA`存入`int`型返回值。非数值默认为大于，如果参数为非数值将返回`1`。如果`vBB,vBB+1`寄存器的值大于`vCC,vCC+1`寄存器的值，则结果为`1`，相等则结果为`0`，小于则结果为`-1`。

`cmp-long vAA,vBB,vCC`：比较`vBB`和`vCC`的`long`值并在`vAA`存入`int`型返回值。如果`vBB`寄存器大于`vCC`寄存器，则结果为`1`，相等则结果为`0`小则结果为`-1`。

## 字段操作指令

### `iput-object` 指令

`iput-object vAA,vBB,字段ID`：根据字段ID将`vAA`寄存器的值存入实例的对象引用字段，`vBB`寄存器中是该实例的引用。例如：

```
iput-object v0, p0, Lbin/mt/apksignaturekillerplus/HookApplication;->appPkgName:Ljava/lang/String;
```

可以看到例子中将`v0`寄存器中`Ljava/lang/String;`类型的值（实例的对象引用）存入到`p0`寄存器中，`p0`寄存器中是`Lbin/mt/apksignaturekillerplus/HookApplication;`类型下字段名为`appPkgName`的实例字段，并且实例字段`appPkgName`的存储值类型为`Ljava/lang/String;`

### `iput-boolean` 指令

`iput-boolean vAA,字段ID`：根据字段ID将`vAA`寄存器的值存入实例的`boolean`型字段，`vBB`寄存器中是该实例的引用。例如：

```
iput-boolean p1, p0, Lbin/mt/plugin/api/translation/TranslationEngine$Configuration;->acceptTranslated:Z
```

可以看到例子中将`p1`寄存器中`boolean`型的值存入到`p0`寄存器中，`p0`寄存器中是`Lbin/mt/plugin/api/translation/TranslationEngine$Configuration;`类型下字段名为`acceptTranslated`的实例字段，并且实例字段`acceptTranslated`的存储值类型为`boolean`。

### `iput-wide` 指令

`iput-wide vAA,vBB,字段ID`：根据字段ID将`vAA,vAA+1`寄存器的值存入实例的`double/long`型字段，`vBB`寄存器中是该实例的引用。例如：

```
iput-wide v1, p0, Lcom/alipay/android/phone/mrpc/core/l;->e:J
```

可以看到例子中将`v1,v2`寄存器中`long`型的值存入到`p0`寄存器中，`p0`寄存器中是`Lcom/alipay/android/phone/mrpc/core/l;`类型下字段名为`e`的实例字段，并且实例字段`e`的存储值类型为`long`。
