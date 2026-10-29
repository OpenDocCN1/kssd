
不难看出，`:cond_9`处的那小段代码就是显示**你不是捐赠用户，不能改变过超过4个receiver**。

其实这里要改变这个跳转，有3个地方可以修改。

因为这里的跳转实际上是根据3个条件相与的结果来进行的（要`(!c.ka.booleanValue() && l.bJ() >= 4 && !l.jp.contains(jy.jE))`这个表达式为真才会跳转到执行弹出not donated提示框的代码）。

条件1：如果`h.c.ka`的值为真，则跳转到`:cond_0`

条件2：如果已经禁用的`reciever`数量(`v2`寄存器）小于 4 个（`v4`寄存器），则跳转到`:cond_0`

`if-lt v2, v4, :cond_0`

条件3：`if-eqz v2, :cond_9`

也就是说，代码逻辑是：

```
if( !condition_1 && !condition_2 && !condition_3 )
执行提醒捐赠的代码
else
执行正常的保存设置代码
```

最初我采用的破解方法是把`if-eqz v2, :cond_9`这个跳转修改为了`if-nez v2, :cond_9`，这样当然是可以的，因为改变任意一个条件都是可以成功的。

现在我采用的破解方法是让“`h.c.ka`的值为真”，怎么让它为真呢？等下往下看时我会说到。因为这个`h.c.ka`的值不单单是影响**可以修改的receiver的数量**，还影响到free版的应用的另外两个选项：**阻止应用重启、启用Chuck Norris模式**，如果只修改一处地方就能影响到两外以上的跳转（这些跳转正是我们需要的），何乐而不为呢？因此，这里我不再按老方法去修改跳转了。

==================================================

然后再看`com\rs\autorun\misc\AutorunPreferencesActivity.smali`

因为这个`AutorunPreferences`是修改配置时的`Activity`，所以看下它的代码。看它是怎么处理**复选框被点击事件**的。根据编程经验加上反编译出来的`smali`代码，可以知道，这个应用只是把`pro`版的功能给隐匿了，相应的操作代码还是有的。因此，只需要让程序自己觉得是`pro`版就行了。

这里它对于`AutorunPreferencesActivity`的事件响应逻辑是这样的：当`pro`版才能使用的复选框被点击时，判断是否满足相应的条件，不满足的话就弹出提示框，提示此功能只有`pro`版才能使用。并把复选框的值设置为`false`.

`com\rs\autorun\misc\AutorunPreferencesActivity.smali`相关代码：

代码比较多，我这里只帖出关键的：
