
# Dalvik 指令集

## 移动结果指令

- `move-result vx` 操作将上一条指令的返回值存储到指令中的目标寄存器`vx`中。例如，`move-result v1`表示将上一条指令的返回值移动到寄存器`v1`中。

- `move-result-wide vx` 操作与`move-result`类似，但是用于保存长整数或双精度浮点数类型的返回值。例如，`move-result-wide v1`表示将长整数或双精度浮点数类型的返回值移动到寄存器`v1`中。

- `move-result-object vx` 操作与`move-result`类似，但是用于保存引用类型的返回值。例如，`move-result-object v1`表示将引用类型的返回值移动到寄存器`v1`中。

- `move-exception vx` 操作用于捕获异常对象并将其移动到指定寄存器`vx`中。例如，`move-exception v1`表示将异常对象移动到寄存器`v1`中。

## 返回指令（重点）

- `return-void`：用于表示函数从一个`void`方法中返回，返回值为空。例如，`return-void`表示从当前方法中返回值为空。

- `return vx`：用于返回一个整数类型的值，该值存储在寄存器`vx`中。例如，`return v1`表示从当前方法中返回寄存器`v1`中存储的整数类型的值。

- `return-wide vx`：用于返回一个长整数或双精度浮点数类型的值，该值存储在寄存器`vx`和`vx+1`中。例如，`return-wide v1`表示从当前方法中返回寄存器`v1`和`v2`中存储的长整数或双精度浮点数类型的值。

- `return-object vx`：用于返回一个对象引用类型的值，该值存储在寄存器`vx`中。例如，`return-object v1`表示从当前方法中返回寄存器`v1`中存储的对象引用类型的值。

- `return-boolean vx`：用于返回一个布尔类型的值，该值存储在寄存器`vx`中。例如，`return-boolean v1`表示从当前方法中返回寄存器`v1`中存储的布尔类型的值。

- `return-byte vx`：用于返回一个字节类型的值，该值存储在寄存器`vx`中。例如，`return-byte v1`表示从当前方法中返回寄存器`v1`中存储的字节类型的值。

- `return-char vx`：用于返回一个字符类型的值，该值存储在寄存器`vx`中。例如，`return-char v1`表示从当前方法中返回寄存器`v1`中存储的字符类型的值。

- `return-short vx`：用于返回一个短整数类型的值，该值存储在寄存器`vx`中。例如，`return-short v1`表示从当前方法中返回寄存器`v1`中存储的短整数类型的值。

## 数据定义指令（重点）

- `const/4 vA,#+B`：用于将一个4位的数值符号扩展为32后赋值给寄存器`vA`。其中，`vA`表示目标寄存器，`#+B`表示常量值，取值范围为-8到7。

- `const/16 vAA, #+BBBB`：用于将一个16位的数据符号扩展为32位后赋给寄存器`vAA`。其中，`vAA`表示目标寄存器，`#+BBBB`表示常量值，取值范围为-32768到32767。

- `const vAA, #+BBBBBBBB`：用于将一个32位的数值赋给寄存器`vAA`。其中，`vAA`表示目标寄存器，`#+BBBBBBBB`表示常量值，取值范围为-2147483648到2147483647。

- `const/high16 vAA, #+BBBB0000`：用于将一个高16位的数值以低16位为零的方式扩展为32位后赋给寄存器`vAA`。其中，`vAA`表示目标寄存器，`#+BBBB0000`表示常量值的高16位。

- `const-wide/16 vAA,#+BBBB`：用于将16位的数值符号扩展为64位后赋值给寄存器`vAA`和`vAA+1`。其中，`vAA`和`vAA+1`表示目标寄存器（因为`dalvik`寄存器是32位的，所以存储64位数据需要两个寄存器存储），`#+BBBB`表示常量值，取值范围为-32768到32767。

- `const-wide/32 vAA, #+BBBBBBBB`：用于将32位的数值符号扩展为64位后赋值给寄存器`vAA`和`vAA+1`。其中，`vAA`和`vAA+1`表示目标寄存器，`#+BBBBBBBB`表示常量值。

- `const-wide vAA, #+BBBBBBBBBBBBBBBB`：用于将64位的数值赋给寄存器`vAA`和`vAA+1`。其中，`vAA`和`vAA+1`表示目标寄存器，`#+BBBBBBBBBBBBBBBB`表示常量值。

- `const-wide/high16 vAA, #+BBBB000000000000`：用于将一个高16位的数值以其他位为零的方式扩展为64位后赋给寄存器`vAA`和`vAA+1`。其中，`vAA`和`vAA+1`表示目标寄存器，`#+BBBB000000000000`表示常量值的高16位。

- `const-string vAA,string@BBBB`：用于将字符串常量引用存储到寄存器`vAA`中，通过字符串ID或字符串。其中，`vAA`表示目标寄存器，`string@BBBB`表示字符串在常量池中的索引。

- `const-string/jumbo vAA, string@BBBBBBBB`：与`const-string`类似，但用于存储较长的字符串常量引用。

- `const-class vAA,type@BBBB`：用于将类对象常量存储到寄存器`vAA`中，通过类型ID或类型（如`Object.class`）。其中，`vAA`表示目标寄存器，`type@BBBB`表示类在常量池中的索引。

- `const-class/jumbo vAAAA, type@BBBBBBBB`：与`const-class`类似，但`const-class/jumbo`指令占用两个字节，值为`0xooff`，并且其操作数比常规的`const-class`指令操作数更大。这是Android4.0中新增的指令。

## 实例操作指令

- `new-instance vAA, type@BBBB`：根据类型或者类型ID新建一个对象实例，并将新建的对象的引用存入目标寄存器`vAA`中。

- `instance-of vA, vB, type@CCCC`：检查`vB`寄存器中的对象引用是否是`type@CCCC`对应类型的实例，如果是`vA`寄存器存入非零值，否则`vA`寄存器存入零。

- `check-cast vAA, type@BBBB`：检查`vAA`寄存器中的对象引用是否可以转换成`type@BBBB`对应类型的实例。如不可转换，抛出`ClassCastException`异常，否则继续执行。另外还有一点需要注意，该指令只能用于转换成引用类型，不能用于转换成基本类型。

- `instance-of/jumbo vAAAA, vBBBB, type@CCCCCCCC`：指令功能与`instance-of vA, vB, type@CCCC`相同，只是寄存器值与指令的索引取值范围更大。

- `new-instance/jumbo vAAAA, type@BBBBBBBB`：指令功能与`new-instance vAA, type@BBBB`相同，只是寄存器值与指令的索引取值范围更大。

## 数组操作指令

- `array-length vA, vB`：计算`vB`寄存器中数组引用的元素长度并将长度存入`vA`寄存器。

- `new-array vA, vB, type@CCCC`：根据指定类型或类型ID（`type@CCCC`）与大小（`vB`寄存器存入数组的长度）构造一个数组，并将数组的引用赋给`vA`寄存器。

- `filled-new-array {vC, vD, vE, vF, vG}, type@BBBB`：根据指定类型或类型ID（`type@BBBB`）与大小（`vA`）构造一个数组并填充数组内容，`vA`寄存器是隐含使用的，除了指定数组的大小外还指定了参数的个数，`vC`~`vG`是使用到的参数寄存器序号。指令会将寄存器`vC`到`vG`中的值填充到新数组中，并将新数组引用保存到寄存器`vA`中。还有一点需要注意，`filled-new-array`指令创建的数组是一个新的对象，它与原数组没有任何关系。另外，该指令只能用于创建引用类型的数组，不能用于创建基本类型的数组。

- `filled-new-array/range {vCCCC .. vNNNN}, type@BBBB`：`filled-new-array/range`指令与`filled-new-array`指令非常类似，只是它的参数寄存器是以范围形式给出的，`vCCCC`和`vNNNN`分别表示参数寄存器的起始位置和结束位置，`type@BBBB`表示数组元素的类型。`vC`是`filled-new-array/range`指令的第一个参数寄存器，表示数组大小。`N=A+C-1`表示参数寄存器的结束位置，其中`A`表示数组大小，`C`表示第一个参数寄存器的编号。因为`filled-new-array/range`指令的参数寄存器是连续的一段，所以`N`的值等于`A+C-1`，即参数寄存器的结束位置。例如，如果`filled-new-array/range {v2 .. v5}, type@BBBB`指令被执行，那么数组大小为`v2`中存储的整数值，参数寄存器的编号为`v2`、`v3`、`v4`、`v5`，因此`N=A+C-1=4+v2-1=3+v2`。

- `fill-array-data vAA, 偏移量`：用指定的字面量数组数据填充到目标数组中，字面量数组数据的位址是当前指令位址加偏移量的和。该指令填充的数组类型必须为基本类型的数组和字符串数组，其中基本类型包括`boolean`、`byte`、`short`、`char`、`int`、`float`和`double`。填充数组时，字面量数组数据的每个元素都必须与目标数组的元素类型相同。另外，该指令只能用于填充已经创建的目标数组，不能用于创建新的数组对象。

  举个例子，假设有如下代码：

  ```
  ...
  fill-array-data v0, :array_0   // 将字面量数组数据填充到 v0 寄存器所代表的数组中，:array_0 为字面量数组数据的偏移地址(偏移量)
  ...
  :array_0
  .array-data 2                  // 表示每个数组元素占用两个字节
  0x14ebs                        // 数组元素值为 0x14eb（short 类型）
  0x15f0s                        // 数组元素值为 0x15f0（short 类型）
  0x15c7s                        // 数组元素值为 0x15c7（short 类型）
  0x15c7s                        // 数组元素值为 0x15c7（short 类型）
  0x15c7s                        // 数组元素值为 0x15c7（short 类型）
  0x15c7s                        // 数组元素值为 0x15c7（short 类型）
  0x15c7s                        // 数组元素值为 0x15c7（short 类型）
  0x15c7s                        // 数组元素值为 0x15c7（short 类型）
  ...
  ```

## 异常指令

- `throw vAA`：抛出异常对象，异常对象的引用在`vAA`寄存器。

## 跳转指令（重点）

### 无条件跳转指令

- `goto 目标`：通过短偏移量无条件跳转到目标。

### 分支跳转指令

- `packed-switch vAA, 索引表偏移量`：实现一个`switch`语句，`case`常量是连续的，这个指令使用索引表，`vAA`是在表中找到具体`case`的指令偏移量的索引，如果无法在表中找到`vAA`对应的索引将继续执行下一个指令（即`default case`），执行完`case`中的代码后，如果Java源代码设置了`break`强制跳出`switch`，就会无条件跳转到所有`case`的出口处，如果没有强制跳出`switch`，就会执行完`default case`中的代码后再跳转到所有`case`的出口处继续执行代码。

  举个例子，假设有如下代码：

  Java代码：

  ```java
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
