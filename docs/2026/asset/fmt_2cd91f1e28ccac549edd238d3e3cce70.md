
# [原创] CVE-2023-4069：Maglev图建立阶段的一个漏洞

## 目录

- [环境搭建](#环境搭建)
- [信息搜集](#信息搜集)
- [前置知识](#前置知识)
- [Builtin中的实现](#Builtin中的实现)
- [快速路径的分配流程](#快速路径的分配流程)
- [慢速路径的分配流程](#慢速路径分配流程)
- [Maglev的简单介绍](#Maglev的简单介绍)
- [漏洞分析](#漏洞分析)
- [漏洞利用](#漏洞利用)
- [参考文章](#参考文章)

## 环境搭建

```text
git checkout 5315f073233429c5f5c2c794594499debda307bd
gclient sync -D
python3 tools\dev\gm.py x64.release
```

## 信息搜集

issue链接：[<mark class="encrypted">8c8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3X3g2U0K9s2u0G2L8h3W2#2L8g2)9J5k6h3!0J5k6#2)9J5c8X3W2K6M7%4g2W2M7#2)9J5c8U0b7H3x3o6j5%4y4e0x3H3i4K6g2p5i4K6t1^5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3X3g2U0K9s2u0G2L8h3W2#2L8g2)9J5k6h3!0J5k6#2)9J5c8X3W2K6M7%4g2W2M7#2)9J5c8U0b7H3x3o6j5%4y4e0x3H3</mark> " <mark class="encrypted">6bcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3X3g2U0K9s2u0G2L8h3W2#2L8g2)9J5k6h3!0J5k6#2)9J5c8X3W2K6M7%4g2W2M7#2)9J5c8U0b7H3x3o6j5%4y4e0x3H3</mark>")

[<mark class="encrypted">d80K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0K9s2u0G2L8h3W2#2L8g2)9J5k6s2u0W2N6X3W2W2N6#2)9J5k6h3N6G2L8$3N6D9k6i4y4G2N6i4u0U0k6g2)9J5k6h3y4G2L8g2)9J5c8X3y4Q4x3V1k6$3z5q4)9J5c8Y4j5^5i4K6u0r3i4K6u0n7i4K6u0r3y4o6j5&6y4o6l9H3y4#2)9#2c8q4)9J5z5r3S2@1N6s2m8K6i4K6y4m8i4K6u0r3i4K6u0r3j5$3S2J5L8$3#2A6N6h3#2Q4x3X3c8J5k6i4k6A6k6i4N6Q4x3X3g2Y4L8$3!0Y4L8r3g2K6L8%4g2J5j5$3g2Q4x3X3g2U0L8$3#2Q4x3V1k6U0i4K6u0r3N6U0S2Q4x3V1k6$3z5q4)9J5c8W2)9J5b7W2)9J5c8U0b7$3z5e0b7H3x3o6M7`.</mark> " <mark class="encrypted">fcdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0K9s2u0G2L8h3W2#2L8g2)9J5k6s2u0W2N6X3W2W2N6#2)9J5k6h3N6G2L8$3N6D9k6i4y4G2N6i4u0U0k6g2)9J5k6h3y4G2L8g2)9J5c8X3y4Q4x3V1k6$3z5q4)9J5c8Y4j5^5i4K6u0r3i4K6u0n7i4K6u0r3y4o6j5&6y4o6l9H3y4H3`.`.</mark>")

revision commit hash : [<mark class="encrypted">86cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0K9s2u0G2L8h3W2#2L8g2)9J5k6h3N6G2L8$3N6D9k6i4y4G2N6i4u0U0k6g2)9J5k6h3y4G2L8g2)9J5c8Y4j5^5i4K6u0r3N6U0S2Q4x3V1k6Q4x3V1u0Q4x3V1k6W2k6o6V1K6j5X3g2X3y4$3q4T1y4K6R3$3k6o6f1K6y4U0N6U0x3X3q4W2y4K6R3^5x3U0V1J5x3X3x3J5x3$3q4S2x3r3g2V1j5e0j5@1i4K6g2p5i4K6t1^5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0K9s2u0G2L8h3W2#2L8g2)9J5k6h3N6G2L8$3N6D9k6i4y4G2N6i4u0U0k6g2)9J5k6h3y4G2L8g2)9J5c8Y4j5^5i4K6u0r3N6U0S2Q4x3V1k6Q4x3V1u0Q4x3V1k6W2k6o6V1K6j5X3g2X3y4$3q4T1y4K6R3$3k6o6f1K6y4U0N6U0x3X3q4W2y4K6R3^5x3U0V1J5x3X3x3J5x3$3q4S2x3r3g2V1j5e0j5@1</mark> " <mark class="encrypted">12fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0K9s2u0G2L8h3W2#2L8g2)9J5k6h3N6G2L8$3N6D9k6i4y4G2N6i4u0U0k6g2)9J5k6h3y4G2L8g2)9J5c8Y4j5^5i4K6u0r3N6U0S2Q4x3V1k6Q4x3V1u0Q4x3V1k6W2k6o6V1K6j5X3g2X3y4$3q4T1y4K6R3$3k6o6f1K6y4U0N6U0x3X3q4W2y4K6R3^5x3U0V1J5x3X3x3J5x3$3q4S2x3r3g2V1j5e0j5@1</mark>")

diff链接:

[<mark class="encrypted">c76K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0K9s2u0G2L8h3W2#2L8g2)9J5k6h3N6G2L8$3N6D9k6i4y4G2N6i4u0U0k6g2)9J5k6h3y4G2L8g2)9J5c8Y4j5^5i4K6u0r3N6U0S2Q4x3V1k6Q4x3V1u0Q4x3V1k6W2k6o6V1K6j5X3g2X3y4$3q4T1y4K6R3$3k6o6f1K6y4U0N6U0x3X3q4W2y4K6R3^5x3U0V1J5x3X3x3J5x3$3q4S2x3r3g2V1j5e0j5@1i4K6t1#2y4f1g2Q4x3U0f1J5x3g2)9J5c8W2)9#2c8q4)9J5z5r3S2@1N6s2m8K6i4K6y4m8i4K6u0r3i4K6u0r3j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2Y4L8$3!0Y4L8r3g2K6L8%4g2J5j5$3g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3z5q4)9J5c8Y4j5^5i4K6u0r3i4K6u0n7i4K6u0r3k6h3b7&6x3$3u0W2k6U0N6S2j5U0M7^5y4X3b7#2x3K6j5%4j5K6u0S2k6e0M7^5z5o6t1&6x3U0u0U0x3U0y4S2j5e0m8W2k6r3p5$3y4q4)9J5y4e0g2q4i4K6t1#2x3U0q4Q4x3V1j5`.</mark> " <mark class="encrypted">557K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0K9s2u0G2L8h3W2#2L8g2)9J5k6h3N6G2L8$3N6D9k6i4y4G2N6i4u0U0k6g2)9J5k6h3y4G2L8g2)9J5c8Y4j5^5i4K6u0r3N6U0S2Q4x3V1k6Q4x3V1u0Q4x3V1k6W2k6o6V1K6j5X3g2X3y4$3q4T1y4K6R3$3k6o6f1K6y4U0N6U0x3X3q4W2y4K6R3^5x3U0V1J5x3X3x3J5x3$3q4S2x3r3g2V1j5e0j5@1i4K6t1#2y4f1g2Q4x3U0f1J5x3g2)9J5c8R3`.`.</mark>")

## 前置知识

[`Reflect.construct()`](<mark class="encrypted">926K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2E0L8%4A6A6L8r3I4S2i4K6u0W2L8%4u0Y4i4K6u0r3P5X3S2Q4x3X3c8o6e0W2)9J5c8X3c8G2j5%4y4Q4x3V1k6i4k6h3u0Q4x3V1k6v1j5i4k6S2f1$3y4J5K9i4m8@1i4K6u0r3f1X3g2X3k6i4u0W2L8X3y4W2i4K6u0r3c8$3I4G2j5X3q4D9i4K6g2X3e0$3u0B7k6h3y4@1M7#2)9J5c8W2u0W2k6X3I4W2j5%4c8Q4x3V1k6U0L8$3&6K6N6s2u0#2j5%4c8Q4x3U0y4Q4c8e0S2Q4b7f1k6Q4b7f1c8Q4c8e0k6Q4b7U0y4Q4z5e0f1`.</mark> "Reflect.construct()")

函数原型

```javascript
Reflect.construct(target, argumentsList[, newTarget])
```

参数

- [`target`](# "target")：被运行的目标构造函数
- [`argumentsList`](# "argumentsList")：类数组，目标构造函数调用时的参数。
- [`newTarget可选`](# "newTarget可选")：作为新创建对象的原型对象的`constructor`属性，参考[`new.target`](# "new.target")操作符，默认值为`target`。

使用实例

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

FastNewObject的实现代码，位于`src/builtins/builtins-constructor-gen.cc`

首先获取上下文环境，其中包括`target`和`new_target`，接着定义一个`call_runtime`的`label`，如果快速路径分配失败，则会通过这个`label`跳转到慢速路径进行对象分配。

调用`FastNewObject`进入快速路径，如果快速路径分配失败，则会跳转到`BIND(&call_runtime)`，然后执行`Runtime::kNewObject`，进入慢速路径的分配

```c++
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

对于慢速路径的`runtime`函数可以在`vscode`里全局搜索`RUNTIME_FUNCTION(Runtime_xxxxx`，这里的`xxxxx`代表函数名，上方的`kNewObject`，那么就用这里的函数名应该是`NewObject`，所以就可以搜索`RUNTIME_FUNCTION(Runtime_NewObject)`

![](upload/attach/202507/971428_RR7PY4HQVS44X3M.png)

## 快速路径的分配流程

这里是全部完整的代码，位于`src/builtins/builtins-constructor-gen.cc`
