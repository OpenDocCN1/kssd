
```js
{
    name: 'Ljava/lang/Object;',
    type: 'pointer',
    size: 1,
    defaultValue: NULL,
    isCompatible (v) {
      if (v === null) {
        return true;
      }
      if (v === undefined) {
        return false;
      }
      const isWrapper = v.$h instanceof NativePointer;
      if (isWrapper) {
        return true;
      }
      return typeof v === 'string';
    },
    fromJni (h, env, owned) {
      if (h.isNull()) {
        return null;
      }
      return factory.cast(h, factory.use('java.lang.Object'), owned);
    },
    toJni (o, env) {
      if (o === null) {
        return NULL;
      }
      if (typeof o === 'string') {
        return env.newStringUtf(o);
      }
      return o.$h;
    }
};
```

而一个 js 数组对象是没有`.$h`的，于是你会得到莫名其妙的报错。而 `Java.cast`时会检查`isCompatible`，结果还是因为同样的原因，报错，没有办法进行转换。坑，frida 认为对象数组不是对象……

所以……怎么做呢，只有找到对象数组对应的 `type`，做一次`toJni`，然后再用目标类型的`type`，做一次`fromJni`，比如说我在`XposedFridaBridge`中做的：

```js
var env = Java.vm.getEnv()
var retType = fridaMethod._p[4]
var hhmRetType = XposedBridge.handleHookedMethod.overloads[0]._p[4]
return retType.fromJni(hhmRetType.toJni(xposedResult, env), env, false)
```

还有什么坑呢？就是基本类型直接是 javascript 类型，这些类型也是没有`.$h`的，所以，他们也不是 `Object`……只能手动用`java.lang.Integer`这样的对象进行转换了。

### 总结

`Frida`中 `Java` 对象对应的是 `frida-java-bridge`，其中数据类型的转换是由`type.js`负责的，很不幸在映射的时候与 `java` 中对象并不一致。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
