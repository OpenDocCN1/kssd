
```
package com.java.day6_reflect;

import org.junit.Test;

import java.lang.reflect.Method;

public class ReflectMethod {

    @Test
    public void methods1() throws Exception{
        // 第一种方式：获取类的全部成员方法(只能获取public修饰的)
        Class<?> a = Zoon.class;
        // 调用类的getMethods方法，可以得到类的全部成员方法。但是这个方法会把类的每一个成员方法都封装成成员方法对象，然后把所有成员方法对象放到一个成员方法数组中去，只是只能获取public修饰的成员方法。
        Method[] b = a.getMethods();
        // 验证：遍历成员方法数组中的每一个成员方法对象，并打印成员方法名称和成员方法参数个数以及成员方法返回值类型
        for (Method method : b) {
            System.out.println("这是第一种方式获得的成员方法对象" + "成员方法名称：" + method.getName() + "，"
                    + "成员方法参数个数：" + method.getParameterCount() + "，"
                    + "成员方法返回值类型：" + method.getReturnType());
        }
    }

    @Test
    public void methods2() throws Exception{
        // 第二种方式：获取类的全部成员方法(只要存在就能拿到)
        Class<?> a = Zoon.class;
        // 调用类的getDeclaredMethods方法，可以得到类的全部成员方法。这个方法也会把类的每一个成员方法都封装成成员方法对象，然后把所有成员方法对象放到一个成员方法数组中去，但是只要存在就能拿到。
        Method[] b = a.getDeclaredMethods();
        // 验证：遍历成员方法数组中的每一个成员方法对象，并打印成员方法名称和成员方法参数个数以及成员方法返回值类型
        for (Method method : b) {
            System.out.println("这是第二种方式获得的成员方法对象" + "成员方法名称：" + method.getName() + "，"
                    + "成员方法参数个数：" + method.getParameterCount() + "，"
                    + "成员方法返回值类型：" + method.getReturnType());
        }
    }

    @Test
    public void methods3() throws Exception{
        // 第三种方式：获取类的某个成员方法(只能获取public修饰的)
        Class<?> a = Zoon.class;
        // 调用类的getMethod方法，可以得到类的某个成员方法。getMethod方法想要获取到某个成员方法，就需要告诉它要获取的成员方法的名字以及参数类型，只是只能获取public修饰的成员方法。
        Method b = a.getMethod("getName");
        // 验证：打印获取到的某个成员方法的成员方法名称和成员方法参数个数以及成员方法返回值类型
        System.out.println("这是第三种方式获得的成员方法对象" + "成员方法名称：" + b.getName() + "，"
                + "成员方法参数个数：" + b.getParameterCount() + "，"
                + "成员方法返回值类型：" + b.getReturnType());
    }

    @Test
    public void methods4() throws Exception {
        // 第四种方式：获取类的某个成员方法(只要存在就能拿到)
        Class<?> a = Zoon.class;
        // 调用类的getDeclaredMethod方法，可以得到类的某个成员方法。getDeclaredMethod方法想要获取到某个成员方法，就需要告诉它要获取的成员方法的名字以及参数类型，但是只要存在就能拿到。
        Method b = a.getDeclaredMethod("setName", String.class);
        // 验证：打印获取到的某个成员方法的成员方法名称和成员方法参数个数以及成员方法返回值类型
        System.out.println("这是第四种方式获得的成员方法对象" + "成员方法名称：" + b.getName() + "，"
                + "成员方法参数个数：" + b.getParameterCount() + "，"
                + "成员方法返回值类型：" + b.getReturnType());
    }

    @Test
    public void methods5() throws Exception{
        // 触发类的成员方法执行
        Class<?> a = Zoon.class;
        // 在使用Method类来触发类的成员方法执行时，需要先获取到该成员方法的对象，然后通过Method对象来进行操作。
        // 而且想要触发某个类的成员方法执行都需要告诉Method对象需要对哪个对象的这个成员方法进行操作。
        Method b = a.getDeclaredMethod("getName");
        Method c = a.getDeclaredMethod("setName", String.class);
        Zoon d = new Zoon();
        c.setAccessible(true); // 禁止检查访问控制(暴力反射)
        c.invoke(d, "熊猫");  // Method类.invoke(需要执行的成员方法所属对象, 执行成员方法所需参数)
        System.out.println(b);
    }

}
```

打印结果：

![](https://bbs.kanxue.com/upload/attach/202308/926486_GFWJJF8ZY9A42PU.png)

到这我们就将反射全部讲解完毕，我们已经充分认识了什么是反射，以及反射的核心作用是用来获取类的各个组成部分并执行他们。

## 二、smali基础语法

#### dalvik字节码

1、先来了解一下dalvik虚拟机：

dalvik虚拟机是Android 5.0以前用于运行安卓应用的虚拟机，从 Android 4.4 开始，Google 开始引入了全新的虚拟机 ART（Android Runtime），直到Android5.0开始ART虚拟机就替代了dalvik虚拟机。既然dalvik虚拟机被ART虚拟机替代了，那我们还有学的必要吗？ART 是向下兼容的，ART虚拟机对DEX字节码的运行是兼容的，因此针对 Dalvik 开发的应用也能在 ART 环境中运作。但是Dalvik 采用的一些技术并不适用于 ART，但不妨碍我们了解和学习dalvik字节码。

2、我们再来了解一下dalvik寄存器和寄存器的命名方法：

Dalvik寄存器中的寄存器都是32位大小，支持所有类型，对于小于或等于32位的类型，使用一个寄存器就可以了；对于64位(`long`和`double`)类型，需要使用两个相邻的寄存器来存储。

寄存器的命名方法有两种：v命名法和p命名法：

​ v命名法：局部变量寄存器用v开头数字结尾的符号来表示，如`v0`、 `v1`、`v2`。

​ p命名法：函数参数寄存器则使用p开头数字结尾的符号来表示，如`p0`、`p1`、`p2`。

特别注意的是，`p0`不一定是函数中的第一个参数，在非`static`函数中，`p0`代指“this"，`p1`表示函数的第一个 参数，`p2`代表函数中的第二个参数。而在`static`函数中`p0`才对应第一个参数(因为Java的`static`方法中没有`this`方法）

寄存器赋值：

|  |  |
| --- | --- |
| 1 2 3 | `const/4 p5, 0x1 //p5赋值1`    `const/16 v0, 0xa //v0赋值10，在16进制里a表示10` |

`const/4`和`const/16`是表示定义一个4位和16位值，这里只是修饰一下，记住Dalvik寄存器是32位寄存器，这只是表示32位寄存器中存的值是4位和16位而已，不要被这个给影响了。

3、再来了解一下dalvik字节码的类型与Java数据类型的对应关系：

数据类型对应：

| smali类型 | java类型 | 注释 |
| --- | --- | --- |
| `V` | `void` | 无返回值 |
| `Z` | `boolean` | 布尔值类型，返回0或1 |
| `B` | `byte` | 字节类型，返回字节 |
| `S` | `short` | 短整数类型，返回数字 |
| `C` | `char` | 字符类型，返回字符 |
| `I` | `int` | 整数类型，返回数字 |
| `J` | `long` （64位 需要2个寄存器存储） | 长整数类型，返回数字 |
| `F` | `float` | 单浮点类型，返回数字 |
| `D` | `double` （64位 需要2个寄存器存储） | 双浮点类型，返回数字 |
| `string` | `String` | 文本类型，返回字符串 |
| `Lxxx/xxx/xxx` | `object` | 对象类型，返回对象 |

`long`类型和`double`类型都是64位，需要两个寄存器来存储，有时在smali代码中，原本从0到n的寄存器在中间少了一个，那就有可能是其中有数据是`long`类型或者`double`类型的，在赋值代码中虽然用到了两个寄存器，但隐藏了其中一个。

4、了解smali和Java之间的转换流程：

`xxx.java` ==> `xxx.class` ==> 使用`dx`工具将`.class`打包成`.dex`文件 ==> 使用`baksmali`工具将`.dex`文件反编译成`.smali`文件 ==> 反编译成smali文件后对该文件进行修改成功 ==> 使用`samli`工具将`.smali`文件打包成`.dex`文件(回编译)

5、了解dalvik方法：

```
//一个私有、静态、不可变的方法   方法名

.method private static final onCreate$lambda-2([Ljava/lang/String;)Z//(这里面是方法的参数)这里是方法返回值类型，表示布尔值类型，返回假或真

```

`.method`表示定义了一个方法；

`private`、`static`、`final`表示该方法是一个一个私有、静态、不可变的方法；

`onCreate$lambda-2`是该方法定义的方法名，方法名后面括号中是该方法的参数；

`[Ljava/lang/String;`该参数表示接收字符串类型的一维数组，其中`[`表示一维数组，在smali的方法参数中每一个`[`就表示一维数组，比如`[[`就表示二维数组；java中类是一种数据类型，类用`L`表示，`L`后面接的是完整类名，也就是包名和类名。

6、了解dalvik字段：

dalvik创建字段需要指定字段的访问标志(修饰符)、字段名、字段类型和初始值等信息。下面是一个示例字段的创建代码：

`.field private static count:I = 0`

上述代码创建了一个名为`count`的私有静态字段，类型为整型，初始值为0。

#### dalvik指令集

1、dalvik指令集格式：

基础字节码-名称后缀/字节码后缀 目的寄存器 源寄存器/常量

- Dalvik指令集中参数采用从源寄存器到目标寄存器的方式。

- 根据dalvik字节码的大小与类型不同，可能会添加名称后缀以消除岐义。具体来说，字节码的类型可以分为常规类型和特殊类型，根据类型的不同添加的后缀也不同，常规类型的字节码不添加任何后缀，而特殊类型的字节码根据具体类型添加后缀，可能是以下几种后缀之一：

  - `-boolean`：布尔类型
  - `-byte`：字节类型
  - `-char`：字符类型
  - `-short`：短整型
  - `-int`：整型
  - `-long`：长整型
  - `-float`：浮点型
  - `-double`：双精度浮点型
  - `-object`：对象类型
  - `-string`：字符串类型
  - `-class`：类类型
  - `-void`：无返回值类型

- 常规类型的字节码主要有以下几种：

  - `move`：将一个寄存器的值复制到另一个寄存器中
  - `add`：将两个寄存器的值相加，并将结果存储到目标寄存器中
  - `sub`：将两个寄存器的值相减，并将结果存储到目标寄存器中
  - `mul`：将两个寄存器的值相乘，并将结果存储到目标寄存器中
  - `div`：将两个寄存器的值相除，并将结果存储到目标寄存器中
  - `rem`：将两个寄存器的值取余，并将结果存储到目标寄存器中
  - `neg`：将一个寄存器的值取反，并将结果存储到目标寄存器中
  - `and`：将两个寄存器的值进行按位与操作，并将结果存储到目标寄存器中
  - `or`：将两个寄存器的值进行按位或操作，并将结果存储到目标寄存器中
  - `xor`：将两个寄存器的值进行按位异或操作，并将结果存储到目标寄存器中
  - `shl`：将一个寄存器的值左移指定位数，并将结果存储到目标寄存器中
  - `shr`：将一个寄存器的值右移指定位数，并将结果存储到目标寄存器中
  - `ushr`：将一个寄存器的值无符号右移指定位数，并将结果存储到目标寄存器中
  - `not`：将一个寄存器的值进行按位取反操作，并将结果存储到目标寄存器中
  - `cmp`：比较两个寄存器的值，并将结果存储到目标寄存器中

- 此外，根据字节码的大小与布局的不同，也可能添加字节码后缀以消除岐义。这些后缀通过在字节码主名称后添加斜杠“/”来分隔开。例如，一些指令可能会添加以下后缀：

  - `-wide`：64位宽度
  - `-from16`：源为16位寄存器引用
  - `-to16`：目标为16位寄存器引用
  - `-range`：指示该指令的操作数是一个范围。

- 32位常规类型的字节码没有添加任何后缀。

- 64位常规类型的字节码添加 `-wide`后缀。

- 例如这条指令：`move-wide/from16 vAA, vBBBB`：

  - 这条指令是移动命令，用于将源寄存器中存储的数据移动到目标寄存器中。
  - 其中`wide`表示数据宽度为64位，`from16`表示源寄存器为一个16位寄存器引用，`vAA`表示目标寄存器，它的取值范围是v0v65535。
  - `Dalvik`指令集中的大多数指令都需要使用寄存器作为源操作数或目的操作数。其中，`A`/`B`/`C`/`D`/`E`/`F`/`G`/`H`表示一个4位数值，可以表示0v15的寄存器。`AA`/`BB`/`CC`/`DD`/`EE`/`FF`/`GG`/`HH`表示一个8位数值，可以表示0v255的寄存器。`AAAA`/`BBBB`/`CCCC`/`DDDD`/`EEEE`/`FFFF`/`GGGG`/`HHHH`表示一个16位数值，可以表示0v65535的寄存器。

2、dalvik指令集的使用：

- 空操作指令

  - 空操作指令的助记符为`nop`。它的值为`00`，通常`nop`指令被用来作对齐代码之用，无实际操作，就和空格差不多。

- 数据操作指令

  - `move`指令的三种作用：

    - 第一种作用：进行赋值操作
    - 第二种作用：`move-result`接收方法返回值操作
    - 第三种作用：处理异常的操作

  - `move`指令集的使用：

    - `move vx, vy` 指令将寄存器`vy`中的值移动到寄存器`vx`中。例如，`move v1, v2`表示将寄存器`v2`中的值移动到寄存器`v1`中，源寄存器与目的寄存器都为4位。

    - `move/from16 vx, vy` 指令将16位寄存器`vy`中的值移动到寄存器`vx`中。例如，`move/from16 v1, v2`表示将16位寄存器`v2`中的值移动到寄存器`v1`中，源寄存器为16位，目的寄存器为8位(范围必须在256以内)。注意，寄存器中存储的是值的引用，也可以理解为存储的是值的地址。

    - `move-wide/from16 vx, vy` 指令用于移动`long`/`double`类型的数据，将其从16位寄存器`vy`移动到范围必须在256内(8位)的寄存器`vx`中。例如，`move-wide/from16 v1, v2`表示将16位寄存器`v2`中的`long`/`double`类型的值移动到8位寄存器`v1`中。注意，寄存器中存储的是值的引用，也可以理解为存储的是值的地址。

    - `move/16 vx, vy` 指令将寄存器`vy`的值移动到寄存器`vx`中。例如，`move/16 v1,v2`表示将16位寄存器`v2`中的值移动到16位寄存器`v1`中，源寄存器与目的寄存器都为16位。

    - `move-object vx, vy` 指令将一个对象引用从寄存器`vy`移动到寄存器`vx`中。例如，`move-object v1, v2`表示将对象引用从寄存器`v2`移动到寄存器`v1`中。

    - `move-object/from16 vx, vy` 指令将对象引用从16位寄存器`vy`移动到寄存器`vx`中。例如，`move-object/from16 v1, v2`表示将对象引用从16位寄存器`v2`移动到寄存器`v1`中。
