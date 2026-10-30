
# [原创]看雪CTF2017第六题 Ericky-apk详细writeup（从一个安卓新手的角度）

## 概述

本题是安卓`cm`，目测肯定需要调试`so`。

准备工具：

1. `ApkIde`改之理（其他类似的也行，能够反编译`apk`，得到`jar,so`等）
2. `IDA`（用于调试`so`），需要`6.x`以上，忘了是`x`几，我用的`6.6`
3. `adb`(`ApkIde`改之理就有)

## 反编译

将`6-Ericky kanxue.apk`拖进`ApkIDE`改之理，等待编译（没有加壳），`ok`。

在右侧树结构栏中，找到`smali`->`android`->`com`->`miss`->`rfchen`，列表中就是`java`层的主要函数。

点击`MainActivity.smali`，然后点击工具栏中`jd-gui.exe`，抓到`java`源码查看。

```
public class MainActivity extends Activity

{

  private EditText ˊˊﹶˊﹶﹶﹶˊﹶˊﹶˊˊˊˊˊˊˊﹶﹶﹶﹶﹶˊﹶﹶˊˊˊﹶﹶﹶˊﹶˊˊﹶﹶﹶˊˊˊﹶˊﹶﹶﹶˊﹶﹶˊﹶﹶﹶﹶﹶˊﹶﹶﹶˊˊﹶﹶˊˊﹶˊˊˊﹶˊˊﹶﹶˊˊˊﹶˊﹶﹶˊˊˊﹶˊˊﹶﹶˊˊˊﹶﹶˊˊˊﹶˊˊﹶﹶˊﹶˊﹶﹶˊﹶﹶˊﹶﹶﹶˊˊˊˊﹶﹶﹶﹶˊﹶˊˊˊ = null;

  private Button ﹶˊﹶˊﹶﹶﹶﹶﹶﹶﹶﹶﹶﹶﹶˊﹶﹶﹶˊﹶˊˊﹶˊˊﹶﹶˊˊˊﹶﹶˊﹶˊﹶﹶﹶﹶˊﹶﹶˊˊﹶﹶﹶﹶﹶˊˊˊˊˊˊﹶˊˊﹶﹶﹶˊˊﹶﹶˊˊﹶﹶˊˊˊﹶˊﹶˊﹶˊﹶﹶˊﹶﹶˊˊˊﹶﹶˊﹶﹶﹶﹶˊﹶﹶˊˊﹶﹶﹶﹶˊﹶﹶﹶˊˊﹶˊˊﹶﹶﹶﹶˊˊˊˊˊﹶﹶˊˊﹶˊﹶ = null;

  protected void onCreate(Bundle paramBundle)

  {

    super.onCreate(paramBundle);

    setContentView(2130968603);

    this.ﹶˊﹶˊﹶﹶﹶﹶﹶﹶﹶﹶﹶﹶﹶˊﹶﹶﹶˊﹶˊˊﹶˊˊﹶﹶˊˊˊﹶﹶˊﹶˊﹶﹶﹶﹶˊﹶﹶˊˊﹶﹶﹶﹶﹶˊˊˊˊˊˊﹶˊˊﹶﹶﹶˊˊﹶﹶˊˊﹶﹶˊˊˊﹶˊﹶˊﹶˊﹶﹶˊﹶﹶˊˊˊﹶﹶˊﹶﹶﹶﹶˊﹶﹶˊˊﹶﹶﹶﹶˊﹶﹶﹶˊˊﹶˊˊﹶﹶﹶﹶˊˊˊˊˊﹶﹶˊˊﹶˊﹶ = ((Button)findViewById(2131427415));

    this.ˊˊﹶˊﹶﹶﹶˊﹶˊﹶˊˊˊˊˊˊˊﹶﹶﹶﹶﹶˊﹶﹶˊˊˊﹶﹶﹶˊﹶˊˊﹶﹶﹶˊˊˊﹶˊﹶﹶﹶˊﹶﹶˊﹶﹶﹶﹶﹶˊﹶﹶﹶˊˊﹶﹶˊˊﹶˊˊˊﹶˊˊﹶﹶˊˊˊﹶˊﹶﹶˊˊˊﹶˊˊﹶﹶˊˊˊﹶﹶˊˊˊﹶˊˊﹶﹶˊﹶˊﹶﹶˊﹶﹶˊﹶﹶﹶˊˊˊˊﹶﹶﹶﹶˊﹶˊˊˊ = ((EditText)findViewById(2131427416));

    this.ﹶˊﹶˊﹶﹶﹶﹶﹶﹶﹶﹶﹶﹶﹶˊﹶﹶﹶˊﹶˊˊﹶˊˊﹶﹶˊˊˊﹶﹶˊﹶˊﹶﹶﹶﹶˊﹶﹶˊˊﹶﹶﹶﹶﹶˊˊˊˊˊˊﹶˊˊﹶﹶﹶˊˊﹶﹶˊˊﹶﹶˊˊˊﹶˊﹶˊﹶˊﹶﹶˊﹶﹶˊˊˊﹶﹶˊﹶﹶﹶﹶˊﹶﹶˊˊﹶﹶﹶﹶˊﹶﹶﹶˊˊﹶˊˊﹶﹶﹶﹶˊˊˊˊˊﹶﹶˊˊﹶˊﹶ.setOnClickListener(new View.OnClickListener()

    {

      public void onClick(View paramView)

      {

        MainActivity.this.ˊˊﹶˊﹶﹶﹶˊﹶˊﹶˊˊˊˊˊˊˊﹶﹶﹶﹶﹶˊﹶﹶˊˊˊﹶﹶﹶˊﹶˊˊﹶﹶﹶˊˊˊﹶˊﹶﹶﹶˊﹶﹶˊﹶﹶﹶﹶﹶˊﹶﹶﹶˊˊﹶﹶˊˊﹶˊˊˊﹶˊˊﹶﹶˊˊˊﹶˊﹶﹶˊˊˊﹶˊˊﹶﹶˊˊˊﹶﹶˊˊˊﹶˊˊﹶﹶˊﹶˊﹶﹶˊﹶﹶˊﹶﹶﹶˊˊˊˊﹶﹶﹶﹶˊﹶˊˊˊ();

      }

    });

  }

  public void ˊˊﹶˊﹶﹶﹶˊﹶˊﹶˊˊˊˊˊˊˊﹶﹶﹶﹶﹶˊﹶﹶˊˊˊﹶﹶﹶˊﹶˊˊﹶﹶﹶˊˊˊﹶˊﹶﹶﹶˊﹶﹶˊﹶﹶﹶﹶﹶˊﹶﹶﹶˊˊﹶﹶˊˊﹶˊˊˊﹶˊˊﹶﹶˊˊˊﹶˊﹶﹶˊˊˊﹶˊˊﹶﹶˊˊˊﹶﹶˊˊˊﹶˊˊﹶﹶˊﹶˊﹶﹶˊﹶﹶˊﹶﹶﹶˊˊˊˊﹶﹶﹶﹶˊﹶˊˊˊ()

  {

    String str = this.ˊˊﹶˊﹶﹶﹶˊﹶˊﹶˊˊˊˊˊˊˊﹶﹶﹶﹶﹶˊﹶﹶˊˊˊﹶﹶﹶˊﹶˊˊﹶﹶﹶˊˊˊﹶˊﹶﹶﹶˊﹶﹶˊﹶﹶﹶﹶﹶˊﹶﹶﹶˊˊﹶﹶˊˊﹶˊˊˊﹶˊˊﹶﹶˊˊˊﹶˊﹶﹶˊˊˊﹶˊˊﹶﹶˊˊˊﹶﹶˊˊˊﹶˊˊﹶﹶˊﹶˊﹶﹶˊﹶﹶˊﹶﹶﹶˊˊˊˊﹶﹶﹶﹶˊﹶˊˊˊ.getText().toString().trim();

    StringBuilder localStringBuilder = new StringBuilder();

    localStringBuilder.append(str);

    if (utils.check(localStringBuilder.toString().trim()))

    {

      Toast.makeText(this, MainActivity.1.utils.dbcb("뙩あ嵓ﳈ"), 0).show();

      return;

    }

    Toast.makeText(this, MainActivity.1.utils.dbcb("뙸ぞ崌ﳯ�຿핣�晌鬗㞕뵯"), 0).show();

  }

}
```

这混淆的函数名我也是醉了，但这都不重要。输入`key`之后，然后点击按钮，进入`onClick`，调用了上面代码中第二个函数（什么？我怎么知道的，因为它们哪个...点号...的函数名相同！！）。

然后调用了`utils.check`来验证，成功提示！这里成功和错误提示的字符串做过变换，通过`utils.dbcb`解密，不细看了，不重要！

进入`utils.java`，看到加载了`so`，调用的是这个`so`的导出函数，看反编译目录`lib/armeabi-v7a`（只提供了`arm`的`so`，要有个`x86`的好了），[知道这个`so`是`librf-chen.so`](https://bbs.kanxue.com/elink@a2eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4S2F1i4K6u0V1i4K6u0V1M7$3!0D9K9h3u0J5k6W2)9J5k6r3y4Z5k6h3&6Q4x3X3b7H3z5o6c8K6K9e0b7H3P5h3u0K6N6X3y4Z5k6o6W2V1K9i4W2T1i4K6u0W2M7$3!0Q4x3V1j5`.)。

```
 //典型的NDK调用，查查就知道了！
package com.miss.rfchen;
public class utils
{
  static
  {
    System.loadLibrary(MainActivity.1.utils.dbcb("ᐲེ雒�蹬"));
  }
  public static native boolean check(String paramString);
}
```

那么重点来了，要分析`librf-chen.so`的`check`函数，才能搞定此题。

## 准备调试

早上提前学习了一下`so`调试方法，找到了看雪安卓大神的教程，就是参考中的`IDA`动态调试技术，然后用上了，很好用！

### 跟着走

下面开始照着做。

1. 连上手机（或者模拟器），使用`adb devices`看看成功连上没有
2. `adb push ../dbgsrv/android_server /sdcard/sv`，教程是直接放入`/data/data`，一般权限不够
3. 然后进入`shell`，`adb shell`，输入`su`，获得`root`权限，然后`cp /sdcard/sv /data/data/sv`
4. 修改`sv`权限，`chmod 777 /data/data/sv`
5. 运行`sv`，`/data/data/sv`，默认监听到`23946`端口，`Listening on port #23946`。这步有个细节，不能直接`adb shell /data/data/sv`，这样权限不够，无法读取到进程信息，需要`adb shell; su; /data/data/sv`
6. 再开一个`cmd`，然后运行`adb forward tcp:23946 tcp:23946`
7. 运行一个`idaq.exe`，然后在菜单`debugger`->`attach`->`remote Armlinux/android debugger`，输入`localhost, 23946, ok`
8. 弹出进程框，按下`Alt+T`，输入`chen`，搜索到`1808 [32] com.miss.rfchen`，`ok`
9. `F9`运行

```
\ApkIDEz>    .\adb.exe shell
shell@your phone:/ $ su
su
root@your phone:/ # /data/data/sv
/data/data/sv
IDA Android 32-bit remote debug server(ST) v1.17. Hex-Rays (c) 2004-2014
```

在界面中输入`key`，然后点击按钮，此时`librf-chen.so`才加载，然后`ctrl+s`，`alt+t`，输入`librf`找到`librf-chen.so`的基地址信息(记为`base`)，记下来。

[用另一个`ida`打开`librf-chen.so`](https://bbs.kanxue.com/elink@281K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4S2F1i4K6u0V1i4K6u0V1K9h3c8S2L8r3W2T1M7X3k6Q4x3X3c8U0K9r3g2F1i4K6u0V1z5i4Z5H3N6h3c8@1y4o6M7@1k6i4c8W2y4$3u0%4M7X3@1$3k6K6q4Y4i4K6u0W2M7$3!0Q4x3V1j5`.)，找到`check`导出函数的偏移地址`00002814`，计算`base+00002814`，然后`g`在`IDA`调试器中输入该地址，加上断点。

```
check(_JNIEnv *,_jclass *,_jstring *) 00002814
```

`IDA`基本调试快捷键和`OD`一样:

```
F9: 运行 
F8: 步过 
F7：步入
```

`F9`，跑起来，然后再次点击按钮，就断下来，进入了`check`。

下面就是跟和调试的过程了，看数据，看流程，分析算法！

### arm汇编基础

得提前有个准备，看看`arm`指令，了解基本的指令，函数调用方式，下面列几个，更多的就看参考中的文章了

```
MOVS 同x86的mov
LDR 加载内存数据到寄存器
STR 寄存器数据存入内存
B/BL 跳转/函数调用
TST/CMP 比较
ADD/SUB 加/减
```

然后最主要的，函数调用的参数传递。`arm`默认使用的`fastcall`，通过`r0,r1,r2,r3`传递参数，超过`4`个参数，使用堆栈传递，`r0`也保存返回值。

### 关键点跟踪

在`check`断下之后，先是一段数据初始化，先滤过，然后`blt sub_2874`，进入关键函数

然后看到通过`MOVS`，`STR`将一些字符放入了内存。

```
.text:0000288A 000 01 60                                         STR             R1, [R0]
.text:0000288C 000 4A 20                                         MOVS            R0, #'J'
.text:0000288E 000 79 21                                         MOVS            R1, #'y'
.text:00002890 000 AD F8 22 00                                   STRH.W          R0, [SP,#arg_22]
.text:00002894 000 AD F8 24 10                                   STRH.W          R1, [SP,#arg_24]
.text:00002898 000 75 21                                         MOVS            R1, #'u'
.text:0000289A 000 AD F8 26 10                                   STRH.W          R1, [SP,#arg_26]
.text:0000289E 000 33 21                                         MOVS            R1, #'3'
```

接着就看让我恐惧的一幕，`b loc_2898`开始各种跳转，指令操作，然后刚跳完又是一个`b xxx`，接着各种跳转，毫无疑问，这是一段花指令了。

```
.text:0000289E                 B               loc_2898
```

### 花指令结构

经过多次跟踪，恶心到快吐的时候，终于看出话指令的基本结构了：
