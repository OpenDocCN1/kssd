
# 写了个脚本解释器

> 原文：[`www.pediy.com/kssd/pediy12/40183.html`](https://www.pediy.com/kssd/pediy12/40183.html)

纯解释执行，不生成语法树和 `opcode`

控制流程是 `yacc` 的状态矩阵，我觉得只要把 `lex` 文件改改，不要出现明文脚本文件，应该可以拿来加密算法![](img/081868d20f20fe7a9b4b88ab0d0d33c6.jpg "狂笑")

测试代码

```
a=10;

n=3;

while(a-- && n)

{

if(a<4)break;

print("while a="+a);

if(a==6)continue;

if(a==8)while(n>1.5)print("sub while n="+(n=n-0.5));

print("next");

}

print(a+n);

print("end");
```

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `testscript.rar`
|
