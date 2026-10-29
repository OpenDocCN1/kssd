

# [原创]java函数转Native化的一些实践

[萌新发帖]由于权限问题，我只能在“茶余饭后”发表帖子，希望这一次过后可以多多在Android安全中发。

目的：完成`MainActivity`中的`onCreate`和`replaceClassloader`两个java函数的jni等价实现

首先声明一下AS工程写的是dex动态加载，然后是为了去加载另一个`1.dex`里的`TestActivity`类

![](https://bbs.kanxue.com/upload/attach/202005/879358_ZNTFK5JX747JY3V.jpg)

接着使用GDA去看要加载的`1.dex`中查看

![](https://bbs.kanxue.com/upload/attach/202005/879358_UY3HA4G7NRKM4JA.jpg)

如果Native转化成功，则会在logcat窗口中看到`i am from TestActivity.onCreate`

开始JNI转化：

![](https://bbs.kanxue.com/upload/attach/202005/879358_BZSZSHCJQSWGRFG.jpg)

## 0x01 使用JNI实现onCreate函数的等价转化

①首先是注释掉`onCreate`，并添加声明

```
protected native void onCreate(Bundle SavedInstanceState);
```

②其中，`setContentView(R.layout.activity_main);`这个语句中会有`R`类，这是自动生成类，我们需要在GDA中观看它的反编译结果。

发现其中都是些静态`Static int`属性，一会儿用`Field`的时候选择`StaticInt`和`Static`

![](https://bbs.kanxue.com/upload/attach/202005/879358_AM4EKG9SW2SC3K4.jpg)

③转Jni层实现

`onCreate`的前两行都没什么大问题，一般的反射思路。

```
//第一行：super.onCreate(savedInstanceState);因此首先需要获取superclass

jclass AppCompatActivity_cls1 = env->FindClass("androidx/appcompat/app/AppCompatActivity");//已经知道类名

jclass MainActivity_cls1 = env->FindClass("com/kanxue/loaddex/MainActivity");

//调用这个函数protected void onCreate(@Nullable Bundle savedInstanceState)

jmethodID superclassOnCreate_mid = env->GetMethodID(AppCompatActivity_cls1,"onCreate","(Landroid/os/Bundle;)V");

env->CallNonvirtualVoidMethod(thiz,AppCompatActivity_cls1,superclassOnCreate_mid,Bundle);



//第二行：setContentView(R.layout.activity_main);

jmethodID setContentView_mid = env->GetMethodID(MainActivity_cls1,"setContentView","(I)V");

jclass R_layout_cls = env->FindClass("com/kanxue/loaddex/R$layout");

jfieldID activity_main_fid=env->GetStaticFieldID(R_layout_cls,"activity_main","I");//这里对应R$layout那个图看出来的

jint activity_main_value = env->GetStaticIntField(R_layout_cls,activity_main_fid);

env->CallVoidMethod(thiz,setContentView_mid,activity_main_value);
```

第三行，我们跟进并观察`getApplication`的函数实现

![](https://bbs.kanxue.com/upload/attach/202005/879358_XFRZVSKVJ9VKVN3.jpg)

![](https://bbs.kanxue.com/upload/attach/202005/879358_4NBTY2PAHGWSVGV.jpg)

```
//第三行：mApplication = this.getApplication();

//其中，public final Application getApplication()

jmethodID getApplication_mid=env->GetMethodID(MainActivity_cls1,"getApplication","()Landroid/app/Application;");

jobject mApplication = env->CallObjectMethod(thiz,getApplication_mid);
```

第四行，原理和第三行一致：

![](https://bbs.kanxue.com/upload/attach/202005/879358_F9Z9QXE96R5KXHC.jpg)

```
//第四行：appContext = this.getApplicationContext();

jmethodID getApplicationContext_mid=env->GetMethodID(MainActivity_cls1,"getApplicationContext","()Landroid/content/Context;");

jobject appContext = env->CallObjectMethod(thiz,getApplicationContext_mid);
```

第五行，调用了`copAssetAndWrite`，因此需要注意私有属性和`Static`类型

> `copyAssetAndWrite`函数
>
> ```
> private static boolean copyAssetAndWrite(String fileName) {
>     try {
>         File cacheDir = appContext.getCacheDir();
>         if (!cacheDir.exists()) {
>             cacheDir.mkdirs();
>         }
>         File outFile = new File(cacheDir, fileName);
>         if (!outFile.exists()) {
>             boolean res = outFile.createNewFile();
>             if (!res) {
>                 return false;
>             }
>         } else {
>             if (outFile.length() > 10) {
>                 return true;
>             }
>         }
>         InputStream is = appContext.getAssets().open(fileName);
>         FileOutputStream fos = new FileOutputStream(outFile);
>         byte[] buffer = new byte[1024];
>         int byteCount;
>         while ((byteCount = is.read(buffer)) != -1) {
>             fos.write(buffer, 0, byteCount);
>         }
>         fos.flush();
>         is.close();
>         fos.close();
>         return true;
>     } catch (IOException e) {
>         e.printStackTrace();
>     }
>     return false;
> }
> ```

![](https://bbs.kanxue.com/upload/attach/202005/879358_4D72X28SVGGVZU5.jpg)

```
//第五行：copyAssetAndWrite("1.dex");

//其中，private static boolean copyAssetAndWrite(String fileName)

jmethodID copyAssetAndWrite_mid=env->GetStaticMethodID(MainActivity_cls1,"copyAssetAndWrite","(Ljava/lang/String;)Z");

jstring arg1 = env->NewStringUTF("1.dex");

env->CallStaticBooleanMethod(MainActivity_cls1,copyAssetAndWrite_mid,arg1);
```

`java.io.File android.content.Context.getCacheDir()` on a null object reference

这里遇到了一个问题，也就是当我在JNI里实现`appContext`的赋值后怎么让JAVA层的`copyAssetAndWrite`函数知道我改了值呢?

解决方法：给`copyAssetAndWrite`新增一个参数，就是`Context`类型的`appContext`，即我们自己来传给它

```
//第五行：copyAssetAndWrite("1.dex");

//其中，private static boolean copyAssetAndWrite(String fileName)

jmethodID copyAssetAndWrite_mid=env->GetStaticMethodID(MainActivity_cls1,"copyAssetAndWrite","(Landroid/content/Context;Ljava/lang/String;)Z");

jstring arg1 = env->NewStringUTF("1.dex");

env->CallStaticBooleanMethod(MainActivity_cls1,copyAssetAndWrite_mid,appContext,arg1);
```

相对应地，`MainActivity`里把`copyAssetAndWrite`改成

```
private static boolean copyAssetAndWrite(Context appContext, String fileName)
```

第六行，可以看到`File`是个构造函数，因此在最后调用的时候要用`NewObject`，而且寻找`methodId`的时候要用`<init>`

![](https://bbs.kanxue.com/upload/attach/202005/879358_YWCRE8C2CXSZMZR.jpg)

因此`File`类，签名为：`()Ljava/io/File;`

![](https://bbs.kanxue.com/upload/attach/202005/879358_SRQN9JNBH84NQTR.jpg)![](https://bbs.kanxue.com/upload/attach/202005/879358_4JE2CEDTZ3MQ5RW.jpg)

```
//第六行：File dataFile = new File(this.getCacheDir(), "1.dex");

//其中，public File getCacheDir()      public File(File parent, String child)

jclass File_cls = env->FindClass("java/io/File");

jmethodID getCacheDir_mid=env->GetMethodID(AppActivity_cls1,"getCacheDir","()Ljava/io/File;");

jobject Cachedir = env->CallObjectMethod(thiz,getCacheDir_mid);

//对于构造函数File,函数名不是File而是<init>

jmethodID File_mid=env->GetMethodID(File_cls,"<init>","(Ljava/io/File;Ljava/lang/String;)V");

jobject dataFile = env->NewObject(File_cls,File_mid,Cachedir,arg1);
```

第七行，

```
//第七行：startTestActivityFirstMethod(this, dataFile.getAbsolutePath());

//其中，public void startTestActivityFirstMethod(Context context, String dexfilepath)

jmethodID startTestActivityFirstMethod_mid=env->GetMethodID(MainActivity_cls1,"startTestActivityFirstMethod",

                                                    "(Landroid/content/Context;Ljava/lang/String;)V");

//其中，public String getAbsolutePath()

jmethodID getAbsolutePath_mid=env->GetMethodID(MainActivity_cls1,"getAbsolutePath","()Ljava/lang/String;");

jstring arg2 = static_cast<jstring >( env->CallObjectMethod(thiz,getAbsolutePath_mid));

env->CallVoidMethod(MainActivity_cls1,startTestActivityFirstMethod_mid,thiz,arg2);
```

报错：`can't call void com.kanxue.loaddex.MainActivity.startTestActivityFirstMethod(android.content.Context, java.lang.String) on instance of java.lang.Class<com.kanxue.loaddex.MainActivity>`

经研究发现，是`CallNonvirtualVoid`和`Callvoid`之间的区别问题

`CallNonVirtualVoidMethod`是调用含子类重写父类函数的函数，`MainActivity`中的`onCreate`是重写，不是继承

`CallVoidMethod`调用的是当前对象自己的函数

因此正确应该是写成：

