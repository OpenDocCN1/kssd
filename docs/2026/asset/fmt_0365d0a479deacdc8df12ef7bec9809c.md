
| 1  2  3  4  5  6  7 | `<application android:theme=" @style/AppTheme" android:label="@string/app_name" android:icon="@mipmap/ic_launcher" android:name="p008s.p009h.p010e.p011l.p012l.ApplicationC0083S" android:allowBackup="false" android:supportsRtl="true" android:usesCleartextTraffic="true" android:roundIcon="@mipmap/ic_launcher_round" android:appComponentFactory="s.h.e.l.l.A">` `<uses-library android:name="org.apache.http.legacy" android:required="false"/>` `<activity android:theme="@style/SplashActivityStyle" android:name="com.tangzhanglao.activity.splash.SplashActivity" android:launchMode="singleTask" android:screenOrientation="portrait">` `<intent-filter>` `<action android:name="android.intent.action.MAIN"/>` `<category android:name="android.intent.category.LAUNCHER"/>` `</intent-filter>` |

可见加壳后的程序名为：`android:name="p008s.p009h.p010e.p011l.p012l.ApplicationC0083S"`

且：`android:appComponentFactory="s.h.e.l.l.A"`

## 我们看下`ApplicationC0083S`

部分代码如下：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38 | `/* renamed from: s.h.e.l.l.S */` `这里是shells，不是shell A；` `public final class ApplicationC0083S extends Application {` `……` `/* renamed from: l；注意这个布尔变量f170l，在整个壳的保护中很关键，是起开关作用的 */` `public static boolean f170l = false;` `……` `public void onCreate() {` `……` `/* 再看下onCreate()入口函数中，这个函数是壳解密的入口，是Native层的。 */` `C0082N.m209ra(this, "com.tangzhanglao.TangzhanglaoApplication");` `if (f172n != null) {` `}` `super.onCreate();` `/* 我们可以来到C0082N类看下； */` `public final class C0082N {` `/* renamed from: al */` `public static native ClassLoader m205al(ClassLoader classLoader, ApplicationInfo applicationInfo, String str, String str2);` `public static native byte[] b2b(byte[] bArr, int i);` `/* renamed from: l */` `public static native boolean m206l(Application application, String str);` `/* renamed from: m */` `public static native void m207m(String str, int i);` `/* renamed from: r */` `public static native boolean m208r(Application application, String str);` `/* renamed from: ra；这里把要解密的程序包名传进去 */` `public static native boolean m209ra(Application application, String str);` `/* renamed from: sa */` `public static native void m210sa(String str, String str2);` |

从这里，往后就是常规爱加密壳保护的方法了；但是注意，这个壳的关键并不在这里，而是在解密程序之前还有一层保护；而且是动态保护的。关键就是上边的这个工厂类：`AppComponentFactoryC0080A`

首先说明一下：整个动态保护的关键，就是在爱加密壳之前又加了一层动态保护，这里的动态不是仅仅通过`DEXclassloader`来实现的，而是通过重写`AppComponentFactory`类，拦截了壳的整个解密过程，也就是说从壳程序加载一直到运行、解密整个过程都是被监控的；我们看下这个类的代码：

|  |  |
| --- | --- |
