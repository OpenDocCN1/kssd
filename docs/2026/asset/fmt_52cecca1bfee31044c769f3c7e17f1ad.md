
## Smali 注解指令

4. `.enum`：用于声明一个枚举类型。语法为`.enum <visibility> <enum_type> {<enum_element>}`，其中`<visibility>`可以是`public`、`protected`、`private`或`package`，`<enum_type>`是枚举类型的全限定名，`<enum_element>`是枚举的元素。

5. `.end enum`：用于结束一个枚举类型的声明。

6. `.field`：用于为字段添加注解。语法为`.field <field_spec> {<annotation>}`，其中`<field_spec>`是字段的声明，`<annotation>`是字段的注解。

7. `.method`：用于为方法添加注解。语法为`.method <method_spec> {<annotation>}`，其中`<method_spec>`是方法的声明，`<annotation>`是方法的注解。

8. `.parameter`：用于为方法参数添加注解。语法为`.parameter <parameter_index> <annotation>`，其中`<parameter_index>`是参数的索引，从0开始，`<annotation>`是参数的注解。

9. `.prologue`：用于指定方法的前导代码。

10. `.line`：用于指定源代码中的行号。

11. `.end method`：用于结束一个方法的声明。

12. `.catch`：用于捕获异常。语法为`.catch <exception_type> {<handler>}`，其中`<exception_type>`是异常类型的全限定名，`<handler>`是捕获异常后的处理代码。

13. `.catchall`：用于捕获所有异常。语法为`.catchall {<handler>}`，其中`<handler>`是捕获异常后的处理代码。

14. `.locals`：用于指定方法中的本地变量数量。语法为`.locals <count>`，其中`<count>`是本地变量的数量。

15. `.end`：用于结束一个代码块或注解的声明。

注解的作用范围可以是类、方法或者字段。

如果注解的作用范围是类，`.annotation`指令会直接定义在smali文件中，例如：

```smali
.class public interface abstract annotation Ll/۬ۘ;
.super Ljava/lang/Object;
.source "H5TW"

# annotation
// 定义注解
.annotation runtime Ljava/lang/annotation/Retention;
// 定义一个运行时注解Retention
// runtime表示该注解在运行时可见，即可以通过反射获取该注解信息
// Ljava/lang/annotation/Retention表示Retention注解的类型签名
value = .enum Ljava/lang/annotation/RetentionPolicy;->CLASS:Ljava/lang/annotation/RetentionPolicy;
// 枚举类型，表示Retention注解的保留策略为CLASS
// .enum表示定义一个枚举类型
// Ljava/lang/annotation/RetentionPolicy表示RetentionPolicy枚举的类型签名
// ->表示枚举类型的访问符
// CLASS表示RetentionPolicy枚举的一个值，表示注解保留在类文件中，但在运行时不可见
.end annotation
// 注解定义结束
```

如果是方法或字段，`.annotation`指令则会包含在方法或字段中定义。

注解方法，例如：

```smali
.method public onTouch(Landroid/view/View;Landroid/view/MotionEvent;)Z
    .locals 0
    .annotation build Landroid/annotation/SuppressLint;
        // 定义一个编译时注解SuppressLint
        // build表示该注解是Android SDK提供的注解
        // Landroid/annotation/SuppressLint表示SuppressLint注解的类型签名
        value = {
            "ClickableViewAccessibility"
        }
        // 定义注解的属性value，表示需要忽略的lint警告类型
        // "ClickableViewAccessibility"表示需要忽略的lint警告类型，即点击事件缺少无障碍支持
    .end annotation
    // 注解定义结束
```

注解字段，例如：

```smali
.field public top:F
.annotation runtime Ljava/lang/Deprecated;
// 定义一个运行时注解Deprecated
// runtime表示该注解在运行时可见，即可以通过反射获取该注解信息
// Ljava/lang/Deprecated表示Deprecated注解的类型签名
.end annotation
// 注解定义结束
.end field
```

## 编写一个简单的安卓APP

到此Java和smali基础已经学习结束，接下来我们一起学习写一个简单的安卓APP，写完后我们一起把我们写的这个安卓APP逆向分析后破解掉，这样我们就会对安卓逆向和安卓开发之间的关系也有一定了解，并且也会提升我们的逆向水平。

> 孙子曰：“知彼知己者，百战不殆；不知彼而知己，一胜一负，不知彼，不知己，每战必殆。”

那我们开始写一个简单的安卓APP吧！我们使用的工具是Android Studio，在写之前，我们需要安装Android Studio并创建一个项目：

安装Android Studio并创建项目可以参考这篇帖子：

[Android studio安装与创建第一个helloworld项目\_安卓helloworld项目下载\_王 三 二的博客-CSDN博客](<https://bbs.kanxue.com/elink@8eaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4q4#2K9h3y4C8k6h3k6X3K9h3y4A6k6h3&6@1i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5I4x3e0b7$3x3K6x3%4z5o6R3>).

但是我们不使用`empty activity`项目类型，而是使用`empty views activity`项目类型，因为我的`empty activity`项目类型的项目和网上的完全不一样，反而是`empty views activity`项目类型的项目和网上的`empty activity`项目类型的项目是一致的，所以我就创建`empty views activity`项目类型的项目了。

配置好环境后我们就来写一个简单的安卓APP，作用就是从主`Activity`跳转到我们创建的`Activity`。

### 第一步：编写主 `Activity` 布局

写一个安卓APP与写一个网站颇有相似之处，第一步我们需要去编写布局文件，下面是主`Activity`布局的XML代码：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!" />

    <Button
        android:id="@+id/skip"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="跳转" />

</LinearLayout>
```

让我们逐行解释每个元素的作用：

1. `<?xml version="1.0" encoding="utf-8"?>`：这是XML文件的声明，指定了文件的版本和编码。
2. `<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android">`是根视图的开始标签，并使用`xmlns:android`属性定义了一个XML命名空间。这个命名空间是用来引用Android的命名空间，以便可以使用Android提供的特定属性和元素。
3. `android:layout_width="match_parent"`和`android:layout_height="match_parent"`：这两个属性设置根视图的宽度和高度为与父容器相匹配，即填充整个屏幕。
4. `android:orientation="vertical"`：这个属性指定了子视图在`LinearLayout`中的排列方向为垂直方向。
5. `android:gravity="center"`：这个属性指定了子视图在`LinearLayout`中的对齐方式为居中对齐。
6. `<TextView>`：这是一个`TextView`视图，用于显示文本内容。
7. `android:layout_width="wrap_content"`和`android:layout_height="wrap_content"`：这两个属性设置`TextView`的宽度和高度为根据内容自适应。
8. `android:text="Hello World!"`：这个属性设置`TextView`显示的文本内容为"Hello World!"。
9. `<Button>`：这是一个`Button`视图，用于显示按钮。
10. `android:id="@+id/skip"`：这个属性为`Button`指定了一个唯一的ID。
11. `android:layout_width="wrap_content"`和`android:layout_height="wrap_content"`：这两个属性设置`Button`的宽度和高度为根据内容自适应。
12. `android:text="跳转"`：这个属性设置`Button`显示的文本内容为"跳转"。
13. `</LinearLayout>`：这是`LinearLayout`的闭合标签，表示`LinearLayout`的定义结束。

### 第二步：创建目标 `Activity` 布局

既然我们需要从主`Activity`跳转到我们创建的`Activity`，那么第二步我们就创建我们自己的`Activity`，那自然需要先去编写布局文件，下面是我们需要创建的`Activity`的布局文件：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/my_activity1"/>
</LinearLayout>
```

让我们逐行解释每个元素的作用：

1. `<?xml version="1.0" encoding="utf-8"?>`：这是XML文件的声明，指定了文件的版本和编码。
2. `<LinearLayout>`：这是根视图，使用了`LinearLayout`作为布局容器。
3. `xmlns:android="http://schemas.android.com/apk/res/android"`：这是Android命名空间的声明。
4. `android:layout_width="match_parent"`和`android:layout_height="match_parent"`：这两个属性设置根视图的宽度和高度为与父容器相匹配，即填充整个屏幕。
5. `android:orientation="vertical"`：这个属性设置`LinearLayout`的子视图在垂直方向上排列。
6. `android:gravity="center"`：这个属性设置`LinearLayout`的子视图在垂直和水平方向上居中对齐。
7. `<TextView>`：这是一个`TextView`视图，用于显示文本内容。
8. `android:layout_width="wrap_content"`和`android:layout_height="wrap_content"`：这两个属性设置`TextView`的宽度和高度根据内容自适应。
9. `android:text="@string/my_activity1"`：这个属性设置`TextView`显示的文本内容为字符串资源中的`my_activity1`。

这个`Activity`的需要显示字符串被我写到了`res/values/strings.xml`文件中：

```xml
<resources>
    <string name="app_name">My Application</string>
    <string name="my_activity1">My First Activity</string>
</resources>
```

### 第三步：编写 Java 交互逻辑

写完了我们需要创建的`Activity`布局文件，第三步我们需要在Java代码里面完成交互的逻辑编写：

```java
package com.example.myapplication;

import android.os.Bundle;

import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;

public class MyActivity extends AppCompatActivity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_my);
    }
}
```

让我们逐行解释每个元素的作用：

1. `public class MyActivity extends AppCompatActivity`：这是`MyActivity`类的定义，它继承自`AppCompatActivity`类。
