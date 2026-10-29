
```
int __fastcall Java_com_illyb_main_NativeClass_userSaveReg(int a1, int a2, int a3, int a4, int a5, int a6, int a7, int a8)
{
  int v8; // r4@1
  int v9; // ST0C_4@1
  int v10; // r5@1
  int v11; // r6@1
  char *v12; // r7@1
  int v13; // ST0C_4@1
  int v14; // r0@1
  int v15; // r5@1
  int v16; // r0@1
  int v17; // r0@1
  int v18; // r0@3
  int result; // r0@3
  int v20; // r0@4

  v8 = a1;
  v9 = ((int (*)(void))sub_919C)();
  v10 = sub_919C(v8, a5);
  v11 = sub_919C(v8, a6);
  v12 = (char *)sub_919C(v8, a7);
  v13 = areYouOk(v9, v10, v11, v12, a8);
  _JNIEnv::ReleaseStringUTFChars(v8, a5, v10);
  _JNIEnv::ReleaseStringUTFChars(v8, a6, v11);
  _JNIEnv::ReleaseStringUTFChars(v8, a7, v12);
  v14 = (*(int (__fastcall **)(int, _DWORD))(*(_DWORD *)v8 + 24))(v8, "com/illyb/tool/OperatorService");
  v15 = v14;
  v16 = _JNIEnv::GetStaticMethodID(v8, v14, "getRegTimes", "(Landroid/content/Context;Ljava/lang/String;)I");
  v17 = _JNIEnv::CallStaticIntMethod(v8, v15, v16);
  if ( v13 && v17 <= 4 )
  {
    v18 = _JNIEnv::GetStaticMethodID(v8, v15, "saveReg", "(Landroid/content/Context;Ljava/lang/String;)Z");
    result = _JNIEnv::CallStaticBooleanMethod(v8, v15, v18);
  }
  else
  {
    v20 = _JNIEnv::GetStaticMethodID(v8, v15, "saveRegTimes", "(Landroid/content/Context;I)Z");
    _JNIEnv::CallStaticBooleanMethod(v8, v15, v20);
    result = 0;
  }
  return result;
}
```

Java调用JNI函数时，`a1(R0)`代表`JNIEnv*`，`a2(R1)`代表`thiz`，`a3(R2)`才是第1个参数，这里`a7`就是传过来的注册码。这部分代码的大意是先将Java传过来的参数转换成JNI能识别的类型（由ARM代码对照`JNI_ENV`函数表得知`sub_919C`实际上是`GetStringUTFChars`），再由`areYouOk`进行注册码合法性的验证，最后根据验证结果执行后面的动作调用Java中的不同方法。从前面的分析得知`userSaveReg`的返回值为1 注册成功，因此不能进入`else`的部分。那如何跳到`if`里呢，取决于`v13&&v17`的结果，`v13`是`areYouOk`的返回值，`v17`是`Lcom/illyb/tool/OperatorService/getRegTimes`的返回值，`getRegTimes`代码：

```
  private static int getRegTimes(Context paramContext, String paramString)
  {
    return paramContext.getSharedPreferences("reg", 0).getInt("regtimes", 0);
  }
```

以上代码从`/data/data/com.vipios/shared_prefs/reg.xml`将`regtimes`值取出，这个`regtimes`经分析为输入注册码的次数x2，那么`if`的条件应该是限制多次重试注册码的，那么`v13&&v17`的值必须<=4。

那么这里是否可以修改代码直接跳转到`if`里面呢？答案是可以的，但是不建议这么做，为什么呢？继续往下看。现在看来问题的关键就在`areYouOk`上，我们进入`areYouOk`的地盘，在IDA的`Exports`页面下找到`areYouOk`，点击进入，然后`jump to xref`：

![](https://bbs.kanxue.com/upload/attach/201412/8799_7g5s21vzpvek31y.png)

看到了吗，共有7处调用了`areYouOk`，如果在上面的`userSaveReg`里直接修改，剩下来的函数也要一个个修改，麻烦而且容易出错。通过进入其他调用`areYouOk`函数进行分析，更加肯定了`areYouOk`就是关键，而且返回1可以满足所有函数的正确跳转。

那么我们还是来修改`areYouOk`吧。ARM中函数的返回值存放在`R0`，我们只要在返回前将`R0`置为1就可以了。在`areYouOk`中往下拉到最后，或切换到图形模式可将流程看得更清楚些，看到返回前代码为：

```
loc_9518

ADD     SP, SP, #0x1FC

MOVS    R0, R4			      ;保存返回结果

POP     {R4-R7,PC}		    ;恢复进入函数前现场
```

典型的函数返回代码，现在我们只要将`MOVS R0, R4`改为`MOVS R0, #1`就离胜利不远了。在IDA中切换到`Hex View`窗口，`MOVS R0, R4`机器码为`20 1C`，那么要改成什么呢？真是头痛啊！

最后的利器隆重登场---- Arm汇编转换器该拿下遮羞布了，以上看到一个代码两个字节，需要用`thumb`模式转换，转换器汇编窗口输入`.code 16`，再输入要转换的代码：

![](https://bbs.kanxue.com/upload/attach/201412/8799_nswo3jzg83pcpt9.png)

OK，结果出来了，上面是原来的指令`MOVS R0, R4`机器码为`20 1C`，下面为修改后的指令`MOVS R0, #1`机器码为`01 20`。

好吧，UltraEdit久等了，谢幕就由你来吧（其实这才是最后的利器）！打开`libillyb.so`，Ctrl-G跳转到IDA中显示的修改地址，核实无误将`20 1C`修改为`01 20`，保存，`libillyb.so`复制回lib目录，编译运行，提示注册成功了，请注意注册成功后下面的按钮也变了。

![](https://bbs.kanxue.com/upload/attach/201412/8799_xdo53q2b6fhosr2.png)

手贱点了一下隐藏，于是程序抽屉的图标点不动了，说程序没安装，重启后图标消失，只有卸载重装才找回来了。至此爆破结束，各位耐心看我啰啰嗦嗦到现在的看官谢谢了！

[教程附件.rar](https://bbs.kanxue.com/attach-download-94429.htm)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [001.png](javascript:void(0);)（64.49kb，17次下载）
- [002.png](javascript:void(0);)（4.03kb，9次下载）
- [003.png](javascript:void(0);)（46.71kb，18次下载）
- [004.png](javascript:void(0);)（144.84kb，11次下载）
- [005.png](javascript:void(0);)（14.63kb，11次下载）
- [006.png](javascript:void(0);)（8.03kb，4次下载）
- [007.png](javascript:void(0);)（150.16kb，9次下载）
- [教程附件.rar](javascript:void(0);)（2.73MB，225次下载）
