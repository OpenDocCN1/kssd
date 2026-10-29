
# [原创]Xposed Hook技巧，代理abstract

题意：代理抽象就是说，对于一个抽象类的实例化参数，我们需要进行hook他的结果的操作，这种匿名类或者对象往往有很多构造函数的参数是我们无法构造的，或者传入空可能需要和正常的处理方式相悖，增加复杂度。（说了这么多，还是上代码直接）

目录：
1. 代理接口
2. 代理抽象类
3. 其它

## 1、代理接口

### 1.1 需要进行Xposed，需要拿到接口执行的对象的值。即`onExecuted`被调用的时候的`TestModel`的值

```java
public class TestModel {

    public int code;

    public String msg;

}
```

```java
public interface ITest {

    void onExecuted(TestModel paramT);

}
```

```java
public class MainActivity extends AppCompatActivity {

    @Override

    protected void onCreate(Bundle savedInstanceState) {

        super.onCreate(savedInstanceState);

        setContentView(R.layout.activity_main);



        ((Button)findViewById(R.id.btnTest2)).setOnClickListener(new View.OnClickListener() {

            @Override

            public void onClick(View view) {

                ITest testInter = new ITest() {

                    @Override

                    public void onExecuted(TestModel paramT) {

                        Toast.makeText(getBaseContext(), paramT.msg, Toast.LENGTH_LONG).show();

                    }

                };



                Test2(testInter);

            }

        });

    }



    public void Test2(ITest test){

        TestModel model = new TestModel();

        model.msg = "success";

        test.onExecuted(model);

    }

}
```

### 1.2、 一般实现，我们回先拿到匿名类hook他的`onExecuted`，然后初始化匿名类对象，`XposedHelper.callMethod`(`Test2`, 匿名对象），得到结果。

### 1.3、我们这里使用代理接口

```java
findAndHookMethod("com.android.testabstract.MainActivity", lpparam.classLoader, "onCreate", Bundle.class, new XC_MethodHook() {

    @Override

    protected void afterHookedMethod(final MethodHookParam param) throws Throwable {

        super.afterHookedMethod(param);



        final Object pthis = param.thisObject; // 拿到对象用于下一步调用

        hookInterface(lpparam.classLoader, pthis);

    }

});



private void hookInterface(final ClassLoader classLoader, final Object pthis){

    new Thread(new Runnable() {

        @Override

        public void run() {

            try {

                Class<?> class_ITest = classLoader.loadClass("com.android.testabstract.ITest");

                Object obj_proxy = Proxy.newProxyInstance(classLoader, new Class<?>[]{class_ITest}, new InvocationHandler() {

                    @Override

                    public Object invoke(Object o, Method method, Object[] objects) throws Throwable {

                        // 这里接可以拿到，接口执行的方法和执行的参数了。

                        Log.i("testProxy", "method " + method.getName());

                        Log.i("testProxy", "objects.length " + (objects != null ? objects.length : "null"));



                        return null;

                    }

                });



                XposedHelpers.callMethod(pthis, "Test2", obj_proxy);

            }

            catch (NoClassDefFoundError fe){

                Log.i("testProxy", "fe " + fe.getMessage());

                fe.printStackTrace();

            }

            catch (Exception e){

                Log.i("testProxy", "e " + e.getMessage());

                e.printStackTrace();

            }

        }

    }).start();

}
```

## 2、代理抽象类

### 2.1、代码定义，类对象沿用上面的即可。

```java
public abstract class TestAb<T> {

    public final void onExecuted(T paramT, Exception paramException) {

        onSuccess(paramT);

    }



    protected abstract void onSuccess(T paramT);

}
```

```java
public class MainActivity extends AppCompatActivity {

    @Override

    protected void onCreate(Bundle savedInstanceState) {

        super.onCreate(savedInstanceState);

        setContentView(R.layout.activity_main);

        

        ((Button)findViewById(R.id.btnTest)).setOnClickListener(new View.OnClickListener() {

            @Override

            public void onClick(View view) {

                TestAb<TestModel> testAb = new TestAb<TestModel>() {

                    @Override

                    protected void onSuccess(TestModel paramT) {

                        Toast.makeText(getBaseContext(), paramT.msg, Toast.LENGTH_LONG).show();

                    }

                };



                Test(testAb);

            }

        });

    }

    public void Test(TestAb testAb){

        TestModel model = new TestModel();

        model.msg = "success";

        testAb.onExecuted(model, null);

    }

}
```

### 2.2、一般处理方式，我们同样先拿到匿名的类（实例化抽象类的类），再hook他的`onSuccess`等待拿到结果，然后实例化匿名类，最后调用。

### 2.3 代理的处理方式：由于我们没有可以继承的抽象类，所以需要自己定义一个类库，然后实例化他，但是定义的类库仅仅用于使用，不用于实现，使用`compileOnly`引入。

```java
private void hookFail(final Object pthis){

    new Thread(new Runnable() {

        @Override

        public void run() {

            try {

                TestAb<TestModel> test = new TestAb<TestModel>() {

                    @Override

                    protected void onSuccess(TestModel paramT) {

                        Log.i("testAbProxy", "paramT " + paramT.msg);

                    }

                };



                XposedHelpers.callMethod(pthis, "Test", test);

            }

            catch (NoClassDefFoundError fe){

                Log.i("tt===tt", "fe " + fe.getMessage());

                fe.printStackTrace();

            }

            catch (Exception e){

                Log.i("tt===tt", "e " + e.getMessage());

                e.printStackTrace();

            }

        }

    }).start();

}
```

### 虽然看上去很短，没毛病，实际调用才发现。有问题。`NoClassDefineError`，怎么回事呢，，，由于apk的包里面，确实找不到这个类，所以就有这样的错误。怎么解决。

### 2.4、怎么办，解决。既然找不到，那么我们手动帮他找到，我们用自己的`ClassLoader`，载入自己的类，总没问题了吧。

```java
 // 定义实现抽象的对象。下面用到

public class TestTestAb extends TestAb<TestModel> {



    public TestTestAb(){

    }



    @Override

    protected void onSuccess(TestModel paramT) {

        Log.i("tt===tt", "hooked msg " + paramT.msg);

    }

}
```

```java
// 定义hook代码

private void hookTest2(final Object pthis){

    new Thread(new Runnable() {

        @Override

        public void run() {

            try {

                // 这里针对art，直接这样使用，dalvik可能就不一样了

                String path = "/data/app/com.android.testxpabs-1/base.apk";

                if(!new File(path).exists()){

                    path = "/data/app/com.android.testxpabs-2/base.apk";

                }



                PathClassLoader pathClassLoader = new PathClassLoader(path, pthis.getClass().getClassLoader());

                Class<?> class_ = pathClassLoader.loadClass("com.android.testxpabs.TestTestAb");

                Object obj_ = class_.newInstance();

                Class<?> class_TestModel = pathClassLoader.loadClass("com.android.testabstract.TestModel");



                findAndHookMethod("com.android.testxpabs.TestTestAb", pathClassLoader,

                        "onSuccess", class_TestModel, new XC_MethodReplacement() {

                            @Override

                            protected Object replaceHookedMethod(MethodHookParam param) throws Throwable {

                                Log.i("tt===tt", "hook replace " + param.args[0]);

                                return null;

                            }

                        });



                // 把当前APK加入

                XposedHelpers.callMethod(pthis, "Test", obj_);

            }

            catch (NoClassDefFoundError fe){

                Log.i("testAbProxy", "fe " + fe.getMessage());

                fe.printStackTrace();

            }

            catch (Exception e){

                Log.i("testAbProxy", "e " + e.getMessage());

                e.printStackTrace();

            }

        }

    }).start();

}
```

这样执行就没问题了，同时对于实现类我们也是可以hook的了。

### 2.5 当然这不是唯一解决办法，我们通用可以把，当前hook的apk，加入到待hookAPP的dex的`pathList`集合中，这样就可以直接用`pthis`对象的`ClassLoader`，进行加载和hook了。

对于这种实现方式，是不是眼熟呀。。。看看其他

## 3、其他

### 3.1、对Xposed的免重启更新也可以用他实现。
