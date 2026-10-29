
这里我对它进行了调用查找，看到只有 1 个地方，我们跟进去看看。跟进去之后，它就是以下内容，只是一个简单的赋值给`sAddSecurityFactorProcessCallback`。我们再对它进行调用查找，看看是什么地方对它进行调用。

```java
public static void setAddSecurityFactorProcessCallback(a aVar) {
    sAddSecurityFactorProcessCallback = aVar;
}
```
