
```
function print_string(addr){

    var base_addr = Module.findBaseAddress("libnet_crypto.so");

    if(base_addr){

        console.log(base_addr.add(addr).readCString());

    }

}
```

使用`frida`附加到目标APP，调用`print_string(0x17D0C1)`可得到如下结果:

![图片描述](https://bbs.kanxue.com/upload/attach/202007/847002_TN4AUFPQWV8RAUH.png)

那么与其对应的`sub_49864+1`就是`sign`方法的地址。

### 3.3 使用unicorn还原字符串

被“孤挺花”混淆的字符串可以用`unicorn`仿真器进行修复，具体原理可参考大佬的博客：`ffdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2D9k6h3q4V1M7X3!0&6j5h3I4Q4x3X3g2U0L8W2)9J5c8W2)9K6c8Y4m8Q4x3@1b7&6y4K6t1`.

## 4、定位关键函数

`sub_49864` `f5`结果如下：

![图片描述](https://bbs.kanxue.com/upload/attach/202007/847002_ST9QHT8HUQE8EQU.png)

鄙人定位关键函数的方法是使用`frida` hook输入和输出对比结果来进行定位。最终定位的关键函数为:

```
sub_63CB8->sub_63C1C->sub_61A70->sub_61DBA
```

## 5、还原sign

函数`sub_61DBA`的`cfg`图如下：

![图片描述](https://bbs.kanxue.com/upload/attach/202007/847002_U86D6REV3Y3VYSS.png)

看到这个`cfg`图， 当时觉得这不白给吗，什么混淆都没有。 脑子一热一边动态调试，一遍翻译`arm`指令。 花了点时间，翻译了500多行`arm`指令瞬间觉得不对劲了，这尼玛大部分都是指令替换膨胀代码。 粗略计算了下，该函数大概有5000条指令，这尼玛什么时候是头，顶不住顶不住，为了头发着想，偷了个懒，直接`f5` copy伪代码，修改了一下，发现结果对上了，果断放弃翻译`arm`汇编，有兴趣的可以试下翻译。

`f5`的伪代码，有大量类似于:

```
(~v5 & 0xF92342E3 | v5 & 0x6DCBD1C) ^ (HIDWORD(v4) & 0x6DCBD1C | ~HIDWORD(v4) & 0xF92342E3);
```

的运算，可以替换成

```
v5 ^ (HIDWORD(v4)
```

等形式。

```
（注：Z为常量）
(~X & Z | X & ~Z) ^ (Y & ~Z| ~Y & Z);  x ^ y
(~X & Z | X & ~Z) = X ^ ~Z = ~X ^ Z
推理过程:
    (~X & Z | X & ~Z) ^ (Y & ~Z| ~Y & Z)
  = X ^ Z ^ Y ^ Z
  = X ^ Y
```

## 6、结果验证

`hook` `sign`方法，获取加密参数：

![图片描述](https://bbs.kanxue.com/upload/attach/202007/847002_8Y45MN3R5V7P2J6.png)

`c`运行结果图：

![图片描述](https://bbs.kanxue.com/upload/attach/202007/847002_GW2XFWN3ZS9V6UY.png)

## 7、杂谈

逆过老版本的某右APP，当时的只有个`sign`加签，`post`提交参数和返回结果还没有`aes`加解密（怀念以前不穿衣服的你），整体难度应该算是容易的吧。老版本的`sign`没记错的话就是`md5`(待加签的字符串进行变形)，新版本的话，有点像是自己魔改的`md5`，不太确定，但整体感觉上是像。另外，`aes`加解密这块好像是比`sign`难度高点，还没去看。还发现个`hook`点可以让`app`使用老接口，`frida`代码：

```
    var nd3 = Java.use("nd3");

    var ia = Java.use("ia$a");

    nd3.a.overload('java.lang.String').implementation = function(arg1){

        return false;

    }

    ia.c.implementation = function(){

        var result = this.c();

        console.log(result);

        return 1;

    }
```

感觉该`so`是练习反混淆的好样本，`cfg`不太复杂，想练习的朋友可以试试先`trace`，然后根据`trace`手动修复，再写脚本还原。强烈建议各位胆大信息的大佬，去翻译核心算法的`arm`汇编，试过的都说爽（逃）。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
