
```javascript
Java.perform(function() {
    var hookClass = undefined;

    var ClassUse = Java.use("java.lang.Class");

    var objectclass = Java.use("java.lang.Object");

    var dexclassLoader = Java.use("dalvik.system.DexClassLoader");

    var orininclass = Java.use("cn.chaitin.geektan.crackme.MainActivity");

    var Integerclass = Java.use("java.lang.Integer");

    var mainAc = orininclass.$new();

    dexclassLoader.loadClass.overload("java.lang.String").implementation = function(name) {
        var hookname = "cn.chaitin.geektan.crackme.MainActivityPatch";

        var result = this.loadClass(name, false);

        if (name == hookname) {
            var hookClass = result;

            var hookClassCast = Java.cast(hookClass, ClassUse);

            console.log("-----------------------------GET Constructor-------------------------------------");

            var ConstructorParam = Java.array("Ljava.lang.Object;", [objectclass.class]);

            var Constructor = hookClassCast.getDeclaredConstructor(ConstructorParam);

            console.log("Constructor:" + Constructor);

            console.log("orinin:" + mainAc);

            var instance = Constructor.newInstance([mainAc]);

            console.log("patchAc:" + instance);

            send(instance);

            console.log("-----------------------------GET Methods----------------------------");

            var func = hookClassCast.getDeclaredMethods();

            console.log(func);

            console.log("--------------------------GET Joseph Function---------------------------");

            console.log(func[0]);

            var f = func[0];

            var num1 = Integerclass.$new(5);

            var num2 = Integerclass.$new(6);

            var numArr1 = Java.array("Ljava.lang.Object;", [num1, num2]);

            var num3 = Integerclass.$new(7);

            var num4 = Integerclass.$new(8);

            var numArr2 = Java.array("Ljava.lang.Object;", [num3, num4]);

            console.log("-----------------------------GET Array------------------------------");

            console.log(numArr1);

            console.log(numArr2);

            var rtn1 = f.invoke(instance, numArr1);

            var rtn2 = f.invoke(instance, numArr2);

            console.log("--------------------------------FLAG---------------------------------");

            console.log("DDCTF{" + rtn1 + rtn2 + "}");

            console.log("--------------------------------OVER--------------------------------");

            return result;
        }

        return result;
    };
});
```

执行后发现成功：D

![](https://bbs.kanxue.com/upload/attach/201807/811277_H3F66QKXWTUSKZ5.gif)

得到最终答案：

![](https://bbs.kanxue.com/upload/attach/201807/811277_Q8Z3YAUUHSDS4N2.png)

## 总结

通过对这个例子的完整过程进行学习，可以说算是掌握了 Frida Java 中一些比较高级的用法：

- `Java.array`：任意数据的构造
- `Java.cast`：类型转换
- `overload`：方法重载
- `spawn`

同时也对 Frida 如何 Hook 动态加载 DEX 的方法有了一个清晰的思路。当然，Frida 的学习之路还很长……

[冰与火的战歌：Windows 内核攻防实战高级班！从零到实战，融合 AI 与 Windows 内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2018-7-8 14:36

被 ghostmazeW 编辑

，原因：
