
## 4、算法分析

类型：对注册码进行数学运算检验

a、将12位的注册码分成四段，每段三个字符。假设注册码为 `n1 n2 n3 n4 n5 n6 n7 n8 n9 n10 n11 n12`。

b、对上面注册码的要求：`n4-n7+n1=n10+1`，`n8-n2+n5=n11+1`，`n9-n6+n3=n12-1`。

c、只要上面三要求都满足，就成功注册了。

我找到的几个注册码：`ABCDBCAHICGJ`、`EEEEEEEEEEDDF`。

## 5、注册信息保存在文件 `setup.ini` 中

```
[Serial]
Uses=J
                    <===好象是使用次数
Serial=EEEEEEEEEDDF
```
