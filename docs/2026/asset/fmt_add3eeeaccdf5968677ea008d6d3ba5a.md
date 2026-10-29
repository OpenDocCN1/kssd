
具体流程和`fishhook`一致，不过经过实验有个坑，就是在`before_objc_msgSend_inline`和`after_objc_msgSend_inline`中暂时不能调用OC方法，因为会造成如下死循环。

![](https://bbs.kanxue.com/upload/attach/202201/889798_XRKAKE368UB768X.png)

## 得到`objc_msgSend`的参数

由于参数保存在`x0-x8`寄存器中，那么在`before_objc_msgSend`中对寄存器分析，就可以获取到入参。

```
// 打印调用的OC方法的对象和方法名，和最多两个参数。
void printSpecificParam_fish(id self, SEL _cmd, uintptr_t param1, uintptr_t param2,uintptr_t lr)
{
    // 此方法中暂只能使用C方法,使用OC方法可能会导致寄存器异常导致崩溃，经测试，发生在相同方法调用相同方法时崩溃
    // NSlog可以用
    const char *className = object_getClassName(self);
    const char *selector = sel_getName(_cmd);
    // NSLog(@"class : %s, methodname : %s",className,selector);
    if ( strcmp( selector, "fileExistsAtPath:" ) == 0) {
        NSLog(@"class : %s, methodname : %s, param1 : %@",className,selector,param1);
    }else {
        NSLog(@"class : %s, methodname : %s",className,selector);
    }
}
```

这里有个坑，`self`参数，在一些app中会引入引用计数问题导致崩溃，还没有具体查询。作为TODO吧，现在的`printSpecificParam_fish`参数入参需要修改一下，将`self`去除：

```
void printSpecificParam_fish( SEL _cmd, uintptr_t param1, uintptr_t param2,uintptr_t lr)
{
    const char *selector = sel_getName(_cmd);
    NSLog(@"methodname : %s",selector);
}
```

## 运行结果

![](https://bbs.kanxue.com/upload/attach/202201/889798_RW2AKZCN3Y25SSF.png)

可以看到打印了所有的OC方法的方法名和参数。

## 最后

完成代码可以参考[github地址](https://bbs.kanxue.com/elink@47cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6i4h3f1D9I4y4g2)9J5c8X3S2G2L8$3E0a6j5X3A6U0i4K6g2X3e0i4y4Y4M7$3g2F1k6l9`.`.)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2022-1-25 11:13 被LeoW丨编辑 ，原因： 更换图片

[#逆向分析](https://bbs.kanxue.com/forum-166-1-189.htm)

[#基础理论](https://bbs.kanxue.com/forum-166-1-188.htm)

[#HOOK注入](https://bbs.kanxue.com/forum-166-1-192.htm)

[#源码分析](https://bbs.kanxue.com/forum-166-1-194.htm)
