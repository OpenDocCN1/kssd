
可以看出以上布局文件是一个垂直方向的线性布局（`LinearLayout`），包含一个相对布局（`RelativeLayout`）和一个文本视图（`TextView`）。

相对布局中包含一个按钮（`Button`），按钮的ID为`closeButton`，宽度和高度都是包裹内容（`wrap_content`），按钮上显示的文本内容来自于字符串资源文件（`@string/button_text_one`），按钮位于相对布局的右上角，距离父布局的右边和顶部都有8dp的边距。

文本视图的ID为`closeText`，宽度和高度都是包裹内容（`wrap_content`），文本内容来自于字符串资源文件（`@string/text_one`），文本的字体大小为18sp，文本居中显示。

整个布局的宽度和高度都是匹配父元素（`match_parent`），布局中的元素按照垂直方向排列。

有了布局文件自然要有`Activity`绑定布局，而这个`Activity`就是我们的主`Activity`，也是我们最开始需要作为广告弹出的`Activity`。

```java
package com.example.fourmodule;

import androidx.appcompat.app.AppCompatActivity;
import android.annotation.SuppressLint;
import android.content.Intent;
import android.os.Bundle;
import android.os.Handler;
import android.view.View;
import android.widget.Button;
import java.lang.Runnable;

public class MainActivity extends AppCompatActivity {
    private Handler handler; // 用于处理延迟任务的Handler对象
    private Runnable closeRunnable; // 延迟任务的Runnable对象

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // 初始化Handler和Runnable
        handler = new Handler();
        closeRunnable = new Runnable() {
            @Override
            public void run() {
                closeActivity();
            }
        };

        // 启动计时器，延迟五秒后关闭界面
        handler.postDelayed(closeRunnable, 5000);

        // 关闭按钮点击事件
        Button closeButton = findViewById(R.id.closeButton);
        closeButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // 取消计时器并关闭界面
                handler.removeCallbacks(closeRunnable);
                closeActivity();
            }
        });

        // 根布局点击事件
        @SuppressLint({"MissingInflatedId", "LocalSuppress"}) View rootView = findViewById(R.id.closeText);
        rootView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // 启动服务
                startService(new Intent(MainActivity.this, MyService.class));
            }
        });
    }

    private void closeActivity() {
        // 关闭界面
        finish();
        // 进入新的界面
        startActivity(new Intent(MainActivity.this, NewActivity.class));
    }
}
```

以上代码是一个名为`MainActivity`的活动类，继承自`AppCompatActivity`。它包含了一些操作和事件处理逻辑。

1. 在`onCreate()`方法中，首先通过`setContentView()`方法将布局文件`activity_main.xml`与该活动关联起来。

2. 初始化`Handler`和`Runnable`对象。`Handler`用于处理延迟任务，`Runnable`表示一个可执行的任务。在这里，我们创建了一个匿名内部类实现了`Runnable`接口的`closeRunnable`对象，它的`run()`方法会调用`closeActivity()`方法。

3. 启动计时器，使用`handler.postDelayed()`方法延迟5秒后执行`closeRunnable`的`run()`方法，也就是调用`closeActivity()`方法。

4. 通过`findViewById()`方法获取到关闭按钮的实例，并为其设置点击事件。当按钮被点击时，会取消计时器并调用`closeActivity()`方法关闭当前界面。

5. 同样地，通过`findViewById()`方法获取到根布局的实例，并为其设置点击事件。当根布局被点击时，会启动一个服务（`MyService`）。

6. `closeActivity()`方法用于关闭当前界面并启动一个新的界面（`NewActivity`）。`finish()`方法用于关闭当前活动，`startActivity()`方法用于启动新的活动。

我们来看一下这个`Activity`的整体布局是怎样的：

![](https://bbs.kanxue.com/upload/attach/202311/926486_YUQMTXZW8H4478Q.webp)

写好了`MainActivity`自然要注册这个`Activity`了，随即我在应用清单文件中对这个`Activity`进行了注册：

```xml
<activity
    android:name=".MainActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

现在第一个主`Activity`就算是完事了，可以看到在这个`Activity`的右上角有一个关闭按钮，只要一按或者5秒一到就会结束这个`Activity`并跳转到新的`Activity`，但如果在此期间点击了那串文字(为了省事我就用文字模拟为广告)就会触发服务，这样就有点像那开启广告了。而服务我是这样实现的：

```java
package com.example.fourmodule;

import android.app.Service;
import android.content.Intent;
import android.os.IBinder;
import android.widget.Toast;

import androidx.annotation.Nullable;

public class MyService extends Service {

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        // 在服务启动时执行的逻辑
        Toast.makeText(this, "服务已启动", Toast.LENGTH_SHORT).show();

        return START_STICKY;
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Toast.makeText(this, "服务已关闭", Toast.LENGTH_SHORT).show();
    }

}
```

以上代码是一个名为`MyService`的服务类，继承自`Service`。它包含了一些操作和事件处理逻辑。

1. `onStartCommand()`方法是在服务启动时执行的逻辑。在这里，我们使用`Toast.makeText()`方法显示一个短暂的提示消息，提示服务已经启动。

2. `onBind()`方法用于绑定服务。在这里，我们返回`null`，表示该服务不支持绑定。

3. `onDestroy()`方法是在服务销毁时执行的逻辑。在这里，我们使用`Toast.makeText()`方法显示一个短暂的提示消息，提示服务已经关闭。

服务这个组件想要正常使用自然也要注册的，所以我在应用清单文件中对这个服务组件进行了注册：

```xml
<service android:name=".MyService" />
```

如果我们没有触发这个服务，而是关闭了这个活动(广告)，那我们就会跳转到一个新的`Activity`，既然是`Activity`，那么就先来讲讲这个新`Activity`的布局文件：
