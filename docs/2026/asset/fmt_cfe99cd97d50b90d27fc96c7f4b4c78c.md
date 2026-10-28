
例如，开发者可以为屏幕尺寸大于普通屏幕的设备提供不同的布局资源，以充分利用额外的屏幕空间。还可以提供不同的字符串资源，以便根据设备的语言设置翻译界面中的文本。如需为不同设备配置提供这些不同资源，除默认资源以外，开发者还需提供备用资源。这个现在可能不太明白，但后面实战会讲到。

在Android应用程序中，`res`文件夹中的资源文件可通过引用其资源 ID 来应用该资源。所有资源 ID 都在项目的 `R` 类中进行定义，该类由 `aapt` 工具自动生成，可以通过这些ID值来访问和使用应用程序中的资源。

那`R`类和`res`文件夹的关系是怎么样的呢？

`R`类与`res`文件夹下的资源文件之间的关系如下：

1. `R`类的包名与应用程序的包名相同，即`com.example.myapp`。

2. `R`类中的子类与`res`文件夹下的子文件夹相对应，如`R$drawable`对应`drawable`文件夹，`R$layout`对应`layout`文件夹（`$`前面的类名表示表示子类，`$`后面的类名表示父类）。

3. `R`类中的每个子类都包含了对应资源文件的ID值，如`R$drawable`中包含了所有`drawable`文件夹下的图片的ID值。

4. `R`类中的ID值是由Android编译器（`aapt`工具）自动生成的，每个ID值都是唯一的，可以通过这些ID值来访问和使用对应的资源文件。

虽说所有资源 ID 都在项目的 `R` 类中进行定义，但是有的安卓应用程序中`R`类中有`attr`子类，而`res`下却没有`attr`子目录的。遇到这种情况不要觉得惊讶，下面就要着重讲讲`res`下的`values`子目录了。

所有资源ID都在项目的`R`类中进行定义，也就是说是可以通过资源ID来进行引用的资源那就会在项目的`R`类中进行定义。所以`res`文件夹下的子目录也就官方列出的那些，而且每个子目录都装有特定类型的资源，资源还不能乱放，那有的在`R`类中定义了资源ID的资源但`res`下没有对应资源的子类，如`attr`、`bool`等资源都会在`values`子目录中声明，前面官方文档中也提到了`values/`目录中的文件可描述多个资源，所有在`values`子目录中一个xml文件就描述了特定类型的多个资源。我们来看看这里面有哪些资源在`values`子目录中并且于`R`类中声明：

- Bool：包含布尔值的 XML 资源，保存在 的 XML 文件：`res/values-small/bools.xml`。

- color：包含颜色值（十六进制颜色）的 XML 资源，保存在 的 XML 文件：`res/values/colors.xml`。

- dimen：包含尺寸值（及度量单位）的 XML 资源，保存在 的 XML 文件：`res/values/dimens.xml`。

- id：提供应用资源和组件的唯一标识符的 XML 资源，保存在 的 XML 文件：`res/values/ids.xml`。

- integer：包含整数值的 XML 资源，保存在 的 XML 文件：`res/values/integers.xml`。

- integers：提供整数数组的 XML 资源，保存在 的 XML 文件：`res/values/integers.xml`。

- array：提供（可用于可绘制对象数组）的 XML 资源，保存在 的 XML 文件：`res/values/arrays.xml`。

这些虽说是安卓官方文档所展示的`values`文件夹中的资源类型，但其实`values`中的资源类型还不止这些，如`drawable`、`plural`等资源类型。还有一点，上面所述的资源类型`integers`和`array`都是通过名称进行引用的，而不是通过资源ID来进行引用的。

总的来说就是通过资源ID来进行引用的资源那就会在项目的`R`类中进行定义，在`R`类中定义的资源在`res`下的子目录中找不到，那就去`res/values`中寻找。有的资源类型没有在`R`类中定义的是因为该资源类型不是通过资源的ID去引用的，而是通过名称等其他方式进行的引用。

为了更好的理解这玩意实战中的作用，我们来进行次实战——将程序的默认启动activity改为我们自己的activity。

我这里使用的是`apktool`。

第一步：先把要反编译的apk文件放到`apktool`所在的文件夹

第二步：在此文件夹中打开`powershell`并且输入`cmd`

第三步：输入命令：`apktool.bat d 要反编译的apk文件名`

![](upload/attach/202307/926486_HQMC3DW2AFSDDY8.png)

完成上面步骤后在`apktool`文件夹中就会生成一个与要反编译的apk文件同名的文件夹，反编译的结果就在里面

![](upload/attach/202307/926486_96EAJKZ5NTAGMR5.png)

我们下一步需要通过反编译查看Java源码，这里使用的是`dex2jar`。

第一步：将apk文件修改为.zip后缀后将.dex文件解压到`dex2jar`目录下

第二步：在`dex2jar`目录下使用命令行中输入“`d2j-dex2jar.bat <dex文件名>`”命令运行`dex2jar`工具

第三步：等待`dex2jar`工具完成转换。转换完成后，会在`dex2jar`目录下生成一个与.dex文件同名的.jar文件，这个就是源码了

![](upload/attach/202307/926486_BYJK67NY2PJ6FKJ.png)

我们使用`jd-gui`打开刚才得到的源码，大概就是这个样子：

![](upload/attach/202307/926486_33QX27E9TSCMKSN.png)

这些工具在爱盘中都有存放，可以直接拿来使用。大家可能有些疑惑，为什么有那么多好使的工具，非要如此麻烦使用这三个工具呢？原因也很简单，虽然这三个工具用起来麻烦些，但是在显示上会更加的直观。

环境配置好，我们接下来就正式进行实战操作了：

第一步：写一个简单的Activity和布局文件。

Activity：

```
###### Class com.p005zj.wuaipojie.p006ui.MyActivity (com.zj.wuaipojie.ui.MyActivity)
.class public Lcom/zj/wuaipojie/ui/MyActivity;
.super Landroidx/appcompat/app/AppCompatActivity;
.source "MyActivity.java"
# direct methods
.method public constructor <init>()V
.registers 1
.line 7
invoke-direct {p0}, Landroidx/appcompat/app/AppCompatActivity;-><init>()V
return-void
.end method
# virtual methods
.method public onBackPressed()V
.registers 2
invoke-super {p0}, Landroidx/appcompat/app/AppCompatActivity;->onBackPressed()V
invoke-virtual {p0}, Landroid/app/Activity;->finish()V
return-void
.end method
.method protected onCreate(Landroid/os/Bundle;)V
.registers 6
const p1, 0x7f0b007e
invoke-virtual {p0, p1}, Lcom/zj/wuaipojie/ui/MyActivity;->setContentView(I)V
new-instance v0, Landroid/os/Handler;
invoke-direct {v0}, Landroid/os/Handler;-><init>()V
iput-object v0, p0, Lcom/zj/wuaipojie/ui/MyActivity;->handler:Landroid/os/Handler;
new-instance v0, Lcom/zj/wuaipojie/ui/MyActivity$1;
invoke-direct {v0, p0}, Lcom/zj/wuaipojie/ui/MyActivity$1;-><init>(Lcom/zj/wuaipojie/ui/MyActivity;)V
const-wide/16 v1, 0x12c
invoke-virtual {p0, v0, v1, v2}, Landroid/os/Handler;->postDelayed(Ljava/lang/Runnable;J)Z
return-void
.line 20
new-instance v0, Landroid/content/Intent;
const-class v1, Lcom/zj/wuaipojie/ui/MainActivity;
invoke-direct {v0, p0, v1}, Landroid/content/Intent;-><init>(Landroid/content/Context;Ljava/lang/Class;)V
invoke-virtual {p0, v0}, Landroid/content/Context;->startActivity(Landroid/content/Intent;)V
return-void
.end method
.method public onCreateOptionsMenu(Landroid/view/Menu;)Z
.registers 3
invoke-super {p0, p1}, Landroidx/appcompat/app/AppCompatActivity;->onCreateOptionsMenu(Landroid/view/Menu;)Z
const/4 v0, 0x1
return v0
.end method
.method public onOptionsItemSelected(Landroid/view/MenuItem;)Z
.registers 3
invoke-super {p0, p1}, Landroidx/appcompat/app/AppCompatActivity;->onOptionsItemSelected(Landroid/view/MenuItem;)Z
const/4 v0, 0x1
return v0
.end method
```

可能有的同学看起smali代码看起来比较累，那下面是该activity转成Java后的代码：

```Java
import android.content.Intent;
import android.os.Bundle;
import android.os.Handler;
import androidx.appcompat.app.AppCompatActivity;
public class MyActivity extends AppCompatActivity {
private Handler handler;
@Override
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity\_my);
handler = new Handler();
handler.postDelayed(new Runnable() {
@Override
public void run() {
Intent intent = new Intent(MyActivity.this, MainActivity.class);
startActivity(intent);
finish();
}
}, 5000); // 5秒后跳转
}
}
```

布局文件：

```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- 声明 XML 文件版本和编码方式。-->
<RelativeLayout
xmlns:android="http://schemas.android.com/apk/res/android"
android:layout\_width="match\_parent"
android:layout\_height="match\_parent">
<!-- RelativeLayout 布局，定义根布局。-->
<TextView
android:id="@+id/text\_view"
android:layout\_width="wrap\_content"
android:layout\_height="wrap\_content"
android:layout\_centerInParent="true"
android:text="Hello World!"
android:textSize="24sp"/>
<!-- TextView 组件，显示文本。-->
</RelativeLayout>
<!-- RelativeLayout 布局结束。-->
```

第二步：将Activity和布局文件放到反编译得到的文件夹中去，并定义好对应的资源ID。

找Activity文件的位置我是使用MT管理器去查找一个Activity文件，得到该文件的路径，然后我就顺着路径找了过去，就这样我将我写的Activity放到了里头：

![](upload/attach/202307/926486_DNVJ84VW6EYDZHT.png)

放好了Activity文件，那布局文件放在哪里呢？之前就提到了`res/layout`文件夹就包含了应用程序中所有的布局文件，那我们直接将布局文件丢`res/layout`文件夹里就行了，但到`res`文件夹中我看到了别的东西：

![](upload/attach/202307/926486_8F3TE8X3P4U2GF5.png)

可以看到上面有好多`layout-xx`的文件夹，还记得这是什么吗？`layout-xx`是Android应用程序中用于支持不同屏幕方向和多语言的布局文件目录，比如：`layout-land`：用于存放横屏方向下的布局文件，当设备旋转为横屏时，系统会自动加载该目录下的布局文件。

那我们直接将布局文件丢到`layout`文件夹中去：

![](upload/attach/202307/926486_CXZTYMJT4MY8BDA.png)

我们添加了一个布局文件，之前说过每一个资源文件都有对应的ID值，我们可以通过这些ID值来访问和使用对应的资源文件。那我们需要去给我们这个布局文件添加资源ID，这怎么搞呢？别急，还记得之前说过`R`类和`res`的子文件夹之间的关系吗？我们在`res/layout`文件夹中添加了布局文件，那么我们需要到`R$layout`文件中去添加这个布局文件的资源ID：

![](upload/attach/202307/926486_FXFHKA6ZW2G4V8S.png)

能看出来规律吗？其实在最底下添加一个资源ID，而这个资源ID的值就是上一个加一，添加完后一定要去`res/values\public.xml`文件中去确认一下资源ID是否具有唯一性，我这里添加的资源ID并不具备唯一性，后面在`values\public.xml`文件中添加资源ID时才发现。而且要注意资源ID是具有固定的格式：`0xPPTTEEEE`。

`PP`：表示某一个资源ID所在包的Package ID，包的命名空间，在资源ID中占两位大小，取值范围为[01, 7f]，第三方应用均为7f。

`TT`：表示某一个资源ID的资源类型，资源ID有anim、attr、color、dimen、drawable、id、layout等资源类型。

`EEEE`：表示某一个资源ID在所属资源类型中的偏移量。

我们添加资源ID时要注意所添加的资源ID是否具有唯一性以及是否符合资源ID的固定格式。

我们继续讲，这里有个情况，当时我看到`R$layout`中记录的资源ID与`res/layout`中的布局文件数量不对等，这是因为在某些情况下，反编译后可能会出现多个`R`类。

在开发者构建相关应用模块时，库模块会先编译到 AAR 文件中，然后再添加到应用模块中。因此，每个库都有自己的 `R` 类，并根据库的软件包名称命名。所需的所有软件包中都会创建从主模块和库模块生成的 `R` 类，包括主模块的软件包和库的软件包。

或许有的人不清楚什么是库模块，下面就讲一讲什么是库模块：

Android 库的结构与 Android 应用模块的结构相同。它为构建应用提供了所需的一切，包括源代码、资源文件和 Android 清单。

不过，Android 库将编译为开发者可以用作 Android 应用模块依赖项的 Android ARchive (AAR) 文件，而不是编译为在设备上运行的 APK。与 JAR 文件不同，AAR 文件会为 Android 应用提供以下功能：

- AAR 文件可以包含多项 Android 资源和一个清单文件，让开发者除了能够在 Kotlin 或 Java 类和方法中进行捆绑以外，还能够在布局和可绘制对象等共享资源中进行捆绑。

- AAR 文件可以包含 C/C++ 库，供应用模块的 C/C++ 代码使用。

库模块在以下情况下非常有用：

- 构建使用某些相同组件（例如 activity、服务或 UI 布局）的多个应用

- 构建存在于多个 APK 变体中的应用，例如使用相同核心组件的免费版本和付费版本

无论在哪种情况下，开发者都只需将想要重复使用的文件移到库模块中，然后将库添加为每个应用模块的依赖项。

多个库模块的资源与相关应用模块的资源进行合并难免会出现资源合并冲突的情况，为了避免资源合并冲突的情况，安卓做了以下方案：

1. 向 Android 应用模块添加对库模块的引用后，开发者可以设置它们的相对优先级。在构建时，库会按照优先级由低到高的顺序逐一与应用合并。

2. 如果库模块和应用模块中都定义了给定的资源 ID，系统会使用应用模块中的资源。如果多个 AAR 库之间发生冲突，系统会使用依赖项列表中优先列出的库（最靠近 `dependencies` 块顶部）中的资源。

3. 为避免资源冲突，也可能会使用非传递`R`类，这样做有助于确保每个模块的 `R` 类仅包含对其自身资源的引用，而不会从其依赖项中提取引用，从而帮助防止资源重复。如果无法做到这一点，开发者也会考虑使用模块独有（或在所有项目模块中具有唯一性）的前缀或其他一致的命名方案。
