
### 第三件：在`R$layout`类中声明我们新的布局文件的资源ID；

我们完成以上步骤后，我们需要将`MyActivity`类中的`activity_my`替换成`activity_new`：

```smali
sget v0, Lcom/example/myapplication/R$layout;->activity_new:I
```

以上寄存器要联系上下文，比如以上代码是将`activity_new`布局的资源ID储在寄存器`v0`中，再将`v0`寄存器中的资源ID作为参数传递给`setContentView`方法。所以寄存器要联系上下文，不要直接复制粘贴。

然后回编译成功后，再使用MT管理器对修改后的apk文件进行重新签名，让我们来看看最终的结果如何：

![](https://bbs.kanxue.com/upload/attach/202308/926486_NDA5P2JX48W35CK.png)

点击跳转后：

![](https://bbs.kanxue.com/upload/attach/202308/926486_H29Q3B78T5PNK42.png)

从结果来看我们成功完成了替换掉点击跳转按钮后跳转的Activity！现在这篇文章也到此结束，这篇文章重点是让大家对Java基础、smali汇编基础、安卓开发基础有简单的了解，最后还是那句话：“知彼知己者，百战不殆；不知彼而知己，一胜一负，不知彼，不知己，每战必殆。”

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2025-8-7 11:50 被黎明与黄昏编辑，原因：文字缺失

[#基础理论](https://bbs.kanxue.com/forum-161-1-117.htm)

上传的附件：

- [`My Application.apk`](javascript:void(0);)（3.09MB，19次下载）
