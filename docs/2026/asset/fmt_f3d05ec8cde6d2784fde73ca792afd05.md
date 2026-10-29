
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
android:layout\_width="match\_parent"
android:layout\_height="match\_parent"
android:orientation="vertical"
android:gravity="center">
<TextView
android:layout\_width="wrap\_content"
android:layout\_height="wrap\_content"
android:text="Hello World!" />
<Button
android:id="@+id/skip"
android:layout\_width="wrap\_content"
android:layout\_height="wrap\_content"
android:text="跳转" />
</LinearLayout>
```

让我们逐行解释每个元素的作用：

1. `<?xml version="1.0" encoding="utf-8"?>`：这是XML文件的声明，指定了文件的版本和编码。

2. `<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"`是根视图的开始标签，并使用`xmlns:android`属性定义了一个XML命名空间。这个命名空间是用来引用Android的命名空间，以便可以使用Android提供的特定属性和元素。

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

既然我们需要从主Activity跳转到我们创建的Activity，那么第二步我们就创建我们自己的Activity，那自然需要先去编写布局文件，下面是我们需要创建的Activity的布局文件：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
android:layout\_width="match\_parent"
android:layout\_height="match\_parent"
android:orientation="vertical"
android:gravity="center">
<TextView
android:layout\_width="wrap\_content"
android:layout\_height="wrap\_content"
android:text="@string/my\_activity1"/>
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

这个Activity的需要显示字符串被我写到了`res/values/strings.xml`文件中：

```xml
<resources>
<string name="app\_name">My Application</string>
<string name="my\_activity1">My First Activity</string>
</resources>
```

写完了我们需要创建的Activity布局文件，第三步我们需要在Java代码里面完成交互的逻辑编写：

```Java
package com.example.myapplication;
import android.os.Bundle;
import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;
public class MyActivity extends AppCompatActivity {
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity\_my);
}
}
```

让我们逐行解释每个元素的作用：

1. `public class MyActivity extends AppCompatActivity`：这是`MyActivity`类的定义，它继承自`AppCompatActivity`类。

2. `@Override`：这是一个注解，表示该方法是对父类方法的重写。

3. `protected void onCreate(@Nullable Bundle savedInstanceState)`：这是`MyActivity`类的`onCreate()`方法，它是活动的入口点，会在活动被创建时被调用。

4. `super.onCreate(savedInstanceState)`：这是调用父类的`onCreate()`方法，确保父类的初始化工作得到执行。

5. `setContentView(R.layout.activity_my)`：这个方法设置活动的布局文件为`activity_my.xml`，通过传入布局文件的资源ID来实现。

我们完成了我们自己创建的Activity在Java代码里面的交互逻辑编写，第四步是主Activity在Java代码里面的交互逻辑编写：

```Java
package com.example.myapplication;
import androidx.appcompat.app.AppCompatActivity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
public class MainActivity extends AppCompatActivity {
@Override
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity\_main);
Button skip = findViewById(R.id.skip);
skip.setOnClickListener(new View.OnClickListener() {
@Override
public void onClick(View v) {
Intent intent = new Intent();
intent.setClass(MainActivity.this, MyActivity.class);
startActivity(intent);
}
});
}
}
```

让我们逐行解释每个元素的作用：

1. `public class MainActivity extends AppCompatActivity`：这是`MainActivity`类的定义，它继承自`AppCompatActivity`类。

2. `@Override`：这是一个注解，表示该方法是对父类方法的重写。

3. `protected void onCreate(Bundle savedInstanceState)`：这是`MainActivity`类的`onCreate()`方法，它是活动的入口点，会在活动被创建时被调用。

4. `super.onCreate(savedInstanceState)`：这是调用父类的`onCreate()`方法，确保父类的初始化工作得到执行。

5. `setContentView(R.layout.activity_main)`：这个方法设置活动的布局文件为`activity_main.xml`，通过传入布局文件的资源ID来实现。

6. `Button skip = findViewById(R.id.skip)`：这行代码通过调用`findViewById()`方法，找到布局文件中的`Button`视图，并将其实例化为名为`skip`的`Button`对象。

7. `skip.setOnClickListener(new View.OnClickListener() {...})`：这行代码为`Button`视图设置了一个点击事件的监听器，当用户点击按钮时，监听器中的代码会被执行。

8. `Intent intent = new Intent()`：这行代码创建了一个`Intent`对象，用于启动另一个活动。

9. `intent.setClass(MainActivity.this, MyActivity.class)`：这行代码设置`Intent`的目标活动为`MyActivity`类。

10. `startActivity(intent)`：这行代码启动了目标活动，即启动了`MyActivity`。

现在完成了主Activity在Java代码里面的交互逻辑编写，最后一步便是将我们自己编写的`Activity`在应用清单文件中声明：

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:tools="http://schemas.android.com/tools">
<application
android:allowBackup="true"
android:dataExtractionRules="@xml/data\_extraction\_rules"
android:fullBackupContent="@xml/backup\_rules"
android:icon="@mipmap/ic\_launcher"
android:label="@string/app\_name"
android:roundIcon="@mipmap/ic\_launcher\_round"
android:supportsRtl="true"
android:theme="@style/Theme.MyApplication"
tools:targetApi="31">
<activity
android:name=".MainActivity"
android:exported="true">
<intent-filter>
<action android:name="android.intent.action.MAIN" />
<category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
</activity>
<activity android:name=".MyActivity"/>
</application>
</manifest>
```

到现在简单的安卓APP已经写完了，下面让我们看看我们自己编写的简单的安卓APP：

![](upload/attach/202308/926486\_83R9UDQAKPJ4PNB.png)

点击跳转后：

![](upload/attach/202308/926486\_4A9UC42VJ4E6CVZ.png)

从结果来看，这个简单的安卓APP是编写成功了。但是我们开发安卓APP还未结束，还需将APK进行打包才可以将其安装在手机或者模拟器上，Android Studio打包APK可以参考下面这篇文章：

<a href="elink@1a8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4q4I4i4K6g2X3y4U0p5&6y4U0x3H3y4K6c8Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5J5y4U0f1I4x3K6b7I4z5b7`.`.">Android开发之打包APK详解\_android打包apk\_不服输的小乌龟的博客-CSDN博客</a>

将APK打包成功后便可以成功安装在我们自己的手机或者模拟器上了，到此我们自己写的这个简单的安卓APP也就编写完成了。

下一步我们需要将自己写的安卓APP逆向了做一件事情——>那就是替换掉点击跳转按钮后跳转的`Activity`。

首先软件逆向第一步找到关键代码的位置是重中之重，而我们要完成替换掉点击跳转按钮后跳转的`Activity`那首先需要先找到跳转按钮，如果我们需要找的内容并未写在Native层，没有被加密，也不是从服务器返回的，那就主要从这几个方向找：首先肯定是直接搜索字符串；其次通过工具获取到要找内容的资源ID后再去资源文件夹寻找相关的资源调用；还有就是通过一些相关的消息框、按钮等这些控件的调用代码来寻找位置；最次就是从程序入口处一步步往下追去寻得蛛丝马迹。

而我们写的安卓APP简单，后三者皆可使用，要问我为什么第一种在这不好使，那是因为显示在屏幕上的字符串全写在了资源文件中。这里我就打算使用最次的方法，因为我们写的这个简单，而且还好运用刚刚学会的smali汇编语言。

首先我们可以直接通过MT管理器的Activity记录功能直接获得入口点Activity的类名，然后便可搜索类名`com.example.myapplication.MainActivity`寻到以下内容：
