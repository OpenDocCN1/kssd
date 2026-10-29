
# 反射构造器获取与使用

```Java
package com.java.day6\_reflect;
import org.junit.Test;
import java.lang.reflect.Constructor;
public class ReflectConstructors {
// @Test注释是JUnit框架中用于标记测试方法的注释。通过使用@Test注释，JUnit可以识别并执行被标记的方法作为测试用例。
// 在使用@Test注释之前，需要引入JUnit框架。可以通过在项目中添加JUnit依赖来实现。
// 被@Test修饰的方法必须是公共的、方法必须不能带有参数、方法必须没有返回值。
// 被@Test注释标记的方法不能抛出任何异常。如果测试方法需要测试某个异常情况，可以使用@Test注释的expected参数来指定期望的异常类型。
@Test
public void test() {
// 第一种方式：获取全部构造器(只能获取public修饰的)
Class<?> a = Zoon.class; // 想要完成反射第二步、第三步，那都需要得到类的Class对象
// 调用类的getConstructors方法，可以得到类的全部构造器。但是这个方法会把类的每一个构造器都封装成构造器对象，然后把所有构造器对象放到一个构造器数组里面去。
Constructor[] b = a.getConstructors();
// 验证：遍历构造器数组中的每一个构造器对象，并打印构造器名称和参数个数
for (Constructor constructor : b) {
System.out.println("这是第一种方式获得的构造器对象："+constructor.getName()+"，这个构造器有"+constructor.getParameterCount()+"个参数！");
}
}
@Test
public void test1() {
// 第二种方式：获取全部构造器(只要存在就能拿到)
Class<?> a = Zoon.class;
Constructor[] c = a.getDeclaredConstructors();
// 验证：遍历构造器数组中的每一个构造器对象，并打印构造器名称和参数个数
for (Constructor constructor : c) {
System.out.println("这是第二种方式获得的构造器对象："+constructor.getName()+"，这个构造器有"+constructor.getParameterCount()+"个参数！");
}
}
@Test
public void test2() throws Exception{
// 第三种方式：获取某一个构造器(只能获取public修饰的)
Class<?> a = Zoon.class;
// 格式：Constructor 变量 = 类的class.getConstructor(Class<?>... parameterTypes);
// 获取有参构造器
// 获取有参构造器需要往getConstructor方法中填写对应有参构造器的参数类型来匹配有参构造器
Constructor e = a.getConstructor(String.class, int.class);
System.out.println("这是第三种方式获得的有参构造器对象："+e.getName()+"，这个构造器有"+e.getParameterCount()+"个参数！");
Constructor d = a.getConstructor(); // 获取无参构造器
System.out.println("这是第三种方式获得的无参构造器对象："+d.getName());
}
@Test
public void test3() throws Exception{
// 第四种方式：获取某一个构造器(只要存在就能拿到)
Class<?> a = Zoon.class;
// 格式：Constructor 变量 = 类的class.getDeclaredConstructor(Class<?>... parameterTypes);
Constructor d = a.getDeclaredConstructor(); // 获取无参构造器
System.out.println("这是第四种方式获得的无参构造器对象："+d.getName());
// 获取有参构造器
// 获取有参构造器需要往getDeclaredConstructor方法中填写对应有参构造器的参数类型来匹配有参构造器
Constructor e = a.getDeclaredConstructor(String.class, int.class);
System.out.println("这是第四种方式获得的有参构造器对象："+e.getName()+"，这个构造器有"+e.getParameterCount()+"个参数！");
}
}
```

我们获取到了构造器自然要使用，而反射获取构造器的作用主要是初始化对象并返回。要怎么使用呢？我简单的讲解一下，我先把`Zoon`类的有参数构造器从`public`修饰变为`private`修饰，然后如下使用反射获取的构造器：

```Java
@Test
public void test4() throws Exception{
Class<?> a = Zoon.class;
Constructor f = a.getDeclaredConstructor(String.class, int.class);
f.setAccessible(true); // 将构造器对象方法setAccessible的flag参数设置为true，表示禁止检查访问控制(暴力反射)。简而言之：就算是修饰为私有的，我也能在外面访问。
// 通过newInstance构造器对象方法调用此构造器对象表示的构造器，并传入参数，完成对象的初始化并返回
Zoon f2 = (Zoon) f.newInstance("长颈鹿", 3); // 因为我们在获取类的构造器的时候，接收构造器对象的变量在接收时并不知道这个构造器是哪个类的构造器，导致构造器对象返回的类型还是Object类型，所以需要在这里强转类型。
}
```

现在反射第二步也算是完成了。

## 反射学习第三步：获取类的成员变量和成员方法

获取类的成员变量和成员方法与获取类的构造器类似，我就简单讲讲。

我先把`Zoon`类的成员变量`age`和成员方法`setName`从`public`修饰的变为了`private`修饰的。

### 获取并使用类的成员变量

```Java
package com.java.day6\_reflect;
import org.junit.Test;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
public class ReflectField {
@Test
public void fields1() throws Exception{
// 第一种方式：获取类的全部成员变量(只能获取public修饰的)
Class<?> a = Zoon.class;
// 调用类的getFields方法，可以得到类的全部成员变量。但是这个方法会把类的每一个成员变量都封装成成员变量对象，然后把所有成员变量对象放到一个成员变量数组中去，只是只能获取public修饰的成员变量。
Field[] b = a.getFields();
// 验证：遍历成员变量数组中的每一个成员变量对象，并打印成员变量名称和成员变量类型
for (Field field : b) {
System.out.println("这是第一种方式获得的成员变量对象" + "成员变量名称：" + field.getName() + "，" + "成员变量类型：" + field.getType());
}
}
@Test
public void fields2() throws Exception{
// 第二种方式：获取类的全部成员变量(只要存在就能拿到)
Class<?> a = Zoon.class;
// 调用类的getDeclaredFields方法，可以得到类的全部成员变量。这个方法也会把类的每一个成员变量都封装成成员变量对象，然后把所有成员变量对象放到一个成员变量数组中去，但是只要存在就能拿到。
Field[] b = a.getDeclaredFields();
// 验证：遍历成员变量数组中的每一个成员变量对象，并打印成员变量名称和成员变量类型
for (Field field : b) {
System.out.println("这是第二种方式获得的成员变量对象" + "成员变量名称：" + field.getName() + "，" + "成员变量类型：" + field.getType());
}
}
@Test
public void fields3() throws Exception{
// 第三种方式：获取类的某个成员变量(只能获取public修饰的)
Class<?> a = Zoon.class;
// 调用类的getField方法，可以得到类的某个成员变量。这个方法想要获取到某个成员变量，就需要告诉它要获取的成员变量的名字，只是只能获取public修饰的成员变量。
Field b = a.getField("name");
// 验证：打印获取到的某个成员变量的成员变量名称和成员变量类型
System.out.println("这是第三种方式获得的成员变量对象" + "成员变量名称：" + b.getName() + "，" + "成员变量类型：" + b.getType());
}
@Test
public void fields4() throws Exception {
// 第四种方式：获取类的某个成员变量(只要存在就能拿到)
Class<?> a = Zoon.class;
// 调用类的getDeclaredField方法，可以得到类的某个成员变量。这个方法想要获取到某个成员变量，就需要告诉它要获取的成员变量的名字，但是只要存在就能拿到。
Field b = a.getDeclaredField("name");
Field c = a.getDeclaredField("age");
// 验证：打印获取到的某个成员变量的成员变量名称和成员变量类型
System.out.println("这是第四种方式获得的成员变量对象" + "成员变量名称：" + b.getName() + "，" + "成员变量类型：" + b.getType());
System.out.println("这是第四种方式获得的成员变量对象" + "成员变量名称：" + c.getName() + "，" + "成员变量类型：" + c.getType());
}
@Test
public void fields5() throws Exception{
// 使用类的成员变量
Class<?> a = Zoon.class;
Field[] b = a.getDeclaredFields();
Zoon c = new Zoon();
for (Field field : b) {
// 在使用Field类来设置值或获取值时，都需要先获取到该成员变量的对象，然后通过Field对象来进行操作。
// 而且不管是设置值、还是获取值，都需要告诉Field对象需要对哪个对象的这个成员变量进行操作。
if (field.getName().equals("age")) {
field.setAccessible(true); // 禁止检查访问控制(暴力反射)
field.set(c, 5); // Field类.set(需要设置值的成员变量所属对象, 想要设置的值)
}else {
field.set(c, "老虎");
}
System.out.println(field.get(c));
}
}
}
```

打印结果：

![](upload/attach/202308/926486\_H3J8Q5JU3G3GBW8.png)

### 获取类的成员方法并执行

```Java
package com.java.day6\_reflect;
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
c.invoke(d, "熊猫"); // Method类.invoke(需要执行的成员方法所属对象, 执行成员方法所需参数)
System.out.println(b);
}
}
```

打印结果：

![](upload/attach/202308/926486\_GFWJJF8ZY9A42PU.png)

到这我们就将反射全部讲解完毕，我们已经充分认识了什么是反射，以及反射的核心作用是用来获取类的各个组成部分并执行他们。

## 二、smali基础语法

### dalvik字节码

1、先来了解一下dalvik虚拟机：
