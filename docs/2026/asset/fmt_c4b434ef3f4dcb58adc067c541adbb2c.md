
- [环境搭建](https://bbs.kanxue.com/#%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA)
- [信息搜集](https://bbs.kanxue.com/#%E4%BF%A1%E6%81%AF%E6%90%9C%E9%9B%86)
- [前置知识](https://bbs.kanxue.com/#%E5%89%8D%E7%BD%AE%E7%9F%A5%E8%AF%86)
  - [Builtin中的实现](https://bbs.kanxue.com/#Builtin%E4%B8%AD%E7%9A%84%E5%AE%9E%E7%8E%B0)
  - [快速路径的分配流程](https://bbs.kanxue.com/#%E5%BF%AB%E9%80%9F%E8%B7%AF%E5%BE%84%E7%9A%84%E5%88%86%E9%85%8D%E6%B5%81%E7%A8%8B)
  - [慢速路径的分配流程](https://bbs.kanxue.com/#%E6%85%A2%E9%80%9F%E8%B7%AF%E5%BE%84%E7%9A%84%E5%88%86%E9%85%8D%E6%B5%81%E7%A8%8B)
  - [Maglev的简单介绍](https://bbs.kanxue.com/#Maglev%E7%9A%84%E7%AE%80%E5%8D%95%E4%BB%8B%E7%BB%8D)
- [漏洞分析](https://bbs.kanxue.com/#%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90)
- [漏洞利用](https://bbs.kanxue.com/#%E6%BC%8F%E6%B4%9E%E5%88%A9%E7%94%A8)
- [参考文章](https://bbs.kanxue.com/#%E5%8F%82%E8%80%83%E6%96%87%E7%AB%A0)

# 环境搭建

|  |  |
| --- | --- |
| 1 2 3 | `git checkout 5315f073233429c5f5c2c794594499debda307bd` `gclient sync -D` `python3 tools\dev\gm.py x64.release` |

# 信息搜集

issue链接：[1c0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3X3g2U0K9s2u0G2L8h3W2#2L8g2)9J5k6h3!0J5k6#2)9J5c8X3W2K6M7%4g2W2M7#2)9J5c8U0b7H3x3o6j5%4y4e0x3H3](https://bbs.kanxue.com/elink@caaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3X3g2U0K9s2u0G2L8h3W2#2L8g2)9J5k6h3!0J5k6#2)9J5c8X3W2K6M7%4g2W2M7#2)9J5c8U0b7H3x3o6j5%4y4e0x3H3)

[60aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0K9s2u0G2L8h3W2#2L8g2)9J5k6s2u0W2N6X3W2W2N6#2)9J5k6h3N6G2L8$3N6D9k6i4y4G2N6i4u0U0k6g2)9J5k6h3y4G2L8g2)9J5c8X3y4Q4x3V1k6$3z5q4)9J5c8Y4j5^5i4K6u0r3i4K6u0n7i4K6u0r3y4o6j5&6y4o6l9H3y4H3`.`.](https://bbs.kanxue.com/elink@c4bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0K9s2u0G2L8h3W2#2L8g2)9J5k6s2u0W2N6X3W2W2N6#2)9J5k6h3N6G2L8$3N6D9k6i4y4G2N6i4u0U0k6g2)9J5k6h3y4G2L8g2)9J5c8X3y4Q4x3V1k6$3z5q4)9J5c8Y4j5^5i4K6u0r3y4o6j5&6y4o6l9H3y4H3`.`.)

revision commit hash：[cccK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0K9s2u0G2L8h3W2#2L8g2)9J5k6h3N6G2L8$3N6D9k6i4y4G2N6i4u0U0k6g2)9J5k6h3y4G2L8g2)9J5c8Y4j5^5i4K6u0r3N6U0S2Q4x3V1k6Q4x3V1u0Q4x3V1k6W2k6o6V1K6j5X3g2X3y4$3q4T1y4K6R3$3k6o6f1K6y4U0N6U0x3X3q4W2y4K6R3^5x3U0V1J5x3X3x3J5x3$3q4S2x3r3g2V1j5e0j5@1](https://bbs.kanxue.com/elink@067K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0K9s2u0G2L8h3W2#2L8g2)9J5k6h3N6G2L8$3N6D9k6i4y4G2N6i4u0U0k6g2)9J5k6h3y4G2L8g2)9J5c8Y4j5^5i4K6u0r3N6U0S2Q4x3V1k6Q4x3V1u0Q4x3V1k6W2k6o6V1K6j5X3g2X3y4$3q4T1y4K6R3$3k6o6f1K6y4U0N6U0x3X3q4W2y4K6R3^5x3U0V1J5x3X3x3J5x3$3q4S2x3r3g2V1j5e0j5@1)

diff链接：

[d84K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0K9s2u0G2L8h3W2#2L8g2)9J5k6h3N6G2L8$3N6D9k6i4y4G2N6i4u0U0k6g2)9J5k6h3y4G2L8g2)9J5c8Y4j5^5i4K6u0r3N6U0S2Q4x3V1k6Q4x3V1u0Q4x3V1k6W2k6o6V1K6j5X3g2X3y4$3q4T1y4K6R3$3k6o6f1K6y4U0N6U0x3X3q4W2y4K6R3^5x3U0V1J5x3X3x3J5x3$3q4S2x3r3g2V1j5e0j5@1i4K6t1#2y4f1g2Q4x3U0f1J5x3g2)9J5c8R3`.`.](https://bbs.kanxue.com/elink@d91K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0K9s2u0G2L8h3W2#2L8g2)9J5k6h3N6G2L8$3N6D9k6i4y4G2N6i4u0U0k6g2)9J5k6h3y4G2L8g2)9J5c8Y4j5^5i4K6u0r3N6U0S2Q4x3V1k6Q4x3V1u0Q4x3V1k6W2k6o6V1K6j5X3g2X3y4$3q4T1y4K6R3$3k6o6f1K6y4U0N6U0x3X3q4W2y4K6R3^5x3U0V1J5x3X3x3J5x3$3q4S2x3r3g2V1j5e0j5@1i4K6t1#2y4f1g2Q4x3U0f1J5x3g2)9J5c8R3`.`.)

# 前置知识

## `Reflect.construct()`

函数原型

|  |  |
| --- | --- |
| 1 | `Reflect.construct(target, argumentsList[, newTarget])` |

### 参数

- [`target`](https://bbs.kanxue.com/elink@4c3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2E0L8%4A6A6L8r3I4S2i4K6u0W2L8%4u0Y4i4K6u0r3P5X3S2Q4x3X3c8o6e0W2)9J5c8X3c8G2j5%4y4Q4x3V1k6i4k6h3u0Q4x3V1k6v1j5i4k6S2f1$3y4J5K9i4m8@1i4K6u0r3f1X3g2X3k6i4u0W2L8X3y4W2i4K6u0r3c8$3I4G2j5X3q4D9i4K6g2X3e0$3u0B7k6h3y4@1M7#2)9J5c8W2u0W2k6X3I4W2j5%4c8Q4x3V1k6U0L8$3&6K6N6s2u0#2j5%4c8Q4x3U0y4@1j5i4u0Y4k6i4b7`](https://bbs.kanxue.com/elink@4c3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2E0L8%4A6A6L8r3I4S2i4K6u0W2L8%4u0Y4i4K6u0r3P5X3S2Q4x3X3c8o6e0W2)9J5c8X3c8G2j5%4y4Q4x3V1k6i4k6h3u0Q4x3V1k6v1j5i4k6S2f1$3y4J5K9i4m8@1i4K6u0r3f1X3g2X3k6i4u0W2L8X3y4W2i4K6u0r3c8$3I4G2j5X3q4D9i4K6g2X3e0$3u0B7k6h3y4@1M7#2)9J5c8W2u0W2k6X3I4W2j5%4c8Q4x3V1k6U0L8$3&6K6N6s2u0#2j5%4c8Q4x3U0y4@1j5i4u0Y4k6i4b7`)：被运行的目标构造函数。
- [`argumentsList`](https://bbs.kanxue.com/elink@78cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2E0L8%4A6A6L8r3I4S2i4K6u0W2L8%4u0Y4i4K6u0r3P5X3S2Q4x3X3c8o6e0W2)9J5c8X3c8G2j5%4y4Q4x3V1k6i4k6h3u0Q4x3V1k6v1j5i4k6S2f1$3y4J5K9i4m8@1i4K6u0r3f1X3g2X3k6i4u0W2L8X3y4W2i4K6u0r3c8$3I4G2j5X3q4D9i4K6g2X3e0$3u0B7k6h3y4@1M7#2)9J5c8W2u0W2k6X3I4W2j5%4c8Q4x3V1k6U0L8$3&6K6N6s2u0#2j5%4c8Q4x3U0y4S2M7X3N6#2L8h3g2F1N6s2y4D9K9i4y4@1)：类数组，目标构造函数调用时的参数。
- [`newTarget可选`](https://bbs.kanxue.com/elink@578K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2E0L8%4A6A6L8r3I4S2i4K6u0W2L8%4u0Y4i4K6u0r3P5X3S2Q4x3X3c8o6e0W2)9J5c8X3c8G2j5%4y4Q4x3V1k6i4k6h3u0Q4x3V1k6v1j5i4k6S2f1$3y4J5K9i4m8@1i4K6u0r3f1X3g2X3k6i4u0W2L8X3y4W2i4K6u0r3c8$3I4G2j5X3q4D9i4K6g2X3e0$3u0B7k6h3y4@1M7#2)9J5c8W2u0W2k6X3I4W2j5%4c8Q4x3V1k6U0L8$3&6K6N6s2u0#2j5%4c8Q4x3U0y4F1k6i4N6@1j5i4u0Y4k6i4b7`](https://bbs.kanxue.com/elink@578K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2E0L8%4A6A6L8r3I4S2i4K6u0W2L8%4u0Y4i4K6u0r3P5X3S2Q4x3X3c8o6e0W2)9J5c8X3c8G2j5%4y4Q4x3V1k6i4k6h3u0Q4x3V1k6v1j5i4k6S2f1$3y4J5K6R3$3u0B7k6h3y4@1M7#2)9J5c8W2u0W2k6X3I4W2j5%4c8Q4x3V1k6U0L8$3&6K6N6s2u0#2j5%4c8Q4x3U0y4F1k6i4N6@1j5i4u0Y4k6i4b7`)：作为新创建对象的原型对象的`constructor`属性，参考[`new.target`](https://bbs.kanxue.com/elink@dcbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2E0L8%4A6A6L8r3I4S2i4K6u0W2L8%4u0Y4i4K6u0r3P5X3S2Q4x3X3c8o6e0W2)9J5c8X3c8G2j5%4y4Q4x3V1k6i4k6h3u0Q4x3V1k6v1j5i4k6S2f1$3y4J5K6R3$3u0B7k6h3y4@1M7#2)9J5c8W2u0W2k6X3I4W2j5%4c8Q4x3V1k6U0L8$3&6K6N6s2u0#2j5%4c8Q4x3U0y4F1k6i4N6@1j5i4u0Y4k6i4b7`)操作符，默认值为`target`。

### 使用实例

```javascript
function OneClass() {
  this.name = "one";
}

function OtherClass() {
  this.name = "other";
}

// 创建一个对象：
var obj1 = Reflect.construct(OneClass, args, OtherClass);
console.log(obj1.name); // 'one'
console.log(obj1 instanceof OneClass); // false
console.log(obj1 instanceof OtherClass); // true
```

## Builtin中的实现

`FastNewObject`的实现代码，位于`src/builtins/builtins-constructor-gen.cc`。

首先获取上下文环境，其中包括`target`和`new_target`，接着定义一个`call_runtime`的`label`，如果快速路径分配失败，则会通过这个`label`跳转到慢速路径进行对象分配。

调用`FastNewObject`进入快速路径，如果快速路径分配失败，则会跳转到`BIND(&call_runtime)`，然后执行`Runtime::kNewObject`，进入慢速路径的分配。

```cpp
TF_BUILTIN(FastNewObject, ConstructorBuiltinsAssembler) {
  auto context = Parameter<Context>(Descriptor::kContext);
  auto target = Parameter<JSFunction>(Descriptor::kTarget);
  auto new_target = Parameter<JSReceiver>(Descriptor::kNewTarget);

  Label call_runtime(this);

  TNode<JSObject> result =
      FastNewObject(context, target, new_target, &call_runtime);
  Return(result);

  BIND(&call_runtime);
  TailCallRuntime(Runtime::kNewObject, context, target, new_target);
}
```

对于慢速路径的`runtime`函数可以在vscode里全局搜索`RUNTIME_FUNCTION(Runtime_xxxxx`，这里的`xxxxx`代表函数名。上方的`kNewObject`，那么就用这里的函数名，应该是`NewObject`，所以就可以搜索`RUNTIME_FUNCTION(Runtime_NewObject)`。

![](https://bbs.kanxue.com/upload/attach/202507/971428_RR7PY4HQVS44X3M.png)

## 快速路径的分配流程

这里是全部完整的代码，位于`src/builtins/builtins-constructor-gen.cc`。

|  |  |
| --- | --- |
